# Guía Completa: Arquitecturas Limpias (Clean Architecture)

Esta guía explica de manera clara, profesional y profundamente detallada
qué es la Arquitectura Limpia, cuáles son sus principios, cómo se
organiza, por qué es usada por empresas modernas y cómo aplicarla
correctamente en proyectos Java/Spring Boot, Node.js, .NET u otros
lenguajes.

------------------------------------------------------------------------

# 1. ¿Qué es la Arquitectura Limpia?

La Arquitectura Limpia (Clean Architecture), propuesta por Robert C.
Martin (Uncle Bob), es un enfoque de diseño que tiene como objetivo
principal **separar responsabilidades**, **independizar el negocio de
las herramientas** y **garantizar que el código sea fácil de mantener,
probar y extender**.

Su idea fundamental es:

**El dominio (reglas de negocio) debe ser independiente de frameworks,
UI, bases de datos o servicios externos.**

------------------------------------------------------------------------

# 2. ¿Por qué usar Arquitectura Limpia?

La Arquitectura Limpia permite:

-   Reducir la complejidad del sistema.
-   Facilitar pruebas unitarias y de integración.
-   Cambiar frameworks sin afectar el negocio (Spring, Angular, React,
    etc.).
-   Sustituir bases de datos sin reescribir lógica.
-   Evitar código "spaghetti".
-   Escalar y mantener el sistema a largo plazo.
-   Crear sistemas más robustos y menos acoplados.

Ejemplo común:

Cambiar de MySQL a PostgreSQL solo afecta un Adaptador, no la lógica de
negocio.

------------------------------------------------------------------------

# 3. Principios Fundamentales

Estos son los pilares de Clean Architecture:

## 3.1 Independencia de Frameworks

El negocio NO puede depender del framework.

Ejemplo: - Spring Boot es infraestructura. - NestJS es
infraestructura. - .NET Controllers es infraestructura.

El dominio se mantiene libre de anotaciones y librerías externas.

------------------------------------------------------------------------

## 3.2 Testabilidad

El dominio debe ser tan simple que pueda probarse sin cargar servidores,
bases de datos ni contenedores.

------------------------------------------------------------------------

## 3.3 Independencia de UI

Puedes cambiar la interfaz (REST, GraphQL, CLI, WebSockets) sin tocar el
dominio.

Ejemplo: - Cambiar un controlador REST por una cola RabbitMQ requiere
solo adaptadores.

------------------------------------------------------------------------

## 3.4 Independencia de la base de datos

El dominio no debe conocer SQL, JPA, Repositories, JdbcTemplate, ni
drivers.

------------------------------------------------------------------------

## 3.5 Independencia de servicios externos

Servicios como AWS S3, Twilio, Stripe, correo o APIs externas deben
estar desacoplados del dominio.

------------------------------------------------------------------------

# 4. El Círculo de la Arquitectura Limpia

El modelo clásico se representa así:

          +--------------------------+
          |        Frameworks        |
          +--------------------------+
          |      Adaptadores         |
          +--------------------------+
          |     Casos de Uso         |
          +--------------------------+
          |        Entidades         |
          +--------------------------+

Se basa en **dependencias hacia adentro**:

-   Las capas externas dependen de las internas.
-   Las internas no conocen ni dependen de las externas.

------------------------------------------------------------------------

# 5. Capas de la Arquitectura

A continuación, se explican las capas una por una.

------------------------------------------------------------------------

# 5.1 Entidades (Domain / Enterprise Business Rules)

Son las **reglas de negocio más puras** del sistema.

Incluyen: - Entidades del dominio - Value Objects - Agregados - Lógica
inmutable y estable

No deben tener: - Anotaciones de frameworks - Accesos a BD - Llamadas
HTTP

Ejemplo:

``` java
public class Pago {
    private final BigDecimal monto;
    private final LocalDate fecha;

    public Pago(BigDecimal monto, LocalDate fecha) {
        if (monto.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("monto invalido");
        }
        this.monto = monto;
        this.fecha = fecha;
    }
}
```

------------------------------------------------------------------------

# 5.2 Casos de Uso (Application Business Rules)

Son las **acciones del sistema**.

Ejemplos: - Procesar un pago - Registrar un usuario - Generar una
factura - Crear una reserva

Un caso de uso: - Orquesta entidades - Aplica lógica del negocio - No
sabe de frameworks - No sabe de infraestructura

Ejemplo:

``` java
public class ProcesarPagoUseCase {

    private final PagoRepository pagoRepository;

    public ProcesarPagoUseCase(PagoRepository pagoRepository) {
        this.pagoRepository = pagoRepository;
    }

    public Pago ejecutar(ComandoPago comando) {
        Pago pago = new Pago(comando.monto(), comando.fecha());
        return pagoRepository.guardar(pago);
    }
}
```

------------------------------------------------------------------------

# 5.3 Adaptadores (Interface Adapters)

Son los encargados de **traducir datos** entre el mundo externo e
interno.

Incluye: - Controllers - DTOs - Mappers - Presenters - Gateways -
Repositorios concretos

Ejemplo:

``` java
@RestController
@RequestMapping("/pagos")
public class PagoController {

    private final ProcesarPagoUseCase procesarPagoUseCase;

    public PagoController(ProcesarPagoUseCase procesarPagoUseCase) {
        this.procesarPagoUseCase = procesarPagoUseCase;
    }

    @PostMapping
    public PagoDTO crear(@RequestBody PagoDTO dto) {
        Pago pago = procesarPagoUseCase.ejecutar(new ComandoPago(dto.monto(), dto.fecha()));
        return PagoDTO.desde(pago);
    }
}
```

------------------------------------------------------------------------

# 5.4 Frameworks / Infraestructura

Todo lo que cambia rápido: - Spring Boot - JPA / Hibernate - Bases de
datos - Servicios externos - AWS, RabbitMQ, Kafka - Cron Jobs -
Seguridad (JWT, OAuth2)

Ejemplo de infraestructura:

``` java
@Repository
public interface JpaPagoRepository extends JpaRepository<PagoEntity, Long> {}
```

------------------------------------------------------------------------

# 6. Beneficios de la Arquitectura Limpia

1.  Cambiar el ORM no afecta el dominio.
2.  Cambiar MySQL por MongoDB no rompe casos de uso.
3.  Cambiar REST por GraphQL no afecta reglas del negocio.
4.  El sistema es altamente testeable.
5.  El ciclo de vida del proyecto es largo, sano y escalable.
6.  Es fácil agregar funcionalidades mediante nuevos casos de uso.
7.  Código más legible y mantenible.

------------------------------------------------------------------------

# 7. Ejemplo real aplicado: Sistema de Pagos

Suponiendo tu microservicio de pagos:

### Entidades

-   Pago
-   Usuario
-   Transacción

### Casos de uso

-   Crear pago
-   Validar saldo
-   Aplicar tarifa
-   Enviar notificación al usuario

### Adaptadores

-   Controlador REST
-   Mapper Pago ↔ DTO
-   Repo JPA
-   Cliente HTTP para notificaciones

### Infraestructura

-   MySQL/H2/PostgreSQL
-   Spring Boot
-   Docker
-   AWS SNS/S3

------------------------------------------------------------------------

# 8. Fallos comunes que Clean Architecture evita

-   Acoplar el negocio a JPA\
-   Casos de uso dentro del controlador\
-   Crear entidades anémicas (solo getters/setters)\
-   Lógica mezclada con acceso a BD\
-   Servicios gigantes con miles de líneas\
-   Código dependiente de frameworks\
-   Dificultad para probar casos de uso

------------------------------------------------------------------------

# 9. ¿Cuándo usar Clean Architecture?

Recomendada para: - Microservicios con vida larga - Backend
empresarial - Sistemas de pagos - Plataformas de reservas -
Marketplaces - Apps escalables

No recomendada para: - Proyectos extremadamente pequeños o desechables -
Prototipos rápidos que no escalarán

------------------------------------------------------------------------

# 10. Conclusión

La Arquitectura Limpia permite:

-   Diseñar software desacoplado
-   Mantener el foco en el negocio
-   Evitar dependencias peligrosas
-   Ser independiente de frameworks y herramientas
-   Crear sistemas robustos, mantenibles y extensibles

Es una arquitectura moderna adoptada por empresas como: - Netflix\
- Uber\
- Mercado Libre\
- Amazon\
- Nubank\
- Rappi

Su propósito es claro:\
**Un software que resiste el tiempo, el cambio y la complejidad.**