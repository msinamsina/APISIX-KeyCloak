# APISIX + Keycloak OIDC Gateway Project

A production-ready Dockerized project featuring Apache APISIX as an API Gateway with Keycloak OpenID Connect authentication.

---

## 📐 Architecture Diagram

```
                                    EXTERNAL NETWORK
    ┌─────────────────────────────────────────────────────────────────────────────┐
    │                                                                             │
    │   ┌─────────────┐                                                           │
    │   │   Browser   │                                                           │
    │   │   Client    │                                                           │
    │   └──────┬──────┘                                                           │
    │          │                                                                  │
    │          │ HTTP Requests                                                    │
    │          │                                                                  │
    └──────────┼──────────────────────────────────────────────────────────────────┘
               │
               ▼
    ┌──────────────────────────────────────────────────────────────────────────────┐
    │  PORT 9080 (HTTP) / 9443 (HTTPS)                                             │
    │  ┌────────────────────────────────────────────────────────────────────────┐  │
    │  │                        APACHE APISIX GATEWAY                           │  │
    │  │                         (Standalone Mode)                              │  │
    │  │                                                                        │  │
    │  │   ┌──────────────────────────────────────────────────────────────┐    │  │
    │  │   │                      ENABLED PLUGINS                          │    │  │
    │  │   │  • openid-connect    (OIDC Authentication)                   │    │  │
    │  │   │  • proxy-rewrite     (URL/Header Rewriting)                  │    │  │
    │  │   │  • consumer-restriction (Access Control)                     │    │  │
    │  │   └──────────────────────────────────────────────────────────────┘    │  │
    │  │                                                                        │  │
    │  │   ROUTES:                                                              │  │
    │  │   /auth/*     ──────────────────────▶  Keycloak (Admin/Login)         │  │
    │  │   /realms/*   ──────────────────────▶  Keycloak (OIDC Endpoints)      │  │
    │  │   /test/*     ──[OIDC Protected]───▶  Test Service (Nginx)            │  │
    │  │   /resources/*────────────────────▶  Keycloak (Static Assets)         │  │
    │  │                                                                        │  │
    │  └────────────────────────────────────────────────────────────────────────┘  │
    └──────────────────────────────────────────────────────────────────────────────┘
               │
               │ INTERNAL NETWORK (apisix-network)
               ▼
    ┌──────────────────────────────────────────────────────────────────────────────┐
    │                           INTERNAL SERVICES                                   │
    │                     (NOT exposed to host network)                            │
    │                                                                              │
    │   ┌─────────────────────┐    ┌─────────────────────┐    ┌────────────────┐  │
    │   │      KEYCLOAK       │    │    TEST SERVICE     │    │   POSTGRES     │  │
    │   │   (Port 8080)       │    │   (Port 80)         │    │  (Port 5432)   │  │
    │   │                     │    │                     │    │                │  │
    │   │  ┌───────────────┐  │    │  ┌───────────────┐  │    │  ┌──────────┐  │  │
    │   │  │ Realm:        │  │    │  │    Nginx      │  │    │  │ keycloak │  │  │
    │   │  │ apisix-demo   │  │    │  │  Static Site  │  │    │  │    DB    │  │  │
    │   │  ├───────────────┤  │    │  └───────────────┘  │    │  └──────────┘  │  │
    │   │  │ Client:       │  │    │                     │    │                │  │
    │   │  │ apisix-client │  │    │  ❌ NOT EXPOSED     │    │  ❌ NOT EXPOSED │  │
    │   │  ├───────────────┤  │    │     TO HOST         │    │     TO HOST    │  │
    │   │  │ Users:        │  │    │                     │    │                │  │
    │   │  │ • testuser    │  │    └─────────────────────┘    └────────────────┘  │
    │   │  │ • adminuser   │  │                                                    │
    │   │  └───────────────┘  │                                                    │
    │   │                     │                                                    │
    │   │  ❌ NOT EXPOSED     │                                                    │
    │   │     TO HOST         │                                                    │
    │   └─────────────────────┘                                                    │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- Docker Engine 20.10+
- Docker Compose v2.0+
- 4GB+ available RAM
- Ports 9080, 9443 available

### Start the Project

```bash
# Navigate to the project directory
cd project

# Start all services
docker-compose up -d

# Watch the logs (optional)
docker-compose logs -f

# Wait for all services to be healthy (~90-120 seconds for Keycloak)
docker-compose ps
```

### Verify Services Are Running

```bash
# Check all containers are healthy
docker-compose ps

# Expected output:
# NAME           STATUS                   PORTS
# apisix         Up (healthy)             0.0.0.0:9080->9080/tcp, ...
# keycloak       Up (healthy)             8080/tcp (internal only)
# postgres       Up (healthy)             5432/tcp (internal only)
# test-service   Up (healthy)             80/tcp (internal only)
```

---

## 🔗 Access Points

| Service | URL | Description |
|---------|-----|-------------|
| **APISIX Gateway** | http://localhost:9080 | Main API Gateway entry point |
| **Keycloak (via APISIX)** | http://localhost:9080/auth | Keycloak admin console |
| **Protected Service** | http://localhost:9080/test | OIDC-protected test service |
| **OIDC Discovery** | http://localhost:9080/realms/apisix-demo/.well-known/openid-configuration | OIDC metadata |

---

## 🔐 Default Credentials

### Keycloak Admin Console
- **URL:** http://localhost:9080/auth/admin
- **Username:** `admin`
- **Password:** `admin`

### Test Users (for OIDC login)

| Username | Password | Roles |
|----------|----------|-------|
| `testuser` | `testpassword` | user, api-access |
| `adminuser` | `adminpassword` | admin, user, api-access |

---

## 🧪 Testing the OIDC Flow

### Step 1: Access the Protected Service

Open your browser and navigate to:

```
http://localhost:9080/test
```

You will be automatically redirected to the Keycloak login page.

### Step 2: Login with Test User

Enter the credentials:
- **Username:** `testuser`
- **Password:** `testpassword`

### Step 3: Access Granted

After successful authentication, you will be redirected back to the protected test service page showing:
- Authentication success message
- Architecture flow diagram
- Available endpoints

### Step 4: Verify Session

Open a new browser tab and access http://localhost:9080/test again. You should be directly served without re-authentication (session is maintained).

### Step 5: Test Logout

Navigate to:
```
http://localhost:9080/test/logout
```

This will end your session. Accessing `/test` again will require re-authentication.

---

## 📋 Testing Commands

### Test OIDC Discovery Endpoint

```bash
curl -s http://localhost:9080/realms/apisix-demo/.well-known/openid-configuration | jq .
```

### Test Protected Service (Without Auth)

```bash
# This should redirect to Keycloak login (302 redirect)
curl -I http://localhost:9080/test

# Expected: HTTP/1.1 302 Moved Temporarily
# Location: http://localhost:9080/realms/apisix-demo/protocol/openid-connect/auth?...
```

### Test Keycloak Access via APISIX

```bash
# Access Keycloak through APISIX
curl -I http://localhost:9080/auth/

# Expected: HTTP/1.1 200 OK
```

### Get Access Token (Direct Grant for Testing)

```bash
# Get access token using password grant
curl -s -X POST \
  "http://localhost:9080/realms/apisix-demo/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=apisix-client" \
  -d "client_secret=apisix-client-secret" \
  -d "username=testuser" \
  -d "password=testpassword" \
  -d "grant_type=password" | jq .
```

### Verify Keycloak is NOT Directly Exposed

```bash
# This should FAIL (connection refused)
curl http://localhost:8080
# Expected: curl: (7) Failed to connect to localhost port 8080

# This should WORK (through APISIX)
curl http://localhost:9080/auth/
# Expected: Keycloak welcome page or redirect
```

---

## 📁 Project Structure

```
project/
├── docker-compose.yml          # Main Docker Compose file
├── README.md                   # This documentation
│
├── apisix/
│   ├── config.yaml             # APISIX main configuration (standalone mode)
│   └── apisix.yaml             # Routes and upstreams configuration
│
├── keycloak/
│   └── realm-export.json       # Pre-configured realm with users & client
│
└── test-service/
    ├── default.conf            # Nginx configuration
    └── index.html              # Protected service frontend
```

---

## ⚙️ Configuration Details

### APISIX Plugins Used

| Plugin | Purpose |
|--------|---------|
| `openid-connect` | OIDC authentication with Keycloak |
| `proxy-rewrite` | URL and header rewriting |
| `consumer-restriction` | Access control based on consumers |

### Routes Configuration

| Route | Path | Auth | Backend |
|-------|------|------|---------|
| keycloak_auth | `/auth`, `/auth/*` | None | Keycloak (8080) |
| keycloak_realms | `/realms/*` | None | Keycloak (8080) |
| keycloak_resources | `/resources/*` | None | Keycloak (8080) |
| keycloak_js | `/js/*` | None | Keycloak (8080) |
| test_service | `/test`, `/test/*` | **OIDC** | Nginx (80) |

### Keycloak Configuration

- **Realm:** `apisix-demo`
- **Client ID:** `apisix-client`
- **Client Secret:** `apisix-client-secret`
- **Redirect URIs:** `http://localhost:9080/*`

---

## 🔧 Customization

### Change Keycloak Client Secret

1. Edit `keycloak/realm-export.json`:
   ```json
   "secret": "your-new-secret"
   ```

2. Edit `apisix/apisix.yaml`:
   ```yaml
   client_secret: "your-new-secret"
   ```

3. Restart services:
   ```bash
   docker-compose down && docker-compose up -d
   ```

### Add New Users

Edit `keycloak/realm-export.json` and add a new user object in the `users` array:

```json
{
  "username": "newuser",
  "enabled": true,
  "email": "newuser@example.com",
  "credentials": [{
    "type": "password",
    "value": "newpassword",
    "temporary": false
  }],
  "realmRoles": ["user"]
}
```

### Add New Protected Routes

Edit `apisix/apisix.yaml` and add a new route with the `openid-connect` plugin:

```yaml
- id: 8
  uri: /my-service/*
  upstream:
    type: roundrobin
    nodes:
      "my-service:80": 1
  plugins:
    openid-connect:
      discovery: "http://keycloak:8080/realms/apisix-demo/.well-known/openid-configuration"
      client_id: "apisix-client"
      client_secret: "apisix-client-secret"
      redirect_uri: "http://localhost:9080/my-service/callback"
      scope: "openid profile email"
      bearer_only: false
      ssl_verify: false
```

---

## 🛠️ Troubleshooting

### Services Not Starting

```bash
# Check container logs
docker-compose logs keycloak
docker-compose logs apisix

# Restart all services
docker-compose down -v
docker-compose up -d
```

### Keycloak Health Check Failing

Keycloak takes ~90-120 seconds to fully initialize. Wait and check again:

```bash
# Watch logs until "Keycloak ... started"
docker-compose logs -f keycloak
```

### OIDC Redirect Not Working

1. Clear browser cookies for `localhost`
2. Verify Keycloak is accessible: `curl http://localhost:9080/auth/`
3. Check APISIX logs: `docker-compose logs apisix`

### "Invalid redirect_uri" Error

Ensure the redirect URI in your request matches exactly what's configured in Keycloak:
- `http://localhost:9080/test/callback`
- `http://localhost:9080/*`

### APISIX Shows "Unhealthy"

The healthcheck may take a few attempts. Check if APISIX is actually responding:
```bash
curl http://localhost:9080/auth/
```
If this works, APISIX is functioning correctly.

---

## 🧹 Cleanup

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (full reset)
docker-compose down -v

# Remove all related images (optional)
docker rmi apache/apisix:3.8.0-debian
docker rmi quay.io/keycloak/keycloak:23.0.3
docker rmi postgres:15-alpine
docker rmi nginx:alpine
```

---

## 📊 OIDC Flow Diagram

```
┌────────┐                              ┌─────────┐                              ┌──────────┐
│Browser │                              │ APISIX  │                              │ Keycloak │
└───┬────┘                              └────┬────┘                              └────┬─────┘
    │                                        │                                        │
    │ 1. GET /test                           │                                        │
    │───────────────────────────────────────▶│                                        │
    │                                        │                                        │
    │ 2. 302 Redirect to Keycloak            │                                        │
    │◀───────────────────────────────────────│                                        │
    │                                        │                                        │
    │ 3. GET /realms/apisix-demo/.../auth    │                                        │
    │───────────────────────────────────────▶│───────────────────────────────────────▶│
    │                                        │                                        │
    │ 4. Login Page                          │                                        │
    │◀───────────────────────────────────────│◀───────────────────────────────────────│
    │                                        │                                        │
    │ 5. POST credentials                    │                                        │
    │───────────────────────────────────────▶│───────────────────────────────────────▶│
    │                                        │                                        │
    │ 6. 302 Redirect with auth code         │                                        │
    │◀───────────────────────────────────────│◀───────────────────────────────────────│
    │                                        │                                        │
    │ 7. GET /test/callback?code=xxx         │                                        │
    │───────────────────────────────────────▶│                                        │
    │                                        │ 8. Exchange code for tokens            │
    │                                        │───────────────────────────────────────▶│
    │                                        │                                        │
    │                                        │ 9. Access + ID tokens                  │
    │                                        │◀───────────────────────────────────────│
    │                                        │                                        │
    │ 10. Set session cookie                 │                                        │
    │◀───────────────────────────────────────│                                        │
    │                                        │                                        │
    │ 11. Redirect to /test                  │                                        │
    │───────────────────────────────────────▶│                                        │
    │                                        │ 12. Validate session                   │
    │                                        │                                        │
    │ 13. Proxy to test-service              │──────────────────┐                     │
    │                                        │                  │                     │
    │                                        │◀─────────────────┘                     │
    │ 14. Protected content                  │                                        │
    │◀───────────────────────────────────────│                                        │
    │                                        │                                        │
```

---

## 🔒 Security Notes

1. **Keycloak is NOT directly exposed** - All access goes through APISIX
2. **PostgreSQL is NOT directly exposed** - Only accessible within Docker network
3. **Test service is NOT directly exposed** - Only accessible through APISIX after authentication
4. **Change default credentials in production** - Update all passwords and secrets
5. **Enable HTTPS in production** - Configure SSL certificates for port 9443

---

## 📜 License

This project is provided as-is for educational and demonstration purposes.

---

## 🤝 Support

For issues related to:
- **APISIX:** https://github.com/apache/apisix/issues
- **Keycloak:** https://github.com/keycloak/keycloak/issues
- **Docker:** https://docs.docker.com/

---

**Happy Coding! 🎉**
