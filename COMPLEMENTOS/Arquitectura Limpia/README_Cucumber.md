# Configuración y Uso de Cucumber

Cucumber es una herramienta que permite automatizar pruebas basadas en el comportamiento (BDD) utilizando el lenguaje Gherkin. A continuación, se describe cómo configurar y usar Cucumber en un proyecto Java.

## 1. Configuración de Cucumber

### Dependencias
Agrega las siguientes dependencias a tu archivo `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>7.14.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-junit</artifactId>
        <version>7.14.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Estructura del Proyecto
Asegúrate de tener la siguiente estructura de carpetas:

```
src/test/java
    ├── features
    │   └── pagar_orden.feature
    ├── stepdefinitions
    │   └── PagarOrdenSteps.java
    └── runners
        └── RunCucumberTest.java
```

## 2. Escribir Escenarios en Gherkin
Crea un archivo `.feature` en la carpeta `features`:

**Archivo:** `pagar_orden.feature`
```gherkin
Feature: Pagar una orden

  Scenario: Pagar correctamente
    Given una orden con total $100
    And la orden está pendiente
    When el cliente paga la orden
    Then la orden queda pagada

  Scenario: Pagar dos veces
    Given una orden pagada
    When intento pagarla de nuevo
    Then veo un error "orden ya pagada"
```

## 3. Implementar los Pasos (Step Definitions)
Crea una clase en la carpeta `stepdefinitions` para implementar los pasos:

**Archivo:** `PagarOrdenSteps.java`
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import static org.junit.Assert.*;

public class PagarOrdenSteps {

    private double total;
    private String estado;

    @Given("una orden con total \$(\d+)")
    public void unaOrdenConTotal(double total) {
        this.total = total;
        this.estado = "pendiente";
    }

    @Given("la orden está pendiente")
    public void laOrdenEstaPendiente() {
        assertEquals("pendiente", estado);
    }

    @When("el cliente paga la orden")
    public void elClientePagaLaOrden() {
        if (estado.equals("pendiente")) {
            estado = "pagada";
        }
    }

    @Then("la orden queda pagada")
    public void laOrdenQuedaPagada() {
        assertEquals("pagada", estado);
    }

    @Given("una orden pagada")
    public void unaOrdenPagada() {
        this.estado = "pagada";
    }

    @When("intento pagarla de nuevo")
    public void intentoPagarlaDeNuevo() {
        if (estado.equals("pagada")) {
            throw new IllegalStateException("orden ya pagada");
        }
    }

    @Then("veo un error \"orden ya pagada\"")
    public void veoUnErrorOrdenYaPagada() {
        try {
            intentoPagarlaDeNuevo();
        } catch (IllegalStateException e) {
            assertEquals("orden ya pagada", e.getMessage());
        }
    }
}
```

## 4. Configurar el Runner
Crea una clase en la carpeta `runners` para ejecutar las pruebas:

**Archivo:** `RunCucumberTest.java`
```java
package runners;

import org.junit.runner.RunWith;
import io.cucumber.junit.Cucumber;
import io.cucumber.junit.CucumberOptions;

@RunWith(Cucumber.class)
@CucumberOptions(
    features = "src/test/java/features",
    glue = "stepdefinitions"
)
public class RunCucumberTest {
}
```

## 5. Ejecutar las Pruebas
Ejecuta las pruebas con Maven:

```bash
mvn test
```

## Beneficios de Cucumber
- Facilita la colaboración entre equipos.
- Proporciona documentación viva del sistema.
- Automatiza pruebas de aceptación basadas en el comportamiento.