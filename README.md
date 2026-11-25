# NOVA Frontend

Interfaz de NOVA Studio desarrollada con Angular y TypeScript. Incluye servicios, registro, inicio de sesión, perfil y solicitudes de cita.

API: [NOVA-BACKEND](https://github.com/adonado17/NOVA-BACKEND).

## Requisitos

- Node.js 24.15 o superior dentro de la versión 24.
- npm 11.
- Backend local en `http://localhost:3000`.

## Ejecución local

Desde la raíz del repositorio:

```powershell
npm ci
npm start
```

Abre `http://localhost:4200`. `proxy.conf.json` envía las peticiones `/api` al backend local. Cada repositorio se ejecuta en su propia terminal.

## Dirección de la API

| Entorno | Archivo | Dirección |
| --- | --- | --- |
| Desarrollo | `src/environments/environment.ts` | `/api` |
| Producción | `src/environments/environment.prod.ts` | `https://3.19.116.135/api` |

Los servicios importan la dirección desde `src/app/utils/api.ts`. Para cambiar el servidor de producción, edita `apiUrl` en `environment.prod.ts` y vuelve a compilar. Estas variables son públicas; las credenciales de MongoDB, JWT y AWS pertenecen al backend.

## Compilación

```powershell
npm run build
```

Los archivos para publicar quedan en `dist/frontend/browser/`. La compilación usa el entorno de producción. El proxy de desarrollo no se incluye en esos archivos.

## Pruebas

```powershell
npm test -- --watch=false
npm test -- --watch=false --build-target=frontend:build:production
```

Las pruebas revisan el componente principal y las peticiones de citas con JWT mediante respuestas HTTP simuladas.

## Rutas

| Ruta | Pantalla |
| --- | --- |
| `/` | Inicio y servicios |
| `/register` | Registro |
| `/login` | Iniciar sesión |
| `/profile` | Perfil |
| `/reservations` | Solicitudes de cita |

Perfil y citas requieren sesión. Las solicitudes quedan pendientes de confirmación de disponibilidad.

## Archivos principales

- `src/app/pages/`: páginas de la aplicación.
- `src/app/services/`: autenticación y citas.
- `src/app/guards/`: protección de rutas.
- `src/app/interfaces/`: tipos de datos.
- `src/app/shared/`: navegación.
- `src/environments/`: configuración de la API.
- `public/`: recursos estáticos. Las fotografías se cargan desde Unsplash.

## Publicación

El alojamiento independiente en S3 está pendiente. Debe servir `dist/frontend/browser/` y resolver las rutas de Angular hacia `index.html`.

La instancia `https://3.19.116.135` mantiene el despliegue anterior. Para conectar este frontend desde otro dominio, primero hay que desplegar la API independiente y añadir el origen del frontend a `CORS_ORIGINS` en NOVA-BACKEND.
