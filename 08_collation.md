# Collation en MongoDB

Establece los criterios de clasificación de orden y selección en las
consultas de tipo string (idioma y 'fuerza').

Se define a tres niveles:

- A nivel de colección
- A nivel de índice
- A nivel de consulta

## A nivel de colección

Se declara como opción cuando se crea la colección (de manera explícita)

```
use tienda

db.createCollection('articulosConColacion', {collation: {locale: 'es'}}) // La propiedad locale es obligatoria
db.createCollection('articulosSinColacion')
```
Set de datos

db.articulosSinColacion.insert([
    {_id: 1, nombre: 'cafe'},
    {_id: 2, nombre: 'café'},
    {_id: 3, nombre: 'cafE'},
    {_id: 4, nombre: 'cafÉ'},
])

db.articulosConColacion.insert([
    {_id: 1, nombre: 'cafe'},
    {_id: 2, nombre: 'café'},
    {_id: 3, nombre: 'cafE'},
    {_id: 4, nombre: 'cafÉ'},
])

En las opciones de idioma, cambia el orden según este

```
db.articulosConColacion.find().sort({nombre: 1})
```
{ _id: 1, nombre: 'cafe' }
{ _id: 3, nombre: 'cafE' }
{ _id: 2, nombre: 'café' }
{ _id: 4, nombre: 'cafÉ' }

```
db.articulosSinColacion.find().sort({nombre: 1})
```
{ _id: 3, nombre: 'cafE' }
{ _id: 1, nombre: 'cafe' }
{ _id: 4, nombre: 'cafÉ' }
{ _id: 2, nombre: 'café' }

## A nivel de índices

Vemos luego

## A nivel de consulta (con el método collation())

```
db.articulosSinColacion.find().sort({nombre: 1}).collation({locale: 'es'})
```

¿Qué mas opciones tenemos de colación? (En cualquier nivel)

### Strength

Strength 3 (Valor por defecto) Distingue mayúsculas/minúsculas/diacríticos

```
db.articulosSinColacion.find({nombre: 'cafe'}).collation({locale: 'es'}) // Equivalente a strength 3 (defecto)
```

Strength 2 case insensitive

```
db.articulosSinColacion.find({nombre: 'cafe'}).collation({locale: 'es', strength: 2})
```
{ _id: 1, nombre: 'cafe' }
{ _id: 3, nombre: 'cafE' }

Strength 1 case insensitive y diacritic insensitive

```
db.articulosSinColacion.find({nombre: 'cafe'}).collation({locale: 'es', strength: 1})
```

{ _id: 1, nombre: 'cafe' }
{ _id: 2, nombre: 'café' }
{ _id: 3, nombre: 'cafE' }
{ _id: 4, nombre: 'cafÉ' }

Strength 1 y caseLevel true case sensitive  diacritic insensitive

```
db.articulosSinColacion.find({nombre: 'cafe'}).collation({locale: 'es', strength: 1, caseLevel: true})
```

{ _id: 1, nombre: 'cafe' }
{ _id: 2, nombre: 'café' }

Case first "upper" "lower" para que en las consultas ordenadas tengan preferencia unas u otras

```
db.articulosSinColacion.find().collation({locale: 'es', caseFirst: 'upper'}).sort({nombre: 1})
```
{ _id: 3, nombre: 'cafE' }
{ _id: 1, nombre: 'cafe' }
{ _id: 4, nombre: 'cafÉ' }
{ _id: 2, nombre: 'café' }

NumericOrdering ordenar números almacenados como string

Set de datos

db.articulosSinColacion.insert([
    { "_id" : 10, "nombre" : "A1" },
    { "_id" : 11, "nombre" : "A11" },
    { "_id" : 12, "nombre" : "A2" },
    { "_id" : 13, "nombre" : "A21" },
    { "_id" : 14, "nombre" : "A3" }
])

Sin numericOrdering

```
db.articulosSinColacion.find({_id: {$gte: 10}}).collation({locale: 'es'}).sort({nombre: 1})
```
{ _id: 10, nombre: 'A1' }
{ _id: 11, nombre: 'A11' }
{ _id: 12, nombre: 'A2' }
{ _id: 13, nombre: 'A21' }
{ _id: 14, nombre: 'A3' }


```
db.articulosSinColacion.find({_id: {$gte: 10}}).collation({locale: 'es', numericOrdering: true}).sort({nombre: 1})
```

{ _id: 10, nombre: 'A1' }
{ _id: 12, nombre: 'A2' }
{ _id: 14, nombre: 'A3' }
{ _id: 11, nombre: 'A11' }
{ _id: 13, nombre: 'A21' }