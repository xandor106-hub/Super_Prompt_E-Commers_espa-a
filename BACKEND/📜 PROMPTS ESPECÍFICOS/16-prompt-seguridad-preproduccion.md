================================================================================
🔒 PROMPT #16 V1: SEGURIDAD PRE-PRODUCCIÓN — HERRAMIENTAS REALES
================================================================================

[Rol]
Ingeniero de Seguridad DevSecOps especializado en aplicaciones Next.js + Supabase. Conocimiento de OWASP Top 10, herramientas de análisis estático y auditorías de dependencias.

[Contexto]
E-commerce en España con stack: Next.js 14, Supabase, Stripe, Vercel. Versiones anteriores de este sistema referenciaban "Foxguard" como herramienta de seguridad. Foxguard no es una herramienta estándar verificada en el ecosistema npm. Este prompt la sustituye por herramientas reales, mantenidas y documentadas.

[Objetivo]
Crear un sistema completo de seguridad pre-producción con herramientas verificadas, un checklist accionable y scripts de automatización listos para copiar.

---

## Herramientas reales que usaremos

| Herramienta | Propósito | Coste |
|-------------|-----------|-------|
| npm audit | Vulnerabilidades en dependencias | Gratis (incluido en npm) |
| Snyk | Análisis profundo de dependencias + IaC | Gratis hasta 200 scans/mes |
| Semgrep | Análisis estático de código (SAST) | Gratis para open source |
| truffleHog | Detección de secrets en git history | Gratis |
| OWASP ZAP | Análisis dinámico (DAST) de la web | Gratis |

---

## Script maestro de seguridad

Crear archivo: `scripts/security-check.sh`

```bash
#!/bin/bash
# security-check.sh — Auditoría de seguridad pre-producción
# Ejecutar antes de cada release: bash scripts/security-check.sh

set -e
PASS=0
FAIL=0
WARN=0

echo "🔒 AUDITORÍA DE SEGURIDAD — $(date '+%Y-%m-%d %H:%M')"
echo "=================================================="

# 1. Vulnerabilidades en dependencias (npm audit)
echo ""
echo "📦 [1/5] Dependencias — npm audit"
if npm audit --audit-level=high --json > /tmp/npm-audit.json 2>&1; then
  echo "  ✅ Sin vulnerabilidades altas/críticas"
  PASS=$((PASS+1))
else
  VULN_COUNT=$(cat /tmp/npm-audit.json | node -e "const d=JSON.parse(require('fs').readFileSync('/dev/stdin')); console.log(d.metadata?.vulnerabilities?.high + d.metadata?.vulnerabilities?.critical || 0)")
  echo "  ❌ FALLO: $VULN_COUNT vulnerabilidades high/critical encontradas"
  echo "     Ejecuta: npm audit fix"
  FAIL=$((FAIL+1))
fi

# 2. Secrets hardcodeados en el código (truffleHog)
echo ""
echo "🔑 [2/5] Secrets — truffleHog"
if command -v trufflehog &> /dev/null; then
  if trufflehog filesystem . --only-verified --json 2>/dev/null | grep -q '"verified":true'; then
    echo "  ❌ FALLO: Secrets verificados encontrados en el código"
    FAIL=$((FAIL+1))
  else
    echo "  ✅ Sin secrets hardcodeados detectados"
    PASS=$((PASS+1))
  fi
else
  echo "  ⚠️  truffleHog no instalado. Instalar: brew install trufflesecurity/trufflehog/trufflehog"
  WARN=$((WARN+1))
fi

# 3. Análisis estático de código (Semgrep)
echo ""
echo "🔍 [3/5] Análisis estático — Semgrep"
if command -v semgrep &> /dev/null; then
  if semgrep --config=p/typescript --config=p/react --config=p/owasp-top-ten \
     --error --quiet src/ 2>/dev/null; then
    echo "  ✅ Sin problemas de seguridad detectados"
    PASS=$((PASS+1))
  else
    echo "  ❌ FALLO: Problemas de seguridad detectados. Ver output de Semgrep."
    FAIL=$((FAIL+1))
  fi
else
  echo "  ⚠️  Semgrep no instalado. Instalar: pip install semgrep"
  WARN=$((WARN+1))
fi

# 4. Variables de entorno — verificar que no hay .env en git
echo ""
echo "🌍 [4/5] Variables de entorno — revisión .gitignore"
if git check-ignore -q .env.local 2>/dev/null && \
   git check-ignore -q .env.production 2>/dev/null; then
  echo "  ✅ Archivos .env excluidos correctamente de git"
  PASS=$((PASS+1))
else
  echo "  ❌ FALLO: .env.local o .env.production NO están en .gitignore"
  FAIL=$((FAIL+1))
fi

# Verificar que no hay .env commiteado accidentalmente
if git log --all --full-history -- ".env" ".env.local" ".env.production" | grep -q "commit"; then
  echo "  ❌ CRÍTICO: Se detectó un archivo .env en el historial de git"
  echo "     Usar: git filter-repo o BFG Repo Cleaner para limpiar el historial"
  FAIL=$((FAIL+1))
fi

# 5. Headers de seguridad HTTP — verificar next.config.js
echo ""
echo "🛡️  [5/5] Headers de seguridad — next.config.js"
REQUIRED_HEADERS=("X-Frame-Options" "X-Content-Type-Options" "Referrer-Policy" "Content-Security-Policy")
MISSING=0
for header in "${REQUIRED_HEADERS[@]}"; do
  if ! grep -q "$header" next.config.js 2>/dev/null; then
    echo "  ⚠️  Header faltante: $header"
    MISSING=$((MISSING+1))
    WARN=$((WARN+1))
  fi
done
if [ $MISSING -eq 0 ]; then
  echo "  ✅ Todos los headers de seguridad presentes"
  PASS=$((PASS+1))
fi

# Resumen final
echo ""
echo "=================================================="
echo "RESUMEN: ✅ $PASS correctos | ❌ $FAIL fallos | ⚠️  $WARN advertencias"
echo ""

if [ $FAIL -gt 0 ]; then
  echo "🚫 NO desplegar a producción. Corregir $FAIL fallos primero."
  exit 1
else
  echo "✅ Auditoría superada. Proceder con precaución si hay advertencias."
  exit 0
fi
```

---

## Headers de seguridad para next.config.js

```javascript
// next.config.js
const securityHeaders = [
  { key: 'X-DNS-Prefetch-Control', value: 'on' },
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
  { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
  {
    key: 'Content-Security-Policy',
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-eval' 'unsafe-inline' https://js.stripe.com",
      "frame-src https://js.stripe.com https://hooks.stripe.com",
      "connect-src 'self' https://*.supabase.co wss://*.supabase.co https://api.stripe.com",
      "img-src 'self' data: blob: https:",
      "style-src 'self' 'unsafe-inline'",
      "font-src 'self'",
    ].join('; ')
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  }
];

module.exports = {
  async headers() {
    return [{ source: '/(.*)', headers: securityHeaders }];
  }
};
```

⚠️ Ajustar el CSP según los dominios reales de tus servicios. Demasiado restrictivo romperá Stripe.

---

## Checklist de seguridad pre-lanzamiento

### Supabase
- [ ] RLS habilitado en TODAS las tablas (verificar con: SELECT tablename FROM pg_tables WHERE schemaname = 'public' AND rowsecurity = false)
- [ ] Service role key NUNCA expuesta en frontend (verificar que no hay NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY)
- [ ] Supabase project en región EU (Frankfurt o Dublin)
- [ ] Point-in-time recovery activado (plan Pro)
- [ ] Revisar que audit_logs no tiene política SELECT para usuarios normales

### Stripe
- [ ] Usando claves de producción (sk_live_, pk_live_), no de test
- [ ] Webhook secret configurado en producción (diferente al de desarrollo)
- [ ] Stripe Radar activado
- [ ] 3D Secure configurado para transacciones > €30
- [ ] Idempotency implementada en webhook handler

### Next.js / Vercel
- [ ] Variables de entorno configuradas en Vercel (no en .env subido a git)
- [ ] Headers de seguridad HTTP configurados (ver arriba)
- [ ] No hay console.log con datos de usuario en producción
- [ ] next.config.js no expone información sensible en el bundle

### GDPR
- [ ] Banner de cookies funcional y registrando en consent_logs
- [ ] Política de privacidad actualizada y accesible
- [ ] Derecho al olvido implementado y probado
- [ ] DPA firmado con: Supabase, Stripe, Resend/Postmark, analytics provider

---

## Configuración de GitHub Actions (CI)

```yaml
# .github/workflows/security.yml
name: Security audit

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # truffleHog necesita historial completo

      - name: npm audit
        run: npm audit --audit-level=high

      - name: Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/typescript p/react p/owasp-top-ten

      - name: truffleHog — secrets scan
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          only-verified: true
```

---

## Output requerido

Por favor, genera:
1. `scripts/security-check.sh` completo y ejecutable
2. `next.config.js` con headers de seguridad (compatible con Stripe y Supabase)
3. `.github/workflows/security.yml` para CI automático
4. Instrucciones de instalación de cada herramienta (macOS + Linux)
5. Guía para interpretar y resolver los problemas más comunes que encontrará cada herramienta
6. Query SQL para verificar que RLS está activo en todas las tablas de Supabase

---

## Restricciones

- Todas las herramientas deben tener versión gratuita funcional
- El script debe ejecutarse sin configuración previa en un proyecto limpio
- Los fallos deben bloquear el deploy (exit code 1) — las advertencias no
- Compatible con Node.js 18+ y npm 9+
- Sin dependencias de Docker para el script básico

[Aplicar Especificación Global de Eficiencia]
