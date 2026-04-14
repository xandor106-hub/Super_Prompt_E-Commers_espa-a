# BIBLIOTECA DE PROMPTS - ARQUITECTURA FRONTEND PROFESIONAL
# Versión: 2.1
# Orden: Arquitectónico (Principios → Diagnóstico → Configuración → Features)

================================================================================
SECCIÓN 0: PRINCIPIOS ARQUITECTÓNICOS TRANSVERSALES
================================================================================
(Estrategias que aplican a TODOS los prompts posteriores)

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 0-A: ESPECIFICACIÓN GLOBAL DE EFICIENCIA                 │
│ Propósito: Reglas de estructura, lazy loading, TypeScript       │
│ Aplica a: TODOS los prompts que generan código                  │
│ Estado: ✅ PRINCIPIO FUNDACIONAL                                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 0-B: PROMPT #11 V1 - SEPARACIÓN ESTRICTA LÓGICA/VISUAL   │
│ Propósito: Aislar colores/fuentes/espacios de los componentes   │
│ Aplica a: TODOS los prompts de componentes visuales             │
│ Estado: ✅ PRINCIPIO FUNDACIONAL                                │
└─────────────────────────────────────────────────────────────────┘

================================================================================
SECCIÓN 1: DIAGNÓSTICO Y ARQUITECTURA BASE
================================================================================
(Primeros pasos obligatorios antes de codificar)

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 1: PROMPT #0 V1 - DIAGNÓSTICO PRE-INTEGRACIÓN BACKEND    │
│ Propósito: Extraer estructura real de tu Supabase               │
│ Genera: archivo .txt con tablas, columnas, Edge Functions       │
│ Estado: ✅ OBLIGATORIO PRIMERO                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 2: PROMPT #1 V1 - ARQUITECTURA FRONTEND COMPLETA         │
│ Propósito: Diseñar estructura de carpetas profesional           │
│ Genera: árbol de carpetas + dependencias npm                    │
│ Estado: ✅ OBLIGATORIO SEGUNDO                                  │
└─────────────────────────────────────────────────────────────────┘

================================================================================
SECCIÓN 2: CONFIGURACIÓN DEL ENTORNO
================================================================================
(Variables de entorno y seguridad)

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 3: PROMPT #10 V1 - CHECKLIST PRE-CONFIGURACIÓN .ENV      │
│ Propósito: Guía para recopilar todas las claves/URLs necesarias │
│ Genera: checklist imprimible + guía visual por proveedor        │
│ Estado: ✅ OBLIGATORIO TERCERO                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 4: PROMPT #9 V1 - CONFIGURACIÓN DE VARIABLES DE ENTORNO  │
│ Propósito: Generar archivos .env con validación Zod             │
│ Genera: .env.example, client.ts, server.ts, instrucciones       │
│ Estado: ✅ OBLIGATORIO CUARTO                                   │
└─────────────────────────────────────────────────────────────────┘

================================================================================
SECCIÓN 3: SISTEMA VISUAL
================================================================================
(Colores, fuentes, animaciones, fondos - La "piel" de la app)

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 5: PROMPT #8 V1 - SISTEMA VISUAL COMPLETO                │
│ Propósito: Definir paleta, tipografía, animaciones base         │
│ Genera: tailwind.config.js, globals.css, componentes base       │
│ Estado: ✅ OBLIGATORIO QUINTO                                   │
└─────────────────────────────────────────────────────────────────┘

================================================================================
SECCIÓN 4: FEATURES CORE (E-COMMERCE)
================================================================================
(Funcionalidades principales de la tienda)

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 6: PROMPT #2 V2 - TARJETAS DE PRODUCTO INTERACTIVAS      │
│ Propósito: Grid de productos con navegación a detalle           │
│ Genera: ProductCard, ProductGrid, Skeleton, useProducts         │
│ Estado: ✅ OBLIGATORIO SEXTO                                    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 7: PROMPT #3 V1 - PÁGINA DE DETALLE DE PRODUCTO          │
│ Propósito: Vista individual con galería, SEO, añadir carrito    │
│ Genera: page.tsx, generateStaticParams, generateMetadata        │
│ Estado: ✅ OBLIGATORIO SÉPTIMO                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 8: PROMPT #5 V1 - CHECKOUT CON STRIPE (PSD2/SCA)         │
│ Propósito: Flujo de pago seguro con Stripe Checkout             │
│ Genera: useCheckout, checkoutService, página éxito              │
│ Estado: ⚠️ OPCIONAL (solo si hay pagos)                         │
└─────────────────────────────────────────────────────────────────┘

================================================================================
SECCIÓN 5: CUMPLIMIENTO LEGAL (GDPR / ePrivacy)
================================================================================
(Obligatorio para operar en Europa)

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 9: PROMPT #4 V1 - BANNER DE COOKIES GDPR                 │
│ Propósito: Banner de consentimiento granular                    │
│ Genera: CookieConsentBanner, useConsent, consentService         │
│ Estado: ✅ OBLIGATORIO (ePrivacy)                               │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 10: PROMPT #6 V1 - PANEL DE PRIVACIDAD DEL USUARIO       │
│ Propósito: Exportación, portabilidad, derecho al olvido         │
│ Genera: página /cuenta/privacidad, useDataSubjectRights         │
│ Estado: ✅ OBLIGATORIO (GDPR Arts. 15-22)                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ BLOQUE 11: PROMPT #7 V1 - AUDITORÍA LADO CLIENTE                │
│ Propósito: Trazabilidad de eventos GDPR                         │
│ Genera: clientAudit.ts, Edge Function client-audit-log          │
│ Estado: ⚠️ RECOMENDADO (Accountability)                         │
└─────────────────────────────────────────────────────────────────┘

================================================================================
FIN DE LA BIBLIOTECA
================================================================================