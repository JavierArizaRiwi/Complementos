# Guía Completa: Spring Boot Actuator, Micrometer/Prometheus y Grafana

### Qué son, para qué sirven y cómo trabajan juntos

Esta guía explica de manera clara y práctica la diferencia entre
Actuator, Micrometer / Prometheus y Grafana, y cómo se complementan para
monitorear aplicaciones Spring Boot de forma profesional.

------------------------------------------------------------------------

# 1. Spring Boot Actuator

## ¿Qué es?

Spring Boot Actuator es un módulo integrado que expone información
interna del estado y funcionamiento de tu aplicación Spring Boot.

Es el panel de control interno del microservicio.

## ¿Qué puede mostrar Actuator?

Actuator expone endpoints especiales con datos como:

### Estado de salud

-   `/actuator/health`\
-   Muestra si la aplicación está UP o DOWN\
-   Ideal para Kubernetes, Docker, Load Balancers

### Métricas del sistema

-   `/actuator/metrics`
-   CPU, memoria, heap, threads\
-   Tiempo de respuesta por endpoint\
-   Total de peticiones HTTP\
-   Errores 4xx / 5xx\
-   Pool de conexiones JDBC (Hikari)

### Información del entorno

-   Beans cargados\
-   Versiones\
-   Properties expuestas (seguras)

### Ejemplo de respuesta `/actuator/health`

``` json
{
  "status": "UP"
}
```

Actuator expone datos crudos, no visualizaciones.\
Para gráficos, se necesita Prometheus + Grafana.

------------------------------------------------------------------------

# 2. Micrometer + Prometheus

Micrometer es un traductor de métricas.\
Prometheus es una base de datos especializada en métricas.

Ambos trabajan juntos.

------------------------------------------------------------------------

# 2.1 Micrometer

## ¿Qué es?

Micrometer toma todas las métricas que Actuator genera y las transforma
en un formato que herramientas externas puedan entender.

## ¿Qué traduce Micrometer?

-   Métricas de Actuator\
-   Métricas de la JVM\
-   Métricas HTTP\
-   Métricas del pool JDBC\
-   Métricas personalizadas

Micrometer soporta múltiples backends: - Prometheus\
- Datadog\
- New Relic\
- Graphite\
- CloudWatch

En este caso: Prometheus.

------------------------------------------------------------------------

# 2.2 Prometheus

## ¿Qué es?

Prometheus es un servidor de métricas que:

1.  Se conecta a la app Spring Boot\
2.  Raspa (scrapea) métricas desde `/actuator/prometheus`\
3.  Las guarda en una base de datos de series de tiempo

## ¿Qué guarda Prometheus?

Métricas numéricas, como: - Latencias por endpoint\
- Requests por segundo\
- Errores 4xx / 5xx\
- Uso de heap y non-heap\
- Cantidad de hilos\
- Consumo de CPU\
- Cantidad de GC\
- Tiempo por operación\
- Métricas personalizadas de negocio

Prometheus NO guarda: - Logs\
- Excepciones completas\
- Texto

Para logs se usa Loki.

### Ejemplo de métrica que Prometheus almacena:

    http_server_requests_seconds_count{uri="/pago",status="200"} 55

------------------------------------------------------------------------

# 3. Grafana

## ¿Qué es?

Grafana es una plataforma visual de monitoreo.

No almacena datos.\
Los visualiza desde:

-   Prometheus (métricas)
-   Loki (logs)
-   MySQL / PostgreSQL
-   ElasticSearch
-   CloudWatch

## ¿Qué puedes hacer con Grafana?

### Dashboards

-   CPU, memoria, threads\
-   Requests por segundo\
-   Latencia de endpoints\
-   Errores por tipo\
-   Estado del microservicio

### Logs en tiempo real

Solo si integras Loki.

### Alertas

Ejemplos: - Si los errores 500 superan 10 en 1 minuto\
- Si la latencia supera los 300 ms\
- Si el heap pasa del 85 %

### Query avanzado (PromQL)

Permite consultar métricas almacenadas de forma histórica.

------------------------------------------------------------------------

# 4. Cómo trabajan juntos

Arquitectura general:

    [Spring Boot Actuator]
           |
           v
    [Micrometer]
           |
           v
    [Prometheus]
           |
           v
    [Grafana]

Explicación:

-   Actuator genera métricas y health checks.\
-   Micrometer convierte estas métricas al formato Prometheus.\
-   Prometheus las guarda como series de tiempo.\
-   Grafana las visualiza y permite alertas.

------------------------------------------------------------------------

# 5. ¿Qué puedes hacer con esta combinación?

### Saber si el microservicio está sano

Con dashboards de health.

### Medir rendimiento real

-   Tiempo de respuesta\
-   Latencias p95/p99\
-   Errores 500

### Optimizar la aplicación

-   Identificación de endpoints lentos\
-   Cuellos de botella\
-   Uso de memoria y CPU de la JVM

### Monitorear la base de datos

-   Conexiones activas\
-   Latencia de queries (si se exponen métricas)

### Ver logs (solo si integras Loki)

Grafana permite leer logs en tiempo real.

------------------------------------------------------------------------

# 6. Diferencias entre Actuator, Prometheus y Grafana

  --------------------------------------------------------------------------
  Herramienta          Qué es         Tipo de datos            Función
  -------------------- -------------- ------------------------ -------------
  Actuator             Módulo interno JSON                     Exponer salud
                       de Spring Boot                          y métricas

  Micrometer           Librería       Métricas estandarizadas  Traducir
                       puente                                  métricas a
                                                               formato
                                                               Prometheus

  Prometheus           Base de datos  Métricas numéricas       Guardar
                                                               series de
                                                               tiempo

  Grafana              Visualizador   Dashboards y logs        Mostrar
                                                               métricas y
                                                               crear alertas
  --------------------------------------------------------------------------

------------------------------------------------------------------------

# 7. Ejemplo aplicado al microservicio de pagos

### Actuator

-   `/actuator/health` → estado\
-   `http.server.requests` → total de pagos\
-   Métricas de H2 o MySQL\
-   Excepciones

### Prometheus

-   350 pagos en 5 minutos\
-   Latencia promedio: 185 ms\
-   Errores 500: 2

### Grafana

-   Dashboard con:
    -   Latencia\
    -   Throughput\
    -   Errores\
    -   Health\
-   Logs en tiempo real (si usas Loki)

------------------------------------------------------------------------

# 8. Conclusión

-   Actuator es el motor de información.\
-   Micrometer convierte esa información a un estándar.\
-   Prometheus guarda las métricas.\
-   Grafana las visualiza y permite alertas.

Esta es la arquitectura de monitoreo usada por empresas como Netflix,
Spotify, Mercado Libre y Amazon.