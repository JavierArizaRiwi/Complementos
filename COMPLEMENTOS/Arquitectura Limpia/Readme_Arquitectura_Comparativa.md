
# Arquitecturas Limpias + DDD + Comparativa + Ejemplos Prácticos (Java)

Este documento complementa tu guía original de **Arquitectura Limpia** agregando:

- Integración con **Domain-Driven Design (DDD)**  
- Comparativa entre arquitecturas modernas  
- Ejemplos reales de estructuras de carpetas  
- Snippets de código Java para cada arquitectura  
- Material para nivel senior / arquitecto backend  

---

# 11. Cómo se relaciona Clean Architecture con DDD (Domain-Driven Design)

Aunque son enfoques diferentes, **Clean Architecture y DDD funcionan perfectamente juntos**.

## 11.1. ¿Qué aporta Clean Architecture?

- Dependencias hacia adentro  
- Separación estricta entre negocio e infraestructura  
- Claridad en capas y responsabilidades  
- Alta testabilidad  
- Independencia de frameworks  

## 11.2. ¿Qué aporta DDD?

- Lenguaje Ubicuo  
- Entidades ricas (no anémicas)  
- Value Objects  
- Agregados  
- Dominios y subdominios  
- Servicios de dominio  
- Dominios protegidos contra inconsistencias  

## 11.3. Relación entre capas

| Clean Architecture | DDD |
|-------------------|-----|
| Entidades | Entities + Value Objects + Domain Events |
| Casos de uso | Application Services / Commands |
| Adaptadores | Repositories + Controllers + Mappers |
| Infraestructura | Implementaciones concretas |

### Ejemplo conceptual

- Clean Architecture → “El dominio no debe depender de frameworks.”
- DDD → “El dominio debe expresar las reglas del negocio.”

**Juntos → dominio poderoso, estable, testeable.**

---

# 12. Otras Arquitecturas Modernas y Cuándo Usarlas

## 12.1. Hexagonal Architecture (Ports & Adapters)
Ya conocida por tus coders:  
El dominio se comunica con el exterior mediante **puertos** y **adaptadores**.

Ideal cuando:  
- Tienes múltiples interfaces (REST, Kafka, CLI, cron)  
- Cambiarás infraestructura frecuentemente  
- Necesitas independizar el dominio de todo lo externo  

---

## 12.2. Onion Architecture (Arquitectura en Cebolla)

Similares principios a Clean, pero en capas concéntricas:

```
Infra → Application → Domain Services → Domain Model (núcleo)
```

Ideal para:  
- Dominios ricos  
- Fuerte uso de DDD  
- Regla de negocio compleja  

---

## 12.3. CQRS (Command Query Responsibility Segregation)

Divide el sistema en:

- **Commands** → mutación  
- **Queries** → lectura optimizada  

Ideal cuando:  

- El sistema tiene alta carga de consultas  
- Los modelos de lectura son distintos a los de escritura  
- Se necesita escalado independiente por tipo de operación  

---

## 12.4. Event-Driven Architecture

Basada en eventos (Kafka, RabbitMQ).  
Alta asincronía y desacoplamiento.

Ideal cuando:  
- Flujo en tiempo real  
- Sistemas reactivos  
- Orquestación de microservicios  

---

## 12.5. Modular Monolith (Monolito Modular)

Estructura monolítica, pero dividida en módulos aislados semánticamente.

Ideal para:  
- Equipos pequeños o medianos  
- Proyectos que crecerán en el tiempo  
- Antesala de microservicios sin complejidad extrema  

---

## 12.6. Comparativa general

| Arquitectura | Cuándo usarla | Ventajas | Riesgos |
|--------------|----------------|----------|---------|
| Clean | Proyectos medianos/grandes | Claridad, testabilidad | Overkill para apps pequeñas |
| Hexagonal | Muchas interfaces externas | Desacoplamiento extremo | Complejidad si se abusa |
| Onion | Dominio complejo | DDD profundo | Requiere madurez técnica |
| Modular Monolith | Equipos pequeños | Simplicidad + orden | Puede degradarse si no se controla |
| CQRS | Alto tráfico de lectura | Performance | Complejidad extra |
| Event-Driven | Asincronía, resiliencia | Desac acoplamiento | Debugging complejo |

---

# 13. Ejemplos prácticos (estructura + código Java)

---

## 13.1. Clean Architecture + DDD

### Estructura recomendada

```text
src/main/java/com/empresa/pagos
 ├─ domain
 │   ├─ model
 │   ├─ vo
 │   ├─ repository
 │   └─ service
 ├─ application
 │   ├─ usecase
 │   ├─ dto
 │   └─ mapper
 └─ infrastructure
     ├─ persistence
     ├─ web
     └─ config
```

### Ejemplo de Value Object

```java
public record Monto(BigDecimal valor) {
    public Monto {
        if (valor == null || valor.compareTo(BigDecimal.ZERO) <= 0)
            throw new IllegalArgumentException("Monto inválido");
    }
}
```

### Puerto de dominio (Repository)

```java
public interface PagoRepository {
    Pago guardar(Pago pago);
}
```

### Caso de uso

```java
public class ProcesarPagoUseCase {
    private final PagoRepository repo;

    public ProcesarPagoUseCase(PagoRepository repo) { this.repo = repo; }

    public PagoResponse ejecutar(PagoRequest req) {
        Pago pago = new Pago(new Monto(req.monto()), req.referencia());
        return PagoMapper.toResponse(repo.guardar(pago));
    }
}
```

---

## 13.2. Arquitectura Hexagonal (Ports & Adapters)

### Estructura

```text
domain/
  model/
  port/
application/
  usecase/
infrastructure/
  adapter/
  web/
```

### Puerto

```java
public interface NotificacionPort {
    void enviar(String mensaje, String destino);
}
```

### Adaptador

```java
public class EmailNotificacionAdapter implements NotificacionPort {
    public void enviar(String msg, String destino) {
        System.out.println("Enviando email...");
    }
}
```

---

## 13.3. Onion Architecture (Cebolla)

### Estructura

```text
core/
  domain/
  services/
application/
infrastructure/
```

### Servicio de dominio

```java
public class PedidoDomainService {
    public void agregarProducto(Pedido pedido, Producto producto, int cantidad) {
        pedido.agregarLinea(producto, cantidad);
    }
}
```

---

## 13.4. Modular Monolith

```text
src/main/java/com/empresa
 ├─ pagos
 ├─ clientes
 ├─ productos
 └─ shared
```

Cada módulo tiene mini-CA adentro.

---

## 13.5. CQRS

### Estructura

```text
command/
  handler/
query/
  handler/
api/
```

### Command Handler

```java
public class CrearPedidoCommandHandler {
    public void handle(CrearPedidoCommand cmd) {
        PedidoAggregate pedido = new PedidoAggregate(cmd.clienteId(), cmd.lineas());
        repository.save(pedido);
    }
}
```

---

## 13.6. Event-Driven Architecture

### Evento de dominio

```java
public record PagoRealizadoEvent(Long id, BigDecimal monto) {}
```

### Publisher

```java
public interface PagoEventPublisher {
    void publicar(PagoRealizadoEvent event);
}
```

---

# 14. Conclusión

Con estas secciones adicionales ahora tienes:

- Integración avanzada Clean Architecture + DDD  
- Comparativa profesional entre arquitecturas  
- Ejemplos reales de código  
- Estructuras de carpetas enterprise  
- Material totalmente alineado con microservicios, fintech y banca  

Un documento digno de un **Senior Backend / Arquitecto**.

---

FIN DEL DOCUMENTO