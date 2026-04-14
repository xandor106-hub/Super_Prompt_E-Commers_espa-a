# Super Prompt V4.0 — Cookies + Aviso Legal + Política de Privacidad (España)

Necesito implementar el cumplimiento legal completo para una tienda online en España, integrado con Supabase. Por favor, genera todo el código necesario.

## 1. Contexto Legal (España)
- País: España
- Leyes aplicables: RGPD, LOPDGDD, Ley 34/2002 LSSI (Aviso Legal), ePrivacy (Cookies)
- Autoridad de control: AEPD (Agencia Española de Protección de Datos)
- DPO: No requerido (menos de 5.000 usuarios, sin datos sensibles)

## 2. Base de Datos (Supabase) — Tablas necesarias

**Tabla: consent_logs**
- id (UUID, primary key)
- user_id (UUID, nullable)
- consent_type (TEXT: 'necessary', 'analytics', 'marketing', 'functional')
- status (BOOLEAN)
- version (TEXT)
- given_at (TIMESTAMPTZ)
- withdrawn_at (TIMESTAMPTZ, nullable)
- ip_address (INET)
- user_agent (TEXT)

**Tabla: privacy_policy_versions**
- id (UUID, primary key)
- version (TEXT, unique)
- effective_date (DATE)
- content (TEXT)
- is_current (BOOLEAN)

**Tabla: cookie_policy_versions**
- id (UUID, primary key)
- version (TEXT, unique)
- effective_date (DATE)
- cookie_categories (JSONB)
- is_current (BOOLEAN)

## 3. Edge Functions

**Función 1: get-legal-documents**
- Endpoint público
- Devuelve según parámetro 'type': 'aviso-legal', 'privacidad', 'cookies'

**Función 2: get-cookie-banner**
- Devuelve configuración actual del banner

**Función 3: record-cookie-consent**
- Almacena consentimiento en consent_logs

**Función 4: check-user-consent** (backend interno)
- Verifica si un usuario ha dado consentimiento

**Función 5: update-privacy-policy** (admin-only)
- Nueva versión de la política de privacidad

## 4. Frontend Componentes

**Componente 1: CookieBanner**
- Aparece en primera visita
- Botones: Aceptar todas, Rechazar todas, Configurar

**Componente 2: CookieSettingsModal**
- Lista de categorías
- Botón Guardar preferencias

**Componente 3: LegalPages**
- Páginas: Aviso Legal, Política de Privacidad, Política de Cookies

## 5. Contenido Obligatorio — Aviso Legal (Ley 34/2002 LSSI)

- Titular (nombre o razón social)
- NIF/CIF
- Domicilio social
- Email de contacto
- Datos de registro (si aplica)

## 6. Contenido Obligatorio — Política de Privacidad (RGPD + LOPDGDD)

- Responsable del tratamiento
- Finalidades del tratamiento
- Legitimación (contrato, consentimiento, interés legítimo)
- Destinatarios (Stripe, Supabase, mensajería)
- Plazos de conservación (pedidos: 7 años fiscal)
- Derechos (acceso, rectificación, supresión, portabilidad, oposición)
- Derecho a reclamar ante la AEPD
- Contacto para privacidad

## 7. Contenido Obligatorio — Política de Cookies

- Qué son las cookies
- Tipos de cookies que usa la web
- Lista detallada de cada cookie (nombre, duración, propósito)
- Cómo gestionarlas o deshabilitarlas
- Enlace a la guía de cookies de la AEPD

## 8. Output Requerido

Por favor, genera:
1. SQL completo para las tablas
2. Edge Functions (5 funciones) en TypeScript
3. HTML/CSS/JS para el banner, modal y páginas legales
4. Contenido de ejemplo para aviso legal, privacidad y cookies
5. RLS policies
6. Instrucciones de integración
7. Script de inicialización