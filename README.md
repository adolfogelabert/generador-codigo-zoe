# ZOE — Generador de Códigos de Producto y EAN-13

Sistema web para la generación de nomenclatura ZOE y códigos de barras EAN-13, con inventario en Firestore y gestión de liberación de códigos.

**URL Producción:** https://adolfo-gelabert.github.io/generador-codigo-zoe/  
**URL Local:** http://localhost/generador-codigo-zoe/

---

## Módulos Principales

### 1. Generador Único
Genera códigos ZOE uno por uno con su respectivo código de barras EAN-13.

| Campo | Descripción |
|-------|-------------|
| **Prefijo ZOE** | ZCA (Accesorios Auto/Hogar), ZCE (Herrajes), o personalizado (3 letras) |
| **Identificador** | 4 dígitos únicos (generador aleatorio incluido) |
| **Sufijo** | 2 letras de característica (TP=Tope, AF=Pie 8", HH=Manilla, etc.) |
| **EAN-13** | Prefijo empresa (7451304) + secuencial (5 dígitos) + dígito verificador |
| **Descripción** | Nombre del producto en catálogo/Profit |
| **Color/Acabado** | Satin Nickel, Negro, Acero Inox, etc. |
| **Medida/Uso** | Modo Medida (MM/pulgadas), Tipo de uso, o personalizado |

**Funciones:**
- Validador ZOE en vivo (detecta duplicados al instante)
- Asignación automática de código de barras libre
- Búsqueda de producto existente (sesión, inventario, catálogo, Firestore)
- Descarga SVG/PNG del código de barras
- Impresión de etiqueta formato ZOE (hoja horizontal)
- Auto-incremento del secuencial EAN tras cada registro

### 2. Generador por Lotes
Genera múltiples códigos ZOE correlativos con EAN-13 secuenciales.

| Campo | Descripción |
|-------|-------------|
| Prefijo Base | ZCA, ZCE o ZCO |
| Sufijo Base | 2 letras para todo el lote |
| Secuencial EAN inicio | Número inicial del rango (5 dígitos) |
| Identificador inicio | ID ZOE inicial (4 dígitos) |
| Cantidad | 1-100 códigos por lote |

### 3. Inventario de Códigos de Barras
Gestión completa del inventario EAN-13 con filtros y búsqueda.

- **Filtros:** Por prefijo, estado (Libres, Asignados, Liberados) o texto libre
- **Acciones por código:**
  - **Libres (SIN_UTILIZAR):** "Usar" — carga en el formulario
  - **Liberados (LIBERADO):** "Usar" + "Restaurar" a EN_USO
  - **Asignados (EN_USO):** "Ver" detalle + "Liberar" para reutilizar
- **Ordenamiento:** Por EAN-13, ZOE o categoría

### 4. Buscador de Empaques / Diseños
Catálogo de productos diseñados (manillas, cerraduras, masterpacks).

| Filtro | Opciones |
|--------|----------|
| Tipo de producto | Manillas, Cerraduras, Masterpacks |
| Nivel de empaque | Con sub-empaque, Sin sub-empaque, Solo Masterpack |
| Material/Color | Acero/Inox, Negro, Blanco |
| Forma | Redonda, Cuadrada, Ovalada, Placa |
| Búsqueda libre | Por ciudad, nombre, código ZOE o material |

### 5. Asignador de Producto
Workflow de 5 pasos para asignar código a un producto del catálogo:

1. **Buscar producto** por nombre en el catálogo de empaques
2. **Ver datos** auto-generados (nombre, ciudad, material, forma)
3. **Asignar código ZOE** (prefijo + ident + sufijo)
4. **Asignar código de barras** (auto o manual)
5. **Registrar producto completo** en Firestore

### 6. Generador de Reportes
4 tipos de reportes con exportación CSV e impresión:

| Reporte | Contenido |
|---------|-----------|
| **Inventario General** | Resumen de códigos totales, en uso, libres, con/sin ZOE |
| **Códigos ZOE** | Todas las nomenclaturas generadas con estado de asignación |
| **Códigos de Barras** | EAN-13 con secuenciales, rangos, huecos y porcentaje de uso |
| **Liberaciones** | Historial de liberaciones con acciones masivas |

---

## Sistema de Liberación de Códigos

Permite liberar códigos de barras EN_USO para reutilizarlos en nuevos productos.

| Acción | Descripción |
|--------|-------------|
| **Liberar individual** | Libera un código específico (requiere motivo) |
| **Por categoría** | Libera todos los EN_USO de una categoría (ej: "Cerraduras") |
| **Por prefijo ZOE** | Libera todos los EN_USO con un prefijo (ej: ZCE) |
| **Seleccionados** | Libera múltiples códigos seleccionados con checkbox |
| **Liberar TODOS** | Acción destructiva — libera todos los EN_USO (doble confirmación) |
| **Restaurar** | Devuelve un LIBERADO a estado EN_USO |

**Persistencia:** Historial en `localStorage` + Firestore colección `liberaciones`

---

## Base de Datos Firestore

| Colección | ID del documento | Campos |
|-----------|-----------------|--------|
| `inventory` | `{EAN-13}` | bc, zce, cat, estado, carpeta |
| `registros` | Auto | code, ean, prefix, ident, suffix, desc, color, size, fecha |
| `liberaciones` | Auto | bc, zce, cat, carpeta, motivo, fecha, fechaTs |

**Flujo de carga:**
1. Al iniciar, carga desde Firestore
2. Si Firestore está vacío → sube los 97 items del fallback embebido
3. Si hay error de red → usa el array fallback embebido

---

## Archivos del Proyecto

| Archivo | Descripción |
|---------|-------------|
| `index.html` | Aplicación completa (HTML + CSS + JS, ~3,500 líneas) |
| `catalogo-empaques.json` | Catálogo de empaques/diseños (37 productos) |
| `_inventario.json` | Datos de inventario (copia de referencia) |
| `INVENTARIO_CODIGOS_BARRAS.csv` | Export CSV del inventario |
| `images/favicon.png` | Icono de la aplicación |

---

## Dependencias (CDN)

| Librería | Versión | Uso |
|----------|---------|-----|
| Tailwind CSS | CDN | Diseño responsive |
| FontAwesome | 6.4.0 | Iconos |
| JsBarcode | 3.11.5 | Generación de EAN-13 |
| Firebase | 11.1.0 | Firestore (base de datos) |
| Inter (Google Fonts) | 300-800 | Tipografía |

---

## Uso

1. Abrir `index.html` en navegador o acceder a GitHub Pages
2. La primera vez, se suben los 97 códigos de barras a Firestore automáticamente
3. Seleccionar prefijo ZOE → ingresar identificador + sufijo → asignar código de barras → registrar
4. Los registros se sincronizan con Firestore en tiempo real
5. Usar los filtros de Inventario y Reportes para gestionar el catálogo
