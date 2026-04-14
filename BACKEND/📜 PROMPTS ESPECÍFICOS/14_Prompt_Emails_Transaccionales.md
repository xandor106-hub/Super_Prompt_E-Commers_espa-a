================================================================================
📧 PROMPT #14 V1: EMAILS TRANSACCIONALES CON RESEND (OBLIGATORIO LEGAL)
================================================================================

[Rol]
Desarrollador Full-Stack especializado en comunicaciones transaccionales para e-commerce en España. Conocimiento de obligaciones legales de confirmación de pedido (RDL 1/2007).

[Contexto]
Backend Supabase EU, pagos Stripe, Edge Functions en Deno/TypeScript. Stack frontend Next.js 14. Los emails de confirmación de pedido tienen valor contractual en España según el Real Decreto Legislativo 1/2007.

[Tecnología elegida: Resend + React Email]
- Resend: API de emails moderna, GDPR-compliant, región EU disponible
- React Email: plantillas de email como componentes React (mismo lenguaje que el frontend)
- Alternativa aceptable: Postmark (también GDPR-compliant, excelente deliverability)

[Objetivo]
Crear sistema completo de emails transaccionales: plantillas, Edge Function de envío, integración con webhook de Stripe, y gestión de preferencias de email respetando el consentimiento GDPR.

---

## Emails a implementar (obligatorios)

### Email 1: Confirmación de pedido (OBLIGATORIO LEGAL)
Disparador: webhook payment_intent.succeeded
Contenido obligatorio según RDL 1/2007:
- Número de pedido
- Fecha y hora de la compra
- Resumen de productos (nombre, cantidad, precio unitario, IVA)
- Total con IVA desglosado
- Dirección de envío
- Plazo de entrega estimado
- Información sobre derecho de desistimiento (14 días naturales)
- Email y teléfono de contacto del vendedor

### Email 2: Pedido enviado
Disparador: actualización manual de order.status = 'shipped'
Contenido:
- Número de seguimiento
- Empresa de mensajería
- Enlace de tracking
- Fecha estimada de entrega

### Email 3: Solicitud GDPR recibida
Disparador: creación de data_subject_request
Contenido:
- Tipo de solicitud (acceso, supresión, portabilidad)
- Fecha de recepción
- Plazo legal de respuesta (30 días)
- Referencia de la solicitud
- Contacto para seguimiento

### Email 4: Reembolso procesado
Disparador: webhook charge.refunded
Contenido:
- Importe reembolsado
- Plazo de llegada a la cuenta (5-10 días hábiles típico)
- Número de pedido original

---

## Arquitectura técnica

### Estructura de archivos

```
src/
├── emails/                          ← Plantillas React Email
│   ├── OrderConfirmation.tsx
│   ├── OrderShipped.tsx
│   ├── GdprRequestReceived.tsx
│   ├── RefundProcessed.tsx
│   └── components/
│       ├── EmailLayout.tsx          ← Layout base con logo y footer legal
│       ├── OrderSummaryTable.tsx    ← Tabla de productos reutilizable
│       └── LegalFooter.tsx          ← Footer obligatorio RGPD + baja
│
supabase/functions/
└── send-email/
    └── index.ts                     ← Edge Function unificada de envío
```

### Edge Function: send-email

```typescript
// Tipos de email soportados
type EmailType =
  | 'order_confirmation'
  | 'order_shipped'
  | 'gdpr_request'
  | 'refund_processed';

interface SendEmailPayload {
  type: EmailType;
  to: string;
  data: Record<string, unknown>; // datos específicos de cada template
}
```

La función:
1. Valida el tipo de email y los datos requeridos con Zod
2. Verifica que el destinatario existe en profiles
3. Comprueba que el usuario NO ha marcado marketing_consent = false para emails de marketing (los transaccionales siempre se envían — no requieren consentimiento)
4. Renderiza la plantilla React Email a HTML
5. Envía mediante API de Resend
6. Registra el envío en audit_logs

---

## EmailLayout: requisitos legales en footer

Todo email debe incluir en el footer:
- Nombre comercial y razón social
- NIF/CIF
- Dirección física
- Email de contacto
- Enlace a Política de Privacidad
- Enlace para darse de baja de comunicaciones (solo marketing — los transaccionales no tienen baja)
- "Este email es una comunicación transaccional relacionada con tu pedido. No puedes darte de baja de este tipo de comunicaciones."

---

## Consideraciones GDPR para emails

### Emails transaccionales (confirmación, envío, reembolso):
- NO requieren consentimiento — base legal: ejecución de contrato (Art. 6.1.b RGPD)
- SIEMPRE se envían aunque marketing_consent = false
- SÍ deben mencionarse en la Política de Privacidad como tratamiento de datos

### Emails de marketing (newsletters, promociones):
- REQUIEREN consentimiento explícito (marketing_consent = true en profiles)
- Verificar SIEMPRE antes de enviar
- Incluir enlace de baja funcional

---

## Configuración de Resend

```typescript
// Variables de entorno requeridas
RESEND_API_KEY=re_...
EMAIL_FROM=noreply@tudominio.com     ← dominio verificado en Resend
EMAIL_FROM_NAME=Nombre de tu Tienda
RESEND_REGION=eu-west-1             ← datos en EU (RGPD)
```

Verificar dominio en Resend antes de producción (requiere registros DNS: SPF, DKIM, DMARC).

---

## Integración con stripe-webhook

En la Edge Function stripe-webhook, después de actualizar el pedido a 'paid':

```typescript
// Llamar a send-email de forma no bloqueante
const emailPayload = {
  type: 'order_confirmation',
  to: userEmail,
  data: {
    order_id: order.id,
    order_number: order.id.slice(0, 8).toUpperCase(),
    items: orderItems,
    total_cents: order.total_amount,
    shipping_address: order.shipping_address_snapshot,
    created_at: order.created_at
  }
};

// No usar await — no queremos que el email bloquee la respuesta del webhook
fetch(`${Deno.env.get('SUPABASE_URL')}/functions/v1/send-email`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(emailPayload)
}).catch(err => console.error('Email send failed:', err));
```

---

## Output requerido

Por favor, genera:
1. Instalación y configuración de Resend + React Email
2. Componente EmailLayout.tsx con footer legal en español
3. Componente OrderConfirmation.tsx completo (el más importante legalmente)
4. Componente OrderShipped.tsx
5. Componente GdprRequestReceived.tsx
6. Componente RefundProcessed.tsx
7. Edge Function send-email/index.ts completa con validación Zod
8. Integración en stripe-webhook (snippet)
9. Variables de entorno a añadir a .env.example
10. Instrucciones para verificar dominio en Resend (DNS records)

---

## Restricciones

- Los emails deben funcionar en Gmail, Outlook y Apple Mail (los 3 principales en España)
- Usar tablas HTML para layouts de email (no flexbox/grid — compatibilidad)
- Textos en español
- Sin imágenes externas en el email de confirmación (evitar tracking pixels no consentidos)
- El asunto del email debe incluir el número de pedido para fácil búsqueda

[Aplicar Especificación Global de Eficiencia]
