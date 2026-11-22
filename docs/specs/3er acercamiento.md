# **1. Modelo de dominio ajustado**

Lo dividiría en cinco módulos.

## **A. Administración de terceros y personal**

| Entidad           | Nombre técnico    | Responsabilidad                                         |
| ----------------- | ----------------- | ------------------------------------------------------- |
| Empresa / tercero | `BusinessPartner` | Clientes, proveedores y otros terceros.                 |
| Conductor         | `Driver`          | Datos personales, laborales y asignaciones a vehículos. |
| Socio             | `Partner`         | Personas que participan en las utilidades.              |

No crearía una entidad `Carrier` por ahora. La empresa propietaria de la flota es quien presta el servicio.

`BusinessPartner` debe permitir diferentes roles comerciales sin duplicar empresas. Por ejemplo, ESENTIA y WORD CARGO pueden registrarse como clientes, mientras que una llantería puede ser proveedor.

## **B. Administración de flota**

| Entidad                 | Nombre técnico            | Responsabilidad                                     |
| ----------------------- | ------------------------- | --------------------------------------------------- |
| Vehículo                | `Vehicle`                 | Placa, marca, modelo, capacidad y características.  |
| Tipo de vehículo        | `VehicleType`             | Tractocamión, camión, mula, etc.                    |
| Asignación de conductor | `VehicleDriverAssignment` | Historial de conductores asignados a cada vehículo. |

No incluiría remolques como entidades independientes en la primera versión, salvo que también se administren sus placas, propietarios, mantenimiento y asignaciones.

## **C. Operaciones de transporte**

| Entidad                | Nombre técnico          | Responsabilidad                           |
| ---------------------- | ----------------------- | ----------------------------------------- |
| Servicio de transporte | `TransportService`      | Registro de cada viaje u operación.       |
| Tipo de movimiento     | `TransportMovementType` | TRAS, EXPO, IMPO, VACIO.                  |
| Ubicación              | `Location`              | Origen y destino de los servicios.        |
| Manifiesto             | `CargoManifest`         | Información del manifiesto cuando exista. |

`TransportService` será la entidad central de la operación.

Debe contener, como mínimo:

- Fecha del servicio.
- Vehículo asignado.
- Cliente.
- Origen y destino.
- Tipo de movimiento.
- Número de manifiesto, cuando aplique.
- Valor del flete.
- Porcentaje o valor correspondiente al conductor.
- Retención en la fuente.
- ReteICA.
- Descuentos.
- Valor neto de la operación.

No necesitas crear una entidad `Trip` adicional: para el proceso que muestra tu Excel, `TransportService` representa suficientemente cada operación de transporte.

## **D. Facturación y cartera**

Aquí hay una decisión arquitectónica importante: como facturas tanto por viaje como agrupando varios viajes, la factura no debe depender de una relación uno a uno con `TransportService`.

| Entidad               | Nombre técnico             | Responsabilidad                                   |
| --------------------- | -------------------------- | ------------------------------------------------- |
| Factura               | `Invoice`                  | Facturas emitidas y recibidas.                    |
| Detalle de factura    | `InvoiceDetail`            | Conceptos facturados.                             |
| Aplicación de factura | `InvoiceServiceAllocation` | Relaciona una factura con uno o varios servicios. |
| Pago                  | `Payment`                  | Dinero recibido o desembolsado.                   |
| Aplicación de pago    | `PaymentAllocation`        | Relaciona pagos con facturas u obligaciones.      |

La relación sería:

- Un servicio puede estar asociado a una factura.
- Una factura puede agrupar muchos servicios.
- Una factura puede recibir varios pagos.
- Un pago puede distribuirse entre varias facturas.

Esto permite manejar las dos modalidades de facturación sin duplicar operaciones.

También debes contemplar facturas de proveedores, como las facturas de llantas y parqueadero que aparecen en la liquidación.

Por eso `Invoice` debe distinguir entre:

- `Receivable`: factura emitida, cuenta por cobrar.
- `Payable`: factura recibida, cuenta por pagar.

## **E. Finanzas y liquidaciones**

Este módulo es el más importante para controlar la rentabilidad.

| Entidad                    | Nombre técnico            | Responsabilidad                                                   |
| -------------------------- | ------------------------- | ----------------------------------------------------------------- |
| Concepto financiero        | `FinancialConcept`        | Clasificación de ingresos, gastos, descuentos y anticipos.        |
| Movimiento financiero      | `FinancialTransaction`    | Registro individual de cada movimiento económico.                 |
| Liquidación mensual        | `VehicleSettlement`       | Consolidado mensual de cada vehículo.                             |
| Detalle de liquidación     | `VehicleSettlementDetail` | Desglose de ingresos, costos y gastos.                            |
| Liquidación del conductor  | `DriverSettlement`        | Cálculo del salario, anticipos, descuentos y saldo del conductor. |
| Distribución de utilidades | `ProfitDistribution`      | Registro de las utilidades distribuidas a socios.                 |

### **¿Por qué separar estos conceptos?**

Porque no son lo mismo:

- Un gasto es un movimiento financiero.
- Una liquidación es una consolidación de movimientos.
- Un pago es una salida o entrada real de dinero.
- Una distribución de utilidades es una asignación del resultado económico a los socios.

Por ejemplo, el ACPM de un viaje es un movimiento financiero. Al finalizar enero, ese gasto participa en la liquidación mensual del vehículo.

# **2. Relaciones entre las entidades**

Este es el modelo conceptual que implementaría:

**Operación**

**BusinessPartner**

Cliente contratante

**TransportService**

Flete · Origen · Destino · Fecha · Manifiesto

**Vehicle**

Vehículo propio

**Driver**

Conductor asignado

**Facturación**

**Invoice**

Factura emitida o recibida

InvoiceServiceAllocation / InvoiceDetail

**Payment**

Pagos y aplicaciones

**Control financiero**

**FinancialTransaction**

Ingresos · Gastos · Anticipos · Descuentos

Consolidación mensual por vehículo

**VehicleSettlement**

Ingresos − costos − gastos = utilidad neta

**DriverSettlement**

Liquidación conductor

**ProfitDistribution**

Utilidades socios

Una precisión: el gráfico muestra el flujo funcional, no todas las relaciones físicas de las tablas. Por ejemplo, los movimientos financieros también pueden relacionarse directamente con vehículos, conductores, servicios, facturas y liquidaciones.

# **3. Cómo quedaría la liquidación mensual**

Tomando como referencia la hoja `LIQUIDACION STT126`, cada vehículo tendrá una liquidación independiente por período.

Ejemplo conceptual:

**Liquidación mensual**

Ejemplo

# **STT126**

Enero 2026 · Valores ilustrativos basados en el Excel

Ingresos del vehículo

**$19.331.000**

Costos y gastos

**$11.338.358**

**Utilidad neta**

# **$7.992.642**

**Distribución**

Participación socio (50%)

**$3.996.321**

Los valores anteriores corresponden a los datos de la hoja, no a una fórmula contable validada. Antes de implementar, hay que definir exactamente qué conceptos se descuentan para obtener la utilidad distribuible.

La liquidación del conductor debe ser independiente, porque tiene sus propios conceptos: salario, auxilio, anticipos, gastos, seguridad social y saldo por pagar.

# **4. Catálogo de conceptos financieros**

No crearía una entidad por cada gasto. Utilizaría `FinancialConcept` con categorías.

| Categoría             | Conceptos de ejemplo                          |
| --------------------- | --------------------------------------------- |
| `INCOME`              | Fletes, servicios adicionales.                |
| `FUEL`                | ACPM, combustible.                            |
| `TOLL`                | Peajes.                                       |
| `OPERATIONAL_EXPENSE` | Cargue, descargue, lavada, parqueadero.       |
| `MAINTENANCE`         | Llantas, reparaciones.                        |
| `PAYROLL`             | Salario, auxilio, seguridad social, dotación. |
| `ADVANCE`             | Anticipos al conductor o de clientes.         |
| `LOAN`                | Préstamos a conductores.                      |
| `TAX`                 | Retefuente, ReteICA, 4x1000.                  |
| `ADMINISTRATIVE`      | Administración, satelital, seguros.           |
| `OWNER_DISTRIBUTION`  | Utilidades entregadas a socios.               |

Cada movimiento debe tener, como mínimo:

- Fecha.
- Concepto financiero.
- Valor.
- Tipo: ingreso o egreso.
- Vehículo asociado, cuando aplique.
- Conductor asociado, cuando aplique.
- Servicio asociado, cuando aplique.
- Tercero relacionado, cuando aplique.
- Soporte o referencia documental.
- Estado del movimiento.

No todos los movimientos tienen que estar vinculados a un viaje. Por ejemplo, un seguro anual o un préstamo pueden corresponder al vehículo o conductor, respectivamente, sin pertenecer a un servicio particular.

# **5. Relaciones y cardinalidades recomendadas**

Estas son las relaciones que llevaría al modelo relacional:

| Relación                                        | Cardinalidad                               |
| ----------------------------------------------- | ------------------------------------------ |
| `VehicleType` → `Vehicle`                       | 1:N                                        |
| `Vehicle` → `TransportService`                  | 1:N                                        |
| `Driver` → `TransportService`                   | 1:N                                        |
| `BusinessPartner` → `TransportService`          | 1:N                                        |
| `Location` → `TransportService` (origen)        | 1:N                                        |
| `Location` → `TransportService` (destino)       | 1:N                                        |
| `TransportService` → `CargoManifest`            | 1:0..1 inicialmente                        |
| `TransportService` ↔ `Invoice`                  | N:M mediante `InvoiceServiceAllocation`    |
| `Invoice` → `PaymentAllocation`                 | 1:N                                        |
| `Payment` → `PaymentAllocation`                 | 1:N                                        |
| `Vehicle` → `FinancialTransaction`              | 1:N                                        |
| `Vehicle` → `VehicleSettlement`                 | 1:N                                        |
| `VehicleSettlement` → `VehicleSettlementDetail` | 1:N                                        |
| `VehicleSettlement` → `DriverSettlement`        | 1:N o 1:1 según la política de liquidación |
| `VehicleSettlement` → `ProfitDistribution`      | 1:N                                        |

También recomiendo conservar el historial de asignaciones de conductores para que un cambio de conductor no modifique los viajes anteriores.

# **6. Inventario final de entidades**

Modelo recomendado: 17 entidades principales

| #  | Entidad                    | Prioridad                      |
| -- | -------------------------- | ------------------------------ |
| 1  | `BusinessPartner`          | Esencial                       |
| 2  | `Driver`                   | Esencial                       |
| 3  | `Partner`                  | Según participación societaria |
| 4  | `Vehicle`                  | Esencial                       |
| 5  | `VehicleType`              | Esencial                       |
| 6  | `VehicleDriverAssignment`  | Esencial                       |
| 7  | `Location`                 | Esencial                       |
| 8  | `TransportMovementType`    | Esencial                       |
| 9  | `TransportService`         | Esencial                       |
| 10 | `CargoManifest`            | Si se gestiona documentalmente |
| 11 | `Invoice`                  | Esencial                       |
| 12 | `InvoiceDetail`            | Esencial                       |
| 13 | `InvoiceServiceAllocation` | Esencial                       |
| 14 | `Payment`                  | Esencial                       |
| 15 | `PaymentAllocation`        | Esencial                       |
| 16 | `FinancialConcept`         | Esencial                       |
| 17 | `FinancialTransaction`     | Esencial                       |
| 18 | `VehicleSettlement`        | Esencial                       |
| 19 | `VehicleSettlementDetail`  | Esencial                       |
| 20 | `DriverSettlement`         | Esencial                       |
| 21 | `ProfitDistribution`       | Si se distribuyen utilidades   |

La lista tiene 21 entidades contando los componentes financieros y de liquidación. No todas necesitan implementarse desde el primer sprint.

# **7. Decisiones importantes antes de desarrollar**

Hay cuatro reglas que dejaría establecidas en el dominio:

1. **La rentabilidad se calcula por vehículo y período.** No se debe calcular solamente a partir de las facturas, porque existen gastos y movimientos que no pertenecen a una factura.
2. **Las operaciones y las facturas son independientes.** Una factura puede agrupar varios viajes, pero cada viaje conserva sus propios datos.
3. **Los anticipos no son gastos automáticamente.** Deben controlarse como entregas de dinero y posteriormente conciliarse contra los gastos reales o saldos correspondientes.
4. **Las liquidaciones deben conservar su historial.** Una liquidación cerrada no debe cambiar retroactivamente cuando se modifica un viaje o un gasto.
