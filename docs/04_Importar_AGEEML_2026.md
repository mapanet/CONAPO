# 04 — INEGI AGEEML 2026

Este dataset contiene los **catálogos oficiales de códigos y nombres de Entidad, Municipio y Localidad**, además de las **coordenadas de latitud y longitud** para cada localidad.   
Se utiliza en procesos donde los datos llegan sin nombres, **sin claves completas o sin georreferencia**, y sirve como tabla maestra para normalizar y enriquecer información territorial.

## Tabla resultante: `INEGI_AGEEML_2026`

| Columna | Tipo | Notas |
|--------|------|--------|
| CVEGEO | varchar(16) | **Llabe principal** |
| Status | nvarchar(20) | null or `Baja` (fuera de servicio) |
| ISO | varchar(2) | 'MX' (ISO country code) |
| Country | nvarchar](20) | 'Mexico'
| State | nvarchar(85) ||
| Municipality | nvarchar(85) ||
| City | nvarchar(110) | Localidad |
| Type | varchar](1) | 'U' or 'R' (Urbana o Rural |
| Latitude | decimal(15, 6) |  ESPG:4023 |
| Longitude | decimal(15, 6) | ESPG:4023 |
| Altitude | int ||
| geom | geometry | Point ESPG:4023 |
| geog | geography | Point ESPG:4023 |
| Population | int ||
| Dwellings | int |
| CVE_ENT | varchar(2) | Código Estado |
| CVE_MUN | varchar(3) | Código Municipio |
| CVE_LOC | varchar(4) | Código Localidad |
| State_Ant	| nvarchar(85) | nombre original de Estado (para almacenar nombre si renombrado/abreviado) |
| Municipality_Ant | nvarchar(85) | Nombre Municipio original |
| City_Ant | nvarchar(110) | Nombre Localidad original |

## Carpetas de trabajo:

D:\INEGI\AGEEML_2026   
D:\INEGI\AGEEML_2026\Download   

---

# 1 — Descargar Catálogo AGEEML 2026

- **URL:**  [https://www.inegi.org.mx/app/ageeml/#](https://www.inegi.org.mx/app/ageeml/#)   
- **Sección:** Catalogos completos (complete catalogs)   
- **Catálogo principal:** Catálogo de Localidades Nacional ( 296704 Localidades) Fecha de corte: 2026/04   
- **Detalle del archivo:** Minúscula con acento, incluye bajas (ProperCase with accents, included old deleted   

[<img src="/docs/images/INEGI_AGEEEML.png" width="1000">](/docs/images/INEGI_AGEEEML.png)

Archivo descargado: 

**Directorio:** `D:\INEGI\AGEEML_2026\Download\`   
**Archivo:** min_con_acento_baja.zip

Extract from ZIP to working directory:   

D:\INEGI\AGEEML_2026\AGEEML_202651313653_utf.csv



# 2 — Convertir a CSV antes de importar a SQL 

### Propósito

El catálogo AGEEML 2026 viene en un archivo ZIP con un CSV que contiene muchos campos que no necesitamos y valores como `-` y `*` que representan **N/A**.
Antes de importar a SQL Server, generamos un archivo TSV (TAB‑delimited) limpio, con:

- `NOM_ABR` — Nombre abreviado de la entidad
- `LATITUD` (HH MM SS)
- `LONGITUDE`  (HH MM SS)
- `CVE_CARTA` (Referencia de mapa INEGI)

AGEEML usa: 

- '*`
- `-`

En campos como Población y Viviendas, indicando que no existe información.   
En SQL Server, estos deben convertirse en: **NULL**


### Script utilizado

El script completo que realiza esta limpieza está disponible aquí:

[Convert_to_CSV_TSV.ps1](../scripts/Convert_to_CSV_TSV.ps1)

Este script:

- Lee el CSV original del catálogo AGEEML
- Procesa cada fila
- Normaliza valores
- Exporta un TSV limpio y consistente

**D:\INEGI\AGEEML_2026\AGEEML_2026.tsv**

Columnas de ejemplo:

|CVEGEO|Status|CVE_ENT|State|CVE_MUN|Municipality|CVE_LOC|City|Type|Latitude|Longitude|Altitude|Population|Population_M|Population_F|Occupied_Dwellings|
|------|------|-------|-----|-------|------------|-------|----|----|--------|---------|--------|----------|------------|------------|------------------|
|010010001||01|Aguascalientes|001|Aguascalientes|0001|Aguascalientes|U|21.879822|102.296046|1878|863893|419168|444725|246259|
|010010094||01|Aguascalientes|001|Aguascalientes|0094|Granja Adelita|R|21.871874|102.37353|1901|5|||2|
|010010096||01|Aguascalientes|001|Aguascalientes|0096|Agua Azul|R|21.883756|102.357122|1861|41|24|17|12|
|010010100||01|Aguascalientes|001|Aguascalientes|0100|Rancho Alegre|R|21.854683|102.372731|1879|0|0|0|0|
|010010102||01|Aguascalientes|001|Aguascalientes|0102|Los Arbolitos [Rancho]|R|21.78018|102.357295|1861|8|||2|
|010010104||01|Aguascalientes|001|Aguascalientes|0104|Ardillas de Abajo (Las Ardillas)|R|21.945067|102.19192|1994|1|||1|
|010010106||01|Aguascalientes|001|Aguascalientes|0106|Arellano|R|21.801773|102.273954|1891|1169|613|556|281|
|010010112||01|Aguascalientes|001|Aguascalientes|0112|Bajío los Vázquez|R|21.747494|102.124816|1971|41|20|21|9|
|010010113||01|Aguascalientes|001|Aguascalientes|0113|Bajío de Montoro|R|21.757883|102.290131|1871|0|0|0|0|
|010010114|Baja|01|Aguascalientes|001|Aguascalientes|0114|Residencial San Nicolás [Baños la Cantera]|R|21.849498|102.355422|1859|||||
|010010120||01|Aguascalientes|001|Aguascalientes|0120|Buenavista de Peñuelas|R|21.719147|102.293195|1871|1054|542|512|255|
|010010121||01|Aguascalientes|001|Aguascalientes|0121|Cabecita 3 Marías (Rancho Nuevo)|R|21.774682|102.412992|1905|192|92|100|47|




# 3 — Crear INEGI_AGEEML_2026_Staging e importar datos   

Después de generar el archivo limpio **TSV** (ver sección anterior), el siguiente paso es crear la tabla staging en SQL Server.
Esta tabla sirve como área temporal para:

- Cargar el archivo TSV sin transformaciones complejas
- Validar tipos, longitudes y valores nulos
- Confirmar que los códigos ENTIDAD, MUN y LOC están completos
- Verificar que LAT_DEC y LON_DEC son numéricos
- Preparar la generación del CVEGEO (9 dígitos) en la tabla final

La tabla staging siempre debe ser:

- Simple
- Sin índices
- Sin constraints
- Sin claves primarias

Esto garantiza que el BULK INSERT sea rápido y sin bloqueos.

```sql
-----------------------------------------
-- Crear INEGI_AGEEML_2026_Staging 
-----------------------------------------
DROP TABLE IF EXISTS dbo.INEGI_AGEEML_2026_Staging;
GO

CREATE TABLE [dbo].[INEGI_AGEEML_2026_Staging](
    [CVEGEO] [nvarchar](16) NOT NULL,
    [Status] [nvarchar](20) NULL,
    [CVE_ENT] [varchar](2) NULL,
    [State] [nvarchar](85) NOT NULL,
    [CVE_MUN] [varchar](3) NULL,
    [Municipality] [nvarchar](85) NOT NULL,
	[CVE_LOC] [varchar](4) NULL,
    [City] [nvarchar](110) NOT NULL,
    [Type] [varchar](1) NOT NULL,
    [Latitude] [decimal](15, 6) NOT NULL,
    [Longitude] [decimal](15, 6) NOT NULL,
    [Altitude] [int] NOT NULL,
    [Population] [int] NULL,
    [Population_M] [int] NULL,
    [Population_F] [int] NULL,
    [Occupied_Dwellings] [int] NULL
    CONSTRAINT [PK_INEGI_AGEML_2026_Staging] PRIMARY KEY CLUSTERED ([CVEGEO] ASC)
) ON [PRIMARY];
GO

--------------
-- Bulk insert
--------------
BULK INSERT INEGI_AGEEML_2026_staging
FROM 'D:\AXSI\INEGI\AGEEML_2026\AGEEML_2026.tsv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = '\t',
    ROWTERMINATOR = '\n',
    CODEPAGE = '65001',  -- UTF-8
    TABLOCK
);
GO
```

### Resultado esperado

(361168 rows)   


# 4 — Crear tabla final INEGI_AGEEML_2026   

La tabla final **INEGI_AGEEML_2026** es la versión depurada y normalizada del catálogo AGEEML 2026.  
A diferencia de la tabla staging, esta tabla:  

- Incluye campos adicionales para conservar los nombres originales antes de normalizar.
- Genera el CVEGEO (9 dígitos).
- Incluye geometry y geography para usos espaciales.
- Aplica tipos de datos definitivos y restricciones mínimas.
- Está lista para integrarse con Boundaries Layer 5 y Layer 6 del pipeline NSE.
	
```sql
--------------------------------------
-- Crear tabla final INEGI_AGEEML_2026  
--------------------------------------
DROP TABLE IF EXISTS dbo.INEGI_AGEEML_2026;
GO

CREATE TABLE [dbo].[INEGI_AGEEML_2026](
	[CVEGEO] [nvarchar](16) NOT NULL,
	[Status] [nvarchar](20) NULL,
	[ISO] [varchar](2) NULL,
	[Country] [nvarchar](20) NULL,
	[State] [nvarchar](85) NOT NULL,
	[Municipality] [nvarchar](85) NOT NULL,
	[City] [nvarchar](110) NOT NULL,
	[Type] [nvarchar](1) NOT NULL,
	[Latitude] [decimal](15, 6) NOT NULL,
	[Longitude] [decimal](15, 6) NOT NULL,
	[Altitude] [int] NOT NULL,
	[geom] [geometry] NULL,
	[geog] [geography] NULL,
	[Population] [int] NULL,
	[Population_M] [int] NULL,
    [Population_F] [int] NULL,
	[Occupied_Dwellings] [int] NULL,
	[CVE_ENT] [varchar](2) NULL,
	[CVE_MUN] [varchar](3) NULL,
	[CVE_LOC] [varchar](4) NULL,
	[State_Ant] [nvarchar](85) NULL,
	[Municipality_Ant] [nvarchar](85) NULL,
	[City_Ant] [nvarchar](110) NULL
 CONSTRAINT [PK_INEGI_AGEEML_2026] PRIMARY KEY CLUSTERED 
(
	[CVEGEO] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO

ALTER TABLE [dbo].[INEGI_AGEEML_2026] ADD  CONSTRAINT [DF_INEGI_AGEEML_2026_ISO]  DEFAULT (N'MX') FOR [ISO]
GO

ALTER TABLE [dbo].[INEGI_AGEEML_2026] ADD  CONSTRAINT [DF_INEGI_AGEEML_2026_Country]  DEFAULT (N'México') FOR [Country]
GO
```


# 5 — Copiar datos desde la tabla Staging

```sql
------------------------------------------
-- Copiar datos desde la tabla Staging
------------------------------------------
INSERT INTO INEGI_AGEEML_2026 (
    CVEGEO,
    Status,
    CVE_ENT,
    State,
    CVE_MUN,
    Municipality,
    CVE_LOC,
    City,
    Type,
    Latitude,
    Longitude,
    Altitude,
    Population,
    Population_M,
    Population_F,
    Occupied_Dwellings
)
SELECT
    CVEGEO,
    Status,
    CVE_ENT,
    State,
    CVE_MUN,
    Municipality,
    CVE_LOC,
    City,
    Type,
    Latitude,
    Longitude,
    Altitude,
    Population,
    Population_M,
    Population_F,
    Occupied_Dwellings
FROM INEGI_AGEEML_2026_Staging;
```

#### Resultados esperados

(361168 rows affected)   
Todos los registros copiados

### Finalmente borra tabla staging

```sql
DROP TABLE IF EXISTS dbo.INEGI_AGEEML_2026_Staging;
GO
```


# Copiar nombres originales a campos `_Ant`

```sql
---------------------------------------------------------------------------------
-- Copiar nombres originales a campos `_Ant`
---------------------------------------------------------------------------------

Update INEGI_AGEEML_2026 set 
  State_Ant = State,
  Municipality_Ant = Municipality,
  City_Ant = City

--------------------------------------
-- Update state names to short version
--------------------------------------

Update INEGI_AGEEML_2026 set State = 'Coahuila' WHERE State = 'Coahuila de Zaragoza' AND CVE_ENT = '05'
Update INEGI_AGEEML_2026 set State = 'Michoacán' WHERE State = 'Michoacán de Ocampo' AND CVE_ENT = '16'
Update INEGI_AGEEML_2026 set State = 'Veracruz' WHERE State = 'Veracruz de Ignacio de la Llave' AND CVE_ENT = '30'
```

#### Resultados esperados

(12492 rows affected)   
(14318 rows affected)   
(28958 rows affected)  



# 7 — 7 — Crear geom (geometry) y geog (geography) a partir de latitude y longitude

Para permitir intersecciones espaciales, validaciones territoriales, cruces con Boundaries Layer 5 y Layer 6, y consultas geoespaciales en SQL Server, es necesario convertir las coordenadas LAT_DEC y LON_DEC en objetos espaciales:

- geometry → operaciones cartesianas
- geography → operaciones geodésicas (curvatura de la Tierra)

```sql
--------------------------------------------------------------------------------
-- Create geom como POINT (geometry, SRID 4326) desde Latitude y Longitude
-- geometry::Point(X,Y,4326) → X = Lon, Y = Lat
--------------------------------------------------------------------------------
UPDATE INEGI_AGEEML_2026
SET geom = geometry::Point(Longitude, Latitude, 4326);

----------------
-- Validar geom
----------------
SELECT CVEGEO as CVGGEO_Invalid
FROM INEGI_AGEEML_2026
WHERE geom.STIsValid() = 0;
```

#### Expect results

361168 geom (geometrias) creadas   
Ningún CVEGEO_Invalid   

Si alguna geometría es inválida, checar porque:

```sql
---------------------------
-- Si alguna geometría es inválida, checar porque
---------------------------
SELECT CVEGEO, geom.STIsValid(), geom.IsValidDetailed()
FROM INEGI_AGEEML_2026
WHERE geom.STIsValid() = 0;
```

### Copy geom (geometry) to geog (geography)

```sql
---------------------------------------------
-- Copy geom (geometry) to geog (geography)
---------------------------------------------
UPDATE INEGI_AGEEML_2026
SET geog = geography::Point(Latitude, Longitude, 4326);

----------------
-- Valida geog
----------------
SELECT CVEGEO As CVEGEO_Invalid
FROM INEGI_AGEEML_2026
WHERE geog.STIsValid() = 0;
```

#### Resultados esperados

361168 geog (geography geometrías) creadas
No CVEGEO_Invalid   



# 8 — Final Validations

✔Validate Record Count

```sql
----------------------
-- Conteo de registros
----------------------
SELECT COUNT(*) AS Records_Written
FROM INEGI_AGEEML_2026;
```

#### Resultados esperados

Registros: 361168
Esto confirma de que todos los egistros del archivo CSV fueron importados.

### ✔ Visualizar los datos

```sql
SELECT TOP 20 
CVEGEO, 
Status, 
ISO, 
Country, 
State, 
Municipality, 
City, 
Type, 
Latitude, 
Longitude, 
Altitude, 
geom, 
geog, 
Population, 
Population_M, 
Population_F, 
Occupied_Dwellings, 
CVE_ENT, 
CVE_MUN, 
CVE_LOC, 
State_Ant, 
Municipality_Ant, 
City_Ant
FROM dbo.INEGI_AGEEML_2026
```
