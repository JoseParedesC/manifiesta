# Transport ERP — Prototipo MVP: funcionalidades implementadas

**Archivo:** `transport-erp-mvp.html` (HTML + JS vanilla, un solo archivo, sin dependencias).
**Base:** `architecture.md` y el SDD (secciones 01–05, `conclusion final.md`), en especial §2.5 (alcance MVP) y §2.6 (exclusiones).
**Naturaleza:** prototipo de interfaz y reglas de negocio. Los datos viven en memoria y en `localStorage` del navegador; no hay backend .NET/PostgreSQL.

## 1. Identity & Tenants
- 4 empresas (tenants) con datos completamente aislados; selector de empresa en el encabezado.
- 3 usuarios demo con membresías por empresa y rol distinto en cada una.
- RBAC por módulo: `admin` (todo), `operador` (terceros, flota, operaciones), `contador` (facturación, finanzas, liquidaciones, reportes). Los módulos sin permiso se ocultan y se bloquean.
- Control de cupos: alta de membresías limitada por el cupo de la empresa (ej. 3/3 en Transportes Andina).
- Alta y baja de membresías (el usuario no puede quitarse a sí mismo).
- Auditoría: registro de las últimas acciones con fecha, usuario y descripción.

## 2. Business Partners
- CRUD de terceros con rol cliente, proveedor o ambos.
- Validación de NIT único dentro de la empresa.

## 3. Fleet
- CRUD de vehículos: placa (única por empresa), tipo, marca, capacidad, conductor asignado y estado.
- Acuerdo de distribución de utilidades configurable por vehículo (ej. `Empresa:50,Socio A:50`); debe sumar 100 %.
- Alta de conductores.
- Historial de mantenimiento; cada mantenimiento genera automáticamente un gasto directo del vehículo.

## 4. Operations
- Registro de servicios de transporte con fecha, vehículo, conductor, cliente, tipo de movimiento (TRAS, EXPO, IMPO, VACIO), ruta, manifiesto y valor del flete.
- Seguimiento de estado: Abierto, En curso, Completado, Anulado.
- Vínculo visible entre servicio y factura.

## 5. Billing
- Emisión de facturas desde servicios completados sin facturar, con numeración interna consecutiva (`FV-0001`).
- Registro de facturas recibidas de proveedores con adjunto básico (nombre de archivo).
- Documento descargable por factura (HTML imprimible, sin validez DIAN).
- Recaudo (emitidas) y pago (recibidas) parcial o total con validación de saldo.

## 6. Finance
- Movimientos: ingresos, gastos (directos por vehículo o generales de la empresa) y anticipos a conductor.
- Validaciones: gasto directo exige vehículo; anticipo exige conductor; valores positivos.
- Recaudos y pagos generados automáticamente al aplicarse a una factura.
- Cuentas por cobrar y cuentas por pagar con saldo y vencimiento.

## 7. Settlements
- Liquidación mensual por vehículo: ingresos, gastos directos, gastos generales asignados (proporcional a los ingresos del mes) y utilidad.
- Distribución de utilidad según el acuerdo configurado del vehículo.
- Cierre de período: guarda una fotografía financiera y bloquea servicios, gastos y mantenimientos del vehículo en ese mes.
- Reversión de cierre con justificación obligatoria, registrada en auditoría.
- Liquidación de conductores: anticipos frente a gastos justificados y saldo.

## 8. Reporting
- Rentabilidad por vehículo por período, con exportación a CSV.
- Ingresos por cliente.
- Cartera vencida.
- Tablero de inicio con servicios del mes, facturado, CxC, CxP, utilidad consolidada y vehículos activos.

## 9. Reglas transversales cubiertas
- Aislamiento por empresa en todas las consultas.
- Dos niveles de rentabilidad: individual por vehículo y consolidada.
- Valores calculados separados de valores pagados (`amount` frente a `paid`).
- Histórico cerrado inmutable salvo reversión documentada.
- Diseño responsive con modo claro y oscuro.

## 10. Fuera de alcance (según §2.6 del SDD)
DIAN y facturación electrónica, contabilidad de partida doble, nómina legal, GPS, app móvil, inventario de repuestos y mantenimiento preventivo.

## 11. Pendiente frente a la arquitectura objetivo
- Backend ASP.NET Core, EF Core, PostgreSQL, JWT y Row Level Security: el prototipo simula estas capas en el cliente.
- Concurrencia real y transacciones de base de datos.
- Autenticación real (el prototipo solo cambia de usuario con un selector).
- Adjuntos reales y generación de PDF.
- Redondeo e impuestos o retenciones (no modelados).
- Pruebas automatizadas.

## 12. Cómo probarlo
1. Entrar como **Ana (Admin)** en *Transportes Andina*; revisar el tablero y la liquidación de `2026-11`.
2. Cambiar a **Beto (Operador)**: solo ve Terceros, Flota y Operaciones.
3. Completar un servicio, emitirle factura y recaudarla en **Facturación**.
4. Cerrar el período de un vehículo en **Liquidaciones** e intentar registrar un gasto en ese mes (queda bloqueado).
5. Reversar el cierre con justificación y revisar la **Auditoría** en *Empresa*.
6. Cambiar a *Logística Bahía* y comprobar que los datos están aislados.
