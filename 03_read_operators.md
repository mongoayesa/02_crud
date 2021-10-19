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
