# Deployment Guide

## Overview

This document outlines the deployment process for the Consilium application. It covers deployment options, configurations, and best practices for both the frontend and backend components.

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Client Devices                              │
│                                                                     │
│  ┌───────────────────┐   ┌───────────────────┐   ┌───────────────┐  │
│  │    Web Browser    │   │  Mobile Browser   │   │   PWA App     │  │
│  └─────────┬─────────┘   └──────────┬────────┘   └───────┬───────┘  │
└────────────┼──────────────────────────────────────────────┼─────────┘
             │                        │                     │
             ▼                        ▼                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          CDN (CloudFlare)                           │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       Static Hosting (Vercel)                       │
│                                                                     │
│                        Frontend Application                         │
│                                                                     │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      API Gateway (Optional)                        │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   Backend Service (Cloud Run/Heroku)                │
│                                                                     │
│                     FastAPI Backend Application                     │
│                                                                     │
└──────────┬────────────────────────────────────────┬─────────────────┘
           │                                        │
           ▼                                        ▼
┌─────────────────────────┐              ┌─────────────────────────────┐
│     Supabase Cloud      │              │     Vector Database Cloud   │
│                         │              │     (Pinecone/Qdrant)       │
│  - Auth                 │              │                             │
│  - PostgreSQL Database  │              │  - Embeddings Storage       │
│  - Storage              │              │  - Similarity Search        │
└─────────────────────────┘              └─────────────────────────────┘
```

## Frontend Deployment

### Building for Production

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Build for production
npm run build
```

This will generate a production-ready build in the `frontend/dist` directory.

### Deployment Options

#### Vercel (Recommended)

1. Set up a Vercel account and connect your repository
2. Configure build settings:
   - Build Command: `npm run build`
   - Output Directory: `dist`
3. Set environment variables in Vercel dashboard
4. Deploy your application

#### Netlify

1. Set up a Netlify account and connect your repository
2. Configure build settings:
   - Build Command: `npm run build`
   - Publish Directory: `dist`
3. Set environment variables in Netlify dashboard
4. Deploy your application

#### Manual Deployment

1. Build the application
2. Upload the contents of the `dist` directory to your web server
3. Configure your web server with appropriate caching headers and routing

### Environment Variables

Set these environment variables in your hosting platform:

```
VITE_API_URL=https://api.yourdomain.com
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### PWA Configuration

For Progressive Web App functionality:

1. Ensure service worker is properly registered
2. Customize the `manifest.json` file with your app details
3. Generate appropriate icons for different platforms
4. Test PWA installation on various devices

## Backend Deployment

### Containerization

The backend is containerized using Docker for consistent deployment:

```bash
# Build Docker image
docker build -t consilium-backend -f docker/backend.Dockerfile .

# Run container locally
docker run -p 8000:8000 --env-file .env consilium-backend
```

### Deployment Options

#### Cloud Run (Google Cloud)

1. Push Docker image to Google Container Registry:
   ```bash
   gcloud builds submit --tag gcr.io/your-project/consilium-backend
   ```

2. Deploy to Cloud Run:
   ```bash
   gcloud run deploy consilium-backend \
     --image gcr.io/your-project/consilium-backend \
     --platform managed \
     --region us-central1 \
     --allow-unauthenticated
   ```

3. Set environment variables in Cloud Run service settings

#### Heroku

1. Install Heroku CLI and log in
2. Create a Heroku app:
   ```bash
   heroku create consilium-backend
   ```

3. Add Heroku container registry:
   ```bash
   heroku container:login
   heroku container:push web -a consilium-backend
   heroku container:release web -a consilium-backend
   ```

4. Set environment variables:
   ```bash
   heroku config:set KEY=VALUE -a consilium-backend
   ```

#### Railway

1. Connect your GitHub repository to Railway
2. Configure environment variables in Railway dashboard
3. Deploy automatically when code is pushed

### Scaling Configuration

Adjust the following parameters based on your expected load:

```python
# in main.py
import uvicorn

if __name__ == "__main__":
    uvicorn.run(
        "app.main:app",
        host="0.0.0.0",
        port=int(os.getenv("PORT", "8000")),
        workers=int(os.getenv("WORKERS", "4")),  # Adjust workers based on CPU cores
        timeout_keep_alive=int(os.getenv("TIMEOUT_KEEP_ALIVE", "65")),
        log_level="info"
    )
```

### Environment Variables

Set these environment variables in your hosting platform:

```
# Database
DATABASE_URL=your_production_database_url

# Authentication
SECRET_KEY=your_secret_key  # Generate a secure random key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Services
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
XAI_API_KEY=your_xai_api_key
OPENROUTER_API_KEY=your_openrouter_api_key

# Supabase
SUPABASE_URL=your_supabase_url
SUPABASE_SERVICE_KEY=your_supabase_service_key

# Vector DB
VECTOR_DB_URL=your_vector_db_url
VECTOR_DB_API_KEY=your_vector_db_api_key
```

## Database Deployment

### Supabase

1. Create a new project in Supabase
2. Run migrations against your Supabase database:
   ```bash
   DATABASE_URL=your_supabase_url pdm run alembic upgrade head
   ```
3. Configure Row Level Security policies
4. Set up initial data if needed

### Vector Database

#### Pinecone

1. Create an account on Pinecone
2. Create a new index with appropriate dimensions (based on your embedding model)
3. Set the API key and environment in your backend configuration

#### Qdrant

1. Create an account on Qdrant Cloud or set up a self-hosted instance
2. Create a new collection with appropriate vector configuration
3. Set the API key and URL in your backend configuration

## Continuous Deployment

### GitHub Actions

Set up a GitHub Actions workflow for automated deployment:

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        run: cd frontend && npm ci
      - name: Build
        run: cd frontend && npm run build
      - uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
          working-directory: frontend

  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: google-github-actions/auth@v0
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
      - uses: google-github-actions/setup-gcloud@v0
      - name: Build and Deploy
        run: |
          gcloud builds submit --tag gcr.io/${{ secrets.GCP_PROJECT }}/consilium-backend
          gcloud run deploy consilium-backend \
            --image gcr.io/${{ secrets.GCP_PROJECT }}/consilium-backend \
            --platform managed \
            --region us-central1 \
            --allow-unauthenticated
```

## Post-Deployment Tasks

### Database Migrations

For updating the database schema after deployment:

```bash
# Run from a secure environment with database access
DATABASE_URL=your_production_database_url pdm run alembic upgrade head
```

### Health Checks

Implement and monitor health check endpoints:

- `GET /api/health`: Basic backend health check
- `GET /api/health/db`: Database connectivity check
- `GET /api/health/ai`: AI services check

### Monitoring

Set up monitoring using appropriate tools:

- Application monitoring: New Relic, Datadog, or Sentry
- Log aggregation: Elasticsearch, Loki, or CloudWatch
- Uptime monitoring: UptimeRobot or Pingdom
- Performance monitoring: Lighthouse for frontend

### Backup Strategy

1. Configure regular Supabase database backups
2. Set up backup verification procedures
3. Document backup restoration process

## Security Considerations

### SSL/TLS Configuration

Ensure all communications are encrypted with HTTPS:

1. Configure SSL certificates on your hosting platform
2. Set up automatic certificate renewal
3. Implement HSTS headers

### API Rate Limiting

Protect your API from abuse:

```python
# in app/main.py
from fastapi import FastAPI, Request
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.middleware.cors import CORSMiddleware
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.errors import RateLimitExceeded
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

app.add_middleware(
    TrustedHostMiddleware, allowed_hosts=["api.yourdomain.com", "localhost"]
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Secrets Management

Never hardcode sensitive information:

1. Use environment variables for all secrets
2. Rotate API keys regularly
3. Use secret management services like Google Secret Manager or AWS Secrets Manager

## Rollback Procedures

### Frontend Rollback

1. In Vercel/Netlify dashboard, select a previous deployment
2. Click "Redeploy" or "Restore deployment"

### Backend Rollback

For Cloud Run:

```bash
gcloud run services update consilium-backend --image gcr.io/your-project/consilium-backend:previous-tag
```

For Heroku:

```bash
heroku rollback -a consilium-backend
```

### Database Rollback

Use Alembic to revert to a previous migration:

```bash
DATABASE_URL=your_production_database_url pdm run alembic downgrade -1
```

## Troubleshooting Common Issues

### CORS Issues

- Ensure the API's CORS configuration includes your frontend domain
- Check for protocol mismatches (http vs https)
- Verify all necessary headers are allowed

### Environment Variable Problems

- Confirm all required variables are set in the hosting environment
- Check for typos in variable names
- Verify frontend variables are prefixed with `VITE_`

### Database Connection Issues

- Verify database credentials and connection strings
- Check network rules and firewalls
- Ensure database service is running and accessible

## Maintenance Procedures

### Dependency Updates

Regularly update dependencies:

```bash
# Frontend
cd frontend
npm outdated
npm update

# Backend
cd backend
pdm update
```

### Performance Optimization

- Regularly run Lighthouse audits on the frontend
- Monitor API response times and optimize slow endpoints
- Analyze and optimize database queries

### Capacity Planning

- Monitor resource usage and scale accordingly
- Set up alerts for approaching limits
- Document scaling procedures for different components 