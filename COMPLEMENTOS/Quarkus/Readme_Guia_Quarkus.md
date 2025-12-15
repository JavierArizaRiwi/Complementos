
# Guía Completa y Paso a Paso: CRUD en Quarkus  
**Incluye comandos, estructura, menús del IDE, y explicación detallada para tus coders**

---

# 0. PRE-REQUISITOS

### Verificar Java
```bash
java -version
```

### Verificar Maven
```bash
mvn -version
```

### Base de datos PostgreSQL
- DB: `quarkus_crud`
- user: `quarkus`
- pass: `quarkus`

---

# 1. ¿Qué es Quarkus?

Quarkus es un framework Java moderno optimizado para:

- Arranques ultrarrápidos (10–50 ms en nativo)  
- Bajo consumo de RAM (20–50 MB en nativo)  
- Kubernetes, Docker y nube  
- Microservicios reactivamente con Mutiny  
- Compilación nativa con GraalVM  

Su arquitectura se basa en:

### ✔ Build-Time Processing  
Quarkus mueve trabajo del runtime al build time para ser más rápido en producción.

### ✔ Extensiones  
Cada módulo (REST, Hibernate, Kafka, JWT, etc.) está optimizado por una extensión especializada.

---

# 2. Crear Proyecto Quarkus desde Cero

En la terminal:

```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.9.0:create \
  -DprojectGroupId=com.riwi.example \
  -DprojectArtifactId=quarkus-crud-usuarios \
  -DclassName="com.riwi.example.HelloResource" \
  -Dpath="/hello"
```

---

# 3. Abrir el Proyecto en IntelliJ IDEA

1. Abre IntelliJ  
2. **File → Open**  
3. Selecciona `quarkus-crud-usuarios/`  
4. Espera resolución de dependencias  

---

# 4. Instalar Extensiones Necesarias

En terminal dentro del proyecto:

```bash
./mvnw quarkus:add-extension \
  -Dextensions="resteasy-reactive-jackson,hibernate-orm-panache,jdbc-postgresql,smallrye-openapi"
```

Verificar en `pom.xml` que se hayan agregado.

---

# 5. Configurar la Base de Datos

Editar `src/main/resources/application.properties`:

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=quarkus
quarkus.datasource.password=quarkus
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/quarkus_crud

quarkus.hibernate-orm.database.generation=update
quarkus.hibernate-orm.log.sql=true

quarkus.smallrye-openapi.path=/q/openapi
quarkus.swagger-ui.always-include=true
```

---

# 6. Crear la Entidad `Usuario`

En IntelliJ:

- Clic derecho en `src/main/java` → **New → Package**  
- Nombre: `com.riwi.example.domain.model`  

Crear clase `Usuario.java`:

```java
package com.riwi.example.domain.model;

import io.quarkus.hibernate.orm.panache.PanacheEntity;
import jakarta.persistence.Column;
import jakarta.persistence.Entity;

@Entity
public class Usuario extends PanacheEntity {

    @Column(nullable = false)
    public String nombre;

    @Column(nullable = false, unique = true)
    public String email;

    @Column(nullable = false)
    public String rol;
}
```

---

# 7. Crear Repositorio Panache

- Package: `com.riwi.example.domain.repository`
- File: `UsuarioRepository.java`

```java
package com.riwi.example.domain.repository;

import com.riwi.example.domain.model.Usuario;
import io.quarkus.hibernate.orm.panache.PanacheRepository;
import jakarta.enterprise.context.ApplicationScoped;

import java.util.Optional;

@ApplicationScoped
public class UsuarioRepository implements PanacheRepository<Usuario> {

    public Optional<Usuario> findByEmail(String email) {
        return find("email", email).firstResultOptional();
    }
}
```

---

# 8. Crear DTOs

Package: `com.riwi.example.application.dto`

### UsuarioRequest.java
```java
package com.riwi.example.application.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

public class UsuarioRequest {

    @NotBlank
    public String nombre;

    @NotBlank
    @Email
    public String email;

    @NotBlank
    public String rol;
}
```

### UsuarioResponse.java
```java
package com.riwi.example.application.dto;

public class UsuarioResponse {
    public Long id;
    public String nombre;
    public String email;
    public String rol;
}
```

---

# 9. Crear Servicio `UsuarioService`

Package: `com.riwi.example.application.service`

```java
package com.riwi.example.application.service;

import com.riwi.example.application.dto.UsuarioRequest;
import com.riwi.example.application.dto.UsuarioResponse;
import com.riwi.example.domain.model.Usuario;
import com.riwi.example.domain.repository.UsuarioRepository;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.transaction.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@ApplicationScoped
public class UsuarioService {

    @Inject
    UsuarioRepository usuarioRepository;

    public List<UsuarioResponse> listar() {
        return usuarioRepository.listAll()
                .stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }

    public UsuarioResponse obtener(Long id) {
        Usuario usuario = usuarioRepository.findById(id);
        return usuario == null ? null : toResponse(usuario);
    }

    @Transactional
    public UsuarioResponse crear(UsuarioRequest request) {
        Usuario usuario = new Usuario();
        usuario.nombre = request.nombre;
        usuario.email = request.email;
        usuario.rol = request.rol;
        usuario.persist();
        return toResponse(usuario);
    }

    @Transactional
    public UsuarioResponse actualizar(Long id, UsuarioRequest request) {
        Usuario usuario = usuarioRepository.findById(id);
        if (usuario == null) return null;

        usuario.nombre = request.nombre;
        usuario.email = request.email;
        usuario.rol = request.rol;
        return toResponse(usuario);
    }

    @Transactional
    public boolean eliminar(Long id) {
        return usuarioRepository.deleteById(id);
    }

    private UsuarioResponse toResponse(Usuario usuario) {
        UsuarioResponse r = new UsuarioResponse();
        r.id = usuario.id;
        r.nombre = usuario.nombre;
        r.email = usuario.email;
        r.rol = usuario.rol;
        return r;
    }
}
```

---

# 10. Crear Controlador REST (Resource)

Package: `com.riwi.example.infrastructure.resource`

```java
package com.riwi.example.infrastructure.resource;

import com.riwi.example.application.dto.UsuarioRequest;
import com.riwi.example.application.dto.UsuarioResponse;
import com.riwi.example.application.service.UsuarioService;
import jakarta.inject.Inject;
import jakarta.validation.Valid;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;

import java.net.URI;
import java.util.List;

@Path("/usuarios")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class UsuarioResource {

    @Inject
    UsuarioService usuarioService;

    @GET
    public List<UsuarioResponse> listar() {
        return usuarioService.listar();
    }

    @GET
    @Path("/{id}")
    public Response obtener(@PathParam("id") Long id) {
        UsuarioResponse usuario = usuarioService.obtener(id);
        if (usuario == null) return Response.status(Response.Status.NOT_FOUND).build();
        return Response.ok(usuario).build();
    }

    @POST
    public Response crear(@Valid UsuarioRequest request) {
        UsuarioResponse creado = usuarioService.crear(request);
        return Response.created(URI.create("/usuarios/" + creado.id)).entity(creado).build();
    }

    @PUT
    @Path("/{id}")
    public Response actualizar(@PathParam("id") Long id, @Valid UsuarioRequest request) {
        UsuarioResponse actualizado = usuarioService.actualizar(id, request);
        if (actualizado == null) return Response.status(Response.Status.NOT_FOUND).build();
        return Response.ok(actualizado).build();
    }

    @DELETE
    @Path("/{id}")
    public Response eliminar(@PathParam("id") Long id) {
        boolean eliminado = usuarioService.eliminar(id);
        if (!eliminado) return Response.status(Response.Status.NOT_FOUND).build();
        return Response.noContent().build();
    }
}
```

---

# 11. Ejecutar en modo desarrollo

En IntelliJ, abre la terminal:

```bash
./mvnw quarkus:dev
```

Abrir Dev UI:  
👉 `http://localhost:8080/q/dev/`

---

# 12. Probar el CRUD

## Crear usuario
```bash
curl -X POST http://localhost:8080/usuarios \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Javier","email":"javier@example.com","rol":"ADMIN"}'
```

## Listar usuarios
```bash
curl http://localhost:8080/usuarios
```

## Obtener por ID
```bash
curl http://localhost:8080/usuarios/1
```

## Actualizar
```bash
curl -X PUT http://localhost:8080/usuarios/1 \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Javier A","email":"javier.a@example.com","rol":"USER"}'
```

## Eliminar
```bash
curl -X DELETE http://localhost:8080/usuarios/1
```

---

# 13. Swagger UI

- OpenAPI JSON:  
  `http://localhost:8080/q/openapi`

- Swagger UI:  
  `http://localhost:8080/q/swagger-ui`

---

# 14. Dockerizar Proyecto (opcional)

```dockerfile
FROM quay.io/quarkus/quarkus-micro-image:1.0
COPY target/*-runner.jar /application.jar
EXPOSE 8080
CMD ["java", "-jar", "/application.jar"]
```

Build:

```bash
mvn package -Dquarkus.package.type=uber-jar
docker build -t quarkus-crud .
```

---

# 15. Resumen Final

Ya implementaste un CRUD completo usando:

- Quarkus  
- Panache ORM  
- RESTEasy Reactive  
- PostgreSQL  
- DTOs  
- Servicio + Repositorio + Resource  
- Validaciones  
- Swagger UI  

Este README está listo para usar en proyectos, talleres y clases.

---

Fin de la guía 