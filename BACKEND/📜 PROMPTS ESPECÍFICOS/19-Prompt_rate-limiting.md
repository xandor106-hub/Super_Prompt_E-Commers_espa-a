================================================================================
🚦 PROMPT #19 V2: RATE LIMITING + PROTECCIÓN ANTI-ABUSO (VERSIÓN COMPLETA)
================================================================================

[Rol]
Ingeniero de Seguridad especializado en protección de APIs y prevención de abusos en entornos serverless (Supabase Edge Functions + Next.js).

[Contexto]
E-commerce con Supabase Edge Functions (stripe-webhook, send-email, record-consent, gdpr-export, request-erasure) y Next.js API routes. Actualmente sin protección, vulnerable a:
- Ataques de fuerza bruta en login
- Spam en endpoints de GDPR (exportación masiva)
- Abuso de checkout (crear sesiones sin pagar)
- Scraping de productos

[Decisión arquitectónica]
Para un proyecto de 200-500 usuarios iniciales, usar **tabla Supabase** en lugar de Redis externo:
- ✅ Sin dependencias adicionales
- ✅ Gratuito (dentro del plan de Supabase)
- ✅ Suficiente para el volumen esperado
- ⚠️ Si se escala a 10k+ usuarios, migrar a Upstash Redis

---

## 🗄️ Base de Datos: rate_limit_logs

```sql
CREATE TABLE rate_limit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  key TEXT NOT NULL,                    -- "ip:endpoint" o "user_id:endpoint"
  endpoint TEXT NOT NULL,               -- 'login', 'checkout', 'gdpr_export', etc.
  identifier TEXT NOT NULL,             -- IP o user_id
  attempts INTEGER DEFAULT 1,
  first_attempt_at TIMESTAMPTZ DEFAULT now(),
  last_attempt_at TIMESTAMPTZ DEFAULT now(),
  blocked_until TIMESTAMPTZ NULL,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Índices para consultas rápidas
CREATE INDEX idx_rate_limit_key ON rate_limit_logs(key);
CREATE INDEX idx_rate_limit_blocked ON rate_limit_logs(blocked_until) WHERE blocked_until IS NOT NULL;

-- Limpieza automática de registros antiguos (más de 30 días)
CREATE OR REPLACE FUNCTION cleanup_rate_limit_logs() RETURNS void AS $$
BEGIN
  DELETE FROM rate_limit_logs 
  WHERE created_at < NOW() - INTERVAL '30 days'
  AND blocked_until IS NULL;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- RLS: solo service role puede leer/escribir
ALTER TABLE rate_limit_logs ENABLE ROW LEVEL SECURITY;
CREATE POLICY "service_role_only" ON rate_limit_logs USING (false);