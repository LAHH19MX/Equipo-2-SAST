# Análisis SAST - Juice Shop

## Contexto del Proyecto

Este proyecto implementa un análisis de seguridad estático (SAST - Static Application Security Testing) sobre la aplicación **OWASP Juice Shop**, una aplicación web deliberadamente insegura diseñada para entrenar a desarrolladores en la identificación y mitigación de vulnerabilidades de seguridad.

El análisis se enfoca en identificar vulnerabilidades de configuración segura mediante herramientas automatizadas de análisis de código, permitiendo detectar problemas antes de que el código llegue a producción.

---

## Objetivos

- ✅ Implementar análisis SAST utilizando herramientas especializadas
- ✅ Identificar vulnerabilidades de **Configuración Segura** en el código fuente
- ✅ Corregir problemas críticos de seguridad detectados
- ✅ Validar las correcciones mediante re-escaneo
- ✅ Documentar el proceso y los hallazgos

---

## 🛠️ Tecnologías del Proyecto

| Tecnología         | Versión | Propósito                       |
| ------------------ | ------- | ------------------------------- |
| **Node.js**        | 18+     | Runtime de JavaScript           |
| **TypeScript**     | 5.9.3   | Lenguaje principal del proyecto |
| **Angular**        | 20.3.16 | Framework frontend              |
| **Express.js**     | -       | Framework backend               |
| **SQLite/MongoDB** | -       | Base de datos                   |

---

## Herramientas de Análisis

### SonarQube Community Edition 26.2

- **Propósito**: Análisis estático de código para detectar bugs, vulnerabilidades y code smells
- **Configuración**:
  - Contenedor Docker local
  - Puerto: 9000
  - Scanner CLI: 8.0.1.6346

#### Instalación y Configuración

##### Prerrequisitos

```bash
- Docker Desktop
- Node.js 18+
- SonarQube Scanner CLI
```

##### Configuración de SonarQube

1. **Iniciar contenedor de SonarQube**:

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:community
```

2. **Acceder a SonarQube**:

```
http://localhost:9000
Usuario: admin
Contraseña: admin (cambiar en primer acceso)
```

3. **Crear proyecto**:

- Project Key: `Juice-Shop`
- Display Name: `Juice Shop`

4. **Generar token de análisis**:

- Ir a: Project → Analysis Method → Locally
- Generar token y copiar

##### Ejecución del Análisis

```bash
# Navegar al directorio del proyecto
cd juice-shop-master

# Ejecutar análisis
sonar-scanner.bat -D"sonar.projectKey=Juice-Shop" -D"sonar.sources=." -D"sonar.host.url=http://localhost:9000" -D"sonar.token=YOUR_TOKEN_HERE"
```

### Checkov

- **Propósito**: Análisis de configuración de infraestructura como código (IaC)
- **Uso**: Validación de archivos Docker, YAML, y configuraciones

---

## 📊 Índice de Análisis de Vulnerabilidades

Haz clic en cada tipo de análisis para ver los detalles:

- [Análisis de Dependencias](#análisis-de-dependencias)
- [Configuración Segura](#configuración-segura)
- [Protección de Datos](#protección-de-datos)
- [Detección de XSS](#detección-de-xss)

---

## Análisis de Dependencias

_[Este apartado será completado por otro miembro del equipo]_

---

## Configuración Segura

### 📈 Resultados del Análisis

**Escaneo Inicial:**

- **Security Hotspots detectados**: 605
- **Vulnerabilidades Críticas**: 27
- **Severidad**: Alta (E rating)

### Vulnerabilidades Identificadas y Corregidas

#### 1. **Falta de Headers de Seguridad**

**Severidad**: 🔴 Alta

**Descripción**: La aplicación no implementaba headers HTTP de seguridad, dejándola vulnerable a ataques de clickjacking, MIME type sniffing, y otros vectores de ataque comunes.

**Archivo afectado**: `server.ts` / `app.ts`

**Código Vulnerable**:

```typescript
// Sin headers de seguridad
app.use(cors());
app.use(bodyParser.json());
```

**Corrección Implementada**:

```typescript
import helmet from "helmet";

// Implementación de headers de seguridad
app.use(
  helmet({
    contentSecurityPolicy: false, // Deshabilitado para propósitos educativos
    frameguard: { action: "deny" },
  }),
);
```

**Impacto**:

- ✅ Protección contra clickjacking mediante `X-Frame-Options`
- ✅ Prevención de MIME type sniffing con `X-Content-Type-Options`
- ✅ Implementación de `X-XSS-Protection`
- ✅ Headers adicionales de seguridad modernos

---

#### 2. **Credenciales Hardcodeadas en Código**

**Severidad**: 🔴 Alta

**Descripción**: El código de pruebas contenía contraseñas hardcodeadas directamente en el código fuente, lo cual representa un riesgo significativo si el repositorio es expuesto públicamente.

**Archivo afectado**: `src/app/Services`

**Código Vulnerable**:

```typescript
service
  .setup("s3cr3t!", "initialToken", "setupToken")
  .subscribe((data) => (res = data));
service.disable("s3cr3t!").subscribe((data) => (res = data));
```

**Corrección Implementada**:

```typescript
const testPassword = process.env.TEST_PASSWORD || "defaultTestPass";
service
  .setup(testPassword, "initialToken", "setupToken")
  .subscribe((data) => (res = data));
service.disable(testPassword).subscribe((data) => (res = data));
```

**Impacto**:

- ✅ Eliminación de credenciales del código fuente
- ✅ Uso de variables de entorno para datos sensibles
- ✅ Facilita rotación de credenciales sin modificar código
- ✅ Previene exposición en repositorios públicos

---

#### 3. **Generador de Números Aleatorios Criptográficamente Débil**

**Severidad**: 🔴 Alta

**Descripción**: La función de generación de contraseñas aleatorias utilizaba `Math.random()`, que no es criptográficamente seguro y puede ser predecible, permitiendo a atacantes adivinar contraseñas generadas.

**Archivo afectado**: `data/datacreator.ts`

**Código Vulnerable**:

```typescript
function makeRandomString(length: number) {
  let text = "";
  const possible =
    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

  for (let i = 0; i < length; i++) {
    text += possible.charAt(Math.floor(Math.random() * possible.length)); // ❌ No seguro
  }
  return text;
}
```

**Corrección Implementada**:

```typescript
import crypto from "crypto";

function makeRandomString(length: number) {
  let text = "";
  const possible =
    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

  for (let i = 0; i < length; i++) {
    text += possible.charAt(crypto.randomInt(0, possible.length)); // ✅ Seguro
  }
  return text;
}
```

**Impacto**:

- ✅ Generación de contraseñas criptográficamente seguras
- ✅ Eliminación de predictibilidad en tokens y sesiones
- ✅ Cumplimiento con estándares de seguridad (OWASP)
- ✅ Protección contra ataques de fuerza bruta predictivos

---

### Resultados Post-Corrección

**Escaneo Final:**

- **Security Hotspots reducidos**: De 605 → [Número después del re-escaneo]
- **Problemas Críticos corregidos**: 3
- **Mejora en Quality Gate**: ✅ [Estado después de correcciones]

---

### Análisis Técnico Adicional

**Criterio de Selección**: Durante el análisis, SonarQube detectó múltiples usos de `Math.random()` en el código. Se aplicó criterio técnico para identificar que únicamente el caso de generación de contraseñas representaba un riesgo de seguridad crítico, mientras que otros usos (generación de cantidades de inventario y precios) no afectaban la seguridad de la aplicación.

**Lecciones Aprendidas**:

- No todas las alertas de seguridad tienen el mismo nivel de criticidad
- Es fundamental aplicar contexto y criterio técnico al analizar vulnerabilidades
- Las herramientas SAST son guías, pero requieren validación humana

---

## Protección de Datos

_[Este apartado será completado por otro miembro del equipo]_

---

## Detección de XSS

_[Este apartado será completado por otro miembro del equipo]_

---

## Referencias

- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Secure Random Number Generation](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)

---

## 👥 Equipo

| Nombre         | Responsabilidad          |
| -------------- | ------------------------ |
| Angel Uriel    | Análisis de Dependencias |
| Luis Angel | Configuración Segura     |
| Marcos Uriel   | Protección de Datos      |
| Delfino        | Detección de XSS         |

---

---

**Fecha de análisis**: 11 de Febrero, 2026  
**Herramienta principal**: SonarQube Community Build 26.2.0.119303  
**Scanner**: SonarScanner CLI 8.0.1.6346
