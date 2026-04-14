================================================================================
🗑️ PROMPT #23 V1: JOB AUTOMÁTICO DE RETENCIÓN DE DATOS (RGPD ART. 5)
================================================================================

[Rol]
Especialista en cumplimiento RGPD y automatización de bases de datos con Supabase.

[Contexto]
E-commerce con Supabase que almacena datos personales. RGPD Artículo 5 exige:
- Los datos no se conserven más tiempo del necesario
- Plazos de retención definidos por finalidad
- Eliminación automática al vencer el plazo

[Problema actual]
Aunque existe variable `DATA_RETENTION_DAYS` y función `gdpr-delete`, NO hay un job automático que:
- Elimine usuarios anónimos inactivos
- Anonimice datos de usuarios que solicitaron borrado tras el plazo legal
- Elimine logs antiguos de auditoría
- Elimine sesiones expiradas

[Objetivo]
Crear sistema de jobs automáticos programados (cron) en Supabase que ejecuten políticas de retención según plazos legales.

---

## ⚖️ Plazos de retención por tipo de dato (España)

| Tipo de dato | Plazo máximo | Base legal | Acción al vencer |
|--------------|--------------|------------|------------------|
| Datos de facturación | 7 años | Ley General Tributaria (Art. 66-70) | Conservar (obligatorio) |
| Datos de pedido (histórico) | 7 años | Código de Comercio (Art. 30) | Anonimizar (borrar nombre, email, dirección) |
| Datos de navegación (logs) | 12 meses | LSSI + RGPD | Eliminar completamente |
| Usuarios inactivos | 3 años | Interés legítimo | Notificar + eliminar |
| Solicitudes GDPR | 3 años | Recomendación AEPD | Eliminar tras plazo |
| Datos de consentimiento | 5 años | RGPD Art. 7 (prueba) | Conservar (evidencia legal) |
| Datos de tarjeta (tokenizados) | Según contrato Stripe | Stripe gestiona | No almacenados localmente |

---

## 🗄️ Tablas necesarias

### Tabla: retention_policies (configuración)

```sql
CREATE TABLE retention_policies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_name TEXT NOT NULL UNIQUE,  -- 'profiles', 'audit_logs', 'sessions', etc.
  retention_days INTEGER NOT NULL,   -- días a conservar
  action TEXT NOT NULL,              -- 'delete', 'anonymize', 'notify'
  is_active BOOLEAN DEFAULT true,
  last_run_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  
  CONSTRAINT valid_action CHECK (action IN ('delete', 'anonymize', 'notify'))
);

-- Insertar políticas por defecto
INSERT INTO retention_policies (entity_name, retention_days, action) VALUES
  ('audit_logs', 365, 'delete'),                    -- 1 año
  ('profiles_inactive', 1095, 'notify'),            -- 3 años (notificar antes)
  ('sessions', 30, 'delete'),                       -- 30 días
  ('data_subject_requests', 1095, 'delete'),        -- 3 años
  ('rate_limit_logs', 30, 'delete');                -- 30 días

-- RLS: solo service role
ALTER TABLE retention_policies ENABLE ROW LEVEL SECURITY;
CREATE POLICY "service_role_only" ON retention_policies USING (false);