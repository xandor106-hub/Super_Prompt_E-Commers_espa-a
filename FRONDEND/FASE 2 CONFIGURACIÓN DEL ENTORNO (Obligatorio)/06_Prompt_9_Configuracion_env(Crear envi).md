================================================================================
🔐 PROMPT #9 V1: CONFIGURACIÓN DE VARIABLES DE ENTORNO (.env)
================================================================================

[Rol]
Especialista DevSecOps Next.js + Supabase.

[Objetivo]
Generar configuración completa de variables de entorno.

[Variables Requeridas]
Supabase: URL, ANON_KEY, SERVICE_ROLE_KEY
Stripe: PUBLISHABLE_KEY, SECRET_KEY, WEBHOOK_SECRET
App: URL, NAME, DEFAULT_LANG
GDPR: CONSENT_VERSION, PRIVACY_URL, COOKIE_URL, CONTACT_EMAIL, RETENTION_DAYS

[Requisitos de Seguridad]
- NEXT_PUBLIC_ solo para públicas
- .env.local y .env.production en .gitignore
- Validación con Zod

[Formato de Salida]
1. .env.example completo
2. Líneas para .gitignore
3. src/lib/env/client.ts (Zod)
4. src/lib/env/server.ts (Zod)
5. Instrucciones Vercel
6. Script env:check