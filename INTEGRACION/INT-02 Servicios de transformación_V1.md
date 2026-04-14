================================================================================
🔄 PROMPT INT-02: SERVICIOS DE TRANSFORMACIÓN (snake_case ↔ camelCase)
================================================================================

[Rol]
Especialista en integración Supabase + TypeScript + transformación de datos.

[Contexto]
- Backend: Supabase con snake_case (user_id, created_at, price_cents)
- Frontend: Quiere usar camelCase (userId, createdAt, priceCents)

[Objetivo]
Crear utilidades que automaticen la transformación de datos entre ambos formatos.

[Archivos Requeridos]

## 1. src/shared/utils/transformKeys.ts
Funciones:
- toCamelCase(str: string): string
- toSnakeCase(str: string): string
- transformKeysToCamelCase<T>(obj: any): T
- transformKeysToSnakeCase<T>(obj: any): T
- transformArrayToCamelCase<T>(arr: any[]): T[]

## 2. src/shared/utils/formatPrice.ts
Funciones:
- formatPrice(cents: number, currency?: string): string (ej: 1999 → "19,99 €")
- toCents(euros: number): number (ej: 19.99 → 1999)
- validateCents(cents: number): boolean

## 3. src/shared/utils/formatDate.ts
Funciones:
- formatDate(isoString: string): string (ej: "12/04/2026")
- toISOString(date: Date): string

## 4. src/lib/supabase/transformers.ts
Transformadores específicos por tabla:
- transformProfile(dbProfile: ProfileDB): Profile
- transformOrder(dbOrder: OrderDB): Order
- transformProduct(dbProduct: ProductDB): Product
- transformConsentLog(dbLog: ConsentLogDB): ConsentLog

[Requisitos]
- Manejo de null/undefined (no romper)
- Tipado TypeScript estricto
- Compatible con Server Components y Client Components

[Formato de Salida Esperado]
1. Código completo de transformKeys.ts
2. Código completo de formatPrice.ts
3. Código completo de formatDate.ts
4. Código completo de transformers.ts
5. Ejemplo de uso en un service (productService.ts)

[Restricciones]
- No usar librerías externas (solo vanilla TypeScript)
- Incluir JSDoc comments para cada función