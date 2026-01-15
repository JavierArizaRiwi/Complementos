# Ejemplo Práctico: Desarrollo Dirigido por Pruebas (TDD)

El Desarrollo Dirigido por Pruebas (TDD) es una metodología que se centra en escribir pruebas antes de implementar la funcionalidad. Sigue un ciclo iterativo conocido como **Red-Green-Refactor**:

1. **Red (Falla):** Escribir una prueba que falle porque la funcionalidad aún no existe.
2. **Green (Funciona):** Implementar el código mínimo necesario para que la prueba pase.
3. **Refactor (Refactorizar):** Mejorar el diseño del código manteniendo las pruebas en verde.

## Ejemplo: No Pagar Dos Veces

### Paso 1: Escribir el Test (Red)
```java
@Test
void noDebePermitirPagarDosVeces() {
    Order order = new Order(100);
    order.pay();

    assertThrows(IllegalStateException.class, () -> order.pay());
}
```
Este test fallará porque aún no hemos implementado la lógica para evitar pagos duplicados.

### Paso 2: Implementar el Código (Green)
```java
public void pay() {
    if (this.status == OrderStatus.PAID) {
        throw new IllegalStateException("La orden ya está pagada");
    }
    this.status = OrderStatus.PAID;
}
```
Ahora el test pasa.

### Paso 3: Refactorizar el Código
- Revisar nombres de métodos.
- Eliminar duplicación.
- Mejorar la legibilidad.

## Otros Tests

### Cambiar Estado al Pagar
```java
@Test
void debeCambiarEstadoAlPagar() {
    Order order = new Order(100);
    order.pay();

    assertEquals(OrderStatus.PAID, order.getStatus());
}
```

### Validar Total Positivo
```java
@Test
void totalDebeSerPositivo() {
    assertThrows(IllegalArgumentException.class, () -> new Order(0));
}
```

## Beneficios de TDD
- Código más limpio y mantenible.
- Menos errores en producción.
- Diseño impulsado por las pruebas.