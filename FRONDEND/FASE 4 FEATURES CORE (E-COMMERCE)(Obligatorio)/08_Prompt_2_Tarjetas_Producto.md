================================================================================
🃏 PROMPT #2 V2: TARJETAS DE PRODUCTO INTERACTIVAS
================================================================================

[Rol]
Desarrollador Frontend React/Next.js + TypeScript + Tailwind + Framer Motion.

[Contexto]
Backend Supabase con endpoint GET /api/productos.

[Objetivo]
Crear ProductGrid y ProductCard con: grid responsive, navegación a /productos/[slug], accesibilidad, lazy loading imágenes, skeleton, Suspense.

[Recursos Visuales]
- Imágenes: Unsplash/Picsum (dev), /public/images (prod)
- next.config.js con remotePatterns
- Animaciones: Framer Motion (hover scale, fade-in-up stagger)

[Aplicar Especificación Global de Eficiencia]

[Estructura]
features/productos/
├── components/ (ProductCard, ProductGrid, Skeleton)
├── hooks/ (useProducts)
├── services/ (productService)
└── types/ (product.types)

[Formato de Salida]
1. Árbol de carpetas
2. Código completo de todos los archivos
3. Fragmento next.config.js