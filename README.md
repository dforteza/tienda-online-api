# Tienda API

## Descripción

*REST API para gestión de ventas de una tienda online, desarrollada como reto técnico profesional en 5 días.*

## Stack

![Java](https://img.shields.io/badge/Java_17-ED8B00?style=flat&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3-6DB33F?style=flat&logo=spring&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL_8-4479A1?style=flat&logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Flyway](https://img.shields.io/badge/Flyway-62B347?style=flat&logo=flyway&logoColor=white)
![Swagger](https://img.shields.io/badge/OpenAPI-85EA2D?style=flat&logo=swagger&logoColor=black)

`Spring Data JPA` · `MapStruct` · `JaCoCo`

## Arquitectura

Arquitectura por capas con separación estricta de responsabilidades:

```
Controller -> Service -> Repository -> Domain
```

- **DTOs** de entrada y salida desacoplados de las entidades
- **MapStruct** para mapeos en tiempo de compilación
- **GlobalExceptionHandler** con contrato de error uniforme
- **Soft delete** en productos para preservar integridad del historial de pedidos

## Estructura de paquetes

```
src/main/java/com/delogica/tienda_api/
├── controller/        # Endpoints REST, mapeo DTO ↔ respuesta HTTP
├── service/
│   ├── interfaces/    # Contratos de servicio
│   └── impl/          # Lógica de negocio
├── repository/        # Acceso a datos con Spring Data JPA
├── domain/            # Entidades JPA y enums
├── dto/
│   ├── request/       # DTOs de entrada con validaciones
│   ├── update/        # DTOs de actualización con validaciones
│   └── response/      # DTOs de salida
├── mapper/            # Interfaces MapStruct
├── exception/         # Excepciones y GlobalExceptionHandler
└── security/config/   # OpenAPI
```

## Módulos

| Módulo | Endpoints |
|---|---|
| Productos | CRUD + soft delete + paginación y filtrado por nombre |
| Clientes | CRUD + gestión de direcciones + paginación y filtrado por email |
| Pedidos | Creación con validación de stock · Cambio de estado · Paginación y filtrado |

## Decisiones técnicas destacadas

- **Soft delete en productos** — los productos no se eliminan físicamente para preservar el historial de pedidos. `active = false` los excluye de consultas y nuevos pedidos.
- **Snapshot de precio** — el `unitPrice` se captura en el momento de la compra. Los cambios futuros de precio no afectan a pedidos ya creados.
- **Máquina de estados** — las transiciones de pedido están validadas: `CREATED → PAID → SHIPPED`, `CREATED/PAID → CANCELLED`. La cancelación restaura el stock automáticamente.
- **Dirección de envío validada** — al crear un pedido se verifica que la dirección de envío pertenece al cliente que realiza el pedido.

## Instrucciones

### Docker

```bash
docker compose up --build
```

La API quedará disponible en `http://localhost:8080`.

### Local

```bash
# Requisitos: Java 17, MySQL 8
mvn spring-boot:run
```

Configurar credenciales de BD en `application.yml` o mediante variables de entorno:

```
SPRING_DATASOURCE_URL
SPRING_DATASOURCE_USERNAME
SPRING_DATASOURCE_PASSWORD
```

### Documentación interactiva (Swagger)

```
http://localhost:8080/swagger-ui.html
```

### Tests

```bash
mvn verify
```

Reporte de cobertura en `target/site/jacoco/index.html`.

## Recursos

- [Diagrama ERD](docs/img/ERD-TiendaOnline.jpg)
- Decisiones técnicas detalladas: [Customer](docs/decisions/customer-decisions.md) · [Order](docs/decisions/order-decisions.md) · [Product](docs/decisions/product-decisions.md)
- [Esquema de base de datos](docs/database-schema.txt)
- [Plan de tests unitarios](docs/test-cases.txt) · [Reporte de cobertura](docs/test-coverage.txt)
- Colección Postman: `docs/Tienda Online API.postman_collection.json` (+ entorno local incluido)

## Mejoras futuras

- Autenticación y autorización con **Spring Security + JWT**
- Tests de integración con **MockMvc** y **Testcontainers**
- Anonimización de datos de cliente antes del borrado por cumplimiento **GDPR**
