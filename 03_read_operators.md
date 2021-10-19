# Operadores de lectura y proyección

## Operadores de comparación

$gte greaterthanequal $gt greaterthan $lte lowerthanequal $lt lowerthan

Type bracketing (Cuando una consulta usa operadores de comparación, solo devuelve los
valores que cumplan el criterio de la consulta y el mismo tipo de dato)

db.foo.insert([
    {a: 'a'},
    {a: 'A'},
    {a: 'C'},
    {a: 2},
    {a: 1},
    {a: 0.45},
    {a: -1},
    {a: 11},
    {a: '2'},
    {a: '1'},
    {a: '-1'},
    {a: '11'},
])

```
db.foo.find({a: {$gte: 2}}) // Solo devolverá los que cumplan la condición y sean de tipo numérico
```

```
db.foo.find({a: {$gte: 'A'}}) // Solo devolverá los que cumplan la condición y sean de tipo string
```

$eq equal

```
db.clientes.find({dni: {$eq: '658374678P'}}) // Idem asignación normal se usa sobretodo en Aggregation
```

$in includes ¡Ojo certificación!
{$in: [<valor>, <valor>, ...]}

Busca en cualquier campo los valores definidos en su array como un OR inclusivo
Devolverá cualquier documento que en el campo al que le pasamos este operador tenga
alguno de los elementos de su array

```
db.clientes2.find({nombre: {$in: ["Carlos","Inés","John"]}})
```
Devuelve los que en el campo nombre tienen el valor "Carlos" ó "Inés" ó "John"

También se puede usar en arrays

```
db.monitores.find({clases: {$in: ["esgrima","pesas"]}})
```

Devuelve los que alguno de los elementos de su array clases es "esgrima" o alguno de los elementos de su array clases es "pesas"

Expresiones regulares en las consultas (sin operador)

```
db.clientes3.find({nombre: /^S/})
```

Precisamente $in usa expresiones regulares como valor puesto que este operador no soporta
el operador $regex en combinación

```
db.monitor.find({clases: {$in: ["futbol", /^p/]}})
```

$ne notequal

```
db.clientes.find({apellidos: {$ne: "Pérez"}})
```
Los que en el campo apellidos no tienen el valor Pérez y los que no tienen el campo apellidos


```
db.clientes.find({apellidos: {$ne: "Pérez", $exists: true}})
```
En este segundo caso devuelve estrictamente los que no tienen el valor "Pérez" en apellidos

## Operadores lógicos

$and y $or (vistos anteriormente)

$not
{$not: <expresión>}


```
db.clientes.find({edad: {$not: {$gte: 18}}}) // Idem a $lt 18
```

$nor
{$nor: [<expresiones>,<expresión>,...]} Cuando no cumplen alguna de las expresiones

```
db.clientes.find({
    $nor: [
        {edad: {$lt: 18}},
        {nombre: "María"}
    ]
})
```
Devuelve los que no son menores de 18 y los que no se llaman "María"

Se pueden utilizar para encontrar los que no tienen un conjunto de valores

```
db.clientes.find({
    $nor: [
        {nombre: "Pilar"},
        {nombre: "Juan"}
    ]
})
```

## Operadores de evaluación

$type (visto anteriormente)

$exists
{$exists: <boolean>}

```
db.clientes.find({edad: {$exists: false}})
```
Devuelve los que no contienen el campo edad

En el caso de pasar true al operador también devuelve los que en el campo tienen el tipo-valor null

$regex (expresiones regulares)
{$regex: <expresión-regular>, $options: <opciones>}

Set de datos

db.clientes5.insert([
    {nombre: 'Luis', apellidos: 'García Pérez'},
    {nombre: 'Pedro', apellidos: 'Gutiérrez López'},
    {nombre: 'Sara', apellidos: 'López Gómez'},
    {nombre: 'María', apellidos: 'Pérez Góngora'},
    {nombre: 'Juan', apellidos: 'Pérez \nGóngora'},
])


```
db.clientes5.find({apellidos: {$regex: 'ez$'}}) // todos los que finalicen en ez
```

```
db.clientes5.find({apellidos: {$regex: '^gu', $options: 'i'}}) // opción case-insensitive
```

```
db.clientes5.find({apellidos: {$regex: '^G', $options: 'm'}}) // incluya saltos de línea (inclusivo)
```

```
db.clientes5.find({apellidos: {$regex: 'Gó m', $options: 'x'}}) // omite espacios en blanco
```

```
db.clientes5.find({apellidos: {$regex: 'gó m', $options: 'ix'}}) // se pueden combinar las opciones
```

$comment

```
db.clientes5.find({apellidos: {$regex: 'gó m', $options: 'ix'}, $comment: 'Devuelve los que contengan gom'})
```

## Operadores de Array

$all, $elemMatch (vistos anteriormente)

$size
Recibe un entero para realizar consultas de campos que tengan un determinado número de elementos

## Operadores de Proyección

$ ¡Ojo certificación!
Define en el documento de proyección los elementos a devolver de un campo array
en función de una expresión definida en el documento de consulta
- Solo devuelve el primer elemento del array que cumpla la condición
db.<colección>.find({<array>:<valor>},{"<array>.$": 1})


Set de datos

```
use videogames

db.results.insert([
    {player: 'Pepe', game: 'Tetris', points: [79,102,89,101]},
    {player: 'Laura', game: 'Tetris', points: [120,99,100,120]}
])
```

```
db.results.find(
    {game: 'Tetris', points: {$gte: 100}},
    {_id: 0, "points.$": 1}
)
```

$elemMatch en proyección

Set de datos

db.games.insert([
    {
        game: 'Tetris',
        players: [
            {name: 'pepe', maxScore: 98},
            {name: 'luisa', maxScore: 110},
            {name: 'John', maxScore: 105},
        ]
    },
    {
        game: 'Mario Bros',
        players: [
            {name: 'pepe', maxScore: 70},
            {name: 'luisa', maxScore: 98},
            {name: 'John', maxScore: 110},
        ]
    }
])

Funciona como el anterior ($) pero permite filtrar en campos de subdocumentos
y no hace falta que la expresión esté en la consulta

db.games.find(
    {game: 'Tetris'},
    {_id: 0, players: {$elemMatch: {maxScore: {$gte: 100}}}}
)

Devuelve todos los documentos con game igual a 'Tetris' pero solamente el campo player con el campo maxScore
con el primer valor que supere 100

Posibles respuestas de certificación a la operación anterior

a) {
    "players": [
            {name: 'luisa', maxScore: 110},
            {name: 'John', maxScore: 105},
    ]
}

b) {
    "players": [
            {name: 'luisa', maxScore: 110},
            {name: 'John', maxScore: 105},
    ]
}
{
    "players": [
            {name: 'John', maxScore: 110},
    ] 
}

c) {
    "players": [
            {name: 'luisa', maxScore: 110},
    ]
}
{
    "players": [
            {name: 'John', maxScore: 110},
    ] 
}

d) {
    "players": [
            {name: 'luisa', maxScore: 110}, // Correcta
    ]
}

$slice en proyección ¡Ojo certificación!
db.<colección>.find({<consulta>},{<array>: {$slice: <valor>}})
Los valores a pasar a slice son idem al operador de JS del mismo nombre

```
db.results.find({}, {points: {$slice: 3}}) // devuelve los tres primeros
```
```
db.results.find({}, {points: {$slice: -2}}) // devuelve los dos últimos
```
```
db.results.find({}, {points: {$slice: [2,2]}}) // skip-limit o posición-nº elementos
```


