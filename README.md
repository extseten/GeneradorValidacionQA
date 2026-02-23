# Generador de Queries - Documentación Técnica Completa

## Aplicación Web para Validación de Calidad de Datos

| Atributo | Valor |
|----------|-------|
| **Versión** | 3.0.0 |
| **Archivo** | `docs/web/generador_query_qa_v3.html` |
| **Fecha** | 2026-02-20 |
| **Autor** | Sergio Tena |
| **Tecnología** | HTML5 / CSS3 / JavaScript (Vanilla) |
| **Dependencia** | SheetJS (xlsx.js) para exportación Excel |

---

## 1. Resumen Ejecutivo

### 1.1 ¿Qué es el Generador de Queries?

El **Generador de Queries v3** es una aplicación web que automatiza la creación de queries SQL para validación de datos en **Google BigQuery**. Permite generar queries de forma rápida y estandarizada sin necesidad de escribir SQL manualmente, con soporte para tablas encriptadas (campos `BYTES`).

### 1.2 Propósito

Facilitar el proceso de validación de calidad de datos mediante:
- ✅ Generación automática de queries SQL
- ✅ Estandarización del proceso de validación
- ✅ Reducción de errores humanos
- ✅ Exportación de evidencias a Excel
- ✅ Soporte para múltiples tipos de tablas y tipos de información
- ✅ Validación de tablas encriptadas (campos BYTES vs STRING)
- ✅ Análisis automático de pipelines ETL/SQL

### 1.3 Beneficios Clave

| Beneficio | Descripción |
|-----------|-------------|
| **⚡ Rapidez** | Genera queries en segundos |
| **🎯 Precisión** | Queries estandarizados sin errores de sintaxis |
| **📊 Trazabilidad** | Exportación a Excel para evidencias |
| **🔄 Flexibilidad** | Múltiples modos, tipos de tabla y tipos de información |
| **💻 Sin Instalación** | Funciona en cualquier navegador |
| **🔒 Seguro** | No envía datos a servidores externos |
| **🔐 Encriptado** | Validación estadística de campos BYTES |

---

## 2. Arquitectura de Comboboxes (v3)

La versión 3 introduce **tres comboboxes independientes** que controlan la generación de queries:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     ESTRUCTURA DE COMBOBOXES - v3                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  COMBO 1: Tipo de Validación                                                │
│  ┌──────────────────────────────────────────────┐                           │
│  │ • Pruebas Unitarias (Data Engineer)  [ut]    │                           │
│  │ • Validación QA (Analista QA)        [qa]    │                           │
│  │ • Análisis Pipeline (SP)             [pipeline]                          │
│  └──────────────────────────────────────────────┘                           │
│                                                                             │
│  COMBO 2: Tipo de Tabla  (solo para ut / qa)                                │
│  ┌──────────────────────────────────────────────┐                           │
│  │ • Tabla Simple              [simple]         │                           │
│  │ • Tabla con RECORD          [record]         │                           │
│  │ • Tabla Diferente Estructura [different]     │                           │
│  └──────────────────────────────────────────────┘                           │
│                                                                             │
│  COMBO 3: Tipo de Información  (solo para ut / qa)                          │
│  ┌──────────────────────────────────────────────┐                           │
│  │ • En Claro                  [clear]          │                           │
│  │ • Encriptado (BYTES)        [encrypted]      │                           │
│  └──────────────────────────────────────────────┘                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Comportamiento según combinación

| Combo 1 | Combo 2 | Combo 3 | Reglas generadas |
|---------|---------|---------|-----------------|
| ut | simple | clear | UT-01, UT-02, UT-03, UT-04 |
| ut | record | clear | UT-01, UT-02, UT-03, UT-04 (con UNNEST) |
| ut | different | clear | UT-01-DIFF, UT-03-DIFF, UT-04-DIFF (mapeo manual) |
| ut | simple/record/different | encrypted | UT-E01, UT-E02, UT-E03, UT-E04 |
| qa | simple | clear | R01–R08 |
| qa | record | clear | R01–R08 (con UNNEST) |
| qa | different | clear | R01-DIFF, R07-DIFF, R08-DIFF (mapeo manual) |
| qa | simple/record/different | encrypted | RE01, RE02, RE03, RE04 |
| pipeline | — | — | PL-01, PL-02, PL-03 (+ botón especial) |

---

## 3. Modos de Validación

### 3.1 Modo 1: Pruebas Unitarias (Data Engineer)

**Objetivo:** Validar integridad básica de datos después de cada carga ETL.

#### 3.1.1 En Claro

| Regla | Nombre | Tipo | Descripción |
|-------|--------|------|-------------|
| **UT-01** | Conteo de Registros | Directo | COUNT(*) origen = COUNT(*) destino + `diferencia` + `estado` PASS/FAIL |
| **UT-02** | Valores Únicos | 2 pasos | Sin duplicados en TODOS los campos (filtro `data_type != 'BYTES'` si aplica) |
| **UT-03** | Integridad Bidireccional | 2 pasos | FULL OUTER JOIN origen ↔ destino |
| **UT-04** | Resumen de Diferencias | 2 pasos | FULL OUTER JOIN mostrando solo diferencias campo a campo |

#### 3.1.2 Encriptado (BYTES)

| Regla | Nombre | Tipo | Descripción |
|-------|--------|------|-------------|
| **UT-E01** | Conteo de Registros | Directo | COUNT(*) sin alterar por encriptación |
| **UT-E02** | Top 5 Valores Frecuentes | Meta-Query | Compara distribución STRING vs `TO_HEX(BYTES)` |
| **UT-E03** | Valores Nulos | Meta-Query | Compara nulos/vacíos STRING (`''`) vs BYTES (`b""`) |
| **UT-E04** | Top 5 Longitud de Campos | Meta-Query | Compara `OCTET_LENGTH` origen vs destino |

### 3.2 Modo 2: Validación QA (Analista QA)

**Objetivo:** Validar reglas de calidad entre ambientes ORIGEN y DESTINO.

#### 3.2.1 En Claro

| Regla | Nombre | Tipo | Descripción |
|-------|--------|------|-------------|
| **R01** | Existencia de Tabla | Directo | Tabla existe con `fecha_creacion` y `fecha_actualizacion` |
| **R02** | Cabeceras Iguales | Directo | Columnas origen = columnas destino |
| **R03** | Tipos de Datos | Directo | Tipos de datos correctos |
| **R04** | Conteo de Registros | Directo | Registros origen = destino + `diferencia` + `estado` PASS/FAIL |
| **R05** | Campos No Nulos | EXECUTE IMMEDIATE | Sin valores nulos en campos (single-step dinámico) |
| **R06** | Sin Duplicados | 2 pasos | Sin registros duplicados (incluye campos RECORD si aplica) |
| **R07** | Valores Coinciden | 2 pasos | Valores iguales por PK (con LIMIT 1000 y nota para quitar límite) |
| **R08** | Resumen de Diferencias | 2 pasos | FULL OUTER JOIN mostrando solo diferencias campo a campo |

#### 3.2.2 Encriptado (BYTES)

| Regla | Nombre | Tipo | Descripción |
|-------|--------|------|-------------|
| **RE01** | Conteo de Registros | Directo | COUNT(*) sin alterar por encriptación |
| **RE02** | Top 5 Valores Frecuentes | Meta-Query | Compara distribución STRING vs `TO_HEX(BYTES)` |
| **RE03** | Valores Nulos | Meta-Query | Compara nulos/vacíos STRING vs BYTES (`b""`) |
| **RE04** | Top 5 Longitud de Campos | Meta-Query | Compara `OCTET_LENGTH` origen vs destino |

### 3.3 Modo 3: Validación Entre Capas / Análisis de Pipeline

**Objetivo:** Validar integridad de datos entre capas del Data Lake (Raw→Master, Master→Business) analizando el código de SPs.

| Regla | Nombre | Tipo | Descripción |
|-------|--------|------|-------------|
| **PL-01** | Resumen del Pipeline | Directo | Conteo por cada paso del SP con filtros y JOINs |
| **PL-02** | Registros Perdidos | Directo | Tablas intermedias vs MASTER/BUSINESS |
| **PL-03** | Calidad MASTER/BUSINESS | Directo | Duplicados, nulos, huérfanos y conteo vs orígenes |

---

## 4. Tipos de Tabla Soportados

### 4.1 Tabla Simple

Para tablas con estructura plana sin campos anidados.

**Ejemplo de estructura:**
```sql
CREATE TABLE poliza (
    numero_poliza STRING,
    nombre_cliente STRING,
    fecha_emision DATE,
    monto_prima NUMERIC
);
```

**Características:**
- Usa `INFORMATION_SCHEMA.COLUMNS` para obtener campos
- Filtro `AND data_type != 'BYTES'` en modo "En Claro" si el checkbox "tiene campos BYTES" está activo
- Filtro `AND data_type = 'BYTES'` en modo "Encriptado" si el checkbox está activo

### 4.2 Tabla con RECORD/ARRAY

Para tablas con campos anidados (STRUCT/RECORD) en BigQuery.

**Ejemplo de estructura:**
```sql
CREATE TABLE poliza_detalle (
    numero_poliza STRING,
    datos_poliza STRUCT<
        nombre_cliente STRING,
        coberturas ARRAY<STRUCT<codigo STRING, monto NUMERIC>>
    >
);
```

**Características especiales:**
- Usa `INFORMATION_SCHEMA.COLUMN_FIELD_PATHS` para obtener campos anidados
- Aplica `UNNEST` para aplanar estructuras
- Requiere configurar la **columna RECORD** para origen y destino
- El checkbox **"Tiene campos BYTES"** también está disponible en modo "En Claro" para excluir campos BYTES

### 4.3 Tabla con Diferente Estructura

Para comparar tablas donde las columnas tienen nombres diferentes entre origen y destino.

**Ejemplo:**
| Origen | Destino |
|--------|---------|
| `moneda_id` | `id_moneda` |
| `plan_nro` | `numero_plan` |
| `fec_emision` | `fecha_emision` |

**Características especiales:**
- Requiere configurar PKs de ORIGEN y DESTINO por separado
- Genera plantillas de mapeo manual con `<<CAMPO_ORIGEN>>` y `<<CAMPO_DESTINO>>`
- Detección de sinónimos comunes (ej. `numero_` / `nro_`, `id_` / `_id`)
- UT-01 / R04 incluyen consultas DIFF para mostrar registros solo en cada lado

---

## 5. Tipo de Información: Encriptado (BYTES)

### 5.1 ¿Cuándo usarlo?

Cuando la tabla de **destino** tiene campos encriptados como tipo `BYTES` y la tabla de **origen** tiene los mismos campos como `STRING`.

### 5.2 Checkbox "Tiene campos BYTES"

Disponible en los campos **ORIGEN** y **DESTINO** para todos los tipos de tabla:

| Modo Información | Checkbox activo | Efecto en INFORMATION_SCHEMA |
|-----------------|----------------|------------------------------|
| En Claro | Sí | `AND data_type != 'BYTES'` (excluye BYTES) |
| Encriptado | Sí | `AND data_type = 'BYTES'` (solo BYTES) |

### 5.3 Meta-Queries (Reglas E02, E03, E04 / RE02, RE03, RE04)

Las reglas de tipo "Meta-Query" funcionan en **dos pasos**:

```
PASO 1: Ejecutar la meta-query en BigQuery
        → Devuelve el campo `sql_generado` con el SQL dinámico

PASO 2: Copiar el contenido de `sql_generado`
        → Pegar y ejecutar directamente en BigQuery
```

### 5.4 Uso de DECLARE para WHERE

Para evitar errores de "Unclosed string literal" en BigQuery, los filtros `WHERE` se normalizan en una sola línea y se almacenan en variables `DECLARE`:

```sql
DECLARE tabla_origen STRING DEFAULT 'proyecto.dataset.tabla';
DECLARE tabla_destino STRING DEFAULT 'proyecto.dataset.tabla';
DECLARE wh_origen STRING DEFAULT "WHERE campo = 'valor'";
DECLARE wh_destino STRING DEFAULT "WHERE campo = 'valor'";
```

---

## 6. Interfaz de Usuario

### 6.1 Estructura de la Pantalla

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 GENERADOR DE QUERIES - VALIDACIÓN DE CALIDAD DE DATOS v3                │
│     Sistema para generación automática de queries SQL para BigQuery         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SELECCIÓN DE MODO                                                          │
│  ┌──────────────────────┐ ┌──────────────────────┐ ┌─────────────────────┐ │
│  │ Tipo de Validación:  │ │ Tipo de tabla:        │ │ Tipo de información:│ │
│  │ [▼ Prueb. Unitarias] │ │ [▼ Tabla Simple]      │ │ [▼ En Claro]        │ │
│  └──────────────────────┘ └──────────────────────┘ └─────────────────────┘ │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────┐  ┌──────────────────────────────┐        │
│  │      🔷 ORIGEN               │  │      🔷 DESTINO              │        │
│  │  Proyecto: [___________]     │  │  Proyecto: [___________]     │        │
│  │  Dataset:  [___________]     │  │  Dataset:  [___________]     │        │
│  │  Tabla:    [___________]     │  │  Tabla:    [___________]     │        │
│  │  Filtro:   [___________]     │  │  Filtro:   [___________]     │        │
│  │  Join:     [___________]     │  │  Join:     [___________]     │        │
│  │  ☐ Tiene campos BYTES        │  │  ☐ Tiene campos BYTES        │        │
│  └──────────────────────────────┘  └──────────────────────────────┘        │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │  🔑 PRIMARY KEYS (separadas por coma)                             │      │
│  │  [numero_poliza, id_producto]                                     │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  [ ⚡ Generar Queries ]    [ 📊 Exportar a Excel ]                           │
│    (oculto en Pipeline)                                                     │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📋 QUERIES GENERADOS                                                       │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │ UT-01: Conteo de Registros                              [Copiar] │      │
│  │ ┌────────────────────────────────────────────────────────────┐   │      │
│  │ │ SELECT cnt_origen, cnt_destino, diferencia, estado ...     │   │      │
│  │ └────────────────────────────────────────────────────────────┘   │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Campos de Entrada

| Campo | Descripción | Obligatorio | Notas |
|-------|-------------|-------------|-------|
| **Proyecto** | ID del proyecto GCP | Sí | Para origen y destino |
| **Dataset** | Nombre del dataset | Sí | Para origen y destino |
| **Tabla** | Nombre de la tabla | Sí | Para origen y destino |
| **Filtro** | Condición WHERE opcional | No | Se normaliza a una sola línea en DECLARE |
| **Join** | Tablas adicionales para JOIN | No | Para consultas con tablas relacionadas |
| **Primary Keys** | Claves primarias | Según tipo | No requerido en modo Encriptado |
| **Columna RECORD** | Nombre del campo RECORD | Solo Tabla RECORD | Para origen y destino |
| **Tiene campos BYTES** | Checkbox filtro | No | Activa filtro `data_type` en INFORMATION_SCHEMA |

### 6.3 Estilo Visual

La aplicación sigue un estilo corporativo con:
- **🔵 Azul oscuro** (`#003366`): Header y títulos de sección
- **🔵 Azul secundario** (`#0066CC`): Bordes y selects
- **⚪ Blanco**: Fondo limpio y claro
- **🟠 Naranja** (`#FF6600`): Botones de acción principal y badges
- **🟣 Púrpura** (`#7B1FA2`): Badge de modo Encriptado

---

## 7. Flujo de Uso

### 7.1 Pruebas Unitarias / Validación QA — En Claro

```
PASO 1: CONFIGURACIÓN
═════════════════════
1.1 Abrir generador_query_qa_v3.html en navegador
1.2 Seleccionar Tipo de Validación: "Pruebas Unitarias" o "Validación QA"
1.3 Seleccionar Tipo de tabla: "Simple", "RECORD" o "Diferente Estructura"
1.4 Seleccionar Tipo de información: "En Claro"

PASO 2: INGRESO DE DATOS
════════════════════════
2.1 Completar campos de ORIGEN y DESTINO
2.2 Ingresar PKs separadas por coma
2.3 (Si es RECORD) Ingresar columna RECORD para origen y destino
2.4 (Opcional) Activar checkbox "Tiene campos BYTES" para excluirlos

PASO 3: GENERACIÓN
═══════════════════
3.1 Click en "⚡ Generar Queries"
3.2 Se generan automáticamente los queries según la combinación seleccionada

PASO 4: EJECUCIÓN (queries de 2 pasos)
═══════════════════════════════════════
4.1 Copiar Query AUXILIAR (PASO 1) → Ejecutar en BigQuery
4.2 Copiar resultado (columna `bloque` o `columnas_group`)
4.3 Pegar en Query FINAL donde dice <<PEGAR AQUÍ>>
4.4 Ejecutar Query FINAL

PASO 5: EXPORTACIÓN
═════════════════════
5.1 Click en "📊 Exportar a Excel"
5.2 Se descarga archivo con Instrucciones, Inputs y Resultados
```

### 7.2 Pruebas Unitarias / Validación QA — Encriptado (BYTES)

```
PASO 1: CONFIGURACIÓN
═════════════════════
1.1 Seleccionar Tipo de información: "Encriptado (BYTES)"
1.2 (Si es RECORD) Ingresar columna RECORD para origen y destino

PASO 2: INGRESO DE DATOS
════════════════════════
2.1 Completar campos de ORIGEN y DESTINO (sin PKs, no requeridas)
2.2 Activar checkbox "Tiene campos BYTES" para filtrar INFORMATION_SCHEMA
2.3 Filtros WHERE se normalizarán a una sola línea automáticamente

PASO 3: GENERACIÓN
═══════════════════
3.1 Click en "⚡ Generar Queries"
3.2 Se generan: UT-E01/RE01 (conteo directo) + UT-E02 a E04 / RE02 a RE04 (meta-queries)

PASO 4: META-QUERIES (2 pasos)
══════════════════════════════
4.1 Ejecutar la meta-query → obtener `sql_generado`
4.2 Copiar el contenido de `sql_generado`
4.3 Pegar y ejecutar directamente en BigQuery como nuevo query
```

### 7.3 Análisis de Pipeline (SP)

```
PASO 1: SELECCIÓN
══════════════════
1.1 Seleccionar "Análisis Pipeline (SP)" en Tipo de Validación
    → Los combos Tipo de tabla y Tipo de información se ocultan
    → El botón "⚡ Generar Queries" se oculta (no aplica en este modo)

PASO 2: INGRESO DEL SP
═══════════════════════
2.1 Pegar código SQL del SP principal en el área de texto
2.2 (Opcional) Click en "+ Agregar SP Predecesor" para analizar pipelines múltiples

PASO 3: DETECCIÓN AUTOMÁTICA
══════════════════════════════
3.1 Click en "🔍 Analizar Pipeline"
    El sistema detecta automáticamente:
    📥 Tablas RAW   (datasets con prefijo raw_*)
    ⚙️  Tablas TEMP  (dataset temp)
    📊 Tablas MASTER/BUSINESS
    🔑 PKs desde condiciones ON de los JOINs
    📋 Filtros WHERE por tabla
    🔗 Tablas de config_ (configuración)

PASO 4: VERIFICACIÓN
══════════════════════
4.1 Revisar tablas detectadas en el diagrama visual
4.2 (Opcional) Ingresar PKs manualmente si la detección no es correcta

PASO 5: GENERACIÓN
═══════════════════
5.1 Click en "🔧 Generar Queries de Validación Pipeline"
5.2 Se generan: PL-01, PL-02, PL-03

PASO 6: EXPORTACIÓN
═════════════════════
6.1 Click en "📊 Exportar a Excel"
6.2 Se descarga archivo con evidencia de los 3 queries PL
```

---

## 8. Detalle de Queries Generados

### 8.1 Queries de Pruebas Unitarias — En Claro

#### UT-01: Conteo de Registros

```sql
-- UT-01: Conteo de Registros
WITH conteos AS (
    SELECT
        (SELECT COUNT(*) FROM `proyecto.origen.tabla` WHERE ...) AS cnt_origen,
        (SELECT COUNT(*) FROM `proyecto.destino.tabla` WHERE ...) AS cnt_destino
)
SELECT
    cnt_origen  AS registros_origen,
    cnt_destino AS registros_destino,
    cnt_origen - cnt_destino AS diferencia,
    CASE WHEN cnt_origen = cnt_destino THEN 'PASS' ELSE 'FAIL' END AS estado
FROM conteos;
```
**Resultado esperado:** `diferencia = 0, estado = PASS`

#### UT-02: Valores Únicos (2 pasos)

**PASO 1 - Query Auxiliar:**
```sql
SELECT STRING_AGG(column_name, ', ') as columnas_group
FROM `proyecto.destino.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'tabla'
  AND data_type NOT IN ('ARRAY', 'STRUCT', 'RECORD', 'GEOGRAPHY', 'JSON')
  AND data_type != 'BYTES';  -- Si "Tiene campos BYTES" está activado en En Claro
```

**PASO 2 - Query Final:**
```sql
SELECT <<COLUMNAS>>, COUNT(*) as cantidad
FROM `proyecto.destino.tabla`
GROUP BY <<COLUMNAS>>
HAVING COUNT(*) > 1
ORDER BY cantidad DESC;
```
**Resultado esperado:** `0 registros`

#### UT-03: Integridad Bidireccional (2 pasos)

**PASO 1 - Query Auxiliar:** Genera bloque de columnas con CASE WHEN por cada campo común (desde `INFORMATION_SCHEMA`).

**PASO 2 - Query Final:**
```sql
WITH origen AS (SELECT * FROM `origen` WHERE ...),
     destino AS (SELECT * FROM `destino` WHERE ...)
SELECT DISTINCT
    COALESCE(CAST(A.pk AS STRING), CAST(B.pk AS STRING)) AS pk,
    <<PEGAR BLOQUE AQUÍ>>
FROM origen A
FULL OUTER JOIN destino B ON CAST(A.pk AS STRING) = CAST(B.pk AS STRING)
LIMIT 1000;
```

#### UT-04: Resumen de Diferencias (2 pasos)

Similar a UT-03 pero el Query Final solo devuelve registros donde **al menos un campo tiene diferencia** usando `CASE WHEN SUM(diff_*) > 0`.

---

### 8.2 Queries de Validación QA — En Claro

#### R01: Existencia de Tabla

```sql
SELECT 
    project_id, dataset_id, table_id,
    DATE(TIMESTAMP_MILLIS(creation_time)) as fecha_creacion,
    DATE(TIMESTAMP_MILLIS(last_modified_time)) as fecha_actualizacion,
    row_count as cantidad_registros,
    ROUND(size_bytes / (1024*1024), 2) as tamano_mb
FROM `proyecto.dataset.__TABLES__`
WHERE table_id = 'tabla';
```

#### R05: Campos No Nulos (EXECUTE IMMEDIATE — un solo paso)

```sql
EXECUTE IMMEDIATE (
    SELECT CONCAT(
        'SELECT ',
        STRING_AGG(
            CONCAT(
                '''', column_name, ''' as campo, ',
                'COUNT(*) as total, ',
                'COUNTIF(', column_name, ' IS NULL) as nulos, ',
                'CASE WHEN COUNTIF(', column_name, ' IS NULL) = 0 THEN ''PASS'' ELSE ''FAIL'' END as estado'
            ), ' UNION ALL SELECT '
        ),
        ' FROM `proyecto.dataset.tabla` WHERE ...'
    )
    FROM `proyecto.dataset.INFORMATION_SCHEMA.COLUMNS`
    WHERE table_name = 'tabla'
      AND data_type NOT IN ('ARRAY','STRUCT','RECORD','GEOGRAPHY','JSON')
);
```

#### R07: Valores Coinciden (2 pasos — nota de LIMIT)

El Query Final incluye `LIMIT 1000` y el comentario `-- si desea el total quitar el limit 1000`.

---

### 8.3 Queries Encriptados (UT-E / RE)

#### UT-E01 / RE01: Conteo de Registros

```sql
WITH conteos AS (
    SELECT
        (SELECT COUNT(*) FROM `origen` WHERE ...) AS cnt_origen,
        (SELECT COUNT(*) FROM `destino` WHERE ...) AS cnt_destino
)
SELECT cnt_origen, cnt_destino,
       cnt_origen - cnt_destino AS diferencia,
       CASE WHEN cnt_origen = cnt_destino THEN 'PASS' ELSE 'FAIL' END AS estado
FROM conteos;
```

#### UT-E02 / RE02: Top 5 Valores Frecuentes (Meta-Query)

Genera SQL que compara los 5 valores más frecuentes por campo BYTES:
- **ORIGEN (STRING):** `CAST(r.campo AS STRING) AS valor`
- **DESTINO (BYTES):** `TO_HEX(r.campo) AS valor`
- Usa `RANK() OVER (ORDER BY COUNT(*) DESC)` con `QUALIFY ranking <= 5`

#### UT-E03 / RE03: Valores Nulos (Meta-Query)

Genera SQL que compara nulos y vacíos por campo BYTES:
- **ORIGEN (STRING):** `COUNTIF(r.campo IS NULL)`, `COUNTIF(r.campo = '')`
- **DESTINO (BYTES):** `COUNTIF(r.campo IS NULL)`, `COUNTIF(r.campo = b"")`

#### UT-E04 / RE04: Top 5 Longitud de Campos (Meta-Query)

Genera SQL que compara `OCTET_LENGTH` por campo BYTES con `QUALIFY ranking <= 5`.

---

### 8.4 Queries de Validación Entre Capas (Pipeline)

#### PL-01: Resumen del Pipeline

```sql
SELECT 1 as paso,
    'tabla_raw' as tabla_origen, 'tabla_temp' as tabla_destino,
    (SELECT COUNT(*) FROM `raw.tabla`) as cnt_origen,
    (SELECT COUNT(*) FROM `temp.tabla`) as cnt_destino,
    (SELECT COUNT(*) FROM `raw.tabla`) -
    (SELECT COUNT(*) FROM `temp.tabla`) as diferencia,
    CASE WHEN diferencia = 0 THEN '✅ OK' ELSE '⚠️ REVISAR' END as estado,
    'ON condición JOIN' as joins_on,
    'WHERE condición' as filtros_where,
    'CREATE/INSERT' as transformacion
UNION ALL
...
ORDER BY paso;
```

#### PL-02: Registros Perdidos

UNION ALL de todos los pasos comparando tablas intermedias vs MASTER/BUSINESS.

#### PL-03: Calidad MASTER/BUSINESS

Incluye duplicados, nulos en PKs, conteo total vs orígenes, y huérfanos.

---

## 9. Exportación a Excel

### 9.1 Estructura del Archivo Excel

#### Hoja 1: Instrucciones

| Contenido |
|-----------|
| INSTRUCCIONES DE USO |
| Modo utilizado (UT / QA / Pipeline) |
| Tipo de tabla |
| Tipo de información (En Claro / Encriptado) |
| Pasos a seguir |
| Criterios de éxito por regla |
| Fecha de generación |

#### Hoja 2: Inputs (tabular)

| Ambiente | Proyecto | Dataset | Tabla | Filtro | Join |
|----------|----------|---------|-------|--------|------|
| ORIGEN | mi-proyecto | uat_master | poliza | fecha >= '2024-01-01' | — |
| DESTINO | mi-proyecto | prd_master | poliza | fecha >= '2024-01-01' | — |

> Para modo Encriptado o Tabla con RECORD, se agregan filas adicionales con los checkboxes "Tiene campos BYTES" y columna RECORD.

#### Hoja 3: Resultados (combinada)

| Código | Regla | Tipo | Query Auxiliar (o Meta-Query) | Query Final | Resultado Esperado | Resultado Obtenido | Estado | Fecha Ejecución Regla | Observación |
|--------|-------|------|-------------------------------|-------------|-------------------|-------------------|--------|----------------------|-------------|
| UT-01 | Conteo | Directo | N/A (directo) | SELECT... | diferencia = 0, PASS | | | | |
| UT-E01 | Conteo Encriptado | Directo | N/A (directo) | SELECT... | diferencia = 0, PASS | | | | |
| UT-E02 | Top 5 Valores | Meta-Query (2 pasos) | `-- UT-E02: ... SELECT sql_generado ...` | (PASO 2) Copiar sql_generado y ejecutar en BigQuery | Distribución comparable (STRING vs HEX) | | | | |
| UT-E03 | Valores Nulos | Meta-Query (2 pasos) | `-- UT-E03: ... SELECT sql_generado ...` | (PASO 2) Copiar sql_generado y ejecutar en BigQuery | nulos ORIGEN = nulos DESTINO | | | | |
| UT-E04 | Longitud Campos | Meta-Query (2 pasos) | `-- UT-E04: ... SELECT sql_generado ...` | (PASO 2) Copiar sql_generado y ejecutar en BigQuery | Longitudes BYTES consistentes | | | | |

> **Nota sobre Meta-Queries:** Para las reglas E02-E04 / RE02-RE04, la columna "Query Auxiliar (o Meta-Query)" contiene el SQL completo que genera el campo `sql_generado`. La columna "Query Final" contiene la instrucción para ejecutar ese resultado. No se filtra por `isAuxiliary` para estas reglas — se exportan todas.

### 9.2 Nombre del Archivo

- **Pruebas Unitarias / QA:** `Validacion_{modo}_{tipo}_{tabla}_{fecha}.xlsx`
- **Pipeline:** `Validacion_Pipeline_{tabla_master}_{fecha}.xlsx`

---

## 10. Características Técnicas

### 10.1 Tecnologías Utilizadas

| Tecnología | Versión | Uso |
|------------|---------|-----|
| **HTML5** | — | Estructura de la página |
| **CSS3** | — | Estilos y diseño responsivo con variables CSS |
| **JavaScript** | ES6+ | Lógica de generación de queries |
| **SheetJS (xlsx.js)** | 0.18.5 | Exportación a Excel |

### 10.2 Compatibilidad de Navegadores

| Navegador | Versión Mínima | Estado |
|-----------|----------------|--------|
| Chrome | 80+ | ✅ Soportado |
| Firefox | 75+ | ✅ Soportado |
| Edge | 80+ | ✅ Soportado |
| Safari | 13+ | ✅ Soportado |
| IE | — | ❌ No soportado |

### 10.3 Seguridad

- ✅ **Sin backend:** Todo se procesa en el navegador
- ✅ **Sin envío de datos:** Los datos nunca salen del navegador
- ✅ **Sin cookies:** No almacena información
- ✅ **Solo SheetJS como dependencia externa** (CDN)

---

## 11. Funciones JavaScript Principales

### 11.1 Variables Globales

```javascript
let currentMode = 'ut';           // 'ut' | 'qa' | 'pipeline'
let currentTableType = 'simple';  // 'simple' | 'record' | 'different'
let currentInfoType = 'clear';    // 'clear' | 'encrypted'
let generatedQueries = [];        // Queries generados para exportación
let generatedPipelineQueries = []; // Queries de pipeline para exportación
```

### 11.2 Funciones de Control de UI

```javascript
changeMode()           // Cambia modo; oculta/muestra combos y botones
changeTableType()      // Cambia tipo de tabla; controla campos RECORD/different
changeInfoType()       // Cambia tipo de info; controla sección encriptado y checkboxes BYTES
validateForm()         // Valida campos obligatorios según combinación activa
getInputValues()       // Recopila todos los valores del formulario (incl. checkboxes BYTES)
```

### 11.3 Funciones de Generación

```javascript
generateQueries()                  // Dispatcher: llama la función correcta según modo+tipo+info
generateUTQueries(v)               // Genera UT-01 a UT-04 (simple y record)
generateQAQueries(v)               // Genera R01 a R08 (simple y record)
generateUTQueriesDifferent(v)      // Genera UT-01-DIFF, UT-03, UT-04 (diferente estructura)
generateQAQueriesDifferent(v)      // Genera R01-DIFF, R07, R08 (diferente estructura)
generateEncryptedQueries(v)        // Genera UT-E01 a E04 / RE01 a RE04 (encriptado)
generatePipelineQueries()          // Genera PL-01, PL-02, PL-03
```

### 11.4 Función `generateEncryptedQueries` — Detalle

```javascript
function generateEncryptedQueries(v) {
    const isRecord = currentTableType === 'record';

    // WHERE normalizado a una sola línea (para DECLARE en BigQuery)
    const whOrigenSafe = whereOrigen.replace(/\s*\n\s*/g, ' ').trim();
    const whDestinoSafe = whereDestino.replace(/\s*\n\s*/g, ' ').trim();

    // DECLARE variables para evitar "Unclosed string literal"
    const declareBase = `DECLARE tabla_origen STRING DEFAULT '${tablaOrigen}';
DECLARE tabla_destino STRING DEFAULT '${tablaDestino}';
DECLARE wh_origen STRING DEFAULT "${whOrigenSafe}";
DECLARE wh_destino STRING DEFAULT "${whDestinoSafe}";`;

    // Filtros INFORMATION_SCHEMA según checkboxes
    const bytesFilterOrigen = v.origenHasBytes ? "\n    AND data_type = 'BYTES'" : '';
    const bytesFilterDestino = v.destinoHasBytes ? "\n    AND data_type = 'BYTES'" : '';

    // Prefijo de regla: UT-E (unitarias) o RE (qa)
    const rP = currentMode === 'ut' ? 'UT-E' : 'RE';

    // Genera: Conteo(E01), Top5Frecuentes(E02), Nulos(E03), Longitud(E04)
}
```

### 11.5 Funciones de Pipeline

```javascript
parseSQL(sql)                        // Extrae tablas, PKs, filtros y relaciones del SP
cleanTableName(tableName)            // Valida formato proyecto.dataset.tabla
analyzePipeline()                    // Orquesta el análisis del código SQL
displayPipelineResults()             // Muestra diagrama de tablas detectadas
addPredecessorSP()                   // Agrega textarea para SP predecesor
removePredecessorSP(button)          // Elimina SP predecesor
```

### 11.6 bytesFilter en INFORMATION_SCHEMA

En modo "En Claro" con checkbox "Tiene campos BYTES" activo:
```javascript
const bytesOpUT = currentInfoType === 'encrypted' ? "= 'BYTES'" : "!= 'BYTES'";
const bytesFilterOrigen = v.origenHasBytes ? `\n    AND data_type ${bytesOpUT}` : '';
```

Esto asegura que las consultas a `INFORMATION_SCHEMA` excluyan o incluyan campos BYTES según el contexto.

---

## 12. Solución de Problemas

### 12.1 Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| "Campos requeridos" | Falta proyecto, dataset o tabla | Completar todos los campos obligatorios |
| "Primary Keys vacías" | No se ingresaron PKs | Ingresar al menos una PK |
| "Unclosed string literal" | Filtro WHERE con saltos de línea | La app normaliza automáticamente; verificar filtro |
| "Trailing comma after WITH" | Error en CTE generado | Reportar — ya fue corregido en v3 |
| Excel no se descarga | Bloqueador de popups | Permitir descargas del sitio |
| No detecta tablas del SP | SP con CTEs o formato no estándar | Ingresar PKs manualmente en el campo |
| Comboboxes sin respuesta | Corrupción UTF-8 del archivo | Usar siempre el archivo original `_v3.html` |

### 12.2 Verificación de Queries

Antes de ejecutar, verificar:
1. ✅ Nombres de proyecto/dataset/tabla correctos
2. ✅ Backticks (`` ` ``) alrededor de nombres completos
3. ✅ Filtros con sintaxis SQL válida (sin saltos de línea en literales)
4. ✅ PKs escritas exactamente como en la tabla
5. ✅ Para RECORD: nombre de columna RECORD correcto en origen y destino

---

## 13. Integración con Plan de Validación QA

### 13.1 Uso por Nivel y Responsable

| Nivel | Modo | Responsable | Reglas |
|-------|------|-------------|--------|
| **1** | Pruebas Unitarias — En Claro | Data Engineer | UT-01 a UT-04 |
| **1E** | Pruebas Unitarias — Encriptado | Data Engineer | UT-E01 a UT-E04 |
| **1B** | Análisis de Pipeline | Data Engineer / QA | PL-01, PL-02, PL-03 |
| **2** | Validación QA — En Claro | Analista QA | R01–R08 |
| **2E** | Validación QA — Encriptado | Analista QA | RE01–RE04 |

### 13.2 Evidencias Generadas

El Excel exportado sirve como evidencia para:
- ✅ Auditorías de calidad de datos
- ✅ Documentación de pruebas
- ✅ Trazabilidad de validaciones
- ✅ Actas de ratificación
- ✅ Seguimiento de pipelines ETL

---

## 14. Historial de Versiones

| Versión | Archivo | Fecha | Cambios principales |
|---------|---------|-------|---------------------|
| 1.0.0 | `index.html` | 2026-01-20 | Versión inicial: UT-01 a UT-03, R01-R07, Pipeline |
| 1.1.0 | `generador_query_qa.html` | 2026-01-28 | Copia estable con comentario autor |
| 2.0.0 | `generador_query_qa_v2.html` | 2026-02-05 | UT-04/R08 Resumen de Diferencias, tabla diferente estructura |
| 3.0.0 | `generador_query_qa_v3.html` | 2026-02-20 | Nuevo combo "Tipo de información", tablas encriptadas (BYTES), checkbox "Tiene campos BYTES", fix UT diferente estructura, normalización WHERE multilinea |

---
 
