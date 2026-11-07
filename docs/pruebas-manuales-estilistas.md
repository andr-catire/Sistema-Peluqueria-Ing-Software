# Probar el flujo de estilistas sin frontend

Esta guía permite probar el registro de un cliente, la creación de un servicio, la búsqueda por
correo, la promoción a estilista y la desvinculación usando `curl`, sin implementar el login ni el
frontend.

## Requisitos

- Backend detenido si ya estaba corriendo.
- `backend/.env` con `APP_ADMIN_EMAIL` y `APP_ADMIN_PASSWORD`.
- La columna obsoleta `servicios.precio` eliminada de Neon. Si todavía existe, `POST /api/servicios`
  falla con `null value in column "precio"`.
- Opcional: `jq` para extraer ids de las respuestas.

## 1. Arrancar el backend con HTTP Basic de prueba

```bash
cd backend
APP_BASIC_AUTH=true ./mvnw spring-boot:run
```

`APP_BASIC_AUTH=true` activa HTTP Basic **solo para esta ejecución**. Sin esa variable, Basic queda
desactivado y las rutas protegidas siguen respondiendo `401` hasta que exista el login con JWT.
No uses esta variable en producción.

## 2. Cargar las credenciales del administrador

En otra terminal, desde la raíz del repositorio:

```bash
ADMIN_EMAIL="$(sed -n 's/^APP_ADMIN_EMAIL=//p' backend/.env)"
ADMIN_PASSWORD="$(sed -n 's/^APP_ADMIN_PASSWORD=//p' backend/.env)"
BASE_URL="http://localhost:3000"
```

No imprimas esas variables. El administrador se crea al arrancar la aplicación solo si todavía no
existe ningún usuario con rol `ADMIN`. Si ya existía con otra contraseña, usa las credenciales con
las que se creó.

Comprueba que Basic funciona:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' \
  -u "$ADMIN_EMAIL:$ADMIN_PASSWORD" \
  "$BASE_URL/api/personal/buscar?email=prueba@correo.com"
```

- `200` o `404`: Basic autenticó correctamente.
- `401`: las credenciales no coinciden con el administrador existente.

## 3. Crear un servicio (solo ADMIN)

```bash
NOMBRE_SERVICIO="Corte prueba $(date +%s)"

RESPUESTA_SERVICIO="$(curl -sS -u "$ADMIN_EMAIL:$ADMIN_PASSWORD" \
  -H 'Content-Type: application/json' \
  -d "{\"nombre\":\"$NOMBRE_SERVICIO\",\"costo\":25.50,\"duracionMinutos\":45,\"descripcion\":\"Servicio de prueba\"}" \
  "$BASE_URL/api/servicios")"

printf '%s\n' "$RESPUESTA_SERVICIO"
```

Respuesta esperada: `201 Created` con el `id` del servicio.

Si tienes `jq`, extrae el id de esa misma respuesta:

```bash
SERVICIO_ID="$(printf '%s' "$RESPUESTA_SERVICIO" | jq -r '.id')"
echo "Servicio: $SERVICIO_ID"
```

No repitas el mismo nombre: un servicio activo duplicado responde `409 DUPLICATE`.

## 4. Registrar un cliente (público, sin credenciales)

```bash
SUFIJO="$(date +%s)"
EMAIL="ana.prueba.$SUFIJO@correo.com"
CEDULA="V-$SUFIJO"

RESPUESTA_CLIENTE="$(curl -sS -H 'Content-Type: application/json' \
  -d "{\"nombre\":\"Ana Prueba\",\"email\":\"$EMAIL\",\"password\":\"secreta123\",\"cedula\":\"$CEDULA\",\"telefono\":\"0414-1234567\"}" \
  "$BASE_URL/api/auth/registro")"

printf '%s\n' "$RESPUESTA_CLIENTE"
```

Respuesta esperada: `201 Created` con `rol: "CLIENTE"`.

Con `jq`, extrae el id de esa misma respuesta:

```bash
CLIENTE_ID="$(printf '%s' "$RESPUESTA_CLIENTE" | jq -r '.id')"
echo "Cliente: $CLIENTE_ID"
```

## 5. Buscar la cuenta por correo (solo ADMIN)

```bash
curl -sS -u "$ADMIN_EMAIL:$ADMIN_PASSWORD" \
  "$BASE_URL/api/personal/buscar?email=$EMAIL"
```

Respuesta esperada: `200 OK` con `id`, `nombre`, `email` y `rol: "CLIENTE"`.

Si buscas una cuenta `PERSONAL` o `ADMIN`, responde `409 NOT_PROMOTABLE`: la búsqueda está pensada
solo para cuentas que se pueden promover.

## 6. Promover la cuenta a estilista (solo ADMIN)

```bash
curl -sS -u "$ADMIN_EMAIL:$ADMIN_PASSWORD" \
  -H 'Content-Type: application/json' \
  -d "{\"idUsuario\":$CLIENTE_ID,\"serviciosIds\":[$SERVICIO_ID]}" \
  "$BASE_URL/api/personal/promover"
```

Respuesta esperada: `201 Created` con `rol: "PERSONAL"` y la lista de servicios asignados.

En este paso, la ficha de `clientes` se elimina y se crea la de `personal`: la cuenta queda con un
solo perfil. Si la cuenta tuviera citas registradas como cliente, la operación responde
`409 CLIENT_WITH_APPOINTMENTS`.

## 7. Verificar el perfil del estilista (solo PERSONAL)

Usa las credenciales del cliente que acabas de promover:

```bash
curl -sS -u "$EMAIL:secreta123" \
  "$BASE_URL/api/personal/mi-perfil"
```

Respuesta esperada: `200 OK` con el perfil y sus servicios. El id sale del usuario autenticado, no
de la URL.

También puedes comprobar que ya no se puede volver a promover:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' \
  -u "$ADMIN_EMAIL:$ADMIN_PASSWORD" \
  "$BASE_URL/api/personal/buscar?email=$EMAIL"
```

Debe responder `409` porque el rol ya es `PERSONAL`.

## 8. Verificar en Neon que solo queda un perfil

En el editor SQL de Neon:

```sql
SELECT u.id, u.email, u.rol,
       c.id AS cliente_id,
       p.id AS personal_id
FROM usuarios u
LEFT JOIN clientes c ON c.id = u.id
LEFT JOIN personal p ON p.id = u.id
WHERE u.email = 'ana.prueba...@correo.com';
```

Después de promover, `cliente_id` debe ser `NULL` y `personal_id` debe tener valor.

## 9. Desvincular al estilista (solo ADMIN)

```bash
curl -sS -i -X DELETE \
  -u "$ADMIN_EMAIL:$ADMIN_PASSWORD" \
  "$BASE_URL/api/personal/$CLIENTE_ID/desvincular"
```

Respuesta esperada: `204 No Content`.

Después:

- `GET /api/personal/mi-perfil` con las credenciales de esa cuenta debe responder `403`, porque el
  rol volvió a `CLIENTE`.
- La fila `personal` se conserva para no romper citas y asistencias históricas.
- Si el estilista tiene citas futuras en estado `PENDIENTE` o `CONFIRMADA`, la desvinculación
  responde `409 PENDING_APPOINTMENTS`.

## Errores que puedes provocar para comprobar las reglas

| Caso | Respuesta esperada |
|---|---|
| Registrar dos veces el mismo correo | `409 EMAIL_TAKEN` |
| Registrar dos veces la misma cédula | `409 CEDULA_TAKEN` |
| Promover una cuenta `PERSONAL` | `409 ALREADY_EMPLOYEE` |
| Promover una cuenta `ADMIN` | `409 INVALID_ROLE` |
| Asignar un servicio inactivo | `409 INACTIVE_SERVICE` |
| Promover un cliente con citas | `409 CLIENT_WITH_APPOINTMENTS` |
| Desvincular con citas futuras activas | `409 PENDING_APPOINTMENTS` |
| Llamar una ruta de admin sin credenciales | `401 UNAUTHORIZED` |
| Llamar una ruta de admin con un cliente | `403 FORBIDDEN` |

## Limpieza

Los datos de prueba quedan en Neon. Puedes eliminarlos desde el editor SQL:

```sql
DELETE FROM personal_servicios WHERE personal_id = <CLIENTE_ID>;
DELETE FROM personal WHERE id = <CLIENTE_ID>;
DELETE FROM clientes WHERE id = <CLIENTE_ID>;
DELETE FROM usuarios WHERE id = <CLIENTE_ID>;
DELETE FROM servicios WHERE id = <SERVICIO_ID>;
```

Si la cuenta tiene citas o asistencias, no borres la fila `personal` sin revisar esas referencias.
