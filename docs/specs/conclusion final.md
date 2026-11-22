Hay una decisión importante: **cada vehículo debe funcionar como una unidad de negocio independiente, pero la empresa también debe tener una visión financiera consolidada.**

Esto significa que tendremos dos niveles de rentabilidad:

1. **Rentabilidad individual:** cuánto produce y cuánto cuesta mantener cada vehículo.
2. **Rentabilidad empresarial:** cuánto gana o pierde la empresa considerando todos los vehículos y los gastos generales.

Además, como indicaste que el destino de las utilidades depende de otros acuerdos, no vamos a asumir que todos los vehículos distribuyen el 50 % a un socio. Esa regla debe ser configurable.

He contrastado estas decisiones con las tres hojas del Excel. Hay algunos conceptos que merecen especial atención: los anticipos al conductor, los anticipos de clientes, los préstamos, los gastos generales, las facturas de proveedores y los pagos a socios.

# **1. Arquitectura definitiva del dominio**

Organizaría el sistema en seis módulos:

## **ERP de transporte**

**Terceros y personal**

Clientes, proveedores, conductores y beneficiarios

**Flota**

Vehículos y asignaciones

**Operaciones**

Servicios, manifiestos y viajes

**Facturación y cartera**

Facturas, cuentas y recaudos

**Finanzas**

Gastos, ingresos, anticipos y préstamos

**Liquidaciones**

Rentabilidad y distribución de utilidades

# **2. Entidades definitivas**

Voy a separar las entidades esenciales de las que pueden incorporarse posteriormente.

## **A. Terceros y personal**

| Entidad              | Nombre técnico      | Función                                                           |
| -------------------- | ------------------- | ----------------------------------------------------------------- |
| Tercero              | `BusinessPartner`   | Clientes, proveedores y demás organizaciones o personas externas. |
| Conductor            | `Driver`            | Información laboral y personal de los conductores.                |
| Socio / beneficiario | `ProfitBeneficiary` | Personas o entidades que pueden recibir utilidades.               |

Un mismo tercero puede tener varios roles. Por ejemplo, una empresa puede ser cliente y proveedor.

No necesitas entidades separadas para generador de carga, transportista, remitente y destinatario si no tienes procesos que justifiquen su independencia.

## **B. Flota**

| Entidad                 | Nombre técnico            | Función                                   |
| ----------------------- | ------------------------- | ----------------------------------------- |
| Vehículo                | `Vehicle`                 | Placa, marca, modelo, capacidad y estado. |
| Tipo de vehículo        | `VehicleType`             | Clasificación de vehículos.               |
| Asignación de conductor | `VehicleDriverAssignment` | Historial de conductores asignados.       |

Como la flota es propia, el vehículo pertenece a la empresa. No necesitas `VehicleOwner` en esta primera versión.

## **C. Operaciones**

| Entidad                | Nombre técnico          | Función                                     |
| ---------------------- | ----------------------- | ------------------------------------------- |
| Servicio de transporte | `TransportService`      | Cada operación o viaje realizado.           |
| Tipo de movimiento     | `TransportMovementType` | TRAS, EXPO, IMPO, VACIO.                    |
| Ubicación              | `Location`              | Origen y destino.                           |
| Manifiesto             | `CargoManifest`         | Número y datos documentales del manifiesto. |

El servicio debe tener relaciones con el vehículo, conductor, cliente, origen y destino.

## **D. Facturación y cartera**

| Entidad                     | Nombre técnico             | Función                                   |
| --------------------------- | -------------------------- | ----------------------------------------- |
| Factura                     | `Invoice`                  | Facturas emitidas y recibidas.            |
| Detalle de factura          | `InvoiceDetail`            | Conceptos facturados.                     |
| Asociación factura-servicio | `InvoiceServiceAllocation` | Servicios incluidos en una factura.       |
| Pago                        | `Payment`                  | Dinero recibido o desembolsado.           |
| Aplicación de pago          | `PaymentAllocation`        | Distribución de pagos entre obligaciones. |

Una factura puede agrupar varios servicios y un servicio puede tener varias facturas parciales si el negocio lo permite.

## **E. Finanzas**

| Entidad                | Nombre técnico         | Función                                                       |
| ---------------------- | ---------------------- | ------------------------------------------------------------- |
| Concepto financiero    | `FinancialConcept`     | Catálogo de ingresos, gastos y descuentos.                    |
| Movimiento financiero  | `FinancialTransaction` | Registro económico de cada movimiento.                        |
| Cuenta financiera      | `FinancialAccount`     | Bancos, caja y otros medios de disponibilidad de dinero.      |
| Préstamo               | `Loan`                 | Obligaciones y préstamos a conductores u otros beneficiarios. |
| Aplicación de anticipo | `AdvanceSettlement`    | Conciliación de anticipos contra gastos u obligaciones.       |

Aquí introduzco dos entidades adicionales respecto a la propuesta anterior:

- `FinancialAccount`: porque el Excel registra transferencias y pagos que deben poder rastrearse por cuenta.
- `Loan`: porque un préstamo no es simplemente un gasto; genera una obligación pendiente de recuperar.

## **F. Liquidaciones**

| Entidad                          | Nombre técnico             | Función                                               |
| -------------------------------- | -------------------------- | ----------------------------------------------------- |
| Liquidación mensual del vehículo | `VehicleSettlement`        | Resultado económico del vehículo en un período.       |
| Detalle de liquidación           | `VehicleSettlementDetail`  | Conceptos que componen el resultado.                  |
| Liquidación del conductor        | `DriverSettlement`         | Salario, anticipos, descuentos y saldo.               |
| Distribución de utilidades       | `ProfitDistribution`       | Registro de utilidades asignadas.                     |
| Beneficiario de distribución     | `ProfitDistributionDetail` | Participación económica de cada beneficiario.         |
| Asignación de gastos generales   | `ExpenseAllocation`        | Distribución de gastos empresariales entre vehículos. |

Esta última entidad es fundamental por tu respuesta de que los gastos son mixtos.

# **3. La regla financiera más importante: gastos individuales y generales**

Tu respuesta de gastos mixtos determina que debemos distinguir tres tipos de costos.

| Tipo                 | Ejemplo del Excel                                            | Responsable         |
| -------------------- | ------------------------------------------------------------ | ------------------- |
| Directo del vehículo | ACPM, peajes, llantas, seguro de la mula                     | Vehículo específico |
| Directo del servicio | Cargue, descargue, gastos de un viaje                        | Servicio específico |
| General de empresa   | Administración, gastos administrativos y bancarios generales | Empresa             |

La regla debe ser:

- Los gastos directos de un servicio se asignan al servicio y, por consecuencia, al vehículo.
- Los gastos directos del vehículo se asignan directamente a este.
- Los gastos generales permanecen inicialmente en la empresa.
- Si un gasto general debe repartirse, se registra su distribución mediante `ExpenseAllocation`.

**No todos los gastos generales deben distribuirse obligatoriamente.** Debes poder decidir si se mantienen a nivel empresarial o se asignan a uno o varios vehículos.

### **Ejemplo**

Supongamos que tenemos tres vehículos:

**Gastos mensuales**

Vehículo STT126

**$2.000.000**

Vehículo STT138

**$1.500.000**

Vehículo STT549

**$1.800.000**

**Gastos directos**\
**$5.300.000**

Administración general

**$900.000**

Otros gastos empresariales

**$300.000**

**Gastos generales**\
**$1.200.000**

**Gastos totales empresa**

# **$6.500.000**

Si decides distribuir los $1.200.000 entre los vehículos, el sistema debe registrar cuánto se asignó a cada uno y con qué criterio.

Los criterios pueden ser:

- Porcentaje fijo.
- Proporcional a los ingresos.
- Proporcional a la cantidad de viajes.
- Valor manual.

Así puedes consultar tanto la rentabilidad individual antes de gastos generales como la rentabilidad después de asignarlos.

# **4. Reglas de negocio financieras**

Estas son las reglas que establecería como parte del dominio.

## **Regla 1. Rentabilidad por vehículo**

Cada vehículo tendrá una liquidación mensual única por período.

La fórmula conceptual será:

\\[ U_v = I_v - G_v \\]

Donde:

- \\(U_v\\): utilidad operativa del vehículo.
- \\(I_v\\): ingresos generados.
- \\(G_v\\): gastos y costos imputables al vehículo.

Y tendremos dos resultados:

\\[ U\_{\text{antes}} = I_v - G\_{\text{directos}} \\]

\\[ U\_{\text{después}} = U\_{\text{antes}} - G\_{\text{generales asignados}} \\]

Esto evita que un vehículo parezca menos rentable solamente porque absorbió gastos de toda la empresa.

## **Regla 2. Anticipos de clientes**

En la hoja de operaciones aparecen anticipos de empresa, fechas de anticipos y saldos.

Un anticipo recibido no debe registrarse como un ingreso adicional si ya se reconoció el valor completo del servicio.

Ejemplo:

| Concepto          | Valor      |
| ----------------- | ---------- |
| Flete             | $2.000.000 |
| Retenciones       | $37.120    |
| Anticipo recibido | $1.000.000 |
| Saldo pendiente   | $962.880   |

El anticipo reduce la cuenta por cobrar, no el ingreso generado por el servicio.

## **Regla 3. Anticipos a conductores**

Los anticipos al conductor deben manejarse como valores entregados pendientes de legalización o conciliación.

Ejemplo:

| Concepto           | Valor    |
| ------------------ | -------- |
| Anticipo entregado | $550.000 |
| ACPM               | $370.965 |
| Peajes             | $156.800 |
| Cargue y descargue | $30.000  |
| Gastos soportados  | $557.765 |
| Diferencia         | -$7.765  |

En este ejemplo, el conductor habría gastado $7.765 más de lo recibido.

El sistema debe permitir:

- Registrar el anticipo.
- Registrar los gastos.
- Adjuntar soportes.
- Conciliar el anticipo.
- Identificar sobrantes o faltantes.
- Generar un saldo pendiente cuando corresponda.

No debes contabilizar el anticipo y posteriormente volver a contabilizar el mismo dinero como gasto sin realizar la conciliación correspondiente.

## **Regla 4. Préstamos**

Un préstamo al conductor no es un gasto operativo.

Debe tener:

- Beneficiario.
- Valor inicial.
- Fecha de desembolso.
- Saldo pendiente.
- Abonos.
- Estado.

El desembolso representa una cuenta por cobrar al beneficiario. Los abonos reducen el saldo.

## **Regla 5. Gastos de mantenimiento y activos**

Un gasto de llantas, reparación o mantenimiento se registra contra el vehículo correspondiente.

No obstante, debes distinguir entre mantenimiento ordinario y adquisición o mejora de activos, porque su tratamiento financiero puede ser diferente.

## **Regla 6. Liquidación del conductor**

La liquidación del conductor debe permanecer separada de la rentabilidad del vehículo.

Puede contener:

- Salario.
- Auxilio de transporte.
- Seguridad social.
- Dotación.
- Intereses de cesantías.
- Anticipos.
- Descuentos.
- Préstamos.
- Saldo a pagar.

Esto es especialmente importante porque un mismo conductor puede cambiar de vehículo durante el año.

## **Regla 7. Distribución de utilidades**

No se establecerá un porcentaje universal.

Cada distribución debe identificar:

- Vehículo que generó la utilidad.
- Período de origen.
- Utilidad disponible para distribuir.
- Beneficiarios.
- Porcentajes o valores asignados.
- Fecha de aprobación.
- Estado de pago.

La distribución no debe modificar la utilidad operativa del vehículo. Es una aplicación posterior del resultado económico.

Además, el modelo debe permitir que una utilidad quede retenida en la empresa y que no se distribuya.

# **5. Relaciones definitivas del modelo**

Este es el esquema relacional que llevaría a implementación.

\#chatgpt-mermaid-\_r_1ci\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_1ci\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_1ci\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_1ci\_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_1ci\_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_1ci\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_1ci\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_1ci\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_1ci\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_1ci\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_1ci\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_1ci\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_1ci\_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_1ci\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_1ci\_ p{margin:0;}#chatgpt-mermaid-\_r_1ci\_ .entityBox{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_1ci\_ .relationshipLabelBox{fill:rgb(48, 48, 48);opacity:0.7;background-color:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_1ci\_ .relationshipLabelBox rect{opacity:0.5;}#chatgpt-mermaid-\_r_1ci\_ .labelBkg{background-color:rgba(48, 48, 48, 0.5);}#chatgpt-mermaid-\_r_1ci\_ .edgeLabel{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_1ci\_ .edgeLabel .label rect{fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_1ci\_ .edgeLabel .label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_1ci\_ .edgeLabel .label{fill:rgb(31, 78, 148);font-size:14px;}#chatgpt-mermaid-\_r_1ci\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_1ci\_ .edge-pattern-dashed{stroke-dasharray:8,8;}#chatgpt-mermaid-\_r_1ci\_ .node rect,#chatgpt-mermaid-\_r_1ci\_ .node circle,#chatgpt-mermaid-\_r_1ci\_ .node ellipse,#chatgpt-mermaid-\_r_1ci\_ .node polygon{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_1ci\_ .relationshipLine{stroke:rgb(175, 175, 175);stroke-width:1px;fill:none;}#chatgpt-mermaid-\_r_1ci\_ .marker{fill:none!important;stroke:rgb(175, 175, 175)!important;stroke-width:1;}#chatgpt-mermaid-\_r_1ci\_ [data-look=neo].labelBkg{background-color:rgba(48, 48, 48, 0.5);}#chatgpt-mermaid-\_r_1ci\_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_1ci\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_1ci\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_1ci\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_1ci\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_1ci\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_1ci\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_1ci\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}BUSINESS_PARTNERTRANSPORT_SERVICEVEHICLE_TYPEVEHICLEVEHICLE_DRIVER_ASSIGNMENTDRIVERTRANSPORT_MOVEMENT_TYPELOCATIONCARGO_MANIFESTINVOICEINVOICE_DETAILINVOICE_SERVICE_ALLOCATIONPAYMENT_ALLOCATIONPAYMENTFINANCIAL_CONCEPTFINANCIAL_TRANSACTIONFINANCIAL_ACCOUNTLOANVEHICLE_SETTLEMENTVEHICLE_SETTLEMENT_DETAILDRIVER_SETTLEMENTPROFIT_DISTRIBUTIONPROFIT_DISTRIBUTION_DETAILPROFIT_BENEFICIARYEXPENSE_ALLOCATIONcontractsclassifieshasassignedperformsdrivesclassifiesorigindestinationdocumentscontainsallocatesbilledreceivesappliesclassifiesincursgeneratesrelatesprocessestrackssettlescontainsincludesallocatesdistributesreceivesdistributesreceives

Hay una consideración: las relaciones de los movimientos financieros con vehículos y servicios son opcionales, porque no todos los movimientos pertenecen a un viaje o vehículo.

También implementaría las relaciones con conductores y terceros cuando corresponda.

# **6. Modelo de datos: campos principales**

Estas serían las propiedades mínimas para que el dominio sea implementable.

| Entidad                    | Campos importantes                                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `BusinessPartner`          | Id, identificación, tipo de identificación, razón social, nombre comercial, teléfono, dirección, estado.     |
| `Driver`                   | Id, identificación, nombres, apellidos, teléfono, fecha de ingreso, estado.                                  |
| `ProfitBeneficiary`        | Id, tipo de beneficiario, identificación, nombre, datos de pago, estado.                                     |
| `Vehicle`                  | Id, placa, marca, modelo, año, capacidad, tipo de vehículo, estado.                                          |
| `VehicleDriverAssignment`  | VehicleId, DriverId, fecha inicio, fecha fin.                                                                |
| `TransportService`         | Fecha, VehicleId, DriverId, CustomerId, OriginId, DestinationId, MovementTypeId, flete, retenciones, estado. |
| `CargoManifest`            | TransportServiceId, número, fecha, referencia documental.                                                    |
| `Invoice`                  | Número, tipo, tercero, fecha emisión, vencimiento, subtotal, impuestos, total, estado.                       |
| `InvoiceDetail`            | InvoiceId, concepto, cantidad, precio, impuesto, subtotal.                                                   |
| `InvoiceServiceAllocation` | InvoiceId, TransportServiceId, valor asignado.                                                               |
| `Payment`                  | Tercero, fecha, valor, cuenta financiera, medio de pago, referencia.                                         |
| `PaymentAllocation`        | PaymentId, InvoiceId, valor aplicado.                                                                        |
| `FinancialConcept`         | Código, nombre, categoría, naturaleza, requiere vehículo, requiere soporte.                                  |
| `FinancialTransaction`     | Fecha, concepto, valor, vehículo, servicio, conductor, tercero, estado, referencia.                          |
| `FinancialAccount`         | Nombre, tipo, banco, número de cuenta, estado.                                                               |
| `Loan`                     | Beneficiario, valor inicial, fecha, saldo, estado.                                                           |
| `VehicleSettlement`        | VehicleId, período, ingresos, costos directos, gastos asignados, utilidad, estado.                           |
| `VehicleSettlementDetail`  | SettlementId, concepto, origen, valor, tipo.                                                                 |
| `DriverSettlement`         | DriverId, SettlementId, salario, descuentos, anticipos, saldo.                                               |
| `ProfitDistribution`       | SettlementId, utilidad disponible, fecha, estado.                                                            |
| `ProfitDistributionDetail` | DistributionId, BeneficiaryId, porcentaje, valor asignado, valor pagado.                                     |
| `ExpenseAllocation`        | FinancialTransactionId, VehicleId, criterio, porcentaje, valor asignado.                                     |

`AdvanceSettlement` debe implementarse como una entidad de conciliación o un mecanismo equivalente de aplicación de anticipos. No debe quedar reducido a un campo dentro de `FinancialTransaction`.

# **7. Decisiones técnicas que recomiendo**

Para tu implementación con .NET, PostgreSQL y arquitectura Clean Architecture, establecería estas restricciones:

| Regla                   | Restricción                                                        |
| ----------------------- | ------------------------------------------------------------------ |
| Liquidación mensual     | Unicidad de `VehicleId + Year + Month`.                            |
| Placa                   | Única entre vehículos activos y registros históricos.              |
| Asignación de conductor | Evitar asignaciones simultáneas incompatibles.                     |
| Factura                 | No permitir que su valor aplicado supere su saldo.                 |
| Pago                    | No permitir aplicaciones superiores al valor disponible.           |
| Anticipo                | No permitir conciliaciones superiores al saldo pendiente.          |
| Distribución            | No permitir asignaciones superiores a la utilidad disponible.      |
| Liquidación cerrada     | No permitir modificaciones directas.                               |
| Gastos generales        | La suma asignada no puede superar el valor distribuible del gasto. |

Las liquidaciones deberían tener estados:

`DRAFT → REVIEWED → APPROVED → CLOSED`

Y las transacciones financieras deberían poder tener estados como:

`PENDING → CONFIRMED → RECONCILED`

No mezclaría el estado de aprobación de una operación con su estado de pago.

# **8. Qué construir primero**

No implementaría los 22 componentes del dominio al mismo tiempo. Este sería el orden:

1. **Fundación**

   `BusinessPartner`, `Driver`, `Vehicle`, `VehicleType`, `Location`, `VehicleDriverAssignment`.
2. **Operación**

   `TransportService`, `TransportMovementType`, `CargoManifest`.
3. **Finanzas operativas**

   `FinancialConcept`, `FinancialTransaction`, `FinancialAccount`, `Loan` y conciliación de anticipos.
4. **Facturación y cartera**

   `Invoice`, `InvoiceDetail`, `InvoiceServiceAllocation`, `Payment`, `PaymentAllocation`.
5. **Liquidaciones**

   `VehicleSettlement`, `VehicleSettlementDetail`, `DriverSettlement`.
6. **Rentabilidad empresarial**

   `ExpenseAllocation`, `ProfitBeneficiary`, `ProfitDistribution`, `ProfitDistributionDetail`.

# **9. Qué NO incluiría todavía**

Para mantener el proyecto enfocado en la operación real, dejaría fuera:

| Entidad              | Motivo                                                                 |
| -------------------- | ---------------------------------------------------------------------- |
| `Carrier`            | La empresa opera su propia flota.                                      |
| `Shipper`            | Se representa mediante `BusinessPartner`.                              |
| `Consignee`          | Puede ser un dato adicional del servicio si se necesita.               |
| `Trailer`            | No hay evidencia de administración independiente de remolques.         |
| `Route`              | Origen y destino son suficientes inicialmente.                         |
| `FuelPurchase`       | Se maneja mediante movimientos financieros.                            |
| `TollPayment`        | Se maneja mediante movimientos financieros.                            |
| `AccountsReceivable` | Se puede calcular desde facturas y aplicaciones de pagos inicialmente. |
| `AccountsPayable`    | Se puede calcular desde facturas recibidas y pagos.                    |
| `GeneralLedger`      | No estás construyendo todavía un sistema de contabilidad general.      |

Una aclaración: el modelo propuesto controla cartera operativa, obligaciones y rentabilidad, pero no sustituye un sistema de contabilidad financiera con plan de cuentas, comprobantes contables y asientos de doble partida.

# **10. Dos definiciones pendientes que no debemos inventar**

Hay dos aspectos que tu respuesta todavía deja abiertos y que deben resolverse con reglas configurables.

**A. Distribución de utilidades**

El Excel muestra una distribución del 50 % a un socio para el caso de STT126, pero no debemos generalizarla a toda la flota.

La entidad `ProfitDistributionDetail` permitirá manejar:

- Un beneficiario con participación fija.
- Varios beneficiarios con porcentajes diferentes.
- Distribuciones por valores específicos.
- Utilidades retenidas sin distribuir.
- Acuerdos diferentes por vehículo y período.

**B. Criterio de gastos generales**

Debemos permitir asignación manual y automática. No asumiría que todos los vehículos absorben los gastos por igual.

Esto puede evolucionar a políticas de distribución configurables.

# **11. Resultado final**

**Modelo de dominio propuesto**

# **26**

Entidades y componentes identificados

# **6**

Módulos funcionales

# **1**

Liquidación mensual por vehículo

# **2**

Niveles de rentabilidad

**Principios fundamentales**

- Cada vehículo es una unidad de negocio.
- Cada servicio tiene trazabilidad operativa y financiera.
- Los anticipos no se confunden con ingresos o gastos.
- Los gastos generales pueden distribuirse sin perder su origen.
- Las facturas y los servicios son independientes.
- Las utilidades se distribuyen según acuerdos configurables.
- La empresa conserva su rentabilidad consolidada.

**Mi recomendación arquitectónica:** utiliza `VehicleSettlement` como el agregado principal del cierre financiero mensual, pero no conviertas la liquidación en el origen de los movimientos. Los movimientos financieros deben existir independientemente y la liquidación debe consolidarlos mediante referencias trazables.

Con esto tienes una base suficientemente sólida para comenzar el diseño de entidades EF Core, relaciones PostgreSQL, servicios de dominio y reglas de negocio sin sobredimensionar el ERP.