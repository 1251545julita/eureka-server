# Eureka Server — Servidor de descubrimiento de servicios

**Diplomatura en Desarrollo de Software FinTech: IA y Microservicios — UTN / Codeki**
**Student: Julita, Jesica R.**

---

## 1. Qué es y para qué sirve

Servidor de registro y descubrimiento de servicios (Service Registry) basado en **Spring Cloud Netflix Eureka**.
Cada microservicio (por ejemplo **customer-service** y **product-service**) se registra acá al iniciar, y consulta
este servidor para encontrar la ubicación de los demás, sin depender de direcciones fijas.

La dirección de este servidor se distribuye a los microservicios a través del **config-server**: el archivo
`application.yml` de **config-repo** (configuración general, común a todos) define `eureka.client.service-url.defaultZone`,
así ningún microservicio necesita declararla por su cuenta.

---

## 2. Qué se necesita antes de empezar

- **JDK 21** instalado.
- **IntelliJ IDEA** (Community alcanza).
- Este servidor **no necesita base de datos**: el registro se mantiene en memoria.
- **config-server** y **config-repo** son necesarios para los microservicios cliente (ver punto 7), no para levantar este servidor.

Versiones utilizadas:

| Componente | Versión |
|------------|---------|
| Spring Boot | 4.1.1 |
| Spring Cloud | 2025.1.3 |
| Java | 21 |

---

## 3. Cómo abrirlo en IntelliJ IDEA

1. Luego de clonar el repositorio `https://github.com/1251545julita/eureka-server` abrir IntelliJ.
2. **File > Open** → elegir la carpeta `eureka-server`, la que contiene el `pom.xml`.
   No abrir el `pom.xml` directamente.
3. Cuando pregunte, aceptar **Trust Project**.
4. Verificar el JDK en `File > Project Structure > Project > SDK`: tiene que apuntar a un **JDK 21**. Si no aparece ninguno, elegir `Add SDK > Download JDK` y descargar una distribución 21.
5. Si no ve el panel de Maven: `View > Tool Windows > Maven`, y tocar el botón de recargar.
6. En `src/main/resources/application.yml` verificar el puerto del servidor (`server.port: 8761`).

**Si el `pom.xml` aparece en gris o no hay carpeta `External Libraries`:** clic derecho
sobre `pom.xml` → **Add as Maven Project**.

---

## 4. Configuración

### En este proyecto: `application.yml`

Eureka Server tiene su propia configuración local, que **no** viene del config-server:

```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8761

eureka:
  client:
    register-with-eureka: false   # el servidor no se registra a sí mismo
    fetch-registry: false         # ni descarga el registro
```

### En config-repo: configuración general

La dirección de Eureka la define el archivo general de config-repo (`application.yml`), que el config-server entrega
a **todos** los microservicios:

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

Además, config-repo tiene un archivo `.yml` por cada microservicio (por ejemplo `product-service.yml`,
`customer-service.yml`) con su configuración específica. Eureka Server no tiene archivo propio ahí.

---

## 5. Cómo ejecutarlo

> **Orden:** primero **config-server** (puerto 8888) y **eureka-server** (puerto 8761), después
> **product-service** (puerto 8082) y por último **customer-service** (puerto 8081).
> Los microservicios necesitan que los dos primeros estén activos.

### Desde IntelliJ

Abrí `src/main/java/ar/edu/utn/diplomatura/eurekaserver/EurekaServerApplication.java` y hacé clic en
la flecha verde ▶ que está al lado de `public static void main`.
La salida aparece en la consola de abajo.

### Desde la terminal, con Maven

```
mvn clean compile        # Compilar
mvn test                 # Ejecutar los tests
mvn spring-boot:run      # Ejecutar la aplicación
mvn clean package        # Generar el paquete ejecutable
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar
```

### Desde la terminal, utilizando Maven Wrapper

El Maven Wrapper permite ejecutar Maven sin necesidad de tenerlo instalado globalmente.
En Linux, macOS o Git Bash:

```bash
./mvnw clean compile
./mvnw test
./mvnw spring-boot:run
./mvnw clean package
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar
```

En PowerShell (Windows):

```powershell
.\mvnw.cmd clean compile
.\mvnw.cmd test
.\mvnw.cmd spring-boot:run
.\mvnw.cmd clean package
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar
```

---

## 6. Cómo verificar que funciona

Con la aplicación levantada, abrí en el navegador el panel de Eureka:

http://localhost:8761

Tenés que ver la pantalla **Spring Eureka** con la sección **Instances currently registered with Eureka**.
Al principio va a estar vacía; a medida que levantes los microservicios van a ir apareciendo en la lista.

También se puede consultar el registro en formato JSON, desde **Git Bash**:

```bash
curl -i -H "Accept: application/json" http://localhost:8761/eureka/apps
```

> **Qué significa el `-i`:** muestra también los encabezados de la respuesta, incluido el código de estado.

Para comprobar que el config-server entrega la dirección de Eureka a los microservicios:

```bash
curl -i http://localhost:8888/product-service/default
```

En la respuesta tiene que aparecer `eureka.client.service-url.defaultZone` junto con las propiedades propias de product-service.

---

## 7. Cómo se registra un microservicio

Cada microservicio cliente necesita la dependencia `spring-cloud-starter-netflix-eureka-client` y apuntar al
config-server en su `application.yml`:

```yaml
spring:
  application:
    name: customer-service     # nombre con el que aparece en Eureka
  config:
    import: "optional:configserver:http://localhost:8888"
```

El `defaultZone` **no** se declara en el microservicio: llega desde el `application.yml` general de config-repo.

Al iniciar, el servicio aparece en el panel con su nombre en mayúsculas (por ejemplo `CUSTOMER-SERVICE`).

---

## 8. Estructura del proyecto

```
eureka-server/
├── pom.xml
├── README.md
├── src/main/resources/application.yml       # Puerto y configuración propia del servidor Eureka
├── src/test/*                               # Tests de arranque de la aplicación
└── src/main/java/ar/edu/utn/diplomatura/eurekaserver/
    └── EurekaServerApplication.java         # Clase principal, anotada con @EnableEurekaServer
```

---

## 9. Arquitectura

```
config-repo (git)
  ├── application.yml        # general: defaultZone de Eureka
  ├── product-service.yml
  └── customer-service.yml
        │
        ▼
config-server (:8888) ── entrega configuración ──┐
                                                  ▼
                        product-service (:8082)   customer-service (:8081)
                                  │                       │
                                  └─ se registran y consultan ─┐
                                                               ▼
                                                   eureka-server (:8761)
```

- **config-repo** guarda la configuración general y la de cada microservicio.
- **config-server** la sirve a los microservicios, incluida la URL de Eureka.
- **eureka-server** mantiene el registro de qué servicios están activos y dónde.

---

## 10. Problemas frecuentes

**`error: invalid source release: 21`.** El JDK configurado es anterior a 21. Revisá el punto 3.

**El botón ▶ no aparece al lado del `main`.** IntelliJ todavía no terminó de indexar, o no reconoció la carpeta como proyecto Maven. Esperá a que termine la barra de progreso de abajo, o recargá el proyecto desde el panel de Maven.

**`Port 8761 was already in use`.** Hay otra aplicación usando ese puerto. Cerrala o cambiá `server.port` en `application.yml`. Si lo cambiás, actualizá también el `defaultZone` en config-repo.

**En los logs aparece `Cannot execute request on any known server` o `Connection refused`.** Eureka Server intenta registrarse a sí mismo como cliente. Verificá que `eureka.client.register-with-eureka` y `eureka.client.fetch-registry` estén en `false` en el `application.yml` de este proyecto.

**Un microservicio no aparece en el panel.** Revisá que eureka-server esté levantado **antes**, que el config-server entregue el `defaultZone` (punto 6) y que el servicio tenga `spring.application.name`. Recordá que config-server lee de git lo que está commiteado: si cambiaste `application.yml` en config-repo, hacé commit y reiniciá el microservicio.

**El microservicio falla con `Could not locate PropertySource`.** El config-server no está levantado o respondió con error. Probá `curl -i http://localhost:8888/product-service/default` y, si da 500, revisá la consola del config-server.

**Los acentos se ven como `?` o `�` en la consola.** Pasa en `cmd` y en PowerShell por el codepage de Windows. Corré `chcp 65001` antes de ejecutar, o usá la consola de IntelliJ.
