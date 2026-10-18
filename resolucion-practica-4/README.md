# Entrega práctica 4 — Streaming

- **Nombre:** Narela Armani
- **student_id:** narela_armani
- **Escala:** small

Respondé cada pregunta en una a tres oraciones. Cuando se pide un dato, copiá el valor que te dio el notebook.

## Log y offsets

**1.** En la cola quedaron 2 mensajes (pago_4 y pago_5) y el log siguio teniendo los 5. En una cola leer un mensaje lo borra, en cambio en un log como Kafka leer no cambia nada, el consumidor solo avanza su offset.

**2.** Despues de la llegada 2 antifraude habia leido 180 mensajes y reportes 0. No lo afecto, porque cada consumidor tiene su propio checkpoint y leer no borra nada del topic, asi que cada uno avanza a su ritmo.

**3.** El offset es la posicion hasta donde leyo un consumidor en el log, aca es la version de la tabla Delta que hace de topic. Cada consumidor lo guarda en su propio checkpoint, que apunta a la proxima version a leer.

**4.** El replay leyo 280 mensajes, o sea todo el topic. Kappa propone tener un solo codigo de streaming y para reprocesar lanzar una consulta nueva que lea el log desde el principio, mientras que Lambda tiene una capa batch y otra de streaming con la misma logica escrita dos veces.

## Ingesta y enriquecimiento

**5.** Bronze proceso 60, 120, 40, 40 y 20 filas nuevas en cada llegada y 0 al reejecutar sin archivos nuevos. Lo que evita leer dos veces el mismo archivo es el checkpoint de Auto Loader, que se acuerda que archivos ya leyo.

**6.** Aparecieron INVALID_AMOUNT (bad_amount) y UNKNOWN_CUSTOMER (bad_customer), los dos en la llegada stream_002. Bronze guarda el texto original para no perder el dato, asi si cambia el contrato o hay que corregir algo se puede volver a procesar.

**7.** El join agrego country desde silver_customers y category desde silver_products. Es un join stream-tabla, porque cada evento del stream se cruza con tablas estaticas de referencia.

## Tiempo y ventanas

**8.** Event time es cuando paso la compra y processing time es cuando llega al sistema. late_ok ocurrio a las 12:03 pero se proceso recien en la llegada 2, junto con compras de 12:06 y 12:08.

**9.** El watermark quedo en 11:54, 11:58, 12:16, 12:17 y 12:35 al terminar cada llegada. Se calcula como el maximo event time visto menos 10 minutos.

**10.** Aparecieron en la llegada 3. Antes no, porque el watermark todavia no habia pasado el fin de ninguna ventana, y en append una ventana se emite recien cuando cierra.

**11.** late_bad se descarto y no sumo en ninguna ventana. late_ok llego cuando el watermark estaba en 11:54, asi que 12:03 estaba dentro del margen, en cambio late_bad (12:02) llego cuando el watermark ya estaba en 12:16 y su ventana ya estaba cerrada.

**12.** Tumbling son ventanas fijas de 5 min que no se pisan, hopping son de 10 min que avanzan cada 5 y se solapan (un evento cae en dos), y session es por cliente y se cierra despues de 5 min sin compras, asi que dura lo que dure la actividad. Ejemplos: tumbling para ventas cada 5 minutos en un dashboard, hopping para un promedio movil de los ultimos 10 minutos para alertas, y session para analizar las sesiones de compra de cada usuario.

**13.** En update se emitio dos veces: en la llegada 1 con 40 compras y 6000 de monto, y en la llegada 2 corregida con 60 compras y 8400. En append se emitio una sola vez, en la llegada 3, con 60 compras y 8400. Update te da resultados antes pero el consumidor tiene que bancarse recibir la misma ventana varias veces, y append es mas simple porque emite el valor final una vez, pero con mas demora.

**14.** Con 1 minuto se gana latencia, las ventanas cierran antes y hay que guardar menos estado. Lo que se pierde es completitud, porque cualquier evento que llegue con mas de 1 minuto de atraso se descarta.

## Garantías y fallas

**15.** Sin deduplicar quedaron 220 filas y deduplicando por event_id 200. El productor puede mandar el mismo evento dos veces porque si no esta seguro de que llego, reintenta (at-least-once).

**16.** Al reiniciar con el mismo checkpoint no se escribio nada nuevo y quedaron 200 filas, eso es exactly-once dentro de Spark. Al perder el checkpoint arranco de cero y reescribio todo, quedaron 400 filas, o sea at-least-once.

**17.** Porque el MERGE solo inserta si el event_id no esta en la tabla, entonces reprocesar los mismos eventos no agrega nada y quedo en 200 las dos veces. Que sea idempotente significa que aplicarla dos veces tiene el mismo efecto que aplicarla una.

## CDC

**18.** El log genero 13 eventos de cambio: 6 insert, 3 update_preimage, 3 update_postimage y 1 delete. Un UPDATE genera dos porque guarda la fila como estaba antes (preimage) y como quedo despues (postimage).

**19.** Si, la tabla reconstruida quedo igual a la original. El log tiene la historia que la tabla ya no tiene, por ejemplo que el cliente 2 estuvo en CL antes de BR y que el cliente 3 existio y se borro.

## Cierre

**20.** Spark Structured Streaming lo elegiria si ya uso Spark o Databricks y quiero mezclar batch y streaming con el mismo codigo, su ventaja es esa integracion con el lakehouse. Flink para streaming de baja latencia de verdad (evento a evento) con mucho estado, su ventaja es la latencia y el manejo de estado y event time. Kafka Streams si todo ya esta en Kafka y quiero algo liviano dentro de mi app, su ventaja es que es una libreria y no necesita un cluster aparte.
