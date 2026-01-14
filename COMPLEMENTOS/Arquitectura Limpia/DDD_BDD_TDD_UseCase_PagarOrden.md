# Guía Completa: Caso de Uso Real - Pagar una Orden

## Dominio: Ecommerce

### 1. DDD — Modelar el Mundo Real

El primer paso en el Diseño Dirigido por el Dominio (DDD) es identificar los conceptos clave del negocio.

### ¿Qué cosas existen en el negocio?

#### Conceptos del Dominio
- **Order (Orden)**: Representa una orden de compra.
- **Money (Dinero)**: Un objeto de valor que encapsula la cantidad y la moneda.
- **Payment (Pago)**: Representa el acto de pagar una orden.
- **OrderStatus (Estado de la Orden)**: Define el estado actual de la orden (pendiente, pagada, etc.).

Reglas importantes:
- Nada de bases de datos ni frameworks en esta etapa.
- Solo el negocio.

#### Value Object: Money

Reglas del negocio:
- No puede ser negativo.
- Las operaciones deben ser en la misma moneda.

Ejemplos:
- `Money(10, USD)` (válido)
- `Money(-5, USD)` (inválido)

#### Entidad: Order

Reglas del negocio:
- Una orden tiene un total.
- Una orden no se puede pagar dos veces.
- Una orden pagada cambia de estado.

---

### 2. BDD — Describir el Comportamiento

El Desarrollo Dirigido por el Comportamiento (BDD) utiliza historias en lenguaje natural para describir qué debe pasar.

#### Escenario 1: Pagar Correctamente
- Dado que existe una orden con total $100.
- Y la orden está pendiente.
- Cuando el cliente paga la orden.
- Entonces la orden queda pagada.

Este escenario es entendible para:
- Negocio
- QA
- Desarrolladores

#### Escenario 2: Pagar Dos Veces
- Dado que existe una orden pagada.
- Cuando intento pagarla de nuevo.
- Entonces veo un error "orden ya pagada".

#### Escenario 3: Total Inválido
- Dado que una orden tiene total $0.
- Cuando intento pagarla.
- Entonces veo un error "total inválido".

Esto define qué debe pasar, no cómo.

---

### 3. TDD — Construir las Reglas (Tests)

El Desarrollo Dirigido por Pruebas (TDD) se enfoca en escribir pruebas antes de implementar las reglas.

#### Test 1: No Pagar Dos Veces
- Dado una orden pagada.
- Cuando `pay()`.
- Entonces lanza error.

Falla → No existe.
Hago lo mínimo.
Limpio.

#### Test 2: Cambiar Estado al Pagar
- Cuando `order.pay()`.
- Entonces `order.status == PAID`.

#### Test 3: Total Válido
- Cuando `order.total > 0`.
- Entonces pasa la validación.

Cada test impulsa el diseño. No escribes reglas “porque sí”.

---

### 4. Use Case (Hexagonal)

El caso de uso orquesta las acciones, pero no valida reglas del dominio.

#### Use Case: PayOrder

Responsabilidad:
1. Buscar la orden.
2. Pedirle que se pague.
3. Guardarla.

```java
order = repository.find(id);
order.pay();
repository.save(order);
```

#### ¿Qué NO va en el Use Case?
- Validar dinero.
- Verificar estado.
- Reglas de negocio.

Estas responsabilidades viven en:
- Value Objects
- Entidades

---

### 5. Cómo Encaja Todo (Mapa Mental)

1. BDD → Define historias.
2. DDD → Modela conceptos.
3. TDD → Construye reglas.
4. Use Case → Ejecuta la acción.

---

## Otro Caso Real: Crear Usuario

### BDD
- Dado un email válido.
- Cuando creo un usuario.
- Entonces el usuario queda registrado.

- Dado un email inválido.
- Cuando creo un usuario.
- Entonces veo error.

### DDD
- **Email** → Value Object.
- **User** → Entidad.

### TDD
- Test email inválido.
- Test email válido.
- Test usuario duplicado.

### Use Case
```java
new User(email);
repository.save(user);
```

---

## Señales de que lo Estás Haciendo Bien

- El use case es chico.
- Las reglas están en el dominio.
- Los tests te dicen qué escribir.
- El código se lee como negocio.

---

## Frase Final

- BDD acuerda el comportamiento.
- DDD protege el conocimiento.
- TDD garantiza que no lo rompas.

---

Si necesitas un ejemplo completo en código (Java, TypeScript, Python) o quieres modelar un caso real paso a paso, avísame.