================================================================================
🧾 PROMPT #17 V1: SISTEMA DE FACTURACIÓN LEGAL (ESPAÑA)
================================================================================

[Rol]
Arquitecto backend especializado en facturación electrónica y cumplimiento fiscal en España (IVA, numeración legal, AEAT).

[Contexto]
E-commerce con Supabase + Stripe. Órdenes ya existen. Falta generar facturas legales tras pago.

[Objetivo]
Crear sistema completo de facturación conforme a normativa española:
- Generación de facturas
- Numeración correlativa
- PDF descargable
- Persistencia legal

---

## ⚖️ Requisitos legales (España)

- Numeración correlativa (sin saltos)
- Fecha de emisión
- Datos del emisor (razón social, CIF)
- Datos del cliente
- Desglose de IVA
- Total final
- Conservación: mínimo 4 años

---

## 🗄️ Base de Datos

### Tabla: invoices

- id (UUID)
- order_id (UUID, unique)
- invoice_number (TEXT, unique) ← "2026-000001"
- issue_date (DATE)
- customer_name (TEXT)
- customer_nif (TEXT, nullable)
- customer_address (TEXT)
- subtotal_cents (INTEGER)
- tax_cents (INTEGER)
- total_cents (INTEGER)
- currency (TEXT, default 'EUR')
- pdf_url (TEXT)
- created_at (TIMESTAMPTZ)

---

## 🔢 Numeración correlativa (CRÍTICO)

Formato:
- YYYY-XXXXXX (ej: 2026-000123)

Crear función SQL:
- get_next_invoice_number()
- Bloqueo con SELECT FOR UPDATE
- Evitar duplicados en concurrencia

---

## ⚙️ Edge Function: generate-invoice

Trigger:
- payment_intent.succeeded

Flujo:
1. Buscar orden
2. Generar número factura
3. Calcular IVA
4. Generar PDF
5. Subir a Supabase Storage
6. Guardar en invoices
7. Log en audit_logs

---

## 📄 Generación PDF

Usar:
- pdf-lib o similar

Contenido:
- Logo
- Datos empresa
- Datos cliente
- Tabla productos
- IVA desglosado
- Total

---

## 🔐 RLS

- Usuario: puede ver sus facturas
- Admin: acceso total
- INSERT: solo service role

---

## 📤 Endpoint: get-user-invoices

Devuelve:
- Lista de facturas del usuario
- URL descarga
