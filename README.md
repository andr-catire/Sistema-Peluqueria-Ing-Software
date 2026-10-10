# Sistema de Peluqueria

Base de trabajo para un sistema de gestion de peluqueria con API en Express, persistencia en
PostgreSQL mediante Prisma y una aplicacion cliente en Vue.

## Estructura

- `backend/`: API, reglas de negocio, pruebas y esquema Prisma.
- `frontend/`: interfaz Vue, vistas, componentes y acceso a la API.
- `docs/`: brief del producto, diagramas C4 y contrato de API.

## Inicio rapido

1. Copia `backend/.env.example` a `backend/.env` y completa la URL de PostgreSQL y el telefono.
2. Copia `frontend/.env.example` a `frontend/.env` si necesitas cambiar la URL de la API.
3. Instala dependencias ejecutando `npm install` dentro de `backend/` y `frontend/`.
4. Inicia la API con `npm run dev` dentro de `backend/`.
5. Inicia la interfaz con `npm run dev` dentro de `frontend/`.

No guardes credenciales reales en Git. Los archivos `.env` estan ignorados; los `.env.example`
solo contienen valores de ejemplo.