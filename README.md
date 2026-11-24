# Eureka Server — Servidor de descubrimiento de servicios

**Diplomatura en Desarrollo de Software FinTech: IA y Microservicios — UTN / Codeki**
**Student: Julita, Jesica R.**

---

## 1. Qué es y para qué sirve

Servidor de registro y descubrimiento de servicios (Service Registry) basado en **Spring Cloud Netflix Eureka**.
Cada microservicio (por ejemplo **customer-service** y **product-service**) se registra acá al iniciar, y consulta
este servidor para encontrar la ubicación de los demás, sin depender de direcciones fijas.

---

## 2. Qué se necesita antes de empezar

- **JDK 21** instalado.
- **IntelliJ IDEA** (Community alcanza).
- Este servidor **no necesita base de datos**: el registro se mantiene en memoria.

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
6. En `application.yml` (o `application.properties`) verificar el puerto del servidor (`server.port: 8761`).

**Si el `pom.xml` aparece en gris o no hay carpeta `External Libraries`:** clic derecho
sobre `pom.xml` → **Add as Maven Project**.

---

## 4. Cómo ejecutarlo

> **Orden:** primero levantar **eureka-server** (puerto 8761), después **product-service** (puerto 8082)
> y por último **customer-service** (puerto 8081).

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

## 5. Cómo verificar que funciona

Con la aplicación levantada, abrí en el navegador el panel de Eureka:

http://localhost:8761

Tenés que ver la pantalla **Spring Eureka** con la sección **Instances currently registered with Eureka**.
Al principio va a estar vacía; a medida que levantes los microservicios van a ir apareciendo en la lista.

También se puede consultar el registro en formato JSON, desde **Git Bash**:

```bash
curl -i -H "Accept: application/json" http://localhost:8761/eureka/apps
```

> **Qué significa el `-i`:** muestra también los encabezados de la respuesta, incluido el código de estado.

---

## 6. Cómo registrar un microservicio

Cada microservicio cliente necesita la dependencia `spring-cloud-starter-netflix-eureka-client`
y esta configuración en su `application.yml`:

```yaml
spring:
  application:
    name: customer-service     # nombre con el que aparece en Eureka

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

Al iniciar, el servicio aparece en el panel con su nombre en mayúsculas (por ejemplo `CUSTOMER-SERVICE`).

---

## 7. Estructura del proyecto

```
eureka-server/
├── pom.xml
├── README.md
├── src/main/resources/application.yml       # Puerto y configuración del servidor Eureka
├── src/test/*                               # Tests de arranque de la aplicación
└── src/main/java/ar/edu/utn/diplomatura/eurekaserver/
    └── EurekaServerApplication.java         # Clase principal, anotada con @EnableEurekaServer
```

---

## 8. Arquitectura

```
                      ┌──────────────────────────┐
                      │  eureka-server (:8761)   │
                      │  registro de servicios   │
                      └───────▲──────────▲───────┘
                  registra y  │          │  registra y
                  consulta    │          │  consulta
          ┌───────────────────┴──┐   ┌───┴───────────────────┐
          │ customer-service     │──>│ product-service       │
          │ (:8081 /clientes)    │   │ (:8082 /cuentas)      │
          └──────────────────────┘   └───────────────────────┘
```

---

## 9. Problemas frecuentes

**`error: invalid source release: 21`.** El JDK configurado es anterior a 21. Revisá el punto 3.

**El botón ▶ no aparece al lado del `main`.** IntelliJ todavía no terminó de indexar, o no reconoció la carpeta como proyecto Maven. Esperá a que termine la barra de progreso de abajo, o recargá el proyecto desde el panel de Maven.

**`Port 8761 was already in use`.** Hay otra aplicación usando ese puerto. Cerrala o cambiá `server.port` en `application.yml`.

**En los logs aparece `Cannot execute request on any known server` o `Connection refused`.** Por defecto, Eureka Server intenta registrarse a sí mismo como cliente. En `application.yml` agregá `eureka.client.register-with-eureka: false` y `eureka.client.fetch-registry: false`.

**Un microservicio no aparece en el panel.** Revisá que eureka-server esté levantado **antes**, que el `defaultZone` apunte a `http://localhost:8761/eureka/` y que el servicio tenga `spring.application.name`.

**Los acentos se ven como `?` o `�` en la consola.** Pasa en `cmd` y en PowerShell por el codepage de Windows. Corré `chcp 65001` antes de ejecutar, o usá la consola de IntelliJ.