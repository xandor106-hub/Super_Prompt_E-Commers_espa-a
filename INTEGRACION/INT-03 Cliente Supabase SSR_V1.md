================================================================================
🔐 PROMPT INT-03: CLIENTE SUPABASE SSR (Next.js 14 App Router)
================================================================================

[Rol]
Especialista en Next.js App Router + Supabase + cookies y autenticación.

[Contexto]
Proyecto Next.js 14 con App Router. Necesito integración segura con Supabase que funcione en Server Components y Client Components.

[Objetivo]
Generar la configuración completa del cliente Supabase para SSR.

[Archivos Requeridos]

## 1. src/lib/env/client.ts (validación Zod)
Variables públicas:
- NEXT_PUBLIC_SUPABASE_URL
- NEXT_PUBLIC_SUPABASE_ANON_KEY
- NEXT_PUBLIC_APP_URL
- NEXT_PUBLIC_APP_NAME
- NEXT_PUBLIC_DEFAULT_LANG
- NEXT_PUBLIC_CONSENT_VERSION

## 2. src/lib/env/server.ts (validación Zod)
Variables secretas (solo backend):
- SUPABASE_SERVICE_ROLE_KEY
- DATA_RETENTION_DAYS
- STRIPE_SECRET_KEY (opcional)
- STRIPE_WEBHOOK_SECRET (opcional)

## 3. src/lib/supabase/client.ts
- createBrowserClient con @supabase/ssr
- Singleton pattern para evitar múltiples instancias
- Solo para componentes 'use client'

## 4. src/lib/supabase/server.ts
- createServerClient con manejo de cookies
- Para usar en Server Components y Server Actions

## 5. src/middleware.ts
- Refrescar sesión en cada request
- Manejar redirección por autenticación
- Configuración de matcher

## 6. src/lib/supabase/admin.ts (opcional)
- Cliente con service_role key
- Solo para operaciones administrativas (NUNCA en cliente)

[Requisitos]
- Usar @supabase/ssr (no la versión antigua @supabase/auth-helpers)
- Zod para validación de variables de entorno
- Manejo de errores consistente
- Soporte para desarrollo local y producción (Vercel)

[Formato de Salida Esperado]
1. package.json (dependencias necesarias)
2. Código completo de src/lib/env/client.ts
3. Código completo de src/lib/env/server.ts
4. Código completo de src/lib/supabase/client.ts
5. Código completo de src/lib/supabase/server.ts
6. Código completo de src/middleware.ts
7. Código completo de src/lib/supabase/admin.ts (opcional)
8. Instrucciones de configuración paso a paso

[Restricciones]
- No exponer service_role key al cliente
- Incluir .gitignore para .env.local y .env.production
- Explicar cómo configurar variables en Vercel