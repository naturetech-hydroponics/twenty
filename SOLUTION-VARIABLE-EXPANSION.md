# ✅ SOLUTION: Fixed Coolify Variable Expansion Issue

## 🎯 Root Cause Identified

The server was failing with this error:
```
FRONTEND_URL must be a URL address
SERVER_URL must be a URL address
```

**Problem**: The docker-compose.yaml was using variable substitution syntax like:
```yaml
- FRONTEND_URL=$SERVICE_FQDN_SERVER
- SERVER_URL=$SERVICE_FQDN_SERVER
- PG_DATABASE_URL=postgres://postgres:$SERVICE_PASSWORD_POSTGRES@db:5432/default
```

Coolify was **NOT expanding these variables** - the server was receiving the literal string `$SERVICE_FQDN_SERVER` instead of the actual URL `https://twenty.naturetech.co.il`.

## ✅ Solution Applied

Modified `docker-compose.yaml` to use **environment variable passthrough** instead of substitution:

### Changed From:
```yaml
- FRONTEND_URL=$SERVICE_FQDN_SERVER
- SERVER_URL=$SERVICE_FQDN_SERVER
- PG_DATABASE_URL=postgres://postgres:$SERVICE_PASSWORD_POSTGRES@db:5432/default
```

### Changed To:
```yaml
- FRONTEND_URL
- SERVER_URL
- PG_DATABASE_URL
```

This tells Docker Compose to pass through the environment variables that Coolify sets, rather than trying to substitute them.

## 📋 Next Steps

1. **Commit the updated `docker-compose.yaml`** to your GitHub repository
2. **Redeploy in Coolify** - it will pull the updated docker-compose.yaml
3. **Monitor the deployment** - the server should now start successfully!

## 🔧 What Changed in docker-compose.yaml

**Server service** (lines 29-35):
- `FRONTEND_URL` - now uses passthrough
- `SERVER_URL` - now uses passthrough
- `PG_DATABASE_URL` - now uses passthrough

**Worker service** (lines 107-112):
- `PG_DATABASE_URL` - now uses passthrough
- `SERVER_URL` - now uses passthrough

## ✅ Your Environment Variables (Keep These in Coolify)

```bash
SERVICE_BASE64_32_SECRET=oXJVUL95veaun7yUebqLn9A3LTocCUga
SERVICE_FQDN_SERVER=https://twenty.naturetech.co.il
SERVICE_PASSWORD_POSTGRES=FfhKfjrp0cBRcSbE3pYzihYcDiNhiRDe
DOMAIN=twenty.naturetech.co.il
FRONTEND_URL=https://twenty.naturetech.co.il
SERVER_URL=https://twenty.naturetech.co.il
PG_DATABASE_URL=postgres://postgres:FfhKfjrp0cBRcSbE3pYzihYcDiNhiRDe@db:5432/default
APP_SECRET=oXJVUL95veaun7yUebqLn9A3LTocCUga
STORAGE_TYPE=local
# ... (other optional variables)
```

These will now be properly passed to the containers!

## 🚀 Expected Result

After redeploying:
1. ✅ Database will start and become healthy
2. ✅ Redis will start
3. ✅ Server will receive proper URLs
4. ✅ Server will pass health check
5. ✅ Worker will start
6. ✅ You can access https://twenty.naturetech.co.il

## 📝 Why This Happened

Docker Compose has two ways to handle environment variables:

1. **Substitution** (doesn't work in Coolify):
   ```yaml
   - FRONTEND_URL=$SERVICE_FQDN_SERVER
   ```
   Docker Compose tries to substitute `$SERVICE_FQDN_SERVER` at compose time, but Coolify sets these as runtime environment variables.

2. **Passthrough** (works in Coolify):
   ```yaml
   - FRONTEND_URL
   ```
   Docker Compose passes the environment variable from the host to the container without modification.

Coolify expects passthrough syntax, not substitution syntax!
