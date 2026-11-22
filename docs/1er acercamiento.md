## Primer Acercamiento

### Excel con operaciones de los carros (hojas) -> categorizado por placa
1. Control de manifiesto -> Informacion detallada del viaje (nro de manifiesto, origen, destivo, vlr flete)
2. Cartera -> Informacion de movimientos (impuestos, gastos, fletes)
3. Producido del carro en el viaje 



### Requisito funcional
- Llenar la informacion de la cartera y tenerla disponible para analisis y reportes
- Al ingresar el la placa, debe aparecer el producido (impuestos, gastos, fletes) para dar el producido mensual 
- El registro de mulas debe tener el socio principal (quien recibe el pago), y socios asicionales (con quien reparten un %)
- Cartera por empresa, por placa y por socio



### Matriz Decisiones tecnicas


## **1. Entidades comerciales y participantes**

| Entidad (ES)             | Nombre técnico (EN) | Descripción                                                                                     |
| ------------------------ | ------------------- | ----------------------------------------------------------------------------------------------- |
| Generador de carga       | `Shipper`           | Empresa que genera o contrata la necesidad de transporte.                                       |
| Transportista            | `Carrier`           | Empresa habilitada que presta el servicio de transporte.                                        |
| Cliente                  | `Customer`          | Empresa o persona que mantiene una relación comercial con la transportadora.                    |
| Remitente                | `Consignor`         | Persona o empresa que remite o entrega la mercancía.                                            |
| Destinatario             | `Consignee`         | Persona o empresa que recibe la mercancía.                                                      |
| Propietario del vehículo | `VehicleOwner`      | Persona o empresa propietaria del vehículo.                                                     |
| Conductor                | `Driver`            | Persona encargada de conducir el vehículo.                                                      |
| Tenedor del vehículo     | `VehicleHolder`     | Persona que tiene la tenencia o explotación del vehículo sin ser necesariamente su propietario. |
| Intermediario            | `Broker`            | Tercero que intermedia en la contratación del transporte.                                       |

## **2. Entidades de vehículos y recursos**

| Entidad (ES)            | Nombre técnico (EN)    | Descripción                                                       |
| ----------------------- | ---------------------- | ----------------------------------------------------------------- |
| Vehículo                | `Vehicle`              | Mula, camión, automóvil, motocicleta, etc.                        |
| Semirremolque           | `SemiTrailer`          | Remolque acoplado a un tractocamión.                              |
| Remolque                | `Trailer`              | Unidad remolcada por un vehículo.                                 |
| Conjunto vehicular      | `VehicleCombination`   | Combinación de tractocamión y semirremolque.                      |
| Tipo de vehículo        | `VehicleType`          | Clasificación: tractocamión, camión, camioneta, motocicleta, etc. |
| Marca                   | `VehicleBrand`         | Fabricante del vehículo.                                          |
| Modelo                  | `VehicleModel`         | Modelo comercial del vehículo.                                    |
| Configuración vehicular | `VehicleConfiguration` | Configuración de ejes y características técnicas.                 |

## **3. Entidades operativas y logísticas**

| Entidad (ES)            | Nombre técnico (EN)   | Descripción                                       |
| ----------------------- | --------------------- | ------------------------------------------------- |
| Operación de transporte | `TransportOperation`  | Operación general de transporte de mercancía.     |
| Manifiesto de carga     | `CargoManifest`       | Documento que ampara el transporte de mercancías. |
| Remesa terrestre        | `TransportRemittance` | Documento asociado a la mercancía transportada.   |
| Viaje                   | `Trip`                | Desplazamiento realizado por un vehículo.         |
| Ruta                    | `Route`               | Trayecto entre origen y destino.                  |
| Origen                  | `Origin`              | Punto de inicio del transporte.                   |
| Destino                 | `Destination`         | Punto final del transporte.                       |
| Mercancía               | `Goods`               | Bienes objeto del transporte.                     |
| Tipo de mercancía       | `GoodsType`           | Clasificación de la mercancía.                    |
| Carga                   | `Cargo`               | Conjunto de mercancías transportadas.             |
| Punto de carga          | `LoadingPoint`        | Lugar donde se carga la mercancía.                |
| Punto de descarga       | `UnloadingPoint`      | Lugar donde se descarga la mercancía.             |

## **4. Entidades financieras y de cartera**

| Entidad (ES)          | Nombre técnico (EN)    | Descripción                                       |
| --------------------- | ---------------------- | ------------------------------------------------- |
| Cuenta por cobrar     | `AccountsReceivable`   | Dinero pendiente de recaudo de clientes.          |
| Cuenta por pagar      | `AccountsPayable`      | Obligaciones pendientes de pago a terceros.       |
| Factura               | `Invoice`              | Documento de cobro por servicios prestados.       |
| Pago                  | `Payment`              | Registro de un pago recibido o realizado.         |
| Abono                 | `PaymentAllocation`    | Aplicación de un pago a una obligación.           |
| Anticipo              | `AdvancePayment`       | Dinero entregado antes de completar la operación. |
| Liquidación de viaje  | `TripSettlement`       | Liquidación económica de un viaje.                |
| Concepto de cobro     | `ChargeConcept`        | Concepto por el cual se cobra un valor.           |
| Concepto de descuento | `DeductionConcept`     | Concepto por el cual se descuenta un valor.       |
| Movimiento financiero | `FinancialTransaction` | Registro de un movimiento económico.              |

## **5. Entidades geográficas y administrativas**

| Entidad (ES)    | Nombre técnico (EN) |
| --------------- | ------------------- |
| País            | `Country`           |
| Departamento    | `Department`        |
| Municipio       | `Municipality`      |
| Ciudad          | `City`              |
| Dirección       | `Address`           |
| Sucursal        | `Branch`            |
| Sede            | `Office`            |
| Banco           | `Bank`              |
| Cuenta bancaria | `BankAccount`       |

## **6. Relaciones principales del dominio**

Este sería un modelo conceptual simplificado:

**Shipper**

Generador de carga

Contrata el servicio

**Carrier**

Transportista

Asigna recursos y ejecuta

**Vehicle**

Vehículo

**Driver**

Conductor

**CargoManifest**

Manifiesto de carga

**Trip**

Ejecución del viaje

**FinancialTransaction**

Cartera, pagos y liquidaciones

## **7. Mi recomendación de arquitectura**

Para evitar problemas futuros en tu ERP, tendría en cuenta tres decisiones:

1. **Company y Person:** Una entidad `Party` puede representar personas y organizaciones, con roles como `Shipper`, `Carrier` o `Consignee`.
2. **Vehicle y VehicleOwner:** nunca mezclaría el propietario con el vehículo. Un vehículo puede cambiar de propietario sin perder su historial operativo.
3. **CargoManifest y TransportRemittance:** son documentos diferentes en el contexto del transporte terrestre colombiano. No conviene tratarlos como sinónimos.
