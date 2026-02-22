# Guía de Ejecución del Pipeline CI/CD DevSecOps

## 🚀 Inicio Rápido

### Opción 1: Ejecutar en GitHub Actions (Recomendado)

1. **Fork o empuja el repositorio a GitHub**
   ```bash
   git add .
   git commit -m "feat: DevSecOps pipeline implementation"
   git push origin main
   ```

2. **Accede a GitHub Actions**
   - Ve a: `https://github.com/YOUR_USERNAME/practica2/actions`
   - Deberías ver el workflow "DevSecOps CI/CD Pipeline - Complete Workflow"
   - El pipeline se ejecutará automáticamente en cada push

3. **Monitorea la ejecución**
   - Click en el workflow que está corriendo
   - Verás el progreso de cada etapa
   - Los logs de cada paso están disponibles

### Opción 2: Ejecutar Localmente (Para testing)

requiere tener instalado:
- Docker
- Docker Compose
- Node.js 20+
- Python 3.11+

```bash
# Instalar herramientas SAST/SCA
pip install semgrep
npm install -g trivy

# Ejecutar cada etapa por separado
bash ./scripts/run-pipeline.sh
```

---

## ⚙️ Configuración Requerida

### Secretos de GitHub (si usas variables sensibles)

Si necesitas usar secretos en production, configura en:
`Settings → Secrets and variables → Actions`

**Secretos recomendados:**
```
USERS_DB_HOST = localhost
USERS_DB_USER = postgres
USERS_DB_PASSWORD = secure_password
ACADEMIC_DB_HOST = localhost
ACADEMIC_DB_USER = postgres
ACADEMIC_DB_PASSWORD = secure_password
REGISTRY_TOKEN = (para pushear a Docker Hub)
```

Para **CI sin DB real**, el pipeline crea`.env` de ejemplo que funcionan.

---

## 📊 Interpretando la Salida

### Verde ✅ (Éxito)

```
✅ Reproducible installation successful
✅ Dependency scan completed
✅ Tests passed
✅ Image built: users-service:abc123def
✅ Health check: API Gateway
```

**Acción:** Merge de PR / Deploy a producción

### Amarillo ⚠️ (Warnings)

```
⚠️  Lint warnings detected
⚠️  Non-critical warnings (npm ls)
⚠️  ESLint not configured, installing...
```

**Acción:** Revisar pero usualmente no bloquea

### Rojo ❌ (Fallo)

```
❌ SAST: SQL Injection detected in users-service/src/auth.services.js
❌ Container: 3 CRITICAL vulnerabilities in users-service
❌ Tests failed: 5 tests failing
```

**Acción:** Fix el problema, no puedes desplegar hasta solucionarlo

---

## 🔍 Verificar Resultados

### 1. Ver logs del Pipeline

```bash
# En GitHub Actions, expandir cada etapa:
1. Setup & Dependency Validation
   └─ npm ci para users-service
   └─ npm ci para academic-service
   └─ npm ci para api-gateway

2. Code Quality Analysis
   └─ Backend ESLint check
   └─ Frontend ESLint check

3. Automated Testing
   └─ Test: users-service
   └─ Test: academic-service
   └─ Test: api-gateway
   └─ Test: frontend

## etc...
```

### 2. Ver Artefactos Generados

Después de la ejecución, descarga en "Artifacts":
- `lock-files-*`: package-lock.json (validación)
- `test-reports-*`: Cobertura de tests
- `trivy-scan-*`: Reportes de vulnerabilidades en JSON
- `pipeline-report`: Resumen ejecutivo

### 3. Examinar una Etapa Específica

Ejemplo: "¿Qué vulnerabilidades encontró Trivy?"

```bash
# Descarga trivy-scan-users-service.json
# Ver vulnerabilidades detectadas:
{
  "Results": [
    {
      "VulnerabilityID": "CVE-2021-3129",
      "Severity": "HIGH",
      "Title": "ALPACA: Application Layer Protocol Confusion",
      "PkgName": "openssl",
      "InstalledVersion": "1.1.1k",
      "FixedVersion": "1.1.1l"
    }
  ]
}
```

---

## 🛠️ Troubleshooting

### Problema: "npm ci falló - package-lock.json desactualizado"

**Solución:**
```bash
cd backend/users-service
npm install  # Actualiza lock file
git add package-lock.json
git commit -m "update: dependencies"
git push
```

### Problema: "ESLint: Too many warnings"

**Solución:**
```bash
# Localamente, ve qué warnings hay
npm run lint

# Fixa automáticamente lo que pueda
npx eslint src/ --fix

git add . && git commit -m "fix: lint warnings"
```

### Problema: "npm audit: Vulnerabilidades no se pueden fixear"

**Solución:**
1. Ver qué librería tiene el CVE
2. Esperar a que el maintainer la actualice
3. O usar una alternativa más segura

**Ejemplo:**
```bash
npm audit
# Finds: lodash < 4.17.21 has Prototype Pollution

# Opción 1: Esperar
npm audit fix

# Opción 2: Cambiar librería
npm uninstall lodash
npm install underscore  # alternativa
```

### Problema: "Trivy: CRITICAL vulnerabilities en alpine base"

**Solución:**

Upgrade la imagen base en Dockerfile
```dockerfile
# ❌ Vieja (vulnerable)
FROM node:16-alpine

# ✅ Nueva (parcheada)
FROM node:20-alpine
```

### Problema: "Tests fallando: Timeout en smoke tests"

**Solución:**
```bash
# Los servicios tardan en levantarse, aumenta sleep
# En devsecops.yml:
- name: "Integration: Start Docker Compose"
  run: |
    docker compose -f backend/docker-compose.yml up -d
    sleep 30  # Aumenta de 20 a 30 segundos
```

---

## 📈 Métricas & Monitoreo

### Siguiendo el Estado del Pipeline

```
Métrica                      | Baseline | Target
-----------------------------|----------|--------
Tiempo total del pipeline    | 15 min   | < 5 min
Tasa de éxito (pass rate)    | 80%      | > 95%
Vulnerabilidades detectadas  | N/A      | < 5 CRITICAL
Cobertura de tests           | 50%      | > 80%
Build reproducible           | No       | Sí
```

### GitHub Actions Dashboard

- **Workflows**: Muestra histórico de ejecuciones
- **Branch protection**: Requiere que el pipeline pase antes de merge
- **Artifacts**: Retiene reportes por 30 días

---

## 🔐 Continuidad: Después del Pipeline

### Lo que hace el pipeline:
✅ Detecta vulnerabilidades antes de producción
✅ Asegura código de calidad
✅ Valida que las mitigaciones no rompan el flujo

### Lo que NECESITA haga el equipo:
⚠️ Revisar findings del pipeline
⚠️ Fixear vulnerabilidades detectadas
⚠️ No mergear PRs que fallan sin justificación
⚠️ Mantener dependencias actualizadas

### Lo que hace DevOps/SRE en prod:
🔒 Secrets management seguro
🔒 Network policies
🔒 Logging & Monitoring
🔒 Incident Response

---

## 📚 Referencias

- [GitHub Actions Documentation](https://docs.github.com/actions)
- [Semgrep Rules](https://semgrep.dev/r)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

## 🎯 Checklist Final

Antes de considerar el pipeline "completo":

- [ ] Pipeline ejecuta sin errores en push a `main`
- [ ] Todos los tests pasan green ✅
- [ ] Trivy no reporta CRITICAL vulnerabilities
- [ ] Semgrep no detecta ERROR nivel issues
- [ ] npm audit muestra MODERATE o mejor
- [ ] Smoke tests validan endpoints funcionan
- [ ] Documentación técnica está actualizada
- [ ] PR requiere que pipeline pase antes de merge

---

**Generado:** 2026-02-22  
**Versión:** 1.0  
**Última actualización:** 2026-02-22
