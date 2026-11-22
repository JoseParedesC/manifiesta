## **1. Entidades que realmente necesitas**

Esta es mi propuesta reducida, tomando como referencia las columnas y movimientos que aparecen en las tres hojas del Excel.

### **A. Participantes**

| Entidad           | Nombre en código  | Necesidad                                                                    |
| ----------------- | ----------------- | ---------------------------------------------------------------------------- |
| Empresa / tercero | `BusinessPartner` | Clientes que contratan viajes, transportistas, proveedores y otros terceros. |
| Conductor         | `Driver`          | Conductor responsable de los viajes, anticipos, salarios y liquidaciones.    |
| Socio             | `Partner`         | Persona que participa en las utilidades del negocio, cuando aplique.         |

**Importante:** no crearía inicialmente entidades independientes para `Shipper`, `Consignor`, `Consignee`, `Broker` y `Carrier`.

Una misma empresa puede desempeñar distintos papeles. Por ejemplo, ESENTIA aparece como empresa contratante en los registros. Es más práctico tener una entidad `BusinessPartner` con roles o tipos de relación.

### **B. Recursos**

| Entidad          | Nombre en código | Necesidad                                         |
| ---------------- | ---------------- | ------------------------------------------------- |
| Vehículo         | `Vehicle`        | Identificado principalmente por su placa.         |
| Tipo de vehículo | `VehicleType`    | Clasificación de mula, camión, tractocamión, etc. |

No necesitas crear `SemiTrailer`, `Trailer` y `VehicleCombination` desde el principio, salvo que debas controlar esos activos por separado.

### **C. Operaciones**

| Entidad                        | Nombre en código        | Necesidad                                             |
| ------------------------------ | ----------------------- | ----------------------------------------------------- |
| Viaje / servicio de transporte | `TransportService`      | Registra cada operación realizada.                    |
| Tipo de movimiento             | `TransportMovementType` | TRAS, EXPO, IMPO, VACIO, según tus registros.         |
| Manifiesto de carga            | `CargoManifest`         | Número y datos del manifiesto, cuando exista.         |
| Ubicación operativa            | `Location`              | Orígenes y destinos como PLANTA, SPRC, CONTECAR, etc. |

En tu archivo, cada fila de la hoja STT126 representa una operación con fecha, placa, origen, destino, empresa, tipo de movimiento y flete.

Por tanto, `TransportService` debe ser el centro del proceso operativo.

### **D. Finanzas y liquidaciones**

| Entidad                | Nombre en código       | Necesidad                                                  |
| ---------------------- | ---------------------- | ---------------------------------------------------------- |
| Concepto financiero    | `FinancialConcept`     | ACPM, peajes, sueldo, lavada, comisión, préstamos, etc.    |
| Movimiento financiero  | `FinancialTransaction` | Registra ingresos, egresos, anticipos, pagos y descuentos. |
| Liquidación            | `Settlement`           | Consolida el resultado económico de un período o viaje.    |
| Detalle de liquidación | `SettlementDetail`     | Desglosa ingresos, gastos, descuentos y otros conceptos.   |
| Factura                | `Invoice`              | Documenta facturación emitida o recibida.                  |
| Aplicación de pago     | `PaymentAllocation`    | Relaciona pagos y anticipos con sus obligaciones.          |

Aquí hay una decisión importante: no debes crear una tabla independiente para cada clase de gasto.

Por ejemplo, ACPM, peajes, cargue, descargue, lavado, seguros y préstamos pueden representarse mediante `FinancialConcept` y `FinancialTransaction`.

## **2. Cómo se relacionan en la operación real**

Este sería el flujo que modelaría:

**BusinessPartner**

Empresa contratante

Solicita el servicio

**TransportService**

Fecha · Origen · Destino · Flete · Tipo de movimiento

Asigna recursos

**Vehicle**

Placa

**Driver**

Conductor

Genera movimientos económicos

**FinancialTransaction**

Fletes · Anticipos · ACPM · Peajes · Gastos · Pagos

Consolida el período

**Settlement**

Liquidación del vehículo y del conductor

## **3. Lo que revela la hoja de liquidación**

La hoja `LIQUIDACION STT126` muestra algo que debemos contemplar desde el principio: existen dos niveles de liquidación.

**Liquidación del vehículo:**

- Producido total del mes.
- Anticipos entregados.
- ACPM.
- Peajes.
- Gastos adicionales.
- Salario del conductor.
- Seguridad social.
- Administración.
- Seguros.
- Llantas y mantenimiento.
- Utilidad neta.
- Distribución de utilidades a socios.

**Liquidación del conductor:**

- Salario.
- Auxilio de transporte.
- Descuentos.
- Anticipos.
- Gastos asumidos.
- Diferencias o descuadres.
- Total a pagar.

No mezclaría ambas liquidaciones en una sola tabla. Utilizaría `Settlement` con diferentes tipos de liquidación.

## **4. Modelo definitivo recomendado**

Esta sería mi primera versión del modelo de dominio:

| Entidad                 | Responsabilidad                                  |
| ----------------------- | ------------------------------------------------ |
| `BusinessPartner`       | Empresas y terceros relacionados con el negocio. |
| `Driver`                | Conductores.                                     |
| `Partner`               | Socios del negocio.                              |
| `Vehicle`               | Vehículos administrados.                         |
| `VehicleType`           | Clasificación de vehículos.                      |
| `Location`              | Orígenes y destinos.                             |
| `TransportService`      | Operaciones o viajes realizados.                 |
| `TransportMovementType` | Tipos TRAS, EXPO, IMPO, VACIO.                   |
| `CargoManifest`         | Manifiestos asociados a operaciones.             |
| `FinancialConcept`      | Clasificación de ingresos, gastos y descuentos.  |
| `FinancialTransaction`  | Movimientos económicos.                          |
| `Settlement`            | Cabecera de liquidaciones.                       |
| `SettlementDetail`      | Detalles de liquidaciones.                       |
| `Invoice`               | Facturas emitidas y recibidas.                   |
| `PaymentAllocation`     | Aplicación de pagos y anticipos.                 |

15 entidades propuestas

## **5. Entidades que NO crearía todavía**

Evitaría sobredimensionar el proyecto:

| Entidad descartada inicialmente | Motivo                                                                                                              |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `Shipper`                       | Se resuelve con `BusinessPartner` y sus roles.                                                                      |
| `Consignor`                     | No se evidencia la necesidad de gestionarlo independientemente.                                                     |
| `Consignee`                     | Puede incorporarse como participante del servicio si se requiere.                                                   |
| `Broker`                        | No aparece como proceso diferenciado en los registros.                                                              |
| `Trailer`                       | No se evidencia control independiente de remolques.                                                                 |
| `Route`                         | Origen y destino son suficientes inicialmente.                                                                      |
| `FuelPurchase`                  | Se representa mediante movimientos financieros.                                                                     |
| `TollPayment`                   | Se representa mediante movimientos financieros.                                                                     |
| `DriverLoan`                    | Los préstamos pueden manejarse como movimientos y obligaciones.                                                     |
| `AccountsReceivable`            | Puede derivarse inicialmente de facturas y saldos pendientes, aunque posteriormente conviene formalizar la cartera. |
