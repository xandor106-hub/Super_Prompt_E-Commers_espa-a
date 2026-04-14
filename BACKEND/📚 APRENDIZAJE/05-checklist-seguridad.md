# Checklist de Seguridad — Antes de Lanzar

## Base de Datos
- [ ] RLS habilitado en TODAS las tablas
- [ ] Políticas RLS probadas con diferentes usuarios
- [ ] Índices creados para consultas frecuentes
- [ ] Backups automáticos habilitados

## Autenticación
- [ ] MFA disponible (recomendado)
- [ ] Rate limiting en login (5 intentos / 15 min)
- [ ] JWT expiry: 15 minutos
- [ ] Refresh token expiry: 7 días

## API y Edge Functions
- [ ] CORS configurado correctamente
- [ ] Rate limiting por usuario/IP
- [ ] Validación de inputs en todas las funciones
- [ ] Webhooks verificados (Stripe)

## Código
- [ ] `npx foxguard . --severity high` → 0 errores
- [ ] `npx foxguard secrets .` → 0 secretos hardcodeados
- [ ] `npm audit` → sin vulnerabilidades críticas

## Legal (España)
- [ ] Aviso Legal publicado (footer)
- [ ] Política de Privacidad publicada (footer)
- [ ] Política de Cookies publicada (footer)
- [ ] Banner de cookies funcionando
- [ ] Consentimientos guardados en base de datos

## Cumplimiento
- [ ] Supabase en región EU (Frankfurt/Dublin)
- [ ] Stripe con 3D Secure activado
- [ ] Analytics sin cookies (Plausible) o con consentimiento
- [ ] DPO: No requerido (confirmado)

## Monitoreo
- [ ] Auditoría de logs habilitada
- [ ] Alertas configuradas (opcional)
- [ ] Plan de respuesta a brechas (72 horas AEPD)