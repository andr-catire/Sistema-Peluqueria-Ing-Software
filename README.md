# Sistema para Peluquería

Sistema web para gestionar una peluquería: catálogo de servicios, personal, clientes y agenda de citas.
Proyecto académico de la UCAB (Ingeniería de Software, Equipo 57-6) con proyección comercial.

## Stack

| Capa | Tecnología |
|---|---|
| Frontend | Vue 3 (Composition API), Vue Router, Pinia, Axios y Vite |
| Backend | Java 21, Spring Boot y Maven (`./mvnw`) |
| Persistencia | Spring Data JPA (Hibernate) |
| Migraciones | Flyway |
| Seguridad | Spring Security + BCrypt (JWT pendiente) |
| Base de datos | PostgreSQL alojado en Neon |

## Estructura

```text
peluqueria/
├─ backend/                 API Spring Boot, reglas de negocio y pruebas
│  └─ src/main/resources/
│     └─ db/migration/      Migraciones Flyway
├─ frontend/                Aplicación Vue 3 + Vite
├─ docs/                    Brief, diagramas C4, contrato de API y Trello
└─ README.md
```

## Requisitos

- Java 21.
- Node.js 22 LTS y npm (la versión esperada está en `.nvmrc`).
- PostgreSQL 16 local o un proyecto PostgreSQL en Neon.
- Git.

Comprueba las versiones:

```bash
java --version
node --version
npm --version
```

Si usas nvm, activa la versión del proyecto:

```bash
nvm install
nvm use
```

## Configuración local

### 1. Base de datos en Neon

Una persona del equipo crea el proyecto en [Neon](https://neon.tech/) y agrega a los demás integrantes.
Usen una rama `development` y la base `neondb` para el trabajo diario; mantengan `production` separada.

En **Connect**, seleccionen la rama `development`, la base `neondb` y el rol de desarrollo. Copien dos
cadenas conservando los parámetros SSL:

- **Connection pooling activado** para `DB_URL` (el host suele contener `-pooler`).
- **Connection pooling desactivado** para `DB_DIRECT_URL` (host directo, lo usa Flyway).

Spring necesita el formato JDBC. El usuario y la contraseña van separados de la URL:

```text
postgresql://USUARIO:CLAVE@HOST-pooler.REGION.aws.neon.tech/neondb?sslmode=require
        ↓
jdbc:postgresql://HOST-pooler.REGION.aws.neon.tech/neondb?sslmode=require
```

También pueden usar un PostgreSQL local y crear allí la base `peluqueria`.

### 2. Variables del backend

Crea `backend/.env` (archivo local, no se sube a Git) con:

```dotenv
PORT=3000
FRONTEND_URL=http://localhost:5173

DB_URL=jdbc:postgresql://HOST-pooler.REGION.aws.neon.tech/neondb?sslmode=require
DB_DIRECT_URL=jdbc:postgresql://HOST.REGION.aws.neon.tech/neondb?sslmode=require
DB_USER=
DB_PASSWORD=

APP_ADMIN_EMAIL=
APP_ADMIN_PASSWORD=
TELEFONO_PELUQUERIA=

JPA_SHOW_SQL=false
# JWT_SECRET=  (pendiente: historia de inicio de sesión)
```

Dónde obtener cada valor:

- **`PORT`**: puerto local del backend; por defecto `3000`. En producción puede inyectarlo la plataforma.
- **`FRONTEND_URL`**: URL del frontend permitida por CORS. Local: `http://localhost:5173`.
  Producción: el dominio real del frontend.
- **`DB_URL`**: Neon Dashboard > proyecto > **Connect**; rama `development`, base `neondb`, rol de
  desarrollo y **Connection pooling activado** (host con `-pooler`). Convierte
  `postgresql://...` en `jdbc:postgresql://...`.
- **`DB_DIRECT_URL`**: igual, pero con **Connection pooling desactivado** (host directo, sin
  `-pooler`). Flyway la usa para aplicar las migraciones.
- **`DB_USER` y `DB_PASSWORD`**: usuario y contraseña del rol de Neon; aparecen en la cadena de
  conexión o en **Project > Roles**.
- **`APP_ADMIN_EMAIL` y `APP_ADMIN_PASSWORD`**: los elige el equipo; crean el primer `ADMIN` solo si
  no existe ninguno. La contraseña debe tener 8 caracteres o más.
- **`TELEFONO_PELUQUERIA`**: teléfono corporativo que muestra la interfaz; lo proporciona el dueño
  del salón.
- **`JPA_SHOW_SQL`**: `true` solo para depurar SQL; normalmente `false`.
- **`JWT_SECRET`**: pendiente para la historia de inicio de sesión; se genera con
  `openssl rand -base64 48`.

`application.properties` solo contiene referencias `${...}`; nunca valores reales.

### 3. Backend

```bash
cd backend
./mvnw spring-boot:run
```

La API queda en `http://localhost:3000`. Al arrancar, Flyway aplica las migraciones pendientes y
Hibernate valida el esquema; no crea ni modifica tablas por su cuenta.

### 4. Frontend

```bash
cd frontend
npm install
npm run dev
```

Crea `frontend/.env` con:

```dotenv
VITE_API_URL=http://localhost:3000/api
```

`VITE_API_URL` es la URL del backend terminada en `/api`; en producción usa el dominio público del
backend. Reinicia Vite si cambias este archivo.

Abre la dirección que muestra Vite, normalmente `http://localhost:5173`.

## Endpoint de salud

```bash
curl http://localhost:3000/api/health
```

Respuesta esperada:

```json
{ "status": "ok" }
```

La vista inicial del frontend también muestra este estado.

## Base de datos y migraciones

- El esquema **solo** cambia con migraciones en `backend/src/main/resources/db/migration/`.
- La migración inicial es `V1__crear_usuarios.sql`.
- Nunca edites una migración ya aplicada: crea una nueva (`V2__...sql`, `V3__...sql`).
- `spring.jpa.hibernate.ddl-auto=validate`; no se usa `update` ni `create`.

## Comandos de calidad

```bash
# Backend
cd backend
./mvnw clean verify

# Frontend
cd frontend
npm run lint
npm run build
```

La prueba que levanta todo el contexto está deshabilitada hasta que el equipo configure una base de
datos PostgreSQL de pruebas.

## Gestión de tareas

El proyecto se gestiona en el tablero de Trello
[ingeniería de software](https://trello.com/b/gM0zISBq/ingenieria-de-software). Allí están las listas,
las tareas y el estado actual del equipo.

## Documentación

- [`docs/BRIEF EQUIPO 57 6.pdf`](docs/BRIEF%20EQUIPO%2057%206.pdf): brief del producto.
- `docs/`: diagramas C4 y contrato de la API (se incorporan conforme avancen las historias).

## Seguridad

- No subas archivos `.env`, credenciales de Neon ni secretos.
- Las contraseñas se guardan con BCrypt.
- Por defecto todas las rutas requieren autenticación, salvo `GET /api/health`.
- Los errores de la API usan el formato `{ "error": "CODIGO", "message": "..." }`.
