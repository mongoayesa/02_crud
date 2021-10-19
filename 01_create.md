# Métodos de inserción de docs en MongoDB

## Método insert()

Sintaxis

```
db.<colección>.insert(
    <documento | array de documentos>,
    {
        writeConcern: <valores> // tiene que ver con replicaSet
        ordered: <boolean>
    }
)
```

### Insertar un doc en una colección sin _id

use clinica

```
db.inventario.insert({articulo: 'zueco', cantidad: 100}) // Si la colección no existe //// MongoDB la crea
```
Devuelve objeto
{ acknowledged: true,
  insertedIds: { '0': ObjectId("616d41a8d95ac25f419bca15") } }

###  Insertar un doc en una colección con _id

- El valor de _id debe ser único
- El valor de _id será inmutable
- El valor de _id puede ser cualquier tipo, incluso doc embebido, excepto array ¡ojo pregunta certi!

```
db.inventario.insert({articulo: 'gasa 40', cantidad: 1000, _id: {old: 'FG5646', new: 'ADF68376'}})
```

{ acknowledged: true,
  insertedIds: { '0': { old: 'FG5646', new: 'ADF68376' } } }

```
db.inventario.insert({articulo: 'gasa 40', cantidad: 1000, _id: ['SGSJH3253','DGSJG532753']})
```
MongoBulkWriteError: The '_id' value cannot be of type array

### Inserción de múltiples documentos (array de docs)

```
db.inventario.insert([
    {articulo: 'bisturi 20', cantidad: 10},
    {articulo: 'bisturi 30', cantidad: 15},
    {articulo: 'bisturi 40', cantidad: 20},
])
```

- Las operaciones de inserciones múltiples (insert() o insertMany()) son atómicas a nivel de documento

### Inserción de múltiples documentos con ordered true (valor por defecto) ¡Ojo Certificación!
(cuando se produzcan errores por alguno de los documentos)

```
db.empleados.insert([
    {_id: 10, nombre: 'Carlos', apellidos: 'Pérez'},
    {_id: 20, nombre: 'Juan', apellidos: 'Pérez'},
    {_id: 20, nombre: 'Sara', apellidos: 'Pérez'}, // A partir del error aunque la conexion no tenga problemas
    {_id: 40, nombre: 'Pilar', apellidos: 'Pérez'}, // no se ejecutan el resto de inserciones
    {_id: 50, nombre: 'Raquel', apellidos: 'Pérez'},
]) // ordered: true
```

```
db.empleados.find()
```
{ _id: 10, nombre: 'Carlos', apellidos: 'Pérez' }
{ _id: 20, nombre: 'Juan', apellidos: 'Pérez' }

### Inserción de múltiples documentos con ordered false ¡Ojo Certificación!
(cuando se produzcan errores por alguno de los documentos)

```
db.empleados.insert([
    {_id: 60, nombre: 'Carlos', apellidos: 'Pérez'},
    {_id: 70, nombre: 'Juan', apellidos: 'Pérez'},
    {_id: 70, nombre: 'Sara', apellidos: 'Pérez'}, // A partir del error aunque la conexion no tenga problemas
    {_id: 90, nombre: 'Pilar', apellidos: 'Pérez'}, // con ordered false se ejecutan el resto de inserciones
    {_id: 100, nombre: 'Raquel', apellidos: 'Pérez'},
], {ordered: false}) 
```

```
db.empleados.find()
```

...
{ _id: 60, nombre: 'Carlos', apellidos: 'Pérez' }
{ _id: 70, nombre: 'Juan', apellidos: 'Pérez' }
{ _id: 90, nombre: 'Pilar', apellidos: 'Pérez' }
{ _id: 100, nombre: 'Raquel', apellidos: 'Pérez' }

Alta probabilidad de que caiga una pregunta del estilo a bloque de código y pregunta tras la operación cuantos docs se insertarán

### Inserción de documentos con subdocumentos

Set de datos

```
db.empleados.insert({
    nombre: 'Carlos', 
    apellidos: 'Pérez',
    direcciones: [
        {
            calle: 'Principe de Vergara, 100',
            localidad: {
                cp: '28010',
                ciudad: 'Madrid'
            }
        },
        {
            calle: 'Av. constitución, 100',
            localidad: {
                cp: '28300',
                ciudad: 'Móstoles'
            }
        },

    ]
})
```

Límites

- 100 niveles de anidado
- 16 MB en cuanto a tamaño máximo de doc

## Método insertOne()

Idem a insert solo permite un documento

Sintaxis

```
db.<colección>.insertOne(
    <documento>,
    {
        writeConcern: <valores> // tiene que ver con replicaSet
    }
)
```

Resto idéntico, lo único es que no acepta el método explain()

## Método insertMany()

Idem a insert pero recibe array de varios docs

Sintaxis

```
db.<colección>.insertMany(
    <array de documentos>,
    {
        writeConcern: <valores> // tiene que ver con replicaSet
        ordered: <boolean>
    }
)
```

Resto idéntico, lo único es que no acepta el método explain()

### Otros métodos que realizan inserciones

db.<colección>.save() // Actualiza o inserta deprecated
db.<colección>.updateOne()
db.<colección>.updateMany()
db.<colección>.findAndModifiy()
db.<colección>.findOneAndUpdate()
db.<colección>.findOneAndReplace()
db.<colección>.bulkWrite()
