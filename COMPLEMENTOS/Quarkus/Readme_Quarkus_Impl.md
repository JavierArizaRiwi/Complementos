# Guía práctica: Quarkus para desarrollo backend moderno

Esta guía explica **qué es Quarkus**, cómo funciona, su arquitectura interna, sus ventajas/desventajas, comparativa con Spring Boot y cómo iniciar un proyecto real.  
Pensada como un README técnico para ti y tus coders.

---

# 1. ¿Qué es Quarkus?

Quarkus es un **framework moderno para Java**, diseñado para:

- **Arranques ultrarrápidos**  
- **Bajo consumo de memoria**  
- **Optimización para contenedores (Docker/Kubernetes)**  
- **Compilación nativa** con GraalVM  
- **Programación reactiva y asincrónica**  
- **Microservicios cloud-native**

Su lema oficial es:

> *“Supersonic Subatomic Java”*  
(Supersónico y subatómico en consumo de RAM)

Quarkus fue creado por Red Hat y es el framework preferido por muchos equipos que trabajan en Kubernetes/OpenShift, microservicios y funciones serverless.

---

# 2. ¿Por qué usar Quarkus?

## 2.1. Arranques MUY rápidos

Ejemplo típico:

| Framework | Tiempo de arranque |
|----------|---------------------|
| Spring Boot | 1.5–3 segundos |
| Quarkus JVM | 0.1–0.3 segundos |
| Quarkus nativo | 0.01–0.03 segundos |

## 2.2. Bajo consumo de memoria

| Framework | RAM típica |
|----------|------------|
| Spring Boot | 300–600 MB |
| Quarkus JVM | 100–200 MB |
| Quarkus nativo | 20–50 MB |

Esto lo hace perfecto para:

- Kubernetes
- Lambdas/FaaS
- Microservicios de alta densidad

## 2.3. Integración nativa con estándares cloud

Quarkus trae soporte first-class para:

- RESTEasy Reactive (REST)  
- Mutiny (reactivo)  
- Hibernate ORM + Panache  
- Hibernate Reactive  
- R2DBC  
- Kafka / AMQP  
- OpenAPI / Swagger  
- MicroProfile  
- GraphQL  
- Qute (motor de plantillas)

---

# 3. Arquitectura de Quarkus

Quarkus está construido con dos pilares:

## 3.1. Build-time processing (procesamiento en tiempo de compilación)

Quarkus **mueve trabajo del runtime al build-time**, reduciendo uso de CPU y memoria en producción.

Ejemplos:

- Generación de proxies  
- Escaneo de clases  
- Preparación de configuraciones  
- Registro estático para GraalVM  
- Validaciones

## 3.2. EXTENSIONES (el corazón de Quarkus)

Cada funcionalidad (ORM, Kafka, HTTP, etc.) está encapsulada en **extensiones**.

Ejemplos:

| Extensión | Función |
|----------|---------|
| quarkus-resteasy-reactive | API REST |
| quarkus-hibernate-orm-panache | ORM simplificado |
| quarkus-mongodb-panache | MongoDB |
| quarkus-smallrye-openapi | OpenAPI |
| quarkus-smallrye-reactive-messaging-kafka | Kafka |
| quarkus-redis-client | Redis |

Las extensiones permiten que Quarkus optimice cada módulo para rendimiento.

---

# 4. Crear un proyecto Quarkus

### 4.1. Usando Maven

```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.9.0:create \
    -DprojectGroupId=com.riwitienda \
    -DprojectArtifactId=pagos-quarkus \
    -DclassName="com.riwitienda.pagos.HolaResource" \
    -Dpath="/hola"
```

### 4.2. Usando CLI de Quarkus

```bash
quarkus create app com.riwitienda:pagos-quarkus
```

---

# 5. Estructura básica del proyecto

```text
src/
 ├─ main/java
 │    └─ com/riwitienda/pagos
 │         ├─ HolaResource.java
 ├─ main/resources
 │    ├─ application.properties
 └─ test/java
```

Archivo REST básico:

```java
@Path("/hola")
public class HolaResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hola() {
        return "Hola desde Quarkus!";
    }
}
```

---

# 6. Modo dev ultrarrápido

Uno de los puntos fuertes de Quarkus:

```bash
mvn quarkus:dev
```

- Recarga **instantánea**  
- Logs inteligentes  
- Dev UI (`http://localhost:8080/q/dev/`)  
- Vista de rutas, health checks, métricas, etc.

---

# 7. Programación Reactiva con Mutiny

Quarkus usa **Mutiny**, una API reactiva más simple que Reactor.

Tipos principales:

- `Uni<T>` → 0..1 elemento (similar a `Mono<T>`)
- `Multi<T>` → 0..N elementos (similar a `Flux<T>`)

Ejemplo:

```java
@GET
@Path("/reactivo")
@Produces(MediaType.TEXT_PLAIN)
public Uni<String> reactivo() {
    return Uni.createFrom().item("Hola reactivo!");
}
```

---

# 8. Quarkus + RESTEasy Reactive (REST avanzado)

```java
@Path("/usuarios")
public class UsuarioResource {

    @Inject
    UsuarioService service;

    @GET
    public Multi<UsuarioResponse> listar() {
        return service.listar();
    }

    @GET
    @Path("/{id}")
    public Uni<Response> obtener(@PathParam("id") Long id) {
        return service.obtener(id)
                .onItem().ifNotNull().transform(Response::ok)
                .onItem().ifNull().continueWith(Response.status(404).build())
                .map(ResponseBuilder::build);
    }
}
```

---

# 9. Persistencia con Panache ORM

Panache simplifica Hibernate:

### Entidad:

```java
@Entity
public class Usuario extends PanacheEntity {
    public String nombre;
    public String email;
}
```

### Servicio:

```java
@ApplicationScoped
public class UsuarioService {

    public List<Usuario> listar() {
        return Usuario.listAll();
    }

    public Usuario crear(Usuario u) {
        u.persist();
        return u;
    }
}
```

### Reactivo (Hibernate Reactive + Mutiny)

```java
@Entity
public class Usuario extends PanacheEntityBase {
    public String nombre;
    public String email;
}
```

Servicio:

```java
public Uni<List<Usuario>> listar() {
    return Usuario.listAll();
}
```

---

# 10. Configuración (`application.properties`)

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=pagos_user
quarkus.datasource.password=pagos_pass
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/pagosdb

quarkus.hibernate-orm.database.generation=update

quarkus.http.port=8080
```

---

# 11. Quarkus vs Spring Boot

### Comparativa rápida

| Característica | Spring Boot | Quarkus |
|----------------|-------------|---------|
| Arranque | Lento–medio | Muy rápido |
| RAM | Alta | Baja |
| Reactivo | Reactor/WebFlux | Mutiny/RESTEasy Reactive |
| Compilación nativa | Soporte parcial | Soporte first-class |
| Ecosistema | Enorme | Creciendo rápido |
| Facilidad | Más simple | Más técnico |
| Ideal para | Cualquier app | Microservicios cloud-native |

### ¿Quién gana?

- Para **backend empresarial grande** → Spring Boot  
- Para **microservicios en Kubernetes / cloud** → **Quarkus**  
- Para **lambda/functions** → Quarkus nativo  
- Para **altísima densidad de pods** → Quarkus nativo o JVM

---

# 12. Dockerizar un servicio Quarkus

```dockerfile
FROM quay.io/quarkus/quarkus-micro-image:1.0
COPY target/*-runner.jar /application.jar
EXPOSE 8080
CMD ["java", "-jar", "/application.jar"]
```

Build:

```bash
mvn package -Dquarkus.package.type=uber-jar
docker build -t pagos-quarkus .
```

---

# 13. Compilación nativa (GraalVM)

```bash
mvn package -Pnative
```

Ejemplo de performance:

- Arranque: **15 ms**  
- RAM: **25 MB**

---

# 14. Ventajas y desventajas de Quarkus

### Ventajas

- Rapidísimo arranque  
- Muy bajo consumo de recursos  
- Perfecto para contenedores  
- Soporte nativo para MicroProfile  
- API reactiva elegante (Mutiny)  
- Dev mode muy rápido  
- Extensiones optimizadas

### Desventajas

- Curva de aprendizaje si vienes de Spring  
- Comunidad más pequeña  
- Menos ejemplos y tutoriales  
- Ecosistema menos maduro que Spring (aunque muy sólido)

---

# 15. ¿Cuándo usar Quarkus en tu organización?

Recomendado si:

- Usas Kubernetes / OpenShift  
- Quieres apps extremadamente livianas  
- Alto consumo de microservicios  
- Integración con Kafka u otras plataformas reactivas  
- Necesitas cold starts muy rápidos en serverless

No recomendado si:

- Todo tu stack está en Spring Boot y cambiar rompería uniformidad  
- Necesitas librerías o frameworks que solo existen en Spring  
- Tu equipo tiene poca experiencia con programación reactiva

---

# 16. Proyecto base recomendado para tus coders

Estructura sugerida:

```text
src/main/java
 ├─ domain
 │   ├─ model
 │   └─ service
 ├─ application
 │   ├─ dto
 │   └─ mappers
 └─ infrastructure
      ├─ resource       (REST)
      ├─ persistence    (Panache)
      ├─ config
      └─ messaging
```

Este estilo mezcla:

- Clean Architecture  
- Ports & Adapters  
- Mutiny  
- Panache  

---

# 17. Resumen final

- Quarkus es ideal para **microservicios cloud-native**, apps reactivas y contenedores.  
- Arranca rápido, consume poca RAM y tiene integración nativa con GraalVM.  
- Mutiny ofrece un modelo reactivo más intuitivo que Reactor.  
- Panache simplifica la persistencia.  
- Es muy competitivo frente a Spring Boot en entornos modernos.  

---

¿Quieres también una versión PDF, un demo completo Quarkus + PostgreSQL + REST API o una arquitectura hexagonal con Quarkus?