# Guía completa de MongoDB

## 1. Introducción

MongoDB es una base de datos NoSQL orientada a documentos. En lugar de tablas y filas, guarda documentos en formato BSON (una versión binaria de JSON) dentro de colecciones. No exige un esquema fijo, por lo que cada documento puede tener campos distintos y escala de forma horizontal.

## 2. Conceptos fundamentales

| MongoDB | Equivalente en SQL | Descripción |
| --- | --- | --- |
| Base de datos | Base de datos | Contenedor de colecciones |
| Colección | Tabla | Conjunto de documentos |
| Documento | Fila | Registro en formato clave-valor |
| Campo | Columna | Dato dentro de un documento |
| _id | Clave primaria | Identificador único de cada documento |
| Índice | Índice | Estructura que acelera las consultas |
| Pipeline de agregación | GROUP BY / JOIN | Cadena de etapas para transformar datos |

Ejemplo de documento:

```javascript
{
  "_id": ObjectId("6728a1f4c2b9e3a1d4f50001"),
  "nombre": "Laptop Lenovo",
  "categoria": "Computo",
  "precio": 2500,
  "stock": 15,
  "etiquetas": ["oficina", "portatil"],
  "proveedor": {
    "nombre": "TecnoSur",
    "ciudad": "Lima"
  }
}
```

## 3. Instalación y conexión

1. Descargar MongoDB Community Server desde el sitio oficial de MongoDB e instalarlo.
2. Instalar MongoDB Shell (`mongosh`) y, si se desea, MongoDB Compass.
3. Verificar la instalación:

```bash
mongod --version
mongosh --version
```

4. Iniciar el servicio:
   - Windows: servicio MongoDB
   - Linux: `sudo systemctl start mongod`

5. Conectarse al servidor local:

```bash
mongosh "mongodb://localhost:27017"
```

El puerto por defecto es `27017`. En Compass, la cadena de conexión es la misma.

## 4. Tipos de datos BSON

| Tipo | Ejemplo | Uso |
| --- | --- | --- |
| String | "Laptop" | Texto |
| Int32 / Int64 / Double | 15, 2500.50 | Números |
| Decimal128 | NumberDecimal("19.99") | Dinero con precisión exacta |
| Boolean | true | Verdadero o falso |
| Date | new Date() | Fecha y hora |
| ObjectId | ObjectId() | Identificador único |
| Array | ["a", "b"] | Listas de valores |
| Objeto embebido | { ciudad: "Lima" } | Documento dentro de otro |
| Null | null | Valor nulo |

## 5. Comandos de bases de datos y colecciones

```javascript
show dbs;                 // lista las bases de datos
use tienda;               // selecciona o crea la base tienda

db;                       // muestra la base actual

db.createCollection("productos");
show collections;         // lista las colecciones

db.productos.renameCollection("articulos");
db.articulos.drop();      // elimina la colección
db.dropDatabase();        // elimina la base de datos actual
db.stats();              // estadísticas de la base
```

> Una base de datos solo aparece en `show dbs` cuando contiene al menos una colección con datos.

## 6. Operaciones CRUD

### 6.1 Crear (insert)

```javascript
db.productos.insertOne({
  nombre: "Mouse",
  categoria: "Accesorios",
  precio: 45,
  stock: 100
});

db.productos.insertMany([
  { nombre: "Teclado", categoria: "Accesorios", precio: 120, stock: 60 },
  { nombre: "Laptop", categoria: "Computo", precio: 2500, stock: 15 },
  { nombre: "Monitor", categoria: "Computo", precio: 800, stock: 30 }
]);
```

### 6.2 Leer (find)

```javascript
db.productos.find();                                 // todos los documentos

db.productos.find({ categoria: "Computo" });        // con filtro

db.productos.findOne({ nombre: "Mouse" });         // un solo documento

db.productos.find({}, { nombre: 1, precio: 1, _id: 0 });

db.productos.find().sort({ precio: -1 }).limit(3); // los 3 más caros

db.productos.find().skip(2).limit(2);             // paginación

db.productos.countDocuments({ categoria: "Computo" });

db.productos.distinct("categoria");
```

### 6.3 Actualizar (update)

```javascript
db.productos.updateOne({ nombre: "Mouse" }, { $set: { precio: 50 } });

db.productos.updateMany({ categoria: "Accesorios" }, { $inc: { stock: 10 } });

db.productos.updateOne({ nombre: "Teclado" }, { $push: { etiquetas: "mecanico" } });

db.productos.updateOne(
  { nombre: "Webcam" },
  { $set: { precio: 150 } },
  { upsert: true }
);

db.productos.replaceOne(
  { nombre: "Monitor" },
  { nombre: "Monitor", categoria: "Computo", precio: 850, stock: 25 }
);
```

### 6.4 Eliminar (delete)

```javascript
db.productos.deleteOne({ nombre: "Webcam" });
db.productos.deleteMany({ stock: { $lt: 5 } });
```

## 7. Operadores

### Comparación

```javascript
{ precio: { $gte: 100, $lte: 1000 } }
```

### Lógicos

```javascript
{ $or: [{ stock: { $lt: 20 } }, { precio: { $gt: 2000 } }] }
```

### Elemento

```javascript
{ proveedor: { $exists: true } }
```

### Texto y patrón

```javascript
{ nombre: { $regex: "^L", $options: "i" } }
```

### Arreglos

```javascript
{ etiquetas: { $all: ["oficina", "portatil"] } }
```

### Actualización

```javascript
{ $unset: { descuento: "" } }
{ $addToSet: { etiquetas: "oferta" } }
```

## 8. Agregación

El pipeline procesa los documentos por etapas. Las más usadas son:

- `$match`: filtrar
- `$group`: agrupar
- `$sort`: ordenar
- `$project`: elegir campos
- `$lookup`: unir colecciones
- `$unwind`: separar arreglos
- `$limit`: limitar resultados

### Ejemplo: valor del inventario por categoría

```javascript
db.productos.aggregate([
  {
    $group: {
      _id: "$categoria",
      productos: { $sum: 1 },
      valorInventario: { $sum: { $multiply: ["$precio", "$stock"] } },
      precioPromedio: { $avg: "$precio" }
    }
  },
  { $sort: { valorInventario: -1 } }
]);
```

### Unión entre colecciones con `$lookup`

```javascript
db.ventas.aggregate([
  {
    $lookup: {
      from: "clientes",
      localField: "clienteId",
      foreignField: "_id",
      as: "cliente"
    }
  },
  { $unwind: "$cliente" },
  { $project: { _id: 0, fecha: 1, total: 1, "cliente.nombre": 1 } }
]);
```

## 9. Índices

Sin índice, MongoDB revisa todos los documentos de la colección. Un índice acelera las búsquedas y los ordenamientos, aunque ocupa espacio y ralentiza un poco las escrituras.

```javascript
db.productos.createIndex({ nombre: 1 });                 // simple, ascendente

db.productos.createIndex({ categoria: 1, precio: -1 }); // compuesto

db.clientes.createIndex({ dni: 1 }, { unique: true });  // único

db.productos.getIndexes();
db.productos.dropIndex("nombre_1");

db.productos.find({ nombre: "Mouse" }).explain("executionStats");
```

## 10. Modelado de datos

| Modelo | Cuándo usarlo | Ejemplo |
| --- | --- | --- |
| Embebido | Los datos se consultan siempre juntos y la relación es de uno a pocos | Detalle de productos dentro de una venta |
| Referenciado | Los datos crecen mucho o se comparten entre documentos | Venta que guarda el `clienteId` |

Recomendación: modelar según las consultas que hará la aplicación, no según las tablas de un esquema relacional. El límite de tamaño de un documento es de 16 MB.

## 11. Validación de esquema

Aunque MongoDB es flexible, se pueden imponer reglas:

```javascript
db.createCollection("clientes", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["nombre", "dni"],
      properties: {
        nombre: { bsonType: "string" },
        dni: { bsonType: "string", minLength: 8, maxLength: 8 },
        email: { bsonType: "string" }
      }
    }
  }
});
```

## 12. Administración y gestión de la información

### 12.1 Importar y exportar datos

```bash
mongoimport --db tienda --collection productos --file productos.json --jsonArray
mongoexport --db tienda --collection productos --out productos.json --jsonArray
```

### 12.2 Copias de seguridad y restauración

```bash
mongodump --db tienda --out C:\respaldo
mongorestore --db tienda C:\respaldo\tienda
```

### 12.3 Usuarios y roles

```javascript
use admin;

db.createUser({
  user: "admin",
  pwd: "CambiarClave123",
  roles: ["userAdminAnyDatabase", "readWriteAnyDatabase"]
});

use tienda;

db.createUser({
  user: "vendedor",
  pwd: "Venta2026",
  roles: [{ role: "readWrite", db: "tienda" }]
});
```

| Rol | Permiso |
| --- | --- |
| read | Solo lectura de la base |
| readWrite | Lectura y escritura |
| dbAdmin | Administración de la base |
| userAdmin | Gestión de usuarios |
| root | Acceso total |

> Para exigir autenticación, activa `security.authorization: enabled` en el archivo de configuración `mongod.conf` y reinicia el servicio.

### 12.4 Monitoreo

```javascript
db.serverStatus();      // estado general del servidor
db.currentOp();        // operaciones en curso
db.productos.stats();   // estadísticas de una colección
```

## 13. Replicación y sharding

- Replica set: varios servidores mantienen copias de los mismos datos (un primario y varios secundarios). Da alta disponibilidad: si el primario falla, se elige otro.
- Sharding: reparte los datos de una colección en varios servidores según una clave de fragmentación. Permite escalar cuando un solo servidor no alcanza.
- MongoDB Atlas: servicio en la nube que administra estos dos mecanismos.

## 14. Aplicación: control de ventas

### 14.1 Datos para poblar

```javascript
use ventasdb;

db.clientes.insertMany([
  { _id: 1, nombre: "Ana Torres", dni: "45871236", ciudad: "Lima" },
  { _id: 2, nombre: "Luis Ramos", dni: "40125698", ciudad: "Arequipa" },
  { _id: 3, nombre: "Rosa Díaz", dni: "47963214", ciudad: "Cusco" }
]);

db.productos.insertMany([
  { _id: 1, nombre: "Laptop", precio: 2500, stock: 15 },
  { _id: 2, nombre: "Mouse", precio: 45, stock: 100 },
  { _id: 3, nombre: "Teclado", precio: 120, stock: 60 }
]);

db.ventas.insertMany([
  {
    _id: 1,
    clienteId: 1,
    fecha: new Date("2026-10-05"),
    detalle: [
      { productoId: 1, cantidad: 1, precio: 2500 },
      { productoId: 2, cantidad: 2, precio: 45 }
    ]
  },
  {
    _id: 2,
    clienteId: 2,
    fecha: new Date("2026-10-12"),
    detalle: [
      { productoId: 3, cantidad: 3, precio: 120 }
    ]
  },
  {
    _id: 3,
    clienteId: 1,
    fecha: new Date("2026-10-20"),
    detalle: [
      { productoId: 2, cantidad: 5, precio: 45 }
    ]
  }
]);
```

### 14.2 Consultas de gestión

```javascript
// Ventas del cliente 1
db.ventas.find({ clienteId: 1 });

// Total de cada venta
db.ventas.aggregate([
  {
    $project: {
      fecha: 1,
      total: {
        $sum: {
          $map: {
            input: "$detalle",
            as: "d",
            in: { $multiply: ["$$d.cantidad", "$$d.precio"] }
          }
        }
      }
    }
  }
]);

// Producto más vendido
db.ventas.aggregate([
  { $unwind: "$detalle" },
  { $group: { _id: "$detalle.productoId", unidades: { $sum: "$detalle.cantidad" } } },
  { $sort: { unidades: -1 } },
  { $limit: 1 }
]);

// Ventas por cliente con su nombre
db.ventas.aggregate([
  { $unwind: "$detalle" },
  {
    $group: {
      _id: "$clienteId",
      total: { $sum: { $multiply: ["$detalle.cantidad", "$detalle.precio"] } }
    }
  },
  { $lookup: { from: "clientes", localField: "_id", foreignField: "_id", as: "cliente" } },
  { $unwind: "$cliente" },
  { $project: { _id: 0, cliente: "$cliente.nombre", total: 1 } }
]);

// Descontar stock tras una venta
db.productos.updateOne({ _id: 2 }, { $inc: { stock: -5 } });
```

Resultado esperado de la consulta final: Ana Torres con 2975 y Luis Ramos con 360 en ventas acumuladas.

## 15. Ventajas y desventajas

| Ventajas | Desventajas |
| --- | --- |
| Esquema flexible, fácil de cambiar | Sin integridad referencial automática |
| Escala de forma horizontal | Los datos pueden duplicarse al embeber |
| Consultas rápidas con índices | Uniones menos naturales que en SQL |
| Formato JSON cercano al código de la aplicación | Requiere diseñar bien el modelo para no perder rendimiento |
| Replicación y alta disponibilidad integradas | Mayor consumo de memoria que una base relacional |

## 16. Ejercicio propuesto

1. Crear la base `biblioteca` con las colecciones `libros` y `socios`.
2. Insertar 5 libros y 3 socios con `insertMany`.
3. Buscar los libros con stock menor a 3 y ordenarlos por título.
4. Crear un índice único sobre el DNI de los socios.
5. Con una agregación, contar cuántos libros hay por género.
6. Respaldar la base con `mongodump` y restaurarla en otra base.

### Ejemplo de solución

```javascript
use biblioteca;

db.libros.insertMany([
  { titulo: "El alquimista", genero: "Ficcion", stock: 2 },
  { titulo: "Clean Code", genero: "Tecnologia", stock: 5 },
  { titulo: "Dune", genero: "Ciencia Ficción", stock: 1 },
  { titulo: "Cien años de soledad", genero: "Realismo mágico", stock: 3 },
  { titulo: "Python para todos", genero: "Tecnologia", stock: 4 }
]);

db.socios.insertMany([
  { nombre: "Ana", dni: "12345678" },
  { nombre: "Luis", dni: "87654321" },
  { nombre: "Marta", dni: "45678912" }
]);

db.libros.find({ stock: { $lt: 3 } }).sort({ titulo: 1 });

db.socios.createIndex({ dni: 1 }, { unique: true });

db.libros.aggregate([
  { $group: { _id: "$genero", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
]);
```

## 17. Conclusiones

- MongoDB guarda la información en documentos flexibles agrupados en colecciones.
- Los comandos CRUD, los operadores y la agregación cubren la mayoría de las necesidades de una aplicación.
- Los índices, los roles de usuario y las copias de seguridad son esenciales para gestionar los datos con rendimiento y seguridad.

## Recursos recomendados

- [Documentación oficial de MongoDB](https://www.mongodb.com/docs/)
- [MongoDB Shell (mongosh)](https://www.mongodb.com/docs/mongodb-shell/)
- [MongoDB Atlas](https://www.mongodb.com/atlas)

---

Si quieres, puedo convertir esta guía en una versión más corta para estudio, una versión con ejercicios resueltos o un archivo de notas para clase en formato Markdown o PDF.
