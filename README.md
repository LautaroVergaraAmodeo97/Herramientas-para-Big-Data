# Herramientas-para-Big-Data

Mediante esta guía vamos a emular un ambiente de trabajo para poder implementar Big Data. Vamos a utilizar herramientas como Putty,Hadoop,Hive,etc.

Previamente tiene que tener instalado una máquina virtual (te recomiendo que uses VirtualBox pero puede ser cualquier máquina virtual de su agrado) y otros programas para facilitarle el aprendizaje.

# Instalación de Putty

Putty es un emulador de terminal que los usuarios de Windows pueden usar para conectarse a un sitio web. Ahora le vamos a explicar como utilizarlo para que usted pueda manejar la consola de Linux desde Windows.

En la terminal de Linux usted va a poner el comando que le paso aquí abajo para que usted pueda ver su dirección IP.
```
hostname -I
```
A usted le va a aparecer esto. Eso quiere decir que usted ya puede utilizar la terminal de Linux desde Windows con toda comodidad.

![Terminal putty](Img/putty.jpg)




Una vez instalado Putty vamos a proceder a seguir con la guía.

# Entorno Docker con Hadoop,Spark y Hive

En este espacio vamos a mostrarle a usted como implementar Hadoop,Hive,HBase,MongoDB,Neo4J;Zeppelin y Kafka. Espero que le sea útil esta guía.

# 1) HDFS

En este caso vamos a utilizar el archivo docker-compose-v1.yml para este ejercicio.

- Lo primero que vamos a realizar es el copiado del archivo a una carpeta llamada Datasets dentro del contenedor "namenode"

```
  sudo docker exec -it namenode bash
  cd home
  mkdir Datasets
  exit
  sudo docker cp <path><archivo> namenode:/home/Datasets/<archivo>
```
- Nos ubicamos en el contenedor "namenode"
```
  sudo docker exec -it namenode bash
```
- Vamos a crear un directorio HDFS llamado "/data"
```
  hdfs dfs -mkdir -p /data
```
- Por último copiadmos los archivos csv provisto a HDSF
```
  hdfs dfs -put /home/Datasets/*(nombre del archivo csv) /data
```

Todo este proceso se puede realizar mediante Putty.

# 2) Hive

En este caso vamos a utilizar el archivo docker-compose-v2.yml

Vamos a crear tablas en Hive a partir de los csv ingestados en HDFS.

Para poder realizar esto, primero nos tenemos que ubicar dentro del contendor correspondiente al servidor de Hive y ahí ejecutar los scrips que vamos a ir pasando.

```
  sudo docker exec -it hive-server bash
  hive
```

Anexo: Para poder ejecutar un script de Hive se requiere este comando. Además le ofrezco este pdf por si a usted le sigue interesando la sintaxis de Hive.

```
hive -f <script.hql>
```

*Tengo que colocar aqui el PDF con el cheat sheets

# 3) SQL

En esta sección de SQL vamos a ver como trabajamos con este tipo de consultas para poder mejorar la velocidad de consulta ya que al trabajar con Big Data esto consume mucho espacio en memoria. Así que le recomendamos realizar estas acciones para así poder achicar un poco el espacio de memoria consumida.

```
CREATE INDEX index_name
 ON TABLE base_table_name (col_name, ...)
 AS index_type
 [WITH DEFERRED REBUILD]
 [IDXPROPERTIES (property_name=property_value, ...)]
 [IN TABLE index_table_name]
 [ [ ROW FORMAT ...] STORED AS ...
 | STORED BY ... ]
 [LOCATION hdfs_path]
 [TBLPROPERTIES (...)]
 [COMMENT "index comment"];
```
Ejemplo:
```
hive> CREATE INDEX index_students ON TABLE students(id) 
 > AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' 
 > WITH DEFERRED REBUILD ;
```

# 4) No-SQL

Retomando un poco el rumbo a Big Data, nosotros utilizamos No-Sql porque están optimizadas para operaciones de lectura y escritura rápidas y pueden proporcionar un alto rendimiento para cargas de trabajo intensivas.

En este caso vamos a utilizar HBase. Más adelante vamos a demostrarle como usar MongoDB,Neo4J y Zepellin para realizar esta guía a la perfección.

En este caso vamos a utilizar el archivo docker-compose-v3.yml para estos ejercicios.

- HBase:
```
	1- sudo docker exec -it hbase-master hbase shell

		create 'personal','personal_data'
		list 'personal'
		put 'personal',1,'personal_data:name','Juan'
		put 'personal',1,'personal_data:city','Córdoba'
		put 'personal',1,'personal_data:age','25'
		put 'personal',2,'personal_data:name','Franco'
		put 'personal',2,'personal_data:city','Lima'
		put 'personal',2,'personal_data:age','32'
		put 'personal',3,'personal_data:name','Ivan'
		put 'personal',3,'personal_data:age','34'
		put 'personal',4,'personal_data:name','Eliecer'
		put 'personal',4,'personal_data:city','Caracas'
		get 'personal','4'

	2-En el namenode del cluster:

		hdfs dfs -put personal.csv /hbase/data/personal.csv

	3-sudo docker exec -it hbase-master bash
		
    hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=',' -Dimporttsv.columns=HBASE_ROW_KEY,personal_data:name,personal_data:city,personal_data:age personal hdfs://namenode:9000/hbase/data/personal.csv
		hbase shell
		scan 'personal'
		create 'album','label','image'
		put 'album','label1','label:size','10'
		put 'album','label1','label:color','255:255:255'
		put 'album','label1','label:text','Family album'
		put 'album','label1','image:name','holiday'
		put 'album','label1','image:source','/tmp/pic1.jpg'
		get 'album','label1'
```

- MongoDB:
```
	1) 	sudo docker cp iris.csv mongodb:/data/iris.csv
		  sudo docker cp iris.json mongodb:/data/iris.json

	2)  sudo docker exec -it mongodb bash

	3) 	mongoimport /data/iris.csv --type csv --headerline -d dataprueba -c iris_csv
		  mongoimport --db dataprueba --collection iris_json --file /data/iris.json --jsonArray

	4) mongosh
		use dataprueba
		show collections
		db.iris_csv.find()
		db.iris_json.find()
	
	5) 	mongoexport --db dataprueba --collection iris_csv --fields sepal_length,sepal_width,petal_length,petal_width,species --type=csv --out /data/iris_export.csv
		mongoexport --db dataprueba --collection iris_json --fields sepal_length,sepal_width,petal_length,petal_width,species --type=json --out /data/iris_export.json
				
	6) 	Descargar desde https://search.maven.org/search?q=g:org.mongodb.mongo-hadoop los jar:
		https://search.maven.org/search?q=a:mongo-hadoop-hive
		https://search.maven.org/search?q=a:mongo-hadoop-spark
		
		sudo docker cp mongo-hadoop-hive-2.0.2.jar hive-server:/opt/hive/lib/mongo-hadoop-hive-2.0.2.jar
		sudo docker cp mongo-hadoop-core-2.0.2.jar hive-server:/opt/hive/lib/mongo-hadoop-core-2.0.2.jar
		sudo docker cp mongo-hadoop-spark-2.0.2.jar hive-server:/opt/hive/lib/mongo-hadoop-spark-2.0.2.jar
		sudo docker cp mongo-java-driver-3.12.11.jar hive-server:/opt/hive/lib/mongo-java-driver-3.12.11.jar
		
	7) 	sudo docker cp iris.hql hive-server:/opt/iris.hql
		sudo docker exec -it hive-server bash

	8) 	hiveserver2
		chmod 777 iris.hql
		hive -f iris.hql
```
-  Neo4J
```
		CREATE (a:Location {name: 'A'}),
			   (b:Location {name: 'B'}),
			   (c:Location {name: 'C'}),
			   (d:Location {name: 'D'}),
			   (e:Location {name: 'E'}),
			   (f:Location {name: 'F'}),
			   (a)-[:ROAD {cost: 50}]->(b),
			   (b)-[:ROAD {cost: 50}]->(a),
			   (a)-[:ROAD {cost: 50}]->(c),
			   (c)-[:ROAD {cost: 50}]->(a),
			   (a)-[:ROAD {cost: 100}]->(d),
			   (d)-[:ROAD {cost: 100}]->(a),
			   (b)-[:ROAD {cost: 40}]->(d),
			   (d)-[:ROAD {cost: 40}]->(b),
			   (c)-[:ROAD {cost: 40}]->(d),
			   (d)-[:ROAD {cost: 40}]->(c),
			   (c)-[:ROAD {cost: 80}]->(e),
			   (e)-[:ROAD {cost: 80}]->(c),
			   (d)-[:ROAD {cost: 30}]->(e),
			   (e)-[:ROAD {cost: 30}]->(d),
			   (d)-[:ROAD {cost: 80}]->(f),
			   (f)-[:ROAD {cost: 80}]->(d),
			   (e)-[:ROAD {cost: 40}]->(f),
			   (f)-[:ROAD {cost: 40}]->(e);
			   
		CALL gds.graph.project(
			'miGrafo',
			'Location',
			'ROAD',
			{
				relationshipProperties: 'cost'
			}
		)

		MATCH (l:Location) RETURN l
					
		MATCH (source:Location {name: 'A'}), (target:Location {name: 'E'})
		CALL gds.shortestPath.dijkstra.write.estimate('miGrafo', {
			sourceNode: source,
			targetNode: target,
			relationshipWeightProperty: 'cost',
			writeRelationshipType: 'PATH'
		})
		YIELD nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory
		RETURN nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory

		MATCH (source:Location {name: 'A'}), (target:Location {name: 'E'})
		CALL gds.shortestPath.dijkstra.stream('miGrafo', {
			sourceNode: source,
			targetNode: target,
			relationshipWeightProperty: 'cost'
		})
		YIELD index, sourceNode, targetNode, totalCost, nodeIds, costs, path
		RETURN
			index,
			gds.util.asNode(sourceNode).name AS sourceNodeName,
			gds.util.asNode(targetNode).name AS targetNodeName,
			totalCost,
			[nodeId IN nodeIds | gds.util.asNode(nodeId).name] AS nodeNames,
			costs,
			nodes(path) as path
		ORDER BY index
```

- Zeppelin
```
	HDFS:
	En la máquina anfitrión probar WebHDFS:
		curl "http://<IP_Anfitrion>:9870/webhdfs/v1/?op=LISTSTATUS"
	En el interpreter:
		En la parte de "file"
			Variable hdfs.url = http://<IP_Anfitrion>:9870/webhdfs/v1/
	En nuevo notebook / nueva nota:
		%file
		ls /

	Neo4J:
	En el interpreter
		En la parte de "neo4J"
			Variables 
				neo4J.url = http://<IP_Anfitrion>:7687
				neo4j.auth.user	= neo4j
				neo4j.auth.password	= zeppelin
```
- 
