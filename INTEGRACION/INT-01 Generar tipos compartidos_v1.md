================================================================================
🔧 PROMPT INT-01: GENERAR TIPOS COMPARTIDOS (Backend → Frontend)
================================================================================

[Rol]
Actúa como un DevOps de integración especializado en Supabase + Next.js.

[Objetivo]
Generar las instrucciones y el código para que el frontend tenga los mismos tipos TypeScript que el backend de Supabase.

[Instrucciones]

## 1. Instalación de Supabase CLI
Dame el comando exacto para instalar Supabase CLI en:
- macOS (brew)
- Windows (winget)
- Linux (curl)

## 2. Comando para generar tipos
Dame el comando exacto para generar los tipos desde mi proyecto Supabase:
supabase gen types typescript --project-id [MI_PROYECTO] > src/types/supabase.ts

## 3. Script en package.json
Dame el fragmento para agregar a package.json:
"scripts": {
  "gen:types": "supabase gen types typescript --project-id $(supabase projects list | grep SELECTED | awk '{print $1}') > src/types/supabase.ts"
}

## 4. Estructura del archivo de tipos
Genera el contenido base de `src/types/supabase.ts` con:
- Importación de Database
- Exportación de tipos útiles: Tables, Insert, Update

## 5. Mantenimiento
Explica:
- Cuándo regenerar los tipos (después de cada cambio en la base de datos)
- Cómo automatizar en CI/CD
- Cómo verificar que los tipos están sincronizados

[Formato de Salida Esperado]
1. Comandos para terminal (lista numerada)
2. Script para package.json (código listo para copiar)
3. Código completo de src/types/supabase.ts
4. Explicación paso a paso
5. Ejemplo de uso en un componente

[Restricciones]
- No asumir que el proyecto está inicializado localmente
- Incluir solución si no se tiene Supabase CLI instalado