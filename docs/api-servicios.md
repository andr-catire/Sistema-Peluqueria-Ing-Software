# API de servicios

## POST /api/servicios
Requiere rol `ADMIN`.

```json
{"nombre":"Corte","precio":25.50,"duracionMinutos":45,"descripcion":"Opcional"}
```

- `201`: creado o reactivado (mismo `id` si estaba inactivo).
- `400`: datos inválidos (`VALIDATION_ERROR`).
- `401`: sin sesión.
- `403`: sin rol `ADMIN`.
- `409`: nombre repetido (`DUPLICATE`).

## GET /api/servicios
Público. Devuelve solo los servicios activos, ordenados por nombre.

- `200`: lista de `ServicioRespuesta`.

## GET /api/servicios/{id}
Público. Devuelve un servicio activo.

- `200`: `ServicioRespuesta`.
- `404`: el servicio no existe o está inactivo (`NOT_FOUND`).

## Formato de error
```json
{ "error": "CODIGO", "message": "..." }
```
