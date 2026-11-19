# Sistema Escolar con Node.js, React y MariaDB

Aplicación web completa (Full-Stack) para la administración escolar, dividida en dos proyectos independientes (`backend` y `frontend`) que se comunican mediante una API REST segura.

## Tecnologías Utilizadas

- **Backend:** Node.js, Express 5, MariaDB (`mysql2/promise`), JWT (cookies `httpOnly` + `SameSite=Strict`), `bcryptjs`, `Zod`, `Helmet`, `express-rate-limit`, `Multer` y `node:test`.
- **Frontend:** React 19, Vite (con proxy configurado), React Router DOM 7 y Bootstrap 5.

## Características y Seguridad

- **Autenticación y Roles (`admin` y `usuario`):**
  - Solo existen dos cuentas de administrador: `Admin` (`$3cr3t`) y `Root` (`W@t@SHi09`).
  - **Defensa en profundidad:** Tres triggers en MariaDB (`trg_admin_max_insert`, `trg_admin_max_update`, `trg_admin_no_delete`) impiden crear un tercer administrador o eliminar a los existentes incluso si se accede directamente por SQL.
  - **Principio de mínimo privilegio:** El backend se conecta con el usuario exclusivo `lab_app` que solo posee permisos `SELECT, INSERT, UPDATE, DELETE` sobre `lab_escolar.*`.
- **Protección contra ataques:**
  - Consultas parametrizadas contra Inyección SQL.
  - Validación estricta y lista blanca con **Zod** (mensajes en español y política de contraseñas seguras).
  - Mitigación de XSS y CSRF mediante JWT almacenado en cookie `httpOnly` con `SameSite=Strict`.
  - Protección contra fuerza bruta por IP (`express-rate-limit`) y bloqueo temporal de cuenta tras 5 intentos fallidos (`423 Locked`).
  - Validación de subida de imágenes verificando tamaño, extensión, nombre aleatorio UUID y firma binaria (*magic bytes*).
- **Catálogos CRUD y Perfil:**
  - Gestión completa de **Alumnos**, **Maestros**, **Materias**, **Grupos** y **Usuarios** (solo administradores).
  - Módulo de perfil para actualizar foto y contraseña.

## Estructura del Proyecto

```text
lab-escolar/
├── backend/
│   ├── database/schema.sql        # Esquema SQL, llaves foráneas, triggers y usuario lab_app
│   ├── src/
│   │   ├── config/db.js           # Pool de conexiones a MariaDB
│   │   ├── middleware/            # auth.js, errores.js, subida.js, validar.js
│   │   ├── routes/                # crud.js, auth.routes.js, catalogos.routes.js, usuarios.routes.js
│   │   ├── scripts/seed.js        # Carga inicial de Admin, Root y datos de ejemplo
│   │   ├── validators/esquemas.js # Esquemas Zod
│   │   ├── app.js                 # Configuración Express
│   │   └── server.js              # Punto de entrada del servidor
│   ├── test/api.test.js           # 13 pruebas automáticas con node:test
│   └── api.http                   # Peticiones manuales para REST Client
└── frontend/
    ├── src/
    │   ├── components/            # Avatar.jsx, CrudPage.jsx, RutaProtegida.jsx
    │   ├── config/entidades.js    # Definición declarativa de los 5 catálogos
    │   ├── context/AuthContext.jsx# Estado global de sesión
    │   ├── layouts/               # DashboardLayout.jsx
    │   ├── pages/                 # Login.jsx, Inicio.jsx, Perfil.jsx
    │   ├── api.js                 # Cliente fetch centralizado
    │   ├── App.jsx                # Definición de rutas
    │   └── index.css              # Identidad visual (pizarra verde + lápiz amarillo)
    └── vite.config.js             # Proxy de /api y /uploads hacia http://127.0.0.1:3000
```

## Instalación y Ejecución

### 1. Base de datos (MariaDB)
```bash
sudo mariadb < backend/database/schema.sql
```

### 2. Backend
```bash
cd backend
cp .env.example .env
# Configura DB_PASSWORD y JWT_SECRET en backend/.env
npm install
npm run seed
npm test
npm run dev
```

### 3. Frontend
```bash
cd frontend
npm install
npm run dev
```

Abre `http://localhost:5173` en tu navegador.
