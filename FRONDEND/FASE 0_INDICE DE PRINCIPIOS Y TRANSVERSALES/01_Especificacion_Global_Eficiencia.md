================================================================================
⚙️ ESPECIFICACIÓN GLOBAL DE EFICIENCIA (APLICABLE A TODOS LOS PROMPTS)
================================================================================

## Principios de Eficiencia Obligatorios

1. **Colocación:** Archivos cerca de donde se usan.
2. **Barrel Exports Controlados:** Solo `index.ts` en raíz de feature.
3. **Lazy Loading por Defecto:** `next/dynamic` para features grandes.
4. **Zero Dependencias Innecesarias:** Justificar cada librería.
5. **TypeScript Estricto:** Solo `.ts` / `.tsx`.
6. **Tailwind CSS:** Sin archivos CSS separados.
7. **Server Components por Defecto:** Marcar `'use client'` solo donde necesario.
8. **Carga Progresiva de Imágenes:** `next/image` con prioridad correcta.

## Formato de Salida Esperado

1. Árbol de Carpetas Completo
2. Decisiones de Eficiencia
3. Código de archivos solicitados