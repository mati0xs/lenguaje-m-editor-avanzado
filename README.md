# Lenguaje M en el Editor Avanzado – Limpieza de datos

## Descripción

Este ejercicio consiste en realizar una limpieza de datos utilizando exclusivamente lenguaje M desde el Editor Avanzado de Power Query, sin utilizar los botones de transformación de la interfaz.

El contexto utilizado es el de una empresa ficticia llamada **TechStore**, que recibe información de ventas proveniente de un sistema legacy con problemas de calidad de datos.

La tabla contiene espacios innecesarios en los nombres de productos, categorías con diferentes formatos de mayúsculas y minúsculas, registros de prueba y tipos de datos que necesitan ser definidos correctamente.

---

## Transformaciones realizadas

El script realiza las siguientes transformaciones:

### 1. Limpieza de espacios

Se utiliza `Text.Trim` sobre la columna `nombre_producto`.

Esta función elimina los espacios en blanco que aparecen al principio y al final del texto.

Por ejemplo:

`" Laptop Pro 15 "` → `"Laptop Pro 15"`

Esto permite evitar inconsistencias causadas por espacios innecesarios.

### 2. Estandarización de categorías

Se utiliza `Text.Proper` sobre la columna `categoria`.

Esta función convierte el texto a formato Title Case.

Por ejemplo:

* `accesorios` → `Accesorios`
* `COMPUTACIÓN` → `Computación`
* `Audio` → `Audio`

De esta manera, los valores quedan representados con un formato uniforme.

### 3. Eliminación de registros de prueba

Después de estandarizar la categoría, se utiliza `Table.SelectRows` para conservar solamente los registros cuya categoría sea diferente de `Prueba`.

La condición utilizada es:

```m
each [categoria] <> "Prueba"
```

Esto permite eliminar los dos registros que originalmente tenían la categoría `PRUEBA`.

### 4. Definición de tipos de datos

Finalmente, `Table.TransformColumnTypes` establece los tipos correspondientes a cada columna:

* `id_venta` → `Int64.Type`
* `nombre_producto` → `type text`
* `categoria` → `type text`
* `precio` → `type number`
* `fecha_venta` → `type date`

Además, se especifica la configuración regional `"en-US"` para interpretar correctamente los valores de `precio`, ya que los datos originales utilizan el punto como separador decimal.

---

# Preguntas conceptuales

## 1. ¿Qué hace exactamente el bloque `let...in` en lenguaje M?

El bloque `let...in` permite organizar una consulta de Power Query como una secuencia de pasos.

Dentro de `let` se definen las diferentes transformaciones. Cada paso tiene un nombre y puede utilizar como entrada el resultado de un paso anterior.

Por ejemplo:

```m
LimpiarEspacios = Table.TransformColumns(Origen, ...)
```

En este caso, `LimpiarEspacios` utiliza `Origen` como punto de partida.

Luego:

```m
EstandarizarCategoria = Table.TransformColumns(LimpiarEspacios, ...)
```

utiliza el resultado de `LimpiarEspacios`.

De esta manera se forma una cadena de transformaciones en la que cada paso depende del anterior.

La expresión `in` indica cuál es el resultado final que debe devolver la consulta. En este ejercicio:

```m
in
    TiparColumnas
```

significa que Power Query devuelve el resultado producido por el último paso.

Comprender esta estructura es importante porque permite leer, modificar y depurar consultas directamente desde el código M.

---

## 2. ¿Por qué M es Case Sensitive y qué consecuencia práctica tiene?

M es un lenguaje **Case Sensitive**, lo que significa que diferencia entre mayúsculas y minúsculas.

Por este motivo, las funciones y referencias deben escribirse respetando exactamente su nombre.

Por ejemplo:

```m
Table.TransformColumns
```

es correcto, mientras que:

```m
table.transformcolumns
```

generaría un error porque Power Query no reconoce esa escritura como la función correspondiente.

Lo mismo puede ocurrir con los nombres de los pasos. Si un paso se llama:

```m
LimpiarEspacios
```

una referencia como:

```m
limpiarespacios
```

no sería equivalente.

Por eso es importante respetar exactamente los nombres utilizados dentro de la consulta.

---

## 3. ¿Cuál es la diferencia entre `Text.Trim` y `Text.Clean` en M?

Las dos funciones sirven para limpiar texto, pero realizan tareas diferentes.

`Text.Trim` elimina los espacios en blanco que se encuentran al principio y al final de una cadena.

Por ejemplo:

```m
Text.Trim(" Laptop Pro 15 ")
```

produce:

```text
Laptop Pro 15
```

En cambio, `Text.Clean` elimina caracteres de control que pueden aparecer dentro de un texto, como determinados caracteres no imprimibles provenientes de archivos o sistemas externos.

Por lo tanto, `Text.Trim` es más adecuado cuando el problema consiste específicamente en espacios al inicio o al final, como ocurre en este ejercicio.

`Text.Clean` sería más útil cuando se detectan caracteres no imprimibles o problemas provenientes de la importación de datos desde otros sistemas.

Incluso pueden utilizarse ambas funciones cuando un dataset presenta los dos tipos de problemas.

---

## 4. ¿Por qué filtraste los registros `PRUEBA` después de estandarizar la categoría y no antes?

Primero se estandarizó la columna `categoria` utilizando `Text.Proper` y posteriormente se realizó el filtro.

Esto es importante porque los datos originales presentan diferentes formas de escritura. Por ejemplo, los registros de prueba aparecen como:

```text
PRUEBA
```

Después de aplicar:

```m
Text.Proper
```

el valor pasa a ser:

```text
Prueba
```

Entonces podemos utilizar una condición única:

```m
each [categoria] <> "Prueba"
```

Si el filtro se realizara antes de estandarizar, habría que contemplar las diferentes formas en las que puede aparecer el mismo valor, como `PRUEBA`, `prueba` o `Prueba`.

Por lo tanto, primero se normaliza la información y después se aplica el filtro. Esto hace que la lógica sea más simple y evita que determinados registros de prueba queden sin eliminar debido a diferencias de mayúsculas y minúsculas.

---

# Resultado final

Después de ejecutar el script, la tabla queda con **5 registros**.

Los registros correspondientes a la categoría `PRUEBA` fueron eliminados.

Además:

* Los nombres de productos no tienen espacios al inicio ni al final.
* Las categorías utilizan un formato uniforme.
* Los tipos de datos son los correspondientes a cada campo.
* El precio se interpreta correctamente como número.
* La fecha se interpreta correctamente como fecha.

## Evidencia

La ejecución del script y el resultado final se encuentran documentados mediante las capturas incluidas en la entrega.

**Captura 1:** Editor Avanzado con el código M completo.

**Captura 2:** Resultado final de la tabla con 5 registros.

---

## Conclusión

El ejercicio permitió practicar la construcción de transformaciones directamente mediante lenguaje M, sin depender de la interfaz gráfica de Power Query.

La principal ventaja de trabajar de esta manera es que permite controlar con mayor precisión cada transformación, definir el orden en que se ejecutan y comprender cómo se relacionan los diferentes pasos de una consulta.

También permitió comprobar la importancia de respetar la sintaxis y la sensibilidad a mayúsculas y minúsculas del lenguaje M, así como de mantener correctamente las referencias entre los diferentes pasos.
