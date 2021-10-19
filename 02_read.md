# Métodos de lectura de docs en MongoDB

## Método find()

Sintaxis

```
db.<coleccion>.find(
    <documento-de-consulta>, // combinación de campos, valores y operadores
    <documento-de-proyección>, // opcional donde determinas los campos de cada doc a devolver
)
```

### Consulta de todos los docs de una colección

```
db.empleados.find()
db.empleados.find({}) // usado cuando se introduce proyección
```

Set de datos demo

```
use gimnasio

db.clientes.insert([
    {nombre: 'Pilar', apellidos: 'Pérez', edad: 33, dni: '07456322S'},
    {nombre: 'José', apellidos: 'Gómez', edad: 17, dni: '887654321S'},
    {nombre: 'José', apellidos: 'López', edad: 22, dni: '44321567S'},
])
```

### Consulta de condición de igualdad simple
{<campo>: <valor>}

```
db.clientes.find({nombre: 'José'}) \\ case sensitive y diacritic sensitive
db.clientes.find({nombre: 'josé'})
db.clientes.find({nombre: 'Jose'})
```

En el caso de _id

```
db.clientes.find({_id: ObjectId("616d5249d95ac25f419bca1c")}) // En la shell debe pasarse la construcción de ObjectId
db.clientes.find({_id: "616d5249d95ac25f419bca1c"}) // En la shell no encuentra pero en la mayoría de drivers si
```

### Consulta de condición de igualdad múltiple
{<campo>: <valor>, <campo>: <valor>, <campo>: <valor>, ...}

```
db.clientes.find({nombre: 'José', apellidos: 'Gómez'}) // La coma funciona como un AND lógico
// Devuelve todos los docs de la colección cliente en los que nombre sea igual a 'José' Y apellidos sea igual a 'Gómez'
db.clientes.find({nombre: 'José', apellidos: 'Gómez'}) // No importa el orden
```


### Consulta condición con operadores
{<campo>: {<$operador>: <valor>}}

```
db.clientes.find({edad: {$gte: 18}})
```

### Consulta de condiciones múltiples con el operador lógico $and
Para la mayoría de casos es igual a la coma lógica
{$and: [{consulta}, {consulta}]}

```
db.clientes.find({
    $and: [
        {edad: {$gte: 18}},
        {apellidos: 'López'}
    ]
})
```

### Consulta de condiciones múltiples con el operador lógico $or (inclusivo)
{$or: [{consulta}, {consulta}]}

```
db.clientes.find({
    $or: [
        {edad: {$gte: 18}},
        {apellidos: 'López'}
    ]
})
```

Es posible combinar AND (en coma u operador) con $or

```
db.clientes.find({
    apellidos:'Gómez',
    $or: [
        {edad: {$gte: 18}},
        {nombre: 'José'}
    ]
}) // Devuelve los que apellidos es igual a 'Gómez' y o edad es mayor o igual a 18 o nombre es igual a 'José'
```

Nuevo set de datos

```
db.clientes2.insert([
    {
        nombre: 'Celia',
        apellidos: 'Sánchez',
        domicilio: {
            calle: 'Gran Vía, 90',
            cp: '28003',
            localidad: 'Madrid'
        }
    },
    {
        nombre: 'Carlos',
        apellidos: 'Pérez',
        domicilio: {
            calle: 'Alcalá, 200',
            cp: '28010',
            localidad: 'Madrid'
        }
    },
    {
        nombre: 'Inés',
        apellidos: 'Pérez',
        domicilio: {
            calle: 'Burgos, 10',
            cp: '28900',
            localidad: 'Alcorcón'
        }
    },
])
```

### Consulta de igualdad exacta en campo con documento embebido ¡Ojo certificación!

```
db.clientes2.find({domicilio: {calle: 'Burgos, 10', cp: '28900', localidad: 'Alcorcón'}})
```
Si solo se usa el campo que tiene documento embebido el valor tiene que ser exactament el mismo

db.clientes2.find({domicilio: {calle: 'Burgos, 10', localidad: 'Alcorcón', cp: '28900'}}) // No
se puede alterar el orden de los campos

### Consulta de igualdad en campos de subdocumentos
Notación del punto JS entrecomillada

```
db.clientes2.find({"domicilio.localidad": "Madrid"})
```
Devuelve todos los docs en los que el campo localidad del campo domicilio es igual a 'Madrid'

Set de datos

```
db.monitores.insert([
    {nombre: 'Juan', apellidos: 'Pérez', clases: ['padel','esgrima','pesas']},
    {nombre: 'Sara', apellidos: 'Fernández', clases: ['padel','esgrima']},
    {nombre: 'Carlos', apellidos: 'Pérez', clases: ['esgrima','padel']},
    {nombre: 'Luis', apellidos: 'López', clases: [ 'esgrima', 'pesas' ]}
])
```

### Consulta de igualdad exacta en campos con array ¡Ojo certificación!

db.monitores.find({clases: ['padel','esgrima']}) // Exactamente el que tenga ese array y en ese orden

### Consulta de igualdad de un elemento del array ¡Ojo certificación!

db.monitores.find({clases: 'esgrima'}) // Si a un campo que almacena arrays le pasamos un tipo y solo uno
// primitivo devolverá todos los documentos que contengan ese elemento (con indepencia de si es array o no)

Ejemplo no recomendado de utilizar en el mismo documento de consulta
dos veces el mismo campo

db.monitores.find({clases: 'esgrima', clases: 'padel'}) // resultados poco consistentes

### Consulta de igualdad de varios elementos en array
{$all: [<valor>, <valor>, ...]}

db.monitores.find({clases: {$all: ['esgrima','padel']}}) // Devuelve los docs que tengan en el
// campo clases los valores esgrima y padel con indenpendencia del orden y con independencia de
// si hay más valores

### Consultas simples de comparación en arrays

Set de datos

db.clientes3.insert([
    {nombre: 'Carlos', apellidos: 'Pérez', puntuaciones: [100, 120, 44]},
    {nombre: 'Sara', apellidos: 'López', puntuaciones: [60, 90, 70]},
])

db.clientes3.find({puntuaciones: {$gte: 50}}) // Todos los docs que en alguno de sus
// elementos del array puntuación cumplan esa condición

### Consultas múltiples de comparación en arrays ¡Ojo certificación!

db.clientes3.find({puntuaciones: {$gte:50, $lt: 75}}) // Todos los docs en los que alguno de
// se sus elementos cumplan la expresión o una combinación de elementos cumplan esa expresión

Si queremos que al menos un elemento del array satisfaga la expresión (varias condiciones)
tenemos que usar el operador $elemMatch 

db.clientes3.find({puntuaciones: {$elemMatch: {$gte:50, $lt: 75}}})

### Consultas de elementos en posiciones exactas en array
{"<campo>.<posicion>": <valor>}

```
db.monitores.find({"clases.0": "padel"})
```
Devuelve los docs en los que la primera posición del array clases es 'padel'

Set de datos

```
db.clientes4.insert([
    {
        nombre: "Juan",
        apellidos: "García",
        direcciones: [
            {calle: "Alcalá 40", cp: "28001", localidad: "Vigo"},
            {calle: "Zamora, 13", cp: "34005", localidad: "Madrid"}
        ]
    },
    {
        nombre: "Lucía",
        apellidos: "Gómez",
        direcciones: [
            {calle: "Alcalá 60", cp: "28001", localidad: "Madrid"},
            {calle: "Fuencarral, 13", cp: "28002", localidad: "Madrid"}
        ]
    }
])
```

### Consulta de igualdad exacta en arrays de documentos

```
db.clientes4.find({
    direcciones: [
        {calle: "Alcalá 40", cp: "28001", localidad: "Vigo"},
        {calle: "Zamora, 13", cp: "34005", localidad: "Madrid"}
    ]
})
```
Idem arrays de elementos primitivos, si pasamos directamente un valor de tipo array
debe ser exactamente igual al almacenado


### Consulta de igualdad en campos contenidos en subdocumentos de un array
Notación del punto

```
db.clientes4.find({"direcciones.localidad": "Madrid"})
```

Devuelve todos los docs que en alguno de sus subdocumentos del array (con independencia de su posición)
tienen el campo localidad con valor "Madrid"

### Consulta de igualdad en campos contenidos en subdocumentos en una posición concreta de un array
Notación del punto introduciendo la posición

```
db.clientes4.find({"direcciones.0.localidad": "Madrid"})
```

Devuelve todos los docs en los que en el primer subdocumento del array direcciones el campo localidad tiene el valor "Madrid"

Set de datos

db.monitores2.insert([
    {
        nombre: "Luis",
        apellidos: "López",
        actividades: [
            {clase: "aerobic", turno: "mañana", homologado: false},
            {clase: "aerobic", turno: "tarde", homologado: false},
            {clase: "zumba", turno: "mañana", homologado: true},
        ]
    },
    {
        nombre: "María",
        apellidos: "Pérez",
        actividades: [
            {clase: "aerobic", turno: "tarde", homologado: true},
            {clase: "zumba", turno: "tarde", homologado: false},
        ]
    },
    {
        nombre: "Carlos",
        apellidos: "Gónzalez",
        actividades: [
            {clase: "acquagym", turno: "tarde", homologado: true},
            {clase: "zumba", turno: "tarde", homologado: true},
        ]
    },
])

### Consulta de múltiples condiciones en arrays de documentos que pueden cumplir en combinación ¡Ojo Certificación!

```
db.monitores2.find({"actividades.clase": "aerobic", "actividades.homologado": true})
```

Esta consulta puede inducir a error porque podemos pensar que nos devolvería los monitores que
impartan aerobic y estén homologados pero, como el doc puede cumplir la consulta si una combinación
de subdocumentos la cumple, nos devolverá monitores que no estén homologados en aerobic

### Consulta de múltiples condiciones en arrays de documentos en los que el documento debe cumplir todas las condiciones
Usa $elemMatch ¡Ojo certificación!

```
db.monitores2.find({actividades: {$elemMatch:{clase: "aerobic", homologado: true}}})
```

Devuelve los docs que en el array de actividades al menos uno de sus elementos debe cumplir
las dos condiciones

## Proyección
Sintaxis

```
db.<coleccion>.find(
    <doc-consulta>,
    {<campo>: 1 | 0, <campo>: 1 | 0, ...}
)
```

En el documento de proyección, se pasa como 2º argumento, se especifica es campo retornado con valor 1 y campo sin retornar con valor 0, con las siguientes particularidades.

- Solo se puede especificar los que se devuelven (1) o los que no se excluyen (0)
- Solo en el caso del campo _id se puede poner un signo diferente
- Por defecto _id es 1 y se devolverá siempre salvo que lo especifiquemos como 0
- También existen operadores en proyección que permiten devolver campos en función de expresiones

### Devolución de todos los campos especificados y el campo _id

```
db.clientes.find({}, {nombre: 1})
```
Todos los documentos con el campo nombre y el campo _id

```
db.clientes.find({edad: {$gte: 18}},{nombre: 1, apellidos: 1})
```

### Devolución de todos los campos especificados y exclusión del campo _id

```
db.clientes.find({edad: {$gte: 18}},{nombre: 1, apellidos: 1, _id: 0})
```

### Exclusión de todos los campos especificados y el campo _id

```
db.clientes.find({}, {nombre: 0, apellidos: 0})
```

Devuelve todos los documentos con todos los campos excepto los especificados
Si añadiéramos _id: 0 tampoco devolvería _id

### Combinación de inclusión y exclusión solamente se puede hacer con _id

```
db.clientes.find({}, {nombre: 0, apellidos: 1}) // Error 
```
MongoServerError: Cannot do inclusion on field apellidos in exclusion projection

```
db.clientes.find({}, {nombre: 0, apellidos: 0, _id: 1}) // Pero idem anterior porque _id siempre se devuelve por defecto
```

### Proyección de documentos embebidos
Notación del punto también en proyección

```
db.clientes2.find({}, {nombre: 1, "domicilio.localidad": 1, _id: 0})
```
{ nombre: 'Celia', domicilio: { localidad: 'Madrid' } }
{ nombre: 'Carlos', domicilio: { localidad: 'Madrid' } }
{ nombre: 'Inés', domicilio: { localidad: 'Alcorcón' } }

Idem en arrays de documentos

```
db.clientes4.find({}, {nombre: 1, "direcciones.localidad": 1, _id: 0})
```

{ nombre: 'Juan',
  direcciones: [ { localidad: 'Vigo' }, { localidad: 'Madrid' } ] }
{ nombre: 'Lucía',
  direcciones: [ { localidad: 'Madrid' }, { localidad: 'Madrid' } ] }

## Otras consultas

### Consultas de campos con valor null o inexistentes ¡Ojo certificación!

Set datos

db.monitores2.insert({nombre: "Sergio", apellidos: "Pérez"})
db.monitores2.insert({nombre: "Paula", apellidos: "López", actividades: null})

```
db.monitores2.find({actividades: null})
```
Devuelve los que tengan el campo actividades igual a null y los que no no tengan el campo

Si necesitamos estrictamente los null se puede realizar la consulta por tipo de dato
con el operador $type
{$type: <número-del-tipo | alias-del-tipo>}


```
db.monitores2.find({actividades: {$type: 10}}) 
```
Devuelve estrictamente los que tengan el valor null

