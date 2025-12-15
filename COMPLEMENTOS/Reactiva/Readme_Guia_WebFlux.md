
# 📘 Guía Completa: Programación Reactiva con Spring WebFlux

Esta guía te ayudará a entender **qué es la programación reactiva**, **por qué es importante hoy en día**, y **cómo se usa con Spring WebFlux** para construir APIs modernas, rápidas y escalables.

Está explicada de forma clara, paso a paso, sin asumir conocimientos avanzados.

---

# 1. ¿Qué es la Programación Reactiva?

La programación reactiva es una forma diferente de escribir aplicaciones:

✔ Trabaja con **flujos de datos**  
✔ Maneja operaciones de forma **asincrónica** (sin esperar bloqueado)  
✔ Permite responder a **muchos usuarios al mismo tiempo**  
✔ Reacciona a eventos conforme llegan  

Imagina que tu sistema es un restaurante:

- En el modelo tradicional: cada mesero atiende una única mesa → si la mesa se demora, el mesero queda “bloqueado”.
- En el modelo reactivo: un mesero atiende muchas mesas → trabaja cuando hay algo que hacer, no se queda esperando.

Así funciona WebFlux.

---

# 2. ¿Por qué nació la Programación Reactiva?

Porque el modelo tradicional (Spring MVC) usa **un hilo por petición**, y esto genera problemas:

🚫 Si una petición hace una consulta lenta a la BD, **bloquea un hilo**  
🚫 Con muchos usuarios, necesitas **muchos hilos**, lo cual **consume mucha memoria**  
🚫 No escala en aplicaciones de miles de conexiones  

La programación reactiva soluciona esto trabajando con **menos hilos**, usando un modelo basado en **eventos**.

---

# 3. Spring WebFlux: la versión reactiva de Spring

Spring WebFlux es el framework de Spring que:

- Es **no bloqueante**  
- Funciona con **event loops** (como Node.js)  
- Permite manejar MUCHAS conexiones con POCA memoria  
- Usa **Reactor**, una biblioteca basada en Reactive Streams

WebFlux trabaja con dos tipos principales de flujos:

### 🔹 Mono → representa 0 o 1 elemento  
Ejemplos:  
- Buscar un usuario por ID  
- Crear un pago  

### 🔹 Flux → representa 0…N elementos  
Ejemplos:  
- Listado de productos  
- Stream de eventos  
- WebSockets  

---

# 4. Diferencias entre Spring MVC y WebFlux

| Concepto | Spring MVC (Bloqueante) | Spring WebFlux (Reactivo) |
|---------|---------------------------|----------------------------|
| Modelo | Un hilo por request | Event Loop (pocos hilos) |
| Escalabilidad | Limitada | Muy alta |
| Uso de memoria | Alto | Bajo |
| Ideal para | CRUD normal | Apps con mucha concurrencia |
| Streaming | Limitado | Excelente |
| BDs soportadas | JDBC | R2DBC, Mongo Reactive |

**En pocas palabras:** WebFlux no reemplaza a MVC, cada uno tiene su uso.

---

# 5. Conceptos base en WebFlux para un Junior

### 5.1 Mono → 1 dato
```java
Mono<Usuario> usuario = usuarioService.findById(id);
```

### 5.2 Flux → Muchos datos
```java
Flux<Pago> pagos = pagoService.findAll();
```

### 5.3 Operadores reactivos (muy importantes)

Parecidos a los de Java Streams:

- `map` → transforma un dato
- `flatMap` → encadena procesos asincrónicos
- `filter` → filtra elementos
- `switchIfEmpty` → retorna algo si el flujo viene vacío
- `collectList` → convierte un Flux en una lista

Ejemplo:

```java
return pagoRepository.findAll()
        .filter(p -> p.getMonto() > 1000)
        .map(PagoDTO::from);
```

---

# 6. ¿Cómo funciona el modelo no bloqueante?

En WebFlux:

1. Netty recibe la petición  
2. No bloquea un hilo para cada request  
3. Solo trabaja cuando hay datos listos  
4. Cuando algo termina (BD, API), se “reactiva” el flujo  
5. La respuesta se envía al usuario  

Esto permite tener **miles de peticiones al mismo tiempo** sin necesidad de miles de hilos.

---

# 7. Ejemplo básico de un controlador WebFlux

```java
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

Puntos clave:  
✔ No devuelve `ResponseEntity`, devuelve **Mono/Flux**  
✔ El cuerpo del request también puede ser **Mono**  
✔ Todo es asincrónico  

---

# 8. Capa de servicio con WebFlux

```java
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

Fíjate cómo desaparecen los returns normales y se reemplazan por **Mono** y **Flux**.

---

# 9. Acceso reactivo a bases de datos

Para ser realmente reactivo, necesitas bases de datos que no bloqueen:

### Para bases relacionales:
✔ **R2DBC**

### Para bases NoSQL:
✔ Mongo Reactive  
✔ Cassandra Reactive  

Ejemplo:

```java
public interface PagoRepository extends ReactiveCrudRepository<Pago, String> {}
```

---

# 10. Manejo de errores en WebFlux

```java
return repo.findById(id)
        .switchIfEmpty(Mono.error(new RuntimeException("No existe")))
        .onErrorResume(e -> Mono.error(new CustomException(e.getMessage())));
```

Claves:

- **switchIfEmpty** → si no hay datos, lanza error  
- **onErrorResume** → manejar errores de forma controlada  

---

# 11. ¿Cuándo usar WebFlux? (explicado para un Junior)

### ✔ Sí usar WebFlux si:

- Tu API recibirá **muchas peticiones**
- Haces **llamadas a otros servicios externos**
- Usas **WebSockets o streaming**
- Quieres **optimizar recursos**
- Trabajas con **microservicios modernos**

### ❌ No usar WebFlux si:

- Es un CRUD básico  
- Necesitas JPA (bloqueante)  
- El equipo no domina programación reactiva  
- No habrá mucha concurrencia  

---

# 12. Comparación final (tabla clara)

| Característica | Spring MVC | Spring WebFlux |
|----------------|-------------|-----------------|
| Bloqueante | Sí | No |
| Modelo | Un hilo por request | Event Loop |
| Escalabilidad | Media | Muy alta |
| Performance | Buena | Excelente |
| BD recomendada | JDBC | R2DBC / MongoR |
| Streaming | Limitado | Ideal |
| Complejidad | Baja | Media/Alta |

---

# 13. Conclusión final para un Junior

Spring WebFlux es una tecnología moderna que te permitirá desarrollar:

- APIs más rápidas  
- Sistemas más escalables  
- Microservicios reactivos  
- Aplicaciones con miles de usuarios  

A diferencia del modelo tradicional, WebFlux:

✔ No bloquea hilos  
✔ Consume menos recursos  
✔ Trabaja con flujos (Mono / Flux)  
✔ Está diseñado para sistemas modernos y de alta demanda  

Dominar WebFlux no solo te convierte en un mejor programador Java, sino que también te abre puertas en empresas que requieren **alto rendimiento**, como:

- Bancos  
- Fintech  
- Telecomunicaciones  
- E-commerce  
- Plataformas de streaming  

