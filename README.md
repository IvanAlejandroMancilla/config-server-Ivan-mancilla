# customer-service

Microservicio de gestión de clientes construido con **Spring Boot 3** y **Java 21**. Forma parte de una arquitectura de microservicios: se registra en **Eureka**, obtiene su configuración desde un **Config Server** y se comunica con `product-service` mediante **OpenFeign** para enriquecer la información de cada cliente con sus productos asociados.

## ✨ Funcionalidades

- CRUD completo de clientes (crear, listar, obtener por ID, actualizar, eliminar).
- Validación de documento único: evita registrar clientes duplicados.
- Consulta de un cliente junto con sus productos, obtenidos en tiempo real desde `product-service`.
- Manejo centralizado de errores con respuestas JSON consistentes (`404`, `409`, `503`, `500`).
- Datos de prueba precargados al iniciar la aplicación.

## 🧱 Stack técnico

| Tecnología | Uso |
|---|---|
| Java 21 | Lenguaje |
| Spring Boot 3.3.2 | Framework base |
| Spring Web (MVC) | API REST |
| Spring Data JPA | Persistencia |
| H2 Database | Base de datos en memoria |
| Spring Cloud Netflix Eureka Client | Descubrimiento de servicios |
| Spring Cloud Config | Configuración centralizada |
| Spring Cloud OpenFeign | Comunicación con `product-service` |
| Maven | Gestión de dependencias y build |

## 📁 Estructura del proyecto

```
src/main/java/com/ivanmancilla/customerservice
├── CustomerServiceApplication.java   # Clase principal (@SpringBootApplication, @EnableFeignClients)
├── client/
│   └── ProductClient.java            # Cliente Feign hacia product-service
├── config/
│   └── CustomerDataInitializer.java  # Carga de datos iniciales (clientes de prueba)
├── controller/
│   └── CustomerController.java       # Endpoints REST (/clientes)
├── dto/
│   ├── CustomerRequestDTO.java
│   ├── CustomerResponseDTO.java
│   └── ProductDTO.java
├── entity/
│   └── Customer.java                 # Entidad JPA (tabla customers)
├── exception/
│   ├── CustomerNotFoundException.java
│   ├── DuplicateCustomerException.java
│   ├── ErrorResponse.java
│   └── GlobalExceptionHandler.java   # Manejo global de excepciones
├── mapper/
│   ├── CustomerMapper.java
│   └── GenericMapper.java
├── repository/
│   └── CustomerRepository.java
└── service/
    └── CustomerService.java
```

## ⚙️ Requisitos previos

- JDK 21+
- Maven 3.9+
- (Recomendado, para funcionamiento completo del ecosistema)
  - Un **Config Server** corriendo en `http://localhost:8888`
  - Un **Eureka Server** para el registro del servicio
  - El microservicio `product-service` registrado en Eureka, para que funcione el endpoint de productos por cliente

> El servicio puede levantarse sin el Config Server, Eureka o `product-service`, pero fallará al iniciar si no puede resolver `configserver:http://localhost:8888`, y el endpoint `/clientes/{id}/productos` devolverá la lista de productos vacía si no logra comunicarse con `product-service`.

## 🚀 Cómo ejecutar

```bash
# Clonar el repositorio
git clone https://github.com/IvanAlejandroMancilla/customer-service-Ivan-mancilla.git
cd customer-service-Ivan-mancilla

# Compilar y ejecutar
mvn spring-boot:run
```

También puede empaquetarse y ejecutarse como JAR:

```bash
mvn clean package
java -jar target/customer-service-0.0.1-SNAPSHOT.jar
```

La configuración de la aplicación (`application.yml`) importa propiedades adicionales desde el Config Server:

```yaml
spring:
  application:
    name: customer-service
  config:
    import: "configserver:http://localhost:8888"
```

## 📡 Endpoints

Base path: `/clientes`

| Método | Endpoint | Descripción |
|---|---|---|
| `POST` | `/clientes` | Crea un nuevo cliente |
| `GET` | `/clientes` | Lista todos los clientes |
| `GET` | `/clientes/{id}` | Obtiene un cliente por ID |
| `GET` | `/clientes/{id}/productos` | Obtiene un cliente junto a sus productos (vía `product-service`) |
| `PUT` | `/clientes/{id}` | Actualiza un cliente existente |
| `DELETE` | `/clientes/{id}` | Elimina un cliente |

### Ejemplo — Crear cliente

```bash
curl -X POST http://localhost:8081/clientes \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Ivan Mancilla",
    "documento": "12345678",
    "email": "ivan@example.com",
    "saldo": 500000.00
  }'
```

### Ejemplo — Obtener cliente con productos

```bash
curl http://localhost:8081/clientes/1/productos
```

## 🧩 Modelo de datos

**Customer (entidad, tabla `customers`)**

| Campo | Tipo | Restricciones |
|---|---|---|
| `id` | Long | PK, autogenerado |
| `nombre` | String | Requerido |
| `documento` | String | Requerido, único |
| `email` | String | Requerido |
| `saldo` | BigDecimal | — |

**ProductDTO** (recibido desde `product-service` vía Feign, se embebe en `CustomerResponseDTO.productos`)

| Campo | Tipo |
|---|---|
| `id` | Long |
| `clienteId` | Long |
| `tipo` | String |
| `nombre` | String |
| `descripcion` | String |
| `montoAsociado` | BigDecimal |
| `tasaInteres` | BigDecimal |
| `activo` | boolean |
| `fechaInicio` | LocalDate |
| `fechaVencimiento` | LocalDate |

## ⚠️ Manejo de errores

El `GlobalExceptionHandler` centraliza las respuestas de error con un formato consistente (`timestamp`, `status`, `error`, `message`, `path`):

| Excepción | Código HTTP |
|---|---|
| `CustomerNotFoundException` | 404 Not Found |
| `DuplicateCustomerException` | 409 Conflict |
| `FeignException` (fallo al comunicarse con `product-service`) | 503 Service Unavailable |
| Excepción genérica | 500 Internal Server Error |

## 👤 Autor

**Ivan Alejandro Mancilla**
