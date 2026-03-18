# Service Registry Service

A **service discovery server** built on **Netflix Eureka**. It acts as the central directory for all microservices in the NexaShopping architecture, enabling dynamic service discovery, load balancing, and inter-service communication without hardcoded URLs.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Dependencies](#dependencies)
- [Configuration](#configuration)
- [Service Registration](#service-registration)
- [Service Discovery](#service-discovery)
- [Setup & Installation](#setup--installation)
- [Building & Running](#building--running)
- [Eureka Dashboard](#eureka-dashboard)
- [Management Endpoints](#management-endpoints)
- [Monitoring & Health Checks](#monitoring--health-checks)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

The **Service Registry** is the cornerstone of service discovery in the NexaShopping microservices ecosystem. It:

- ✅ Maintains a **dynamic registry** of all available microservices
- ✅ Enables **automatic service discovery** by name rather than hardcoded IPs
- ✅ Provides **load balancing** across multiple instances of the same service
- ✅ Detects **service failures** via periodic health checks
- ✅ Deregisters **unhealthy services** automatically
- ✅ Provides a **web dashboard** for viewing registered services
- ✅ Supports **client-side load balancing** via cloud-aware clients
- ✅ Built on **Netflix Eureka** - battle-tested service discovery solution

**Application Name:** `service-registry`  
**Default Port:** `9001`  
**Artifact ID:** `Service-Registry`  
**Group ID:** `lk.ijse.eca`

---

## 🏗️ Architecture

The Service Registry sits at the heart of the microservices discovery ecosystem:

```
┌──────────────────────────────────────────────────┐
│   Netflix Eureka Service Registry                │
│   (Port 9001)                                    │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ Service Registry Database                  │ │
│  │ - user-service (8001)                      │ │
│  │ - item-service (8002)                      │ │
│  │ - order-service (8003)                     │ │
│  │ - api-gateway (7000) [client of registry]  │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ Eureka Web Dashboard                       │ │
│  │ (UI for monitoring)                        │ │
│  └────────────────────────────────────────────┘ │
└──────┬───────────────────────────────────────────┘
       │ HTTP/REST
       │
   ┌───┴─────┬──────────┬──────────┐
   │         │          │          │
   ▼         ▼          ▼          ▼
┌─────┐ ┌──────┐ ┌──────┐ ┌──────┐
│API  │ │User  │ │Item  │ │Order │
│Gate │ │Svc   │ │Svc   │ │Svc   │
│way  │ └──────┘ └──────┘ └──────┘
└─────┘
(All register & query the registry)
```

---

## 🛠️ Tech Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| **Java** | JDK 21 | Language runtime |
| **Spring Boot** | 4.0.3 | Application framework |
| **Spring Cloud** | 2025.1.0 | Microservices framework |
| **Netflix Eureka Server** | Latest (2025.1.0) | Service discovery engine |
| **Spring Boot WebMVC** | Latest | Traditional servlet-based web framework |
| **Spring Cloud Config Client** | Latest | Fetches config from Config-Server |
| **Spring Boot DevTools** | Latest (runtime) | Hot reload during development |

---

## 📁 Project Structure

```
service-registry/
├── src/
│   ├── main/
│   │   ├── java/lk/ijse/eca/serviceregistry/
│   │   │   └── ServiceRegistryApplication.java   # Main Spring Boot entry point
│   │   └── resources/
│   │       └── application.yaml                  # Eureka Server configuration
│   └── test/
├── pom.xml                                        # Maven configuration
├── mvnw / mvnw.cmd                              # Maven wrapper scripts
└── README.md                                      # This file
```

### Key Java Classes

- **ServiceRegistryApplication.java**: Spring Boot main application class
  - Annotated with `@EnableEurekaServer` - activates Eureka server functionality
  - Bootstraps on port 9001

---

## 📦 Dependencies

### Core Dependencies

1. **spring-cloud-starter-netflix-eureka-server**
   - Core Netflix Eureka Server functionality
   - Service registration and discovery engine
   - Health check mechanisms
   - Eureka dashboard UI
   - In-memory service registry database

2. **spring-boot-starter-webmvc**
   - Traditional servlet-based web framework
   - Serves HTTP endpoints and dashboard
   - Request handling for registration/discovery APIs

3. **spring-cloud-starter-config**
   - Client for fetching configuration from Config-Server
   - Allows externalized Eureka server settings
   - Configuration URI: `http://localhost:9000`

4. **spring-boot-devtools** (optional, runtime only)
   - Enables hot reload during development
   - Useful for rapid testing of Eureka configuration

---

## ⚙️ Configuration

### application.yaml

```yaml
spring:
  application:
    name: service-registry              # Service name registered with itself

  config:
    import: "configserver:"             # Enable Config-Server integration

  cloud:
    config:
      uri: http://localhost:9000        # Config-Server location
```

### Eureka Server Configuration

The actual Eureka server configuration is typically provided by Config-Server. Here are common properties:

```yaml
eureka:
  instance:
    hostname: localhost                 # Eureka server hostname
  
  server:
    enable-self-preservation: true      # Keep instances if heartbeats missing
    eviction-interval-timer-in-ms: 60000 # Clean up interval (1 min)
    
  client:
    fetch-registry: false               # Registry doesn't fetch from others
    register-with-eureka: false         # Doesn't register with itself
    service-url:
      defaultZone: http://localhost:9001/eureka/  # Eureka endpoint

server:
  port: 9001                            # Eureka runs on port 9001
```

---

## 📝 Service Registration

### How Services Register

When any microservice (user-service, item-service, order-service, api-gateway) starts:

1. **Service reads configuration** from Config-Server
2. **Service finds Eureka location** (from config: `http://localhost:9001/eureka`)
3. **Service sends registration request** with:
   - Service name (e.g., `USER-SERVICE`)
   - Instance IP address and port
   - Health check URL
   - Instance metadata
4. **Eureka stores registration** in registry database
5. **Service sends periodic heartbeats** to stay registered

### Service Registration Example

**User Service Registration:**
```
Service Name: USER-SERVICE
Instance: 192.168.1.100:8001
Health Check: http://192.168.1.100:8001/actuator/health
Status: UP
```

**Item Service Registration:**
```
Service Name: ITEM-SERVICE
Instance: 192.168.1.100:8002
Health Check: http://192.168.1.100:8002/actuator/health
Status: UP
```

---

## 🔍 Service Discovery

### How Services Are Discovered

When API Gateway or another service needs to call a microservice:

1. **Service queries Eureka** for service location
2. **Eureka returns list** of healthy instances
3. **Client picks one instance** (round-robin load balancing)
4. **Service makes request** to the chosen instance

### Discovery Example

**API Gateway Route Setup:**
```yaml
# API Gateway routes to USER-SERVICE by name
routes:
  - id: user-service-route
    uri: lb://USER-SERVICE           # lb:// = Eureka lookup
    predicates:
      - Path=/api/v1/users/**
```

**When request comes:**
```
GET /api/v1/users
  ↓
API Gateway queries: "Where is USER-SERVICE?"
  ↓
Eureka responds: "8001, 8002, 8003 (3 instances)"
  ↓
Gateway picks one: 8001
  ↓
Request goes to: http://localhost:8001/api/v1/users
```

---

## 🚀 Setup & Installation

### Prerequisites

- **Java 21** (or compatible version)
- **Maven 3.6+** (or use the included `mvnw` wrapper)
- **Config-Server running** (port 9000) - must start first!

### Installation Steps

1. **Clone the repository** (if not already done)
   ```bash
   git clone <repository-url>
   cd NexaShopping-Project/platform/service-registry
   ```

2. **Verify Java version**
   ```bash
   java -version
   # Should output Java 21 or compatible
   ```

3. **Ensure Config-Server is running**
   ```bash
   # In another terminal
   cd ../config-server
   ./mvnw spring-boot:run
   # Wait for "Started ConfigServerApplication"
   ```

4. **Build the project**
   ```bash
   ./mvnw clean install
   ```

5. **Verify dependencies**
   ```bash
   ./mvnw dependency:tree
   ```

---

## ▶️ Building & Running

### Option 1: Using Maven (Recommended)

**Build:**
```bash
./mvnw clean package
```

**Run:**
```bash
./mvnw spring-boot:run
```

### Option 2: Run JAR Directly

**After building:**
```bash
./mvnw clean package
java -jar target/Service-Registry-1.0.0.jar
```

### Option 3: IDE Run (IntelliJ/Eclipse)

1. Right-click `ServiceRegistryApplication.java`
2. Select **Run** or **Debug**

### Startup Sequence

**Critical: Services must start in this order:**

1. **Config-Server** (port 9000) ⭐ **START FIRST**
   ```bash
   cd ../config-server
   ./mvnw spring-boot:run
   ```

2. **Service-Registry** (port 9001) ⭐ **START SECOND**
   ```bash
   cd ../service-registry
   ./mvnw spring-boot:run
   ```

3. **API-Gateway** (port 7000)
   ```bash
   cd ../api-gateway
   ./mvnw spring-boot:run
   ```

4. **Microservices** (user-service, item-service, order-service)
   ```bash
   cd ../../services/<service-name>
   ./mvnw spring-boot:run
   ```

---

## 🎨 Eureka Dashboard

### Accessing the Dashboard

Once the Service Registry is running, access the Eureka dashboard:

```
http://localhost:9001
```

### Dashboard Layout

The dashboard shows:

1. **General Information**
   - System Status (UP/DOWN)
   - Eureka Environment
   - Uptime

2. **Instances Currently Registered with Eureka**
   - Service names and instances
   - Status (UP, DOWN, OUT_OF_SERVICE)
   - Instance count per service
   - Last heartbeat times

3. **DS Replicas**
   - Other Eureka servers (if clustered)
   - Replication status

### Example Dashboard View

```
═══════════════════════════════════════════════════════════════
                    Eureka Services
═══════════════════════════════════════════════════════════════

Application: USER-SERVICE
├── Instance: user-service:8001
│   ├── Status: UP ✅
│   ├── Last Heartbeat: 2 seconds ago
│   └── Metadata: version=1.0.0

Application: ITEM-SERVICE
├── Instance: item-service:8002
│   ├── Status: UP ✅
│   ├── Last Heartbeat: 1 second ago
│   └── Metadata: version=1.0.0

Application: ORDER-SERVICE
├── Instance: order-service:8003
│   ├── Status: UP ✅
│   ├── Last Heartbeat: 3 seconds ago
│   └── Metadata: version=1.0.0

Application: API-GATEWAY
├── Instance: api-gateway:7000
│   ├── Status: UP ✅
│   ├── Last Heartbeat: 1 second ago
│   └── Metadata: version=1.0.0
```

---

## 🔧 Management Endpoints

### Eureka Server Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/eureka/web` | GET | Dashboard UI |
| `/eureka/apps` | GET | List all registered applications (JSON) |
| `/eureka/apps/{app-name}` | GET | Get specific application details |
| `/eureka/apps/{app-name}/{instance-id}` | GET | Get specific instance details |

### Example API Calls

**1. Get all registered applications:**
```bash
curl http://localhost:9001/eureka/apps
```

**Response:**
```json
{
  "applications": {
    "application": [
      {
        "name": "USER-SERVICE",
        "instance": [
          {
            "instanceId": "user-service:8001",
            "hostName": "localhost",
            "app": "USER-SERVICE",
            "ipAddr": "192.168.1.100",
            "status": "UP",
            "port": {"$": 8001, "@enabled": true},
            "lastUpdatedTimestamp": 1710850123456
          }
        ]
      }
    ]
  }
}
```

**2. Get specific service (USER-SERVICE):**
```bash
curl http://localhost:9001/eureka/apps/USER-SERVICE
```

**3. Get instance details:**
```bash
curl http://localhost:9001/eureka/apps/USER-SERVICE/user-service:8001
```

---

## 🏥 Monitoring & Health Checks

### Eureka Health Check Mechanism

1. **Service sends heartbeat** every 30 seconds (default)
2. **If 90 seconds pass without heartbeat** - service marked as DOWN
3. **If 180 seconds pass** - service removed from registry (evicted)
4. **On graceful shutdown** - service deregisters immediately

### Health Check Configuration

Services typically expose health endpoints:

```yaml
# In each microservice
management:
  endpoints:
    web:
      exposure:
        include: health
  endpoint:
    health:
      show-details: always
```

### Monitoring Dashboard Metrics

The Eureka dashboard shows:

| Metric | Meaning |
|--------|---------|
| **Renews (last min)** | Heartbeats received in last minute |
| **Renews threshold** | Expected heartbeats |
| **Available replicas** | Other Eureka servers replicating |
| **Connected replicas** | Successfully replicated servers |

---

## 🐛 Troubleshooting

### Issue 1: Port Already in Use

**Error:** `Address already in use: 9001`

**Solution:**
```bash
# Find process using port 9001
netstat -ano | findstr :9001

# Kill the process
taskkill /PID <PID> /F

# Or change port in config
eureka:
  server:
    port: 9010
```

### Issue 2: Config-Server Not Available

**Error:** `Unable to connect to Config Server`

**Solution:**
1. Ensure Config-Server is running on port 9000
2. Check connectivity:
   ```bash
   curl http://localhost:9000/health
   ```
3. Verify `application.yaml` has correct URI
4. Add these to application.yaml if needed:
   ```yaml
   spring:
     cloud:
       config:
         fail-fast: false  # Don't fail if config server unavailable
   ```

### Issue 3: Services Not Registering

**Error:** Dashboard shows no services

**Solutions:**
1. Verify services are running
2. Check service configuration for Eureka URL:
   ```yaml
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:9001/eureka/
   ```
3. Verify service names in logs (should have `Discovered eureka service...`)
4. Check firewall isn't blocking port 9001

### Issue 4: High Renewal Threshold Warning

**Warning:** "Self preservation mode enabled"

**Meaning:** Many services lost heartbeat connection (possible network issue)

**Solutions:**
1. Check network connectivity
2. Verify services are healthy: `curl http://localhost:<SERVICE_PORT>/actuator/health`
3. Check if services are experiencing high load
4. Increase heartbeat interval settings

### Common Log Messages

| Log | Meaning | Action |
|-----|---------|--------|
| `Registered instance with Eureka` | ✅ Healthy | Service registered successfully |
| `Renewing lease from...` | ✅ Healthy | Service sending heartbeats |
| `Evicting instance...` | ⚠️ Warning | Service removed (no heartbeats) |
| `EMERGENCY! EUREKA server...` | ⚠️ Warning | Self-preservation mode (expected in tests) |

---

## 🔐 Security Considerations

1. **Network Isolation**
   - Service Registry should only be accessible from internal network
   - Restrict access via firewall
   - Don't expose to internet

2. **Authentication** (Advanced)
   - Can be configured with Spring Security
   - Use OAuth2 for service-to-service authentication
   - Implement API authentication if exposing externally

3. **Data Privacy**
   - Service registry contains service locations
   - Protect from unauthorized access
   - Monitor access logs

---

## 📊 Performance Characteristics

- **Startup Time**: 3-5 seconds
- **Memory Usage**: ~100-200 MB
- **Heartbeat Frequency**: Every 30 seconds (configurable)
- **Registry Size**: Scales to thousands of instances
- **Lookup Time**: < 1 second for service discovery
- **Throughput**: Can handle 1000s of registrations per second

---

## 🔄 Service Lifecycle States

```
┌─────────────────┐
│   Registering   │ Service sends initial registration
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│      UP         │ Service is healthy and ready
└────────┬────────┘
         │ (if health check fails)
         ▼
┌─────────────────┐
│  OUT_OF_SERVICE │ Service marked as unavailable
└────────┬────────┘
         │ (if continues failing)
         ▼
┌─────────────────┐
│      DOWN       │ Service is down
└────────┬────────┘
         │ (after timeout)
         ▼
┌─────────────────┐
│    Evicted      │ Service removed from registry
└─────────────────┘
```

---

## 📚 References

- [Netflix Eureka Documentation](https://github.com/Netflix/eureka)
- [Spring Cloud Netflix Eureka](https://spring.io/projects/spring-cloud-netflix)
- [Service Discovery Guide](https://cloud.spring.io/spring-cloud-netflix/reference/html/)
- [Eureka Protocol](https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance)

---

## 📝 Notes

- **Stateless Design**: Eureka server is stateless - can be easily replicated
- **Self-Preservation Mode**: Protects against network partitions
- **Leasing Protocol**: Services lease their registration (must renew)
- **Replication**: Supports multiple Eureka servers for HA (advanced setup)

---

## 🤝 Support

For issues or questions:
1. Check the Eureka dashboard: `http://localhost:9001`
2. Verify all services are running
3. Check service logs for registration errors
4. Ensure Config-Server is running before Service-Registry
5. Verify network connectivity between services

If you encounter any issues, feel free to reach out and start a discussion via the Slack workspace.
