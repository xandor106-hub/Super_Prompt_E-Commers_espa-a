# 📚 ÍNDICE MAESTRO — PROMPTS ESPECÍFICOS

## 🎯 Orden de ejecución recomendado

| Orden | # | Prompt | Dependencias | Estado |
|-------|---|--------|--------------|--------|
| 1 | 02 | Cookies + Aviso Legal + Privacidad | Ninguna | ⬜ |--done
| 2 | 03 | Versión personal (España) | 02 | ⬜ |---done
| 3 | 11 | Asistente legal (rellenar datos) | 02, 03 | ⬜ |---falta
| 4 | 12 | Tabla products + inventario | Ninguna | ⬜ |--done
| 5 | 13 | Stripe webhook + idempotency | 12 | ⬜ |---not
| 6 | 14 | Emails transaccionales (Resend) | 12, 13 | ⬜ |---done
| 7 | 15 | Carrito persistente (Zustand) | 12 | ⬜ |
| 8 | 16 | Seguridad pre-producción | Ninguna | ⬜ |
| 9 | 17 | Facturación legal | 12, 13 | ⬜ |
| 10 | 18 | Aceptación de términos | 02 | ⬜ |
| 11 | 19 | Rate limiting + anti-abuso | Ninguna | ⬜ |
| 12 | 20 | Observabilidad (Sentry + logs) | Ninguna | ⬜ |
| 13 | 21 | Devoluciones + reembolsos | 12, 13, 14 | ⬜ |
| 14 | 22 | Tests de integración | 12, 13, 15, 21 | ⬜ |
| 15 | 23 | Job retención datos (RGPD) | 12, 14, 20 | ⬜ |

---

## 📋 Dependencias visuales
