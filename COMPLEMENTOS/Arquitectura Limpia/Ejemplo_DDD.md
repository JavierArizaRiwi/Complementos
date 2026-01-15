# Ejemplo Práctico: Diseño Dirigido por el Dominio (DDD)

El Diseño Dirigido por el Dominio (DDD) se centra en modelar el dominio del negocio, identificando conceptos clave y reglas que reflejen la realidad del sistema.

## Conceptos del Dominio

### Entidad: Orden (Order)
- **Reglas del negocio:**
  - Una orden tiene un total.
  - Una orden no se puede pagar dos veces.
  - Una orden pagada cambia de estado.

```java
public class Order {
    private double total;
    private OrderStatus status;

    public Order(double total) {
        if (total <= 0) {
            throw new IllegalArgumentException("El total debe ser mayor a 0");
        }
        this.total = total;
        this.status = OrderStatus.PENDING;
    }

    public void pay() {
        if (this.status == OrderStatus.PAID) {
            throw new IllegalStateException("La orden ya está pagada");
        }
        this.status = OrderStatus.PAID;
    }

    public OrderStatus getStatus() {
        return status;
    }
}
```

### Value Object: Dinero (Money)
- **Reglas del negocio:**
  - No puede ser negativo.
  - Las operaciones deben ser en la misma moneda.

```java
public class Money {
    private final double amount;
    private final String currency;

    public Money(double amount, String currency) {
        if (amount < 0) {
            throw new IllegalArgumentException("El monto no puede ser negativo");
        }
        this.amount = amount;
        this.currency = currency;
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Las monedas deben coincidir");
        }
        return new Money(this.amount + other.amount, this.currency);
    }
}
```

## Beneficios de DDD
- Enfoque en el negocio, no en la tecnología.
- Código más alineado con los requisitos del dominio.
- Facilita la comunicación entre desarrolladores y expertos del dominio.