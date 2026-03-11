# Contexto de Negocio - Plataforma de Venta de Boletos MVP

## 1. Descripción del Proyecto:

**Nombre del Proyecto:** Plataforma de Venta de Boletos MVP (Ticketing Platform MVP)

**Objetivo del Proyecto:** Entregar un MVP que permita a usuarios comprar boletos para eventos con la experiencia mínima viable: seleccionar asientos, reservar temporalmente, procesar pagos simulados, emitir boletos con código QR y enviar notificación por email. El sistema implementa una arquitectura de microservicios .NET con patrones hexagonales para garantizar escalabilidad y mantenibilidad.

## 2. Flujos Críticos del Negocio:

**Principales Flujos de Trabajo:**
- **Flujo de Compra Principal (P1 - Crítico):** Búsqueda de eventos → Selección de asientos → Reserva temporal (15 min TTL) → Agregar al carrito → Procesamiento de pago → Generación de ticket PDF con QR → Notificación por email
- **Flujo de Navegación y Descubrimiento (P2):** Exploración de eventos, venues y mapas de asientos para encontrar y seleccionar asientos disponibles
- **Flujo de Gestión de Organizadores (P3):** Creación de eventos y configuración de asientos de venues para habilitar la venta

**Módulos o Funcionalidades Críticas:**
- **Catalog Service:** Gestión de eventos, venues y configuración de asientos
- **Inventory Service:** Control de disponibilidad de asientos y reservas temporales con TTL
- **Ordering Service:** Gestión de carritos de compra y órdenes
- **Payment Service:** Procesamiento de pagos (simulado para MVP)
- **Fulfillment Service:** Generación de tickets PDF con códigos QR
- **Notification Service:** Envío de confirmaciones por email
- **Identity Service:** Autenticación y autorización JWT

## 3. Reglas de Negocio y Restricciones:

**Reglas de Negocio Relevantes:**
- **Reserva Temporal:** Los asientos se reservan por 15 minutos máximo; después expiran automáticamente y vuelven a estar disponibles
- **Estados de Asientos:** Ciclo de vida estricto: `available` → `reserved` → `sold`
- **Concurrencia:** Implementación de locks distribuidos (Redis) y locking optimista (PostgreSQL) para prevenir doble-venta de asientos
- **Integridad Transaccional:** Priorizar transacciones locales ACID; evitar sagas complejas en el MVP
- **Arquitectura Hexagonal Obligatoria:** Cada microservicio debe seguir estrictamente Ports & Adapters con dominio puro aislado
- **Schema Isolation:** Cada bounded context opera en su propio schema PostgreSQL (`bc_<nombre>`)

**Regulaciones o Normativas:**
- Cumplimiento con patrones de seguridad JWT para autenticación
- Logs estructurados y trazabilidad completa para auditoría (OpenTelemetry)
- Manejo seguro de datos de pago (aunque simulado en MVP)
- Políticas de retención de datos según esquemas por servicio

## 4. Perfiles de Usuario y Roles

### Perfiles o Roles de Usuario en el Sistema

Basado en la arquitectura del sistema, se identifican los siguientes perfiles implícitos:

| Rol | Descripción |
|---|---|
| **Usuario / Comprador** | Consulta eventos, reserva asientos, gestiona su carrito y realiza pagos. Recibe notificaciones y tickets. |
| **Sistema / Servicios internos** | Los microservicios interactúan entre sí mediante eventos Kafka y llamadas REST internas (no es un rol de usuario humano, pero es un actor clave del sistema). |

> **Nota:** La documentación pública del repositorio no detalla roles administrativos explícitos en esta versión MVP. El Identity Service sugiere que existirá un sistema de autenticación con roles diferenciados en iteraciones futuras.

### Permisos y Limitaciones de Cada Perfil

- El **Usuario / Comprador** puede consultar el catálogo, reservar asientos, gestionar su carrito, pagar y recibir su ticket.
- Un asiento reservado por un usuario no puede ser reservado simultáneamente por otro (garantizado por el lock distribuido en Redis).
- El acceso al sistema requiere autenticación a través del **Identity Service**.

---

## 5. Condiciones del Entorno Técnico:

**Plataformas Soportadas:**
- **Backend:** Microservicios .NET 8+ desplegados en contenedores Docker
- **Frontend:** Aplicación web Next.js con componentes React y TypeScript
- **Base de Datos:** Instancia única PostgreSQL compartida con esquemas aislados por bounded context
- **Entorno de Desarrollo:** Docker Compose para orquestación local completa

**Tecnologías o Integraciones Clave:**
- **Stack Base:** .NET 8+, EF Core 8+ con Npgsql, MediatR, FluentValidation
- **Messaging:** Apache Kafka (Confluent.Kafka) para eventos asíncronos entre servicios  
- **Caché/Locks:** Redis (StackExchange.Redis) para locks distribuidos y reservas temporales
- **Generación PDF/QR:** QRCoder + PdfSharpCore para tickets
- **Observabilidad:** Serilog + OpenTelemetry con exportación a Jaeger
- **Testing:** xUnit + Testcontainers para pruebas de integración
- **Autenticación:** JWT tokens manejados por Identity Service
- **Orquestación:** Docker Compose (desarrollo) con soporte para Kubernetes (futuro)

## 6. Casos Especiales o Excepciones:

**Escenarios Alternos o Excepciones que Deben Considerarse:**

- **Expiración de Reservas Durante Pago:** Si una reserva expira mientras el usuario está en proceso de pago, el pago debe fallar o implementar flujo compensatorio para re-verificar disponibilidad de asiento

- **Fallas en Generación de Tickets Post-Pago:** Si la generación del PDF falla después de un pago exitoso, se debe implementar retry automático y marcar la orden como `pending-fulfillment` con notificación a operaciones

- **Reservas Concurrentes del Mismo Asiento:** Múltiples usuarios intentando reservar el mismo asiento simultáneamente deben ser manejados con locks distribuidos Redis + optimistic locking PostgreSQL

- **Fallas de Conectividad con Servicios Externos:** Implementar circuit breakers y degradación elegante cuando servicios dependientes (Redis, Kafka) no estén disponibles

- **Migraciones de Schema Conflictivas:** Cada servicio debe aplicar migraciones únicamente a su schema asignado (`bc_<servicio>`); requerir revisión de al menos 2 personas para cambios de schema

- **Pérdida de Eventos Entre DB y Kafka:** Para el MVP se acepta el riesgo; planificar patrón Outbox en fases posteriores para garantías más fuertes de consistency

- **Datos de Pago en Desarrollo:** Usar adaptadores simulados con validación de flujo completo pero sin integración real (Stripe sandbox diferido al backlog)

- **TTL de Reservas en Múltiples Zonas Horarias:** Las reservas de 15 minutos deben manejarse en UTC para evitar inconsistencias en deployments distribuidos

---

**Fecha de Creación:** 11 de marzo de 2026  
**Versión del Proyecto:** MVP v1.0  
**Basado en Especificaciones:** `specs/001-ticketing-mvp/spec.md` y Constitution v1.1.0