# API de asistencia
Solo rol `PERSONAL`. El empleado sale de la sesión; la hora la decide el servidor.

## GET /api/asistencia/hoy
Estado del día: fecha, apertura, cierre, entrada, salida, retraso y flags
`puedeRegistrarEntrada`, `puedeRegistrarSalida` y `salidaHabilitadaDesde`.

- `200`: estado actual. `401`: sin sesión. `403`: no es `PERSONAL`.

## POST /api/asistencia/entrada
Sin cuerpo. Registra la entrada una vez al día y calcula el retraso.

- `201`: estado actualizado.
- `409`: `ENTRADA_YA_REGISTRADA` o `ENTRADA_FUERA_DE_HORARIO`.

## POST /api/asistencia/salida
Cuerpo opcional; el motivo solo es obligatorio antes del cierre.

```json
{ "motivo": "Cita médica" }
```

- `200`: estado actualizado.
- `400`: `MOTIVO_REQUERIDO` o `VALIDATION_ERROR` (motivo de más de 200 caracteres).
- `409`: `SIN_ENTRADA` o `SALIDA_YA_REGISTRADA`.

## Errores
```json
{ "error": "CODIGO", "message": "..." }
```
