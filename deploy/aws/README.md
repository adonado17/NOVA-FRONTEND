# Frontend en S3 y CloudFront

Aplicación: [NOVA Studio](https://d1b5jy58q5q89g.cloudfront.net/).

| Recurso | Valor |
| --- | --- |
| Región del bucket | Ohio, `us-east-2` |
| Stack CloudFormation | `nova-frontend` |
| Bucket privado | `nova-frontend-frontendbucket-ieaa1kpp5cs2` |
| Distribución CloudFront | `E12X49PK624K38` |
| API en EC2 | `https://3.19.116.135/api` |

## Cómo se conectan

El navegador descarga HTML, CSS y JavaScript desde CloudFront por HTTPS. CloudFront obtiene los objetos del bucket privado mediante Origin Access Control. La aplicación Angular ejecuta sus servicios en el navegador y llama por HTTPS a la API en EC2; Express consulta MongoDB Atlas y devuelve JSON.

S3 conserva Block Public Access y cifrado AES256. Su política permite acceso únicamente a la distribución indicada. No se publica `.env`, credenciales, código fuente del backend ni `node_modules`.

La función de CloudFront convierte `/`, `/login`, `/register`, `/profile` y `/reservations` en solicitudes de `index.html`, para que Angular resuelva la pantalla. Un archivo o ruta desconocidos conservan su error. La API debe permitir el origen exacto de CloudFront en `CORS_ORIGINS`.

## Plantilla

`frontend.yaml` define el bucket, OAC, política del bucket, política de caché, función de rutas y distribución HTTPS. Usa el certificado del dominio de CloudFront. No registra un dominio ni modifica el plan de la cuenta AWS; los recursos generan uso de AWS conforme al plan y los créditos disponibles.

Las salidas `BucketName`, `DistributionId` y `FrontendOrigin` permiten identificar los recursos para una publicación posterior. Para consultar el stack con una sesión de AWS CLI ya autorizada:

```powershell
aws cloudformation describe-stacks --stack-name nova-frontend --region us-east-2 --query 'Stacks[0].Outputs' --profile nova-deploy
```

## Publicar una actualización

Ejecuta los comandos en PowerShell 7.3 o superior desde la raíz del repositorio. El bloque detiene la publicación si falla npm o AWS CLI. El ejemplo usa el perfil `nova-deploy`; cambia esa variable si tu sesión autorizada tiene otro nombre. Primero revisa que `src/environments/environment.prod.ts` apunte a la API correcta.

```powershell
if ($PSVersionTable.PSVersion -lt [version]'7.3') { throw 'Se requiere PowerShell 7.3 o superior.' }
$ErrorActionPreference = 'Stop'
$PSNativeCommandUseErrorActionPreference = $true

npm ci
npm run build

$novaBucket = 'nova-frontend-frontendbucket-ieaa1kpp5cs2'
$novaDistribution = 'E12X49PK624K38'
$novaAwsProfile = 'nova-deploy'
$novaBuild = (Resolve-Path './dist/frontend/browser').Path

aws s3 sync "$novaBuild/" "s3://$novaBucket/" --exclude index.html --cache-control 'public,max-age=3600' --region us-east-2 --profile $novaAwsProfile

Get-ChildItem -LiteralPath $novaBuild -Recurse -File |
    Where-Object { $_.Name -match '-[A-Z0-9]{8,}\.(js|css)$' } |
    ForEach-Object {
        $novaKey = [IO.Path]::GetRelativePath($novaBuild, $_.FullName).Replace('\', '/')
        aws s3 cp $_.FullName "s3://$novaBucket/$novaKey" --cache-control 'public,max-age=31536000,immutable' --region us-east-2 --profile $novaAwsProfile
    }

aws s3 cp "$novaBuild/index.html" "s3://$novaBucket/index.html" --content-type 'text/html' --cache-control 'no-cache,max-age=0,must-revalidate' --region us-east-2 --profile $novaAwsProfile
aws cloudfront create-invalidation --distribution-id $novaDistribution --paths '/*' --profile $novaAwsProfile
```

Se publican primero los recursos y después el HTML que los referencia. Los archivos antiguos se conservan durante esta actualización para que una pestaña abierta pueda terminar de cargarlos. No vuelvas a subir el proyecto completo al bucket.

## Verificación

Abre la aplicación y recarga directamente `/login` y `/reservations`. Confirma que se cargan los JS/CSS y que un recurso inexistente devuelve error. Revisa `/api/ready` y prueba el login y el CRUD de una cuenta de prueba; elimina únicamente los datos creados para esa prueba.

El despliegue del 24 de noviembre de 2026 pasó 83 comprobaciones de publicación, rutas, recursos, HTTPS, salud y CORS. La comprobación adicional de API pasó 120 verificaciones de registro, sesión, CRUD y aislamiento entre usuarios, con limpieza de sus datos temporales.

Referencias: [OAC y bucket privado](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html), [endpoints web de S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteEndpoints.html).
