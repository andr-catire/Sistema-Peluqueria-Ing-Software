# Sistema de Peluqueria

Base de trabajo para un sistema de gestion de peluqueria con API en Express, persistencia en
PostgreSQL mediante Prisma y una aplicacion cliente en Vue.

## Estructura

- `backend/`: API, reglas de negocio, pruebas y esquema Prisma.
- `frontend/`: interfaz Vue, vistas, componentes y acceso a la API.
- `docs/`: brief del producto, diagramas C4 y contrato de API.

## Inicio rapido

### Requisitos

- Node.js 22 LTS (`.nvmrc`), Git y PostgreSQL 16.
- VS Code recomienda las extensiones compartidas desde `.vscode/extensions.json`.

### Preparacion local

1. Crea la base de datos `peluqueria` en PostgreSQL.
2. Copia `backend/.env.example` a `backend/.env` y configura `DATABASE_URL`, `JWT_SECRET` y el telefono.
3. Copia `frontend/.env.example` a `frontend/.env` si el archivo no existe o si necesitas cambiar la URL de la API.
4. Instala dependencias desde la raiz con `npm --prefix backend install` y `npm --prefix frontend install`.
5. Inicia la API con `npm --prefix backend run dev`.
6. Inicia la interfaz con `npm --prefix frontend run dev`.

Los paquetes incluyen los comandos `lint`; el frontend tambien incluye `build`.

No guardes credenciales reales en Git. Los archivos `.env` estan ignorados; los `.env.example`
solo contienen valores de ejemplo.
