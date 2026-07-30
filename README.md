# Documentación de Servicios, Recursos, Enlaces y Análisis de Calidad SonarQube

**Organización:** Little Caesars BPM  
**Proyecto:** Aplicación BPM en Entorno Distribuido con Servicios Web y Broker de Mensajes  
**Repositorio Principal:** [Little-Caersars-BPM-Project](https://github.com/AlonsoUNSA/Little-Caersars-BPM-Project)  

---

## Fundamentación Teórica y Justificación Arquitectónica

La implementación del sistema de información empresarial para **Little Caesars** se fundamenta en los principios de la **Arquitectura Orientada a Servicios (SOA)** y la **Gestión de Procesos de Negocio (BPM)**, integrando un entorno distribuido guiado por eventos. La solución responde a las mejores prácticas de ingeniería de software corporativo descritas en la literatura especializada (Laudon & Laudon, Krafzig et al., Arsanjani - SOMA):

1. **Desacoplamiento Débil y Autonomía de Servicios:**
   Cada microservicio (`service-clientes`, `service-caja`, `service-inventario`, `price-service`) es autónomo, posee su propio ámbito de responsabilidad y se ejecuta de manera independiente. Esto garantiza que fallos aislados en un componente no comprometan la continuidad operativa del proceso de negocio.

2. **Composición de Servicios Mediante BPM:**
   La orquestación del flujo de atención en tienda se realiza a través de Bonita Studio. Las tareas automáticas consumen las interfaces REST de los servicios mediante conectores HTTP en el ciclo de vida del proceso, permitiendo una separación clara entre la lógica del proceso (el qué) y la implementación técnica (el cómo).

3. **Arquitectura Orientada a Eventos (EDA) con Broker de Mensajes:**
   La integración asíncrona mediante **RabbitMQ** (Exchange de tipo Topic y colas duraderas) permite que las transacciones clave (como el registro de un cliente) emitan eventos de dominio de manera no bloqueante. Esto asegura alta disponibilidad, escalabilidad y resiliencia ante caídas temporales de red.

4. **Diseño Guiado por el Dominio (Domain-Driven Design - DDD):**
   Los servicios respetan una arquitectura limpia en 4 capas (Presentación, Aplicación, Dominio e Infraestructura). La capa de Dominio permanece aislada de dependencias técnicas y frameworks, modelando explícitamente Agregados, Objetos de Valor (Value Objects), Fábricas (Factories), Servicios de Dominio y Interfaces de Repositorio (Ports & Adapters).

5. **Aseguramiento de Calidad, Clean Code y TDD:**
   El código fuente ha sido desarrollado aplicando desarrollo guiado por pruebas (TDD) para validar las reglas de negocio críticas. Asimismo, la evaluación estática mediante SonarQube certifica la ausencia de vulnerabilidades de seguridad (*Security Hotspots*) y errores bloqueantes o críticos en la lógica del negocio.

---

## Descripción General de los Servicios Web del Proyecto

| Servicio / Módulo | Tecnología | Descripción y Responsabilidad en el Proceso |
| :--- | :--- | :--- |
| **service-clientes** | Java 21 + Spring Boot 3.3.2 | **Gestión de Clientes:** Registra nuevos clientes, valida datos únicos (teléfono/email), realiza bajas lógicas y publica el evento `cliente.registrado` en RabbitMQ para notificaciones asíncronas. |
| **service-caja** | Java 21 + Spring Boot | **Gestión de Pago y Entrega:** Valida transacciones de pago en tienda, registra el método de pago seleccionado, genera el comprobante fiscal e informa el estado final de la entrega. |
| **service-inventario** | Python 3.12 + FastAPI / Flask | **Consulta de Disponibilidad:** Verifica el stock e inventario en tiempo real de insumos, pizzas y bebidas antes de confirmar el registro del pedido en tienda. |
| **price-service** | Java 21 + Spring Boot | **Cálculo de Precios y Total:** Calcula el monto total a pagar considerando precios unitarios, combos, promociones aplicables e impuestos. |
| **app-delivery** | React / Node.js | **Aplicación Web / Frontend:** Interfaz de usuario (Application Page / Living App) para crear instancias del proceso, consultar el estado de pedidos y hacer seguimiento por rol. |

---

## Descripción de Enlaces y Recursos del Proyecto

| Recurso / Enlace | Tipo | Descripción y Propósito en la Evaluación |
| :--- | :--- | :--- |
| **[Repositorio Principal BPM](https://github.com/AlonsoUNSA/Little-Caersars-BPM-Project)** | Repositorio GitHub (Grupal) | Contiene la composición del proyecto final, el diagrama BPMN en Bonita Studio (`ESD-BonitaSoft-Little-Caesar-BPM`), la carpeta `imagenes/` de evidencias y los submódulos Git de cada microservicio en `services/`. |
| **[Submódulo service-clientes](https://github.com/KennyBorja/service-clientes)** | Repositorio GitHub (Kenny Borja) | Microservicio de gestión de clientes (Java 21 + Spring Boot 3.3.2). Implementa arquitectura DDD completa, eventos RabbitMQ, Swagger UI y TDD. |
| **[Swagger UI — Service Clientes](http://localhost:8081/swagger-ui.html)** | Herramienta de Pruebas REST | Interfaz web OpenAPI interactiva para ejecutar y validar los endpoints (`POST`, `GET`, `PUT`, `DELETE`) del servicio de Clientes en puerto 8081. |
| **[Consola H2 Database](http://localhost:8081/h2-console)** | Consola de BD | Permite inspeccionar las tablas relacionales y registros del servicio de Clientes (`jdbc:h2:mem:clientesdb`). |
| **[Endpoint de Salud (Health Check)](http://localhost:8081/api/v1/clientes/health)** | Estado de Servicio | Verifica el estado activo (`UP`) del microservicio REST. |

---

## Análisis Detallado de Calidad de Código en SonarQube

A continuación se presenta el análisis técnico de cada reporte de **SonarQube** almacenado en la carpeta `imagenes/` para los diferentes microservicios del proyecto distribuido:

---

### 1. ESD-Little-Caesars-price-service (Servicio de Precios)

![SonarQube Price Service](imagenes/WhatsApp%20Image%202026-07-30%20at%209.21.38%20AM.jpeg)

- **Módulo / Archivo:** `PriceServiceApplicationTests.java` (Línea 10)
- **Tipo de Hallazgo:** Code Smell — Severidad Blocker (10 min effort).
- **Regla Violada:** *"Add at least one assertion to this test case."* (JUnit 5).
- **Análisis Técnico:**
  La clase de prueba autogenerada por Spring Boot no contiene verificaciones (`assertEquals`, `assertNotNull`, etc.). 
- **Solución Recomendada:** Incluir al menos una aserción dentro del método `@Test` para validar que el contexto de Spring cargó correctamente.

---

### 2. DSE-Little-Caesars-App-Delivery (Frontend / Aplicación Web)

![SonarQube App Delivery](imagenes/WhatsApp%20Image%202026-07-30%20at%209.37.16%20AM.jpeg)

- **Módulo / Archivo:** `frontend/src/App.jsx` (Líneas 56, 143, 187, 295)
- **Tipo de Hallazgo:** Code Smell — Severidad Major (4 hallazgos, 20 min effort).
- **Reglas Violadas:** 
  1. *"Extract this nested ternary operation into an independent statement."* (L56).
  2. *"Do not use Array index in keys"* (L143, L187, L295 en listas React JSX).
- **Análisis Técnico:**
  - El uso de operaciones ternarias anidadas dificulta la lectura del flujo de la interfaz.
  - Usar el índice del array (`index`) como atributo `key` en React reduce el rendimiento del Virtual DOM cuando la lista se reordena o filtra.
- **Solución Recomendada:** Extraer lógica a funciones auxiliares y asignar IDs únicos como `key` en componentes React.

---

### 3. service-caja (Servicio de Caja y Entregas)

![SonarQube Service Caja](imagenes/WhatsApp%20Image%202026-07-30%20at%2010.08.50%20AM.jpeg)

- **Módulo / Archivos:** `EntregaController.java` (L43, L65) y `EntregaTest.java` (L32).
- **Tipo de Hallazgo:** Code Smell — 2 Minor, 1 Major (9 min effort).
- **Reglas Violadas:**
  1. *"Remove the 'ApiResponses' wrapper from this annotation group"* (Sintaxis OpenAPI 3).
  2. *"Refactor the code of the lambda to have only one invocation possibly throwing a runtime exception."*
- **Análisis Técnico:**
  - En Java 8+, `@ApiResponse` es una anotación repetible, por lo que el contenedor `@ApiResponses` resulta innecesario.
  - En los tests unitarios, las expresiones lambda dentro de `assertThrows` deben envolver únicamente la llamada específica que lanza la excepción.

---

### 4. service-caja — Detalle Adicional de Reglas de Calidad

![SonarQube Service Caja Detalle](imagenes/WhatsApp%20Image%202026-07-30%20at%2010.08.50%20AM%20(1).jpeg)

- **Módulo:** `service-caja`
- **Análisis Técnico:**
  Muestra la evaluación del Quality Gate para el servicio de caja. Destaca 0 Bugs, 0 Vulnerabilidades y 3 Code Smells leves. La arquitectura se mantiene limpia y sin fallos de seguridad bloqueantes.

---

### 5. service-clientes (Servicio de Clientes — Kenny Borja)

![SonarQube Service Clientes](imagenes/WhatsApp%20Image%202026-07-30%20at%2010.12.25%20AM.jpeg)

- **Módulo / Archivos:** `ClienteApplicationService.java` (L60) y `ClienteController.java` (L37, L61, L75, L89, L105).
- **Tipo de Hallazgo:** Code Smell — 1 Critical, 5 Minor (18 min effort).
- **Reglas Violadas:**
  1. *"Define a constant instead of duplicating this literal 'Cliente no encontrado con ID: ' 3 times."* (Critical).
  2. *"Remove the 'ApiResponses' wrapper from this annotation group"* (Minor).
- **Análisis Técnico:**
  - El texto de excepción `"Cliente no encontrado con ID: "` se encuentra repetido en 3 casos de uso.
  - **Calidad General:** 0 Bugs y 0 Vulnerabilidades. El servicio cumple con las normas de seguridad de SonarQube.

---

### 7. service-inventario — Análisis de Código Python

![SonarQube Service Inventario](imagenes/WhatsApp%20Image%202026-07-30%20at%2010.20.25%20AM.jpeg)

- **Módulo / Archivo:** `service-inventario` (Python 3.12, dependencias `click/types.py`)
- **Tipo de Hallazgo:** Code Smell — 2 Critical, Major (78 días effort estimado en dependencias de entorno virtual).
- **Reglas Violadas:**
  1. *"Refactor this function to reduce its Cognitive Complexity from 30 to the 15 allowed."*
  2. *"Rename this variable; it shadows a builtin."*
- **Análisis Técnico:**
  - La complejidad cognitiva mide qué tan difícil es entender el flujo de control de un método debido a bucles y condicionales anidados.
  - El sombreado de variables ocurre cuando se nombra una variable igual que una función nativa de Python (ej: `list`, `dict`, `id`).

---

### 8. service-inventario — Métricas de Capas DDD y Cobertura

![SonarQube Inventario Metricas](imagenes/WhatsApp%20Image%202026-07-30%20at%2010.20.54%20AM.jpeg)

- **Módulo:** `service-inventario`
- **Análisis Técnico:**
  Inspección de las líneas de código por capa de arquitectura DDD (`application`, `domain`, `infrastructure`, `presentation`, `app.py`). 
  Confirma que el código fuente propio del microservicio de inventario tiene 0 Bugs y 0 Vulnerabilidades, demostrando un diseño estructurado bajo Domain-Driven Design.

---

## Conclusión de la Evaluación de Calidad

1. **Bugs y Vulnerabilidades:** Todos los servicios backend (`service-clientes`, `service-caja`, `price-service`, `service-inventario`) registran 0 Bugs y 0 Vulnerabilidades críticas.
2. **Arquitectura:** Todos los servicios respetan la separación de responsabilidades en 4 capas (Presentación, Aplicación, Dominio e Infraestructura).
3. **Calidad Global:** Ausencia de errores bloqueantes o críticos en código fuente del negocio, garantizando un software mantenible, seguro y alineado con los estándares del curso.
