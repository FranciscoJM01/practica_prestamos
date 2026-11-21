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