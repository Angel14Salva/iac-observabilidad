# Laboratorio de Observabilidad — Angel Salva

**Curso:** Infraestructura como Código  
**Repositorio:** https://github.com/Angel14Salva/iac-observabilidad

---

## Instrucciones para validar el trabajo

### 1. Prerrequisitos
- Docker y Docker Compose instalados
- Puertos libres: `3000`, `3001`, `3100`, `8080`, `8081`, `9090`, `9100`, `12345`

### 2. Levantar el stack
```bash
git clone https://github.com/Angel14Salva/iac-observabilidad
cd iac-observabilidad
docker compose up -d --build
docker compose ps
```

### 3. Verificar servicios
| Servicio | URL |
|---|---|
| Frontend | http://localhost:8080 |
| Backend | http://localhost:3001/metrics |
| Grafana | http://localhost:3000 (admin/admin) |
| Prometheus | http://localhost:9090 |
| Alloy | http://localhost:12345 |

### 4. Verificar datasources en Grafana
- Ir a **Connections → Data sources**
- Confirmar que **Prometheus** y **Loki** están en verde

### 5. Verificar dashboard
- Ir a **Dashboards → Observabilidad — Angel Salva**
- Confirmar los 4 paneles con datos: CPU backend, CPU host, Logs aplicación, Logs infraestructura

### 6. Probar la alarma
```bash
curl "http://localhost:3001/load?seconds=120"
```
- Ir a **Alerting → Alert rules → CPU backend > 50%**
- Esperar transición: Normal → Pending → Firing

---

## Respuestas a las preguntas del laboratorio

### 1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?
Prometheus está diseñado para métricas numéricas en el tiempo (CPU, memoria, contadores), no para texto libre. Los logs son eventos textuales no estructurados como errores, trazas y mensajes de negocio que Prometheus no puede almacenar ni consultar eficientemente. Loki está optimizado para indexar y buscar flujos de logs por etiquetas, permitiendo correlacionar eventos textuales con las métricas de Prometheus en el mismo dashboard de Grafana.

### 2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código?
Al definir los datasources en un archivo `datasources.yml` versionado, la configuración es reproducible y no depende de que alguien recuerde hacer clic en la interfaz. Cualquier persona que clone el repositorio y ejecute `docker compose up` obtiene exactamente el mismo entorno de observabilidad, sin pasos manuales adicionales. Esto elimina el riesgo de errores humanos y facilita el trabajo en equipo.

### 3. ¿Por qué el panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos? ¿Cuál usarías para alertar sobre una aplicación concreta?
El panel de CPU del host muestra el uso agregado de todos los procesos del sistema operativo, incluyendo el kernel, otros contenedores y procesos del sistema. El panel de CPU por contenedor muestra únicamente el consumo de ese contenedor específico, aislado del resto. Para alertar sobre una aplicación concreta usaría el panel de CPU por contenedor, porque permite detectar si esa aplicación específica está consumiendo recursos en exceso sin que el host completo esté saturado.

### 4. ¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?
El **evaluation interval** es cada cuánto tiempo Grafana evalúa la condición de la alarma (en este lab, cada 10 segundos). El **pending period** es cuánto tiempo debe mantenerse la condición verdadera de forma continua antes de pasar a estado Firing (en este lab, 30 segundos). Esto evita falsas alarmas por picos momentáneos: si la CPU sube por 5 segundos y baja, la alarma no se dispara; solo se dispara si se mantiene sobre el umbral durante al menos 30 segundos consecutivos.

---

## Notas sobre adaptaciones en GitHub Codespaces

Durante el desarrollo se encontraron las siguientes diferencias respecto al entorno local:

- **node-exporter:** se eliminó el volumen `/:/host:ro,rslave` y el `pid: host` porque Codespaces no permite montar el filesystem raíz del host. Las métricas de node-exporter reflejan el entorno del contenedor.
- **cAdvisor:** no expone el label `name` en las métricas, solo `id`. La query de CPU del contenedor se adaptó a `{id="/docker"}`.
- **Loki:** el archivo `loki-config.yaml` fue recreado manualmente porque Docker generó un directorio en su lugar durante el primer arranque.
- **Datasources:** la carpeta `grafana/provisioning/datasources/` no existía en el repositorio original y fue creada manualmente.
