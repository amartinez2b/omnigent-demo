# Omnigent Demo - Data Pipeline

Laboratorio completo de ingeniería de datos implementando arquitectura Medallion (Bronze → Silver → Gold) con PySpark y un dashboard interactivo.

## Descripción General

Este proyecto implementa un pipeline analítico sobre las tablas de la base de datos MySQL `fake`:
- **customers**: Dimensión de clientes
- **products**: Dimensión de productos
- **sales**: Tabla de hechos (ventas)
- **shops**: Dimensión de sucursales

## Arquitectura

### Bronze Layer (data/bronze/)
- Ingesta cruda de MySQL
- Almacenamiento en Parquet
- Columnas de auditoría: `_ingest_date`, `_table_source`, `_ingest_batch_id`

### Silver Layer (data/silver/)
- Limpieza y normalización
- Eliminación de duplicados
- Validación de nulos en claves
- Casteo de tipos
- Columnas de auditoría: `_load_date`, `_layer`

### Gold Layer (data/gold/)
- Tabla **comercial**: unión de todas las dimensiones con métricas
- Tablas de agregados por periodo, tienda, producto y cliente
- Métricas calculadas: ventas totales, unidades, ticket promedio

### Dashboard (dashboard/comercial_dashboard.html)
- Indicadores KPI: ventas totales, unidades vendidas
- Gráficos de ventas por tienda, producto y cliente
- Tendencias temporales
- Generado con Plotly

## Requisitos

### Software
- Python 3.8+
- Java 8 o 11 (para Spark y JDBC)
- PySpark 3.0+

### Librerías Python
```bash
pip install pyspark plotly pandas mysql-connector-python python-dotenv
```

### Driver JDBC
El driver MySQL JDBC se configura automáticamente via Maven:
```
mysql:mysql-connector-java:8.0.33
```

## Configuración

### Variables de Entorno (.env)
Crea un archivo `.env` en la raíz del proyecto:

```env
# MySQL Configuration
MYSQL_HOST=www.bigdataybi.com
MYSQL_PORT=3306
MYSQL_DATABASE=fake
MYSQL_USER=curso
MYSQL_PASSWORD=Password123!

# Paths
BRONZE_PATH=data/bronze
SILVER_PATH=data/silver
GOLD_PATH=data/gold
DASHBOARD_PATH=dashboard/comercial_dashboard.html
```

## Estructura del Proyecto

```
omnigent-demo/
├── src/
│   ├── config.py                    # Configuración centralizada
│   ├── bronze_ingestion.py          # Módulo de ingesta MySQL
│   ├── silver_transformations.py    # Módulo de transformaciones
│   ├── gold_comercial.py            # Módulo de tabla comercial
│   ├── dashboard_builder.py         # Módulo de dashboard
│   └── main.py                      # Orquestador principal
├── notebooks/
│   └── omnigent_demo_pipeline.py    # Notebook compatible con Databricks
├── data/
│   ├── bronze/                      # Datos crudos (Parquet)
│   ├── silver/                      # Datos limpios (Parquet)
│   └── gold/                        # Tablas analíticas (Parquet)
├── dashboard/
│   └── comercial_dashboard.html     # Dashboard interactivo
├── outputs/
│   ├── data_model.md                # Análisis de modelo de datos
│   ├── join_strategy.md             # Estrategia de joins
│   ├── medallion_flow.md            # Flujo medallion
│   └── work_plan.md                 # Plan de trabajo
├── .env                             # Variables de entorno
└── README.md                        # Este archivo
```

## Ejecución

### Opción 1: Script Principal
```bash
cd omnigent-demo
python src/main.py
```

### Opción 2: Notebook Databricks
En un cluster Databricks, carga y ejecuta:
```
%run ./notebooks/omnigent_demo_pipeline.py
```

### Opción 3: PySpark Interactivo
```bash
python -m pyspark

from src.main import run_pipeline
run_pipeline()
```

## Ejecución por Capas

Si prefieres ejecutar por capas individualmente:

```python
from pyspark.sql import SparkSession
from src.bronze_ingestion import BronzeIngestion
from src.silver_transformations import SilverTransformations
from src.gold_comercial import GoldComercial
from src.dashboard_builder import DashboardBuilder

spark = SparkSession.builder.appName("omnigent-demo").getOrCreate()

# Bronze
bronze = BronzeIngestion(spark)
bronze.ingest_all_tables()

# Silver
silver = SilverTransformations(spark)
silver.transform_all_tables()

# Gold
gold = GoldComercial(spark)
gold_result = gold.build_gold_layer()

# Dashboard
dashboard = DashboardBuilder()
df_comercial = dashboard.read_comercial(spark)
dashboard.build_dashboard(df_comercial)
```

## Problemas Conocidos y Soluciones

### 1. MySQL no disponible
- **Síntoma**: Error de conexión JDBC
- **Solución**: El código maneja esto gracefully, generando DataFrames vacíos y documentando el bloqueo en los logs
- **Impacto**: Se generan todas las transformaciones pero sin datos

### 2. Driver MySQL JDBC no encontrado
- **Síntoma**: `ClassNotFoundException: com.mysql.cj.jdbc.Driver`
- **Solución**: PySpark debe descargar automáticamente el JAR. Si no, descárgalo manualmente:
```bash
# Opción manual
wget https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-8.0.33.jar
```

### 3. Memoria insuficiente en Spark
- **Solución**: Aumentar memoria en SparkSession:
```python
spark = SparkSession.builder \
    .config("spark.driver.memory", "4g") \
    .config("spark.executor.memory", "4g") \
    .getOrCreate()
```

### 4. Sucursal desconocida (cod_sucursal = 4)
- **Hallazgo**: 20 de 100 ventas refieren a `cod_sucursal = 4`, que no existe en shops
- **Tratamiento**: En silver, se agregó un registro "Unknown Shop" con id_sucursal = 999 para el LEFT JOIN
- **Reconciliación**: 
  - INNER JOINs: 80 filas
  - LEFT JOINs: 100 filas

## Métricas Esperadas

### Sumario Comercial
- **Total de Ventas**: ~$XXXXX (depende de datos)
- **Total de Unidades**: ~XXX
- **Número de Transacciones**: 100
- **Ticket Promedio**: ~$XXX
- **Tiendas Activas**: 3 (+ 1 desconocida)
- **Productos Activos**: 5
- **Clientes Activos**: 10

## Dashboard

El dashboard incluye 6 visualizaciones:

1. **Total Sales (KPI)**: Ingresos brutos totales
2. **Units Sold (KPI)**: Total de unidades vendidas
3. **Sales by Shop**: Gráfico de barras horizontal
4. **Sales by Product (Top 10)**: Gráfico de barras horizontal
5. **Sales by Customer (Top 10)**: Gráfico de barras horizontal
6. **Sales Over Time**: Línea de evolución diaria

Acceder: `dashboard/comercial_dashboard.html`

## Monitoreo y Logging

Todos los módulos escriben logs con timestamps y niveles:
- INFO: Operaciones normales
- WARNING: Situaciones anómalas (ej: tablas vacías)
- ERROR: Fallos que impiden continuar

Revisar salida estándar o configurar handlers de logging personalizados.

## Rendimiento

### Tiempos típicos (con datos en memoria):
- Bronze: ~5-10 segundos
- Silver: ~3-5 segundos
- Gold: ~2-3 segundos
- Dashboard: ~2-3 segundos
- **Total**: ~15-25 segundos

### Optimizaciones aplicadas:
- Particionamiento automático de Spark (`spark.sql.shuffle.partitions = 4`)
- Adaptive Query Execution habilitado
- Coalesce(1) para escribir un único archivo Parquet por tabla
- Pandas conversion only on final agregates

## Extensiones Futuras

- [ ] Validaciones DQX (Data Quality eXtended)
- [ ] Tabla de auditoría con SCD Type 2 para cambios en dimensiones
- [ ] Incremental load soportando cambios
- [ ] Alertas en casos de integridad referencial
- [ ] Pruebas unitarias con pytest y PySpark testing
- [ ] API REST para consultas analíticas
- [ ] Integración con Power BI o Tableau

## Créditos

Laboratorio desarrollado como parte del programa de capacitación BigDataYBI usando:
- **PySpark** para transformaciones distribuidas
- **Plotly** para visualizaciones interactivas
- **MySQL** como fuente de datos
- **Omnigent** para orquestación de agentes

## Licencia

Proyecto educativo. Libre para uso educativo y comercial.

## Contacto

Para preguntas o mejoras, contactar al equipo de BigDataYBI.
