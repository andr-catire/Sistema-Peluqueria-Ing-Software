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
- Una base PostgreSQL 16 local o un proyecto PostgreSQL alojado en Neon.
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

### 1. Crear la base de datos compartida en Neon

Para que todo el equipo trabaje con los mismos datos, una persona crea un proyecto en [Neon](https://neon.tech/) y agrega a los demas integrantes como miembros del proyecto desde la configuracion de acceso del proyecto/equipo. Usen una sola rama `development` y su base `neondb` para el trabajo diario; mantengan `production` separada.

En **Connect**, cada integrante selecciona la rama `development`, la base `neondb`, el rol de desarrollo (por ejemplo `neondb_owner`) y el compute `Primary`. Copia dos cadenas conservando los parametros SSL que Neon entrega: deja **Connection pooling** activado para la URL de `DATABASE_URL` (el host suele contener `-pooler`) y desactivalo para copiar la URL directa de `DIRECT_URL`.

Si los nombres de base, rol o branch iniciales no son los que desea el equipo, crealos o cambialos desde Neon antes de copiar las URL. No publiques esas URL en el repositorio ni las compartas en canales publicos.

Como alternativa, cada integrante puede usar su PostgreSQL local para desarrollo aislado. En ese caso, configura `DATABASE_URL` y `DIRECT_URL` con la URL local y crea alli la base `peluqueria`.

### 2. Configurar variables de entorno

Crea los archivos locales a partir de las plantillas, ejecutando desde la raiz:

```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
```

Edita `backend/.env` y reemplaza los valores de ejemplo con las dos URL de Neon. La contrasena es la del rol de base de datos que seleccionaste en Neon y aparece dentro de la URL; copia la cadena completa, no inventes una contrasena de PostgreSQL local. Usa la URL pooled en `DATABASE_URL` para las conexiones de la aplicacion y la URL directa en `DIRECT_URL` para Prisma CLI y las migraciones:

```text
DATABASE_URL="postgresql://USUARIO:CONTRASENA@HOST-pooler.REGION.aws.neon.tech/BASE?sslmode=require"
DIRECT_URL="postgresql://USUARIO:CONTRASENA@HOST.REGION.aws.neon.tech/BASE?sslmode=require"
```

Usa exactamente los hosts y parametros que entrega Neon; la forma anterior es ilustrativa. Si la contrasena contiene caracteres especiales, usa su codificacion URL en la cadena de conexion. La URL compartida actualmente es directa (no contiene `-pooler`); hasta copiar la variante pooled, puede usarse en ambas variables para probar. Configura tambien un valor privado y largo para `JWT_SECRET`, el puerto de API en `PORT` (por defecto `3000`) y el telefono de la peluqueria en `TELEFONO_PELUQUERIA`. La plantilla incluye `FRONTEND_URL`; el CORS actual de Express esta abierto y todavia no usa esa variable para restringir origenes.

Cada integrante debe crear su propio `backend/.env` local con las URL de la misma rama `development`; nunca se manda ese archivo a GitHub. Los miembros del proyecto pueden obtener las cadenas desde Neon. Si se comparte la cadena fuera de Neon, hacerlo por un canal privado y solo con el equipo. Para usar la aplicacion, cada integrante inicia su backend local, que conectara a la misma base Neon; para consultar la base directamente pueden usar Neon SQL Editor o un cliente PostgreSQL con la misma URL. No publiquen URL con contrasena en `.env.example`, README, capturas ni commits.

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

Prisma usa `DATABASE_URL` para la conexion de la aplicacion y `DIRECT_URL` para operaciones de esquema. El esquema esta en `backend/prisma/schema.prisma`, pero por ahora no contiene modelos ni migraciones de tablas de la aplicacion. Cuando se agreguen modelos, genera el cliente y crea una migracion desde una base local:

```bash
npm --prefix backend run prisma:generate
npm --prefix backend run prisma:migrate
```

Confirma la migracion en Git junto con los cambios del esquema. Para aplicar migraciones ya creadas a la base compartida de Neon, ejecuta:

```bash
npm --prefix backend run prisma:deploy
```

Acuerden quien las aplica y no ejecuten cambios de esquema simultaneamente. No uses una base compartida de produccion para desarrollo.

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
