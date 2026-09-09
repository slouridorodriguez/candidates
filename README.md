# candidates

ETL para el Workshop 1 del curso. Toma el CSV de 50,000 candidatos, lo limpia, arma un modelo estrella y lo carga en una base SQLite para poder sacar KPIs de contratación sin volver a tocar el CSV.

## Estructura

- `data/` -> el CSV original y la base `.sqlite` que genera el pipeline.
- `pipelines-etl/` -> `01_etl_pipeline.ipynb` (Extract, Transform, Load) y `02_kpis_visualizaciones.ipynb` (queries de KPIs y gráficos).
- `diagrams/` -> el diagrama del star schema.

## Cómo correrlo

Los notebooks están pensados para Colab. Hay que montar Google Drive y ajustar `ruta_base` a la carpeta donde se tenga el CSV. El orden importa: primero `01_etl_pipeline.ipynb` (crea y llena la base `.sqlite`), y después `02_kpis_visualizaciones.ipynb`, que se conecta a esa misma base para sacar los KPIs — no vuelve a tocar el CSV.

## Regla de negocio

Un candidato queda como contratado (`is_hired`) si `Code Challenge Score >= 7` y `Technical Interview >= 7`. Si el Technical Interview vino nulo, no cumple la condición y queda como no contratado, sin inventar el dato.

## Diseño del modelo

La tabla de hechos (`fact_postulaciones`) guarda solo las métricas por postulación (yoe, los dos puntajes, si quedó contratado). Tecnología, seniority y fecha quedan como dimensiones aparte porque son los criterios con los que filtro los KPIs. País se dejó dentro de `dim_candidatos` porque no tiene jerarquía propia y es un solo dato por candidato. El diagrama completo está en `diagrams/`.

## KPIs usados

- Contratados por tecnología
- Contratados por año
- Contratados por seniority
- Contratados por país a lo largo de los años (USA, Brasil, Colombia, Ecuador)

## Decisiones

Durante el desarrollo se tomaron las siguientes decisiones técnicas:

* **Code Challenge Score fuera de rango:** se encontraron 1,000 filas (2% del dataset) con Code Challenge Score = 100, fuera del rango lógico de 0-10. Al ser una cantidad significativa y no un caso aislado, se asumió error de captura del puntaje y se corrigió al máximo válido (10) en lugar de descartar las filas.

* **Valores centinela en Yoe:** se encontraron 500 filas con Yoe = -5.0 y otras 500 con Yoe = 99.0 - valores exactos y repetidos, no una dispersión típica de errores de tipeo, lo que indica que representan un código de "dato faltante" disfrazado de número. Como Yoe no participa en la regla HIRED, el impacto es solo de calidad de datos, no de los KPIs. Se trataron como NaN en vez de inventar un valor de experiencia real.

* **Columnas categóricas (Country, Technology, Seniority):** se revisaron los valores únicos de cada una en busca de inconsistencias de texto (mayúsculas/minúsculas distintas, espacios de más). No se encontraron duplicados de este tipo.

* **Nulos en Seniority:** se rellenaron con "Unknown" en vez de eliminarse, para no perder la información de puntajes de esos candidatos.

* **Technical Interview nulo:** no se rellena ni se asume un valor - al no cumplir la condición `>= 7`, el candidato queda automáticamente excluido de HIRED sin necesidad de una regla aparte.
