================================================================================
🧪 PROMPT #22 V1: TESTS DE INTEGRACIÓN (CHECKOUT, RLS, WEBHOOKS)
================================================================================

[Rol]
Ingeniero de Calidad especializado en testing de e-commerce con Supabase, Stripe y Next.js.

[Contexto]
E-commerce completo con: Supabase (RLS, Edge Functions), Stripe (webhooks, checkout), Next.js 14. No hay tests automatizados. Esto es un RIESGO CRÍTICO para producción.

[Problema]
Sin tests automatizados:
- Un cambio en RLS puede exponer datos de otros usuarios
- Un cambio en webhook puede romper idempotencia
- Un cambio en checkout puede cobrar sin descontar stock
- No se puede hacer deploy con confianza

[Objetivo]
Crear suite completa de tests de integración que cubran:
1. RLS policies (seguridad: usuarios ven solo sus datos)
2. Flujo de checkout completo (stock, Stripe, webhook)
3. Edge Functions críticas (GDPR, devoluciones, facturación)
4. Webhook de Stripe (idempotencia, rollback)

---

## 🛠️ Stack de testing elegido

| Herramienta | Propósito | Justificación |
|-------------|-----------|----------------|
| **Vitest** | Framework de testing | Rápido, compatible con Next.js y Supabase |
| **@supabase/supabase-js** | Cliente para tests | Mismo que en producción |
| **@supabase/ssr** | Tests con sesión | Simular usuarios autenticados |
| **@testing-library/react** | Tests de componentes | Solo si hay UI crítica |
| **stripe-mock** | Mock de Stripe | Para tests offline (opcional) |
| **supabase-js/test-utils** | Helpers para RLS | Verificar políticas |

---

## 📁 Estructura de tests
