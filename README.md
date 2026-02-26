# Generador de Queries — Documentación Técnica Completa

## Aplicación Web para Validación de Calidad de Datos

| Atributo | Valor |
|----------|-------|
| **Versión** | 5.0.0 |
| **Archivo** | `docs/web/generador_query_qa_v5.html` |
| **Fecha** | 2026-02-26 |
| **Autor** | Sergio Tena |
| **Tecnología** | HTML5 / CSS3 / JavaScript (Vanilla) |
| **Dependencia** | SheetJS (xlsx.js) para exportación Excel |

> **Nota de versión anterior:** La documentación de las versiones 1.x–3.x se encuentra en `DOCUMENTACION_GENERADOR_WEB.md`. Este documento cubre las versiones 4.x y 5.x.

---

## 1. Resumen Ejecutivo

### 1.1 ¿Qué es el Generador de Queries?

El **Generador de Queries** es una aplicación web que automatiza la creación de queries SQL para validación de datos en **Google BigQuery**. Permite generar queries de forma rápida y estandarizada sin necesidad de escribir SQL manualmente, con soporte para tablas encriptadas (campos `BYTES`), tablas RECORD/ARRAY y **queries especializados para desencriptado y análisis de casuísticas**.

### 1.2 Propósito

Facilitar el proceso de validación de calidad de datos mediante:

- ✅ Generación automática de queries SQL estándar (UT/QA) y especializados (ADEX)
- ✅ Estandarización del proceso de validación
- ✅ Reducción de errores humanos en SQL dinámico
- ✅ Exportación de evidencias a Excel
- ✅ Soporte para tablas simples, RECORD/ARRAY y diferente estructura
- ✅ Validación de tablas encriptadas (campos BYTES vs STRING)
- ✅ Queries de desencriptado con `SELECT sql` para integración con el Desencriptador Interseguro
- ✅ Reporte de casuísticas de discrepancias entre origen y destino

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
| **🔍 Casuísticas** | Reporte detallado de diferencias por tipo de discrepancia |
| **🔓 Integración** | Botón de acceso directo al Desencriptador Interseguro |

---

## 2. Arquitectura de Controles (v5)

La versión 5 mantiene los **tres comboboxes** de control y añade un **checkbox de Queries adicionales**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     ESTRUCTURA DE CONTROLES - v5                             │
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
│  [ ⚡ Generar Queries ]  [ 🔍 Queries adicionales ☐ ]  ← NUEVO v5           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Checkbox "Queries adicionales" *(nuevo en v5)*

- Se ubica **a la derecha** del botón "Generar Queries".
- Cuando está **desmarcado**: genera las reglas estándar (UT/QA) según la combinación de combos.
- Cuando está **marcado**: omite las reglas estándar y genera **solo 2 queries especializados** (ADEX-01 y ADEX-02).
- Compatible con tabla **Simple** y tabla **RECORD**.
- Al generar con este modo activo, aparece automáticamente un **banner de acceso al Desencriptador Interseguro**.

### 2.2 Comportamiento según combinación

| Combo 1 | Combo 2 | Combo 3 | Queries adicionales | Reglas generadas |
|---------|---------|---------|---------------------|-----------------|
| ut | simple | clear | ☐ | UT-01, UT-02, UT-03, UT-04 |
| ut | record | clear | ☐ | UT-01, UT-02, UT-03, UT-04 (con UNNEST) |
| ut | different | clear | ☐ | UT-01-DIFF, UT-03-DIFF, UT-04-DIFF |
| ut/qa | simple/record | clear/encrypted | ☑ | **ADEX-01** + **ADEX-02** |
| qa | simple | clear | ☐ | R01–R08 |
| qa | record | clear | ☐ | R01–R08 (con UNNEST) |
| pipeline | — | — | — | PL-01, PL-02, PL-03 |

---

## 3. Correcciones Técnicas Aplicadas (v4 → v5)

### 3.1 R07 y UT-03 — Eliminación de `FORMAT('''...%s...''', bloque)`

**Problema:** R07 y UT-03 usaban `EXECUTE IMMEDIATE FORMAT('''...%s...''', bloque)` para sustituir el bloque de columnas. Las comillas simples de `'IGUAL'` y `'DIFERENTE'` dentro de `FORMAT('''...''')` causaban `Syntax error: Unclosed string literal`.

**Solución aplicada:** Se adoptó el mismo patrón de R08/UT-04:

| Parte | Antes (v4) | Ahora (v5) |
|-------|-----------|-----------|
| Construcción del `bloque` | `STRING_AGG(FORMAT('''...THEN 'IGUAL'...''', col))` | `STRING_AGG(CONCAT("...", col, "..."))` |
| EXECUTE IMMEDIATE | `EXECUTE IMMEDIATE FORMAT('''...%s...''', bloque)` | `EXECUTE IMMEDIATE '''...''' \|\| bloque \|\| '''...'''` |

**Por qué funciona `CONCAT("...")`:**  
Los strings con dobles comillas `"..."` en BigQuery permiten contener comillas simples internas como caracteres normales. El valor de `bloque` queda como `A.col1 ... THEN 'IGUAL' ...`, que es SQL válido cuando se inserta via `|| bloque ||`.

Aplica a los 4 casos:
- UT-03 Simple (EXECUTE IMMEDIATE)
- UT-03 RECORD (EXECUTE IMMEDIATE)
- R07 Simple (EXECUTE IMMEDIATE)
- R07 RECORD (EXECUTE IMMEDIATE)

### 3.2 Checkbox BYTES visible para "Diferente estructura"

**Problema:** El checkbox "Tiene campos BYTES" no se mostraba cuando `currentTableType === 'different'`.

**Solución:** Se actualizó `updateBytesCheckboxVisibility()` para incluir la condición `currentTableType === 'different'`.

---

## 4. Queries Adicionales — ADEX *(nuevo en v5)*

Activados con el checkbox "🔍 Queries adicionales". Generan **exactamente 2 cuadros** de query, omitiendo todas las reglas UT/QA estándar.

### 4.1 ADEX-01: Query para desencriptar tabla

**Objetivo:** Generar un `SELECT` dinámico desde los metadatos de la tabla destino, aplicando el prefijo `v_tipo_` a los campos BYTES para que el Desencriptador Interseguro los procese automáticamente.

#### 4.1.1 ADEX-01 — Tabla Simple

**Estructura del query generado:**

```sql
DECLARE dataset_id      STRING DEFAULT 'mi_dataset';
DECLARE table_id        STRING DEFAULT 'mi_tabla';
-- ⚠️ Condición de filtro (sin WHERE). Deja TRUE para traer todo.
DECLARE where_condition STRING DEFAULT """PERIODO = '2025-12-01'""";
-- ⚠️ Códigos: C_DOCUMENT_NUMBER, C_DOCUMENT_TYPE, C_NAME, C_EMAIL, C_ADDRESS, C_CELLPHONE
DECLARE codigo_tipo     STRING DEFAULT 'C_NAME';

DECLARE sql STRING;

SET sql = (
    SELECT CONCAT(
        'SELECT\n',
        STRING_AGG(
            CASE
                WHEN data_type = 'BYTES'
                THEN FORMAT("    r.%s,\n    '%s' AS v_tipo_%s",
                            column_name, codigo_tipo, column_name)
                ELSE CONCAT('    r.', column_name)
            END,
            ',\n' ORDER BY ordinal_position
        ),
        '\nFROM `proyecto.dataset.tabla` r\nWHERE ',
        where_condition,
        '\nLIMIT 100;'
    )
    FROM `proyecto.dataset.INFORMATION_SCHEMA.COLUMNS`
    WHERE table_name = table_id
);

-- Muestra el SELECT generado (copia y pega en BigQuery para ejecutarlo)
SELECT sql;
```

**Campos especiales generados:**
- `r.CAMPO` — campo sin prefijo (se incluye tal cual)
- `r.CAMPO_BYTES, 'C_NAME' AS v_tipo_CAMPO_BYTES` — campo BYTES con código KMS para desencriptado

> **Nota:** Se usa `"""..."""` (triple doble comilla) para el `DEFAULT` de `where_condition`, permitiendo que el filtro contenga comillas simples sin errores de parseo.

#### 4.1.2 ADEX-01 — Tabla RECORD

Similar al Simple, pero consulta `INFORMATION_SCHEMA.COLUMN_FIELD_PATHS` para obtener los sub-campos del RECORD y genera:

```sql
DECLARE record_col      STRING DEFAULT 'DATOS_VEHICULO';
DECLARE where_condition STRING DEFAULT """PERIODO = '2025-12-01'""";
DECLARE codigo_tipo     STRING DEFAULT 'C_NAME';

SET sql = (
    SELECT FORMAT(
        'SELECT\n%s\nFROM `proyecto.dataset.tabla` t,\nUNNEST(t.%s) AS r\nWHERE %s\nLIMIT 100;',
        STRING_AGG(
            CASE
                WHEN data_type = 'BYTES'
                THEN FORMAT("    r.%s,\n    '%s' AS v_tipo_%s",
                            REPLACE(field_path, CONCAT(record_col, '.'), ''),
                            codigo_tipo,
                            REPLACE(REPLACE(field_path, ...), '.', '_'))
                ELSE CONCAT('    r.', REPLACE(field_path, CONCAT(record_col, '.'), ''))
            END,
            ',\n' ORDER BY field_path
        ),
        record_col,
        where_condition
    )
    FROM `proyecto.dataset.INFORMATION_SCHEMA.COLUMN_FIELD_PATHS`
    WHERE table_name = table_id
      AND column_name = record_col
      AND field_path != record_col
);

SELECT sql;
```

### 4.2 ADEX-02: Reporte de casuísticas

**Objetivo:** Comparar solo las columnas comunes entre origen y destino (`COLUMNAS_COMUNES` = INNER JOIN de metadatos), generando un reporte agrupado por tipo de discrepancia.

**Tipos de casuística detectados:**

| Casuística | Descripción |
|------------|-------------|
| `Solo_en_destino` / `Solo_en_destino_Business` | El campo tiene valor en destino pero NULL/vacío en origen |
| `Solo_en_origen` / `Solo_en_origen_Produccion` | El campo tiene valor en origen pero NULL/vacío en destino |
| `Diferencia_mayusculas_minusculas` | Valores iguales al normalizar con `LOWER()` pero distintos sin normalizar |
| `Diferencia_espacios_trim` | Valores iguales al aplicar `TRIM()` pero distintos sin aplicarlo |
| `Valor_distinto_otro` | Diferencia que no cae en ninguna de las categorías anteriores |

#### 4.2.1 ADEX-02 — Tabla Simple

**Metadatos usados:**
- `METADATOS_ORIGEN`: `INFORMATION_SCHEMA.COLUMNS` de la tabla origen
- `METADATOS_DESTINO`: `INFORMATION_SCHEMA.COLUMNS` de la tabla destino
- `COLUMNAS_COMUNES`: INNER JOIN de ambos, excluyendo PKs y tipos complejos

**Estructura de cada bloque de comparación (por campo):**
```sql
SELECT
    'mi_tabla' AS tabla,
    'CAMPO'    AS campo,
    (CASE
        WHEN (O.CAMPO IS NULL OR TRIM(CAST(O.CAMPO AS STRING)) = '')
             AND (D.CAMPO IS NOT NULL AND TRIM(CAST(D.CAMPO AS STRING)) <> '')
             THEN 'Solo_en_destino'
        WHEN (O.CAMPO IS NOT NULL AND TRIM(CAST(O.CAMPO AS STRING)) <> '')
             AND (D.CAMPO IS NULL OR TRIM(CAST(D.CAMPO AS STRING)) = '')
             THEN 'Solo_en_origen'
        WHEN LOWER(TRIM(CAST(O.CAMPO AS STRING))) = LOWER(TRIM(CAST(D.CAMPO AS STRING)))
             AND TRIM(CAST(O.CAMPO AS STRING)) <> TRIM(CAST(D.CAMPO AS STRING))
             THEN 'Diferencia_mayusculas_minusculas'
        WHEN TRIM(CAST(O.CAMPO AS STRING)) = TRIM(CAST(D.CAMPO AS STRING))
             AND CAST(O.CAMPO AS STRING) <> CAST(D.CAMPO AS STRING)
             THEN 'Diferencia_espacios_trim'
        ELSE 'Valor_distinto_otro'
    END) AS casuistica,
    D.pk1, D.pk2
FROM `destino` D
INNER JOIN `origen` O ON D.pk1 = O.pk1 AND D.pk2 = O.pk2
WHERE COALESCE(TRIM(CAST(O.CAMPO AS STRING)), '') <> COALESCE(TRIM(CAST(D.CAMPO AS STRING)), '')
```

**EXECUTE IMMEDIATE:**
```sql
EXECUTE IMMEDIATE '''
WITH diferencias AS (
''' || bloque || '''
)
SELECT
    tabla, campo, casuistica,
    COUNT(*) AS cantidad_registros,
    ARRAY_AGG(STRUCT(pk1, pk2) ORDER BY pk1, pk2 LIMIT 5) AS muestra_claves
FROM diferencias
GROUP BY 1, 2, 3
ORDER BY 1, 2, 4 DESC;
''';
```

**Manejo de columnas sin coincidencias:**
```sql
IF bloque IS NULL THEN
    SELECT 'Sin columnas comunes entre las dos tablas. Verifica los nombres y filtros.' AS error;
ELSE
    EXECUTE IMMEDIATE '''...''' || bloque || '''...''';
END IF;
```

#### 4.2.2 ADEX-02 — Tabla RECORD *(adaptado en v5)*

**Diferencia clave con Simple:** AMBAS tablas tienen columna RECORD/ARRAY. Se usan CTEs `origen_cte` y `destino_cte` con UNNEST, y los metadatos provienen de `COLUMN_FIELD_PATHS` para ambas.

**Metadatos usados:**
- `METADATOS_ORIGEN`: `INFORMATION_SCHEMA.COLUMN_FIELD_PATHS` filtrando por `column_name = recColOrigen`
- `METADATOS_DESTINO`: `INFORMATION_SCHEMA.COLUMN_FIELD_PATHS` filtrando por `column_name = recColDestino`
- Ambos aplican `AND data_type NOT IN ('ARRAY', 'STRUCT', 'RECORD', 'GEOGRAPHY', 'JSON')`
- `COLUMNAS_COMUNES`: INNER JOIN de ambos, excluyendo PKs

**Estructura de cada bloque de comparación (por campo):**
```sql
SELECT
    'RECORD_COL' AS array_nombre,
    'CAMPO'      AS campo,
    (CASE ... END) AS casuistica,
    DA.pk1, DA.pk2
FROM destino_cte DA
INNER JOIN origen_cte O ON DA.pk1 = O.pk1 AND DA.pk2 = O.pk2
WHERE COALESCE(TRIM(CAST(O.CAMPO AS STRING)), '') <> COALESCE(TRIM(CAST(DA.CAMPO AS STRING)), '')
```

**EXECUTE IMMEDIATE:**
```sql
EXECUTE IMMEDIATE '''
WITH origen_cte AS (
    SELECT t.pk1, t.pk2, r.*
    FROM `origen` t, UNNEST(t.RECORD_ORIGEN) AS r
    WHERE filtroOrigen
),
destino_cte AS (
    SELECT t.pk1, t.pk2, d.*
    FROM `destino` t, UNNEST(t.RECORD_DESTINO) AS d
    WHERE filtroDestino
),
diferencias AS (
''' || bloque || '''
)
SELECT
    array_nombre, campo, casuistica,
    COUNT(*) AS cantidad_registros,
    ARRAY_AGG(STRUCT(pk1, pk2) ORDER BY pk1, pk2 LIMIT 5) AS muestra_claves
FROM diferencias
GROUP BY 1, 2, 3
ORDER BY 1, 2, 4 DESC;
''';
```

---

## 5. Banner de Acceso al Desencriptador *(nuevo en v5)*

Cuando el checkbox "🔍 Queries adicionales" está marcado y se presiona "Generar Queries", aparece automáticamente un **banner** en la zona de resultados con acceso directo al Desencriptador de Interseguro:

```
┌──────────────────────────────────────────────────────────────────────┐
│  🔓  Herramienta de Desencriptado / Encriptado                       │
│      Para ejecutar las queries generadas, utiliza el servicio de     │
│      desencriptado de Interseguro.                 [🔑 Abrir]        │
└──────────────────────────────────────────────────────────────────────┘
```

- **URL:** https://demo-decryptor-726731649140.us-central1.run.app/
- Se abre en una **nueva pestaña** (`target="_blank"`).
- Solo visible cuando "Queries adicionales" está activo.

### 5.1 Funcionalidades del Desencriptador Interseguro

El desencriptador soporta:

| Pestaña | Descripción |
|---------|-------------|
| 🔓 Desencriptar | Desencripta un dato individual usando tipo de campo |
| 🔒 Encriptar | Encripta un dato individual usando tipo de campo |
| 🗄️ Desencriptar SQL | Ejecuta un SELECT en BigQuery y guarda los datos desencriptados en una tabla destino |
| 🔐 Encriptar SQL | Ejecuta un SELECT en BigQuery y guarda los datos encriptados en una tabla destino |

**Convención de campos en las queries SQL:**

| Prefijo de alias | Función |
|-----------------|---------|
| `v_key_CAMPO` | Campo de clustering (se incluye tal cual) |
| `'C_CODE' AS v_tipo_CAMPO` | Campo a desencriptar/encriptar con el código KMS indicado |
| `v_part_CAMPO` | Campo para crear partición DATE en la tabla destino |
| *(sin prefijo)* | Se incluye tal cual sin transformación |

**Códigos KMS disponibles:**
- `C_DOCUMENT_NUMBER` — Número de documento
- `C_DOCUMENT_TYPE` — Tipo de documento
- `C_NAME` — Nombre y apellidos
- `C_EMAIL` — Email o correo electrónico
- `C_ADDRESS` — Dirección
- `C_CELLPHONE` — Teléfono

---

## 6. Manejo de Comillas en BigQuery Scripting

Un problema frecuente en la generación de SQL dinámico es el anidamiento de comillas. La v5 aplica las siguientes estrategias:

### 6.1 `DECLARE DEFAULT` con comillas simples en el valor

**Problema:** `DECLARE where_condition STRING DEFAULT 'PERIODO = '2025-12-01'';` rompe el parser.

**Solución:** Usar triple doble comilla para el string de `DEFAULT`:
```sql
DECLARE where_condition STRING DEFAULT """PERIODO = '2025-12-01'""";
```
BigQuery acepta `"""..."""` como string literal. Las comillas simples dentro son caracteres normales. Aplicado en ADEX-01 Simple y RECORD.

### 6.2 `EXECUTE IMMEDIATE` con variable `bloque`

**Problema:** `EXECUTE IMMEDIATE FORMAT('''...%s...''', bloque)` falla cuando `bloque` contiene comillas simples (como `'IGUAL'`).

**Solución:** Concatenación de strings con operador `||`:
```sql
EXECUTE IMMEDIATE ''' ... ''' || bloque || ''' ... ''';
```
Dentro de `'''...'''`, las comillas simples aisladas son caracteres normales. El `||` concatena en tiempo de ejecución de BigQuery. Aplicado en UT-03, R07, ADEX-02.

### 6.3 `STRING_AGG(CONCAT(...))` en lugar de `STRING_AGG(FORMAT('''...''', ...))`

**Problema:** `FORMAT('''...THEN 'IGUAL'...''', col)` puede generar conflictos de comillas en BigQuery al ser evaluado en un contexto `SET bloque = (...)`.

**Solución:** `CONCAT("...THEN 'IGUAL'...", col, "...")` usando strings con comillas dobles, donde las comillas simples son caracteres normales. Aplicado en UT-03 y R07 EXECUTE IMMEDIATE.

---

## 7. Tipos de Tabla Soportados

### 7.1 Tabla Simple

Para tablas con estructura plana sin campos anidados.

**Características:**
- Usa `INFORMATION_SCHEMA.COLUMNS` para obtener campos
- Filtro `AND data_type != 'BYTES'` en modo "En Claro" si el checkbox "tiene campos BYTES" está activo
- Filtro `AND data_type = 'BYTES'` en modo "Encriptado" si el checkbox está activo

### 7.2 Tabla con RECORD/ARRAY

Para tablas con campos anidados (STRUCT/RECORD) en BigQuery.

**Características especiales:**
- Usa `INFORMATION_SCHEMA.COLUMN_FIELD_PATHS` para obtener campos anidados del RECORD
- Aplica `UNNEST` para aplanar estructuras en CTEs (`origen_cte`, `destino_cte`)
- Requiere configurar la **columna RECORD** para origen y destino
- En ADEX-02, **ambas tablas** (origen y destino) son tratadas como RECORD: se usan dos CTEs con UNNEST
- El checkbox **"Tiene campos BYTES"** también disponible en modo "En Claro"

### 7.3 Tabla con Diferente Estructura

Para comparar tablas donde las columnas tienen nombres diferentes entre origen y destino.

**Características especiales:**
- Requiere configurar PKs de ORIGEN y DESTINO por separado
- Genera plantillas de mapeo manual con `<<CAMPO_ORIGEN>>` y `<<CAMPO_DESTINO>>`
- Detección de sinónimos comunes
- El checkbox "Tiene campos BYTES" es visible también para este tipo (corregido en v4)

---

## 8. Interfaz de Usuario

### 8.1 Estructura de la Pantalla

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 GENERADOR DE QUERIES - VALIDACIÓN DE CALIDAD DE DATOS                   │
│     (sin número de versión en el título desde v5)                           │
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
│  [ ⚡ Generar Queries ]  [ 🔍 Queries adicionales ☐ ]  [ 📊 Exportar Excel ]│
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [Solo cuando Queries adicionales está activo:]                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │ 🔓  Herramienta de Desencriptado / Encriptado            [🔑 Ir] │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│  📋 QUERIES GENERADOS                                                       │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │ ADEX-01: Query para desencriptar tabla             [Copiar]      │      │
│  │ ADEX-02: Reporte de casuísticas                    [Copiar]      │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Campos de Entrada

| Campo | Descripción | Obligatorio | Notas |
|-------|-------------|-------------|-------|
| **Proyecto** | ID del proyecto GCP | Sí | Para origen y destino |
| **Dataset** | Nombre del dataset | Sí | Para origen y destino |
| **Tabla** | Nombre de la tabla | Sí | Para origen y destino |
| **Filtro** | Condición WHERE opcional | No | Soporta comillas simples en el valor (via `"""..."""`) |
| **Join** | Tablas adicionales para JOIN | No | Para consultas con tablas relacionadas |
| **Primary Keys** | Claves primarias | Según tipo | No requerido en modo Encriptado |
| **Columna RECORD** | Nombre del campo RECORD | Solo Tabla RECORD | Para origen y destino |
| **Tiene campos BYTES** | Checkbox filtro | No | Activa filtro `data_type` en INFORMATION_SCHEMA |
| **Queries adicionales** | Checkbox modo ADEX | No | Activa generación de ADEX-01 y ADEX-02 |

---

## 9. Flujo de Uso

### 9.1 Queries Estándar (UT/QA)

```
PASO 1: CONFIGURACIÓN
═════════════════════
1.1 Abrir generador_query_qa_v5.html en navegador
1.2 Seleccionar Tipo de Validación: "Pruebas Unitarias" o "Validación QA"
1.3 Seleccionar Tipo de tabla: "Simple", "RECORD" o "Diferente Estructura"
1.4 Seleccionar Tipo de información: "En Claro" o "Encriptado (BYTES)"
1.5 Verificar que "Queries adicionales" esté DESMARCADO

PASO 2: INGRESO DE DATOS
════════════════════════
2.1 Completar campos de ORIGEN y DESTINO
2.2 Ingresar PKs separadas por coma
2.3 (Si es RECORD) Ingresar columna RECORD para origen y destino
2.4 (Opcional) Activar checkbox "Tiene campos BYTES" para excluirlos

PASO 3: GENERACIÓN
═══════════════════
3.1 Click en "⚡ Generar Queries"
3.2 Se generan los queries UT-01 a UT-04 / R01 a R08 según combinación

PASO 4: EXPORTACIÓN
═════════════════════
4.1 Click en "📊 Exportar a Excel"
```

### 9.2 Queries Adicionales (ADEX — Desencriptado y Casuísticas)

```
PASO 1: CONFIGURACIÓN
═════════════════════
1.1 Seleccionar Tipo de tabla: "Simple" o "Tabla con RECORD"
1.2 MARCAR el checkbox "🔍 Queries adicionales"

PASO 2: INGRESO DE DATOS
════════════════════════
2.1 Completar campos de ORIGEN y DESTINO
2.2 Ingresar PKs separadas por coma
2.3 (Si es RECORD) Ingresar columna RECORD para origen y destino
2.4 (Opcional) Ingresar filtro WHERE — soporta comillas simples (ej. PERIODO = '2025-12-01')

PASO 3: GENERACIÓN
═══════════════════
3.1 Click en "⚡ Generar Queries"
3.2 Aparecen 2 cuadros: ADEX-01 y ADEX-02
3.3 Aparece banner de acceso al Desencriptador Interseguro

PASO 4: USAR ADEX-01 (Desencriptar tabla)
═══════════════════════════════════════════
4.1 Copiar y ejecutar en BigQuery → obtener el valor de la columna `sql`
4.2 Copiar ese SQL generado
4.3 Ir al Desencriptador: https://demo-decryptor-726731649140.us-central1.run.app/
    → Pestaña "🗄️ Desencriptar SQL"
4.4 Pegar el SQL en el campo "Query SQL"
4.5 Indicar la tabla destino en BigQuery
4.6 Ejecutar y guardar

PASO 5: USAR ADEX-02 (Reporte de casuísticas)
═══════════════════════════════════════════════
5.1 Copiar y ejecutar directamente en BigQuery
5.2 Revisar el reporte agrupado por campo y tipo de casuística
5.3 La columna `muestra_claves` muestra hasta 5 ejemplos de PKs afectadas
```

---

## 10. Detalle de Queries Generados

### 10.1 Queries UT — En Claro (sin cambios respecto a v3)

Ver `DOCUMENTACION_GENERADOR_WEB.md` sección 8.1.

### 10.2 Queries QA — En Claro

#### R07: Valores Coinciden (EXECUTE IMMEDIATE — corregido en v5)

**Modo EXECUTE IMMEDIATE (un solo script):**
```sql
DECLARE bloque STRING;

SET bloque = (
    WITH ORIGEN AS (
        SELECT column_name
        FROM `proy.dataset.INFORMATION_SCHEMA.COLUMNS`
        WHERE table_name = 'tabla'
          AND column_name NOT IN ('pk1', 'pk2')
    ),
    DESTINO AS (...)
    SELECT STRING_AGG(
        CONCAT(
            "A.", a.column_name, " AS valor_origen_", a.column_name, ", ",
            "B.", b.column_name, " AS valor_destino_", b.column_name, ", ",
            "CASE WHEN A.", a.column_name, " IS NULL AND B.", b.column_name,
            " IS NULL THEN 'IGUAL' ",
            "WHEN UPPER(TRIM(CAST(A.", a.column_name, " AS STRING))) = ...",
            " THEN 'IGUAL' ",
            "ELSE 'DIFERENTE' END AS estado_", a.column_name
        ),
        ',\n'
    )
    FROM ORIGEN a JOIN DESTINO b ON UPPER(a.column_name) = UPPER(b.column_name)
);

EXECUTE IMMEDIATE '''
WITH origen AS (...), destino AS (...)
SELECT DISTINCT pk_cols,
    ''' || bloque || '''
FROM origen A
FULL OUTER JOIN destino B
    ON COALESCE(CAST(A.pk AS STRING), '') = COALESCE(CAST(B.pk AS STRING), '')
ORDER BY 1, 2
LIMIT 1000;
''';
```

**Resultado esperado:** `0 registros` con diferencias.

#### R08: Resumen de Diferencias (EXECUTE IMMEDIATE)

Igual al patrón de R07 pero genera dos variables (`bloque_diff` y `bloque_sum`) con `SELECT AS STRUCT` y usa:
```sql
EXECUTE IMMEDIATE '''...''' || bloque_diff || '''...''' || bloque_sum || '''...''';
```

### 10.3 ADEX-01: Query para desencriptar tabla

Ver sección 4.1 de esta documentación.

### 10.4 ADEX-02: Reporte de casuísticas

Ver sección 4.2 de esta documentación.

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
changeMode()                       // Cambia modo; oculta/muestra combos y botones
changeTableType()                  // Cambia tipo de tabla; controla campos RECORD/different
changeInfoType()                   // Cambia tipo de info; controla sección encriptado y checkboxes BYTES
updateBytesCheckboxVisibility()    // Muestra/oculta checkboxes BYTES según modo+tipo (incl. 'different')
validateForm()                     // Valida campos obligatorios según combinación activa
getInputValues()                   // Recopila todos los valores del formulario
```

### 11.3 Funciones de Generación

```javascript
generateQueries()                  // Dispatcher: llama la función correcta según modo+tipo+info+adicionales
generateAdditionalQueries(v)       // Genera ADEX-01 + ADEX-02 (simple o record)
generateUTQueries(v)               // Genera UT-01 a UT-04 (simple y record)
generateQAQueries(v)               // Genera R01 a R08 (simple y record)
generateUTQueriesDifferent(v)      // Genera UT-01-DIFF, UT-03, UT-04 (diferente estructura)
generateQAQueriesDifferent(v)      // Genera R01-DIFF, R07, R08 (diferente estructura)
generateEncryptedQueries(v)        // Genera UT-E01 a E04 / RE01 a RE04 (encriptado)
generatePipelineQueries()          // Genera PL-01, PL-02, PL-03
```

### 11.4 Lógica del dispatcher `generateQueries()`

```javascript
function generateQueries() {
    const queriesAdicionales = document.getElementById('queriesAdicionalesCheck')?.checked ?? false;

    if (queriesAdicionales) {
        queries = generateAdditionalQueries(v);
        // → Inyecta banner del Desencriptador en el DOM
    } else if (currentInfoType === 'encrypted') {
        queries = generateEncryptedQueries(v);
    } else if (currentTableType === 'different') {
        queries = currentMode === 'ut' ? generateUTQueriesDifferent(v) : generateQAQueriesDifferent(v);
    } else {
        queries = currentMode === 'ut' ? generateUTQueries(v) : generateQAQueries(v);
    }
}
```

### 11.5 Función `generateAdditionalQueries(v)` — Detalle

```javascript
function generateAdditionalQueries(v) {
    const isRecord = currentTableType === 'record';
    const pkList   = v.primaryKeys.split(',').map(p => p.trim());

    // Variables de filtros
    const whereDestinoStr = v.filtroDestino || 'TRUE';
    const whereOrigenRaw  = v.filtroOrigen  ? v.filtroOrigen.trim()  : '';
    const whereDestinoRaw = v.filtroDestino ? v.filtroDestino.trim() : '';

    // Filtro BYTES (excluye BYTES si destinoHasBytes = true)
    const bytesFilterDestino = v.destinoHasBytes ? "\n          AND data_type != 'BYTES'" : '';

    const queries = [];

    if (isRecord) {
        // ADEX-01 RECORD + ADEX-02 RECORD
    } else {
        // ADEX-01 Simple + ADEX-02 Simple
    }

    return queries;
}
```

---

## 12. Solución de Problemas

### 12.1 Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `Unclosed string literal at [N:M]` | Filtro WHERE con comillas simples | La app usa `"""..."""` en DECLAREs; verificar el filtro |
| `EXECUTE IMMEDIATE sql string cannot be NULL` | `COLUMNAS_COMUNES` vacío (sin campos comunes) | ADEX-02 devuelve mensaje: "Sin columnas comunes..." |
| `Syntax error inside FORMAT('''...''')` | Comillas simples en el bloque dinámico | Corregido en v5: se usa `CONCAT("...")` y `|| bloque ||` |
| "Campos requeridos" | Falta proyecto, dataset o tabla | Completar todos los campos obligatorios |
| "Primary Keys vacías" | No se ingresaron PKs | Ingresar al menos una PK |
| Excel no se descarga | Bloqueador de popups | Permitir descargas del sitio |
| Comboboxes sin respuesta | Error JS en el script | Usar la versión original del archivo; no editar manualmente |

### 12.2 Verificación de Queries

Antes de ejecutar en BigQuery, verificar:
1. ✅ Nombres de proyecto/dataset/tabla correctos
2. ✅ Backticks (`` ` ``) alrededor de nombres completos
3. ✅ Filtros con sintaxis SQL válida
4. ✅ PKs escritas exactamente como en la tabla
5. ✅ Para RECORD: nombre de columna RECORD correcto en origen y destino
6. ✅ Para ADEX-01: `codigo_tipo` con el código KMS correcto antes de copiar
7. ✅ Para ADEX-02: verificar que hay columnas comunes entre origen y destino

---

## 13. Exportación a Excel

Sin cambios respecto a la documentación v1 (ver `DOCUMENTACION_GENERADOR_WEB.md` sección 9).

Los queries ADEX-01 y ADEX-02 se incluyen en la exportación cuando se generan.

---

## 14. Historial de Versiones

| Versión | Archivo | Fecha | Cambios principales |
|---------|---------|-------|---------------------|
| 1.0.0 | `index.html` | 2026-01-20 | Versión inicial: UT-01 a UT-03, R01-R07, Pipeline |
| 2.0.0 | `generador_query_qa_v2.html` | 2026-02-05 | UT-04/R08 Resumen de Diferencias, tabla diferente estructura |
| 3.0.0 | `generador_query_qa_v3.html` | 2026-02-20 | Nuevo combo "Tipo de información", tablas encriptadas (BYTES), checkbox "Tiene campos BYTES", normalización WHERE multilinea |
| 4.0.0 | `generador_query_qa_v4.html` | 2026-02-24 | Fix comillas anidadas en R07/UT03 (FORMAT→CONCAT+\|\|), checkbox BYTES visible para "Diferente estructura", mejoras COALESCE en JOIN y ORDER BY con todas las PKs |
| 5.0.0 | `generador_query_qa_v5.html` | 2026-02-26 | **Nuevo:** checkbox "🔍 Queries adicionales", ADEX-01 (desencriptar tabla Simple/RECORD), ADEX-02 (reporte casuísticas Simple/RECORD con COLUMNAS_COMUNES), banner de acceso al Desencriptador Interseguro, título sin número de versión visible, fix WHERE con comillas simples en ADEX-01 (DEFAULT `"""..."""`), ADEX-02 RECORD usa `COLUMN_FIELD_PATHS` para ambas tablas + CTEs `origen_cte`/`destino_cte` con UNNEST |

---

## 15. Integración con el Desencriptador Interseguro

El botón "🔑 Abrir Desencriptador" enlaza a: https://demo-decryptor-726731649140.us-central1.run.app/

### 15.1 Flujo completo de desencriptado con ADEX-01

```
BigQuery (tabla con BYTES)
        │
        ▼
  [ADEX-01 — generado por la herramienta]
  DECLARE where_condition STRING DEFAULT """filtro""";
  DECLARE codigo_tipo STRING DEFAULT 'C_NAME';
  SET sql = (...); SELECT sql;
        │
        ▼
  BigQuery ejecuta → devuelve columna `sql` con SELECT dinámico
        │
        ▼
  Copiar el contenido de `sql`
        │
        ▼
  Desencriptador Interseguro → 🗄️ Desencriptar SQL
  • Pegar SELECT en "Query SQL"
  • Indicar tabla destino
  • ▶️ Ejecutar y guardar
        │
        ▼
  Tabla destino en BigQuery con datos desencriptados
```

### 15.2 Convención de campos para el Desencriptador

Los campos que ADEX-01 genera con el patrón `'C_CODE' AS v_tipo_CAMPO` son reconocidos automáticamente por el Desencriptador para determinar qué campos desencriptar y con qué clave KMS.

---

*Documentación generada: 2026-02-26 | Versión del documento: 2.0.0*
