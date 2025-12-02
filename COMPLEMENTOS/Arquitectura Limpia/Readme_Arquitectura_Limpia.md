# Guía práctica de arquitecturas limpias en backend

Esta guía resume las **arquitecturas limpias más populares**, cómo se implementan en proyectos típicos (por ejemplo, Java/Spring Boot) y sus **ventajas / desventajas**.  
La idea es que te sirva como README teórico‑práctico para ti y tus coders.

---

## 1. ¿Qué es una arquitectura “limpia”?

Una arquitectura limpia busca principalmente:

- **Bajo acoplamiento**: que la lógica de negocio no dependa de frameworks, DBs, ni detalles externos.  
- **Alta cohesión**: cada módulo tiene una responsabilidad clara.  
- **Testabilidad**: dominio testeable sin levantar contenedores, DB, ni servidores.  
- **Sustituibilidad**: poder cambiar tecnologías (DB, framework web, mensajería) con impacto mínimo.

La idea clave:  
> **El dominio y las reglas de negocio son el centro. Todo lo demás es detalle de infraestructura.**

---

## 2. Arquitecturas más usadas

Veremos estas 4:

1. **Arquitectura en capas (Layered / N-Tier)** – el clásico de toda la vida.  
2. **Hexagonal (Ports & Adapters)** – muy usada en microservicios.  
3. **Onion Architecture** – prima hermana de hexagonal, con enfoque en círculos concéntricos.  
4. **Clean Architecture (Uncle Bob)** – refinamiento que combina ideas de onion + hexagonal.

> Nota: Las 3 últimas suelen agruparse bajo el paraguas de “arquitecturas limpias”.

---

## 3. Arquitectura en capas (Layered Architecture)

### 3.1. Idea general

Separar la app en capas horizontales:

- **Presentación / API** (controllers, views)  
- **Servicio / Negocio** (services)  
- **Persistencia / Datos** (repositories, DAOs)  
- + opcionalmente **capa de dominio** explícita (entities, DTOs)

Flujo típico:

`Controller → Service → Repository → DB`

### 3.2. Ejemplo de estructura (Java / Spring Boot)

```text
com.riwitienda.pagos
 ├─ web
 │   └─ controller
 ├─ service
 │   └─ impl
 ├─ domain
 │   ├─ model
 │   └─ dto
 └─ repository
```

### 3.3. Ventajas

- Muy **fácil de entender** para juniors.  
- Soportada por casi todos los ejemplos y tutoriales.  
- Ideal para **aplicaciones CRUD simples**.  
- Menos código “ceremonial”.

### 3.4. Desventajas

- El dominio suele **depender de frameworks** (anotaciones de JPA, etc.).  
- Riesgo de que **todo termine en la capa service** como “cajón de sastre”.  
- Difícil de **testear reglas de dominio aisladas**; muchas veces se mokea media aplicación.  
- A medida que crece el proyecto, es fácil romper la separación de responsabilidades.

### 3.5. Cuándo usarla

- Proyectos internos pequeños / medianos.  
- MVPs, PoCs o servicios CRUD sin lógica de negocio compleja.  
- Equipos con poco tiempo de aprendizaje en arquitectura.

---

## 4. Arquitectura Hexagonal (Ports & Adapters)

### 4.1. Idea general

Organizar la app alrededor del **dominio** y de **“puertos”** (interfaces) que definen cómo el exterior interactúa con él.  
Los **adaptadores** implementan esos puertos y conectan con el mundo externo (DB, HTTP, cola, etc.).

Core:

- **Dominios + Casos de uso** (aplicación) en el centro.  
- **Puertos de entrada**: cómo el exterior llama a la app (REST, eventos…).  
- **Puertos de salida**: cómo la app habla con el exterior (repositorios, brokers…).  
- **Adaptadores**: implementaciones concretas de esos puertos.

### 4.2. Ejemplo de estructura (Java / Spring Boot)

```text
com.riwitienda.pagos
 ├─ application
 │   ├─ port
 │   │   ├─ in      (interfaces de casos de uso)
 │   │   └─ out     (interfaces de repositorios, gateways…)
 │   └─ service     (implementación de casos de uso, orquestación)
 ├─ domain
 │   ├─ model       (entidades de dominio, value objects)
 │   └─ events      (eventos de dominio, si aplica)
 └─ infrastructure
     ├─ web         (controllers REST → adaptadores de entrada)
     ├─ persistence (JpaRepository, mappers → adaptadores de salida)
     └─ config      (config Spring, beans, seguridad…)
```

### 4.3. Ventajas

- **Domino independiente** de frameworks y de la base de datos.  
- Alta **testabilidad**: se testean casos de uso y dominio sin infraestructura.  
- Permite cambiar DB, transporte (REST → gRPC, etc.) con impacto acotado.  
- Muy buena para **microservicios de negocio**.

### 4.4. Desventajas

- **Más compleja de entender** al inicio (más carpetas, más interfaces).  
- Genera más “código ceremonial” (interfaces, mappers, DTOs).  
- Si el dominio es muy simple, puede sentirse **sobrediseñada**.

### 4.5. Cuándo usarla

- Microservicios con lógica de negocio **moderada a compleja**.  
- Sistemas donde sabes que puede cambiar la infraestructura (DB, colas).  
- Equipos que priorizan testabilidad, mantenibilidad y clean code.

---

## 5. Onion Architecture

### 5.1. Idea general

Muy similar a Hexagonal, pero representada en **capas concéntricas** (anillos):

1. **Dominio** (entidades, value objects) – núcleo.  
2. **Servicios de dominio / aplicación** – reglas de negocio y casos de uso.  
3. **Interfaces (repositorios, gateways)** – contratos hacia afuera.  
4. **Infraestructura** – implementaciones concretas (DB, UI, API, etc.).

Regla de oro:

> Las capas externas pueden depender de las internas, pero **nunca al revés**.

### 5.2. Diferencia práctica vs Hexagonal

- Hexagonal enfatiza **puertos y adaptadores**.  
- Onion enfatiza la **dirección de dependencias** y los **anillos**.  
- En la práctica, muchas implementaciones son una mezcla de ambas.

### 5.3. Ventajas

- Misma mayoría de beneficios que hexagonal:
  - dominio puro  
  - testabilidad alta  
  - desac acoplamiento entre dominio e infraestructura
- Enfatiza de forma muy clara la **regla de dependencias**.

### 5.4. Desventajas

- A efectos prácticos, es casi igual de compleja que hexagonal.  
- Puede ser demasiado abstracta para equipos juniors.

### 5.5. Cuándo usarla

- Cuando quieras **explicar visualmente** la idea de que el dominio es el centro.  
- Proyectos grandes donde la regla de dependencias deba ser muy clara.

---

## 6. Clean Architecture (Uncle Bob)

### 6.1. Idea general

Clean Architecture es un refinamiento que combina:

- Capas concéntricas (Onion)  
- Puertos y adaptadores (Hexagonal)  
- Principios SOLID

Capas típicas:

1. **Entities (Enterprise Business Rules)** – dominio puro.  
2. **Use Cases (Application Business Rules)** – orquestan casos de uso.  
3. **Interface Adapters** – adaptan datos de/para frameworks, UI, DB.  
4. **Frameworks & Drivers** – la capa más externa (Spring, DB, UI, etc.).

### 6.2. Ejemplo de estructura (Java / Spring Boot)

```text
com.riwitienda.pagos
 ├─ domain           (entidades, value objects, reglas de negocio)
 ├─ application
 │   ├─ usecase      (casos de uso)
 │   └─ port
 │       ├─ in
 │       └─ out
 └─ infrastructure
     ├─ web          (controllers, DTOs REST)
     ├─ persistence  (JPA, mappers)
     ├─ config
     └─ messaging    (Kafka, RabbitMQ, etc.)
```

### 6.3. Ventajas

- Marco mental **muy claro** para proyectos grandes.  
- Mantiene al dominio **independiente de detalles técnicos**.  
- Facilita aplicar SOLID, DDD, pruebas unitarias y de integración limpias.  
- Ideal para **equipos senior** o proyectos donde la mantenibilidad es clave.

### 6.4. Desventajas

- **Curva de aprendizaje alta**; puede abrumar a juniors.  
- Más “boilerplate” (interfaces, DTOs, mappers, etc.).  
- No vale la pena para APIs CRUD muy pequeñas.

### 6.5. Cuándo usarla

- Sistemas estratégicos, con vida útil larga y fuerte evolución.  
- Productos donde se espera crecimiento de equipo y del código.  
- Cuando ya dominas layered y hexagonal, y quieres dar el siguiente paso.

---

## 7. Comparativa rápida

### 7.1. Tabla conceptual

| Arquitectura      | Complejidad | Acoplamiento a frameworks | Testabilidad | Ideal para                          |
|-------------------|-------------|---------------------------|-------------|-------------------------------------|
| Layered clásica   | Baja        | Alto                      | Media       | CRUD simples, MVP, prototipos      |
| Hexagonal         | Media       | Bajo                      | Alta        | Microservicios de negocio           |
| Onion             | Media       | Bajo                      | Alta        | Dominios grandes, visión concéntrica|
| Clean Architecture| Alta        | Muy bajo                  | Muy alta    | Sistemas grandes y de larga vida    |

### 7.2. Ventajas globales de arquitecturas limpias (Hexagonal / Onion / Clean)

- Dominios **más expresivos y ricos**.  
- Cambios de infraestructura menos dolorosos.  
- Mejor base para **DDD, CQRS, Event Sourcing**, etc.  
- Testing más barato: muchos tests de dominio sin levantar el mundo.

### 7.3. Desventajas globales

- Más código e interfaces.  
- Requiere **disciplina de equipo** (si no, se vuelve un “Frankenstein”).  
- Para proyectos pequeños puede ser “matar moscas a cañonazos”.

---

## 8. Recomendación práctica para ti y tus coders

1. **Nivel 1 – Dominar bien la arquitectura por capas clásica**  
   - Responsabilidades claras por capa.  
   - Evitar meter toda la lógica en `ServiceImpl`.  
   - Separar DTOs de entidades.

2. **Nivel 2 – Introducir Hexagonal poco a poco**  
   - Empezar usando puertos (interfaces) para repositorios y gateways externos.  
   - Mover la lógica de negocio a servicios de aplicación más puros.  
   - Mantener el dominio sin dependencias de Spring ni JPA cuando sea posible.

3. **Nivel 3 – Evolucionar a Clean Architecture en proyectos grandes**  
   - Añadir capas de `usecase`, `port.in`, `port.out`.  
   - Dividir el proyecto en módulos (si el tamaño lo justifica).  
   - Integrar DDD táctico (agregados, value objects, dominios ricos).

> Consejo: **no todo proyecto necesita la arquitectura más avanzada**.  
> El truco está en elegir la más simple que siga siendo sostenible en el tiempo.

---

## 9. Plantilla mínima para iniciar un proyecto “limpio”

Si quieres arrancar un proyecto relativamente limpio (mezcla Layered + Hexagonal) sin complicarte demasiado:

```text
com.riwitienda.pagos
 ├─ domain
 │   ├─ model          (entidades, value objects)
 │   └─ service        (reglas de dominio puras, si aplica)
 ├─ application
 │   ├─ port
 │   │   ├─ in         (casos de uso)
 │   │   └─ out        (repositorios, gateways…)
 │   └─ service        (implementación de casos de uso)
 └─ infrastructure
     ├─ web            (controllers, DTOs de entrada/salida)
     ├─ persistence    (JPA, mappers)
     └─ config         (Spring, seguridad, etc.)
```

Con eso ya tienes:

- Una base que suena a **Clean Architecture**,  
- Pero lo bastante simple para empezar a trabajar con tu equipo sin bloquearlos.

---

Fin de la guía.