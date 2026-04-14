================================================================================
🔄 PROMPT #21 V1: SISTEMA DE DEVOLUCIONES Y REEMBOLSOS (LEGAL + STRIPE)
================================================================================

[Rol]
Especialista en cumplimiento legal de e-commerce en España (RDL 1/2007) e integrador Stripe para reembolsos.

[Contexto]
E-commerce con Stripe, facturación implementada (#17), webhooks resilientes (#13). Falta el sistema de devoluciones, que es OBLIGATORIO por ley en España (14 días naturales de desistimiento).

[Problema legal]
El Real Decreto Legislativo 1/2007 (Ley de Consumidores) establece:
- Derecho de desistimiento: 14 días naturales desde la recepción del producto
- Devolución sin justificación
- Reembolso en 14 días desde la solicitud
- Obligación de informar al consumidor
- Excepciones: productos personalizados, sellados, perecederos

[Objetivo]
Crear sistema completo de devoluciones con tracking de estado, validación de plazos, integración con Stripe refunds, y cumplimiento legal.

---

## 🗄️ Base de Datos

### Tabla: returns

```sql
CREATE TABLE returns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  return_number TEXT UNIQUE NOT NULL,  -- "RET-2026-000001"
  status TEXT NOT NULL DEFAULT 'pending',
  -- pending, approved, rejected, refunded, completed, expired
  reason TEXT NOT NULL,  -- 'defective', 'wrong_product', 'no_longer_want', 'other'
  reason_details TEXT,  -- explicación adicional
  items JSONB NOT NULL DEFAULT '[]',  -- [{order_item_id, quantity, reason}]
  
  -- Fechas críticas (legales)
  order_delivered_at TIMESTAMPTZ NOT NULL,  -- fecha de entrega del pedido
  requested_at TIMESTAMPTZ DEFAULT now(),   -- fecha de solicitud
  approved_at TIMESTAMPTZ,
  rejected_at TIMESTAMPTZ,
  refunded_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  
  -- Datos de devolución
  return_tracking_number TEXT,      -- nº seguimiento de la devolución
  return_carrier TEXT,              -- empresa que envía el cliente
  refund_amount_cents INTEGER,      -- importe reembolsado en céntimos
  refund_id TEXT,                   -- ID del refund en Stripe
  refund_reason TEXT,               -- motivo del reembolso para Stripe
  
  -- Validación legal
  is_within_legal_period BOOLEAN GENERATED ALWAYS AS (
    requested_at <= (order_delivered_at + INTERVAL '14 days')
  ) STORED,
  
  notes TEXT,  -- notas internas del administrador
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  
  CONSTRAINT valid_status CHECK (status IN (
    'pending', 'approved', 'rejected', 'refunded', 'completed', 'expired'
  )),
  CONSTRAINT valid_reason CHECK (reason IN (
    'defective', 'wrong_product', 'no_longer_want', 'other'
  )),
  CONSTRAINT valid_return_number_format CHECK (return_number ~ '^RET-[0-9]{4}-[0-9]{6}$')
);

-- Índices
CREATE INDEX idx_returns_user_id ON returns(user_id);
CREATE INDEX idx_returns_order_id ON returns(order_id);
CREATE INDEX idx_returns_status ON returns(status);
CREATE INDEX idx_returns_requested_at ON returns(requested_at);
CREATE UNIQUE INDEX idx_returns_number ON returns(return_number);

-- RLS
ALTER TABLE returns ENABLE ROW LEVEL SECURITY;

CREATE POLICY "users_view_own_returns" ON returns
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "users_create_own_returns" ON returns
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "service_role_all_returns" ON returns
  USING (false);  -- solo service role puede modificar

-- Trigger updated_at
CREATE TRIGGER returns_updated_at
  BEFORE UPDATE ON returns
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();