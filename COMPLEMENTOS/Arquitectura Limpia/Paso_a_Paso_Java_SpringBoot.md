# Paso a Paso: Aplicar Clean Architecture + DDD con Java y Spring Boot

## 1. Configuración Inicial del Proyecto

1. **Crear un proyecto Spring Boot**:
   - Usar [Spring Initializr](https://start.spring.io/) para generar un proyecto con las siguientes dependencias:
     - Spring Web
     - Spring Data JPA
     - H2 Database (o cualquier base de datos de tu elección)
     - Lombok (opcional, para reducir el boilerplate)

2. **Estructura de Carpetas**:
   Organiza el proyecto siguiendo las capas de Clean Architecture:
   ```
   src/
    ├── main/
    │   ├── java/
    │   │   ├── com.ejemplo.proyecto/
    │   │   │   ├── domain/
    │   │   │   │   ├── entities/
    │   │   │   │   ├── valueobjects/
    │   │   │   │   ├── repositories/
    │   │   │   │   └── services/
    │   │   │   ├── application/
    │   │   │   │   ├── usecases/
    │   │   │   │   └── dto/
    │   │   │   ├── infrastructure/
    │   │   │   │   ├── persistence/
    │   │   │   │   ├── messaging/
    │   │   │   │   └── controllers/
    │   │   │   └── config/
    │   └── resources/
    │       └── application.yml
   ```

---

## 2. Implementación del Dominio

1. **Crear Entidades**:
   Define las entidades principales del dominio en `domain/entities`.
   ```java
   package com.ejemplo.proyecto.domain.entities;

   import java.math.BigDecimal;

   public class Producto {
       private Long id;
       private String nombre;
       private BigDecimal precio;

       // Constructor, getters y setters
   }
   ```

2. **Crear Value Objects**:
   Define objetos de valor en `domain/valueobjects`.
   ```java
   package com.ejemplo.proyecto.domain.valueobjects;

   public class Dinero {
       private final BigDecimal monto;
       private final String moneda;

       public Dinero(BigDecimal monto, String moneda) {
           this.monto = monto;
           this.moneda = moneda;
       }

       // Getters y lógica de validación
   }
   ```

3. **Definir Interfaces de Repositorios**:
   Define contratos en `domain/repositories`.
   ```java
   package com.ejemplo.proyecto.domain.repositories;

   import com.ejemplo.proyecto.domain.entities.Producto;
   import java.util.List;

   public interface ProductoRepositorio {
       List<Producto> obtenerTodos();
       Producto guardar(Producto producto);
   }
   ```

---

## 3. Implementación de Casos de Uso

1. **Crear Casos de Uso**:
   Implementa la lógica de negocio en `application/usecases`.
   ```java
   package com.ejemplo.proyecto.application.usecases;

   import com.ejemplo.proyecto.domain.entities.Producto;
   import com.ejemplo.proyecto.domain.repositories.ProductoRepositorio;

   public class CrearProductoUseCase {
       private final ProductoRepositorio repositorio;

       public CrearProductoUseCase(ProductoRepositorio repositorio) {
           this.repositorio = repositorio;
       }

       public Producto ejecutar(Producto producto) {
           // Validar reglas de negocio
           return repositorio.guardar(producto);
       }
   }
   ```

2. **Crear DTOs**:
   Define objetos de transferencia de datos en `application/dto`.
   ```java
   package com.ejemplo.proyecto.application.dto;

   public class ProductoDTO {
       private String nombre;
       private String precio;

       // Constructor, getters y setters
   }
   ```

---

## 4. Implementación de la Infraestructura

1. **Implementar Repositorios**:
   Crea implementaciones técnicas en `infrastructure/persistence`.
   ```java
   package com.ejemplo.proyecto.infrastructure.persistence;

   import com.ejemplo.proyecto.domain.entities.Producto;
   import com.ejemplo.proyecto.domain.repositories.ProductoRepositorio;
   import org.springframework.stereotype.Repository;

   import java.util.ArrayList;
   import java.util.List;

   @Repository
   public class ProductoRepositorioImpl implements ProductoRepositorio {
       private final List<Producto> productos = new ArrayList<>();

       @Override
       public List<Producto> obtenerTodos() {
           return productos;
       }

       @Override
       public Producto guardar(Producto producto) {
           productos.add(producto);
           return producto;
       }
   }
   ```

2. **Crear Controladores**:
   Define endpoints en `infrastructure/controllers`.
   ```java
   package com.ejemplo.proyecto.infrastructure.controllers;

   import com.ejemplo.proyecto.application.usecases.CrearProductoUseCase;
   import com.ejemplo.proyecto.application.dto.ProductoDTO;
   import org.springframework.web.bind.annotation.*;

   @RestController
   @RequestMapping("/productos")
   public class ProductoController {
       private final CrearProductoUseCase crearProductoUseCase;

       public ProductoController(CrearProductoUseCase crearProductoUseCase) {
           this.crearProductoUseCase = crearProductoUseCase;
       }

       @PostMapping
       public String crearProducto(@RequestBody ProductoDTO productoDTO) {
           // Convertir DTO a entidad y ejecutar caso de uso
           return "Producto creado con éxito";
       }
   }
   ```

---

## 5. Configuración y Pruebas

1. **Configurar Spring Boot**:
   - Define las propiedades en `application.yml`.
   ```yaml
   spring:
     datasource:
       url: jdbc:h2:mem:testdb
       driver-class-name: org.h2.Driver
       username: sa
       password: password
     jpa:
       hibernate:
         ddl-auto: update
   ```

2. **Escribir Pruebas**:
   - Usa JUnit y Mockito para probar los casos de uso y controladores.
   ```java
   @SpringBootTest
   public class CrearProductoUseCaseTest {
       @Test
       public void testCrearProducto() {
           // Escribir pruebas unitarias
       }
   }
   ```

---

## 6. Beneficios

- **Separación de responsabilidades**: Cada capa tiene un propósito claro.
- **Facilidad de pruebas**: El dominio y los casos de uso son independientes.
- **Escalabilidad**: Preparado para microservicios y arquitecturas modernas.

---

## 7. Aplicando TDD y BDD en Proyectos Java

### 7.1 Test-Driven Development (TDD)

TDD es una metodología que consiste en escribir pruebas antes de implementar el código. Los pasos básicos son:

1. **Escribir una prueba que falle**: Define el comportamiento esperado.
2. **Implementar el código mínimo**: Escribe el código necesario para que la prueba pase.
3. **Refactorizar**: Mejora el código manteniendo las pruebas en verde.

#### Ejemplo de TDD con JUnit

1. **Escribir la prueba**:
   ```java
   package com.ejemplo.proyecto;

   import com.ejemplo.proyecto.domain.entities.Producto;
   import org.junit.jupiter.api.Test;

   import static org.junit.jupiter.api.Assertions.*;

   public class ProductoTest {
       @Test
       public void testCrearProducto() {
           Producto producto = new Producto(1L, "Laptop", BigDecimal.valueOf(1500));
           assertEquals("Laptop", producto.getNombre());
           assertEquals(BigDecimal.valueOf(1500), producto.getPrecio());
       }
   }
   ```

2. **Implementar el código mínimo**:
   ```java
   package com.ejemplo.proyecto.domain.entities;

   import java.math.BigDecimal;

   public class Producto {
       private Long id;
       private String nombre;
       private BigDecimal precio;

       public Producto(Long id, String nombre, BigDecimal precio) {
           this.id = id;
           this.nombre = nombre;
           this.precio = precio;
       }

       public String getNombre() {
           return nombre;
       }

       public BigDecimal getPrecio() {
           return precio;
       }
   }
   ```

3. **Refactorizar**: Mejora el diseño del código si es necesario.

---

### 7.2 Behavior-Driven Development (BDD)

BDD se enfoca en el comportamiento del sistema, utilizando un lenguaje natural para describir las pruebas. Herramientas como Cucumber son ideales para implementar BDD en Java.

#### Ejemplo de BDD con Cucumber

1. **Definir el escenario en Gherkin**:
   Crea un archivo `.feature` en `src/test/resources/features`.
   ```gherkin
   Feature: Gestión de Productos
     Scenario: Crear un nuevo producto
       Given un producto con nombre "Laptop" y precio "1500"
       When el producto es guardado
       Then el producto debe estar disponible en el sistema
   ```

2. **Implementar los pasos en Java**:
   Crea una clase en `src/test/java` para definir los pasos.
   ```java
   package com.ejemplo.proyecto.steps;

   import com.ejemplo.proyecto.domain.entities.Producto;
   import io.cucumber.java.en.*;

   import static org.junit.jupiter.api.Assertions.*;

   public class ProductoSteps {
       private Producto producto;

       @Given("un producto con nombre {string} y precio {string}")
       public void crearProducto(String nombre, String precio) {
           producto = new Producto(1L, nombre, BigDecimal.valueOf(Double.parseDouble(precio)));
       }

       @When("el producto es guardado")
       public void guardarProducto() {
           // Simular guardado (en memoria o base de datos)
       }

       @Then("el producto debe estar disponible en el sistema")
       public void verificarProducto() {
           assertNotNull(producto);
           assertEquals("Laptop", producto.getNombre());
       }
   }
   ```

3. **Ejecutar las pruebas**:
   Configura Cucumber en tu proyecto y ejecuta las pruebas con tu IDE o línea de comandos.

---

## 8. Conclusión

TDD y BDD son metodologías complementarias que ayudan a garantizar la calidad del software. Mientras que TDD se enfoca en pruebas técnicas, BDD permite alinear el desarrollo con los requisitos del negocio. Ambas prácticas son ideales para proyectos Java con Spring Boot.