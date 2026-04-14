================================================================================
📊 PROMPT #20 V2: OBSERVABILIDAD Y MONITORIZACIÓN (VERSIÓN COMPLETA)
================================================================================

[Rol]
Ingeniero DevOps especializado en observabilidad, logging estructurado y alertas para e-commerce en producción.

[Contexto]
E-commerce con Next.js 14, Supabase Edge Functions, Stripe, Vercel. Ya existe tabla `audit_logs` pero no se está usando de forma sistemática. No hay monitorización activa.

[Problema a resolver]
- Cuando algo falla a las 3am, nadie lo sabe
- No hay métricas de salud del sistema
- Los errores no se registran de forma estructurada
- No hay alertas para eventos críticos (checkout fallido, stock bajo)

[Stack elegido]
| Herramienta | Propósito | Coste |
|-------------|-----------|-------|
| Sentry | Errores frontend + backend | Gratis (5k eventos/mes) |
| Supabase Audit Logs | Trazabilidad de acciones | Incluido |
| Vercel Analytics | Métricas de rendimiento | Gratis |
| Health checks | Estado del sistema | Propio |
| Email/Slack alerts | Notificaciones | Gratis (Resend + webhook) |

---

## 📋 Métricas clave a monitorear

### Métricas de negocio (alertas críticas)

| Métrica | Umbral | Acción |
|---------|--------|--------|
| Checkout fallido | > 5 en 10 min | Alerta email + Slack |
| Pago rechazado (Stripe) | > 10 en 1 hora | Alerta email |
| Stock bajo | < 5 unidades | Log + email diario |
| Usuario no puede login | > 20 en 5 min | Posible ataque → bloquear IPs |

### Métricas técnicas

| Métrica | Umbral | Acción |
|---------|--------|--------|
| Error 500 | > 0 en 1 min | Alerta crítica |
| Edge Function timeout | > 2 en 5 min | Revisar rendimiento |
| Webhook Stripe fallido | > 0 | Alerta inmediata |
| Tiempo respuesta > 3s | > 10 en 1 min | Revisar DB |

---

## 🗄️ Extensión de audit_logs (logging estructurado)

Ya tienes la tabla `audit_logs`. Solo necesitas usarla consistentemente.

### Función reusable para logging

```typescript
// src/lib/audit/logger.ts
import { createClient } from '@/lib/supabase/server';

type LogLevel = 'info' | 'warn' | 'error' | 'critical';
type LogAction =
  | 'checkout_initiated'
  | 'checkout_succeeded'
  | 'checkout_failed'
  | 'payment_webhook_received'
  | 'payment_webhook_failed'
  | 'stock_decremented'
  | 'stock_incremented'
  | 'stock_low'
  | 'email_sent'
  | 'email_failed'
  | 'gdpr_export_requested'
  | 'gdpr_erasure_requested'
  | 'login_success'
  | 'login_failed'
  | 'rate_limit_exceeded';

interface LogEntry {
  action: LogAction;
  level?: LogLevel;
  user_id?: string;
  metadata?: Record<string, unknown>;
  error_message?: string;
}

export async function logEvent(entry: LogEntry): Promise<void> {
  // Solo en producción
  if (process.env.NODE_ENV !== 'production') {
    console.log(`[AUDIT DEV] ${entry.action}`, entry.metadata);
    return;
  }

  try {
    const supabase = await createClient();
    await supabase.from('audit_logs').insert({
      action: entry.action,
      user_id: entry.user_id,
      metadata: {
        level: entry.level || 'info',
        ...entry.metadata,
        error: entry.error_message,
        timestamp: new Date().toISOString()
      }
    });
  } catch (error) {
    // Fallback: log a consola si falla la DB
    console.error('Failed to log to audit_logs:', error);
    console.log('[FALLBACK LOG]', entry);
  }
}