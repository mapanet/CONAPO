# 02 — INEGI Censo 2020 (Datos a Nivel Manzana)

Este dataset contiene datos de población y viviendas del Censo 2020 a nivel manzana  
(AGEB + Manzana). Es un insumo fundamental para el pipeline de NSE.

---

## 📌 Aclaraciones Importantes

- Se crea un dataset del Censo 2020 a nivel manzana para obtener Población, Viviendas y Viviendas Habitadas a nivel Manzana (Block).
- Este dataset se utiliza en el Paso 5.9 del NSE para actualizar **Population y Dwellings** a nivel Colonia (Neighborhood) mediante agregación ponderada.
- También puede agregarse para obtener totales a nivel **AGEB, Ciudad, Municipio y Estado.**
- Posteriormente se puede calcular:
**Unoccupied_Dwellings = Dwellings – Occupied_Dwellings**

### Terminología

| Español | Inglés | Significado |
|---------|---------|---------|
| AGEB | Basic Geo‑Statistical Area | INEGI statistical unit |
| Manzana | Block | Smallest urban unit |

---

## Tabla resultante: `INEGI_Censo_2020_AGEB`

| Column | Type | Notes |
|--------|------|--------|
| ENTIDAD | varchar(2) | Código Estado |
| NOM_ENT | nvarchar(85) | Nombre Estado |
| MUN | varchar(3) | Código Municipio |
| NOM_MUN | nvarchar(85) | Nombre Municipio |
| LOC | varchar(4) | Código Localidad |
| NOM_LOC | nvarchar(110) | Nombre Localidad |
| AGEB | varchar(4) | Código Area |
| MZA | varchar(4) | Código Manzana |
| POBTOT | int | Población |
| VIVTOT | int | Viviendas |
| TVIVHAB | int | Viviendas Ocupadas |

Carpeta de trabajo:

`D:\AXSI\INEGI\Censo_2020\Download`


---

# 1 — Descargar Datos del Censo 2020 (SCITEL)

Descargamos los datos del Censo 2020 a nivel manzana desde INEGI SCITEL:

**URL:**  
https://www.inegi.org.mx/app/scitel/Default?ev=10  
**Sección:** *Resultados por AGEB y Manzana Urbana*

[<img src="/docs/images/Censo_2020_1.png" width="1000">](/docs/images/Censo_2020_1.png)


### IMPORTANTE — NO usar los botones grises de CSV o XLSX

En el panel izquierdo, verás botones grises de CSV y XLSX.  
Estos exportan el dataset completo, el cual contiene demasiados campos que no necesitamos.

Solo queremos:

- **Poblacion Total**  
- **Total Viviendas**  
- **Viviendas Ocupadas**  


## ✔ Proceso de descarga (Repetir para los 32 Estados)

En el panel derecho, selecciona:

1. **Identificación geográfica** → todo marcado  
2. **Población** → *Población total*  
3. **Vivienda** → *Total de viviendas*  
4. **Vivienda** → *Total de viviendas habitadas*  

Pasos para descaragar los 32 estados:

1. En el panel izquierdo, selecciona un estado (ejemplo: Aguascalientes).
2. En la parte inferior derecha → haz clic en Generar Consulta.
3. En la parte inferior central → haz clic en Exportar a → CSV.
4. Guarda el archivo en:
```code
D:\AXSI\INEGI\Censo_2020\Tabulados_AGEB_Manzana   
```

Haz click en el botón `Regresar` del naegador para seleccionar el siguinet estado.

Ejemplo de resultados:

[<img src="/docs/images/Censo_2020_3.png" width="1000">](/docs/images/Censo_2020_3.png)


## ✔ Verifica que has descargado los 32 estados

| Archivo |
|-----------|
| RESAGEBURB2020 - 01 Aguascalientes.csv |
| RESAGEBURB2020 - 02 Baja California.csv |
| RESAGEBURB2020 - 03 Baja California Sur.csv |
| RESAGEBURB2020 - 04 Campeche.csv |
| RESAGEBURB2020 - 05 Coahuila de Zaragoza.csv |
| RESAGEBURB2020 - 06 Colima.csv |
| RESAGEBURB2020 - 07 Chiapas.csv |
| RESAGEBURB2020 - 08 Chihuahua.csv |
| RESAGEBURB2020 - 09 Ciudad de México.csv |
| RESAGEBURB2020 - 10 Durango.csv |
| RESAGEBURB2020 - 11 Guanajuato.csv |
| RESAGEBURB2020 - 12 Guerrero.csv |
| RESAGEBURB2020 - 13 Hidalgo.csv |
| RESAGEBURB2020 - 14 Jalisco.csv |
| RESAGEBURB2020 - 15 México.csv |
| RESAGEBURB2020 - 16 Michoacán de Ocampo.csv |
| RESAGEBURB2020 - 17 Morelos.csv |
| RESAGEBURB2020 - 18 Nayarit.csv |
| RESAGEBURB2020 - 19 Nuevo León.csv |
| RESAGEBURB2020 - 20 Oaxaca.csv |
| RESAGEBURB2020 - 21 Puebla.csv |
| RESAGEBURB2020 - 22 Querétaro.csv |
| RESAGEBURB2020 - 23 Quintana Roo.csv |
| RESAGEBURB2020 - 24 San Luis Potosí.csv |
| RESAGEBURB2020 - 25 Sinaloa.csv |
| RESAGEBURB2020 - 26 Sonora.csv |
| RESAGEBURB2020 - 27 Tabasco.csv |
| RESAGEBURB2020 - 28 Tamaulipas.csv |
| RESAGEBURB2020 - 29 Tlaxcala.csv |
| RESAGEBURB2020 - 30 Veracruz.csv |
| RESAGEBURB2020 - 31 Yucatán.csv |
| RESAGEBURB2020 - 32 Zacatecas.csv |



# 2 — Concatenar Todos los Archivos en un Solo CSV (TSV) Limpio

### Propósito
Combinar los 32 archivos CSV de los estados en un solo archivo UTF‑8 (sin BOM), separado por TAB, listo para importación masiva en SQL Server.

### Archivo de salida

`RESAGEBURB2020_ALL_TAB.csv`

- Codificación: **UTF‑8 no BOM**  
- Separador: **TAB**  
- Reemplazar todos los `*` por cadena vacía (NULL en SQL)

### Script de Python 

```python
import pandas as pd
import glob

# List all CSV files
csv_files = glob.glob(r"D:\AXSI\INEGI\Censo_2020\Tabulados_AGEB_Manzana\RESAGEBURB2020 - *.csv")

dfs = []
for i, f in enumerate(csv_files):
    print("Reading:", f)
    # Force codes to be strings
    df = pd.read_csv(f, dtype={
        "ENTIDAD": str,
        "MUN": str,
        "LOC": str,
        "AGEB": str,
        "MZA": str
    })
    dfs.append(df)

merged = pd.concat(dfs, ignore_index=True)

# Save as comma-separated
# merged.to_csv(r"D:\AXSI\INEGI\Censo_2020\Tabulados_AGEB_Manzana\RESAGEBURB2020_ALL_COMA.csv", index=False)
# Save tab separated
merged.to_csv(r"D:\AXSI\INEGI\Censo_2020\Tabulados_AGEB_Manzana\RESAGEBURB2020_ALL_TAB.csv", index=False, sep="\t")
```


### Archivo de salida esperado

Después de ejecutar el script, deberías obtener:

`RESAGEBURB2020_ALL_TAB.csv`

Puedes abrir el archivo con EditPad Pro, Notepad++ o VS Code y verificar:

- Codificación: **UTF‑8 (sin BOM)**
- Separador: **TAB**
- Sin asteriscos (`*`)

Todas las filas alineadas y completas

Ejemplo de filas:

| ENTIDAD | NOM_ENT        |     MUN | NOM_MUN                            |     LOC | NOM_LOC                      | AGEB | MZA | POBTOT | VIVTOT | TVIVHAB |
|---------|----------------|---------|------------------------------------|---------|------------------------------|------|-----|--------|--------|---------|
| 01      | Aguascalientes | 000     | Total de la entidad Aguascalientes | 0000    | Total de la entidad          | 0000 | 000 | 1425607 | 463972 | 386671 |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0000    | Total del municipio          | 0000 | 000 | 948990  | 313256 | 266942 |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Total de la localidad urbana | 0000 | 000 | 863893  | 286646 | 246259 |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Total AGEB urbana            | 0017 | 000 | 2237    | 1288   | 648    |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 011 | 115     | 80     | 33     |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 012 | 39      | 23     | 10     |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 018 | 0       | 80     |        |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 019 | 0       | 39     |        |



# 3 — Importar el CSV en SQL Server

Primero importamos el CSV crudo en la tabla **INEGI_Censo_2020_AGEB.**
Esta tabla refleja exactamente la estructura del archivo exportado desde **SCITEL.**

```sql
------------------------------------
-- Importar CSV en SQL
--
-- Crear tabla INEGI_Censo_2020_AGEB
-- (Censo 2020 por AGEB y Manzana)
------------------------------------
DROP TABLE IF EXISTS INEGI_Censo_2020_AGEB;
GO

CREATE TABLE INEGI_Censo_2020_AGEB (
    ENTIDAD varchar(2) NOT NULL,
    NOM_ENT nvarchar(100) NULL,
    MUN varchar(3) NOT NULL,
    NOM_MUN nvarchar(100) NULL,
    LOC varchar(4) NOT NULL,
    NOM_LOC nvarchar(150) NULL,
    AGEB varchar(4) NOT NULL,
    MZA varchar(3) NOT NULL,
    POBTOT int NULL,
    VIVTOT int NULL,
    TVIVHAB int NULL, 
);
GO

--------------
-- Bulk Insert
--------------
BULK INSERT INEGI_Censo_2020_AGEB_Staging
FROM 'D:\AXSI\INEGI\Censo_2020\Tabulados_AGEB_Manzana\RESAGEBURB2020_ALL_TAB.csv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = '\t',
    ROWTERMINATOR = '\n',
    CODEPAGE = '65001',  -- UTF-8
    TABLOCK
);
GO
```

#### Resultado esperado

(863069 registros)      


Consulta de prueba:

```sql
------------------------
-- Listar los primeros 10 registros
------------------------
SELECT TOP (10)
    ENTIDAD,
    NOM_ENT,
    MUN,
    NOM_MUN,
    LOC,
    NOM_LOC,
    AGEB,
    MZA,
    POBTOT,
    VIVTOT,
    TVIVHAB
FROM dbo.INEGI_Censo_2020_AGEB;
```

| ENTIDAD | NOM_ENT (state)|     MUN | NOM_MUN (municipality)             |     LOC | NOM_LOC (locality)           | AGEB | MZA | POBTOT (population) | VIVTOT (dwellings) | TVIVHAB (occupied dwellings) |
|---------|----------------|---------|------------------------------------|---------|------------------------------|------|-----|--------|--------|---------|
| 01      | Aguascalientes | 000     | Total de la entidad Aguascalientes | 0000    | Total de la entidad          | 0000 | 000 | 1425607 | 463972 | 386671 |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0000    | Total del municipio          | 0000 | 000 | 948990  | 313256 | 266942 |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Total de la localidad urbana | 0000 | 000 | 863893  | 286646 | 246259 |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Total AGEB urbana            | 0017 | 000 | 2237    | 1288   | 648    |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 011 | 115     | 80     | 33     |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 012 | 39      | 23     | 10     |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 018 | 0       | 80     |        |
| 01      | Aguascalientes | 001     | Aguascalientes                     | 0001    | Aguascalientes               | 0017 | 019 | 0       | 39     |        |



# 4 — Actualizar Population y Dwellings en 'Boundaries_AGEB_2025'

```sql
----------------------------------------------------------------------------------------------------------
-- Censo 2020 Paso 3.4 — Actualizar Population y Dwellings en Boundaries_AGEB_2025
--
-- NOTA:
-- Los polígonos de AGEB son 2025; el Censo 2020 es a nivel manzana, por lo que solo cubre áreas urbanas
-- (no rurales). INEGI NO publica Censo a nivel AGEB.
--
-- El Censo a nivel Localidad (tipo 9) sí existe, pero puede NO coincidir con las Colonias del Boundaries.
-- Aun así, lo evaluaremos más adelante.
--
-- En este paso enriquecemos Boundaries_AGEB_2025 con población y viviendas del Censo 2020 para ver si
-- mejora la interpolación AMAI.
--
-- Registros que se actualizarán con Population: 35,668
-- AGEB urbanas no actualizadas (sin datos de manzana): ~29,140
--
-- También probaremos si el Censo 2020 a nivel Localidad coincide con Neighborhood o AGEMLL 2025.
----------------------------------------------------------------------------------------------------------

DROP TABLE IF EXISTS INEGI_Censo_2020_AGEB_SUMMARY;
GO

SELECT
    RIGHT('00' + ENTIDAD, 2) +
    RIGHT('000' + MUN, 3) +
    RIGHT('0000' + LOC, 4) +
    RIGHT('0000' + AGEB, 4) AS CVEGEO,
    SUM(VIVTOT) AS VIVTOT,
    SUM(TVIVHAB) AS TVIVHAB,
    SUM(POBTOT) AS POBTOT
INTO INEGI_Censo_2020_AGEB_SUMMARY
FROM INEGI_Censo_2020_AGEB
WHERE LOC <> '0000'
  AND AGEB <> '0000'
  AND MZA <> '000'
GROUP BY
    RIGHT('00' + ENTIDAD, 2) +
    RIGHT('000' + MUN, 3) +
    RIGHT('0000' + LOC, 4) +
    RIGHT('0000' + AGEB, 4);


-- Actualizar Population y Dwellings en Boundaries_AGEB_2025 desde Censo 2020

UPDATE B
SET 
    B.Population = C.POBTOT,
    B.Dwellings = C.VIVTOT,
    B.Occupied_Dwellings = C.TVIVHAB
FROM Boundaries_AGEB_2025 B
LEFT JOIN INEGI_Censo_2020_AGEB_SUMMARY C
    ON B.CVEGEO = C.CVEGEO;
GO


-- ¿Cuántas AGEB se actualizan con datos?

SELECT COUNT(*) AS AGEB_con_Population 
FROM Boundaries_AGEB_2025
WHERE Population IS NOT NULL;

-- Revisar algunas AGEB urbanas que no tienen datos

SELECT COUNT(*) AS AGEB_sin_Population 
FROM Boundaries_AGEB_2025
WHERE Population IS NULL;

--------------------------------------------------------------
-- Ver algunas AGEB urbanas que no tienen datos del Censo 2020
--------------------------------------------------------------

SELECT CVEGEO, Type, Population, Dwellings, Occupied_Dwellings
FROM Boundaries_AGEB_2025
WHERE Population IS NULL
AND Type = 'Urbana'
ORDER BY CVEGEO;

-- Borra tabla Summary

DROP TABLE IF EXISTS INEGI_Censo_2020_AGEB_SUMMARY;
GO
```


# 5 — Validaciones Finales

Después de cargar la tabla final, ejecutamos un conjunto de consultas de validación para confirmar:

- Que se escribió el número esperado de registros
- Que todos los códigos CVEGEO se generaron correctamente con 16 dígitos
- Que la tabla de staging puede eliminarse de forma segura

### ✔ Validar el conteo de registros

```sql
----------------
-- Contar registros
----------------
SELECT COUNT(*) AS Records_Written
FROM INEGI_Censo_2020_AGEB;
```

#### Resultados esperados

Records_Written: 863069
Esto confirma que todas las filas a nivel manzana de los 32 estados fueron importadas correctamente (mismo conteo que en el CSV consolidado).

### ✔ Validar formato de CVEGEO (16 dígitos)

```sql
-----------------------------------------
-- Validar que CVEGEO tiene 16 caracteres
-----------------------------------------
SELECT 
    COUNT(*) AS CVEGEO_InvalidLength
FROM INEGI_Censo_2020_AGEB
WHERE LEN(CVEGEO) <> 16;
```

#### Resultados esperados

|CVEGEO|Len|
|---------------|-----|
|2402800014141040|	16|
|2402800014141043|	16|
|2402800014141044|	16|
|2402800014141045|	16|
|2402800014141046|	16|
|2402800014141047|	16|
|2402800014141048|	16|
|2402800014141049|	16|
|2402800014141050|	16|
|2402800014141051|	16|

Esto confirma que:

- Todos los códigos concatenados correctamente producen 16 caracteres exactos
- No hay dígitos faltantes
- No existen valores CVEGEO malformados
