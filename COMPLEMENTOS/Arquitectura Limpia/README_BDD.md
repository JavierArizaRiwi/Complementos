# Desarrollo Dirigido por el Comportamiento (BDD)

El Desarrollo Dirigido por el Comportamiento (BDD) es una metodología que utiliza historias en lenguaje natural para describir el comportamiento esperado de un sistema. Esto permite que todos los involucrados (negocio, QA, desarrolladores) tengan una comprensión clara de los requisitos.

## ¿Qué es Gherkin?
Gherkin es un lenguaje diseñado para describir el comportamiento esperado de un sistema utilizando una sintaxis sencilla y estructurada. Cada escenario se compone de:

- **Feature (Funcionalidad):** Describe el objetivo general.
- **Scenario (Escenario):** Define un caso específico.
- **Given (Dado):** El contexto inicial.
- **When (Cuando):** La acción que se realiza.
- **Then (Entonces):** El resultado esperado.

### Ejemplo de Escenarios en Gherkin

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

### Beneficios de BDD
- Facilita la comunicación entre equipos.
- Asegura que todos entiendan los requisitos.
- Proporciona una base para las pruebas automatizadas.