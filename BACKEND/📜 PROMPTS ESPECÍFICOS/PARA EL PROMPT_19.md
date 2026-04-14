📋 Límites por endpoint (ajustables)
Endpoint	Límite	Ventana	Bloqueo tras	Justificación
login	5 intentos	15 minutos	1 hora	Proteger cuentas
signup	3 intentos	1 hora	24 horas	Prevenir spam de registros
checkout	10 sesiones	1 hora	1 hora	Evitar abuso de Stripe
gdpr-export	3 solicitudes	24 horas	7 días	Costoso computacionalmente
gdpr-erasure	1 solicitud	30 días	30 días	Destructivo, requiere revisión
record-consent	50	1 minuto	5 minutos	Consentimiento frecuente pero limitado
send-email	10	1 hora	1 hora	Prevenir spam de emails
búsqueda productos	30	1 minuto	5 minutos	Evitar scraping agresivo
productos (GET)	100	1 minuto	-	Solo warning, no bloqueo


=========================================================================

Edge Function reusable: rate-limit

// supabase/functions/rate-limit/index.ts
import { createClient } from 'jsr:@supabase/supabase-js@2';

interface RateLimitConfig {
  key: string;           // Identificador único (IP + endpoint)
  endpoint: string;      // 'login', 'checkout', etc.
  limit: number;         // Intentos permitidos en la ventana
  windowSeconds: number; // Ventana de tiempo en segundos
  blockSeconds?: number; // Segundos de bloqueo si excede (default = windowSeconds * 2)
}

interface RateLimitResult {
  allowed: boolean;
  remaining: number;
  resetAt: Date;
  blockedUntil?: Date;
  retryAfterSeconds?: number;
}

export async function checkRateLimit(
  config: RateLimitConfig,
  supabaseAdmin: ReturnType<typeof createClient>
): Promise<RateLimitResult> {
  const now = new Date();
  const windowStart = new Date(now.getTime() - config.windowSeconds * 1000);
  const blockSeconds = config.blockSeconds ?? config.windowSeconds * 2;

  // 1. Verificar si ya está bloqueado
  const { data: blocked } = await supabaseAdmin
    .from('rate_limit_logs')
    .select('blocked_until')
    .eq('key', config.key)
    .gt('blocked_until', now.toISOString())
    .maybeSingle();

  if (blocked?.blocked_until) {
    const blockedUntil = new Date(blocked.blocked_until);
    const retryAfterSeconds = Math.ceil((blockedUntil.getTime() - now.getTime()) / 1000);
    return {
      allowed: false,
      remaining: 0,
      resetAt: blockedUntil,
      blockedUntil,
      retryAfterSeconds
    };
  }

  // 2. Contar intentos en la ventana
  const { count, data: existing } = await supabaseAdmin
    .from('rate_limit_logs')
    .select('*')
    .eq('key', config.key)
    .gte('last_attempt_at', windowStart.toISOString());

  const currentAttempts = count ?? 0;

  // 3. Si está dentro del límite, permitir y registrar
  if (currentAttempts < config.limit) {
    // Upsert: actualizar o insertar
    if (existing && existing.length > 0) {
      await supabaseAdmin
        .from('rate_limit_logs')
        .update({
          attempts: currentAttempts + 1,
          last_attempt_at: now.toISOString(),
          blocked_until: null
        })
        .eq('id', existing[0].id);
    } else {
      await supabaseAdmin
        .from('rate_limit_logs')
        .insert({
          key: config.key,
          endpoint: config.endpoint,
          identifier: config.key.split(':')[0],
          attempts: 1,
          first_attempt_at: now.toISOString(),
          last_attempt_at: now.toISOString()
        });
    }

    const resetAt = new Date(now.getTime() + config.windowSeconds * 1000);
    return {
      allowed: true,
      remaining: config.limit - currentAttempts - 1,
      resetAt
    };
  }

  // 4. Excedió el límite → bloquear
  const blockedUntil = new Date(now.getTime() + blockSeconds * 1000);
  
  if (existing && existing.length > 0) {
    await supabaseAdmin
      .from('rate_limit_logs')
      .update({ blocked_until: blockedUntil.toISOString() })
      .eq('id', existing[0].id);
  }

  return {
    allowed: false,
    remaining: 0,
    resetAt: blockedUntil,
    blockedUntil,
    retryAfterSeconds: blockSeconds
  };
}