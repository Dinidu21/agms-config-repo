# AGMS Configuration Repository

<p align="center">
  <img src="https://img.shields.io/badge/Configuration-Management-blue?style=for-the-badge" alt="Configuration Management" />
  <img src="https://img.shields.io/badge/Microservices-Architecture-purple?style=for-the-badge&logo=microservices" alt="Microservices" />
</p>

## Table of Contents

1. [Overview](#overview)
2. [Purpose](#purpose)
3. [Directory Structure](#directory-structure)
4. [Configuration Files](#configuration-files)
5. [How It Works](#how-it-works)
6. [Configuration Details](#configuration-details)
7. [Environment Variables](#environment-variables)
8. [Spring Profiles](#spring-profiles)
9. [Service-Specific Configuration](#service-specific-configuration)
10. [External Integrations](#external-integrations)
11. [Security](#security)
12. [Usage](#usage)
13. [Best Practices](#best-practices)
14. [Troubleshooting](#troubleshooting)

---

## Overview

This repository contains the centralized configuration files for the **Automated Greenhouse Management System (AGMS)** microservices architecture. It is used by the Spring Cloud Config Server to provide distributed configuration management across all services.

## Purpose

The configuration repository serves several critical functions in the AGMS architecture:

- **Centralized Configuration** - All service configurations stored in a single location
- **Environment-Specific Settings** - Support for different profiles (dev, staging, production)
- **Decoupled Deployments** - Configuration changes don't require service redeployment
- **Consistency** - All services share common settings via `application.yml`
- **External Property Injection** - Sensitive values loaded from environment variables

---

## Directory Structure

```
configs-repo/
│
├── .env                    # Environment variables (local development)
├── .gitignore              # Git ignore rules
├── README.md               # This file
│
├── application.yml          # Shared configuration across all services
│
├── api-gateway.yml         # API Gateway routing and JWT configuration
│
├── automation-service.yml  # Automation service database and Eureka config
│
├── crop-service.yml        # Crop service port and application name
│
├── sensor-service.yml      # Sensor service port and application name
│
└── zone-service.yml        # Zone service database, JPA, and IoT API config
```

---

## Configuration Files

### 1. application.yml (Shared Configuration)

This file contains settings that apply to **all services** in the system.

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    prefer-ip-address: true

jwt:
  secret: ${JWT_SECRET}
```

**Key Properties:**

| Property | Description | Default Value |
|----------|-------------|---------------|
| `eureka.client.service-url.defaultZone` | Eureka server URL | `http://eureka-server:8761/eureka/` |
| `eureka.instance.prefer-ip-address` | Use IP instead of hostname | `true` |
| `jwt.secret` | JWT signing secret (from env) | `${JWT_SECRET}` |

---

### 2. api-gateway.yml

Configuration for the Spring Cloud Gateway service.

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: zone-service
          uri: lb://zone-service
          predicates:
            - Path=/api/zones, /api/zones/**
        - id: sensor-service
          uri: lb://sensor-service
          predicates:
            - Path=/api/sensors, /api/sensors/**
        - id: automation-service
          uri: lb://automation-service
          predicates:
            - Path=/api/automation, /api/automation/**
        - id: crop-service
          uri: lb://crop-service
          predicates:
            - Path=/api/crops, /api/crops/**
    loadbalancer:
      cache:
        ttl: 5s
```

**Route Configuration:**

| Route ID | Service | URL Pattern | Purpose |
|----------|---------|-------------|---------|
| `zone-service` | Zone Service | `/api/zones/**` | Zone CRUD operations |
| `sensor-service` | Sensor Service | `/api/sensors/**` | Sensor data endpoints |
| `automation-service` | Automation Service | `/api/automation/**` | Automation rules & logs |
| `crop-service` | Crop Service | `/api/crops/**` | Crop lifecycle management |

**Load Balancer:** Cache TTL set to 5 seconds for efficient service discovery.

---

### 3. zone-service.yml

Configuration for the Zone Management Service.

```yaml
server:
  port: 8081

spring:
  application:
    name: zone-service

  datasource:
    url: jdbc:mysql://mysql:3306/zone_db?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    database-platform: org.hibernate.dialect.MySQLDialect

eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/

iot:
  api:
    base-url: http://104.211.95.241:8080/api
    username: dinidu
    password: 123456
```

**Configuration Details:**

| Category | Property | Value |
|----------|----------|-------|
| **Server** | Port | `8081` |
| **Application** | Name | `zone-service` |
| **Database** | Type | MySQL 8.0 |
| **Database** | URL | `mysql:3306/zone_db` |
| **Database** | Auto-create | Enabled with `createDatabaseIfNotExist=true` |
| **JPA** | Hibernate | `ddl-auto: update` |
| **JPA** | SQL Logging | Enabled |
| **IoT API** | Base URL | `http://104.211.95.241:8080/api` |
| **IoT API** | Credentials | `dinidu / 123456` |

---

### 4. automation-service.yml

Configuration for the Automation Service.

```yaml
server:
  port: 8083

spring:
  application:
    name: automation-service
  datasource:
    url: jdbc:postgresql://postgres:5432/automation_db
    username: postgres
    password: postgres
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    database-platform: org.hibernate.dialect.PostgreSQLDialect

eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka-server:8761/eureka/
```

**Configuration Details:**

| Category | Property | Value |
|----------|----------|-------|
| **Server** | Port | `8083` |
| **Application** | Name | `automation-service` |
| **Database** | Type | PostgreSQL |
| **Database** | URL | `postgres:5432/automation_db` |
| **JPA** | Hibernate | `ddl-auto: update` |
| **JPA** | SQL Logging | Enabled |

---

### 5. crop-service.yml

Configuration for the Crop Service (Node.js/TypeScript).

```yaml
server:
  port: 8084

spring:
  application:
    name: crop-service
```

**Note:** This is a minimal configuration since the Crop Service uses native Node.js configuration via environment variables (see main project's `docker-compose.yml` for `MONGO_URI`).

---

### 6. sensor-service.yml

Configuration for the Sensor Service (Python/FastAPI).

```yaml
server:
  port: 8082

spring:
  application:
    name: sensor-service
```

**Note:** This is a minimal configuration since the Sensor Service uses native Python environment variables (see main project's `docker-compose.yml` for `REDIS_HOST`, `REDIS_PORT`, `IOT_BASE_URL`, etc.).

---

## How It Works

### 1. Config Server Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Spring Cloud Config Server               │
│                         (Port 8888)                         │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            │ Reads configuration files
                            ▼
                  ┌─────────────────────┐
                  │   configs-repo/     │
                  │   (Git filesystem)  │
                  └─────────────────────┘
```

### 2. Configuration Flow

```
┌──────────────┐     ┌─────────────────┐     ┌──────────────────┐
│   Client     │────►│  Config Server   │────►│  configs-repo/   │
│  Service     │     │   (Port 8888)    │     │  (YAML files)    │
└──────────────┘     └─────────────────┘     └──────────────────┘
                           │
                           │ Returns enriched config
                           ▼
                    ┌─────────────────┐
                    │  Service starts │
                    │  with config    │
                    └─────────────────┘
```

### 3. Service Startup Order

Services depend on the Config Server being healthy before they can retrieve configuration:

1. **Config Server** starts first (port 8888) - serves all configurations
2. **Eureka Server** starts (port 8761) - service discovery
3. **Other Services** start and fetch their specific configurations from Config Server

---

## Environment Variables

The following environment variables are used to inject sensitive data into the configurations:

| Variable | Used In | Purpose | Example |
|----------|---------|---------|---------|
| `JWT_SECRET` | `application.yml` | JWT token signing | `mySecretKeyForJWTTokenGenerationThatIsLongEnough` |

### Setting Environment Variables

**Option 1: Docker Compose (Recommended)**

```yaml
# In docker-compose.yml
services:
  config-server:
    environment:
      JWT_SECRET: ${JWT_SECRET}
```

**Option 2: .env file**

```bash
# In configs-repo/.env
JWT_SECRET=mySecretKeyForJWTTokenGenerationThatIsLongEnough
```

**Option 3: System Environment**

```bash
# Linux/macOS
export JWT_SECRET=mySecretKeyForJWTTokenGenerationThatIsLongEnough

# Windows
set JWT_SECRET=mySecretKeyForJWTTokenGenerationThatIsLongEnough
```

---

## Spring Profiles

The configuration repository supports different Spring profiles for various environments:

### Default Profile (native)

The `native` profile uses the local filesystem to serve configuration files:

```yaml
# docker-compose.yml
environment:
  SPRING_PROFILES_ACTIVE: native
  SPRING_CLOUD_CONFIG_SERVER_NATIVE_SEARCH_LOCATIONS: file:///configs-repo/
```

### Future Profiles

| Profile | Purpose | Configuration Location |
|---------|---------|------------------------|
| `dev` | Development | `configs-repo/dev/` (future) |
| `staging` | Staging | `configs-repo/staging/` (future) |
| `prod` | Production | `configs-repo/prod/` (future) |

---

## Service-Specific Configuration

### Zone Service Configuration

The Zone Service requires the most extensive configuration because it:

1. **Manages database** - MySQL for zone storage
2. **Integrates with IoT API** - External device registration
3. **Uses JPA/Hibernate** - ORM for database operations
4. **Registers with Eureka** - Service discovery

### Automation Service Configuration

The Automation Service requires:

1. **PostgreSQL database** - Automation logs storage
2. **JPA/Hibernate** - ORM for database operations
3. **Eureka registration** - Service discovery
4. **Feign client** - Communication with Zone Service

### API Gateway Configuration

The API Gateway requires:

1. **Route definitions** - Path to service mapping
2. **Load balancer** - Service discovery integration
3. **JWT secret** - Token validation

---

## External Integrations

### IoT API Integration (Zone Service)

```yaml
iot:
  api:
    base-url: http://104.211.95.241:8080/api
    username: dinidu
    password: 123456
```

**Integration Details:**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/login` | POST | Authenticate and get access token |
| `/devices` | POST | Register new IoT device for zone |
| `/devices/telemetry/{deviceId}` | GET | Fetch sensor telemetry |

### Eureka Service Discovery

All services register with Eureka using:

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    prefer-ip-address: true
```

---

## Security

### Sensitive Data Handling

⚠️ **Important Security Notes:**

1. **JWT Secret**: The `JWT_SECRET` environment variable should be a strong, random string in production
2. **Database Credentials**: Currently hardcoded in config files (for development only)
3. **IoT API Credentials**: Currently hardcoded in `zone-service.yml` (for development only)

### Production Recommendations

For production deployments:

- ✅ Use environment variables for all credentials
- ✅ Store secrets in a vault (HashiCorp Vault, AWS Secrets Manager)
- ✅ Enable TLS/SSL for all connections
- ✅ Rotate credentials regularly
- ✅ Use Kubernetes secrets or Docker secrets

---

## Usage

### Accessing Configuration

Services access configuration via the Config Server:

```bash
# Get zone-service configuration
curl http://localhost:8888/zone-service/default

# Get automation-service configuration
curl http://localhost:8888/automation-service/default

# Get api-gateway configuration
curl http://localhost:8888/api-gateway/default

# Get application-wide configuration
curl http://localhost:8888/application/default
```

### Refreshing Configuration

#### Option 1: Restart Service

```bash
docker-compose restart zone-service
```

#### Option 2: Actuator Refresh (if enabled)

```bash
# Requires spring-boot-starter-actuator and refresh endpoint enabled
curl -X POST http://localhost:8081/actuator/refresh
```

#### Option 3: Spring Cloud Bus (recommended for microservices)

For dynamic configuration updates without service restart, use Spring Cloud Bus with a message broker (RabbitMQ or Kafka).

---

## Best Practices

### 1. Version Control

- ✅ Keep configuration in Git
- ✅ Use meaningful commit messages
- ✅ Review changes via pull requests
- ✅ Tag stable configurations

### 2. Configuration Structure

- ✅ Keep common settings in `application.yml`
- ✅ Service-specific settings in dedicated files
- ✅ Use environment variables for sensitive data
- ✅ Document all non-obvious settings

### 3. Change Management

- ✅ Test configuration changes in dev first
- ✅ Use Spring profiles for environment-specific settings
- ✅ Implement configuration versioning
- ✅ Monitor configuration changes in production

### 4. Security

- ✅ Never commit secrets to version control
- ✅ Use `.gitignore` for sensitive files
- ✅ Encrypt sensitive configuration at rest
- ✅ Audit configuration access

---

## Troubleshooting

### Config Server Issues

#### Problem: Services can't reach Config Server

**Symptoms:**

- Service fails to start with connection error
- Configuration not loading

**Solutions:**

```bash
# 1. Verify Config Server is running
curl http://localhost:8888/actuator/health

# 2. Check config files exist
ls -la configs-repo/

# 3. Verify file permissions
chmod -R 755 configs-repo/

# 4. Check Docker volume mount
docker inspect config-server | grep -A 5 Mounts
```

#### Problem: Configuration not found

**Symptoms:**

- 404 error when accessing config endpoint

**Solutions:**

```bash
# 1. Check file naming (must match service name)
# application.yml (common)
# api-gateway.yml (for api-gateway service)
# zone-service.yml (for zone-service)

# 2. Verify search locations
# In docker-compose:
# SPRING_CLOUD_CONFIG_SERVER_NATIVE_SEARCH_LOCATIONS: file:///configs-repo/
```

### Property Loading Issues

#### Problem: Environment variables not resolved

**Symptoms:**

- `${JWT_SECRET}` appears in loaded configuration instead of actual value

**Solutions:**

```bash
# 1. Verify environment variable is set
echo $JWT_SECRET

# 2. Check .env file exists in configs-repo directory
ls -la configs-repo/.env

# 3. Restart Config Server after env change
docker-compose restart config-server
```

### Database Configuration Issues

#### Problem: Cannot connect to database

**Symptoms:**

- Service starts but database operations fail

**Solutions:**

```bash
# 1. Verify database is running
docker-compose ps mysql postgres

# 2. Check database credentials match
# In zone-service.yml:
#   username: root
#   password: root

# 3. Verify database URL is correct
# MySQL: jdbc:mysql://mysql:3306/zone_db
# PostgreSQL: jdbc:postgresql://postgres:5432/automation_db

# 4. Check network connectivity
docker-compose exec zone-service ping mysql
```

---

## File Reference

### Configuration File Summary

| File | Service | Port | Database | Key Features |
|------|---------|------|----------|---------------|
| `application.yml` | All | - | - | Eureka, JWT |
| `api-gateway.yml` | API Gateway | 8080 | - | Routes, Load Balancer |
| `zone-service.yml` | Zone Service | 8081 | MySQL | JPA, IoT API |
| `automation-service.yml` | Automation Service | 8083 | PostgreSQL | JPA |
| `crop-service.yml` | Crop Service | 8084 | MongoDB | (minimal) |
| `sensor-service.yml` | Sensor Service | 8082 | Redis | (minimal) |

---

## Related Documentation

- [Main Project README](../README.md)
- [API Gateway Documentation](../api-gateway/README.md)
- [Zone Service Documentation](../zone-service/README.md)
- [Automation Service Documentation](../automation-service/README.md)
- [Docker Compose Configuration](../docker-compose.yml)

---

## License

This configuration repository is part of the AGMS project and is licensed under the MIT License.

---

<p align="center">
  <strong>🔧 AGMS Configuration Repository 🔧</strong><br>
  Powered by Spring Cloud Config
</p>
