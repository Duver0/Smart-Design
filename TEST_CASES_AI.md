# MATRIZ DE CASOS DE PRUEBA - SMART DESIGN

## HU-P1: Compra de Boleto (Critical Path)

**Historia de Usuario:** Como Cliente, quiero seleccionar un asiento, reservarlo temporalmente, agregarlo al carrito y pagar para recibir mi boleto con QR.

---

## 1. CASOS DE PRUEBA ORIGINALES IMPLEMENTADOS

| ID Prueba | Escenario de Aceptación | Nivel | Técnica / Límites Reales Probados | Evidencia / Ubicación | Estado |
|-----------|-------------------------|-------|-------------------------------------|----------------------|--------|
| TC-P1-01 | Selección y reserva con TTL 15 min. | Unidad/Dominio | Límites: 14:59 (OK), 15:01 (Expira) | ReservationTests.cs | ✅ Implementado |
| TC-P1-02 | Bloqueo de doble reserva simultánea | Aplicación | Concurrencia: Distributed Lock (Redis) | CreateReservationCommandHandlerTests.cs | ✅ Implementado |
| TC-P1-03 | Conversión de Reserva a Orden Draft | Unitario | Transición: Reserva -> Orden (1:1) | CheckoutOrderHandlerTests.cs | ✅ Implementado |
| TC-P1-04 | Pago Exitoso (Simulado) -> Orden Paid | Aplicación | Partición: Balance >= Total | ProcessPaymentHandlerTests.cs | ✅ Implementado |
| TC-P1-05 | Fallo en Pago -> Orden Pending/Failed | Aplicación | Partición: Balance < Total | ProcessPaymentHandlerTests.cs | ✅ Implementado |
| TC-P1-06 | Generación de Ticket PDF con QR | Integración | Validación: QR contiene TicketId hash | TicketEntityTests.cs | ✅ Implementado |

---

## 2. CASOS DE PRUEBA PROPUESTOS (EXPANDIDOS)

| ID Prueba | Caso de Prueba | Prioridad | Técnica de Prueba | Estado Sugerido | Justificación Técnica |
|-----------|----------------|-----------|-------------------|-----------------|----------------------|
| TC-P1-07 | **Selección exitosa de asiento disponible**<br/>Dado que soy un cliente autenticado y visualizo el mapa de asientos<br/>Cuando selecciono un asiento disponible<br/>Entonces se reserva por 15 min y cambia a "reservado" | Alta | Caja Negra - Partición | 🔄 A Implementar | Valida el flujo básico de selección y cambio de estado |
| TC-P1-08 | **Expiración de reserva temporal**<br/>Dado que he reservado un asiento<br/>Cuando pasan 15 minutos sin completar compra<br/>Entonces el asiento vuelve a "disponible" | Crítica | Temporal - TTL | 🔄 A Implementar | Evita bloqueos permanentes de inventario |
| TC-P1-09 | **Agregar asiento reservado al carrito**<br/>Dado que tengo un asiento reservado<br/>Cuando lo agrego al carrito<br/>Entonces se asocia correctamente | Media | Caja Negra - Flujo | 🔄 A Implementar | Garantiza continuidad del proceso de compra |
| TC-P1-10 | **Reserva concurrente del mismo asiento**<br/>Dado que otro usuario reservó el asiento<br/>Cuando intento reservar el mismo<br/>Entonces recibo notificación de no disponible | Crítica | Concurrencia - Race Condition | 🔄 A Implementar | Previene overselling mediante locks distribuidos |
| TC-P1-11 | **Proceso de pago exitoso**<br/>Dado que tengo asiento en carrito<br/>Cuando pago con datos válidos<br/>Entonces recibo boleto con QR por email | Alta | End-to-End | 🔄 A Implementar | Valida el flujo completo de compra exitosa |
| TC-P1-12 | **Pago fallido por datos inválidos**<br/>Dado que tengo asiento en carrito<br/>Cuando ingreso datos inválidos de tarjeta<br/>Entonces el pago se rechaza y asiento permanece reservado | Alta | Caja Negra - Datos Inválidos | 🔄 A Implementar | Manejo de errores en gateway de pago |
| TC-P1-13 | **Expiración durante el pago**<br/>Dado que intento pagar después de 15 min<br/>Cuando confirmo el pago<br/>Entonces se rechaza por expiración | Media | Temporal - Edge Case | 🔄 A Implementar | Validación de TTL durante transacciones |
| TC-P1-14 | **Fallo en generación de boleto**<br/>Dado que el pago fue exitoso<br/>Cuando falla la generación del PDF<br/>Entonces se reintenta y marca como "pendiente" | Media | Recuperación - Retry Pattern | 🔄 A Implementar | Resiliencia en servicios de terceros |
| TC-P1-15 | **Notificación por email**<br/>Dado que completé la compra<br/>Cuando se genera el boleto<br/>Entonces recibo email con PDF adjunto | Baja | Integración - Email Service | 🔄 A Implementar | Confirmación de entrega al cliente |
| TC-P1-16 | **Validación longitud máxima de tarjeta**<br/>Dado que ingreso datos de tarjeta<br/>Cuando excedo longitud máxima<br/>Entonces se rechaza con mensaje de error | Baja | Límites - Validation | 🔄 A Implementar | Sanitización de entrada de datos |
| TC-P1-17 | **Usuario no autenticado**<br/>Dado que no he iniciado sesión<br/>Cuando intento reservar asiento<br/>Entonces me redirige a autenticación | Media | Seguridad - AuthZ | 🔄 A Implementar | Control de acceso a funcionalidades |
| TC-P1-18 | **Pérdida de conectividad con servicios**<br/>Dado que hay fallo en Redis/Kafka<br/>Cuando intento comprar<br/>Entonces se activa circuit breaker | Media | Resiliencia - Circuit Breaker | 🔄 A Implementar | Degradación controlada ante fallos |
| TC-P1-19 | **Prevención de doble-venta**<br/>Dado que un asiento está "vendido"<br/>Cuando otro usuario lo selecciona<br/>Entonces se impide la selección | Crítica | Estado - Consistencia | 🔄 A Implementar | Integridad del inventario |
| TC-P1-20 | **Descarga manual desde historial**<br/>Dado que compré un boleto<br/>Cuando accedo a mi historial<br/>Entonces puedo redescargar el PDF | Baja | Persistencia - Retrieval | 🔄 A Implementar | Recuperación de documentos |
| TC-P1-21 | **Políticas de retención de datos**<br/>Dado que consulto órdenes pasadas<br/>Cuando superan el período de retención<br/>Entonces solo veo datos permitidos | Baja | Compliance - GDPR | 🔄 A Implementar | Cumplimiento normativo |
| TC-P1-22 | **Reserva en evento agotado**<br/>Dado que todos los asientos están "vendidos"<br/>Cuando intento seleccionar<br/>Entonces muestra "sin disponibilidad" | Media | Inventario - Stock Validation | 🔄 A Implementar | UX cuando no hay stock |

---

## 3. TABLA DE AJUSTES REALIZADOS POR EL PROBADOR

| ID Original | ID Ajustado | Tipo de Ajuste | Descripción del Cambio | Justificación Técnica | Impacto |
|-------------|-------------|----------------|------------------------|----------------------|---------|
| TC-P1-01 | TC-P1-01-A | Técnico | Se agregó validación de timezone UTC para TTL | Los TTL deben ser consistentes independientemente de la zona horaria del cliente | Medio |
| TC-P1-02 | TC-P1-02-A | Implementación | Se cambió de database lock a Redis distributed lock | Database locks no escalan en arquitectura distribuida | Alto |
| TC-P1-04 | TC-P1-04-A | Simulación | Se implementó mock del Payment Gateway real | Payment sandbox tiene límites de requests que afectan CI/CD | Medio |
| TC-P1-06 | TC-P1-06-A | Seguridad | Se agregó validación de firma digital en QR | QR codes pueden ser falsificados sin validación criptográfica | Alto |
| TC-P1-10 | TC-P1-10-A | Concurrencia | Se implementó test de load con 1000 usuarios concurrentes | Casos de 2 usuarios son insuficientes para detectar race conditions reales | Crítico |
| TC-P1-13 | TC-P1-13-A | Temporal | Se ajustó timing a 14:58 y 15:02 para evitar flaky tests | Tests con timing exacto de 15:00 fallan por latencia de red | Medio |
| TC-P1-14 | TC-P1-14-A | Observabilidad | Se agregaron métricas de retry attempts y alertas | Fallos silenciosos en retry pueden afectar SLA sin detección | Alto |
| TC-P1-18 | TC-P1-18-A | Resiliencia | Se implementó fallback a base de datos local | Circuit breaker completo deja sistema inoperativo | Crítico |

---

# HU-P2: Navegación y Descubrimiento

**Historia de Usuario:** Como Visitante, quiero ver eventos y mapas de asientos para elegir qué reservar.

---

## 1. CASOS DE PRUEBA ORIGINALES IMPLEMENTADOS

| ID Prueba | Escenario de Aceptación | Nivel | Técnica / Límites Reales Probados | Evidencia / Ubicación | Estado |
|-----------|-------------------------|-------|-------------------------------------|----------------------|--------|
| TC-P2-01 | Consulta de catálogo de eventos activos | Aplicación | Límite: EventDate > Now | GetAllEventsHandlerTests.cs | ✅ Implementado |
| TC-P2-02 | Mapa de asientos refleja disponibilidad | Contrato | Sync: Catalog <> Inventory | system-e2e-test.sh / SeatTests.cs | ✅ Implementado |

---

## 2. CASOS DE PRUEBA PROPUESTOS (EXPANDIDOS)

| ID Prueba | Caso de Prueba | Prioridad | Técnica de Prueba | Estado Sugerido | Justificación Técnica |
|-----------|----------------|-----------|-------------------|-----------------|----------------------|
| TC-P2-03 | **Exploración de eventos disponibles**<br/>Dado que soy un visitante en la plataforma<br/>Cuando accedo a la lista de eventos<br/>Entonces visualizo eventos abiertos con información básica | Alta | Caja Negra - Flujo Principal | 🔄 A Implementar | Validación del punto de entrada principal del sistema |
| TC-P2-04 | **Filtrado de eventos por criterios**<br/>Dado que visualizo la lista de eventos<br/>Cuando aplico filtros por nombre, fecha y ubicación<br/>Entonces se muestran solo eventos que cumplen criterios | Media | Caja Negra - Partición | 🔄 A Implementar | Optimización de búsqueda y UX de descubrimiento |
| TC-P2-05 | **Visualización de mapa interactivo**<br/>Dado que selecciono un evento<br/>Cuando accedo al mapa de asientos<br/>Entonces veo mapa interactivo con estados en tiempo real | Crítica | Integración - Real-time | 🔄 A Implementar | Core functionality para decisión de compra |
| TC-P2-06 | **Actualización tiempo real de asientos**<br/>Dado que visualizo el mapa interactivo<br/>Cuando otro usuario reserva/compra un asiento<br/>Entonces el estado se actualiza en tiempo real | Crítica | Concurrencia - WebSocket/SSE | 🔄 A Implementar | Prevención de overselling y consistencia de datos |
| TC-P2-07 | **Visualización de eventos agotados**<br/>Dado que accedo a la lista<br/>Cuando existe un evento totalmente vendido<br/>Entonces se marca como agotado | Media | Estado - Inventory Validation | 🔄 A Implementar | Gestión de expectativas del usuario |
| TC-P2-08 | **Estados visuales de asientos**<br/>Dado que visualizo el mapa de asientos<br/>Cuando hay asientos en distintos estados<br/>Entonces se distinguen visualmente disponibles/reservados/vendidos | Alta | UI/UX - Estado Visual | 🔄 A Implementar | Claridad en la interfaz para toma de decisiones |
| TC-P2-09 | **Validación límites de filtros**<br/>Dado que ingreso criterios en filtros<br/>Cuando excedo longitud máxima en filtro de nombre<br/>Entonces muestra error y no aplica filtro | Baja | Límites - Input Validation | 🔄 A Implementar | Sanitización de entrada y prevención de errores |
| TC-P2-10 | **Exploración sin autenticación**<br/>Dado que soy visitante sin sesión<br/>Cuando exploro eventos y mapas<br/>Entonces permite navegación sin mostrar opciones de compra | Media | Seguridad - AuthN/AuthZ | 🔄 A Implementar | Modelo freemium y control de acceso |
| TC-P2-11 | **Sin eventos disponibles**<br/>Dado que accedo a la lista<br/>Cuando no hay eventos para reserva<br/>Entonces muestra mensaje de no disponibilidad | Baja | Edge Case - Estado Vacío | 🔄 A Implementar | UX para estados sin contenido |
| TC-P2-12 | **Filtrado por ubicaciones preferidas**<br/>Dado que tengo preferencias de ubicación<br/>Cuando filtro por ubicación<br/>Entonces muestra solo eventos en venues seleccionados | Baja | Personalización - User Preferences | 🔄 A Implementar | Mejora de experiencia personalizada |
| TC-P2-13 | **Manejo de pérdida de conectividad**<br/>Dado que exploro eventos y mapas<br/>Cuando ocurre pérdida de conectividad<br/>Entonces muestra error y permite reintentar | Media | Resiliencia - Network Failure | 🔄 A Implementar | Experiencia robusta ante fallos de red |
| TC-P2-14 | **Visualización de detalles de evento**<br/>Dado que selecciono un evento<br/>Cuando accedo a su detalle<br/>Entonces muestra información completa del evento | Alta | Navegación - Drill Down | 🔄 A Implementar | Información suficiente para decisión de compra |
| TC-P2-15 | **Consulta multi-timezone**<br/>Dado que visualizo el mapa de asientos<br/>Cuando hay reservas TTL en distintas zonas horarias<br/>Entonces muestra disponibilidad usando UTC consistente | Media | Temporal - Timezone Handling | 🔄 A Implementar | Consistencia global en aplicaciones distribuidas |

---

## 3. TABLA DE AJUSTES REALIZADOS POR EL PROBADOR

| ID Original | ID Ajustado | Tipo de Ajuste | Descripción del Cambio | Justificación Técnica | Impacto |
|-------------|-------------|----------------|------------------------|----------------------|---------|
| TC-P2-01 | TC-P2-01-A | Performance | Se agregó paginación para catálogo con >1000 eventos | Consultas sin límite causan timeouts en UI y sobrecarga de red | Alto |
| TC-P2-02 | TC-P2-02-A | Sincronización | Se implementó cache Redis con TTL 30s para estados de asientos | Consultas directas a DB cada 100ms generan carga excesiva | Crítico |
| TC-P2-05 | TC-P2-05-A | Real-time | Se cambió polling a WebSocket para actualizaciones de mapa | Polling cada 2s no es tiempo real y consume ancho de banda | Alto |
| TC-P2-06 | TC-P2-06-A | Escalabilidad | Se implementó message queuing (Kafka) para eventos de estado | Notificaciones síncronas no escalan con miles de usuarios concurrentes | Crítico |
| TC-P2-04 | TC-P2-04-A | Indexación | Se agregaron índices compuestos para filtros combinados | Filtros sin índices causan table scans en catálogos grandes | Alto |
| TC-P2-13 | TC-P2-13-A | UX | Se implementó cache local con service worker para modo offline | Pérdida total de funcionalidad sin conectividad afecta UX | Medio |
| TC-P2-15 | TC-P2-15-A | Consistencia | Se estandarizó todo el sistema a UTC con conversión en frontend | Inconsistencias de timezone causan bugs en reservas TTL | Crítico |

---

# HU-P3: Gestión por Organizador

**Historia de Usuario:** Como Organizador, quiero crear eventos y configurar asientos.

---

## 1. CASOS DE PRUEBA ORIGINALES IMPLEMENTADOS

| ID Prueba | Escenario de Aceptación | Nivel | Técnica / Límites Reales Probados | Evidencia / Ubicación | Estado |
|-----------|-------------------------|-------|-------------------------------------|----------------------|--------|
| TC-P3-01 | Creación de evento con asientos | Integración | Escalabilidad: Max 50k asientos | CreateEventCommandHandlerTests.cs | ✅ Implementado |

---

## 2. CASOS DE PRUEBA PROPUESTOS (EXPANDIDOS)

| ID Prueba | Caso de Prueba | Prioridad | Técnica de Prueba | Estado Sugerido | Justificación Técnica |
|-----------|----------------|-----------|-------------------|-----------------|----------------------|
| TC-P3-02 | **Creación exitosa con información completa**<br/>Dado que soy organizador autenticado<br/>Cuando ingreso datos válidos y configuro layout con precios<br/>Entonces se guarda correctamente y queda listo para publicación | Alta | Caja Negra - Flujo Completo | 🔄 A Implementar | Validación del flujo principal de creación de eventos |
| TC-P3-03 | **Validación de campos obligatorios**<br/>Dado que soy organizador autenticado<br/>Cuando intento crear evento sin campos obligatorios<br/>Entonces muestra errores específicos y bloquea guardado | Alta | Límites - Validation | 🔄 A Implementar | Integridad de datos y prevención de estados inconsistentes |
| TC-P3-04 | **Validación de formato y rango de fecha**<br/>Dado que ingreso fecha del evento<br/>Cuando uso formato incorrecto o fecha pasada<br/>Entonces muestra error y bloquea creación | Media | Límites - Temporal Validation | 🔄 A Implementar | Consistencia temporal y validación de business rules |
| TC-P3-05 | **Configuración de layout con precios válidos**<br/>Dado que configuro layout de asientos<br/>Cuando asigno precios positivos y restricciones válidas<br/>Entonces acepta configuración y la asocia al evento | Alta | Caja Negra - Business Logic | 🔄 A Implementar | Core functionality para monetización del evento |
| TC-P3-06 | **Validación de precios inválidos**<br/>Dado que configuro layout de asientos<br/>Cuando ingreso precios negativos o no numéricos<br/>Entonces muestra errores y bloquea guardado | Media | Límites - Input Validation | 🔄 A Implementar | Prevención de datos inconsistentes en pricing |
| TC-P3-07 | **Aplicación de restricciones por zona**<br/>Dado que configuro restricciones específicas por zona<br/>Cuando guardo la configuración<br/>Entonces valida y aplica restricciones correctamente | Media | Business Rules - Constraints | 🔄 A Implementar | Cumplimiento de regulaciones y accesibilidad |
| TC-P3-08 | **Validación de capacidad máxima**<br/>Dado que configuro layout con >50k asientos<br/>Cuando intento guardar<br/>Entonces muestra error de capacidad excedida | Crítica | Límites - Scalability | 🔄 A Implementar | Prevención de performance degradation y resource exhaustion |
| TC-P3-09 | **Creación sin autenticación**<br/>Dado que no estoy autenticado como organizador<br/>Cuando intento crear eventos<br/>Entonces redirige a autenticación | Alta | Seguridad - AuthN/AuthZ | 🔄 A Implementar | Control de acceso a funcionalidades administrativas |
| TC-P3-10 | **Edición previa a publicación**<br/>Dado que he creado evento no publicado<br/>Cuando modifico información o layout<br/>Entonces permite guardar cambios y actualiza configuración | Media | Estado - Draft Mode | 🔄 A Implementar | Flexibilidad en configuración antes de go-live |
| TC-P3-11 | **Publicación exitosa tras configuración**<br/>Dado que tengo evento con layout válido<br/>Cuando publico el evento<br/>Entonces cambia a estado "publicado" y es visible | Alta | Estado - State Transition | 🔄 A Implementar | Workflow management y visibilidad del evento |
| TC-P3-12 | **Fallo de conectividad al crear**<br/>Dado que estoy creando evento<br/>Cuando ocurre error de conectividad<br/>Entonces muestra mensaje amigable y preserva datos | Media | Resiliencia - Network Failure | 🔄 A Implementar | UX robusta ante fallos de infraestructura |
| TC-P3-13 | **Validación de longitud máxima**<br/>Dado que ingreso datos en campos de texto<br/>Cuando excedo longitud máxima<br/>Entonces muestra error y bloquea guardado | Baja | Límites - String Length | 🔄 A Implementar | Prevención de DoS y overflow attacks |

---

## 3. TABLA DE AJUSTES REALIZADOS POR EL PROBADOR

| ID Original | ID Ajustado | Tipo de Ajuste | Descripción del Cambio | Justificación Técnica | Impacto |
|-------------|-------------|----------------|------------------------|----------------------|---------|
| TC-P3-01 | TC-P3-01-A | Performance | Se implementó creación batch de asientos en chunks de 1000 | Creación síncrona de 50k asientos causa timeouts y lock DB | Crítico |
| TC-P3-02 | TC-P3-02-A | Transaccional | Se agregó patrón Saga para creación evento + asientos | Fallos parciales dejan datos inconsistentes sin atomicidad | Alto |
| TC-P3-08 | TC-P3-08-A | Escalabilidad | Se implementó pre-validación asíncrona de capacidad | Validación síncrona de 50k asientos bloquea UI por >30s | Alto |
| TC-P3-09 | TC-P3-09-A | Autorización | Se agregó validación de roles granular (org_admin, org_member) | AuthN básico insuficiente para gestión multi-organizador | Medio |
| TC-P3-11 | TC-P3-11-A | Concurrencia | Se implementó lock optimista para publicación simultánea | Publicaciones concurrentes del mismo evento generan duplicados | Alto |
| TC-P3-12 | TC-P3-12-A | Persistencia | Se agregó auto-save cada 30s con localStorage | Pérdida total de configuración compleja afecta productividad | Medio |
| TC-P3-05 | TC-P3-05-A | Validación | Se implementó validación de pricing rules por venue type | Precios sin context business pueden violar regulations | Medio |

---

**Última Actualización:** 12 de marzo de 2026  
**Responsable:** Equipo QA - Smart Design  
**Próxima Revisión:** 19 de marzo de 2026