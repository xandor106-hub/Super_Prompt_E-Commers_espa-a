================================================================================
🍪 PROMPT #4 V1: BANNER DE COOKIES GDPR (ePrivacy)
================================================================================

[Rol]
Experto en cumplimiento ePrivacy y GDPR frontend.

[Contexto]
Backend con tabla consent_logs y Edge Function record-consent.

[Objetivo]
Crear CookieConsentBanner que: muestre sin consentimiento, bloquee cookies no esenciales, granularidad, envíe a Edge Function, localStorage.

[Categorías]
necessary, functional, analytics, marketing.

[Requerimientos]
- 'use client'
- Accesible teclado
- Sin dependencias externas

[Aplicar Especificación Global de Eficiencia]

[Formato de Salida]
- CookieConsentBanner.tsx
- useConsent.ts
- consentService.ts
- Lógica bloqueo scripts