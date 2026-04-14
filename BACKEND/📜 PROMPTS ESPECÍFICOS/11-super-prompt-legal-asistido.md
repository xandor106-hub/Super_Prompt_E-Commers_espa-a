
# SUPER PROMPT LEGAL V1.0 — Configuración Asistida para E-commerce en España

Actúa como un asistente legal para configurar mi tienda online en España. No generes ningún archivo todavía. Primero hazme las siguientes preguntas UNA POR UNA. Espera mi respuesta antes de hacer la siguiente pregunta.

Cuando tenga todas las respuestas, generarás los siguientes archivos en la raíz de mi proyecto:

1. `aviso-legal.html`
2. `politica-privacidad.html`
3. `politica-cookies.html`
4. `terminos-condiciones.html`
5. `politica-envios.html`
6. `plantilla-brecha-aepd.md`

---

## 📝 Preguntas (15 en total)

Hazme estas preguntas UNA POR UNA. Después de cada respuesta, continúa con la siguiente.

### Aviso Legal (preguntas 1-5)

1. ¿Cuál es tu **nombre completo** (autónomo) o **razón social** (empresa)?
2. ¿Cuál es tu **NIF** o **CIF**?
3. ¿Cuál es tu **domicilio completo** (calle, número, código postal, ciudad)?
4. ¿Cuál es tu **email de contacto**?
5. ¿Cuál es tu **teléfono de contacto**? (opcional, si no quieres ponerlo di "omitir")

### Política de Envíos (preguntas 6-12)

6. ¿Cuál es el **plazo de envío para España peninsular**? (ej: "2-4 días laborables")
7. ¿Cuál es el **coste de envío para España peninsular**? (ej: "4,99€" o "gratis a partir de 50€")
8. ¿Cuál es el **plazo de envío para Islas Baleares**?
9. ¿Cuál es el **coste de envío para Islas Baleares**?
10. ¿Cuál es el **plazo de envío para Islas Canarias**?
11. ¿Cuál es el **coste de envío para Islas Canarias**?
12. ¿Cuál es el **nombre de la empresa de mensajería**? (ej: "Correos Express", "SEUR", "MRW")

### Términos y Condiciones (preguntas 13-14)

13. ¿Cuál es el **plazo de devolución**? (por defecto 14 días naturales, si lo prefieres di "14 días")
14. ¿Cuál es el **email para gestionar devoluciones**? (normalmente el mismo que el de contacto)

### Confirmación final (pregunta 15)

15. ¿Hay algún **dato adicional** que quieras incluir? (ej: "los gastos de devolución corren por cuenta del cliente" o "no hacemos reembolsos de productos personalizados" — si no, di "no")

---

## 🎯 Instrucciones finales

Cuando haya respondido todas las preguntas:

1. Genera los 6 archivos con el contenido completo, usando mis respuestas para rellenar los placeholders.
2. Cada archivo debe ser HTML válido (excepto `plantilla-brecha-aepd.md` que es Markdown).
3. Incluye fecha de "Última actualización" automática con el día actual.
4. Dame los archivos uno detrás de otro, con el nombre del archivo como título, listos para copiar y pegar.

---

## 📁 Estructura de archivos esperada
raiz-del-proyecto/
├── aviso-legal.html
├── politica-privacidad.html
├── politica-cookies.html
├── terminos-condiciones.html
├── politica-envios.html
└── plantilla-brecha-aepd.md

text

---

## ⚠️ Nota legal

Este asistente genera plantillas basadas en leyes españolas (LSSI, RGPD, LOPDGDD, Ley de Consumidores). No sustituye el asesoramiento legal profesional. Para proyectos con alta facturación o datos sensibles, consulta con un abogado