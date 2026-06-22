# Shopify App — Maison OS
**App de acceso programático a la tienda De Leville. Lee este archivo antes de cualquier operación con el Admin API.**

---

## Credenciales de acceso directo

```
STORE:    34jgvc-dr.myshopify.com
TOKEN:    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
ENDPOINT: https://34jgvc-dr.myshopify.com/admin/api/2026-01/graphql.json
HEADER:   X-Shopify-Access-Token: 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
```

**Patrón de llamada con curl:**
```bash
curl -s -X POST "https://34jgvc-dr.myshopify.com/admin/api/2026-01/graphql.json" \
  -H "Content-Type: application/json" \
  -H "X-Shopify-Access-Token: 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" \
  -d '{"query": "TU_QUERY_AQUI"}'
```

---

## App — Datos de la aplicación

| Campo | Valor |
|---|---|
| **Nombre** | Maison OS |
| **Client ID** | `93db42fb28e13c16fd67f3ced7c402cd` |
| **Client Secret** | `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` |
| **App URL** | https://deleville-os.netlify.app |
| **Proyecto local** | `/Users/santiagosalinas/Documents/Shopify-Projects/de-leville-app/` |
| **Config** | `de-leville-app/shopify.app.toml` |
| **Token generator** | `de-leville-app/get-token.mjs` |

---

## Scopes activos

```
read_products
write_products
read_orders
read_customers
read_analytics
```

### Scopes pendientes de agregar

| Scope | Para qué se necesita |
|---|---|
| `write_files` | Subir imágenes al CDN de Shopify Files (emails, assets) |
| `read_notifications` | Leer templates de notificaciones de email |
| `write_notifications` | Actualizar templates de notificaciones de email |

**Cómo agregar scopes:** Shopify Admin → Settings → Apps and sales channels → Develop apps → Maison OS → Configuration → Admin API scopes → agregar los scopes → Save → API credentials → regenerar el token de acceso.

---

## Capacidades actuales (con scopes vigentes)

- Leer y editar productos (título, descripción, variantes, precio)
- Actualizar inventario
- Leer pedidos y clientes
- Correr queries de analítica

## Capacidades pendientes (requieren agregar scopes arriba)

- Subir imágenes al CDN (`write_files`)
- Desplegar templates de email de notificaciones (`write_notifications`)

---

## REST API (para recursos no disponibles en GraphQL)

```
BASE: https://34jgvc-dr.myshopify.com/admin/api/2026-01
```

Ejemplo notificaciones (requiere scope `read_notifications`):
```bash
curl -s "https://34jgvc-dr.myshopify.com/admin/api/2026-01/notifications.json" \
  -H "X-Shopify-Access-Token: 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"
```

---

## Notas

- El token `shpat_` es un token de acceso permanente (offline token). No expira por tiempo, pero se invalida si se regenera desde el Admin.
- Toda la documentación de productos y variantes está en `PRODUCTOS-ADMIN-API.md`.
- La guía de despliegue de emails está en `/Users/santiagosalinas/Downloads/Emails/Documentacion para Claude/CLI-Deployment-Guide.md`.
