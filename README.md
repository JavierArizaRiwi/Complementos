# README General de Introducción  
### Visión previa de los temas que se van a revisar

Este documento sirve como **introducción general** y como “antesala” a todas las guías técnicas que ya tienes:  
- **Arquitecturas limpias (Layered, Hexagonal, Onion, Clean Architecture)**  
- **Programación Reactiva con Spring WebFlux**  
- **Quarkus y sus modelos reactivos (Mutiny, RESTEasy Reactive)**  
- **Comparativa técnica entre Quarkus y Spring WebFlux**  
- **Monitorización con Prometheus + Grafana**

La idea es que este README funcione como **portada del repositorio** o como guía inicial para tus coders antes de entrar a cada contenido.

---

# 1. Objetivo general del aprendizaje

El objetivo de este conjunto de guías es llevar a los coders (y también a ti como líder técnico) a comprender:

1. **Cómo estructurar software moderno y escalable** usando arquitecturas limpias.  
2. **Cómo manejar alto tráfico y operaciones intensivas de I/O** con programación reactiva.  
3. **Cómo construir microservicios cloud-native** usando tanto **Spring WebFlux** como **Quarkus**.  
4. **Cómo monitorear, medir y observar los servicios** usando Prometheus y Grafana.  
5. **Cómo comparar tecnologías** y elegir la correcta para diferentes escenarios.

En conjunto, estos documentos crean un “mapa” de cómo debe pensar un desarrollador backend moderno.

---

# 2. Qué se revisará en cada módulo

## 2.1. Fundamentos de Arquitectura de Software

Verás las arquitecturas más utilizadas hoy:

- **Arquitectura en capas**  
- **Arquitectura Hexagonal (Ports & Adapters)**  
- **Onion Architecture**  
- **Clean Architecture**

Aquí se cubre:

- Cuándo usar cada una  
- Ventajas y desventajas  
- Estructuras de carpetas sugeridas  
- Cómo aislar el dominio y evitar dependencia con frameworks  
- Cómo aplicarlas en microservicios reales

👉 Este módulo sienta las bases de todo lo que viene después.

---

## 2.2. Programación Reactiva (teoría + práctica)

Se explican los conceptos fundamentales:

- Asincronía  
- Backpressure  
- Streams de datos  
- Programación declarativa  
- Flujo reactivo no bloqueante  
- Diferencias con programación bloqueante tradicional  

También verás:

- **Mono y Flux (Spring WebFlux)**  
- **Uni y Multi (Quarkus Mutiny)**  
- Composición de flujos  
- Manejo de errores  
- Operadores comunes  
- Casos de uso reales (APIs, eventos, pipelines)

👉 Este módulo prepara la mente del coder para WebFlux y Quarkus.

---

## 2.3. Spring WebFlux

Aquí te sumerges en:

- Versión reactiva del stack Spring  
- Controladores no bloqueantes  
- Routers y handlers funcionales  
- WebClient para llamadas reactivas  
- Integración con Mongo Reactive y R2DBC  
- Testing reactivo con StepVerifier y WebTestClient  

Liezo claro:

> *“Si ya conoces Spring MVC, aprender WebFlux es cambiar el tipo de retorno, no el framework.”*

---

## 2.4. Quarkus

Quarkus se explica desde cero:

- Qué es y por qué existe  
- Mutiny (API reactiva simple y elegante)  
- RESTEasy Reactive  
- Panache ORM / Reactive Panache  
- Sus ventajas en cloud-native  
- Compilación nativa con GraalVM  
- Uso en Kubernetes y microservicios escalables  

También se cubre:

- Diferencias prácticas con Spring Boot  
- Cómo crear servicios extremadamente ligeros  
- Cuándo elegir Quarkus en lugar de Spring

---

## 2.5. Comparativa Quarkus vs WebFlux

Se explican las diferencias reales:

| Concepto | WebFlux | Quarkus |
|----------|---------|----------|
| ¿Qué es? | Módulo reactivo dentro de Spring | Framework completo |
| Reactivo | Reactor | Mutiny |
| Enfoque | Reactividad dentro de Spring | Cloud-native + ultrarrápido |
| Ideal para | Equipos Spring | Kubernetes / serverless |
| Rendimiento | Muy bueno | Excelente (especialmente nativo) |

La idea es que el coder pueda responder:

> *“¿Cuál uso? ¿Para qué tipo de proyecto sirve cada uno?”*

---

## 2.6. Monitorización (Prometheus + Grafana)

Para cerrar la formación, se revisan herramientas indispensables:

- Exponer métricas con Spring Actuator  
- Configurar Prometheus para scrapear servicios  
- Levantar Grafana para visualizar dashboards  
- Importar dashboards para Spring Boot  
- Integrar métricas de rendimiento, tráfico y memoria  
- Entender health checks y observabilidad  

Este módulo enseña cómo operar microservicios en producción.

---

# 3. Qué aprenderá el coder al final

Al finalizar todos los módulos, cualquier coder será capaz de:

### ✔ Diseñar microservicios con arquitecturas limpias  
### ✔ Crear APIs reactivas con WebFlux o con Quarkus  
### ✔ Entender cuándo usar un stack reactivo  
### ✔ Comprender la diferencia entre modelos imperativos y reactivos  
### ✔ Optimizar microservicios para Kubernetes / cloud  
### ✔ Monitorizar sus servicios profesionalmente  
### ✔ Entender a nivel arquitectónico Spring WebFlux y Quarkus

Y lo más importante:

> **Tendrá mentalidad moderna de arquitectura backend**, no solo habilidades técnicas aisladas.

---

# 4. ¿Cómo se recomienda estudiar este repositorio?

Orden recomendado:

1. **README general (este)**  
2. **Arquitecturas limpias**  
3. **Programación reactiva**  
4. **Spring WebFlux**  
5. **Quarkus**  
6. **Comparativa**  
7. **Monitorización**  
8. (Opcional) Taller o proyecto de práctica

---

# 5. Resumen final

Este README explica la visión general de lo que se estudiará:

- Arquitecturas limpias  
- Reactividad  
- WebFlux  
- Quarkus  
- Comparación técnica  
- Observabilidad y monitoreo  

Todo orientado a formar a un developer backend **moderno, escalable y cloud-native**.

---

# ¡Listo!  
Este documento sirve como “portada” para todo el material técnico que ya tienes.  
Si quieres también te genero una **versión PDF** o una **slide deck** para presentarlo en tu cohorte.