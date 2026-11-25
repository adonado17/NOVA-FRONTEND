# NOVA Frontend

Interfaz de NOVA Studio desarrollada con Angular y TypeScript. Incluye servicios, registro, inicio de sesión, perfil y solicitudes de cita.

API: [NOVA-BACKEND](https://github.com/adonado17/NOVA-BACKEND).

Aplicación publicada: [NOVA Studio](https://d1b5jy58q5q89g.cloudfront.net/).

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

El frontend está publicado de forma independiente en un bucket privado de S3. CloudFront entrega los archivos por HTTPS y accede al bucket mediante Origin Access Control. El backend continúa en EC2 y las peticiones se envían a `https://3.19.116.135/api`.

La plantilla `deploy/aws/frontend.yaml` administra el bucket, su política, la distribución y la resolución de rutas. Solo las rutas de Angular conocidas se reescriben a `index.html`; los recursos inexistentes conservan su error. `index.html` se publica sin caché de larga duración y los JS/CSS con nombre versionado pueden conservarse un año.

La API permite el origen exacto `https://d1b5jy58q5q89g.cloudfront.net` en `CORS_ORIGINS`. La dirección anterior de EC2 redirige las páginas al frontend y conserva las rutas `/api` para la API.

Instrucciones y recursos desplegados: [Publicación en S3 y CloudFront](deploy/aws/README.md).

## Recorrido del código

Para mostrar el registro, la sesión y las citas, sigue este orden:

1. `src/app/app.routes.ts`: rutas públicas y pantallas protegidas.
2. `src/app/services/auth.service.ts`: peticiones de registro e inicio de sesión, token en `sessionStorage` y cabecera Bearer.
3. `src/app/guards/auth.guard.ts`: redirección al login cuando falta una sesión local.
4. `src/app/pages/reservations/reservations.component.ts` y su HTML: formulario, validación y acciones de la pantalla.
5. `src/app/services/reservation.service.ts`: llamadas HTTP para crear, listar, editar y eliminar citas.
6. `src/app/utils/api.ts` y `src/environments/environment.prod.ts`: dirección del backend de producción.

El guard orienta la navegación; la autorización definitiva se comprueba en la API mediante JWT y el propietario de cada cita.
