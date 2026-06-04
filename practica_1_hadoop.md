# Práctica 1: Hola Mundo en Hadoop (WordCount)

Esta guía paso a paso te mostrará cómo subir un archivo local al sistema de archivos distribuido de Hadoop (HDFS) y cómo ejecutar tu primer trabajo de procesamiento (MapReduce) para contar las palabras de ese archivo.

---

## Paso 1: Preparar el archivo local

Primero, crea un archivo de texto en tu computadora con el contenido que quieres analizar. Abre tu terminal en la carpeta del proyecto y ejecuta:

```bash
echo "Archivo para subir a hadoop, este es mi primer archivo" > texto.txt
```

## Paso 2: Copiar el archivo al contenedor

Hadoop corre dentro de Docker, por lo que primero debemos pasar el archivo físico desde tu computadora hacia la máquina virtual del maestro (el contenedor `namenode`):

```bash
docker cp texto.txt namenode:/texto.txt
```

## Paso 3: Crear el directorio en HDFS (Disco virtual de Hadoop)

A continuación, dile a Hadoop que cree una carpeta interna llamada `/datos` en su propio sistema de archivos. _(Nota: Si la carpeta ya existe de una práctica anterior, puedes saltarte este paso)_:

```bash
docker exec namenode hdfs dfs -mkdir -p /datos
```

## Paso 4: Subir el archivo a HDFS

Mueve el archivo que metimos al contenedor hacia el sistema distribuido de Hadoop:

```bash
docker exec namenode hdfs dfs -put /texto.txt /datos/texto.txt
```

_(Nota: Es normal si aquí ves un mensaje `INFO sasl.SaslDataTransferClient`. Es solo un aviso informativo, no un error)._

## Paso 5: Ejecutar el trabajo MapReduce (Procesamiento)

Lanza el programa `wordcount` de ejemplo. Le indicas la carpeta de entrada (`/datos`) y la carpeta donde debe guardar el resultado (`/resultados`).

**⚠️ ¡Importante!** La carpeta de resultados **no debe existir previamente**. Hadoop la creará automáticamente. Si ya existe, te dará error.

```bash
docker exec namenode yarn jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /datos /resultados
```

## Paso 6: Ver el resultado final

Una vez que el trabajo termine exitosamente, lee el archivo que generó el proceso de reducción (Reduce):

```bash
docker exec namenode hdfs dfs -cat /resultados/part-r-00000
```

¡Listo! Verás en pantalla el conteo de cada palabra.

---

### Comandos Extras muy útiles

- **Ver los archivos dentro de una carpeta de HDFS:**

  ```bash
  docker exec namenode hdfs dfs -ls /datos
  ```

- **Borrar una carpeta en HDFS (muy útil si quieres volver a correr el MapReduce y necesitas borrar la carpeta /resultados):**
  ```bash
  docker exec namenode hdfs dfs -rm -r /resultados
  ```
