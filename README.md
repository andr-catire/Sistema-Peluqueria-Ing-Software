# Sistema de Peluqueria

Base de trabajo para un sistema de gestion de peluqueria con API en Express, persistencia en
PostgreSQL mediante Prisma y una aplicacion cliente en Vue.

## Estructura

- `backend/`: API, reglas de negocio, pruebas y esquema Prisma.
- `frontend/`: interfaz Vue, vistas, componentes y acceso a la API.
- `docs/`: brief del producto, diagramas C4 y contrato de API.

## Configuracion local

### Requisitos

- Node.js 22 LTS y npm. La version esperada esta indicada en `.nvmrc`.
- PostgreSQL 16 en ejecucion.
- Git.
- VS Code es opcional; las extensiones recomendadas estan en `.vscode/extensions.json`.

Comprueba las versiones antes de instalar:

```bash
node --version
npm --version
psql --version
```

Si utilizas nvm, instala y activa la version indicada por el proyecto desde la raiz:

```bash
nvm install
nvm use
```

### 1. Crear la base de datos

Inicia el servicio PostgreSQL. Luego crea una base de datos llamada `peluqueria` desde `psql`:

```bash
psql -U postgres -c "CREATE DATABASE peluqueria;"
```

Tambien puedes crearla desde pgAdmin. Si ya existe, no vuelvas a ejecutar el comando `CREATE DATABASE`.

Comprueba que PostgreSQL acepta conexiones:

```bash
pg_isready
```

### 2. Configurar variables de entorno

Crea los archivos locales a partir de las plantillas, ejecutando desde la raiz:

```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
```

Edita `backend/.env` y reemplaza `TU_PASSWORD` con la contrasena de tu usuario PostgreSQL. `DATABASE_URL` debe tener este formato:

```text
postgresql://USUARIO:CONTRASENA@localhost:5432/peluqueria?schema=public
```

Configura tambien un valor privado y largo para `JWT_SECRET`, el puerto de API en `PORT` (por defecto `3000`) y el telefono de la peluqueria en `TELEFONO_PELUQUERIA`. La plantilla incluye `FRONTEND_URL`; el CORS actual de Express esta abierto y todavia no usa esa variable para restringir origenes.

En `frontend/.env`, usa la URL base de la API:

```dotenv
VITE_API_URL=http://localhost:3000/api
```

Vite solo carga las variables con prefijo `VITE_` en el navegador. Reinicia el servidor de desarrollo si cambias este archivo.

### 3. Instalar dependencias

No hay un `package.json` en la raiz; instala las dependencias de cada aplicacion por separado:

```bash
npm --prefix backend install
npm --prefix frontend install
```

### 4. Iniciar el proyecto

Abre dos terminales desde la raiz del repositorio.

Terminal 1, API Express:

```bash
npm --prefix backend run dev
```

Terminal 2, interfaz Vue/Vite:

```bash
npm --prefix frontend run dev
```

Abre la direccion local que muestra Vite, normalmente `http://localhost:5173`.

### 5. Comprobar la conexion

Comprueba la API desde una terminal:

```bash
curl http://localhost:3000/api/health
```

La respuesta esperada es:

```json
{ "status": "ok" }
```

La vista de salud del frontend tambien muestra esta respuesta. Este endpoint confirma que Express esta respondiendo, pero no comprueba una conexion con PostgreSQL.

### Prisma y base de datos

Prisma lee `DATABASE_URL` desde `backend/.env`. El esquema esta en `backend/prisma/schema.prisma`, pero por ahora no contiene modelos ni migraciones de tablas de la aplicacion. Cuando se agreguen modelos, genera el cliente y crea/aplica la migracion de desarrollo con:

```bash
npm --prefix backend run prisma:generate
npm --prefix backend run prisma:migrate
```

### Comandos de calidad

```bash
npm --prefix backend run lint
npm --prefix backend test
npm --prefix frontend run lint
npm --prefix frontend run build
```

El comando de pruebas del backend usa `node --test`; actualmente no hay pruebas definidas.

### Solucion de problemas

- `pg_isready` no responde: inicia PostgreSQL y confirma que escucha en el puerto `5432`.
- Error de autenticacion de PostgreSQL: revisa usuario y contrasena en `backend/.env` y que exista la base `peluqueria`.
- Error de conexion desde la interfaz: confirma que la API este en el puerto configurado y que `frontend/.env` tenga `VITE_API_URL=http://localhost:3000/api`; reinicia Vite despues de cambiarlo.
- Puerto ocupado: cambia `PORT` en `backend/.env` y actualiza `VITE_API_URL` con el mismo puerto.

No guardes credenciales reales en Git. Los archivos `.env` estan ignorados; solo deben versionarse las plantillas `.env.example` con valores ficticios.
