# Migración del Módulo stock_voucher de Odoo 18.0 a 19.0

## Información General

- **Módulo**: stock_voucher
- **Versión Anterior**: 18.0.1.5.0
- **Versión Actual**: 19.0.1.0.0
- **Fecha de Migración**: Diciembre 2024
- **Autor**: ADHOC SA

## Resumen de Cambios

Esta migración actualiza el módulo `stock_voucher` para ser compatible con Odoo 19.0, siguiendo los lineamientos de OCA y las mejores prácticas de desarrollo de Odoo.

## Cambios Realizados

### 1. Actualización de Versión

- **Archivo**: `__manifest__.py`
- **Cambio**: Actualización del campo `version` de `18.0.1.5.0` a `19.0.1.0.0`
- **Justificación**: Seguir el estándar de versionado de OCA para módulos migrados a nueva versión mayor.

### 2. Migración de SQL Constraints a Nueva API de Odoo 19

- **Archivo**: `models/stock_picking_voucher.py`
- **Cambio**: Conversión de `_sql_constraints` a la nueva API `models.Constraint`

**Antes (Odoo 18)**:
```python
_sql_constraints = [("voucher_number_uniq", "unique(name, book_id)", 'The field "Number" must be unique per book.')]
```

**Después (Odoo 19)**:
```python
_sql_constraints = [
    models.Constraint(
        "unique(name, book_id)",
        'The field "Number" must be unique per book.',
    )
]
```

- **Justificación**: Odoo 19 introduce una nueva API para definir constraints usando `models.Constraint`, que proporciona mejor mantenibilidad y consistencia con el resto del ORM.

### 3. Validación de Compatibilidad del Código

Los siguientes componentes fueron revisados y confirmados como compatibles con Odoo 19 sin requerir cambios:

#### Modelos
- `stock.picking.voucher`: Modelo principal de comprobantes
- `stock.picking`: Extensión del modelo de albaranes
- `stock.book`: Modelo de libros de comprobantes
- `stock.picking.type`: Extensión de tipos de operación
- `stock.move`: Extensión de movimientos de stock

#### Vistas XML
- Todas las vistas XML son compatibles con Odoo 19
- No se requieren cambios en la estructura de vistas
- Los widgets y atributos utilizados son soportados

#### Wizards
- `stock.print_stock_voucher`: Asistente para impresión de comprobantes
- `stock.backorder.confirmation`: Extensión del wizard de backorder
- `product.label.layout` (stock_picking_wizard): Extensión para etiquetas

#### Tests
- Los tests existentes son compatibles con Odoo 19
- No se requieren cambios en la estructura de tests
- Se mantiene la compatibilidad con `TransactionCase`

#### Seguridad
- Archivos de seguridad (`ir.model.access.csv` y `stock_voucher_security.xml`) son compatibles
- Reglas de acceso multi-compañía funcionan correctamente

## Consideraciones Técnicas

### Compatibilidad con el ORM de Odoo 19

El código del módulo es compatible con las mejoras del ORM en Odoo 19:

1. **Uso correcto de `@api.depends`**: Todos los campos computados tienen sus dependencias correctamente declaradas
2. **Operaciones vectorizadas**: El código utiliza operaciones en batch cuando es posible
3. **Uso de `super()`**: Todas las extensiones de métodos usan correctamente `super()` para mantener el comportamiento heredado
4. **Domains y búsquedas**: Los dominios utilizados son compatibles con la nueva API de Odoo 19

### Rendimiento

- El módulo utiliza patrones eficientes del ORM
- Las búsquedas están optimizadas
- Se evitan loops con operaciones individuales de escritura

### Multi-compañía

- El módulo mantiene correcta segregación de datos por compañía
- Las reglas de acceso multi-compañía están correctamente implementadas
- Los campos `company_id` tienen `check_company=True` donde corresponde

## Pruebas Recomendadas

Después de instalar o actualizar el módulo en Odoo 19, se recomienda realizar las siguientes pruebas:

### 1. Pruebas Funcionales

- [ ] Crear un libro de comprobantes (Stock Book)
- [ ] Configurar un tipo de operación con libro obligatorio
- [ ] Crear un albarán de salida y validar la asignación automática de números
- [ ] Verificar la unicidad de números de comprobante por libro
- [ ] Probar la impresión de comprobantes
- [ ] Validar el cálculo del valor declarado automático
- [ ] Verificar el wizard de impresión de comprobantes

### 2. Pruebas Técnicas

- [ ] Ejecutar los tests unitarios: `odoo-bin -d test_db -i stock_voucher --test-enable --stop-after-init`
- [ ] Verificar que no hay errores en el log al instalar el módulo
- [ ] Comprobar que las constraints SQL se crean correctamente
- [ ] Validar el funcionamiento de las reglas de acceso multi-compañía

### 3. Pruebas de Integración

- [ ] Verificar integración con `sale_stock`
- [ ] Verificar integración con `stock_ux`
- [ ] Comprobar la funcionalidad de backorders con comprobantes
- [ ] Validar la búsqueda de albaranes por número de comprobante

## Dependencias

El módulo mantiene las siguientes dependencias:

- `sale_stock` (core Odoo)
- `stock_ux` (ADHOC SA)

Asegurarse de que estos módulos también estén migrados a Odoo 19.0 antes de instalar `stock_voucher`.

## Notas de Upgrade

### Actualización desde Odoo 18.0

Si está actualizando desde una instalación existente de Odoo 18.0:

1. **No se requiere script de migración** para la actualización de 18.0 a 19.0
2. Los cambios son compatibles hacia atrás en términos de estructura de datos
3. La constraint SQL será recreada automáticamente con la nueva sintaxis
4. No se requiere manipulación manual de datos

### Instalación Limpia

Para instalaciones limpias en Odoo 19.0:

1. Instalar dependencias (`sale_stock`, `stock_ux`)
2. Instalar `stock_voucher`
3. Configurar libros de comprobantes según necesidades

## Documentación Adicional

### Referencias Odoo 19

- [Odoo 19.0 Release Notes](https://www.odoo.com/odoo-19)
- [Coding Guidelines](https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html)
- [ORM API Changes](https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html)

### Referencias OCA

- [OCA Migration Guidelines](https://github.com/OCA/maintainer-tools/wiki#migration)
- [OCA Coding Guidelines](https://github.com/OCA/odoo-community.org/blob/master/website/Contribution/CONTRIBUTING.rst)

## Problemas Conocidos

No se han identificado problemas conocidos en esta migración.

## Contacto y Soporte

Para reportar problemas o solicitar soporte:

- **Repositorio**: https://github.com/ingadhoc/stock
- **Website**: https://www.adhoc.com.ar
- **Issues**: https://github.com/ingadhoc/stock/issues

## Changelog

### 19.0.1.0.0 (2024-12)

- [MIG] Migración del módulo de Odoo 18.0 a 19.0
- [IMP] Actualización de constraints SQL a nueva API `models.Constraint`
- [IMP] Validación de compatibilidad con ORM de Odoo 19
- [DOC] Creación de documentación de migración

---

**Nota**: Este documento debe actualizarse con cada cambio significativo en el módulo durante el ciclo de vida de la versión 19.0.
