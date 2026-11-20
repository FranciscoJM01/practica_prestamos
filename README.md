# Práctica: Préstamo de Equipo en Odoo (`prestamo_unicah`)
- **Nombre:** Francisco Javier Martínez
- **Número de cuenta:** TU_NUMERO_DE_CUENTA

## Respuestas
### Paso 1. Catálogo de equipos
1. **¿Qué pasa con un equipo cuando se desmarca el campo Activo? ¿Por qué es mejor que borrarlo?**
   Al desmarcar `active`, Odoo oculta el registro de las búsquedas y listas normales (archivado lógico), pero conserva el registro en la base de datos. Es mejor que borrarlo porque mantiene intacto el historial de préstamos anteriores donde participó ese equipo y evita errores de integridad referencial.
2. **¿Para qué sirve `copy=False` en el campo `codigo`?**
   Evita que al duplicar un registro de equipo desde la interfaz (`Acción > Duplicar`), Odoo copie el código de inventario anterior, obligando a asignar un código único al nuevo equipo.