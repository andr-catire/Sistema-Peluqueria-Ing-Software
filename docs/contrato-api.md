# Contrato de la API

## Servicios

### Listar servicios disponibles

```http
GET /api/servicios
```

Público. Devuelve solo los servicios con `activo = true`, ordenados por nombre.

**200 OK**

```json
[
  {
    "id": 1,
    "nombre": "Corte",
    "descripcion": "Corte de cabello",
    "precio": 12.5,
    "duracionMinutos": 45,
    "imagenUrl": "/api/servicios/1/imagen"
  }
]
```

### Consultar el detalle de un servicio

```http
GET /api/servicios/{id}
```

Público. Devuelve el servicio si existe y está activo.

**200 OK**

```json
{
  "id": 1,
  "nombre": "Corte",
  "descripcion": "Corte de cabello",
  "precio": 12.5,
  "duracionMinutos": 45,
  "imagenUrl": "/api/servicios/1/imagen"
}
```

**404 Not Found**

```json
{
  "error": "NOT_FOUND",
  "message": "El servicio no existe."
}
```

También responde `404` cuando el servicio existe pero está inactivo.

### Consultar la imagen de un servicio

```http
GET /api/servicios/{id}/imagen
```

Público. Devuelve el contenido binario de la imagen del servicio con su `Content-Type`
(`image/webp`, `image/png` o `image/jpeg`) y caché pública de un día. La respuesta del catálogo
incluye `imagenUrl` solo cuando el servicio tiene imagen; el frontend la resuelve contra la URL base
de la API.

**200 OK**

```http
Content-Type: image/webp
Cache-Control: max-age=86400, public
```

**404 Not Found**

```json
{
  "error": "NOT_FOUND",
  "message": "El servicio no tiene imagen."
}
```

## Salud

```http
GET /api/health
```

**200 OK**

```json
{ "status": "ok" }
```

## Formato de error

Todos los errores de negocio usan el mismo cuerpo:

```json
{ "error": "CODIGO", "message": "Mensaje en español" }
```

## Modelo `Servicio`

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id` | entero | — | Identificador generado por la base de datos |
| `nombre` | texto (120) | sí | Nombre del servicio |
| `descripcion` | texto (500) | no | Detalle del servicio |
| `precio` | decimal (10,2) | sí | Costo informativo; se paga en el local |
| `duracionMinutos` | entero | sí | Duración del servicio en minutos |
| `activo` | booleano | sí | Solo los activos se publican en la consulta |
| `imagenUrl` | texto | no | URL relativa de la imagen; `null` si el servicio no tiene imagen |

> La API mantiene `precio` como nombre del campo; en la base de datos la columna se llama `costo`.
> La moneda está pendiente de decisión del equipo. La interfaz formatea el precio con
> `Intl.NumberFormat` y solo muestra el símbolo cuando se configura `VITE_MONEDA`.
