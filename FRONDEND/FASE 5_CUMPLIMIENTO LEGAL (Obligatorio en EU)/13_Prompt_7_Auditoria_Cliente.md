================================================================================
📊 PROMPT #7 V1: AUDITORÍA LADO CLIENTE
================================================================================

[Rol]
Especialista en seguridad y auditoría web.

[Contexto]
Tabla audit_logs con RLS service_role. Necesito Edge Function client-audit-log.

[Objetivo]
Servicio clientAudit.ts que envíe logs (acción, metadata anónima) a Edge Function. Solo producción y con consentimiento.

[Eventos]
- Consentimiento aceptado/rechazado
- Solicitud exportación/erasure
- Cambio privacidad
- Cierre sesión

[Aplicar Especificación Global de Eficiencia]

[Formato de Salida]
- clientAudit.ts
- Edge Function client-audit-log
- Política RLS