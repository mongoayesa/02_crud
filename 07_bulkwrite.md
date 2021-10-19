# Operaciones de escritura en lote (No son transacciones)

## Método bulkWrite()
Sintaxis
db.<coleccion>.bulkWrite(
    [
        <operacion1>,
        <operacion2>,
        ...
    ]
    {
        ordered: <boolean>, // Idem insert()
        writeConcern: <valor>
    }
)

Las operaciones vienen definidas en https://docs.mongodb.com/manual/core/bulk-write-operations/

```
use biblioteca

db.libros.bulkWrite(
    [
        {insertOne: {document: {titulo: 'El Señor de los Anillos', autor: 'J.R.R. Tolkien'}}},
        {updateOne: {
            filter: {titulo: 'La Historia Interminable'},
            update: {$set: {precio: 25}}
        }},
        {deleteOne: {filter: {autor: 'William Shakespeare'}}}
    ]
)
```


