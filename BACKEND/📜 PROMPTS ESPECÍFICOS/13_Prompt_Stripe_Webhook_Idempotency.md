================================================================================
💳 PROMPT #13 V1: STRIPE WEBHOOK RESILIENTE CON IDEMPOTENCY (PSD2)
================================================================================

[Rol]
Integrador Stripe especializado en resiliencia de sistemas de pago para Europa. Experiencia en PSD2, manejo de eventos duplicados y rollback de transacciones.

[Contexto]
Backend Supabase EU con tablas: orders, order_items, products (con función decrement_stock). Edge Function stripe-webhook existente pero SIN protección contra eventos duplicados.

[Problema crítico a resolver]
Stripe puede enviar el mismo webhook múltiples veces (reintentos automáticos si no recibe 200 en <30 segundos). Sin idempotency:
- Se crean órdenes duplicadas
- Se descuenta stock dos veces
- El cliente recibe dos emails de confirmación
- La contabilidad queda corrupta

[Objetivo]
Reescribir la Edge Function stripe-webhook con idempotency completa, manejo de errores robusto, rollback de stock y logging a audit_logs.

---

## Tabla adicional requerida: webhook_events

```sql
CREATE TABLE webhook_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stripe_event_id TEXT UNIQUE NOT NULL,  ← CLAVE: evita duplicados
  event_type TEXT NOT NULL,
  processed_at TIMESTAMPTZ DEFAULT now(),
  status TEXT NOT NULL DEFAULT 'processed',  -- 'processed' | 'failed' | 'ignored'
  error_message TEXT,
  metadata JSONB DEFAULT '{}'
);

-- RLS: solo service role
ALTER TABLE webhook_events ENABLE ROW LEVEL SECURITY;
CREATE POLICY "service_role_only" ON webhook_events USING (false);
```

---

## Lógica de idempotency (CRÍTICA)

```typescript
// Patrón: Check → Process → Record (atómico)

async function handleWebhook(event: Stripe.Event) {

  // PASO 1: Verificar si ya procesamos este evento
  const { data: existing } = await supabase
    .from('webhook_events')
    .select('id, status')
    .eq('stripe_event_id', event.id)
    .single();

  if (existing) {
    // Ya procesado — devolver 200 sin hacer nada
    console.log(`Evento ${event.id} ya procesado. Status: ${existing.status}`);
    return new Response('Already processed', { status: 200 });
  }

  // PASO 2: Reservar el evento (INSERT antes de procesar)
  // Si otro proceso llega al mismo tiempo, el UNIQUE constraint lo bloqueará
  const { error: insertError } = await supabase
    .from('webhook_events')
    .insert({
      stripe_event_id: event.id,
      event_type: event.type,
      status: 'processing'
    });

  if (insertError) {
    // Otro proceso llegó primero — race condition resuelta por el constraint
    return new Response('Duplicate event', { status: 200 });
  }

  // PASO 3: Procesar según tipo de evento
  try {
    await processEvent(event);

    // Marcar como procesado exitosamente
    await supabase
      .from('webhook_events')
      .update({ status: 'processed' })
      .eq('stripe_event_id', event.id);

  } catch (error) {
    // Marcar como fallido para retry manual si es necesario
    await supabase
      .from('webhook_events')
      .update({
        status: 'failed',
        error_message: error.message
      })
      .eq('stripe_event_id', event.id);

    // Devolver 500 para que Stripe reintente
    throw error;
  }
}
```

---

## Eventos a manejar (con rollback)

### payment_intent.succeeded
1. Verificar idempotency (ver patrón arriba)
2. Buscar orden por stripe_payment_intent_id
3. Si la orden ya está en status 'paid' → ignorar (idempotency de negocio)
4. Actualizar order.status = 'paid'
5. Log en audit_logs
6. Encolar email de confirmación (ver Prompt #14 — Emails)

### payment_intent.payment_failed
1. Verificar idempotency
2. Actualizar order.status = 'payment_failed'
3. RESTAURAR stock: llamar a increment_stock() para cada order_item
4. Log en audit_logs con motivo de fallo

### charge.refunded
1. Verificar idempotency
2. Actualizar order.status = 'refunded'
3. Restaurar stock si el producto es físico (is_digital = false)
4. Log en audit_logs

### charge.dispute.created (chargeback)
1. Verificar idempotency
2. Actualizar order.status = 'disputed'
3. Log en audit_logs con metadata de la disputa
4. NO restaurar stock (la disputa puede resolverse a tu favor)

---

## Verificación de firma (OBLIGATORIA)

```typescript
// NUNCA procesar webhooks sin verificar la firma
const signature = req.headers.get('stripe-signature');
if (!signature) {
  return new Response('Missing signature', { status: 400 });
}

let event: Stripe.Event;
try {
  const body = await req.text(); // Leer como texto ANTES de parsear
  event = stripe.webhooks.constructEvent(
    body,
    signature,
    Deno.env.get('STRIPE_WEBHOOK_SECRET')!
  );
} catch (err) {
  // Firma inválida — posible ataque
  await logAudit('webhook_signature_failure', { error: err.message });
  return new Response('Invalid signature', { status: 400 });
}
```

⚠️ CRÍTICO: Leer el body como texto crudo antes de cualquier JSON.parse(). Si parseas primero y luego pasas el objeto, la verificación de firma falla siempre.

---

## Timeout y reintentos de Stripe

Stripe reintenta automáticamente si no recibe 200 en 30 segundos:
- Reintento 1: después de 5 minutos
- Reintento 2: después de 30 minutos
- Hasta 3 días / 15 reintentos

Tu Edge Function DEBE responder en < 10 segundos. Si la lógica es lenta, usa este patrón:

```typescript
// Responder 200 inmediatamente, procesar en background
const ctx = Deno.serveHttp(req);
ctx.respondWith(new Response('OK', { status: 200 }));

// Continuar procesando después de responder
await processEventAsync(event);
```

---

## Tabla de estados de orden (state machine)

```
pending → paid → shipped → delivered
    ↓         ↓
payment_failed  cancelled
    ↓         ↓
  (fin)    refunded
              ↓
          disputed
```

Añadir CHECK constraint en orders:
```sql
ALTER TABLE orders ADD CONSTRAINT valid_status
CHECK (status IN ('pending', 'paid', 'payment_failed', 'shipped',
                  'delivered', 'cancelled', 'refunded', 'disputed'));
```

---

## Output requerido

Por favor, genera:
1. SQL para tabla webhook_events con RLS
2. Edge Function stripe-webhook completa en TypeScript con:
   - Verificación de firma
   - Patrón idempotency completo
   - Handler para los 4 eventos descritos
   - Rollback de stock en fallos y reembolsos
   - Logging a audit_logs
3. Función increment_stock (inversa de decrement_stock para rollbacks)
4. CHECK constraint para estados de orden
5. Comandos curl para simular cada tipo de evento en desarrollo

---

## Variables de entorno requeridas

```
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...   ← diferente para cada endpoint registrado
SUPABASE_URL=...
SUPABASE_SERVICE_ROLE_KEY=...
```

⚠️ El STRIPE_WEBHOOK_SECRET es distinto para webhooks en producción y en local (stripe listen).

[Aplicar Especificación Global de Eficiencia]
