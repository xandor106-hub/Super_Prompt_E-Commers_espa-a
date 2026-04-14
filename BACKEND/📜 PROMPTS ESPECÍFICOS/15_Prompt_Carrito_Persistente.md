================================================================================
🛒 PROMPT #15 V1: CARRITO DE COMPRA PERSISTENTE
================================================================================

[Rol]
Arquitecto Frontend especializado en gestión de estado para e-commerce con Next.js 14, Zustand y Supabase. Conocimiento de sincronización optimista y experiencia de usuario en flujos de compra.

[Contexto]
Frontend Next.js 14 App Router, TypeScript, Tailwind. Backend Supabase EU con tablas: profiles, products, orders. El sistema de checkout (Prompt #5) redirige a Stripe. Los productos tienen stock limitado (decrement_stock).

[Problema a resolver]
Sin gestión de carrito definida:
- El usuario pierde su carrito al cerrar el navegador
- No hay merge entre carrito de invitado y carrito post-login
- No se verifica el stock antes de añadir al carrito
- El checkout no sabe qué productos enviar a Stripe

[Decisión arquitectónica]
- Estado del carrito: Zustand (cliente) — rápido, sin latencia
- Persistencia: localStorage para invitados, Supabase para usuarios autenticados
- Sincronización: al hacer login, merge automático invitado → cuenta
- Verificación de stock: al añadir al carrito Y al iniciar checkout

---

## Tabla requerida en Supabase: cart_items

```sql
CREATE TABLE cart_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id) ON DELETE CASCADE,
  variant_id UUID REFERENCES product_variants(id) ON DELETE CASCADE NULLABLE,
  quantity INTEGER NOT NULL DEFAULT 1 CHECK (quantity > 0 AND quantity <= 99),
  added_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, product_id, variant_id) ← evita duplicados
);

-- RLS
ALTER TABLE cart_items ENABLE ROW LEVEL SECURITY;

CREATE POLICY "users_own_cart" ON cart_items
  FOR ALL USING (auth.uid() = user_id);
```

Esta tabla solo existe para usuarios autenticados. Invitados usan localStorage.

---

## Arquitectura del carrito

### Estructura de archivos

```
src/features/cart/
├── store/
│   └── cartStore.ts           ← Zustand store (estado + acciones)
├── hooks/
│   ├── useCart.ts             ← Hook público para componentes
│   └── useCartSync.ts         ← Sincronización Supabase ↔ localStorage
├── services/
│   └── cartService.ts         ← Llamadas a Supabase
├── components/
│   ├── CartDrawer.tsx          ← Panel lateral del carrito
│   ├── CartItem.tsx            ← Item individual en el carrito
│   ├── CartSummary.tsx         ← Resumen de precios con IVA
│   ├── AddToCartButton.tsx     ← Botón con estado de stock
│   └── CartIcon.tsx            ← Icono en navbar con contador
└── types/
    └── cart.types.ts
```

---

## Zustand Store — cartStore.ts

```typescript
interface CartItem {
  productId: string;
  variantId?: string;
  name: string;
  slug: string;
  image: string;
  priceCents: number;
  quantity: number;
  stockAvailable: number; ← para mostrar límite en UI
  taxRatePercent: number;
}

interface CartStore {
  items: CartItem[];
  isOpen: boolean;
  isLoading: boolean;

  // Acciones
  addItem: (product: CartItemInput, quantity?: number) => Promise<void>;
  removeItem: (productId: string, variantId?: string) => void;
  updateQuantity: (productId: string, quantity: number, variantId?: string) => void;
  clearCart: () => void;
  openCart: () => void;
  closeCart: () => void;

  // Computed (derivados, no almacenados)
  // getSubtotalCents(): number
  // getTaxCents(): number
  // getTotalCents(): number
  // getItemCount(): number
}
```

Usar `zustand/middleware/persist` con `localStorage` como storage.

---

## Lógica de merge al hacer login (IMPORTANTE)

```typescript
// En el hook useCartSync.ts, observar cambios de sesión:

useEffect(() => {
  const { data: { subscription } } = supabase.auth.onAuthStateChange(
    async (event, session) => {
      if (event === 'SIGNED_IN') {
        await mergeGuestCartWithUserCart(session.user.id);
      }
      if (event === 'SIGNED_OUT') {
        clearCart(); // Limpiar carrito de memoria al salir
        // NO limpiar localStorage — el usuario puede volver sin login
      }
    }
  );
  return () => subscription.unsubscribe();
}, []);

async function mergeGuestCartWithUserCart(userId: string) {
  const localItems = getCartFromLocalStorage();
  if (localItems.length === 0) {
    await loadCartFromSupabase(userId);
    return;
  }

  // Merge: si el producto ya existe en Supabase, sumar cantidades
  // Si no existe, insertarlo
  // Respetar siempre el stock máximo disponible
  await cartService.mergeItems(userId, localItems);
  clearLocalStorageCart();
}
```

---

## Verificación de stock en dos momentos

### Momento 1: Al añadir al carrito
```typescript
async function addItem(product: CartItemInput, quantity = 1) {
  // Verificar stock en tiempo real antes de añadir
  const { data: currentProduct } = await supabase
    .from('products')
    .select('stock_quantity')
    .eq('id', product.productId)
    .single();

  const inCartAlready = getQuantityInCart(product.productId);
  const totalRequested = inCartAlready + quantity;

  if (totalRequested > currentProduct.stock_quantity) {
    throw new Error(`Solo quedan ${currentProduct.stock_quantity} unidades disponibles`);
  }

  // Añadir al store de Zustand
  addToStore(product, quantity);

  // Si está autenticado, sincronizar con Supabase
  if (userId) await cartService.upsertItem(userId, product, quantity);
}
```

### Momento 2: Al iniciar checkout
La Edge Function `create-checkout-session` debe verificar stock de nuevo con `decrement_stock` (ver Prompt #13). El carrito en cliente puede estar desactualizado.

---

## CartDrawer — comportamiento esperado

- Desliza desde la derecha (slide-in)
- Se cierra con click fuera, tecla ESC, o botón X
- Muestra skeleton mientras carga
- Cada item: imagen, nombre, variante, precio unitario, selector de cantidad (+/-), botón eliminar
- Subtotal, IVA desglosado, total
- Botón "Ir al pago" → redirige al checkout (requiere login)
- Si no está logueado → redirige a /auth/login?redirect=/checkout

---

## CartSummary — desglose de IVA (obligatorio en España)

```typescript
// Los precios mostrados deben incluir IVA (precio con IVA es el legal en España para B2C)
// Pero el desglose debe mostrarse:

const subtotalCents = items.reduce((acc, item) =>
  acc + (item.priceCents * item.quantity), 0);

const taxCents = items.reduce((acc, item) => {
  const taxAmount = item.priceCents * item.quantity * (item.taxRatePercent / 100);
  return acc + Math.round(taxAmount / (1 + item.taxRatePercent / 100));
}, 0); // IVA incluido en el precio

// Mostrar:
// Subtotal (sin IVA): xx,xx €
// IVA (21%): xx,xx €
// TOTAL: xx,xx €
```

---

## Accesibilidad del carrito

- aria-label="Carrito de compra" en el drawer
- aria-live="polite" para anunciar cambios de cantidad
- Focus management: al abrir el drawer, mover el foco al primer elemento interactivo
- Al cerrar, devolver el foco al botón que lo abrió

---

## Output requerido

Por favor, genera:
1. SQL: tabla cart_items con RLS
2. cartStore.ts (Zustand con persistencia localStorage)
3. cartService.ts (operaciones Supabase: load, upsert, merge, delete)
4. useCart.ts (hook público con verificación de stock)
5. useCartSync.ts (sincronización login/logout)
6. CartDrawer.tsx (panel lateral completo)
7. CartItem.tsx (item con selector de cantidad)
8. CartSummary.tsx (totales con IVA desglosado)
9. AddToCartButton.tsx (con estados: añadir, añadiendo, sin stock)
10. CartIcon.tsx (para navbar con contador de items)

---

## Restricciones

- Máximo 99 unidades por ítem (check en DB y en UI)
- Precios siempre en céntimos internamente, formateados a euros en UI
- El IVA se muestra desglosado (obligatorio B2C en España)
- Sin carrito para usuarios con account_anonymized_at != null
- Los items del carrito no se conservan tras erasure GDPR (cascade delete en DB)
- 'use client' solo en componentes de UI — la lógica de cartService es isomórfica

[Aplicar Especificación Global de Eficiencia]
[Aplicar Prompt #11 — Separación Lógica/Visual]
