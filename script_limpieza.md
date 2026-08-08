# Script de limpieza – TechStore

## Objetivo

Este script realiza la limpieza de una tabla de ventas de TechStore utilizando exclusivamente lenguaje M desde el Editor Avanzado de Power Query.

Las transformaciones realizadas son:

1. Eliminación de espacios al inicio y al final de `nombre_producto`.
2. Estandarización de `categoria` utilizando Title Case.
3. Eliminación de registros de prueba cuya categoría es `Prueba`.
4. Definición de los tipos de datos correctos para cada columna.

## Código M

```m
let
    // Paso 1: Fuente de datos original
    Origen = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("XZBBasMwEEWvMmgdB0m2WndpJ4GWNBAaly5MFoqihcCWjGxBr5Mz9Ai+WEcOhai7mQeP/2faljCyIvAuh8kNcPQOmAAkG9cPYZLKzD8WV8YpXVOKE6e8yCjLqCDnVUs4ooMLo4Y3K7v51l+8UQ6hVEqPzhs3Rqn8J5eLnMfoRqtOXh0ctJpv9i4fPz53dYXDi0hFxhexWFKtmZyHYh/7qrRvIZK6PKP5IoqYWAXsGDrp9Qh1g6QKV+PuV6YWo4v1hOh02sLue9Le4ouaGh5bsjzx8r/nPCP60hcle3jdxpzHn5QideJp518=", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [id_venta = _t, nombre_producto = _t, categoria = _t, precio = _t, fecha_venta = _t]),

    // Paso 2: Eliminar espacios en blanco al inicio y al final del nombre del producto
    LimpiarEspacios = Table.TransformColumns(Origen, {{"nombre_producto", Text.Trim, type text}}),

    // Paso 3: Estandarizar la categoría utilizando Title Case
    EstandarizarCategoria = Table.TransformColumns(LimpiarEspacios, {{"categoria", Text.Proper, type text}}),

    // Paso 4: Eliminar los registros de prueba una vez estandarizada la categoría
    EliminarPruebas = Table.SelectRows(EstandarizarCategoria, each [categoria] <> "Prueba"),

    // Paso 5: Definir los tipos de datos correctos para cada columna
    TiparColumnas = Table.TransformColumnTypes(EliminarPruebas, {
        {"id_venta", Int64.Type},
        {"nombre_producto", type text},
        {"categoria", type text},
        {"precio", type number},
        {"fecha_venta", type date}
    }, "en-US")
in
    TiparColumnas
```

## Resultado

El script produce una tabla final de **5 registros**, ya que elimina los dos registros correspondientes a la categoría `PRUEBA`.

Además:

* Los espacios al inicio y al final de `nombre_producto` fueron eliminados.
* Las categorías quedaron estandarizadas en Title Case.
* Los registros de prueba fueron eliminados.
* Los tipos de datos fueron definidos correctamente.
* Los precios fueron interpretados utilizando la configuración regional `"en-US"`, ya que los datos originales utilizan el punto como separador decimal.
