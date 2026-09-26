# mi-primer-proyecto

# Clase práctica 1 — Ingesta y capa Bronze

## Propósito

Construir la primera capa de un Lakehouse en Databricks a partir de fuentes CSV, JSON y Parquet. Al terminar, cada alumno tendrá un espacio aislado en Unity Catalog y cuatro tablas Bronze trazables.

## Duración estimada

| Bloque | Minutos |
|---|---:|
| Setup y recorrido del workspace | 25 |
| Generación de fuentes | 25 |
| Lectura, esquemas y calidad inicial | 35 |
| Pausa | 10 |
| Parquet, Delta y tablas Bronze | 40 |
| Desafío de evolución de esquema | 35 |
| Puesta en común y cierre | 10 |

## Antes de la clase

1. Completar la [guía paso a paso de Databricks Free Edition](../GUIA_SETUP_DATABRICKS_FREE.md).
2. Confirmar que se puede abrir `00_setup.ipynb` y ejecutar `SELECT current_catalog()`.
3. No instalar paquetes: la práctica usa únicamente Spark y Delta incluidos en la plataforma.

## Resultados esperados

Al finalizar deben existir:

```text
<catalogo>.bigdata_<alumno>.landing
<catalogo>.bigdata_<alumno>.bronze_customers
<catalogo>.bigdata_<alumno>.bronze_products
<catalogo>.bigdata_<alumno>.bronze_transactions
<catalogo>.bigdata_<alumno>.bronze_events
```

## Formato de entrega

La entrega se hace en el repositorio personal `mi-primer-proyecto` que creaste en la [guía de Git y GitHub](../GUIA_GIT_GITHUB.md).

### Estructura

Creá un directorio `resolucion-practica-1` en la raíz del repositorio con estos archivos:

```text
mi-primer-proyecto/
└── resolucion-practica-1/
    ├── README.md
    ├── 01_ingesta_bronze.ipynb
    └── 02_desafio.ipynb
```

| Archivo | Contenido |
|---|---|
| `README.md` | Nombre, `student_id` usado en los notebooks y respuestas de la sección **Entrega breve** de `01_ingesta_bronze`: tres observaciones sobre CSV/JSON, Parquet y Delta, y dónde aparece cada una de las cinco V. Incluí también la reflexión final del desafío (máximo 150 palabras). |
| `01_ingesta_bronze.ipynb` | Notebook ejecutado, con las salidas de las cuatro tablas Bronze y del diagnóstico de calidad. |
| `02_desafio.ipynb` | Notebook con las tres consignas resueltas y las aserciones ejecutadas sin errores. |

### Exportar los notebooks desde Databricks

1. Ejecutá cada notebook completo para que las salidas queden visibles.
2. Abrí **File → Export → IPython Notebook** y descargá el archivo `.ipynb`.
3. Copiá los archivos descargados a `resolucion-practica-1/` dentro de tu copia local del repositorio.

### Publicar la entrega

Desde la carpeta de tu repositorio:

```bash
git add resolucion-practica-1
git commit -m "Entrega práctica 1"
git push origin main
```

Verificá en GitHub que el directorio y los tres archivos aparecen en `main`.

El repositorio tiene que ser **público** para que el docente pueda ver la entrega. Para comprobarlo, abrí su URL en una ventana privada del navegador, sin iniciar sesión: si ves `resolucion-practica-1`, está accesible. Si lo creaste como privado, cambialo desde **Settings → General → Danger Zone → Change repository visibility**.

Enviá la URL de tu repositorio al mail de los profesores.

### Qué no incluir

El repositorio es público: cualquier persona puede ver los archivos y su historial. Revisá [qué implica que sea público](../GUIA_GIT_GITHUB.md#qué-implica-que-el-repositorio-sea-público) antes de hacer el push.

- Datos generados, archivos del volumen ni exportaciones de tablas: se reconstruyen ejecutando los notebooks.
- Tokens, contraseñas u otras credenciales.

#RESPUESTAS

## 1. Leer no es todavía transformar

**¿Por qué `amount` termina como texto?**
Porque el CSV no trae el tipo de dato de cada columna y nosotros tampoco lo especificamos. Sin esa info, Spark lee todo como `string` por default. El Parquet, en cambio, guarda el esquema dentro del archivo, por eso ahí los tipos aparecen bien.

**Experimento con `inferSchema`**
Aunque le pidamos que infiera, `amount` sigue quedando como texto, porque entre los valores hay `N/A`. Alcanza con un solo valor que no sea número para que Spark decida que toda la columna es `string`.

Inferir es trabajo adicional porque Spark tiene que recorrer los datos una vez más solo para adivinar los tipos antes de leerlos. Además, la decisión es inestable: depende de lo que venga en los datos. Si mañana llega un valor raro, el tipo puede cambiar. Lo mejor es definir el esquema nosotros y hacerlo bien de una.

## 3. Diagnóstico inicial de calidad

- Identificadores duplicados: **10**
- Importes que no se pueden convertir a número: **52**

## 4. Delta y el plan de ejecución

**¿Qué agrega Delta respecto de un directorio Parquet?**
- `DESCRIBE DETAIL` muestra la info de la tabla: el formato (`delta`), dónde está guardada, cuántos archivos tiene, cuánto pesa y cuándo se modificó por última vez.
- `DESCRIBE HISTORY` muestra cada versión de la tabla: qué operación se hizo, cuándo, quién la hizo y cuántas filas se escribieron.

Un directorio Parquet es solo una carpeta con archivos y no guarda nada de esto. Delta lo sabe porque tiene el `_delta_log`, un registro de todos los cambios que se le hicieron a la tabla.
