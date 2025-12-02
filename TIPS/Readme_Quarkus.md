
# Guía de Transición: De Spring Boot a Quarkus (para equipos Java)

Si tu equipo viene de **Spring Boot**, esta guía explicará cómo entender Quarkus de forma sencilla y sin choque cultural.

---

## 1. ¿Son parecidos o completamente diferentes?

### ✔ Parecidos:
- Inyección de dependencias  
- REST con anotaciones  
- Uso de perfiles/config  
- Integración con JPA/Hibernate  
- Arquitecturas limpias aplican igual  

### ❌ Diferentes:
- Quarkus se optimiza **en build-time**, no en runtime
- Usa **CDI** en lugar de Spring Context
- Usa **Mutiny** en lugar de Reactor
- Tiene **extensiones** en lugar de starters
- Es nativo de **Kubernetes / OpenShift**

---

## 2. Paralelo directo entre anotaciones

| Spring Boot | Quarkus |
|-------------|----------|
| `@RestController` | `@Path` + métodos JAX-RS |
| `@Autowired` | `@Inject` |
| `@Service` | `@ApplicationScoped` |
| `@Repository` | No siempre necesario con Panache |
| `@RequestMapping` | `@Path` / `@GET` / `@POST` |

---

## 3. Cómo pensar Quarkus si vienes de Spring

### 3.1. Spring es runtime-first  
Escanea todo, crea beans en arranque, configura casi todo dinámicamente.

### 3.2. Quarkus es build-first  
Optimiza el código **antes** de correrlo:  
- crea beans  
- prepara proxies  
- reduce reflección  
- minimiza tiempos de arranque  
- prepara la app para GraalVM

---

## 4. ¿Qué sigue igual?

- El dominio (entidades, DTOs, arquitectura limpia)
- Las pruebas unitarias
- La forma de estructurar microservicios
- El uso de Maven/Gradle
- La lógica de negocio es igual (imperativa o reactiva)

---

## 5. ¿Qué cambia?

### 5.1. La API REST
De:
```java
@RestController
@RequestMapping("/api")
public class UsuarioController {}
```

A:
```java
@Path("/api")
public class UsuarioResource {}
```

### 5.2. Inyección de dependencias
De:
```java
@Autowired
UsuarioService service;
```

A:
```java
@Inject
UsuarioService service;
```

### 5.3. Programación Reactiva
De:
`Mono<T>` / `Flux<T>`  
A:
`Uni<T>` / `Multi<T>`

---

## 6. Cómo introducir Quarkus sin miedo al equipo

1. Mostrar que **no reemplaza a Spring**, sino que convive.
2. Explicar que es ideal para:
   - Kubernetes
   - Serverless
   - Microservicios con miles de instancias
3. Hacer un ejemplo pequeño con Panache.
4. Comparar arranque y RAM vs Spring.
5. Hacer un endpoint reactivo con Mutiny.

---

## 7. Conclusión

Quarkus no es “otro framework raro”.  
Es una **evolución natural** para equipos que ya dominan Java y microservicios.

Si sabes Spring Boot → aprender Quarkus es bastante rápido.
