## 🔐 Enfoque de la Práctica - DevSecOps Implementado

Esta práctica debe ser **implementada, no solo diseñada**.

El objetivo es aplicar **DevSecOps de manera práctica**, integrando:
- ✅ Front-end (React/Vite)
- ✅ Back-end (Microservicios Node.js)
- ✅ Inicio de sesión seguro (JWT + Bcrypt)
- ✅ Arquitectura de microservicios
- ✅ **Automatización CI/CD con seguridad embebida en each stage**

---

## 📊 Pipeline CI/CD DevSecOps

El pipeline está definido en [.github/workflows/devsecops.yml](.github/workflows/devsecops.yml) e implementa **10 etapas de seguridad y calidad**:

### Etapas del Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│ 1️⃣  Setup & Dependency Validation   [npm ci]                    │
│ 2️⃣  Code Quality Analysis           [ESLint]                    │
│ 3️⃣  Automated Testing               [Jest]                      │
│ 4️⃣  Dependency Security Scanning    [npm audit + SCA]           │
│ 5️⃣  Static Security Analysis        [Semgrep + Trufflehog]      │
│ 6️⃣  Environment Setup               [.env validation]           │
│ 7️⃣  Docker Build & Tagging          [Reproducible images]       │
│ 8️⃣  Container Security Scanning     [Trivy]                     │
│ 9️⃣  Integration & Smoke Tests       [docker-compose + curl]     │
│ 🔟 Reporting & Summary              [Artifacts upload]          │
└─────────────────────────────────────────────────────────────────┘
```

### Herramientas de Seguridad

| Herramienta | Función | Fase |
|------------|---------|------|
| **npm ci** | Instalación reproducible | Setup |
| **ESLint** | Análisis de calidad de código | QA |
| **Jest** | Unit & integration tests | Testing |
| **npm audit** | Scanning de dependencias (SCA) | Dependency Security |
| **Semgrep** | SAST: Detección de vulnerabilidades | Static Analysis |
| **Trufflehog** | SAST: Detección de secrets | Static Analysis |
| **Docker Build** | Construcción con trazabilidad | Build |
| **Trivy** | Container scanning: Vulnerabilidades en layers | Container Security |
| **docker-compose** | Smoke tests de integración | Integration Testing |

### Riesgos Mitigados

✅ **Supply Chain Attacks** → Policy: Solo dependencias verificadas  
✅ **Code Injection (SQL, XSS, Command)** → Policy: SAST detección  
✅ **Vulnerable Dependencies** → Policy: npm audit bloqueante  
✅ **Hardcoded Secrets** → Policy: Trufflehog bloqueante  
✅ **Container Vulnerabilities** → Policy: Trivy escaneo completo  

---

## 📖 Documentación Técnica

Para detalles completos de **justificación técnica, arquitectura de decisiones y análisis de riesgos**:

👉 **[DEVSECOPS_JUSTIFICATION.md](DEVSECOPS_JUSTIFICATION.md)** ← Leer este documento

### Contenido de la Justificación Técnica

- ✅ Explicación de cada herramienta y por qué es necesaria
- ✅ Ejemplos de vulnerabilidades detectadas
- ✅ Análisis de riesgos mitigados
- ✅ Comparativas con alternativas consideradas
- ✅ Justificación del orden de ejecución
- ✅ Impacto en seguridad, velocidad y cumplimiento
     

# 1. Adición del Front-end

[ Front-end ]
     |
     | Login / JWT
     v
[ users-service ]
     |
     | JWT
     v
[ api-gateway ]
     |
     v
[ academic-service ]


## Integración DevSecOps (obligatoria)
El Front-end y el inicio de sesión deben estar cubiertos por el pipeline DevSecOps existente:

- SAST: análisis del código de autenticación y manejo de inputs.
- SCA: análisis de dependencias relacionadas con seguridad.
- DAST: pruebas de acceso no autorizado a endpoints protegidos.

**El login no se asume seguro, se valida automáticamente

## Propósito de esta extensión
Consolidar una visión end-to-end DevSecOps, donde:
 - El diseño,
 - La seguridad,
 - La automatización,
  Y la experiencia de usuario,
se integran desde las primeras etapas del desarrollo.

## Pipeline 
Commit / Pull Request
   ↓
Tests automatizados
   ↓
SAST (Semgrep)
   ↓
Build (Docker)
   ↓
SCA (dependencias)
   ↓
Deploy automático
   ↓
DAST (aplicación en ejecución)

## Docker Compose
docker-compose down
docker-compose up --build

## Estructura del Pipeline
Push / Pull Request
   ↓
Install dependencies
   ↓
Tests (backend + frontend)
   ↓
SAST (Semgrep)
   ↓
Build Docker images
   ↓
SCA (Trivy)
   ↓
docker-compose up
   ↓
Smoke tests

## Kubernetes
kubectl apply -f k8s/users-service/
kubectl apply -f k8s/academic-service/
kubectl apply -f k8s/api-gateway/

kubectl get pods
kubectl get services

# Correr api-gateway
minikube service api-gateway
minikube start
# Trabajar con Docker
eval $(minikube docker-env -u)
# Trabajar Docker dentro Kubernetes
1. minikube start --driver=docker
   eval $(minikube docker-env)
2. minikube status
3. kubectl config current-context
4. kubectl get nodes
## Construir las imágenes
docker build -t frontend:latest ../frontend
kubectl get pods -n backend
docker build -t users-service:latest ../backend/users-service

## Desplegar en Kubernetes
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/users-service/
kubectl apply -f k8s/academic-service/
kubectl apply -f k8s/api-gateway/
kubectl apply -f k8s/frontend/

# eliminar cluster
minikube delete


minikube start
eval $(minikube docker-env)

docker build -t users-service backend/users-service
docker build -t academic-service backend/academic-service
docker build -t api-gateway backend/api-gateway
docker build -t frontend frontend

kubectl apply -f k8s/