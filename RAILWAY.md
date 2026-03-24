# Deploying OpenStatus to Railway

This guide covers deploying the OpenStatus stack to Railway as a single project with multiple services.

## Architecture

The deployment consists of 3 services:

| Service | Description | Port | Dockerfile |
|---------|-------------|------|------------|
| **dashboard** | Admin UI (Next.js) | 3000 | `apps/dashboard/Dockerfile` |
| **server** | API Server (Hono/Bun) | 3000 | `apps/server/Dockerfile` |
| **status-page** | Public Status Pages (Next.js) | 3000 | `apps/status-page/Dockerfile` |

## Prerequisites

1. [Railway account](https://railway.app)
2. [Turso database](https://turso.tech) (libSQL)
3. [Resend account](https://resend.com) for email authentication

## Deployment Steps

### 1. Create a New Railway Project

```bash
# Install Railway CLI
curl -fsSL https://railway.app/install.sh | sh

# Login to Railway
railway login

# Initialize project
railway init
```

### 2. Configure Environment Variables

Set the following variables in the Railway dashboard under **Variables** for each service:

#### Dashboard Service Variables
```
NODE_ENV=production
SELF_HOST=true
NEXT_PUBLIC_APP_URL=https://dashboard.<your-domain>
NEXT_PUBLIC_URL=https://dashboard.<your-domain>
NEXT_PUBLIC_SERVER_URL=https://api.<your-domain>
DATABASE_URL=libsql://<your-database>.turso.io
DATABASE_AUTH_TOKEN=<your-turso-token>
AUTH_SECRET=<generate-with-openssl-rand-base64-32>
RESEND_API_KEY=re_<your-resend-key>
```

#### Server Service Variables
```
NODE_ENV=production
DATABASE_URL=libsql://<your-database>.turso.io
DATABASE_AUTH_TOKEN=<your-turso-token>
PORT=3000
```

#### Status Page Service Variables
```
NODE_ENV=production
SELF_HOST=true
NEXT_PUBLIC_APP_URL=https://status.<your-domain>
NEXT_PUBLIC_URL=https://status.<your-domain>
DATABASE_URL=libsql://<your-database>.turso.io
DATABASE_AUTH_TOKEN=<your-turso-token>
```

### 3. Configure Custom Domains

1. In Railway dashboard, select each service
2. Go to **Settings** → **Networking** → **Custom Domains**
3. Add your custom domain:
   - Dashboard: `dashboard.yourdomain.com` → points to `dashboard` service
   - API: `api.yourdomain.com` → points to `server` service
   - Status: `status.yourdomain.com` → points to `status-page` service

### 4. Deploy

```bash
# Deploy all services
railway up

# Or deploy a specific service
railway up --service dashboard
railway up --service server
railway up --service status-page
```

### 5. Verify Deployment

Check the service logs:
```bash
railway logs --service dashboard
railway logs --service server
railway logs --service status-page
```

## Service Communication

Services communicate internally using Railway's DNS:

| From | To | Hostname |
|------|----|----------|
| dashboard | server | `server` |
| server | database | Turso Cloud |
| status-page | server | `server` |

## Optional Services

### Redis (Upstash)

For caching, add Upstash Redis:
```
UPSTASH_REDIS_REST_URL=https://your-redis.upstash.io
UPSTASH_REDIS_REST_TOKEN=your-token
```

### Analytics (Tinybird)

For monitor analytics:
```
TINY_BIRD_API_KEY=your-tinybird-key
```

## Troubleshooting

### Build Failures

If builds fail, ensure:
1. All environment variables are set correctly
2. The Turso database is accessible
3. Network access is allowed for external services

### Health Check Failures

Each service has a health check configured:
- Dashboard: `GET /`
- Server: `GET /ping`
- Status Page: `GET /`

Ensure the health check paths return 200 OK.

### Database Connection Issues

Make sure `DATABASE_URL` uses the `libsql://` protocol:
```
DATABASE_URL=libsql://your-db.turso.io
```

## Environment Reference

See `.env.railway.example` for a complete list of all supported environment variables.
