# Métodos de cursor en MongoDB

db.<coleccion>.metodo1().metodo2()...

## Método count()
Devuelve entero con el número de documentos de la colección

```
use gimansio

db.clientes.find({apellidos: 'Gómez'}).count()
```

## Método skip()
Recibe entero para omitir de la consulta los n primeros documentos

```
db.clientes.find({}).skip(3)
```

## Método limit()
Recibe entero para limitar la consulta a los n documentos

```
db.clientes.find({}).limit(4)
```

Son habitualmente usados juntos en los correspondientes métodos del driver

skip(n).limit(n)

## Método sort()
db.<coleccion>.find(<consulta>,<proyeccion>).sort({<campo>: 1 | -1, <campo>: 1 | -1, ...})

```
db.clientes.find({},{apellidos:1, _id: 0}).sort({apellidos: -1})
```

{ apellidos: 'Pérez' }
{ apellidos: 'López' }
{ apellidos: 'Gómez' }
{}
{}
{}

Para devolver teniendo en cuenta el orden de inserción

```
db.clientes.find().sort({$natural: -1})
```

## Métodos distinct
Devuelve los distintos valores de un campo
db.<coleccion>.distinct(<campo>)

```
db.clientes.distinct('apellidos')
```