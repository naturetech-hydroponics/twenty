# Deploying Twenty CRM from GitHub to Coolify

## Important: Use Docker Compose, Not Dockerfile

While Twenty CRM has Dockerfiles in the repository, **you should use Docker Compose** for deployment because Twenty requires multiple services:

- PostgreSQL database
- Redis cache
- Twenty server
- Twenty worker

## Recommended Approach: Docker Compose from GitHub

### Step 1: In Coolify, Create New Resource

1. Click **"+ New Resource"**
2. Select **"Docker Compose"**
3. Choose **"From Git"**

### Step 2: Connect Your GitHub Repository

1. Select your forked Twenty repository
2. Branch: `main` (or your branch name)
3. **Docker Compose Location**: `/docker-compose.yaml`
4. **Base Directory**: `/` (root)

![Configuration Example](file:///C:/Users/Carpediem/.gemini/antigravity/brain/e1658dc7-6a93-469b-bf57-2caeb2dcae7f/uploaded_image_1768172602537.png)

### Step 3: Configure Build Settings

Since you're deploying from GitHub:

- **Build Pack**: Docker Compose *(not Dockerfile)*
- **Watch Paths**: Leave default or set to `/` to rebuild on any changes

### Step 4: Set Environment Variables

In Coolify's Environment tab, add:

```bash
# Required
DOMAIN=your-domain.com

# Optional (Coolify auto-generates these)
# SERVICE_FQDN_SERVER - auto-populated
# SERVICE_PASSWORD_POSTGRES - auto-populated
# SERVICE_BASE64_32_SECRET - auto-populated

# Version (optional)
TWENTY_VERSION=latest
```

### Step 5: Configure Domain

1. Go to **"Domains"** tab
2. Add your domain (e.g., `crm.yourdomain.com`)
3. Ensure DNS A record points to Coolify server

### Step 6: Deploy

Click **"Deploy"** and monitor the logs.

---

## Alternative: If You Must Use Dockerfile

If you want to deploy using the Dockerfile approach (not recommended for production), you'll need to:

### 1. Deploy Database Separately

Create a PostgreSQL database in Coolify:
- Go to **"+ New Resource"** → **"Database"** → **"PostgreSQL"**
- Note the connection details

### 2. Deploy Redis Separately

Create a Redis instance in Coolify:
- Go to **"+ New Resource"** → **"Database"** → **"Redis"**
- Note the connection URL

### 3. Deploy Twenty Server

In your GitHub deployment settings:

**Build Pack**: Dockerfile
**Dockerfile Location**: `/packages/twenty-docker/twenty/Dockerfile`
**Base Directory**: `/`

**Environment Variables**:
```bash
# Database (from your separate PostgreSQL)
PG_DATABASE_URL=postgres://user:password@postgres-host:5432/dbname

# Redis (from your separate Redis)
REDIS_URL=redis://redis-host:6379

# URLs
FRONTEND_URL=https://your-domain.com
SERVER_URL=https://your-domain.com

# Security
APP_SECRET=your-generated-secret-here
SIGN_IN_PREFILLED=true

# Storage
STORAGE_TYPE=local

# Port
NODE_PORT=3000
```

### 4. Deploy Worker Separately

Deploy another instance with:
- Same Dockerfile
- **Command Override**: `yarn worker:prod`
- Same environment variables as server
- Add: `DISABLE_DB_MIGRATIONS=true`

---

## Why Docker Compose is Better

| Aspect | Docker Compose | Dockerfile |
|--------|----------------|------------|
| **Setup Complexity** | Simple - one deployment | Complex - 4 separate deployments |
| **Service Coordination** | Automatic | Manual networking required |
| **Health Checks** | Built-in dependencies | Manual configuration |
| **Maintenance** | Single deployment to manage | 4 deployments to manage |
| **Environment Variables** | Shared automatically | Must duplicate across services |
| **Recommended** | ✅ Yes | ❌ No |

---

## Troubleshooting GitHub Deployment

### Issue: Build Fails

**Check**:
1. Correct branch selected
2. `docker-compose.yaml` path is correct: `/docker-compose.yaml`
3. Repository is accessible by Coolify

### Issue: "No docker-compose.yaml found"

**Solution**:
- Verify the file exists in repository root
- Check file name is exactly `docker-compose.yaml` (not `.yml`)
- Ensure Base Directory is `/`

### Issue: Services Start But Can't Connect

**Check**:
1. All environment variables are set
2. Magic variables are populated by Coolify
3. Database is healthy before server starts
4. Network connectivity between services

---

## Quick Fix for Your Current Setup

Based on your screenshot, change:

1. **Build Pack**: Change from "Dockerfile" to **"Docker Compose"**
2. **Docker Compose Location**: Set to `/docker-compose.yaml`
3. **Base Directory**: Keep as `/`

Then click **"Continue"** and proceed with environment variable configuration.
