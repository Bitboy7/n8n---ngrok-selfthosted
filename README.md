# n8n + Ngrok + Postgres + Redis (Español)

Este repositorio contiene una configuración de Docker Compose para ejecutar:

- n8n (automatización de flujos)
- Ngrok (túnel público hacia n8n)
- Postgres 17 (base de datos)
- Redis (caché/cola con contraseña)
- pgAdmin 4 (admin de Postgres)
- Evolution API

Ngrok expone el servicio de n8n en Internet y n8n usa la URL pública para webhooks. Postgres y Redis almacenan datos en volúmenes persistentes.

Nota: La imagen de Postgres está fijada a `postgres:17` para garantizar compatibilidad de datos con el layout anterior. Si quieres migrar a Postgres 18+, revisa el documento MIGRACION_POSTGRES.md.

## Requisitos

- Docker Desktop (incluye Docker y Compose): https://docs.docker.com/get-docker/

## Estructura del proyecto

- `docker-compose.yaml`: definición de servicios y volúmenes
- `ngrok.yml`: configuración de túneles de Ngrok
- `nginx/` (opcional): archivos para un reverse proxy Nginx (no está activo en `docker-compose.yaml` por defecto)

## Variables de entorno (.env)

Copia `.env.example` a `.env` y completa los valores:

```
URL=https://tu-dominio.ngrok-free.app
TIMEZONE=America/Mexico_City
NGROK_TOKEN=tu_token_de_ngrok
REDIS_HOST_PASSWORD=una_contraseña_segura
# Variables adicionales para evolution-api: consulta la documentación de su imagen
```

Estas variables se usan así:

- `URL`: URL pública de n8n (debe coincidir con el dominio de `ngrok.yml`)
- `TIMEZONE` y `GENERIC_TIMEZONE`: zona horaria del contenedor n8n
- `NGROK_TOKEN`: token de autenticación de Ngrok
- `REDIS_HOST_PASSWORD`: contraseña requerida por Redis

## Configuración de Ngrok (`ngrok.yml`)

Ejemplo (ya incluido en el repo):

```yaml
version: 2
log_level: debug
tunnels:
  n8n:
    proto: http
    addr: n8n:5678
    domain: tu-dominio.ngrok-free.app
```

Asegúrate de usar el mismo dominio en `URL` dentro de `.env`, incluyendo `https://`.

## Servicios y puertos

- n8n: puerto interno 5678 (expuesto vía Ngrok)
- ngrok: puerto 4040 (interfaz local de Ngrok)
- postgres (db): puerto 5432 interno
- redis: puerto 6379 interno (con contraseña de `.env`)
- pgadmin4: 5050 (mapeado a `http://localhost:5050`)
- evolution-api: 3002 (mapeado a `http://localhost:3002`)

Volúmenes persistentes:

- `n8n_data`, `redis_data`, `postgres_data`, `evolution_instances`

## Uso

Arrancar en segundo plano:

```powershell
docker compose up -d
```

Ver estado y logs rápidos:

```powershell
docker compose ps
docker compose logs --tail 100
```

Detener y eliminar contenedores (volúmenes se conservan):

```powershell
docker compose down
```

## Acceso a las apps

- n8n: abre la URL configurada en `URL` (dominio Ngrok)
- Ngrok Web UI: http://localhost:4040
- pgAdmin 4: http://localhost:5050
  - Usuario: `demo@demo.com`
  - Contraseña: `2020PASS`
  - Conexión a Postgres desde pgAdmin:
    - Host: `db`
    - Puerto: `5432`
    - Usuario: `user`
    - Contraseña: `pass`
    - Base de datos: `evolution`
- Evolution API: http://localhost:3002

## Notas sobre Postgres 17 vs 18+

Las imágenes oficiales de Postgres 18+ cambiaron el layout recomendado del directorio de datos. En este proyecto se fija la versión a `postgres:17` para mantener compatibilidad con el volumen `postgres_data:/var/lib/postgresql/data` y evitar errores de arranque.

Si quieres actualizar a 18+ conservando datos, sigue MIGRACION_POSTGRES.md.

## Solución de problemas

- Postgres entra en bucle de reinicio con un mensaje sobre layout de datos (pg_ctlcluster/18+):

  - Causa: se está usando una imagen 18+ con un volumen inicializado en `/var/lib/postgresql/data`.
  - Solución rápida: fijar `image: postgres:17` (ya aplicado en este repo) o migrar a 18+ siguiendo el documento de migración.

- Ngrok no levanta el túnel:

  - Verifica `NGROK_TOKEN` en `.env` y el dominio en `ngrok.yml`.

- n8n no recibe webhooks:
  - Asegura que `URL` en `.env` coincide con el dominio configurado en `ngrok.yml` y que Ngrok está en ejecución.

## Licencia

MIT
