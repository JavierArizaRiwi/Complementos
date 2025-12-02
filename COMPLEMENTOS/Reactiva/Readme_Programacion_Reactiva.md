# Guía Completa: Programación Reactiva con Spring WebFlux

### Explicación general y luego específica para backend moderno con Spring Boot

Esta guía explica de forma clara, profesional y estructurada qué es la
programación reactiva, por qué existe, qué problemas resuelve y cómo
Spring WebFlux implementa un modelo moderno no bloqueante para alto
rendimiento.

------------------------------------------------------------------------

# 1. ¿Qué es la Programación Reactiva?

La programación reactiva es un paradigma que se centra en:

-   Manejar **flujos de datos asincrónicos**
-   Reaccionar a cambios en tiempo real
-   Trabajar con **eventos**, no con hilos bloqueantes
-   Escalar con menos recursos
-   Evitar saturar hilos por operaciones lentas (I/O)

Se basa en el concepto de **streams** (secuencias de datos) y
**propagación de cambios**.

No es nuevo, pero hoy es esencial en sistemas:

-   De alta concurrencia
-   Streaming (video, chat, WebSockets)
-   Microservicios con miles de peticiones
-   IoT
-   APIs que no deben bloquear

------------------------------------------------------------------------

# 2. ¿Por qué nació la Programación Reactiva?

Para resolver problemas del modelo tradicional servlet (Spring MVC), que
usa un **hilo por petición**.

Problemas del modelo bloqueante:

1.  Si una petición espera respuesta de BD o API externa, el hilo queda
    bloqueado.
2.  Necesitas muchos hilos → alto consumo de RAM.
3.  Threads costosos → contexto, stack, scheduling.
4.  No escala bien a miles de conexiones concurrentes.

------------------------------------------------------------------------

# 3. Spring WebFlux: el framework reactivo de Spring

Spring WebFlux es la alternativa no bloqueante de Spring MVC.

Características:

-   Totalmente **reactivo**
-   Basado en **Netty**, no en Tomcat (aunque funciona en ambos)
-   **No utiliza Thread-per-Request**
-   Basado en **Event Loop**
-   Altamente escalable y eficiente

Spring WebFlux usa:

-   **Reactor**: implementación de Reactive Streams
-   **Mono**: 0..1 elementos
-   **Flux**: 0..N elementos

------------------------------------------------------------------------

# 4. Diferencias entre Spring MVC vs Spring WebFlux

### 4.1 Modelo bloqueante (MVC)

-   Un hilo por petición
-   Si llamas a la BD o API externa → el hilo se bloquea
-   Si hay 500 peticiones → necesitas 500 hilos
-   Escalabilidad limitada

### 4.2 Modelo reactivo (WebFlux)

-   Manejo de peticiones con Event Loop (Netty)
-   No hay bloqueo
-   Puedes atender miles de conexiones con pocos hilos
-   Ideal para cargas altas y streaming

------------------------------------------------------------------------

# 5. Conceptos base en WebFlux

## 5.1 Mono

Representa un stream de **0 o 1 elemento**.

Ejemplo: - Buscar usuario por ID - Guardar un registro - Login

``` java
Mono<Usuario> usuario = usuarioService.findById(id);
```

------------------------------------------------------------------------

## 5.2 Flux

Representa un stream de **0 a N elementos**.

Ejemplo: - Listar productos - Stream de eventos - WebSockets

``` java
Flux<Pago> pagos = pagoService.findAll();
```

------------------------------------------------------------------------

## 5.3 Operadores Reactivos

WebFlux usa operadores funcionales parecidos a Java Streams:

-   `map`
-   `flatMap`
-   `filter`
-   `switchIfEmpty`
-   `zip`
-   `concat`
-   `collectList`

Ejemplo:

``` java
return pagoRepository.findAll()
    .filter(p -> p.getMonto() > 1000)
    .map(PagoDTO::from);
```

------------------------------------------------------------------------

# 6. Flujo no bloqueante explicado

En WebFlux:

1.  Una petición llega a Netty (event loop)
2.  Netty no crea hilos adicionales
3.  Todo se procesa sin bloqueo
4.  Cuando una operación I/O termina, el flujo "reactiva" la respuesta
5.  Se envía al cliente

Es un modelo basado en **callbacks** organizados como flujos
(Mono/Flux).

------------------------------------------------------------------------

# 7. Ejemplo básico de controlador WebFlux

``` java
@RestController
@RequestMapping("/pagos")
public class PagoController {

    private final PagoService service;

    public PagoController(PagoService service) {
        this.service = service;
    }

    @GetMapping("/{id}")
    public Mono<PagoDTO> obtenerPago(@PathVariable String id) {
        return service.obtenerPago(id);
    }

    @GetMapping
    public Flux<PagoDTO> listarPagos() {
        return service.listarPagos();
    }

    @PostMapping
    public Mono<PagoDTO> crearPago(@RequestBody Mono<PagoDTO> pago) {
        return pago.flatMap(service::crearPago);
    }
}
```

------------------------------------------------------------------------

# 8. Programación Reactiva en la capa de servicio

``` java
@Service
public class PagoService {

    private final PagoRepository repo;

    public PagoService(PagoRepository repo) {
        this.repo = repo;
    }

    public Mono<PagoDTO> obtenerPago(String id) {
        return repo.findById(id)
            .map(PagoDTO::from);
    }

    public Flux<PagoDTO> listarPagos() {
        return repo.findAll()
            .map(PagoDTO::from);
    }

    public Mono<PagoDTO> crearPago(PagoDTO dto) {
        return repo.save(dto.toEntity())
            .map(PagoDTO::from);
    }
}
```

------------------------------------------------------------------------

# 9. Acceso reactivo a la base de datos

WebFlux es totalmente no bloqueante.\
Para lograrlo, debe usar:

-   **R2DBC** (base de datos relacional reactiva)
-   O **MongoDB Reactive**
-   O cualquier DB reactiva

Ejemplo reactive con R2DBC:

``` java
public interface PagoRepository extends ReactiveCrudRepository<Pago, String> {}
```

------------------------------------------------------------------------

# 10. Manejo de errores reactivo

``` java
return repo.findById(id)
    .switchIfEmpty(Mono.error(new RuntimeException("No existe")))
    .onErrorResume(e -> Mono.error(new CustomException(e.getMessage())));
```

------------------------------------------------------------------------

# 11. Cuándo usar WebFlux (casos recomendados)

### Sí usar WebFlux si:

-   Tu app tendrá miles de conexiones concurrentes
-   Haces streaming de datos
-   Tienes WebSockets
-   Haces llamadas a servicios externos
-   Estás en microservicios altamente escalables
-   Buscas eficiencia en recursos

### No usar WebFlux si:

-   Tu app es CRUD simple
-   La base de datos es bloqueante (JPA/Hibernate)
-   El equipo no domina programación reactiva
-   No habrá muchas conexiones concurrentes

------------------------------------------------------------------------

# 12. Comparación final

  Característica   Spring MVC      Spring WebFlux
  ---------------- --------------- --------------------------
  Modelo           Bloqueante      No bloqueante
  Hilos            1 por request   Pocos hilos (event-loop)
  Escalabilidad    Limitada        Alta
  Backpressure     No              Sí
  Streaming        Limitado        Ideal
  Performance      Buena           Excelente con alta carga
  BDs soportadas   JDBC            R2DBC, Mongo reactive

------------------------------------------------------------------------

# 13. Conclusión

Spring WebFlux es la solución moderna de Java para sistemas:

-   Asincrónicos\
-   No bloqueantes\
-   Escalables\
-   De alta concurrencia\
-   Dirigidos por eventos

Permite procesar miles de conexiones sin saturar el servidor y está
basado en estándares como **Reactive Streams**, **Publisher**,
**Subscriber**, **Backpressure**, y los tipos reactivamente esenciales
**Mono** y **Flux**.

Para microservicios modernos de alto rendimiento, WebFlux es una de las
mejores tecnologías disponibles en el ecosistema Java.