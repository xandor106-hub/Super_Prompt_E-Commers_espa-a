================================================================================
🔐 PROMPT #6 V1: PANEL DE PRIVACIDAD DEL USUARIO (GDPR)
================================================================================

[Rol]
Desarrollador especializado en interfaces de privacidad GDPR.

[Contexto]
Backend con Edge Functions: request-data-export, request-erasure, request-portability, gdpr-status. Tabla data_subject_requests.

[Objetivo]
Página /cuenta/privacidad con: estado solicitudes, exportar datos, portabilidad, derecho al olvido, toggle consentimientos.

[Requerimientos]
- Confirmación secundaria para destructivas
- Indicar plazo legal 30 días

[Aplicar Especificación Global de Eficiencia]

[Formato de Salida]
- page.tsx
- useDataSubjectRights.ts
- gdprService.ts
- Botones específicos