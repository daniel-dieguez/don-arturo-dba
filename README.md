## Base de datos

El proyecto utiliza **SQL Server** y está compuesto por:

| Elemento | Cantidad |
|----------|----------|
| Tablas   | 28 |
| Procedimientos almacenados (SP) | 24 |

Toda la lógica de acceso a datos vive en
**procedimientos almacenados**. Cada SP recibe el parámetro `opciones`,
que indica qué acción ejecutar (consultar todos, uno, crear, actualizar o cambiar estado). 
Así un solo SP por módulo cubre todo el CRUD y el backend solo llama al SP correspondiente desde el controller.
### Estandarización de los procedimientos almacenados

La mayoría de los SP siguen la misma estructura de operaciones, seleccionadas mediante el parámetro `opciones`:

| Opción | Operación | Descripción |
|--------|-----------|-------------|
| 1 | `create` | Inserta un nuevo registro. |
| 2 | `update` | Actualiza los datos de un registro existente. |
| 3 | `estado` | Cambia el estado de un registro (baja lógica). |
| 4 | `all` | Consulta todos los registros. |
| 5 | `one` | Consulta un registro específico, normalmente por el ID principal de la tabla. |

**Consideraciones importantes:**

- **Baja lógica:** en algunos módulos, como el carrito, la operación se presenta como un `delete`, pero en realidad cambia el estado a `0`. Así los datos nunca se eliminan físicamente y se conserva el historial.
- **Consultas `all`:** se utiliza un `SELECT *` por el momento pero ya cuando se trabaje en front Se seleccionan solo las columnas que el front end necesita, lo que reduce la cantidad de datos transferidos y mejora el rendimiento.
- **Consultas `one`:** filtran por el identificador principal de la tabla, devolviendo un único registro.

**Ventajas de esta estandarización:**

- Facilita saber qué agregar o quitar durante el mantenimiento o al implementar nuevos requerimientos.
- Mantiene un orden consistente entre todos los módulos.
- Permite que el model y el controller del backend reutilicen la misma lógica para cualquier módulo.

> Algunos SP, como `SP_Modulo_Login` y `SP_Modulo_Dashboard`, pueden tener opciones distintas por la naturaleza de su función.

### Columnas de auditoría

La mayoría de las tablas incluyen las siguientes columnas de auditoría:

| Columna | Descripción |
|---------|-------------|
| `creado` | Fecha y hora en que se creó el registro. |
| `creadoPor` | ID de la persona que creó el registro. |
| `actualizado` | Fecha y hora de la última actualización. |
| `actualizadoPor` | ID de la persona que realizó la última actualización. |

Con estas columnas es posible saber **cuándo** se creó o modificó un registro y **quién** lo hizo, ya que `creadoPor` y `actualizadoPor` almacenan el ID de la persona responsable. Esto permite llevar un registro de los movimientos y facilita la trazabilidad y la revisión de cambios.

### Tablas por módulo - pequeña intruducción.

**Administración (ADM)**

| Módulo | Tablas |
|--------|--------|
| Usuarios | `FU_Modulo_Usuario`, `FU_Modulo_Persona` |
| Roles y tipos de usuario | `FU_Modulo_Roles`, `FU_Modulo_TipoUsuario` |
| Ubicación | `FU_Modulo_Departamneto`, `FU_Modulo_Municipios` |
| Métricas | `FU_Modulo_Mes` |

**Productos y catálogo (PROD)**

| Módulo | Tablas |
|--------|--------|
| Productos | `FU_Modulo_Productos`, `FU_Modulo_Catalogo_Productos` |
| Variantes y atributos | `FU_Modulo_Variantes`, `FU_Modulo_Atributos`, `FU_Modulo_Valores_Atributos`, `FU_Modulo_Variante_Atributos` |
| Inventario | `FU_Modulo_Inventario`, `FU_Modulo_Kardex` |

**Carrito y compras**

| Módulo | Tablas |
|--------|--------|
| Carrito | `FU_Modulo_Carrito`, `FU_Modulo_Detalle_Carrito` |
| Compras | `FU_Modulo_Compras`, `FU_Modulo_Detalle_Compra` |
| Bitácoras | `FU_Modulo_Bitacora_Compra`, `FU_Modulo_Bitacora_Transacciones_Compra` |
| Pagos | `FU_Modulo_Pagos` |
| Reservas | `FU_Modulo_Reservas`, `FU_Modulo_Detalle_Reservas` |
| Idempotencia | `FU_Modulo_Idempotencia` |

**Cupones**

| Módulo | Tablas |
|--------|--------|
| Cupones | `FU_Modulo_Cupones`, `FU_Modulo_Catalogo_Cupones`, `FU_Modulo_Uso_Cupones` |

### Procedimientos almacenados

| Módulo | Stored Procedure |
|--------|------------------|
| Login | `SP_Modulo_Login` |
| Usuarios | `SP_Modulo_Persona` |
| Roles | `SP_Modulo_Roles` |
| Tipo de usuario | `SP_Modulo_TipoUsuario` |
| Ubicación | `SP_Modulo_PAIS` |
| Dashboard / métricas | `SP_Modulo_Dashboard` |
| Productos | `SP_Modulo_Productos` |
| Catálogo de productos | `SP_Modulo_Catalogo_Productos` |
| Variantes | `SP_Modulo_Variantes` |
| Atributos | `SP_Modulo_Atributos` |
| Valores de atributos | `SP_Modulo_Valores_Atributos` |
| Atributos por variante | `SP_Modulo_Variante_Atributos` |
| Inventario | `SP_Modulo_Inventario` |
| Carrito | `SP_Modulo_Carrito` |
| Detalle de carrito | `SP_Modulo_Detalle_Carrito` |
| Compras | `SP_Modulo_Compras` |
| Detalle de compra | `SP_Modulo_Detalle_Compra` |
| Pagos | `SP_Modulo_Pagos` |
| Reservas | `SP_Modulo_Reservas` |
| Detalle de reservas | `SP_Modulo_Detalle_Reservas` |
| Idempotencia | `SP_Modulo_Idempotencia` |
| Cupones | `SP_Modulo_Cupones` |
| Catálogo de cupones | `SP_Modulo_Catalogo_Cupones` |
| Uso de cupones | `SP_Modulo_Uso_Cupones` |

### Convenciones

- **Tablas:** prefijo `FU_Modulo_` seguido del nombre del módulo.
- **Procedimientos:** prefijo `SP_Modulo_` seguido del nombre del módulo.
- **Un SP por módulo**, con el parámetro `opciones` para elegir la acción.
- **Baja lógica:** los registros no se eliminan físicamente, se cambia su estado.
- **Idempotencia:** `FU_Modulo_Idempotencia` evita procesar dos veces la misma operación (por ejemplo, un pago repetido).
