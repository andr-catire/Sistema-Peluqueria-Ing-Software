# API de servicios

## POST /api/servicios
Requiere rol `ADMIN`.

```json
{"nombre":"Corte","costo":25.50,"duracionMinutos":45,"descripcion":"Opcional"}
```

- `201`: creado o reactivado (mismo `id` si estaba inactivo).
- `400`: datos inválidos (`VALIDATION_ERROR`).
- `401`: sin sesión.
- `403`: sin rol `ADMIN`.
- `409`: nombre repetido (`DUPLICATE`).

## GET /api/servicios
Público. Devuelve solo los servicios activos, ordenados por nombre.

- `200`: lista de `ServicioRespuesta`.

## Formato de error
```json
{ "error": "CODIGO", "message": "..." }
```
