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
