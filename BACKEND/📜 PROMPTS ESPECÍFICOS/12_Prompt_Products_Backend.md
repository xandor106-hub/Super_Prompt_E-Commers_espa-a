================================================================================
🛍️ PROMPT #12 V1: TABLA PRODUCTS — BACKEND SUPABASE (ESPAÑA)
================================================================================

[Rol]
Arquitecto de Base de Datos especializado en e-commerce con Supabase, RLS y cumplimiento legal europeo.

[Contexto]
Backend Supabase EU (Frankfurt) con tablas existentes: profiles, orders, order_items, consent_logs, audit_logs, data_subject_requests. Stack de pagos: Stripe. País: España.

[Objetivo]
Crear el esquema completo de productos con control de inventario concurrente, categorías, variantes opcionales, y todas las políticas RLS necesarias para operar de forma segura.

[Problema que resuelve]
Sin una tabla products bien definida, el frontend no tiene origen de datos real. Sin control de concurrencia, dos usuarios pueden comprar el último stock simultáneamente (overselling).

---

## Esquema requerido

### Tabla: categories
- id (UUID, primary key, default gen_random_uuid())
- name (TEXT, not null)
- slug (TEXT, unique, not null)
- description (TEXT)
- parent_id (UUID, nullable, self-reference → categories.id) ← para subcategorías
- is_active (BOOLEAN, default true)
- created_at (TIMESTAMPTZ, default now())

### Tabla: products
- id (UUID, primary key, default gen_random_uuid())
- name (TEXT, not null)
- slug (TEXT, unique, not null) ← para URLs amigables /productos/[slug]
- description (TEXT)
- short_description (TEXT) ← para tarjetas de producto
- price_cents (INTEGER, not null, check price_cents >= 0) ← siempre en céntimos
- compare_at_price_cents (INTEGER, nullable) ← precio tachado si hay oferta
- stock_quantity (INTEGER, not null, default 0, check stock_quantity >= 0)
- sku (TEXT, unique, nullable) ← código interno
- images (JSONB, default '[]') ← array de URLs [{url, alt, position}]
- category_id (UUID, references categories.id)
- is_active (BOOLEAN, default false) ← por defecto inactivo hasta publicar
- is_digital (BOOLEAN, default false) ← para productos sin envío físico
- weight_grams (INTEGER, nullable) ← para cálculo de envío
- tax_rate_percent (NUMERIC(5,2), default 21.00) ← IVA España (21%, 10%, 4%)
- metadata (JSONB, default '{}') ← campos adicionales sin migración
- created_at (TIMESTAMPTZ, default now())
- updated_at (TIMESTAMPTZ, default now())

### Tabla: product_variants (opcional — para tallas, colores, etc.)
- id (UUID, primary key, default gen_random_uuid())
- product_id (UUID, references products.id, on delete cascade)
- name (TEXT, not null) ← ej: "Talla M / Color Rojo"
- sku (TEXT, unique, nullable)
- price_cents (INTEGER, nullable) ← si null, hereda de products.price_cents
- stock_quantity (INTEGER, not null, default 0, check stock_quantity >= 0)
- attributes (JSONB, default '{}') ← {size: "M", color: "red"}
- is_active (BOOLEAN, default true)
- created_at (TIMESTAMPTZ, default now())

---

## Control de inventario concurrente (CRÍTICO)

### Función SQL con bloqueo optimista
Crear función `decrement_stock` que use SELECT FOR UPDATE para evitar overselling:

```sql
CREATE OR REPLACE FUNCTION decrement_stock(
  p_product_id UUID,
  p_quantity INTEGER
) RETURNS BOOLEAN AS $$
DECLARE
  v_stock INTEGER;
BEGIN
  -- Bloqueo exclusivo de la fila durante la transacción
  SELECT stock_quantity INTO v_stock
  FROM products
  WHERE id = p_product_id
  FOR UPDATE;

  IF v_stock IS NULL THEN
    RAISE EXCEPTION 'Producto no encontrado';
  END IF;

  IF v_stock < p_quantity THEN
    RETURN FALSE; -- Stock insuficiente
  END IF;

  UPDATE products
  SET stock_quantity = stock_quantity - p_quantity,
      updated_at = now()
  WHERE id = p_product_id;

  RETURN TRUE;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

Esta función se llama desde la Edge Function de checkout ANTES de crear la orden en Stripe. Si devuelve FALSE, no se inicia el pago.

---

## RLS Policies

### products:
- SELECT: is_active = true (público, sin autenticación)
- SELECT admin: service role ve todos (activos e inactivos)
- INSERT: service role only (gestión desde panel admin)
- UPDATE: service role only
- DELETE: NOT ALLOWED — usar is_active = false (soft delete)

### categories:
- SELECT: is_active = true (público)
- INSERT/UPDATE/DELETE: service role only

### product_variants:
- SELECT: is_active = true AND product activo (público)
- INSERT/UPDATE/DELETE: service role only

---

## Trigger: updated_at automático

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER products_updated_at
  BEFORE UPDATE ON products
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

---

## Índices de rendimiento

```sql
-- Búsqueda por slug (URL de producto)
CREATE UNIQUE INDEX idx_products_slug ON products(slug) WHERE is_active = true;

-- Filtrado por categoría activa
CREATE INDEX idx_products_category ON products(category_id) WHERE is_active = true;

-- Búsqueda full-text en español
CREATE INDEX idx_products_search ON products
  USING gin(to_tsvector('spanish', name || ' ' || coalesce(description, '')));

-- SKU lookup
CREATE UNIQUE INDEX idx_products_sku ON products(sku) WHERE sku IS NOT NULL;
```

---

## Datos de ejemplo (seed)

Insertar al menos:
- 3 categorías (electrónica, ropa, hogar)
- 10 productos de ejemplo con stock > 0
- 2 productos con variantes
- 1 producto sin stock para probar el bloqueo

---

## Integración con Edge Function de checkout

La Edge Function `create-checkout-session` debe:
1. Llamar a `decrement_stock(product_id, quantity)` para CADA ítem
2. Si cualquier llamada devuelve FALSE → cancelar todo y devolver 409 Conflict
3. Si todas devuelven TRUE → crear sesión en Stripe
4. Si Stripe falla → restaurar stock con `increment_stock` (función inversa)

---

## Output requerido

Por favor, genera:
1. SQL completo: CREATE TABLE categories, products, product_variants
2. Función decrement_stock + increment_stock (para rollback)
3. Todas las políticas RLS con CREATE POLICY
4. Trigger updated_at
5. Índices
6. Seed de datos de ejemplo
7. Tipos TypeScript generados (equivalente a supabase gen types)

---

## Restricciones importantes

- Precios SIEMPRE en céntimos (INTEGER), nunca DECIMAL/FLOAT
- IVA almacenado en producto (no calculado al vuelo) para snapshots de órdenes
- Nunca DELETE en products — usar is_active = false
- El slug debe ser URL-safe: solo minúsculas, guiones, sin acentos
- Las imágenes como JSONB array, no tabla separada (evita JOINs innecesarios para listados)

[Aplicar Especificación Global de Eficiencia]
