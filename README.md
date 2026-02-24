# Integraciones Diorlatam

## Contenido

- [1. Login Diorlatina](#1-login-diorlatina)
  - [1.1 Flujo](#11-flujo)
  - [1.2 Endpoint](#12-endpoint)
  - [1.3 Request body](#13-request-body)
  - [1.4 Ejemplo de request](#14-ejemplo-de-request)
  - [1.5 Respuesta](#15-respuesta)
- [2. Material Order Timeline API](#2-material-order-timeline-api)
  - [2.1 Endpoint](#21-endpoint)
  - [2.2 Autenticación](#22-autenticación)
  - [2.3 Query parameters](#23-query-parameters)
  - [2.4 Modo por defecto (timeline)](#24-modo-por-defecto-timeline)
  - [2.5 Modo raw](#25-modo-raw)
  - [2.6 Paginación](#26-paginación)
  - [2.7 Errores](#27-errores)

---

## 1. Login Diorlatina

### 1.1 Flujo

1. Cliente entra a `diorlatina.com`.
2. Cliente visita la sección de descargas y hace click en el icono del DAM.
3. Diorlatina envía una solicitud `POST` con la data del usuario.
4. El DAM devuelve una URL de login única para el usuario.
5. Diorlatina redirige al usuario a esta URL.
6. El usuario entra al DAM.

### 1.2 Endpoint

`POST /api/integrations/diorlatina/v1/login`

### 1.3 Request body

- `name` (`string`): nombre completo del usuario.
- `email` (`string`): email del usuario.
- `redirect` (`string`): destino al iniciar sesión. Opciones: `dam`, `email-catalog`, `material-orders`.
- `countries` (`array<object>`): estructura de acceso del usuario.
  - `id` (`string`)
  - `name` (`string`)
  - `clients` (`array<object>`)
    - `id` (`string`)
    - `name` (`string`)
    - `retailers` (`array<object>`)
      - `id` (`string`)
      - `name` (`string`)
      - `points_of_sale` (`array<object>`)
        - `id` (`string`)
        - `name` (`string`)

### 1.4 Ejemplo de request

```http
POST /api/integrations/diorlatina/v1/login HTTP/1.1
Host: www.diorlatam.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
```

```json
{
  "name": "John Doe",
  "email": "john.doe@test.test",
  "redirect": "dam",
  "countries": [
    {
      "id": "7",
      "name": "Bolivia",
      "clients": [
        {
          "id": "45",
          "name": "Aromas",
          "retailers": [
            {
              "id": "485",
              "name": "Aromas Perfumeria",
              "points_of_sale": [
                {
                  "id": "42",
                  "name": "Aromas Casa Zonia Comercio"
                },
                {
                  "id": "43",
                  "name": "Aromas Perfumeria Glamour"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

### 1.5 Respuesta

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
{
  "url": "https://www.diorlatam.com/login/3?expires=1708630057&signature=194dc4744ba7b8339c65f14907957652397492"
}
```

La URL de inicio de sesión es única por usuario, tiene expiración y no se puede reutilizar.

---

## 2. Material Order Timeline API

### 2.1 Endpoint

`GET /api/models/ticket-material-order-placements`

### 2.2 Autenticación

- Header obligatorio: `Authorization: Bearer <token>`
- Header recomendado: `Accept: application/json`
- Token esperado: mismo token de integración Diorlatina (`DIORLATINA_TOKEN`).

Ejemplo:

```bash
curl -X GET "https://www.diorlatam.com/api/models/ticket-material-order-placements" \
  -H "Authorization: Bearer <token>" \
  -H "Accept: application/json"
```

### 2.3 Query parameters

| Parámetro | Tipo | Descripción |
|---|---|---|
| `view` | `string` | Modo de respuesta. Valores: `timeline` (default), `raw`. |
| `per_page` | `number` | Cantidad por página (1-100). |
| `limit` | `number` | Alias de `per_page` (1-100). |
| `page` | `number` | Número de página. |
| `created_from` | `date` | Fecha inicial de filtro `created_at` (`YYYY-MM-DD`). |
| `created_to` | `date` | Fecha final de filtro `created_at` (`YYYY-MM-DD`). |
| `updated_from` | `date` | Fecha inicial de filtro `updated_at` (`YYYY-MM-DD`). |
| `updated_to` | `date` | Fecha final de filtro `updated_at` (`YYYY-MM-DD`). |

Notas:

- En modo `timeline`, los filtros de fecha aplican sobre `tickets.created_at` y `tickets.updated_at`.
- En modo `raw`, los filtros de fecha aplican sobre `ticket_material_order_placements.created_at` y `ticket_material_order_placements.updated_at`.

### 2.4 Modo por defecto (timeline)

Si no se envía `view`, la API devuelve datos agrupados por ticket.

```json
{
  "data": [
    {
      "id": 3558,
      "title": "CAPTURE CREAMS GUATE",
      "status": "completed",
      "franchise": {
        "id": 10,
        "name": "Capture Totale"
      },
      "sub_category": {
        "id": 88,
        "name": "Cremas Soft/Rich y Night"
      },
      "retailer": {
        "id": 6,
        "code": "GTf62542af23a522",
        "name": "Fetiche"
      },
      "countries": [
        {
          "id": 7,
          "name": "Guatemala"
        }
      ],
      "created_at": "2026-02-06T09:46:10.000000Z",
      "updated_at": "2026-02-24T01:01:32.000000Z",
      "items": [
        {
          "id": 3961,
          "label": "Banner Banner principal SoyFetiche",
          "segment_scope": "onsite",
          "segment_type": "banner",
          "row_kind": "segment",
          "placement": {
            "id": 365,
            "name": "Banner principal SoyFetiche",
            "asset_type": "banner",
            "location": "e-merch"
          },
          "weeks": [
            "2026-02-09"
          ],
          "investment": null,
          "units": null,
          "enabled": true
        },
        {
          "id": 3963,
          "label": "Banner Banner principal SoyFetiche",
          "segment_scope": "onsite",
          "segment_type": "banner",
          "row_kind": "segment",
          "placement": {
            "id": 366,
            "name": "Banner principal SoyFetiche",
            "asset_type": "banner",
            "location": "e-merch"
          },
          "weeks": [
            "2026-02-09"
          ],
          "investment": null,
          "units": null,
          "enabled": true
        }
      ]
    }
  ]
}
```

Reglas:

- Solo se incluyen tickets con `items`.
- `items` contiene filas padre (`row_kind = segment`).
- `weeks` se calcula desde filas hijas (`row_kind = segment_time`).
- Las claves de `meta` se aplanan al mismo nivel del item.
- Si una clave de `meta` colisiona con una clave existente, se devuelve como `meta_<clave>`.

#### 2.4.1 Propiedades de cada ticket dentro de `data[]`

| Propiedad | Tipo | Descripción |
|---|---|---|
| `id` | `number` | ID del ticket. |
| `title` | `string` | Titulo del ticket. |
| `status` | `string` | Estado del ticket. Valores: `waiting-for-confirmation`, `pending`, `in-progress`, `waiting-for-approval`, `approved`, `completed`, `draft`. |
| `franchise` | `object` | Franquicia principal del ticket (si existe). |
| `franchise.id` | `number` | ID de la franquicia. |
| `franchise.name` | `string` | Nombre de la franquicia. |
| `sub_category` | `object` | Sub-franquicia / subcategoria del ticket (si existe). |
| `sub_category.id` | `number` | ID de la subcategoria. |
| `sub_category.name` | `string` | Nombre de la subcategoria. |
| `retailer` | `object` | Retailer canonico del ticket (si existe). |
| `retailer.id` | `number` | ID del retailer. |
| `retailer.code` | `string` | Codigo del retailer. |
| `retailer.name` | `string` | Nombre del retailer. |
| `countries` | `array<object>` | Paises asociados al ticket. |
| `countries[].id` | `number` | ID del pais. |
| `countries[].name` | `string` | Nombre del pais. |
| `created_at` | `datetime` | Fecha de creación del ticket. |
| `updated_at` | `datetime` | Fecha de última actualización del ticket. |

#### 2.4.2 `items[]` (segmentos de timeline)

| Propiedad | Tipo | Descripción |
|---|---|---|
| `id` | `number` | ID de la fila padre en `ticket_material_order_placements`. |
| `label` | `string` | Etiqueta legible (ej. `Banner Hero Slot`, `Paid Socials (Instagram)`). |
| `segment_scope` | `string` | Alcance del segmento. Valores: `onsite`, `offsite`. |
| `segment_type` | `string` | Tipo técnico del segmento. Valores: `banner`, `takeover`, `sponsored_products`, `samples`, `newsletter`, `social_owned`, `social_paid`, `google_ads`, `influencer`. |
| `row_kind` | `string` | Tipo de fila. En este modo siempre `segment`. |
| `placement` | `object` | Placement relacionado al segmento (si existe). |
| `placement.id` | `number` | ID del placement. |
| `placement.name` | `string` | Nombre del placement. |
| `placement.asset_type` | `string` | Tipo de asset. Valores observados: `banner`, `email`, `other`, `priority`. |
| `placement.location` | `string` | Ubicación del placement. Valores: `e-merch`, `e-corner`, `other`. |
| `weeks` | `array<string>` | Semanas activas (`YYYY-MM-DD`) construidas desde filas `segment_time`. |
| `investment` | `number` | Inversión del segmento cuando aplica. |
| `units` | `number` | Unidades del segmento cuando aplica. |
| `enabled` | `boolean` | Indica si el segmento está habilitado. |
| `...meta keys` | `mixed` | Campos de `meta` aplanados (ej. `platform`). |

### 2.5 Modo raw

Para recibir filas normalizadas sin agrupación:

`GET /api/models/ticket-material-order-placements?view=raw`

```bash
curl -X GET "https://www.diorlatam.com/api/models/ticket-material-order-placements?view=raw" \
  -H "Authorization: Bearer <token>" \
  -H "Accept: application/json"
```

Cada elemento de `data` representa una fila de `ticket_material_order_placements`.

| Propiedad | Tipo | Descripción |
|---|---|---|
| `id` | `number` | ID de la fila. |
| `row_kind` | `string` | Tipo de fila. Valores: `placement_link`, `segment`, `segment_time`. |
| `ticket_id` | `number` | ID del ticket. |
| `parent_id` | `number` | ID del segmento padre cuando la fila es `segment_time`. |
| `material_order_placement_id` | `number` | ID del placement vinculado cuando aplica. |
| `priority` | `string` | Prioridad legacy, usada principalmente en filas `placement_link`. |
| `segment_scope` | `string` | Alcance del segmento cuando aplica. Valores: `onsite`, `offsite`. |
| `segment_type` | `string` | Tipo técnico del segmento cuando aplica. Valores: `banner`, `takeover`, `sponsored_products`, `samples`, `newsletter`, `social_owned`, `social_paid`, `google_ads`, `influencer`. |
| `start_date` | `date` | Fecha de inicio del segmento cuando aplica. |
| `end_date` | `date` | Fecha de fin del segmento cuando aplica. |
| `investment` | `number` | Monto de inversión del segmento cuando aplica. |
| `enabled` | `boolean` | Si el segmento está habilitado. |
| `units` | `number` | Unidades configuradas cuando aplica. |
| `week_from` | `string` | Semana inicial (`YYYY-MM-DD`) o referencia semanal. |
| `week_to` | `string` | Semana final (`YYYY-MM-DD`) o referencia semanal. |
| `position` | `number` | Posición/orden del segmento cuando aplica. |
| `segment_uid` | `string` | UID único de segmento dentro del ticket cuando aplica. |
| `migrated_from_extra_details_at` | `datetime` | Timestamp de migración desde `extra_details` cuando aplica. |
| `created_at` | `datetime` | Fecha de creación de la fila. |
| `updated_at` | `datetime` | Fecha de actualización de la fila. |
| `placement` | `object` | Placement relacionado con datos descriptivos (`name`, `asset_type`, `location`, `size`, `device`, etc.) cuando existe relación. `asset_type` puede ser `banner`, `email`, `other`, `priority` y `location` puede ser `e-merch`, `e-corner`, `other`. |
| `...meta keys` | `mixed` | Campos de `meta` aplanados. |

### 2.6 Paginación

Ambos modos (`timeline` y `raw`) son paginados y devuelven:

- `data`: elementos de la página actual.
- `links`: URLs de navegación (`first`, `last`, `prev`, `next`).
- `meta`: metadata (`current_page`, `from`, `last_page`, `path`, `per_page`, `to`, `total`, etc.).

Ejemplo rápido:

```bash
curl -X GET "https://www.diorlatam.com/api/models/ticket-material-order-placements?per_page=25&page=2" \
  -H "Authorization: Bearer <token>" \
  -H "Accept: application/json"
```

### 2.7 Errores

- `401 Unauthorized`: token ausente o inválido (`Invalid token`).
