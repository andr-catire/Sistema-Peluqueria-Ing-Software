# Pruebas de consulta y rendimiento — Servicios

## Qué se prueba

### Suite local con H2

La suite local usa el perfil `test` (`backend/src/test/resources/application-test.properties`),
levanta la aplicación completa (Tomcat, Spring, JPA y seguridad) contra una base H2 en memoria en
modo PostgreSQL y no toca Neon.

#### `PruebasServicioConsultaIntegration` (5 pruebas)

1. **Catálogo**: `GET /api/servicios` devuelve solo servicios activos, ordenados por nombre, con
   `precio` e `imagenUrl` cuando corresponde.
2. **Detalle**: `GET /api/servicios/{id}` devuelve nombre, descripción, precio y duración.
3. **No disponible**: un servicio inactivo o inexistente responde `404` con `NOT_FOUND`.
4. **Creación**: `POST /api/servicios` como `ADMIN` recibe `precio`; el valor se guarda en la
   columna `costo`.
5. **Imagen pública**: `GET /api/servicios/{id}/imagen` devuelve los bytes con su `Content-Type`;
   sin imagen responde `404 NOT_FOUND`.

#### `PruebasServicioRendimientoIntegration` (1 prueba)

1. Levanta Tomcat en un puerto aleatorio.
2. Siembra 1000 servicios activos.
3. Calienta 3 peticiones al listado y 3 al detalle.
4. Mide 30 consultas HTTP al listado y 30 al detalle.
5. Exige un máximo menor a **2 segundos** por petición y un promedio menor a **1 segundo** para el
   listado, que es el requisito no funcional del Brief.

### Integración funcional contra Neon

Las clases `PruebasCrearServicioNeon` y `PruebasFlujoEstilistaNeon` se ejecutan solo cuando
`RUN_DB_TESTS=true`. Usan la base configurada en `backend/.env` y revierten la transacción al
terminar. Son pruebas funcionales, no de rendimiento: no siembran 1000 servicios ni miden latencia.
La medición de rendimiento contra Neon de una iteración anterior no forma parte de la suite actual.

## Cómo ejecutarlas

Suite local completa (H2), sin tocar Neon:

```powershell
cd backend
.\mvnw.cmd clean verify
```

Solo consulta y rendimiento:

```powershell
cd backend
.\mvnw.cmd "-Dtest=PruebasServicioConsultaIntegration,PruebasServicioRendimientoIntegration" test
```

Integración funcional contra Neon (requiere `DB_URL`, `DB_USER` y `DB_PASSWORD` en `backend/.env`):

```powershell
cd backend
$env:RUN_DB_TESTS="true"
.\mvnw.cmd "-Dtest=PruebasCrearServicioNeon,PruebasFlujoEstilistaNeon" test
Remove-Item Env:RUN_DB_TESTS
```

Las pruebas de Neon se omiten si no se define `RUN_DB_TESTS=true`.

## Resultados medidos (7 de noviembre de 2026)

Entorno: Java 25.0.5, Spring Boot 4.1.2, H2 2.4.240 en memoria, Windows, un solo hilo.

| Consulta | Peticiones | Promedio | Máximo | Peticiones/s |
|---|---:|---:|---:|---:|
| Listado de 1000 servicios | 30 | 67 ms | 118 ms | 14,8 |
| Detalle de un servicio | 30 | 14 ms | 17 ms | 70,2 |

La suite completa termina con **88 pruebas, 0 fallos y 3 omitidas**:

- `PruebasCrearServicioNeon` y `PruebasFlujoEstilistaNeon`, porque requieren `RUN_DB_TESTS=true`.
- `PruebasAplicacionPeluqueria`, deshabilitada porque requiere una base PostgreSQL de pruebas.

## Datos temporales de las pruebas

- **H2**: la base se crea y se destruye con el contexto de prueba (`create-drop`); no deja datos.
- **Neon**: las dos pruebas funcionales son transaccionales y revierten sus cambios; no siembran
  1000 servicios ni dejan datos.

## Alcance y límite de la medición

La suite H2 mide la aplicación (Tomcat, Spring, JPA y base de datos en memoria) en local, de forma
secuencial y con un solo hilo. Para medir concurrencia real, usar una herramienta de carga externa
(por ejemplo k6, JMeter o `autocannon`) apuntando a:

```text
GET http://localhost:3000/api/servicios
GET http://localhost:3000/api/servicios/{id}
```
