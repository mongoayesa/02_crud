# Métodos de actualización de docs en MongoDB

# Métodos update(), y updateOne(), updateMany()

Sintaxis

```
db.<colección>.update(
    {<documento-de-consulta>}, // misma sintaxis que en los métodos de lectura
    {<documento-de-actualización>}, // definen los cambios en los documentos
    {<documento-de-opciones>}
)

db.<colección>.updateOne(
    {<documento-de-consulta>}, // misma sintaxis que en los métodos de lectura
    {<documento-de-actualización>}, // definen los cambios en los documentos
    {<documento-de-opciones>}
)

db.<colección>.updateMany(
    {<documento-de-consulta>}, // misma sintaxis que en los métodos de lectura
    {<documento-de-actualización>}, // definen los cambios en los documentos
    {<documento-de-opciones>}
)
```

Por defecto el método solo actualiza el primer documento que encuentra

Set de datos

use biblioteca

db.libros.insert([
    {title: 'Cien Años de Soledad', autor: 'Gabriel García Márquez', stock: 10},
    {title: 'La Ciudad y Los Perros', autor: 'Mario Vargas LLosa', stock: 10, prestados: 2},
    {title: 'El Otoño del Patriarca', autor: 'Gabriel García Márquez', stock: 10, prestados: 0},
])

### Actualización del documento completo (excepto _id porque es inmutable)

```
db.libros.update( // Solo lo permite con update porque cambia todo el documento
    {title: 'Cien Años de Soledad'},
    {title: 'Cien Años de Soledad', autor: 'Gabriel García Márquez', stock: 14}
)
```
Importante solo modifica la primera coincidencia de la consulta

### Actualización de campos del documento (utilizando operadores)
$set
{$set: {<campo>: <valor>, <campo>: <valor>, ...}}
Este operador modifica los campos y si el campo no existe lo crea con el valor proporcionado

```
db.libros.updateMany(
    {},
    {$set: {stock: 20}} // Actualiza todos los docs que pasen la consulta
)
```

*Con update para que actualice todos los documentos necesita el parametro multi: true en las opciones

### Actualización en campos con subdocumentos

Añadimos

db.libros.insert({
    title: 'El Quijote',
    autor: {
        nombre: 'Miguel',
        apellidos: 'Cervantes Saavedra',
        pais: 'España'
    } 
})

Notación del punto para campos en subdocumentos

```
db.libros.updateOne(
    {title: 'El Quijote'}, // En caso de más de una coincidencia con updateOne solo actualiza el primero
    {$set: {"autor.apellidos": "De Cervantes Saavedra"}}
)
```

### Actualización de elementos en un array

Seteamos un nuevo campo de array para practicar

db.libros.updateOne(
    {_id: ObjectId("616e9cf3015643598000f15d")},
    {$set: {categorias: ['novela','españa','best-seller']}}
)

Usa la notación del punto con la posición del elemento a actualizar

db.libros.updateOne(
    {_id: ObjectId("616e9cf3015643598000f15d")},
    {$set: {"categorias.1": "colombia"}}
)

### Actualización/inserción con la upsert

db.libros.updateOne(
    {title: 'La Biblia'},
    {$set: {categorias: ['religión','historia']}},
    {upsert: true} // Si no existe el documento lo crea
)

El documento creado con la opción upsert tendrá los campos-valor de la consulta y
los campos-valor de la actualización

{ _id: ObjectId("616ea7836183e4cbfcc0c4b7"),
  title: 'La Biblia',
  categorias: [ 'religión', 'historia' ] }

### Actualización de elementos en una array con $setOnInsert ¡Ojo certificación!

$setOnInsert es un operador similar a $set pero solo se ejecuta si la operación
resulta ser un upsert

db.libros.updateOne(
    {title: 'La Historia Interminable'},
    {$set: {precio: 18}, $setOnInsert: {autor: 'Michael Ende'}},
    {upsert: true}
)

### Actualización para eliminar campos 
$unset
{$unset: {<campo>: ''}}


```
db.libros.updateOne(
    {title: 'La Historia Interminable'},
    {$unset: {precio: ''}}
)
```

### Actualización para renombrar campos 
$rename
{$rename: {<campo>: <nuevo-nombre>}}

```
db.libros.updateMany(
    {},
    {$rename: {title: 'titulo'}}
)
```

### Faltan operadores de actualización

https://docs.mongodb.com/v4.0/reference/operator/update/

