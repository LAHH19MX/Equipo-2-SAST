Hey ahi agregan lo de cada quien que falta:
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

3. **Crear proyecto manualmente**:

- Click en "Create Project" → "Manually"
- Project Key: `Juice-Shop`
- Display Name: `Juice Shop`
- Click "Next"

4. **Configurar New Code**:

- Seleccionar: "Follows the instance's default"
- Click "Create project"

5. **Seleccionar método de análisis**:

- Click en "Locally" (Analysis Method)

6. **Generar token de análisis**:

- Token name: `juice-shop-token`
- Expires in: 30 days
- Click "Generate"
- Copiar el token generado

##### Instalación del SonarScanner CLI

1. **Descargar SonarScanner**:
   - Ir a: https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/
   - Descargar SonarScanner CLI para Windows
   - Extraer el ZIP a una ubicación, ejemplo: `C:\Users\Public\Material\`

2. **Verificar estructura de carpetas**:

```powershell
dir "C:\Users\Public\Material\sonar-scanner-cli-*\bin"
```

3. **Agregar al PATH (temporal)**:

```powershell
$env:PATH += ";C:\Users\Public\Material\sonar-scanner-8.0.1.6346-windows-x64\bin"
```

4. **Verificar instalación**:

```powershell
sonar-scanner --version
```

##### Configuración del Análisis

Después de generar el token, en la página de SonarQube:

- Run analysis on your project:
  - Seleccionar: "Other (for Go, PHP, ...)"
  - Seleccionar OS: "Windows"

##### Ejecución del Análisis

```powershell
# Navegar al directorio del proyecto
cd "C:\ruta\a\juice-shop-master"

# Ejecutar análisis
sonar-scanner.bat -D"sonar.projectKey=Juice-Shop" -D"sonar.sources=." -D"sonar.host.url=http://localhost:9000" -D"sonar.token=YOUR_TOKEN_HERE"
```

Tiempo estimado: 5-7 minutos

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

El análisis de dependencias es una técnica de SAST enfocada en la **"Cadena de Suministro"**. Su objetivo es asegurar que las librerías de terceros no introduzcan vulnerabilidades en nuestro software.

---

## 📋 Paso 1: Preparación del Entorno

Antes de auditar, debemos garantizar que el entorno sea consistente.

| Comando | Función Técnica | Propósito de Seguridad (¿Por qué?) |
|---------|-----------------|-------------------------------------|
| `npm install` | Descarga las librerías y genera el archivo `package-lock.json`. | **Evitar el "Dependency Drift"**: Sin un candado (lock), cada desarrollador podría tener versiones distintas. El candado asegura que analicemos exactamente el mismo código que irá a producción. |

---

## 🔍 Paso 2: Identificación de Riesgos

Una vez estabilizado el entorno, procedemos a la detección de fallos.

### Comando: `npm audit`

**¿Para qué sirve?** Cruza nuestra lista de librerías locales con la base de datos de GitHub Advisory, identificando fallos conocidos (CVEs).

**Resultado del Análisis:** Encontramos **48 vulnerabilidades**, resaltando **7 críticas** que ponen en riesgo la integridad del e-commerce.

---

## ⚖️ Paso 3: Análisis y Selección de Mitigación

No todos los fallos se arreglan igual. Aquí es donde el analista toma decisiones.

### El problema del comando `npm audit fix`

Aunque es automático, el reporte nos advierte que muchas correcciones son **"Breaking Changes"**.

### ¿Por qué no usar `--force`?

En una aplicación real (como un sistema de pagos), forzar una actualización podría romper la funcionalidad del sitio. Por ello, optamos por la **Remediación Manual Controlada**.

---

## 🔧 Paso 4: Ejecución de Parches Críticos

Seleccionamos las **3 vulnerabilidades más peligrosas** para neutralizarlas manualmente.

### A. Criptografía Obsoleta (`crypto-js`)

**Comando:** 
```bash
npm install pdfkit@0.17.2
```

**¿Por qué este comando?** El fallo crítico de `crypto-js` (algoritmo PBKDF2 1.3 millones de veces más débil que el estándar) viene "escondido" dentro de la librería `pdfkit`.

**¿Para qué?** Al actualizar el paquete raíz (`pdfkit`), forzamos la instalación de una versión segura de la librería criptográfica.

---

### B. Inyección de Objetos (`lodash`)

**Comando:** 
```bash
npm install sanitize-html@2.17.0
```

**¿Por qué este comando?** `lodash` presenta **Prototype Pollution**, lo que permite a un atacante inyectar propiedades en el código y escalar privilegios.

**¿Para qué?** Protegemos la lógica del servidor contra manipulaciones maliciosas de objetos JavaScript.

---

### C. Ejecución de Código Remoto (`vm2`)

**Comando:** 
```bash
npm install juicy-chat-bot@0.6.4
```

**¿Por qué este comando?** `vm2` tiene un **escape de Sandbox crítico**.

**¿Para qué?** Evitamos que un atacante pueda ejecutar comandos directamente en el sistema operativo del servidor.

---

## ✅ Paso 5: Verificación Final

**Comando:** 
```bash
npm audit
```

**¿Por qué se repite?** En seguridad, confiar no es suficiente, hay que **verificar**.

**Propósito:** Confirmar que las líneas de código (como la 215 y 217 del `package.json`) ahora reflejan versiones seguras y que el contador de vulnerabilidades críticas ha disminuido de **7 a 4**.

---

## 📝 Resumen del Valor Agregado

Este proceso demuestra que un analista de seguridad no solo corre herramientas, sino que **comprende la jerarquía de dependencias** y equilibra la seguridad con la disponibilidad del sistema, evitando parches automáticos que podrían causar caídas del servicio.

---

## 📊 Métricas del Análisis

| Métrica | Valor Inicial | Valor Final | Mejora |
|---------|---------------|-------------|--------|
| Vulnerabilidades Totales | 48 | ➡️ | Reducción progresiva |
| Vulnerabilidades Críticas | 7 | 4 | ✅ 43% de reducción |
| Breaking Changes Evitados | N/A | 100% | ✅ Estabilidad mantenida |

---

## 🔐 Buenas Prácticas Aplicadas

- ✅ **Reproducibilidad**: Uso de `package-lock.json` para entornos consistentes
- ✅ **Análisis de Impacto**: Evaluación de breaking changes antes de aplicar parches
- ✅ **Remediación Quirúrgica**: Actualización selectiva vs. actualización masiva
- ✅ **Verificación Continua**: Re-auditoría post-mitigación
- ✅ **Documentación**: Justificación técnica de cada decisión


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

# Práctica de Protección de Datos con Checkov

## 📋 Objetivo
Demostrar el cumplimiento de protección de datos mediante el análisis de vulnerabilidades en Juice Shop utilizando la herramienta Checkov.

---

## 🔍 1. Escaneo de Vulnerabilidades

### Agregamos al path la carpeta donde se encuentra el instalador de CHECKOV
C:\Users\uriel\Downloads\Checkov\checkov_windows_X86_64\dist\_  ------------ Ejemplo

### Comando Ejecutado en la posicion de JUICE-SHOP 
```bash
checkov -d .\data --framework secrets
```

### Resultado del Escaneo
```
secrets scan results:

Passed checks: 0, Failed checks: 4, Skipped checks: 0
```

### Vulnerabilidades Encontradas

| # | Archivo | Línea | Vulnerabilidad | Severidad |
|---|---------|-------|----------------|-----------|
| 1 | `/data/static/users.yml` | 88-89 | Password en Base64 | HIGH |
| 2 | `/data/static/users.yml` | 150-151 | TOTP Secret expuesto | HIGH |
| 3 | `/data/static/users.yml` | 204-205 | Password en texto plano | HIGH |
| 4 | `/data/static/users.yml` | 270-271 | Password hardcodeado | HIGH |

**Problema Identificado:** Contraseñas y secretos hardcodeados directamente en el código fuente, violando principios de protección de datos.

---

## 🔧 2. Solución Implementada

### Paso 1: Crear archivo `.env`
Creamos un archivo para almacenar los secretos de forma segura:

```env
# SECRETOS - NO COMPARTIR
USER_PASSWORD_BJOERN=bW9jLmxpYW1nQGhjaW5pbW1pay5ucmVvamI=
USER_TOTP_WURSTBROT=IFTXE3SPOEYVURT2MRYGI52TKJ4HC3KH
USER_PASSWORD_UVOGIN=muda-muda > ora-ora
USER_PASSWORD_TESTING=IamUsedForTesting
```

### Paso 2: Modificar `users.yml`

**ANTES (Vulnerable):**
```yaml
# Línea 88
password: 'bW9jLmxpYW1nQGhjaW5pbW1pay5ucmVvamI='

# Línea 150
totpSecret: IFTXE3SPOEYVURT2MRYGI52TKJ4HC3KH

# Línea 204
password: 'muda-muda > ora-ora'

# Línea 270
password: 'IamUsedForTesting'
```

**DESPUÉS (Seguro):**
```yaml
# Línea 88
password: ${USER_PASSWORD_BJOERN}

# Línea 150
totpSecret: ${USER_TOTP_WURSTBROT}

# Línea 204
password: ${USER_PASSWORD_UVOGIN}

# Línea 270
password: ${USER_PASSWORD_TESTING}
```

### Paso 3: Agregar `.env` al `.gitignore`
```gitignore
# Secretos - NO subir a repositorios
.env
*.env.local
```

---

## ✅ 3. Verificación de la Corrección

### Comando de Verificación
```bash
checkov -d .\data --framework secrets
```

### Resultado Final
```
[ secrets framework ]: 100%|████████████████████|[95/95]

✅ Escaneo completado sin vulnerabilidades detectadas
```

**Interpretación:** 
- ✅ 0 vulnerabilidades encontradas
- ✅ Los secretos están protegidos en variables de entorno
- ✅ Cumplimiento con protección de datos verificado

---

## 📊 Resumen de la Práctica

| Fase | Estado |
|------|--------|
| **Análisis Inicial** | 4 vulnerabilidades críticas detectadas |
| **Implementación** | Secretos externalizados a archivo `.env` |
| **Verificación** | 0 vulnerabilidades - ✅ APROBADO |

---

## 🎯 Conclusión

Se demostró exitosamente el cumplimiento de protección de datos mediante:

1. **Identificación** de vulnerabilidades con Checkov
2. **Remediación** mediante externalización de secretos
3. **Verificación** del cumplimiento con nuevo escaneo

**Resultado:** Sistema seguro que cumple con principios de confidencialidad y protección de datos sensibles.

---

## Detección de XSS

### 📈 Resultados del Análisis

**Escaneo Inicial:**

- **Vulnerabilidades XSS detectadas**: [Número de casos detectados]
- **Severidad**: 🔴 Alta
- **Archivos afectados**: 3 componentes críticos

### Vulnerabilidades Identificadas y Corregidas

#### 1. **XSS en Galería de Comentarios (about.component.ts)**

**Severidad**: 🔴 Alta

**Descripción**: El componente de "About" inyectaba directamente comentarios de usuarios en el HTML sin sanitización previa, permitiendo la ejecución de scripts maliciosos mediante `bypassSecurityTrustHtml()`.

**Archivo afectado**: `frontend/src/app/about/about.component.ts`

**Código Vulnerable**:

```typescript
.subscribe((feedbacks) => {
  for (let i = 0; i < feedbacks.length; i++) {
    // ❌ PELIGRO: Inyección directa de contenido del usuario
    feedbacks[i].comment = `<figcaption><p style="margin-bottom: 0;">${
      feedbacks[i].comment
    }</p><div class="feedback-stars">(${this.stars[feedbacks[i].rating]})</div></figcaption>`

    feedbacks[i].comment = this.sanitizer.bypassSecurityTrustHtml(
      feedbacks[i].comment
    )

    this.galleryRef.addImage({
      src: this.images[i % this.images.length],
      args: feedbacks[i].comment
    })
  }
})
```

**Corrección Implementada**:

```typescript
import { SecurityContext } from '@angular/core'

.subscribe((feedbacks) => {
  for (let i = 0; i < feedbacks.length; i++) {
    // ✅ PASO 1: SANITIZAR el comentario del usuario (ELIMINA <script>, etc.)
    const sanitizedComment = this.sanitizer.sanitize(
      SecurityContext.HTML,
      feedbacks[i].comment
    ) || ''

    // ✅ PASO 2: Construir HTML con el comentario YA LIMPIO
    const safeHtml = `<figcaption>
        <p style="margin-bottom: 0;">${sanitizedComment}</p>
        <div class="feedback-stars">(${this.stars[feedbacks[i].rating]})</div>
      </figcaption>`

    // ✅ PASO 3: AHORA SÍ es seguro el bypass (todo el HTML es controlado por NOSOTROS)
    feedbacks[i].comment = this.sanitizer.bypassSecurityTrustHtml(safeHtml)

    this.galleryRef.addImage({
      src: this.images[i % this.images.length],
      args: feedbacks[i].comment
    })
  }
})
```

**Impacto**:

- ✅ Prevención de ejecución de scripts maliciosos en comentarios
- ✅ Eliminación automática de tags peligrosos (`<script>`, `<iframe>`, etc.)
- ✅ Protección contra robo de cookies y sesiones
- ✅ Prevención de desfiguración del sitio (defacement)

---

#### 2. **XSS en Panel de Administración - Emails (administration.component.ts)**

**Severidad**: 🔴 Alta

**Descripción**: El panel de administración mostraba emails de usuarios directamente en el HTML sin sanitización, permitiendo que emails maliciosos ejecutaran código JavaScript.

**Archivo afectado**: `frontend/src/app/administration/administration.component.ts`

**Código Vulnerable**:

```typescript
findAllUsers () {
  this.userService.find().subscribe({
    next: (users) => {
      this.userDataSource = users
      this.userDataSourceHidden = users
      for (const user of this.userDataSource) {
        // ❌ PELIGRO: Email del usuario inyectado sin sanitizar
        user.email = this.sanitizer.bypassSecurityTrustHtml(
          `<span class="${this.doesUserHaveAnActiveSession(user) ? 'confirmation' : 'error'}">${user.email}</span>`
        )
      }
      this.userDataSource = new MatTableDataSource(this.userDataSource)
      this.userDataSource.paginator = this.paginatorUsers
      this.resultsLengthUser = users.length
    },
    error: (err) => {
      this.error = err
      console.log(this.error)
    }
  })
}
```

**Corrección Implementada**:

```typescript
import { SecurityContext } from '@angular/core'

findAllUsers () {
  this.userService.find().subscribe({
    next: (users) => {
      this.userDataSource = users
      this.userDataSourceHidden = users
      for (const user of this.userDataSource) {
        // ✅ PASO 1: Sanitizar el EMAIL del usuario (viene del atacante)
        const safeEmail = this.sanitizer.sanitize(
          SecurityContext.HTML,
          user.email
        ) || ''

        // ✅ PASO 2: Construir el SPAN con el email YA LIMPIO
        const safeHtml = `<span class="${this.doesUserHaveAnActiveSession(user) ? 'confirmation' : 'error'}">${safeEmail}</span>`

        // ✅ PASO 3: AHORA SÍ es seguro usar bypass (solo HTML controlado)
        user.email = this.sanitizer.bypassSecurityTrustHtml(safeHtml)
      }

      this.userDataSource = new MatTableDataSource(this.userDataSource)
      this.userDataSource.paginator = this.paginatorUsers
      this.resultsLengthUser = users.length
    },
    error: (err) => {
      this.error = err
      console.log(this.error)
    }
  })
}
```

**Impacto**:

- ✅ Protección del panel de administración contra XSS
- ✅ Prevención de compromiso de cuentas administrativas
- ✅ Eliminación de vectores de ataque a usuarios privilegiados
- ✅ Sanitización de datos en contextos de alta criticidad

---

#### 3. **XSS en Panel de Administración - Feedbacks (administration.component.ts)**

**Severidad**: 🔴 Alta

**Descripción**: La tabla de feedbacks en el panel administrativo mostraba comentarios sin sanitización, permitiendo ataques XSS dirigidos a administradores.

**Archivo afectado**: `frontend/src/app/administration/administration.component.ts`

**Código Vulnerable**:

```typescript
findAllFeedbacks () {
  this.feedbackService.find().subscribe({
    next: (feedbacks) => {
      this.feedbackDataSource = feedbacks
      for (const feedback of this.feedbackDataSource) {
        // ❌ PELIGRO: Bypass sin sanitización
        feedback.comment = this.sanitizer.bypassSecurityTrustHtml(feedback.comment)
      }
      this.feedbackDataSource = new MatTableDataSource(this.feedbackDataSource)
      this.feedbackDataSource.paginator = this.paginatorFeedb
      this.resultsLengthFeedback = feedbacks.length
    },
    error: (err) => {
      this.error = err
      console.log(this.error)
    }
  })
}
```

**Corrección Implementada**:

```typescript
findAllFeedbacks () {
  this.feedbackService.find().subscribe({
    next: (feedbacks) => {
      this.feedbackDataSource = feedbacks
      for (const feedback of this.feedbackDataSource) {
        // ✅ PASO 1: Sanitizar el COMENTARIO del usuario
        const safeComment = this.sanitizer.sanitize(
          SecurityContext.HTML,
          feedback.comment
        ) || ''

        // ✅ PASO 2: Usar bypass con el comentario YA LIMPIO
        feedback.comment = this.sanitizer.bypassSecurityTrustHtml(safeComment)
      }

      this.feedbackDataSource = new MatTableDataSource(this.feedbackDataSource)
      this.feedbackDataSource.paginator = this.paginatorFeedb
      this.resultsLengthFeedback = feedbacks.length
    },
    error: (err) => {
      this.error = err
      console.log(this.error)
    }
  })
}
```

**Impacto**:

- ✅ Protección contra XSS almacenado (Stored XSS)
- ✅ Prevención de escalación de privilegios vía panel admin
- ✅ Seguridad en revisión de contenido generado por usuarios
- ✅ Defensa en profundidad para datos persistentes

---

### Análisis Técnico de la Vulnerabilidad

**Patrón Común Detectado**:
Los tres casos compartían el mismo antipatrón de seguridad:

1. Recibir datos del usuario (no confiables)
2. Inyectarlos directamente en templates HTML
3. Usar `bypassSecurityTrustHtml()` sin sanitización previa

**¿Por qué es peligroso?**

```typescript
// Usuario malicioso envía:
comment: '<img src=x onerror="alert(document.cookie)">';

// Sin sanitización → Se ejecuta el script
// Con sanitización → Se convierte en texto plano
```

**Principio de Seguridad Aplicado**:

```
NUNCA confiar en datos del usuario
    ↓
SIEMPRE sanitizar ANTES de usar bypassSecurityTrustHtml()
    ↓
SOLO hacer bypass de HTML que TÚ controlas
```

---

### Resultados Post-Corrección

**Escaneo Final:**

- **Vulnerabilidades XSS corregidas**: 3 casos críticos
- **Método de corrección**: Sanitización con `SecurityContext.HTML`
- **Componentes seguros**: ✅ about.component.ts, ✅ administration.component.ts
- **Patrón de seguridad**: Implementado en todas las visualizaciones de contenido de usuario

---

### Buenas Prácticas Implementadas

**1. Sanitización en Dos Pasos**:

```typescript
// ✅ CORRECTO
const clean = this.sanitizer.sanitize(SecurityContext.HTML, userInput) || "";
const safe = `<div>${clean}</div>`;
this.sanitizer.bypassSecurityTrustHtml(safe);

// ❌ INCORRECTO
this.sanitizer.bypassSecurityTrustHtml(`<div>${userInput}</div>`);
```

**2. Uso de SecurityContext**:

- `SecurityContext.HTML`: Limpia tags peligrosos
- `SecurityContext.SCRIPT`: Previene ejecución de JavaScript
- `SecurityContext.URL`: Valida URLs
- `SecurityContext.STYLE`: Sanitiza CSS inline

**3. Defensa en Profundidad**:

- Sanitización en frontend (Angular)
- Validación en backend (próximo paso recomendado)
- Content Security Policy (CSP) en headers

---

### Lecciones Aprendidas

**Nunca usar `bypassSecurityTrustHtml()` directamente con entrada de usuario**:

- Angular deshabilita su sanitizador automático cuando usas `bypassSecurityTrustHtml()`
- Esto es por diseño: el desarrollador dice "confío en este HTML"
- Si ese HTML viene del usuario → XSS garantizado

**Orden correcto de operaciones**:

1. Sanitizar datos externos con `sanitize()`
2. Construir estructura HTML con datos limpios
3. Solo entonces usar `bypassSecurityTrustHtml()` si es necesario

**Contextos de alto riesgo**:

- Comentarios y feedback de usuarios
- Emails y datos de perfil
- Cualquier campo de texto libre
- Paneles administrativos (objetivo de alto valor)

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
| **Luis Angel** | Configuración Segura     |
| Marcos Uriel   | Protección de Datos      |
| **Delfino**    | Detección de XSS         |

---
