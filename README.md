# 🧩 SGIVU - Servicio de Configuraciones

## 📘 Descripción

Servicio Spring Cloud Config que centraliza y expone la configuración del ecosistema SGIVU. Provee archivos de
configuración versionados en Git para que los microservicios consuman parámetros consistentes en cada ambiente.

## 🧱 Arquitectura y Rol

* Tipo: Microservicio Spring Boot 3 / Spring Cloud Config Server.
* Interactúa con: `sgivu-config-repo` (repositorio Git), `sgivu-discovery`, `sgivu-gateway` y el resto de microservicios
  que consumen configuración.
* Expone endpoints REST para entregar propiedades externas a los servicios cliente.
* No registra sus instancias en Eureka por defecto; puede integrarse si se habilita el cliente Discovery.
* Obtiene configuración desde el repositorio de configuraciones respaldado por Git (utilizando `main` como rama activa).
* No persiste datos propios; sólo lee archivos del repositorio remoto.

## ⚙️ Tecnologías

* **Lenguaje:** Java 21
* **Framework:** Spring Boot 3.5.x, Spring Cloud 2025.0.0 (Leyden)
* **Seguridad:** Spring Boot Actuator (sin autenticación habilitada por defecto)
* **Persistencia:** N/A (datos externos en Git)
* **Infraestructura:** Docker, AWS (EC2)

## 🚀 Ejecución Local

1. Clonar el repositorio.
2. Configurar `src/main/resources/application.yml` o variables de entorno:
    * `SPRING_PROFILES_ACTIVE` (`git` por defecto).
    * `SPRING_CLOUD_CONFIG_SERVER_GIT_URI` si se utiliza un repositorio distinto.

3. Ejecutar con:

   ```bash
   ./mvnw spring-boot:run
   ```

4. Acceder a `http://localhost:8888` y consumir los endpoints del Config Server.

## 🔗 Endpoints Principales

```
GET /{application}/{profile}
GET /{application}/{profile}/{label}
GET /actuator/health
```

* `/{application}/{profile}`: entrega propiedades para la aplicación y perfil solicitados.
* `/{application}/{profile}/{label}`: permite consumir configuración apuntando a una etiqueta o rama específica.
* `/actuator/health`: expone el estado básico del servidor de configuración.

## 🔐 Seguridad

Actualmente el Config Server está expuesto sin autenticación. Se recomienda integrarlo con `sgivu-auth` (OAuth2 + JWT)
mediante un proxy inverso (`sgivu-gateway`) o habilitando seguridad en Spring Security para asegurar que sólo servicios
autorizados obtengan configuración.

## 🧩 Dependencias

* `sgivu-config-repo` (repositorio Git con la configuración centralizada).
* `sgivu-discovery` (opcional si se registra para descubrimiento de servicios).
* `sgivu-gateway` (proxy que puede proteger y enrutar el acceso al Config Server).
* Microservicios SGIVU que consumen configuración externa.

## 🧮 Dockerización

* Contenedor: `sgivu-config`
* Puerto expuesto: `8888`
* Ejemplo de ejecución:

  ```bash
  docker build -t sgivu-config .
  docker run -p 8888:8888 \
    -e SPRING_CLOUD_CONFIG_SERVER_GIT_URI=https://github.com/stevenrq/sgivu-config-repo.git \
    sgivu-config
  ```

## ☁️ Despliegue en AWS

1. Provisionar una instancia EC2 (Amazon Linux 2023 o Ubuntu) con acceso al repositorio Git.
2. Instalar Java 21 / Docker y desplegar el artefacto (JAR o contenedor).
3. Configurar variables de entorno (`SPRING_CLOUD_CONFIG_SERVER_GIT_URI`, credenciales de Git, `SERVER_PORT` si cambia).

## 📊 Monitoreo

* Spring Boot Actuator proporciona endpoints de salud y métricas básicas (`/actuator/*`).
* Puede integrarse con Micrometer para exportar métricas a Prometheus.
* El rastreo distribuido se puede habilitar conectando con Zipkin a través de Spring Cloud Sleuth en los servicios
  cliente.

## ✨ Autor

* **Steven Ricardo Quiñones**
