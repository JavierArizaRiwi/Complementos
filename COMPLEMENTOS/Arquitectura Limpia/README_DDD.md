# Guía Práctica de Clean Architecture + DDD 

## 1. ¿Qué problema resolvemos?
En muchos proyectos el código crece sin orden: lógica mezclada con base de datos, reglas de negocio dentro de controladores, dificultad para cambiar tecnologías.

Clean Architecture y Domain‑Driven Design (DDD) buscan:
- Separar responsabilidades.
- Proteger el negocio del detalle técnico.
- Facilitar mantenimiento, pruebas y evolución.

---

## 2. Conceptos Clave Explicados Simple

### 2.1 Dominio
Es el *corazón del negocio*.  
Ejemplo: Cotización, Proyecto, Orden de Fabricación, Mantenimiento.

### 2.2 Entidad
Objeto con identidad propia.  
Ej: Proyecto(id, cliente, estado)

### 2.3 Value Object
Objeto sin identidad, describe un valor.  
Ej: Dinero(monto, moneda), Fecha, Dirección.

### 2.4 Caso de Uso
Acción del sistema.  
Ej: Crear cotización, Aprobar orden, Programar mantenimiento.

### 2.5 Puerto (Interface)
Contrato que define qué necesita el negocio.  
Ej: RepositorioDeProyectos.

### 2.6 Adaptador
Implementación técnica.  
Ej: RepositorioMySQL, API REST, Cola de Mensajes.

---

## 3. Capas de Clean Architecture

```
[ UI / API ]
     |
[ Application (Casos de Uso) ]
     |
[ Domain (Reglas de Negocio) ]
     |
[ Infrastructure (DB, APIs, Frameworks) ]
```

El dominio nunca conoce frameworks ni bases de datos.

---

## 4. Estructura de Carpetas Agnóstica

```
src/
 ├── domain/
 │   ├── entities/
 │   ├── valueobjects/
 │   ├── repositories/
 │   └── services/
 ├── application/
 │   ├── usecases/
 │   └── dto/
 ├── infrastructure/
 │   ├── persistence/
 │   ├── messaging/
 │   └── controllers/
 └── config/
```

---

## 5. Ejemplo de Flujo Real

### Caso: Crear Cotización

1. Usuario envía datos desde la UI.
2. Controlador llama a `CrearCotizacionUseCase`.
3. Caso de uso valida reglas.
4. Entidad Cotizacion se crea.
5. Repositorio guarda en DB.
6. Evento `CotizacionCreada` se publica.

---

## 6. Bounded Context (DDD)

Separar dominios:

- Ventas
- Producción
- Mantenimiento
- Facturación

Cada uno con su propio modelo.

---

## 7. Eventos de Dominio

Ejemplo:
```
Evento: OrdenFabricacionAprobada
Acciones:
- Notificar producción
- Generar orden de compra
```

---

## 8. Reglas de Oro

1. El dominio no depende de frameworks.
2. Los casos de uso no saben de HTTP.
3. La infraestructura es reemplazable.
4. Todo se prueba desde el dominio.

---

## 9. Beneficios

- Código limpio.
- Escalable.
- Fácil de testear.
- Preparado para microservicios.
- Ideal para arquitecturas modernas.

---

## 10. Otros Conceptos Relacionados

### 10.1 TDD (Test-Driven Development)
Es una metodología de desarrollo basada en escribir pruebas antes de implementar el código.  
Pasos:
1. Escribir una prueba que falle.
2. Implementar el código mínimo para pasar la prueba.
3. Refactorizar el código manteniendo las pruebas verdes.

Beneficios:
- Código más confiable.
- Facilita el diseño.
- Reduce errores en producción.

### 10.2 BDD (Behavior-Driven Development)
Extensión de TDD enfocada en el comportamiento del sistema.  
Se utilizan descripciones en lenguaje natural para definir las pruebas.  
Ejemplo:
```
Dado un usuario autenticado,
Cuando solicita ver su perfil,
Entonces el sistema muestra su información.
```

### 10.3 Arquitectura Hexagonal
También conocida como "Ports and Adapters".  
Separa el núcleo de la aplicación de los detalles técnicos.  
Ventajas:
- Independencia de frameworks.
- Fácil de probar.
- Adaptable a cambios tecnológicos.

### 10.4 CQRS (Command Query Responsibility Segregation)
Divide las operaciones de lectura y escritura en modelos separados.  
Ideal para sistemas con alta concurrencia o necesidades de escalabilidad.

### 10.5 Event Sourcing
Modelo donde el estado del sistema se reconstruye a partir de eventos.  
Ventajas:
- Historial completo de cambios.
- Facilita auditorías y debugging.

---

## 11. Evolución para Nivel Mid

Agregar:
- CQRS
- Event Sourcing
- Arquitectura Hexagonal
- Microservicios por Bounded Context

---

Fin de la guía.