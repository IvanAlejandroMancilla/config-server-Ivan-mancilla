# config-server

Servidor de configuración centralizada construido con **Spring Boot 3** y **Java 21**, basado en **Spring Cloud Config Server**. Es el primer componente que debe iniciarse en el ecosistema: expone la configuración (`application.yml`, `customer-service.yml`, `product-service.yml`, etc.) que los demás microservicios (`eureka-server`, `customer-service`, `product-service`) leen al arrancar.

## ✨ ¿Qué hace?

En lugar de que cada microservicio guarde su propio `application.yml` con datos como el puerto, la URL de Eureka o parámetros de negocio, todos esos archivos se guardan en un **repositorio Git remoto** separado. El `config-server`:

1. Al arrancar, se conecta a ese repositorio Git remoto.
2. Expone esa configuración vía HTTP (por ejemplo `GET /product-service/default`).
3. Los demás servicios, al iniciar, le piden su configuración a este servidor en lugar de leerla de un archivo local.

Esto permite cambiar la configuración de todo el ecosistema (puertos, URLs, feature flags, etc.) desde un único repositorio, sin tener que recompilar ni redistribuir cada microservicio.

## 🧱 Stack técnico

| Tecnología | Uso |
|---|---|
| Java 21 | Lenguaje |
| Spring Boot 3.3.2 | Framework base |
| Spring Cloud Config Server | Servidor de configuración centralizada |
| Spring Web (MVC) | Expone los endpoints HTTP del servidor de config |
| Maven | Gestión de dependencias y build |

## 📁 Estructura del proyecto

```
src/main/java/com/ivanmancilla/configserver
└── ConfigServerApplication.java   # Clase principal (@SpringBootApplication, @EnableConfigServer)

src/main/resources
└── application.yml                # Puerto propio y URL del repo Git de configuración
```

La anotación `@EnableConfigServer` es la que activa toda la funcionalidad de Spring Cloud Config: no hay controllers ni lógica propia, el comportamiento lo aporta la dependencia `spring-cloud-config-server`.

## ⚙️ Configuración (`application.yml`)

```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/IvanAlejandroMancilla/tp-config-repo-IVANMANCILLA
          default-label: main
```

- **`server.port`**: puerto en el que corre el servidor (`8888`).
- **`spring.cloud.config.server.git.uri`**: repositorio Git remoto donde viven los archivos de configuración de cada microservicio.
- **`default-label`**: rama del repositorio que se usa por defecto (`main`).

## ▶️ Cómo ejecutarlo

Debe ser el **primer** servicio en levantarse, ya que el resto depende de él para obtener su configuración.

```bash
cd config-server
./mvnw spring-boot:run
```

Esperá a que el log confirme la conexión exitosa con el repositorio Git antes de iniciar los demás servicios.

## 🔍 Cómo probarlo

Con el servidor corriendo, se puede consultar la configuración de cualquier microservicio directamente por HTTP, siguiendo el patrón `/{nombre-de-la-app}/{profile}`:

```bash
curl http://localhost:8888/product-service/default
curl http://localhost:8888/customer-service/default
curl http://localhost:8888/eureka-server/default
```

Si la respuesta trae el contenido del `.yml` correspondiente en formato JSON, el config-server está funcionando correctamente.

## 🔗 Relación con el resto del ecosistema

Ver el diagrama y la guía de arranque completa en el [README principal](../README.md) del repositorio.
