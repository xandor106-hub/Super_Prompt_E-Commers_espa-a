================================================================================
💳 PROMPT #5 V1: CHECKOUT CON STRIPE (PSD2/SCA)
================================================================================

[Rol]
Integrador Stripe especializado en PSD2/SCA Europa.

[Contexto]
Backend con Edge Functions create-checkout-session y stripe-webhook. Tablas orders y order_items.

[Objetivo]
Flujo checkout: botón pagar → Edge Function → Stripe Checkout → éxito/cancel.

[Requerimientos]
- Estado de carga
- Usuario autenticado
- Verificación pago en éxito

[Aplicar Especificación Global de Eficiencia]

[Formato de Salida]
- useCheckout.ts
- checkoutService.ts
- Página éxito
- CartSummary