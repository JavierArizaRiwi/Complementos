# Guía práctica: Programación Reactiva con Spring WebFlux

Esta guía explica de forma práctica qué es la **programación reactiva**, cómo se implementa con **Spring WebFlux** y cuándo tiene sentido usarla en lugar del stack tradicional bloqueante de Spring MVC.

La idea es que te sirva como README para tu repo o como material de estudio para tus coders.

---

## 1. ¿Qué es la programación reactiva?

La programación reactiva se basa en la idea de **flujos de datos asíncronos** y **no bloqueantes**.

Conceptos clave:

- **Asíncrono**: no bloqueas hilos esperando respuestas; reaccionas cuando llegan.
- **No bloqueante**: los hilos pueden atender muchas peticiones porque no se quedan esperando I/O (DB, red, etc.).
- **Flujos de datos**: en lugar de pensar en valores individuales, piensas en **streams** de eventos.

El estándar en Java es **Reactive Streams**, soportado por librerías como:

- **Project Reactor** (usado por Spring WebFlux)
- **RxJava**
- **Akka Streams**, etc.

Reactive Streams define 4 interfaces fundamentales:

1. `Publisher<T>`
2. `Subscriber<T>`
3. `Subscription`
4. `Processor<T, R>`

Reactor provee dos implementaciones principales:

- `Mono<T>` → 0..1 elementos  
- `Flux<T>` → 0..N elementos

---

## 2. ¿Qué es Spring WebFlux?

**Spring WebFlux** es el stack web reactivo de Spring.

### 2.1. Diferencias con Spring MVC

| Característica      | Spring MVC (Servlet)              | Spring WebFlux (reactivo)         |
|---------------------|-----------------------------------|-----------------------------------|
| Modelo de concurrencia | Hilos por petición (bloqueante) | Event-loop, no bloqueante        |
| Tipo de controladores | `@RestController` tradicionales | `@RestController` + routers funcionales |
| Tipo de retorno     | `ResponseEntity`, objetos normales | `Mono<T>`, `Flux<T>`              |
| Performance         | Muy buena para cargas medias      | Excelente en I/O intensivo, alta concurrencia |
| Librería base       | Servlet API (Tomcat, Jetty…)      | Reactor + Netty / Servlet no bloqueante |

### 2.2. Cuándo usar WebFlux

**Tiene sentido usar WebFlux cuando:**

- Hay muchas operaciones de I/O (llamadas HTTP a otros servicios, DBs reactivas, colas, etc.).
- Necesitas alta escalabilidad con pocos recursos (microservicios con cientos/miles de requests concurrentes).
- Tu ecosistema soporta drivers **reactivos**: MongoDB Reactive, R2DBC, Redis Reactive, WebClient, etc.

**NO es buena idea usarlo cuando:**

- Casi todo tu stack es bloqueante (JDBC, librerías legacy, etc.).
- El tráfico es bajo/medio y la complejidad extra no compensa.
- El equipo aún no domina el modelo reactivo y los deadlines son muy ajustados.

---

## 3. Dependencias básicas para Spring WebFlux

En un proyecto **Spring Boot** con Maven:

```xml
<dependencies>
    <!-- WebFlux en lugar de spring-boot-starter-web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <!-- Para programación reactiva (Project Reactor viene transitivamente) -->
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-core</artifactId>
    </dependency>

    <!-- Si usas una base de datos reactiva, ej. MongoDB -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
    </dependency>

    <!-- Si usas R2DBC (bases relacionales reactivas) -->
    <!--
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-r2dbc</artifactId>
    </dependency>
    <dependency>
        <groupId>io.r2dbc</groupId>
        <artifactId>r2dbc-postgresql</artifactId> <!-- Ejemplo -->
    </dependency>
    -->
</dependencies>
```

> Importante: En un proyecto WebFlux NO se debe incluir `spring-boot-starter-web` (el de MVC tradicional), para evitar conflictos.

---

## 4. Tipos reactivos básicos: Mono y Flux

### 4.1. Mono

Representa **0 o 1 elemento**.

Ejemplos:

```java
Mono<String> mono = Mono.just("Hola");
Mono<String> vacio = Mono.empty();
Mono<String> error = Mono.error(new RuntimeException("Ups"));
```

### 4.2. Flux

Representa **0..N elementos** (un flujo).

```java
Flux<Integer> numeros = Flux.just(1, 2, 3, 4);
Flux<String> desdeLista = Flux.fromIterable(List.of("A", "B", "C"));
Flux<Long> intervalo = Flux.interval(Duration.ofSeconds(1)); // flujo infinito
```

### 4.3. Operadores comunes

- Transformación: `map`, `flatMap`  
- Filtrado: `filter`  
- Combinación: `zip`, `concat`, `merge`  
- Manejo de errores: `onErrorResume`, `onErrorReturn`  
- Suscripción terminal: `subscribe`, `block()` (solo para pruebas, no en código productivo reaccionando a requests)

Ejemplo:

```java
Flux<Integer> pares = Flux.range(1, 10)
    .filter(n -> n % 2 == 0)
    .map(n -> n * 10);
```

---

## 5. Controladores reactivos con anotaciones (`@RestController`)

Puedes seguir usando anotaciones al estilo MVC, pero devolviendo `Mono` o `Flux`.

```java
@RestController
@RequestMapping("/api/usuarios")
@RequiredArgsConstructor
public class UsuarioController {

    private final UsuarioService usuarioService;

    @GetMapping
    public Flux<UsuarioResponse> listar() {
        return usuarioService.listarUsuarios();
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<UsuarioResponse>> obtenerPorId(@PathVariable String id) {
        return usuarioService.obtenerUsuario(id)
                .map(ResponseEntity::ok)
                .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @PostMapping
    public Mono<ResponseEntity<UsuarioResponse>> crear(@RequestBody Mono<UsuarioRequest> requestMono) {
        return requestMono
                .flatMap(usuarioService::crearUsuario)
                .map(u -> ResponseEntity.status(HttpStatus.CREATED).body(u));
    }
}
```

Puntos importantes:

- Se devuelve `Flux<T>` para colecciones y `Mono<T>` para elementos individuales.
- El `@RequestBody` puede ser un `Mono<T>` (útil cuando el cuerpo se procesa de forma reactiva).
- Se usan operadores reactivos para manejar el flujo (map, flatMap, etc.).

---

## 6. Enfoque funcional: Routers y Handlers

Spring WebFlux también permite definir rutas de manera funcional (sin anotaciones).

### 6.1. Handler

```java
@Component
@RequiredArgsConstructor
public class UsuarioHandler {

    private final UsuarioService usuarioService;

    public Mono<ServerResponse> listar(ServerRequest request) {
        return ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(usuarioService.listarUsuarios(), UsuarioResponse.class);
    }

    public Mono<ServerResponse> obtenerPorId(ServerRequest request) {
        String id = request.pathVariable("id");
        return usuarioService.obtenerUsuario(id)
                .flatMap(usuario -> ServerResponse.ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .bodyValue(usuario))
                .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> crear(ServerRequest request) {
        return request.bodyToMono(UsuarioRequest.class)
                .flatMap(usuarioService::crearUsuario)
                .flatMap(usuarioCreado -> ServerResponse.status(HttpStatus.CREATED)
                        .contentType(MediaType.APPLICATION_JSON)
                        .bodyValue(usuarioCreado));
    }
}
```

### 6.2. Router

```java
@Configuration
public class UsuarioRouter {

    @Bean
    public RouterFunction<ServerResponse> usuarioRoutes(UsuarioHandler handler) {
        return RouterFunctions.route()
                .GET("/api/usuarios", handler::listar)
                .GET("/api/usuarios/{id}", handler::obtenerPorId)
                .POST("/api/usuarios", handler::crear)
                .build();
    }
}
```

Este estilo es muy útil para:

- APIs puramente funcionales.
- Endpoints simples.
- Cuando se quiere evitar el uso de anotaciones y tener la configuración centralizada en routers.

---

## 7. Acceso a datos reactivo

### 7.1. Con MongoDB Reactive

Repositorio:

```java
@Repository
public interface UsuarioRepository extends ReactiveMongoRepository<UsuarioDocument, String> {
    Flux<UsuarioDocument> findByActivoTrue();
}
```

Servicio:

```java
@Service
@RequiredArgsConstructor
public class UsuarioService {

    private final UsuarioRepository usuarioRepository;
    private final UsuarioMapper mapper;

    public Flux<UsuarioResponse> listarUsuarios() {
        return usuarioRepository.findAll()
                .map(mapper::toResponse);
    }

    public Mono<UsuarioResponse> obtenerUsuario(String id) {
        return usuarioRepository.findById(id)
                .map(mapper::toResponse);
    }

    public Mono<UsuarioResponse> crearUsuario(UsuarioRequest request) {
        UsuarioDocument doc = mapper.toDocument(request);
        return usuarioRepository.save(doc)
                .map(mapper::toResponse);
    }
}
```

### 7.2. Con R2DBC (bases relacionales)

Ejemplo de repositorio con R2DBC:

```java
@Table("usuarios")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UsuarioEntity {
    @Id
    private Long id;
    private String nombre;
    private String email;
}

public interface UsuarioR2dbcRepository extends ReactiveCrudRepository<UsuarioEntity, Long> {
    Flux<UsuarioEntity> findByActivo(Boolean activo);
}
```

> Ojo: R2DBC implica usar un **driver reactivo**, diferente de JDBC. No se puede mezclar R2DBC y JDBC en las mismas operaciones manteniendo la no-bloqueante.

---

## 8. WebClient: cliente HTTP reactivo

En lugar de `RestTemplate` (bloqueante), en WebFlux se recomienda usar `WebClient`.

```java
@Component
@RequiredArgsConstructor
public class PagosClient {

    private final WebClient webClient = WebClient.builder()
            .baseUrl("http://pagos-service")
            .build();

    public Mono<PagoResponse> obtenerPago(String id) {
        return webClient.get()
                .uri("/api/pagos/{id}", id)
                .retrieve()
                .bodyToMono(PagoResponse.class);
    }

    public Flux<PagoResponse> listarPagosUsuario(String usuarioId) {
        return webClient.get()
                .uri(uriBuilder -> uriBuilder
                        .path("/api/pagos")
                        .queryParam("usuarioId", usuarioId)
                        .build())
                .retrieve()
                .bodyToFlux(PagoResponse.class);
    }
}
```

Ventajas:

- Totalmente integrado con Reactor (`Mono` y `Flux`).
- Ideal para **composición de llamadas a otros microservicios**.

---

## 9. Backpressure (control de presión)

En sistemas reactivos, el **consumidor** puede indicar cuánto puede procesar, para no ser desbordado.

Project Reactor y Reactive Streams soportan backpressure de forma nativa.

Ejemplo con `Flux`:

```java
Flux.range(1, 100)
    .log()
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10); // pido 10 elementos
        }

        @Override
        protected void hookOnNext(Integer value) {
            // procesar valor
            // cuando estés listo para más, puedes pedir explicitamente:
            request(1);
        }
    });
```

En la mayoría de casos, no tienes que manejar el backpressure manualmente, pero entender el concepto ayuda a diseñar sistemas que no colapsen ante picos de carga.

---

## 10. Testing en WebFlux

### 10.1. Test de servicios reactivos con StepVerifier

```java
class UsuarioServiceTest {

    @Mock
    UsuarioRepository usuarioRepository;

    @InjectMocks
    UsuarioService usuarioService;

    @Test
    void listarUsuarios_ok() {
        UsuarioDocument u1 = new UsuarioDocument("1", "Javier", "javier@example.com");
        UsuarioDocument u2 = new UsuarioDocument("2", "Ana", "ana@example.com");

        when(usuarioRepository.findAll()).thenReturn(Flux.just(u1, u2));

        Flux<UsuarioResponse> flujo = usuarioService.listarUsuarios();

        StepVerifier.create(flujo)
                .expectNextMatches(u -> u.getNombre().equals("Javier"))
                .expectNextMatches(u -> u.getNombre().equals("Ana"))
                .verifyComplete();
    }
}
```

### 10.2. Test de controladores con WebTestClient

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UsuarioControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void listarUsuarios_endpoint() {
        webTestClient.get()
                .uri("/api/usuarios")
                .exchange()
                .expectStatus().isOk()
                .expectBodyList(UsuarioResponse.class)
                .hasSize(2);
    }
}
```

---

## 11. Ventajas y desventajas de WebFlux

### 11.1. Ventajas

- **Escalabilidad** con menos hilos → mejor uso de recursos.  
- Modelo ideal para **IO intensivo** (llamadas a otros servicios, DBs reactivas).  
- Composición de flujos (chaining con operadores) muy poderosa.  
- Integración natural con WebSockets, SSE, streaming de datos, etc.

### 11.2. Desventajas

- **Curva de aprendizaje alta** para quienes vienen de MVC tradicional.  
- Depuración y trazas pueden ser más difíciles (flujos asíncronos).  
- Si la mayor parte del stack es bloqueante, se pierde gran parte del beneficio.  
- Código puede volverse ilegible si no se siguen buenas prácticas (encadenamientos enormes, lambdas complejas).

---

## 12. Buenas prácticas

1. **No mezclar bloqueante y reactivo sin necesidad**  
   - Evita llamar a JDBC, `RestTemplate` u otros clientes bloqueantes desde WebFlux.  
   - Si no hay alternativa, encapsula eso en `Schedulers.boundedElastic()` y documenta el coste.

2. **Modela tu dominio de forma clara**  
   - Aunque uses `Mono`/`Flux`, la lógica de negocio debe seguir principios SOLID, DDD, etc.

3. **Mantén los controladores delgados**  
   - Usa servicios (casos de uso) para la lógica, igual que en MVC.

4. **Evita llamar a `block()` o `subscribe()` en código de producción**  
   - Son operaciones terminales; normalmente el framework se encarga de suscribirse.

5. **Usa operadores adecuadamente**  
   - `map` → transformaciones simples  
   - `flatMap` → operaciones asíncronas que devuelven otro `Mono`/`Flux`  
   - `concatMap` / `flatMapSequential` cuando importa el **orden**.

---

## 13. Resumen

- Spring WebFlux implementa el paradigma de **programación reactiva** usando **Project Reactor** (`Mono`, `Flux`).
- Es ideal para servicios con **alta concurrencia e IO intensivo**, siempre que el stack soporte librerías reactivas (DB, HTTP, etc.).
- Ofrece dos estilos principales:
  - Controladores con anotaciones (`@RestController`)  
  - Enfoque funcional con Routers + Handlers
- Se integra con **WebClient** para llamadas HTTP reactivas.
- Requiere cambiar el mindset: pensar en **flujos** y **composición** de operaciones, no en llamadas secuenciales bloqueantes.

Con esta base, puedes construir un microservicio “Pagos” o “Usuarios” 100% reactivo, integrarlo con MongoDB Reactive o R2DBC y exponerlo detrás de un API Gateway en un entorno de microservicios moderno.
