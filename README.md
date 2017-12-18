# Workshop de Big Data con Apache Spark [:)]
# Trabajo Final: M. Jazmín Montero + José Ignacio López Sáez

<!---
Se dispone de un dataset con la principal información de los vuelos aterrizados y despegados de los aeropuertos de la República Argentina desde el 01/01/2014 al último trimestre del 2017.

En el mismo (DB_Seminario_Intensivo.txt)se dispone la información de Fecha y Hora de la operación (aterrizaje o despegue), el número de vuelo ("callsign"), la matrícula de la aeronave, un código que identifica el tipo de aeronave, un código que identifica a la aerolínea y el código OACI de origen y el destino.

En otro dataset (DB_Aux_Seminario_Intensivo.txt) se dispone la información del nombre de la aerolínea para cada código (ejemplo: GLO --> Gol Linhas Aéreas)

Como los datos se suelen obtener en forma mensual, el proceso pensado es de tipo batch (a futuro, se podría pensar en un streaming, pero actualmente no se dispone de dicha tecnología en todos los aeródromos del país).
El objetivo final será poder levantar y procesar el dataset primario utilizando Spark, vincularlo con el auxiliar para obtener el nombre completo de la aerolínea y, finalmente, usando Superset, realizar los principales gráficos que describan la actividad aérea en el país.


Se tomó como referencia principal el código fuente (EtlSteps.scala) del proyecto us-stocks-analysis presentado en clase en la semana del 21/11/2017 en el ITBA en el marco del Seminario Intensivo de Tópicos Avanzados en Datos Complejos.
-->

## Infrastructura
El workshop simula una instalacion de produccion utilizando container de Docker.
[docker-compose.yml](docker-compose.yml) contiene las definiciones y configuraciones para esos servicios y sus respectivas UIs:

* Apache Spark: [Spark Master UI](http://localhost:8080) | [Job Progress](http://localhost:4040)
* Apache Kafka:
* Postgres:
* [Superset](http://superset.incubator.apache.org) [Dashboard](http://localhost:8088/)

Los puertos de acceso a cada servicio quedaron los defaults. Ej: spark-master:7077, postgres: 5432
#Se debe clonar este gitbuh con la siguiente instruccion:
git clone https://github.com/jazminmontero/Bigdata-LopezSaez-Montero.git

IMPORTANTE:
Asegurarse que todos los progamas ejecutables tengan la opción "allow executing file as program”
Debido al peso del csv hubo que guardarlo en un zip, primero se lo debe descomprimir en la carpeta dataset>>Flight

## Levantar ambiente
Instalar [Docker >= 17.03](https://www.docker.com/community-edition).
Correr el script que levanta el ambiente.


```bash
./control-env.sh start

# Access Spark-Master and run spark-shell
docker exec -it wksp_master_1 bash
root@588acf96a879:/app# spark-shell
```
Probar:
```scala
val file = sc.textFile("/dataset/Flight/flight.csv")
file.count
file.take(10).foreach(println)
```
Acceder a http://localhost:8080 y http://localhost:4040 para ver la SPARK-UI

## Codigo
* Análisis de vuelos en Argentina 2014 a 2017

## Compilar el codigo
Compilar y empaquetar el codigo para deploy en el cluster

```bash
cd code/flight
sbt clean assembly
```

## Submit de un job
Conectarse al Spark-Master y hacer submit del programa

```bash
docker exec -it wksp_master_1 bash

cd /app/us-stock-analysis
spark-submit --master 'spark://master:7077' \
  --class "es.arjon.RunAll" \
  --driver-class-path /app/postgresql-42.1.4.jar \
  target/scala-2.11/flight.jar \
    /dataset/Flight /dataset/Flight/flight.csv /dataset/output.parquet
```
Acceder a http://localhost:8080 y http://localhost:4040 para ver la SPARK-UI

<!---
```## Usando Spark-SQL
Usando SparkSQL para acceder a los datos en Parquet y hacer analysis interactiva. 
```

```bash
docker exec -it wksp_master_1 bash
spark-shell
```

```scala
import spark.implicits._
val df = spark.read.parquet("/dataset/output.parquet")
df.show

df.createOrReplaceTempView("flight")


```
-->

## Creando un Dashboard con Superset

* Acceder a http://localhost:8088/, user: `admin`, pass: `superset`.
* Agregar el database (Sources > Databases):
  - Database: `Workshop`
  - SQLAlchemy URI: `postgresql://workshop:w0rkzh0p@postgres/workshop`
  - OK
* Agregar tabla (Sources > Tables) :
  - Database: `workshop`
  - Table Name: `flight`
