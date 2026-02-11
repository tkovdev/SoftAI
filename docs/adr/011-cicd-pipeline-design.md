# ADR-011: CI/CD Pipeline Design

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer  
**Supersedes**: N/A

---

## Context

Continuous Integration and Continuous Deployment (CI/CD) enables:
- Automated builds on every commit
- Automated testing to catch issues early
- Consistent, repeatable deployments
- Reduced manual errors
- Faster feedback loops
- Faster time to market

We need to define:
- CI/CD platform selection
- Pipeline stages and workflow
- Build and test automation
- Deployment strategy (environments, approvals)
- Docker containerization approach
- Secrets management
- Rollback procedures

Without CI/CD:
- Manual deployments are error-prone
- Testing is inconsistent
- Deployments are slow and risky
- Environment inconsistencies cause bugs
- Team productivity suffers

---

## Decision

We will use **GitHub Actions** as our CI/CD platform with **Docker** for containerization, implementing automated build, test, and deployment pipelines with progressive deployment through dev → staging → production environments.

### CI/CD Platform: GitHub Actions

**Why GitHub Actions**:
- Native integration with GitHub repositories
- Free for public repos, generous free tier for private
- Excellent documentation and community support
- Matrix builds for testing multiple configurations
- Rich marketplace of pre-built actions
- Built-in secrets management
- No separate infrastructure to maintain

### Pipeline Architecture

**Three Main Workflows**:
1. **CI Pipeline** (Pull Requests): Build + Lint + Test
2. **CD Pipeline - Staging** (Main branch): Build + Test + Deploy to Staging
3. **CD Pipeline - Production** (Tags): Build + Test + Deploy to Production

**Environment Strategy**:
```
Development (Local) → Feature Branch → Pull Request
                                       ↓ (CI Pipeline)
                                    Main Branch
                                       ↓ (CD Pipeline)
                                    Staging Environment
                                       ↓ (Manual approval)
                                    Production Environment
```

---

## Pipeline Workflows

### 1. CI Pipeline (Pull Requests)

**Trigger**: Pull request opened/updated  
**Purpose**: Verify code quality before merge  
**File**: `.github/workflows/ci.yml`

```yaml
name: CI Pipeline

on:
  pull_request:
    branches: [main, develop]

jobs:
  backend-build-test:
    name: Backend - Build & Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'
          
      - name: Restore dependencies
        run: dotnet restore
        
      - name: Build
        run: dotnet build --configuration Release --no-restore
        
      - name: Run unit tests
        run: dotnet test tests/MyApp.UnitTests --configuration Release --no-build --verbosity normal --logger trx --results-directory ./test-results
        
      - name: Run integration tests
        run: dotnet test tests/MyApp.IntegrationTests --configuration Release --no-build --verbosity normal --logger trx --results-directory ./test-results
        
      - name: Code coverage
        run: dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura /p:CoverageThreshold=70
        
      - name: Publish test results
        uses: dorny/test-reporter@v1
        if: always()
        with:
          name: Backend Test Results
          path: ./test-results/*.trx
          reporter: dotnet-trx
          
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.cobertura.xml
          flags: backend
  
  frontend-build-test:
    name: Frontend - Build & Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22.x'
          cache: 'npm'
          cache-dependency-path: src/web/package-lock.json
          
      - name: Install dependencies
        working-directory: src/web
        run: npm ci
        
      - name: Lint
        working-directory: src/web
        run: npm run lint
        
      - name: Build
        working-directory: src/web
        run: npm run build -- --configuration production
        
      - name: Run unit tests
        working-directory: src/web
        run: npm run test:ci -- --code-coverage
        
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: src/web/coverage/cobertura-coverage.xml
          flags: frontend
  
  docker-build:
    name: Docker - Build Images
    runs-on: ubuntu-latest
    needs: [backend-build-test, frontend-build-test]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        
      - name: Build backend image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./src/MyApp.API/Dockerfile
          push: false
          tags: myapp-api:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          
      - name: Build frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./src/web
          file: ./src/web/Dockerfile
          push: false
          tags: myapp-web:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**Success Criteria**: 
- All tests pass
- Code coverage ≥ 70%
- Build succeeds
- Linting passes

**Result**: PR can be merged or needs fixes

### 2. CD Pipeline - Staging (Main Branch)

**Trigger**: Push to `main` branch  
**Purpose**: Deploy to staging environment for QA  
**File**: `.github/workflows/cd-staging.yml`

```yaml
name: CD Pipeline - Staging

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    name: Build & Push Docker Images
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'
      
      - name: Run tests
        run: dotnet test --configuration Release
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata (tags, labels) for Docker
        id: meta-api
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-api
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            
      - name: Build and push backend image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./src/MyApp.API/Dockerfile
          push: true
          tags: ${{ steps.meta-api.outputs.tags }}
          labels: ${{ steps.meta-api.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Extract metadata for frontend
        id: meta-web
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-web
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            
      - name: Build and push frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./src/web
          file: ./src/web/Dockerfile
          push: true
          tags: ${{ steps.meta-web.outputs.tags }}
          labels: ${{ steps.meta-web.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build-and-push
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Deploy to staging server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /opt/myapp
            docker-compose pull
            docker-compose up -d --remove-orphans
            docker image prune -f
      
      - name: Run database migrations
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            docker exec myapp-api dotnet ef database update --project Infrastructure --startup-project API
      
      - name: Health check
        run: |
          sleep 10
          curl -f https://staging.myapp.com/health || exit 1
  
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: deploy-staging
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22.x'
      
      - name: Install Playwright
        working-directory: src/web
        run: |
          npm ci
          npx playwright install --with-deps
      
      - name: Run E2E tests
        working-directory: src/web
        env:
          BASE_URL: https://staging.myapp.com
        run: npx playwright test
      
      - name: Upload Playwright report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: src/web/playwright-report/
          retention-days: 30
```

**Success Criteria**:
- Build succeeds
- Tests pass
- Deployment successful
- Health check passes
- E2E tests pass on staging

### 3. CD Pipeline - Production (Tags)

**Trigger**: Git tag pushed (e.g., `v1.0.0`)  
**Purpose**: Deploy to production with approval  
**File**: `.github/workflows/cd-production.yml`

```yaml
name: CD Pipeline - Production

on:
  push:
    tags:
      - 'v*.*.*'

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    name: Build & Push Docker Images
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract version
        id: version
        run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
      
      - name: Build and push backend image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./src/MyApp.API/Dockerfile
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-api:${{ steps.version.outputs.VERSION }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-api:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push frontend image
        uses: docker/build-push-action@v5
        with:
          context: ./src/web
          file: ./src/web/Dockerfile
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-web:${{ steps.version.outputs.VERSION }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-web:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build-and-push
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Backup production database
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.PRODUCTION_HOST }}
          username: ${{ secrets.PRODUCTION_USER }}
          key: ${{ secrets.PRODUCTION_SSH_KEY }}
          script: |
            /opt/scripts/backup-database.sh
      
      - name: Deploy to production server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.PRODUCTION_HOST }}
          username: ${{ secrets.PRODUCTION_USER }}
          key: ${{ secrets.PRODUCTION_SSH_KEY }}
          script: |
            cd /opt/myapp
            docker-compose pull
            docker-compose up -d --remove-orphans
            docker image prune -f
      
      - name: Run database migrations
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.PRODUCTION_HOST }}
          username: ${{ secrets.PRODUCTION_USER }}
          key: ${{ secrets.PRODUCTION_SSH_KEY }}
          script: |
            docker exec myapp-api dotnet ef database update --project Infrastructure --startup-project API
      
      - name: Health check
        run: |
          sleep 15
          curl -f https://myapp.com/health || exit 1
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        if: always()
        with:
          status: ${{ job.status }}
          text: 'Production deployment ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

**Success Criteria**:
- Manual approval granted
- Database backup successful
- Deployment successful
- Health check passes
- Team notified

---

## Docker Configuration

### Backend Dockerfile

**File**: `src/MyApp.API/Dockerfile`

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# Copy solution and project files
COPY ["src/MyApp.API/MyApp.API.csproj", "MyApp.API/"]
COPY ["src/MyApp.Application/MyApp.Application.csproj", "MyApp.Application/"]
COPY ["src/MyApp.Core/MyApp.Core.csproj", "MyApp.Core/"]
COPY ["src/MyApp.Infrastructure/MyApp.Infrastructure.csproj", "MyApp.Infrastructure/"]

# Restore dependencies
RUN dotnet restore "MyApp.API/MyApp.API.csproj"

# Copy source code
COPY src/ .

# Build and publish
WORKDIR "/src/MyApp.API"
RUN dotnet publish "MyApp.API.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime
WORKDIR /app

# Create non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser /app
USER appuser

# Copy published app
COPY --from=build /app/publish .

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Entry point
ENTRYPOINT ["dotnet", "MyApp.API.dll"]
```

### Frontend Dockerfile

**File**: `src/web/Dockerfile`

```dockerfile
# Build stage
FROM node:22-alpine AS build
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build Angular app
RUN npm run build -- --configuration production

# Runtime stage
FROM nginx:alpine AS runtime

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Copy built app
COPY --from=build /app/dist/myapp /usr/share/nginx/html

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose

**File**: `docker-compose.yml`

```yaml
version: '3.8'

services:
  api:
    image: ghcr.io/orgname/myapp-api:latest
    container_name: myapp-api
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ConnectionStrings__DefaultConnection=${DB_CONNECTION_STRING}
      - Jwt__SecretKey=${JWT_SECRET}
      - Jwt__Issuer=${JWT_ISSUER}
      - Jwt__Audience=${JWT_AUDIENCE}
    depends_on:
      - db
    networks:
      - myapp-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
  
  web:
    image: ghcr.io/orgname/myapp-web:latest
    container_name: myapp-web
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - api
    networks:
      - myapp-network
    volumes:
      - ./ssl:/etc/nginx/ssl:ro
  
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: myapp-db
    restart: unless-stopped
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=${DB_SA_PASSWORD}
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql
    networks:
      - myapp-network

networks:
  myapp-network:
    driver: bridge

volumes:
  sqldata:
```

---

## Secrets Management

**GitHub Secrets** (configured in repository settings):

**Shared**:
- `GITHUB_TOKEN` (auto-provided)

**Staging**:
- `STAGING_HOST` - Staging server hostname
- `STAGING_USER` - SSH username
- `STAGING_SSH_KEY` - SSH private key
- `STAGING_DB_CONNECTION` - Database connection string

**Production**:
- `PRODUCTION_HOST` - Production server hostname
- `PRODUCTION_USER` - SSH username
- `PRODUCTION_SSH_KEY` - SSH private key
- `PRODUCTION_DB_CONNECTION` - Database connection string
- `JWT_SECRET` - JWT signing key
- `SLACK_WEBHOOK` - Slack notification webhook

**Environment Variables** (in `.env` file on server):
```bash
DB_CONNECTION_STRING=Server=db;Database=MyApp;User=sa;Password=***
JWT_SECRET=your-secret-key-here
JWT_ISSUER=https://myapp.com
JWT_AUDIENCE=https://myapp.com
DB_SA_PASSWORD=strong-password-here
```

---

## Rollback Strategy

**Automatic Rollback**:
- Health check fails after deployment → automatically revert

**Manual Rollback**:
```bash
# SSH into server
ssh user@production-server

# Rollback to previous version
cd /opt/myapp
docker-compose down
docker tag myapp-api:latest myapp-api:rollback
docker pull myapp-api:v1.0.0  # Previous version
docker tag myapp-api:v1.0.0 myapp-api:latest
docker-compose up -d

# Rollback database if needed
dotnet ef database update PreviousMigration
```

**GitHub Actions Manual Rollback**:
- Re-run previous successful production deployment workflow

---

## Consequences

### Positive Consequences
- **Automation**: Eliminates manual deployment errors
- **Speed**: Faster deployment cycles (minutes vs hours)
- **Consistency**: Same process every time
- **Confidence**: Automated testing catches issues early
- **Visibility**: Clear deployment history and status
- **Rollback**: Quick recovery from failed deployments
- **Security**: Secrets managed centrally

### Negative Consequences
- **Initial Setup Time**: Requires upfront configuration
- **Maintenance**: Workflows need updates as project evolves
- **Complexity**: Team needs to understand CI/CD concepts
- **Cost**: CI/CD minutes usage (mitigated by free tier)

### Neutral Considerations
- Need to train team on GitHub Actions
- Require infrastructure for hosting (servers or cloud platform)
- Need monitoring and alerting for production
- Deployment strategy must be communicated to team

---

## Alternatives Considered

### Alternative 1: Azure DevOps
- **Description**: Use Azure DevOps Pipelines for CI/CD
- **Pros**: Excellent Azure integration, robust features, artifact management
- **Cons**: More complex setup, not as GitHub-native, requires separate account
- **Why rejected**: GitHub Actions provides simpler integration for GitHub-hosted projects

### Alternative 2: GitLab CI/CD
- **Description**: Use GitLab's built-in CI/CD
- **Pros**: Excellent CI/CD features, integrated, free tier
- **Cons**: Would require moving repositories to GitLab
- **Why rejected**: Already using GitHub, migration overhead not justified

### Alternative 3: Jenkins
- **Description**: Self-hosted Jenkins server
- **Pros**: Highly customizable, many plugins, free
- **Cons**: Requires server maintenance, more complex setup, older technology
- **Why rejected**: GitHub Actions provides simpler, modern alternative without server management

### Alternative 4: Manual Deployments
- **Description**: Deploy manually via scripts or FTP
- **Pros**: Simple to understand, full control
- **Cons**: Error-prone, slow, not reproducible, doesn't scale
- **Why rejected**: Unacceptable for professional development

---

## Implementation Notes

### Initial Setup Steps

1. **Create GitHub Secrets** in repository settings
2. **Create workflow files** in `.github/workflows/`
3. **Create Dockerfiles** for backend and frontend
4. **Create docker-compose.yml** for deployment
5. **Setup staging/production servers** with Docker installed
6. **Configure environments** in GitHub repository settings
7. **Test CI pipeline** with a pull request
8. **Test CD pipeline** with a main branch push
9. **Configure monitoring** and alerting

### Health Check Endpoint

**Backend** (`/health`):
```csharp
app.MapHealthChecks("/health");

builder.Services.AddHealthChecks()
    .AddDbContextCheck<ApplicationDbContext>();
```

### Environment Configuration

**GitHub Repository Settings** → **Environments**:
- Create `staging` environment (no protection rules)
- Create `production` environment:
  - Required reviewers: 1+
  - Wait timer: 5 minutes
  - Restrict to main branch

---

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Best Practices](https://docs.docker.com/develop/best-practices/)
- [.NET Docker Images](https://hub.docker.com/_/microsoft-dotnet)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [12-Factor App](https://12factor.net/)
- [Continuous Delivery](https://continuousdelivery.com/)
