# Guía Completa: Quarkus

### Explicación general y luego específica para desarrollo backend moderno, cloud-native y de alto rendimiento

Esta guía presenta una explicación completa, clara y profesional de
**Quarkus**, su filosofía, por qué existe, cómo funciona internamente,
sus ventajas sobre frameworks tradicionales como Spring, y cómo
aplicarlo adecuadamente en microservicios modernos.

------------------------------------------------------------------------

# 1. ¿Qué es Quarkus?

Quarkus es un framework Java orientado a crear aplicaciones:

-   Nativas de la nube (cloud‑native)
-   Ultra rápidas
-   De bajo consumo
-   Preparadas para GraalVM
-   Optimizadas para contenedores (Docker / Kubernetes)
-   Muy eficientes en tiempo de inicio y memoria

Fue creado por Red Hat y está diseñado para microservicios modernos y
entornos serverless.

Su slogan oficial: **"Supersonic Subatomic Java"**

------------------------------------------------------------------------

# 2. ¿Por qué existe Quarkus?

Java tradicionalmente tiene tres problemas en el mundo cloud:

1.  Alto tiempo de inicio de aplicaciones\
2.  Alto consumo de memoria\
3.  No está optimizado para contenedores y serverless

Quarkus fue creado para resolverlos.

## 2.1 Problema: Tiempo de arranque

Spring Boot o Jakarta EE requieren cargar:

-   Contenedores Servlet
-   Reflection
-   Proxies
-   Clases dinámicas

Esto hace que iniciar la app tome entre **2--10 segundos**.

Quarkus puede arrancar en **20 a 50 ms**.

------------------------------------------------------------------------

## 2.2 Problema: Consumo de memoria

Una app Spring Boot puede consumir:

-   200--400 MB RAM.

La misma app en Quarkus:

-   20--60 MB RAM.

------------------------------------------------------------------------

## 2.3 Problema: Cloud y Serverless

En AWS Lambda, Google Cloud Run o Kubernetes:

-   El escalado requiere apps que inicien rápido.
-   Pagar por segundo favorece apps ligeras.
-   Se necesitan más pods con menos RAM.

Quarkus fue diseñado específicamente para esto.

------------------------------------------------------------------------

# 3. ¿Qué hace a Quarkus tan rápido?

## 3.1 Build-time Processing

Quarkus mueve el trabajo pesado al **momento de compilación**, no al
tiempo de ejecución.

Ejemplo: - Spring crea proxies al ejecutar - Quarkus los genera al
compilar

Resultado: - Menor uso de CPU - Arranques ultrarrápidos - Menos
necesidad de reflexión

------------------------------------------------------------------------

## 3.2 GraalVM Native Image

Quarkus puede compilar aplicaciones Java a ejecutables nativos.

Ventajas:

-   Ejecutable sin JVM
-   Inicio en milisegundos
-   Bajo consumo de memoria
-   Ideal para Lambdas y Kubernetes

------------------------------------------------------------------------

## 3.3 Runtime optimizado (Netty)

Quarkus usa un stack reactivo basado en Netty:

-   No bloqueante
-   Alto rendimiento
-   Similar a Spring WebFlux

------------------------------------------------------------------------

# 4. Arquitectura y Filosofía

Quarkus combina lo mejor de dos mundos:

### 1. Imperativo (estilo Spring MVC)

-   Fácil de entender
-   Ideal para CRUD y lógica clásica

### 2. Reactivo (Mutiny)

-   Basado en eventos
-   Ideal para concurrencia elevada

Puedes mezclar ambos en el mismo proyecto.

------------------------------------------------------------------------

# 5. Extensiones de Quarkus

Quarkus funciona mediante **extensiones**, que integran tecnologías
externas de forma altamente optimizada.

Ejemplos:

-   RESTEasy Reactive (REST API)
-   Hibernate ORM / Panache
-   Hibernate Reactive
-   R2DBC
-   Mutiny (API reactiva)
-   Kafka
-   gRPC
-   OpenAPI / Swagger UI
-   Scheduler (cron)
-   Security / JWT
-   Redis
-   REST Client

Las extensiones reemplazan configuraciones pesadas de frameworks
tradicionales.

------------------------------------------------------------------------

# 6. Comparación: Quarkus vs Spring Boot

  -----------------------------------------------------------------------
  Característica                Spring Boot              Quarkus
  ----------------------------- ------------------------ ----------------
  Filosofía                     Runtime-heavy            Build-time heavy

  Tiempo de inicio              Segundos                 Milisegundos

  RAM promedio                  200--400 MB              20--60 MB

  Ideal para                    Monolitos y              Microservicios y
                                microservicios           serverless

  Nativo GraalVM                Sí, con mucho esfuerzo   Sí, muy fácil

  Stack                         Imperativo               Imperativo +
                                                         Reactivo

  Curva de aprendizaje          Muy baja                 Media
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 7. Quarkus Imperativo (REST básico)

Ejemplo:

``` java
@Path("/pagos")
public class PagoResource {

    @GET
    @Path("/{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Pago obtenerPago(@PathParam("id") Long id) {
        return new Pago(id, "OK");
    }
}
```

------------------------------------------------------------------------

# 8. Quarkus Reactivo con Mutiny

Mutiny es el modelo reactivo de Quarkus:

-   `Uni<T>` → 0..1 elementos (equivalente a Mono)
-   `Multi<T>` → 0..N elementos (equivalente a Flux)

Ejemplo reactivo:

``` java
@GET
@Path("/stream")
@Produces(MediaType.SERVER_SENT_EVENTS)
public Multi<Integer> stream() {
    return Multi.createFrom().range(1, 10);
}
```

------------------------------------------------------------------------

# 9. Persistencia en Quarkus

## 9.1 Panache (imperativo)

ORM simplificado:

``` java
@Entity
public class Pago extends PanacheEntity {
    public BigDecimal monto;
}
```

------------------------------------------------------------------------

## 9.2 Panache Reactive

ORM reactivo sin bloqueo:

``` java
public Uni<Pago> findPago(Long id) {
    return Pago.findById(id);
}
```

------------------------------------------------------------------------

# 10. Quarkus con GraalVM Native Image

Compilar a binario nativo:

``` bash
./mvnw package -Pnative
```

Ejecutar:

``` bash
./target/mi-app-runner
```

Ventajas:

-   Inicio casi inmediato
-   Menor latencia
-   Mejor ajuste en contenedores

------------------------------------------------------------------------

# 11. ¿Cuándo usar Quarkus?

### Recomendado para:

-   Microservicios cloud‑native
-   Sistemas con alta concurrencia
-   Lambdas AWS, Cloud Run
-   APIs muy rápidas
-   Recursos limitados (RAM/CPU)
-   Grandes despliegues en Kubernetes

### No recomendado para:

-   Equipos muy dependientes de Spring
-   Aplicaciones monolíticas con cientos de librerías incompatibles
-   Proyectos donde no se necesite alta performance

------------------------------------------------------------------------

# 12. Beneficios Clave

1.  Arranque ultrarrápido\
2.  Bajo consumo de memoria\
3.  Mejor para entornos cloud\
4.  Compatible con code quarkus.io (generador oficial)\
5.  Fácil integración con Kafka, Mongo, Redis\
6.  Soporte nativo para GraalVM\
7.  Mutiny es más legible que Reactor

------------------------------------------------------------------------

# 13. Ejemplo real aplicado: Microservicio de Pagos

### Imperativo:

-   API REST\
-   Panache/JPA\
-   Validación\
-   Transacciones

### Reactivo:

-   Streaming de pagos en tiempo real\
-   Servicio de eventos con Kafka\
-   Acceso a BD reactiva\
-   SSE para dashboards

### Nativo:

-   Despliegue en Cloud Run\
-   Arranque inmediato\
-   Costos drásticamente reducidos

------------------------------------------------------------------------

# 14. Conclusión

Quarkus es un framework moderno, altamente optimizado para ecosistemas
cloud y microservicios.

Ofrece:

-   Alto rendimiento\
-   Bajo consumo\
-   Opciones nativas\
-   Soporte imperativo y reactivo\
-   Integración fluida con tecnologías modernas

Es una alternativa poderosa a Spring Boot para arquitecturas
cloud‑native, serverless y de alta concurrencia.