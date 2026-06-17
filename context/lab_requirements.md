# Laboratorio omnigent-demo

## Objetivo

Construir un pipeline PySpark que ingeste todas las tablas de la base MySQL `fake`:

- customers
- products
- sales
- shops

El pipeline debe implementar arquitectura medallion:

## Bronze

Ingestar tablas crudas desde MySQL y guardarlas en formato Parquet.

Tablas esperadas:

- data/bronze/customers
- data/bronze/products
- data/bronze/sales
- data/bronze/shops

## Silver

Limpiar y estandarizar datos:

- Normalizar nombres de columnas.
- Convertir tipos de datos.
- Eliminar duplicados.
- Validar nulos en claves.
- Agregar columnas de auditoría.

Tablas esperadas:

- data/silver/customers
- data/silver/products
- data/silver/sales
- data/silver/shops

## Gold

Unir las cuatro tablas para generar una tabla final llamada `comercial`.

La tabla `comercial` debe contener métricas comerciales como:

- ventas totales
- unidades vendidas
- cantidad de transacciones
- ventas por cliente
- ventas por producto
- ventas por tienda
- ventas por fecha
- ticket promedio

La tabla debe guardarse en:

- data/gold/comercial

## Dashboard

Crear un dashboard interactivo en HTML con:

- ventas totales
- unidades vendidas
- ventas por tienda
- ventas por producto
- ventas por cliente
- evolución temporal de ventas
- ticket promedio

El dashboard debe guardarse en:

dashboard/comercial_dashboard.html