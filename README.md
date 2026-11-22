# CGS

Centre for Gender Studies, Nasarawa State University, Keffi. Built with Next.js.

## Local development

Use Node.js 24 and npm:

```sh
npm ci
cp .env.example .env.local
npm run dev
```

## Deploy with Dokploy

Create a **Docker Compose** service in your Dokploy project, connect this
repository and select the branch to deploy. Configure these settings:

| Setting | Value |
| --- | --- |
| Build path | `/` (repository root) |
| Compose type | `Docker Compose` |
| Compose path | `./docker-compose.yml` |
| Domain Service Name | `web` |
| Domain Path | `/` |
| Domain Container Port | `3000` |

In the **Environment** tab, set the public origin to your domain:

```dotenv
NEXT_PUBLIC_SITE_URL=https://cgs.example.edu.ng
```

Compose passes this variable to the Dockerfile as a build argument. Replace the
example with the actual public URL, including `https://`. This value
is embedded during the build in metadata, structured data, sitemap and robots
URLs. Changing it requires a rebuild; setting only a runtime environment variable
will not update the generated pages. Without an override, the Dockerfile preserves
the existing `https://cgs-nu.vercel.app` default.

Save the settings, add your domain in **Domains**, select service `web` and
container port `3000`, then deploy. Dokploy adds the Traefik routing labels and
network configuration automatically. Redeploy after changing domain settings.
Enable HTTPS with Let's Encrypt once DNS resolves to the server. Compose exposes
port `3000` internally; no host port mapping is needed. The image already
sets `NODE_ENV=production`, `PORT=3000` and `HOSTNAME=0.0.0.0`, and starts the
standalone server automatically.

The multi-stage image installs locked dependencies with `npm ci`, builds the app,
and copies the standalone server, static files and public assets into a non-root
runtime. It includes an HTTP health check at `/`. No database or persistent volume
is required for the current app. Local `.env` files are excluded from the Docker
context. The build server needs access to npm and Google Fonts because the app
uses `next/font/google`.

The Compose service builds the existing Dockerfile, inherits its health check,
and restarts unless explicitly stopped. Select standard Docker Compose mode:
Docker Stack cannot build the image from the repository.

See Dokploy's [Docker Compose settings](https://docs.dokploy.com/docs/core/docker-compose)
and [Compose domain settings](https://docs.dokploy.com/docs/core/docker-compose/domains).

### Verify the container locally

```sh
cp .env.example .env.local
# Edit NEXT_PUBLIC_SITE_URL in .env.local before building.
docker compose --env-file .env.local config
docker compose --env-file .env.local build
docker compose --env-file .env.local run --rm --name cgs -p 3000:3000 web
```

Open `http://localhost:3000`. In another terminal, check the homepage, assets and
generated metadata, then inspect the health status:

```sh
curl --fail http://localhost:3000/
curl --fail http://localhost:3000/assets/cgs-logo.jpg --output /dev/null
curl --fail http://localhost:3000/sitemap.xml
curl --fail http://localhost:3000/robots.txt
docker inspect --format '{{.State.Health.Status}}' cgs
```
