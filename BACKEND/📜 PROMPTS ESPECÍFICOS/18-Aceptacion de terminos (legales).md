================================================================================
📜 PROMPT #18 V1: ACEPTACIÓN DE TÉRMINOS Y CONDICIONES (LEGAL)
================================================================================

[Rol]
Especialista en cumplimiento LSSI + RGPD en e-commerce.

[Contexto]
Checkout con Stripe ya implementado. Falta registro legal de aceptación de términos.

[Objetivo]
Implementar aceptación obligatoria de términos antes del pago + registro verificable.

---

## ⚖️ Requisito legal

Antes de pagar:
- Usuario DEBE aceptar términos
- Debe poder demostrarse (timestamp + versión)

---

## 🗄️ Base de Datos

### Tabla: terms_acceptance

- id (UUID)
- user_id (UUID)
- terms_version (TEXT)
- accepted_at (TIMESTAMPTZ)
- ip_address (INET)
- user_agent (TEXT)

---

## 🎯 Frontend

### En checkout:

Checkbox obligatorio:

[ ] Acepto los Términos y Condiciones

- No permitir pago sin marcar
- Enlace a /terminos-condiciones

---

## ⚙️ Edge Function: record-terms-acceptance

Input:
- user_id
- terms_version

Acción:
- Insert en terms_acceptance

---

## 🔒 Integración con checkout

ANTES de crear sesión Stripe:

1. Verificar aceptación
2. Si no → error

---

## 🔁 Versionado

Cuando cambien términos:
- Nueva versión
- Obligar aceptación nueva

---

## 🔐 RLS

- Usuario solo ve sus registros
- INSERT: permitido autenticado
- UPDATE/DELETE: NO permitido
