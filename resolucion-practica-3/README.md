# Entrega práctica 3 — Bases de datos NoSQL

- **Nombre:** Narela Armani
- **student_id:** narela_armani
- **Escala:** small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dio el notebook.

## CAP y PACELC

**1.** En modo CP, C devolvio error en las dos operaciones ("ERROR: no disponible" en el read y "ERROR: sin quorum" en el write), o sea que sacrifica disponibilidad. En modo AP, C respondio 100, que era un dato viejo porque ya se habia escrito 80, y acepto el write de 50, asi que sacrifica consistencia.

**2.** Despues de la reparacion las tres replicas quedaron en 50. Se perdio la escritura saldo=80 que se hizo en A, porque last-write-wins se quedo con la version mas nueva, que era la de C.

**3.** Con W=1 la p99 dio 9.6 ms y con W=3 dio 55.6 ms. Esperar mas replicas tarda mas porque hay que esperar a la remota, pero a cambio ganas consistencia: el dato queda confirmado en todas las replicas.

**4.** Pudo seguir escribiendo el lado ABC, porque tiene 3 de 5 replicas y eso es mayoria (quorum). DE tiene solo 2, no llega al quorum y le devuelve error a todo.

## Clave-valor

**5.** El GET en memoria fue unas 1,134,947 veces mas rapido que el GET con Spark. Redis guarda todo en memoria porque leer de RAM es muchisimo mas rapido que ir a disco o lanzar una consulta, y la idea es devolver un valor por clave en milisegundos.

**6.** Porque para la base el valor es opaco, el pais esta adentro del JSON y solo se puede buscar por clave, entonces hubo que parsear todos los valores para filtrar. La ventaja es que el GET/SET por clave es O(1), rapidisimo y facil de repartir entre nodos, y la desventaja es que no podes consultar por el contenido del valor sin recorrer todo o armarte indices a mano.

**7.** Con hash mod N se movio el 79.6% de las claves y con hashing consistente el 21.2%. Conviene el segundo porque al agregar un nodo solo se mueven las claves de su tramo del anillo (mas o menos 1/(N+1)) y no casi todos los datos por la red.

## Documental

**8.** La consulta relacional devolvio 11 filas y la documental 1 documento. Dentro del documento quedaron embebidas todas las transacciones del cliente, cada una con una copia del producto (product_id, category, price), ademas del profile y los stats.

**9.** Spark armo un esquema que es la union de todos los campos y completo con null los que faltaban en cada documento. Se dice que tiene esquema flexible porque cada documento puede traer campos distintos sin definir un esquema antes, y el esquema se infiere recien al leer.

**10.** El documento mas grande pesa 5.2 KB. Si un documento crece sin limite cada lectura y escritura se vuelve mas pesada y en algun momento choca con el limite de 16 MB de MongoDB.

**11.** Resuelve bien las consultas por entidad, tipo traer un cliente con todo su historial o filtrar dentro de sus transacciones, sin joins. Le cuestan las consultas que cruzan documentos, como agregar por categoria, porque hay que desarmar los arrays con explode y volver a filas.

## Grafos

**12.** El grafo tiene 5000 nodos Customer, 2361 nodos Device y 5400 aristas USES.

**13.** SQL necesito 3 joins para el patron de 2 saltos y 5 para el de 4. Neo4j es mas eficiente porque cada nodo guarda punteros a sus vecinos (index-free adjacency), entonces cada salto es seguir un puntero y no hacer un join contra toda la tabla de aristas.

**14.** Tardo 14 iteraciones en converger y el componente mas grande tiene 23 nodos.

**15.** Con hash(id) mod 4 quedo el 74.4% de las aristas entre maquinas distintas. Es dificil distribuir un grafo porque las relaciones cruzan las particiones, entonces cada salto puede ser un viaje por la red, y cortar el grafo sin romper muchas aristas no es facil.

## Vectorial

**16.** El vector resume el comportamiento de compra del cliente: que proporcion gasta en cada categoria, que proporcion paga por cada canal, el ticket promedio y la proporcion de compras grandes. La similitud coseno mide que tan parecida es la direccion de dos vectores, 1 es que apuntan igual.

**17.** Entre los vecinos de clientes marcados hay un 31.6% de marcados, contra una tasa base de 14.8%. Buscar vecinos parecidos sirve para detectar clientes que se comportan como uno sospechoso, o sea para clasificar o recomendar por similitud.

**18.** Con nprobe=4 el recall dio 0.872 escaneando el 6% de los vectores. Con ANN ganas velocidad porque comparas contra muchos menos vectores, pero perdes exactitud porque se te pueden escapar algunos vecinos verdaderos.

## Columnar

**19.** En CSV ocupan 50.0 MB, en JSON 121.1 MB y en Parquet 5.3 MB. Para la consulta sobre Parquet Spark leyo solo la columna amount (`ReadSchema: struct<amount:decimal(12,2)>`), y con los datos ordenados se pueden saltear 7 de 8 archivos.

**20.** Es bueno para analitica porque lee solo las columnas que necesita, comprime mejor y puede saltear bloques con min/max. Para modificar una fila por vez no conviene porque la fila esta repartida en todas las columnas y los archivos son inmutables, entonces hay que reescribir el archivo entero.
