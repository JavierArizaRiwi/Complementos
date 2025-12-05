# Complemento de Observabilidad Avanzada: Loki + Promtail + Grafana

Este archivo complementa un sistema de monitoreo basado en **Prometheus + Grafana**, agregando **logs centralizados** mediante Loki y Promtail. Está diseñado para principiantes, con un enfoque paso a paso.

---

## Objetivo

El objetivo es agregar observabilidad completa al proyecto, integrando:

- **Métricas:** Prometheus.
- **Logs centralizados:** Loki.
- **Recolección de logs:** Promtail.
- **Visualización unificada:** Grafana.

El flujo final será:

```
Spring Boot → Promtail → Loki → Grafana
```

---

## Estructura del Proyecto

La estructura del proyecto debe ser la siguiente:

```
pagos/
  docker-compose.yml
  Dockerfile
  pom.xml
  prometheus/
    prometheus.yml
  loki/
    loki-config.yml
  promtail/
    promtail-config.yml
  logs/              # Carpeta creada automáticamente por Docker
  src/
    main/java/...
    main/resources/application.yml
```

---

## 1. Configuración de Logs en Spring Boot

En el archivo `src/main/resources/application.yml`, agrega la siguiente configuración:

```yaml
logging:
  file:
    name: /var/log/pagos/app.log
  pattern:
    file: "%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n"
```

Esto permite que Promtail pueda leer los logs generados por la aplicación.

---

## 2. Configuración de Loki

Crea el archivo `loki/loki-config.yml` con el siguiente contenido:

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache
  filesystem:
    directory: /loki/chunks

limits_config:
  ingestion_rate_mb: 4
  ingestion_burst_size_mb: 6

chunk_store_config:
  max_look_back_period: 7d

table_manager:
  retention_deletes_enabled: true
  retention_period: 7d
```

---

## 3. Configuración de Promtail

Crea el archivo `promtail/promtail-config.yml` con el siguiente contenido:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: pagos-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: pagos
          __path__: /var/log/pagos/*.log
```

---

## 4. docker-compose.yml (Versión Extendida)

El archivo `docker-compose.yml` debe incluir los siguientes servicios:

```yaml
services:
  app:
    build:
      context: .
    container_name: pagos-app
    ports:
      - "8080:8080"
    volumes:
      - ./logs:/var/log/pagos
    environment:
      SPRING_PROFILES_ACTIVE: h2

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus:/etc/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    container_name: grafana
    depends_on:
      - prometheus
      - loki
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana

  loki:
    image: grafana/loki:2.9.0
    container_name: loki
    volumes:
      - ./loki:/etc/loki
    command: -config.file=/etc/loki/loki-config.yml
    ports:
      - "3100:3100"

  promtail:
    image: grafana/promtail:2.9.0
    container_name: promtail
    volumes:
      - ./promtail/promtail-config.yml:/etc/promtail/promtail-config.yml
      - ./logs:/var/log/pagos
    command: -config.file=/etc/promtail/promtail-config.yml

volumes:
  grafana-storage:
```

---

## 5. Levantar los Servicios

Para iniciar todos los servicios, ejecuta los siguientes comandos:

```bash
docker compose down --remove-orphans
docker compose up --build
```

---

## 6. Acceder a los Servicios

| Servicio     | URL                      |
|--------------|--------------------------|
| Aplicación   | http://localhost:8080    |
| Prometheus   | http://localhost:9090    |
| Grafana      | http://localhost:3000    |
| Loki API     | http://localhost:3100    |

---

## 7. Configurar Loki en Grafana

1. Accede a Grafana en `http://localhost:3000`.
2. Inicia sesión con las credenciales predeterminadas: `admin / admin`.
3. Ve a `Connections` → `Data sources`.
4. Haz clic en `Add data source`.
5. Selecciona **Loki**.
6. Configura la URL como:

```
http://loki:3100
```

7. Guarda y prueba la configuración.

---

## 8. Consultar Logs en Grafana

1. En el menú izquierdo, selecciona **Explore**.
2. Elige **Loki** como fuente de datos.
3. Ejecuta la siguiente consulta:

```
{job="pagos"}
```

Esto mostrará los logs en tiempo real.

---

## 9. Dashboard Recomendado

Para una mejor visualización, importa el siguiente dashboard oficial:

- **Dashboard ID:** 13639
- **Nombre:** "Spring Boot Logs + Metrics"

---

## Conclusión

Con este complemento, tu ecosistema de observabilidad ahora incluye:

- **Métricas:** Prometheus.
- **Logs:** Loki y Promtail.
- **Dashboards:** Grafana.

Este sistema es ideal para microservicios, entornos de producción o educativos.