# INEGI Importación de datos terroriales INEGI 

## Objectivo

Este repositorio contiene los procedimientos oficiales utilizados para importar, normalizar y preparar datasets de INEGI, AGEEML, CONAPO y Censo 2020, con el objetivo de enriquecer las capas territoriales de Boundaries con información demográfica y territorial confiable.

Los scripts y documentos dentro de /docs describen paso a paso cómo obtener:

- Población por localidad (Censo 2020)
- Población por AGEB (Censo 2020)
- Viviendas totales y ocupadas
- Localidades oficiales (AGEEML 2026)
- Municipios y estados normalizados
- Proyecciones de población CONAPO 2020–2026
- Crecimiento poblacional desde el Censo 2020 a la fecha
- Metadatos territoriales para integración con Boundaries

Estos datos se utilizan para enriquecer tabla Boundaries (delimitacion de colonias) creada en el reporitiorio: https://github.com/mapanet/NSE   

- Layer 6 — Colonias
- Layer 5 — Ciudades
- Layer 2 — Municipios
- Layer 1 — Estados

### Índice de Documentación

[01 Importar INEGI Censo 2020 (nivel localidad)](docs/01_Importar_INEGI_Censo_2020_nivel_localidad.md)   

- Población total
- Viviendas totales
- Viviendas habitadas
- Claves de localidad, municipio y estado
- Normalización de CVEGEO y claves territoriales
- Uso para enriquecer Boundaries con población por localidad

[02 Importar_INEGI_Censo_2020_AGEB](docs/02_Importar_INEGI_Censo_2020_AGEB.md)   

- Población por AGEB
- Viviendas totales y ocupadas
- Variables demográficas clave
- Normalización de CVEGEO
- Uso para interpolación AGEB ↔ colonia (NSE y población)

[03 Importar_Catalogo de Localidades_2025](docs/03_Importar_Localidades_2025.md)   

- Nombres oficiales de localidades
- Claves de municipio y estado
- Tipo de localidad (urbana/rural)
- Integración con Boundaries para etiquetado territorial
- Base para fallback rural en NSE

[04 Importar_AGEEML_2026](docs/04_Importar_AGEEML_2026.md)   

Importación del AGEEML 2026 (Localidades y Municipios):

- Nombres oficiales de municipios y estados
- Claves normalizadas
- Localidades actualizadas 2026
- Base territorial para Boundaries
- Corrección de nombres y metadatos en capas 1–5

[05 Importar proyecciones CONAPO 2020–2026](docs/05_Importar_CONAPO_Population.md)   

- Población estimada por municipio
- Crecimiento anual
- Proyección desde Censo 2020 a 2026
- Cálculo de población actualizada para Boundaries
- Integración con capas municipales y estatales

## Objetivo del Repositorio

Este repositorio existe para:

1. Centralizar todos los procedimientos de importación de datos oficiales 
INEGI, AGEEML, CONAPO y Censo 2020.

2. Normalizar claves territoriales

- CVEGEO
- Claves de localidad
- Claves de municipio
- Claves de estado

3. Generar datasets consistentes para Boundaries (colonias) Incluyendo:

- Population
- Dwellings
- Occupied_Dwellings
- Growth_2020_2026
- Locality
- Municipalit
- State

4. Mantener trazabilidad y reproducibilidad

Cada documento explica:

- Fuente oficial
- Pasos de importación
- Normalización
- Validaciones
- Integración con SQL y geoprocesos

5. Servir como base para pipelines mayores Como:

- NSE AMAI por colonia ( https://github.com/mapanet/NSE )
- Crecimiento poblacional municipal
- Actualización de capas territoriales

## Integración con Boundaries

Los datos importados aquí alimentan:

**Layer 6 — Colonias**
Interpolación AGEB ↔ colonia para población y viviendas.

**Layer 5 — Ciudades**
Agregación por localidad y municipio.

**Layer 2 — Municipios**
Población actualizada con CONAPO.

**Layer 1 — Estados**
Agregación estatal y metadatos AGEEML.




