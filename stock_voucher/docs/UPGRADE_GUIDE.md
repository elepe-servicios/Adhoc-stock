# Guía de Actualización - stock_voucher v19.0

## Prerequisitos

### Software
- Odoo 19.0 instalado y funcionando
- Python 3.10 o superior
- PostgreSQL 13 o superior

### Módulos Dependientes
Antes de actualizar `stock_voucher`, asegurarse de tener actualizados:
- ✅ `sale_stock` (módulo core de Odoo)
- ✅ `stock_ux` (del repositorio Adhoc-stock)

## Proceso de Actualización

### Paso 1: Backup

```bash
# Backup de la base de datos
pg_dump -U odoo -d nombre_base_datos > backup_pre_upgrade_$(date +%Y%m%d_%H%M%S).sql

# Backup del directorio de addons (opcional pero recomendado)
tar -czf addons_backup_$(date +%Y%m%d_%H%M%S).tar.gz /path/to/odoo/addons
```

### Paso 2: Actualizar el Código

```bash
cd /path/to/Adhoc-stock
git fetch origin
git checkout 19.0
git pull origin 19.0
```

### Paso 3: Detener Odoo

```bash
# Si usa systemd
sudo systemctl stop odoo

# O si está ejecutando manualmente
pkill -f odoo-bin
```

### Paso 4: Actualizar el Módulo

```bash
# Modo de desarrollo (recomendado para primera actualización)
./odoo-bin -d nombre_base_datos -u stock_voucher --stop-after-init

# O en modo normal
./odoo-bin -d nombre_base_datos -u stock_voucher
```

### Paso 5: Verificar el Log

Buscar en el log:
- ✅ "Module stock_voucher: updated"
- ✅ "Loading stock_voucher: success"
- ❌ No debe haber errores SQL
- ❌ No debe haber tracebacks de Python

```bash
tail -f /var/log/odoo/odoo.log | grep -i "stock_voucher\|error\|warning"
```

### Paso 6: Reiniciar Odoo

```bash
sudo systemctl start odoo
```

### Paso 7: Pruebas Post-Actualización

#### 7.1 Verificar Instalación del Módulo

1. Acceder a Odoo como administrador
2. Ir a **Aplicaciones**
3. Buscar "Stock Voucher"
4. Verificar que aparezca como instalado
5. Verificar que la versión sea `19.0.1.0.0`

#### 7.2 Probar Funcionalidad Básica

```sql
-- Verificar que la constraint se haya recreado correctamente
SELECT conname, pg_get_constraintdef(oid) 
FROM pg_constraint 
WHERE conrelid = 'stock_picking_voucher'::regclass;
```

Debería mostrar la constraint `unique(name, book_id)`.

#### 7.3 Prueba Funcional Rápida

1. **Crear un Libro de Comprobantes**
   - Ir a: Inventario → Configuración → Books
   - Crear nuevo libro
   - Verificar que se cree correctamente

2. **Configurar Tipo de Operación**
   - Ir a: Inventario → Configuración → Tipos de Operación
   - Seleccionar un tipo de operación de salida
   - Activar "Book Required"
   - Asignar el libro creado

3. **Crear Albarán de Prueba**
   - Crear un pedido de venta
   - Generar albarán
   - Validar el albarán
   - Verificar que se asignen números de comprobante automáticamente

4. **Verificar Impresión**
   - Imprimir el comprobante
   - Verificar que el PDF se genere correctamente

## Resolución de Problemas

### Error: "La columna 'xxx' no existe"
```bash
# Esto no debería suceder, pero si ocurre:
# 1. Restaurar backup
# 2. Verificar que todas las dependencias estén actualizadas
# 3. Actualizar primero las dependencias
./odoo-bin -d nombre_base_datos -u sale_stock,stock_ux
# 4. Luego actualizar stock_voucher
./odoo-bin -d nombre_base_datos -u stock_voucher
```

### Error: "Constraint violation"
```bash
# Si hay duplicados en números de comprobante:
# 1. Identificar los duplicados
SELECT name, book_id, COUNT(*) 
FROM stock_picking_voucher 
GROUP BY name, book_id 
HAVING COUNT(*) > 1;

# 2. Corregir manualmente si es necesario
# 3. Volver a ejecutar la actualización
```

### Módulo no se actualiza
```bash
# Forzar actualización completa
./odoo-bin -d nombre_base_datos -u stock_voucher -i stock_voucher --stop-after-init
```

### Problemas de permisos
```sql
-- Verificar permisos de acceso
SELECT * FROM ir_model_access WHERE model_id IN (
    SELECT id FROM ir_model WHERE model LIKE '%stock%voucher%'
);

-- Recargar si es necesario
UPDATE ir_model_access SET active = TRUE WHERE model_id IN (
    SELECT id FROM ir_model WHERE model LIKE '%stock%voucher%'
);
```

## Rollback (En caso de problemas graves)

### Opción 1: Restaurar desde Backup

```bash
# Detener Odoo
sudo systemctl stop odoo

# Restaurar base de datos
psql -U odoo -d postgres -c "DROP DATABASE nombre_base_datos;"
psql -U odoo -d postgres -c "CREATE DATABASE nombre_base_datos OWNER odoo;"
psql -U odoo -d nombre_base_datos < backup_pre_upgrade_YYYYMMDD_HHMMSS.sql

# Restaurar código anterior
cd /path/to/Adhoc-stock
git checkout 18.0

# Reiniciar Odoo
sudo systemctl start odoo
```

### Opción 2: Desinstalar y Reinstalar

```bash
# Solo si la actualización no afectó datos críticos
./odoo-bin -d nombre_base_datos -u stock_voucher --uninstall
./odoo-bin -d nombre_base_datos -i stock_voucher
```

## Checklist de Verificación

### Pre-actualización
- [ ] Backup de base de datos realizado
- [ ] Backup de código realizado
- [ ] Dependencias actualizadas
- [ ] Entorno de prueba disponible (recomendado)
- [ ] Notificación a usuarios sobre mantenimiento

### Durante actualización
- [ ] Odoo detenido correctamente
- [ ] Código actualizado desde repositorio
- [ ] Comando de actualización ejecutado
- [ ] Log revisado sin errores

### Post-actualización
- [ ] Módulo aparece como instalado
- [ ] Versión correcta (19.0.1.0.0)
- [ ] Constraint SQL verificada
- [ ] Libros de comprobantes accesibles
- [ ] Albaranes pueden crearse y validarse
- [ ] Números de comprobante se asignan correctamente
- [ ] Impresión funciona correctamente
- [ ] Tests unitarios pasados (opcional)
- [ ] Usuarios notificados de finalización

## Monitoreo Post-Actualización

Durante las primeras 48 horas después de la actualización:

### Revisar logs regularmente
```bash
# Buscar errores relacionados
grep -i "stock.voucher\|stock_voucher" /var/log/odoo/odoo.log | grep -i error
```

### Consultas SQL de monitoreo
```sql
-- Verificar cantidad de vouchers creados post-upgrade
SELECT DATE(create_date), COUNT(*) 
FROM stock_picking_voucher 
WHERE create_date > NOW() - INTERVAL '2 days'
GROUP BY DATE(create_date);

-- Verificar pickings sin voucher (cuando debería tener)
SELECT p.name, p.state, pt.book_required 
FROM stock_picking p 
JOIN stock_picking_type pt ON p.picking_type_id = pt.id 
WHERE pt.book_required = TRUE 
  AND p.state = 'done'
  AND NOT EXISTS (SELECT 1 FROM stock_picking_voucher WHERE picking_id = p.id)
  AND p.create_date > NOW() - INTERVAL '2 days';
```

## Contacto de Soporte

Si encuentra problemas durante la actualización:

1. **Revisar la documentación**: Consultar [MIGRATION_V19.md](MIGRATION_V19.md)
2. **Buscar en GitHub Issues**: https://github.com/ingadhoc/stock/issues
3. **Crear nuevo issue**: Incluir logs, versión de Odoo, y pasos para reproducir
4. **Contacto directo**: https://www.adhoc.com.ar

## Notas Importantes

⚠️ **IMPORTANTE**: Esta actualización es de bajo riesgo pero siempre se recomienda:
1. Probar primero en un entorno de staging/desarrollo
2. Realizar la actualización fuera de horario laboral
3. Tener el backup a mano
4. Informar a los usuarios con anticipación

✅ **VENTAJAS**: 
- Mejoras de rendimiento del ORM de Odoo 19
- Nueva API de constraints más mantenible
- Compatibilidad con futuras versiones

---

**Última actualización**: Diciembre 2024  
**Versión del documento**: 1.0  
**Autor**: ADHOC SA
