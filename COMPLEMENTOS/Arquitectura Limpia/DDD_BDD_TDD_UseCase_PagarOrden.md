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

El Desarrollo Dirigido por el Comportamiento (BDD) utiliza historias en lenguaje natural para describir qué debe pasar. Estas historias suelen escribirse en un formato estándar llamado **Gherkin**, que permite estructurar los escenarios de manera clara y comprensible para todos los involucrados.

#### ¿Qué es Gherkin?
Gherkin es un lenguaje diseñado para describir el comportamiento esperado de un sistema utilizando una sintaxis sencilla y estructurada. Cada escenario se compone de:

- **Feature (Funcionalidad):** Describe el objetivo general.
- **Scenario (Escenario):** Define un caso específico.
- **Given (Dado):** El contexto inicial.
- **When (Cuando):** La acción que se realiza.
- **Then (Entonces):** El resultado esperado.

#### Ejemplo de Escenarios en Gherkin

**Feature:** Pagar una orden

**Scenario 1: Pagar Correctamente**
```
Given una orden con total $100
And la orden está pendiente
When el cliente paga la orden
Then la orden queda pagada
```

**Scenario 2: Pagar Dos Veces**
```
Given una orden pagada
When intento pagarla de nuevo
Then veo un error "orden ya pagada"
```

**Scenario 3: Total Inválido**
```
Given una orden con total $0
When intento pagarla
Then veo un error "total inválido"
```

Este formato es entendible para:
- Negocio
- QA
- Desarrolladores

---

### 3. TDD — Construir las Reglas (Tests)

El Desarrollo Dirigido por Pruebas (TDD) se enfoca en escribir pruebas antes de implementar las reglas. Este enfoque sigue un ciclo iterativo conocido como **Red-Green-Refactor**:

1. **Red (Falla):** Escribir una prueba que falle porque la funcionalidad aún no existe.
2. **Green (Funciona):** Implementar el código mínimo necesario para que la prueba pase.
3. **Refactor (Refactorizar):** Mejorar el diseño del código manteniendo las pruebas en verde.

#### Ejemplo del Ciclo TDD

**Test 1: No Pagar Dos Veces**
- **Dado** una orden pagada.
- **Cuando** `pay()`.
- **Entonces** lanza error.

1. **Red:** Escribir el test:
```java
@Test
void noDebePermitirPagarDosVeces() {
    Order order = new Order(100);
    order.pay();

    assertThrows(IllegalStateException.class, () -> order.pay());
}
```
Este test fallará porque aún no hemos implementado la lógica para evitar pagos duplicados.

2. **Green:** Implementar el código mínimo:
```java
public void pay() {
    if (this.status == OrderStatus.PAID) {
        throw new IllegalStateException("La orden ya está pagada");
    }
    this.status = OrderStatus.PAID;
}
```
Ahora el test pasa.

3. **Refactor:** Mejorar el diseño del código:
- Revisar nombres de métodos, eliminar duplicación, etc.

#### Otros Tests

**Test 2: Cambiar Estado al Pagar**
- **Cuando** `order.pay()`.
- **Entonces** `order.status == PAID`.

**Test 3: Total Válido**
- **Cuando** `order.total > 0`.
- **Entonces** pasa la validación.

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