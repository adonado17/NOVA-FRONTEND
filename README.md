# NOVA Frontend

Aplicación Angular y TypeScript de NOVA Studio. Incluye la portada, el catálogo de servicios, el registro, el inicio de sesión, el perfil y el CRUD de solicitudes de citas. El diseño adaptable y las funciones existentes se conservan en este repositorio independiente.

El servidor Express y la conexión con MongoDB Atlas se encuentran en [NOVA-BACKEND](https://github.com/adonado17/NOVA-BACKEND).

## Requisitos

- Node.js 24, desde la versión 24.15 y anterior a la 25.
- npm 11. El proyecto declara npm 11.17.0 en `packageManager`.
- Para desarrollo local, el backend debe estar funcionando en `http://localhost:3000`.

Todos los comandos siguientes se ejecutan desde la raíz de **NOVA-FRONTEND**.

## Instalar y desarrollar

```powershell
npm ci
npm start
```

Abre `http://localhost:4200`. El proxy definido en `proxy.conf.json` reenvía las solicitudes `/api` a `http://localhost:3000`. El frontend y el backend se inician en terminales separadas, cada uno desde su propio repositorio.

## Configuración de la API

Los servicios de autenticación y reservas utilizan la URL central exportada por `src/app/utils/api.ts`.

| Compilación | Archivo de configuración | URL base |
| --- | --- | --- |
| Desarrollo y pruebas habituales | `src/environments/environment.ts` | `/api`, mediante el proxy local |
| Producción | `src/environments/environment.prod.ts` | `https://3.19.116.135/api` |

`angular.json` sustituye el archivo de desarrollo por el de producción mediante `fileReplacements` al ejecutar la compilación de producción. Para cambiar el backend público, modifica únicamente `apiUrl` en `environment.prod.ts` y vuelve a compilar.

`https://3.19.116.135` corresponde a la instancia actual, que todavía ejecuta la versión anterior. Antes de usar este frontend desde otro dominio se debe desplegar NOVA-BACKEND separado y configurar `CORS_ORIGINS` con el origen real del frontend. Separar y publicar los repositorios en GitHub no actualiza AWS.

Estas direcciones son configuración pública que forma parte del JavaScript enviado al navegador. No se deben incluir contraseñas, `MONGODB_URI`, secretos JWT ni credenciales de AWS en los archivos de Angular. El backend mantiene sus secretos y autoriza los orígenes del frontend mediante CORS.

## Compilar

```powershell
npm run build
```

La configuración predeterminada es **producción** y el resultado publicable queda en `dist/frontend/browser/`. Ese directorio contiene `index.html`, los archivos JavaScript y CSS compilados y los recursos de `public/`.

Para generar una compilación que use la API de desarrollo:

```powershell
npm run build -- --configuration development
```

El proxy solo funciona con el servidor de desarrollo de Angular; no forma parte de los archivos estáticos compilados.

## Pruebas

```powershell
npm test -- --watch=false
```

Las pruebas comprueban el renderizado básico de la aplicación y las peticiones del CRUD con JWT mediante un cliente HTTP simulado. No crean usuarios ni citas en el servidor público.

La misma prueba del servicio también se puede ejecutar con la configuración de producción para comprobar la selección de la URL pública:

```powershell
npm test -- --watch=false --build-target=frontend:build:production
```

## Rutas

| Ruta | Función |
| --- | --- |
| `/` | Portada, servicios, beneficios y contacto |
| `/register` | Crear una cuenta |
| `/login` | Iniciar sesión |
| `/profile` | Consultar el perfil; requiere sesión |
| `/reservations` | Crear, consultar, actualizar y eliminar solicitudes de citas; requiere sesión |

Las solicitudes de cita permanecen pendientes de confirmación. El guard controla el acceso a las páginas protegidas y la API verifica el JWT en las operaciones protegidas.

## Organización y recursos

- `src/app/pages/`: páginas de la aplicación.
- `src/app/services/`: comunicación HTTP con la API y sesión.
- `src/app/guards/`, `interfaces/` y `shared/`: protección de rutas, tipos y navegación compartida.
- `src/environments/`: dirección de la API por configuración de compilación.
- `public/`: recursos estáticos incluidos en el resultado del build; `public/img/favicon.svg` conserva el favicon de NOVA.

Las fotografías actuales de la portada y los servicios conservan sus URLs de Unsplash. Los recursos locales se incluyen desde `public/`; no dependen de carpetas del repositorio anterior.

## Publicación en S3 pendiente

Este repositorio prepara el frontend estático independiente. La creación del alojamiento en S3 y su publicación quedan pendientes de la clase; aquí no se crean buckets, políticas ni recursos AWS.

Al configurar ese alojamiento se deberá publicar el contenido de `dist/frontend/browser/`, resolver las rutas de la SPA hacia `index.html` y registrar el origen real del frontend en la configuración CORS de NOVA-BACKEND. La publicación y la configuración de HTTPS se verificarán en ese paso.
