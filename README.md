# Práctica: Préstamo de Equipo en Odoo (`prestamo_unicah`)
- **Nombre:** Francisco Javier Martínez
- **Número de cuenta:** TU_NUMERO_DE_CUENTA

## Respuestas
### Paso 1. Catálogo de equipos
1. **¿Qué pasa con un equipo cuando se desmarca el campo Activo? ¿Por qué es mejor que borrarlo?**
   Al desmarcar `active`, Odoo oculta el registro de las búsquedas y listas normales (archivado lógico), pero conserva el registro en la base de datos. Es mejor que borrarlo porque mantiene intacto el historial de préstamos anteriores donde participó ese equipo y evita errores de integridad referencial.
2. **¿Para qué sirve `copy=False` en el campo `codigo`?**
   Evita que al duplicar un registro de equipo desde la interfaz (`Acción > Duplicar`), Odoo copie el código de inventario anterior, obligando a asignar un código único al nuevo equipo.
### Paso 2. Registro de préstamos
1. **Este modelo usa `_name` y `_inherit` al mismo tiempo. ¿Qué hace cada uno aquí?**
   `_name = "prestamo.prestamo"` crea un modelo y una tabla nueva en PostgreSQL, mientras que `_inherit = ["mail.thread"]` incorpora de forma abstracta (herencia de mixin) los campos, métodos y funcionalidades del chatter (seguidores, mensajes y registro de auditoría) sin modificar el modelo original `mail.thread`.
2. **¿Qué hace `tracking=True` en el campo `state`?**
   Registra automáticamente en el chatter cada cambio de valor del estado del préstamo, indicando el usuario, la fecha/hora y el estado anterior y nuevo.
### Paso 3. Equipos del préstamo
1. **En `fields.One2many("prestamo.linea", "prestamo_id")`, ¿qué indica cada parámetro?**
   El primer parámetro (`"prestamo.linea"`) indica el modelo hijo al que apunta la relación, y el segundo (`"prestamo_id"`) indica el campo `Many2one` en el modelo hijo que almacena la referencia hacia el registro padre (`prestamo.prestamo`).
2. **¿Qué pasa con las líneas cuando se borra un préstamo? ¿Qué parte del código lo define?**
   Se eliminan automáticamente todas las líneas asociadas a ese préstamo. Esto lo define el atributo `ondelete="cascade"` en el campo `prestamo_id` del modelo `prestamo.linea`.
### Paso 4. Flujo de estados
1. **Los botones ya se ocultan según el estado. ¿Por qué los métodos vuelven a validar el estado?**
   Porque ocultar un botón en XML solo restringe la interfaz visual; un usuario o llamada externa vía RPC/API podría invocar directamente el método Python. Validar el estado en el servidor garantiza la integridad de la lógica de negocio sin importar desde dónde se ejecute.
2. **¿Qué hace `prestamo.linea_ids.equipo_id.write(...)`? ¿Por qué no hace falta un `for` para cada equipo?**
   Recorre la relación en memoria para obtener un *recordset* con todos los equipos de las líneas del préstamo y ejecuta una actualización masiva (`UPDATE`) sobre todos ellos en una sola operación ORM, ya que en Odoo los recordsets son colecciones vectorizadas.
### Paso 5. Reglas del préstamo
1. **El campo `equipo_id` ya filtra los equipos disponibles. ¿Por qué hace falta el TODO 5?**
   Porque el filtro (`domain`) solo actúa al momento de seleccionar el equipo en el formulario. Si el préstamo queda en `Borrador` y mientras tanto otro usuario presta ese equipo o lo envía a mantenimiento, al pulsar `Prestar` más tarde el equipo ya no estaría disponible; el TODO 5 evita prestar un equipo ocupado.
2. **¿En qué momento se ejecuta un método con `@api.constrains`? ¿Y la validación de `action_prestar`?**
   `@api.constrains` se ejecuta automáticamente cada vez que se crea o modifica y guarda un registro en la base de datos (`create` / `write`). La validación dentro de `action_prestar` se ejecuta únicamente cuando se invoca dicha acción (por ejemplo, al hacer clic en el botón **Prestar**).
### Paso 6. Contacto, kanban y búsqueda
1. **¿Qué pasa si arrastra una tarjeta del kanban a otra columna? ¿Por qué?**
   La tarjeta se mueve de columna y actualiza automáticamente el campo `estado` del equipo en la base de datos, porque la vista `<kanban>` tiene configurado `default_group_by="estado"` sobre un campo `Selection` editable.
2. **El campo `prestamo_ids` se agregó a `res.partner`. ¿Se crea una columna nueva en la tabla de contactos? Explique.**
   No se crea ninguna columna en `res_partner`. Los campos `One2many` son relaciones inversas virtuales que consultan dinámicamente la clave foránea `partner_id` almacenada en la tabla `prestamo_prestamo`.