# Shopify Admin API — Editar productos De Leville

Acceso directo al Admin API de la tienda. Usa esto para leer y modificar productos, variantes, precios, descripciones e inventario via GraphQL.

## Credenciales

```
STORE: 34jgvc-dr.myshopify.com
TOKEN: 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
ENDPOINT: https://34jgvc-dr.myshopify.com/admin/api/2026-01/graphql.json
HEADER: X-Shopify-Access-Token: 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
```

---

## Cómo hacer una llamada (patrón base)

```bash
curl -s -X POST "https://34jgvc-dr.myshopify.com/admin/api/2026-01/graphql.json" \
  -H "Content-Type: application/json" \
  -H "X-Shopify-Access-Token: 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" \
  -d '{"query": "TU_QUERY_AQUI"}'
```

---

## Queries de lectura

### Listar todos los productos (id, título, descripción, variantes, precio)
```graphql
{
  products(first: 50) {
    nodes {
      id
      title
      descriptionHtml
      variants(first: 10) {
        nodes {
          id
          title
          price
          inventoryQuantity
        }
      }
    }
  }
}
```

### Buscar producto por título
```graphql
{
  products(first: 5, query: "title:Classico") {
    nodes {
      id
      title
      descriptionHtml
    }
  }
}
```

### Ver un producto por ID
```graphql
{
  product(id: "gid://shopify/Product/ID_AQUI") {
    id
    title
    descriptionHtml
    variants(first: 10) {
      nodes { id title price inventoryQuantity }
    }
  }
}
```

---

## Mutations de escritura

### Actualizar título y descripción
```graphql
mutation {
  productUpdate(input: {
    id: "gid://shopify/Product/ID_AQUI"
    title: "Nuevo título"
    descriptionHtml: "<p>Nueva descripción HTML</p>"
  }) {
    product { id title }
    userErrors { field message }
  }
}
```

### Actualizar precio de una variante
```graphql
mutation {
  productVariantUpdate(input: {
    id: "gid://shopify/ProductVariant/ID_VARIANTE"
    price: "1250.00"
  }) {
    productVariant { id price }
    userErrors { field message }
  }
}
```

### Actualizar múltiples campos a la vez
```graphql
mutation {
  productUpdate(input: {
    id: "gid://shopify/Product/ID_AQUI"
    title: "Título"
    descriptionHtml: "<p>Descripción</p>"
    tags: ["tag1", "tag2"]
    metafields: [{
      namespace: "custom"
      key: "clave"
      value: "valor"
      type: "single_line_text_field"
    }]
  }) {
    product { id title }
    userErrors { field message }
  }
}
```

---

## Pasar variables en una mutation (más limpio para Claude)

```bash
curl -s -X POST "https://34jgvc-dr.myshopify.com/admin/api/2026-01/graphql.json" \
  -H "Content-Type: application/json" \
  -H "X-Shopify-Access-Token: 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" \
  -d '{
    "query": "mutation productUpdate($input: ProductInput!) { productUpdate(input: $input) { product { id title } userErrors { field message } } }",
    "variables": {
      "input": {
        "id": "gid://shopify/Product/ID_AQUI",
        "descriptionHtml": "<p>Descripción nueva</p>"
      }
    }
  }'
```

---

## IDs de productos actuales

Obtén la lista actualizada con:
```bash
curl -s -X POST "https://34jgvc-dr.myshopify.com/admin/api/2026-01/graphql.json" \
  -H "Content-Type: application/json" \
  -H "X-Shopify-Access-Token: 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" \
  -d '{"query": "{ products(first: 50) { nodes { id title } } }"}' | python3 -m json.tool
```

---

## Si el token expira

El token es offline (sin expiración) pero si se revoca, recupéralo con:

```bash
node /tmp/get-shopify-token.js
```

Si el archivo no existe, encuéntralo en este proyecto en la sesión donde fue creado, o recrea la app desde:
- Partners Dashboard: `https://dev.shopify.com/dashboard/155237859/apps/338533285889`
- App TOML: `/tmp/de-leville-app/shopify.app.toml`
- Client ID: `93db42fb28e13c16fd67f3ced7c402cd`
- Client Secret: `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`
