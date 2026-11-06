# Práctica de MongoDB con mongosh

## 1) Conectar a MongoDB

```javascript
mongosh "mongodb://localhost:27017"
```

## 2) Ver bases y crear una base nueva

```javascript
show dbs;
use tienda;
db;
```

## 3) Crear una colección y agregar documentos

```javascript
db.createCollection("productos");

show collections;

db.productos.insertMany([
  { nombre: "Mouse", categoria: "Accesorios", precio: 45, stock: 100 },
  { nombre: "Teclado", categoria: "Accesorios", precio: 120, stock: 60 },
  { nombre: "Laptop", categoria: "Computo", precio: 2500, stock: 15 },
  { nombre: "Monitor", categoria: "Computo", precio: 800, stock: 30 }
]);
```

## 4) Leer datos

```javascript
db.productos.find();

db.productos.find({ categoria: "Computo" });

db.productos.findOne({ nombre: "Mouse" });

db.productos.find({}, { nombre: 1, precio: 1, _id: 0 });

// Ordenar por precio de mayor a menor
 db.productos.find().sort({ precio: -1 }).limit(3);

// Conteo
 db.productos.countDocuments({ categoria: "Computo" });
```

## 5) Actualizar datos

```javascript
db.productos.updateOne({ nombre: "Mouse" }, { $set: { precio: 50 } });

db.productos.updateMany({ categoria: "Accesorios" }, { $inc: { stock: 10 } });

db.productos.updateOne({ nombre: "Teclado" }, { $push: { etiquetas: "mecanico" } });
```

## 6) Eliminar datos

```javascript
db.productos.deleteOne({ nombre: "Webcam" });
db.productos.deleteMany({ stock: { $lt: 5 } });
```

## 7) Operadores de filtro

```javascript
// mayor o igual
db.productos.find({ precio: { $gte: 100, $lte: 1000 } });

// OR
db.productos.find({ $or: [{ stock: { $lt: 20 } }, { precio: { $gt: 2000 } }] });

// expresión regular
db.productos.find({ nombre: { $regex: "^L", $options: "i" } });
```

## 8) Agregación

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

## 9) Índices

```javascript
db.productos.createIndex({ nombre: 1 });
db.productos.getIndexes();
db.productos.dropIndex("nombre_1");
```

## 10) Caso práctico de ventas

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
    detalle: [{ productoId: 3, cantidad: 3, precio: 120 }]
  },
  {
    _id: 3,
    clienteId: 1,
    fecha: new Date("2026-10-20"),
    detalle: [{ productoId: 2, cantidad: 5, precio: 45 }]
  }
]);
```

### Consulta ventas por cliente

```javascript
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
```

## 11) Ejercicio propuesto

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

## 12) Resumen rápido

```javascript
show dbs;
use tienda;
db.productos.find();
db.productos.insertOne({ nombre: "Mouse", precio: 45, stock: 100 });
db.productos.updateOne({ nombre: "Mouse" }, { $set: { precio: 50 } });
db.productos.deleteOne({ nombre: "Mouse" });
```
