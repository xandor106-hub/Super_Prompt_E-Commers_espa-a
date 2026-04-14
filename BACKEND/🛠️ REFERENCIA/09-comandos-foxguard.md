# Comandos Útiles de Foxguard

## Instalación
```bash
npm install -g foxguard
# o
npx foxguard [comando]
Comandos básicos
bash
# Escanear todo el proyecto
npx foxguard .

# Escanear solo archivos modificados
npx foxguard --changed .

# Escanear solo vulnerabilidades altas/críticas
npx foxguard --severity high .

# Buscar secretos hardcodeados
npx foxguard secrets .

# Escanear un archivo específico
npx foxguard archivo.js

# Instalar pre-commit hook (escanea antes de cada commit)
npx foxguard init

# Formato JSON para integración con CI/CD
npx foxguard . --format json
Integración con VS Code
Instalar extensión foxguard desde el marketplace

Escanea automáticamente al guardar (Ctrl+S)

Muestra subrayados en el código

Salida típica
text
src/auth/login.js
  14:5  CRITICAL  js/no-sql-injection
        SQL query built with template literal interpolation

WARNING 1 issue in 1 file: 1 critical