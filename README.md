# Trivy: Guía Técnica Avanzada y CheatSheet para Analistas de Vulnerabilidades

> **Autor:**  Daniel Gallego Badía
> **Módulo 6:** Vulnerabilidades - Actividad: El Arsenal de un Analista  
> **Herramienta:** Trivy (Aqua Security) 
> **Objetivo del recurso:** Proporcionar una guía operativa y rápida sobre el uso avanzado de Trivy para la identificación, análisis y gestión de vulnerabilidades en entornos basados en contenedores, código e infraestructura como código (IaC).

---

## 1. Introducción y Contexto

En el análisis de vulnerabilidades moderno, la superficie de ataque ya no se limita a sistemas operativos tradicionales expuestos en red, sino que abarca artefactos de software, imágenes de contenedores, dependencias de terceros y configuraciones de infraestructura en la nube. **Trivy** se ha consolidado como una herramienta imprescindible en el arsenal del analista debido a sus características principales:

* **Versatilidad analítica:** Capaz de escanear imágenes de contenedores, sistemas de archivos locales, repositorios de código fuente, configuraciones de IaC (Kubernetes, Dockerfile,...) y dependencias de lenguajes (SBOM).
* **Velocidad y precisión:** Diseñada para integrarse en flujos de trabajo rápidos (CI/CD) sin comprometer la profundidad de la detección de CVEs.
* **Bases de datos centralizadas:** Consulta múltiples fuentes de vulnerabilidades actualizadas en tiempo real (NVD, GitHub Advisories, distros Linux, etc.).

---

## 2. CheatSheet Profesional de Comandos y Flags

A continuación se recogen los comandos más eficientes para el trabajo diario del analista, acompañados de sus parámetros avanzados.

### 2.1. Escaneo de Imágenes de Contenedores
Ideal para analizar la seguridad de imágenes Docker antes de su despliegue en producción.

```bash
# Escaneo básico de una imagen local o remota
trivy image python:3.9-alpine

# Escaneo filtrando únicamente vulnerabilidades de alta y crítica severidad
trivy image --severity HIGH,CRITICAL nginx:latest

# Escaneo omitiendo la actualización automática de la base de datos (ideal para entornos aislados o CI/CD rápido)
trivy image --skip-db-update my-custom-app:v1

# Escaneo incluyendo la detección de vulnerabilidades en paquetes de lenguajes de programación (OS packages + Application dependencies)
trivy image --scanners vuln,config,secret node:16
```

### 2.2. Escaneo de Sistemas de Archivos y Repositorios
Permite auditar el código fuente de un proyecto o un directorio local en busca de fallos de dependencias.

```bash
# Analizar el directorio actual en busca de vulnerabilidades en dependencias (npm, pip, composer, maven, etc.)
trivy fs /path/to/project

# Detectar secretos hardcodeados (contraseñas, API keys, tokens) junto con vulnerabilidades
trivy fs --scanners vuln,secret .
```

### 2.3. Escaneo de Infraestructura como Código (IaC) y Configuración
Fundamental para identificar malas configuraciones en despliegues de Kubernetes, Terraform o Dockerfiles.

```bash
# Analizar ficheros de configuración de Terraform o Kubernetes en un directorio
trivy config /path/to/terraform/configs

# Mostrar únicamente los resultados que supongan un fallo de seguridad activo
trivy config --exit-code 1 /path/to/k8s/manifests
```

### 2.4. Generación de Informes y SBOM (Software Bill of Materials)
Para documentar hallazgos de cara a informes técnicos o auditorías de cumplimiento.

```bash
# Exportar el resultado del escaneo en formato JSON para integrarlo en herramientas de gestión (DefectDojo, Faraday)
trivy image --format json -o trivy-report.json python:3.4

# Generar un SBOM estandarizado en formato CycloneDX (JSON)
trivy image --format cyclonedx -o sbom.json my-app:latest
```

---

## 3. Caso Práctico: Análisis y Detección de una Imagen Vulnerable

Para demostrar el criterio técnico en una entrevista o portfolio, simularemos un flujo de análisis real sobre una imagen intencionadamente vulnerable.

### Paso 1: Lanzamiento del escaneo con enfoque forense
Ejecutamos Trivy sobre una versión antigua de una imagen base que contiene múltiples CVEs conocidas y problemas de configuración:

```bash
trivy image --severity CRITICAL python:3.8-slim
```

### Paso 2: Interpretación del Output (Crash / Triage)
El output estructurado de Trivy nos devuelve información crítica dividida por secciones:
1. **Target:** Nombre de la imagen y sistema operativo base (ej. Debian 10).
2. **Library / Vulnerability ID:** Identificador único (ej. `CVE-2021-33574`).
3. **Severity:** Clasificación del riesgo (`CRITICAL`, `HIGH`).
4. **Installed Version vs Fixed Version:** Indica la versión actual instalada en el paquete vulnerable y la versión parcheada recomendada para mitigar el fallo.

### Paso 3: Triaje y Mitigación
* **Acción inmediata:** Actualizar la imagen base del Dockerfile a una versión con soporte activo y parches aplicados (ej. pasar de `python:3.8-slim` a `python:3.11-slim`).
* **Acción compensatoria:** Si la vulnerabilidad reside en una dependencia que no se puede actualizar, aplicar restricciones en tiempo de ejecución o aislar el contenedor mediante perfiles de seguridad estrictos (AppArmor / Seccomp).

---

## 4. Limitaciones y Comparativa frente a Otras Herramientas

* **Frente a Escáneres de Red tradicionales (Nessus / OpenVAS):** Trivy está enfocado puramente en artefactos de software, contenedores y dependencias a nivel de fichero/imagen, mientras que Nessus evalúa hosts de red e infraestructura IP activa. Se complementan perfectamente en una auditoría integral.
* **Limitación principal:** Requiere conectividad inicial para descargar la base de datos de vulnerabilidades (aunque permite el uso de bases de datos cacheadas o descargadas offline para entornos corporativos sin salida a Internet).

---

## 5. Recomendaciones de Bastionado (DevSecOps)

Para evitar que estas vulnerabilidades lleguen a entornos de producción, se recomienda:
1. **Shift-Left Security:** Integrar Trivy directamente en el pipeline de CI/CD (GitHub Actions, GitLab CI) haciendo que el build falle (`--exit-code 1`) si se detectan vulnerabilidades críticas.
2. **Escaneos periódicos:** Programar escaneos diarios de las imágenes almacenadas en el registro corporativo (Docker Hub, AWS ECR, Harbor) ya que cada día se descubren nuevas CVEs en librerías ya desplegadas.
