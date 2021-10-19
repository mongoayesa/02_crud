# Métodos de borrado de documentos en MongoDB

## Método remove()
Sintaxis

db.<colección>.remove(
    <documento-consulta>,
    justOne: <boolean> // Borra solo la primera coincidencia o todas
)

## Método deleteOne()

db.<colección>.deleteOne(
    <documento-consulta>, // Borra el documento de la primara coincidencia
)

## Método deleteMany()

db.<colección>.deleteMany(
    <documento-consulta>, // Borra todos los documentos coincidentes
)

Lógicamente si seleccionamos todos los documentos la colección se vacía pero se mantiene luego
para eliminar la colección y todos sus metadatos, índices etc habrá que usar

db.<colección>.drop()


