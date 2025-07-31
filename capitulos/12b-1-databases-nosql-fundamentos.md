# Capítulo 12B-1: Bases de Datos NoSQL - Fundamentos y Bases de Datos de Documentos

## 12B.1 Introducción a NoSQL

### 12B.1.1 ¿Qué es NoSQL?

NoSQL (Not Only SQL) se refiere a una amplia clase de sistemas de gestión de bases de datos que difieren del modelo relacional tradicional. Estos sistemas están diseñados para escenarios específicos y ofrecen modelos de datos más flexibles que las bases de datos relacionales.

**Características principales:**
- Esquemas flexibles o sin esquema
- Escalabilidad horizontal sencilla
- Optimizados para modelos de datos específicos
- Alta disponibilidad y tolerancia a fallos
- Rendimiento optimizado para casos de uso particulares

### 12B.1.2 Historia y evolución

- **Antecedentes (1960s-1990s)**: Sistemas pre-relacionales como bases de datos jerárquicas y de red
- **Primeras bases de datos NoSQL (1998)**: El término "NoSQL" apareció por primera vez
- **Auge (2000s)**: Surgimiento de servicios web a gran escala (Google, Amazon)
- **Puntos de inflexión**:
  - 2004: Publicación del paper BigTable de Google
  - 2007: Paper Dynamo de Amazon
  - 2009: Conferencia NoSQL en San Francisco establece el término
- **Consolidación (2010s)**: Maduración de las soluciones y adopción empresarial
- **Actualidad**: Convivencia con bases de datos SQL en arquitecturas poliglot persistence

### 12B.1.3 El teorema CAP

El teorema CAP (Consistency, Availability, Partition Tolerance) establece que un sistema distribuido no puede garantizar simultáneamente estas tres propiedades:

- **Consistencia (C)**: Todos los nodos ven los mismos datos al mismo tiempo
- **Disponibilidad (A)**: Cada petición recibe una respuesta (sin error)
- **Tolerancia a particiones (P)**: El sistema sigue funcionando a pesar de fallos en la comunicación entre nodos

**Categorías según CAP:**
- **CA**: Consistencia y disponibilidad (PostgreSQL, sistemas tradicionales)
- **CP**: Consistencia y tolerancia a particiones (MongoDB, HBase)
- **AP**: Disponibilidad y tolerancia a particiones (Cassandra, CouchDB)

### 12B.1.4 Clasificación de bases de datos NoSQL

#### Por modelo de datos

1. **Bases de datos de documentos**
   - Almacenan documentos semiestructurados (JSON, BSON)
   - Ejemplos: MongoDB, CouchDB, Firestore

2. **Bases de datos clave-valor**
   - Simples pares clave-valor
   - Ejemplos: Redis, DynamoDB, Riak

3. **Bases de datos columnares**
   - Almacenan datos en columnas en lugar de filas
   - Ejemplos: Cassandra, HBase, ScyllaDB

4. **Bases de datos de grafos**
   - Optimizadas para relaciones entre entidades
   - Ejemplos: Neo4j, JanusGraph, ArangoDB

5. **Bases de datos multi-modelo**
   - Combinan varios modelos de datos
   - Ejemplos: ArangoDB, OrientDB, FaunaDB

#### Por arquitectura

1. **Distribuidas**: Diseñadas para funcionar en múltiples nodos (Cassandra, MongoDB)
2. **En memoria**: Almacenamiento primario en RAM (Redis, Aerospike)
3. **Embebidas**: Funcionan dentro de una aplicación (LevelDB, SQLite)

### 12B.1.5 SQL vs NoSQL

| Característica       | SQL                         | NoSQL                      |
|----------------------|-----------------------------|----------------------------|
| Modelo de datos      | Tablas relacionales         | Variado (documentos, KV, etc.) |
| Esquema              | Rígido y predefinido        | Flexible o sin esquema     |
| Escalabilidad        | Principalmente vertical     | Principalmente horizontal  |
| Transacciones        | ACID completo               | BASE/ACID parcial o configurable |
| Consultas            | SQL estándar                | APIs específicas           |
| Casos de uso         | Datos estructurados, transaccionales | Big Data, tiempo real, alta disponibilidad |
| Consistencia         | Fuerte                      | Variable (fuerte a eventual) |

**Cuándo usar NoSQL:**
- Volúmenes de datos extremadamente grandes
- Estructuras de datos variables o desconocidas
- Necesidad de escalabilidad horizontal sencilla
- Requisitos de rendimiento extremos
- Casos de uso específicos alineados con un modelo de datos particular

**Cuándo usar SQL:**
- Datos fuertemente estructurados y relacionales
- Necesidad de transacciones ACID
- Consultas complejas con joins y agregaciones
- Requisitos estrictos de consistencia
- Reportes y análisis complejos

## 12B.2 Bases de Datos de Documentos

Las bases de datos documentales almacenan datos en documentos semiestructurados, típicamente en formatos como JSON o BSON (Binary JSON). Este modelo se adapta bien a datos jerárquicos y variables.

### 12B.2.1 Conceptos clave

**Terminología:**
- **Documento**: Unidad básica de datos (equivalente a una fila en SQL)
- **Colección**: Grupo de documentos (similar a una tabla)
- **Campo**: Par clave-valor dentro de un documento
- **Documentos embebidos**: Documentos anidados dentro de otros documentos

**Características principales:**
- Esquema flexible (cada documento puede tener estructura diferente)
- Datos autocontenidos (menos necesidad de joins)
- Consultas sobre estructuras jerárquicas
- Índices secundarios para optimización

### 12B.2.2 MongoDB

MongoDB es la base de datos documental más popular, diseñada para facilitar el desarrollo y escalar con facilidad.

#### Estructura básica

```javascript
// Documento en MongoDB
{
  "_id": ObjectId("5f8e46c3b9e02d8a1c491d57"),
  "nombre": "Juan Pérez",
  "email": "juan@ejemplo.com",
  "direccion": {
    "calle": "Av. Principal 123",
    "ciudad": "Madrid",
    "codigo_postal": "28001"
  },
  "telefonos": ["555-1234", "555-5678"],
  "activo": true,
  "puntos": 150,
  "fecha_registro": ISODate("2023-01-15T14:30:45.123Z")
}
```

#### Operaciones CRUD

```javascript
// Crear (insertar) documentos
db.usuarios.insertOne({
  nombre: "Ana López",
  email: "ana@ejemplo.com",
  activo: true
});

// Leer (consultar) documentos
db.usuarios.find({ activo: true });
db.usuarios.findOne({ email: "ana@ejemplo.com" });

// Actualizar documentos
db.usuarios.updateOne(
  { email: "ana@ejemplo.com" },
  { $set: { puntos: 200 } }
);

// Eliminar documentos
db.usuarios.deleteOne({ email: "ana@ejemplo.com" });
```

#### Consultas avanzadas

```javascript
// Consultas con operadores
db.productos.find({
  precio: { $gt: 100, $lt: 500 },
  categoria: { $in: ["electrónica", "computadoras"] }
});

// Proyecciones (seleccionar campos)
db.usuarios.find(
  { activo: true },
  { nombre: 1, email: 1, _id: 0 }  // Incluir nombre y email, excluir _id
);

// Agregaciones
db.ventas.aggregate([
  { $match: { fecha: { $gte: new Date("2023-01-01") } } },
  { $group: {
      _id: "$categoria",
      total: { $sum: "$monto" },
      promedio: { $avg: "$monto" },
      count: { $sum: 1 }
    }
  },
  { $sort: { total: -1 } }
]);
```

#### Indexación

```javascript
// Índice simple
db.usuarios.createIndex({ email: 1 });  // 1 para ascendente, -1 para descendente

// Índice compuesto
db.productos.createIndex({ categoria: 1, nombre: 1 });

// Índice único
db.usuarios.createIndex({ email: 1 }, { unique: true });

// Índice de texto
db.articulos.createIndex({ contenido: "text" });

// Consulta con índice de texto
db.articulos.find({ $text: { $search: "mongodb nosql" } });
```

#### Ejemplo de conexión desde Node.js

```javascript
const { MongoClient } = require('mongodb');

async function main() {
  const uri = "mongodb://localhost:27017";
  const client = new MongoClient(uri);

  try {
    await client.connect();
    const database = client.db("miapp");
    const usuarios = database.collection("usuarios");

    // Insertar un documento
    const resultado = await usuarios.insertOne({
      nombre: "Carlos Ruiz",
      email: "carlos@ejemplo.com",
      fecha_registro: new Date()
    });
    console.log(`Documento insertado con ID: ${resultado.insertedId}`);

    // Buscar documentos
    const cursor = usuarios.find({ nombre: { $regex: /Carlos/ } });
    const resultados = await cursor.toArray();
    console.log("Documentos encontrados:", resultados);
    
  } finally {
    await client.close();
  }
}

main().catch(console.error);
```

#### Modelado de datos en MongoDB

**Estrategias de modelado:**

1. **Documentos embebidos** (desnormalización):

```javascript
// Modelo embebido (desnormalizado)
{
  "_id": ObjectId("5f8e46c3b9e02d8a1c491d57"),
  "titulo": "Guía de MongoDB",
  "autor": {
    "nombre": "Ana García",
    "email": "ana@ejemplo.com",
    "bio": "Experta en bases de datos NoSQL"
  },
  "comentarios": [
    {
      "usuario": "Pedro",
      "texto": "Excelente artículo",
      "fecha": ISODate("2023-03-15T10:30:00Z")
    },
    {
      "usuario": "María",
      "texto": "Muy útil",
      "fecha": ISODate("2023-03-16T14:20:00Z")
    }
  ]
}
```

2. **Referencias** (normalización):

```javascript
// Colección autores
{
  "_id": ObjectId("5f8e46c3b9e02d8a1c491d57"),
  "nombre": "Ana García",
  "email": "ana@ejemplo.com",
  "bio": "Experta en bases de datos NoSQL"
}

// Colección articulos
{
  "_id": ObjectId("6a1b23c4d5e6f7g8h9i0j1k2"),
  "titulo": "Guía de MongoDB",
  "autor_id": ObjectId("5f8e46c3b9e02d8a1c491d57"),
  "contenido": "..."
}

// Colección comentarios
{
  "_id": ObjectId("7b2c34d5e6f7g8h9i0j1k2l3"),
  "articulo_id": ObjectId("6a1b23c4d5e6f7g8h9i0j1k2"),
  "usuario": "Pedro",
  "texto": "Excelente artículo",
  "fecha": ISODate("2023-03-15T10:30:00Z")
}
```

**Cuándo usar cada enfoque:**

- **Embebidos** (desnormalización):
  - Relaciones "uno a pocos"
  - Datos leídos juntos frecuentemente
  - Datos que no crecen indefinidamente
  - Prioridad en rendimiento de lectura

- **Referencias** (normalización):
  - Relaciones "uno a muchos" o "muchos a muchos"
  - Datos que crecen continuamente
  - Datos que se actualizan frecuentemente
  - Necesidad de acceso independiente

#### Transacciones en MongoDB

MongoDB soporta transacciones multi-documento desde la versión 4.0:

```javascript
// Iniciar sesión
const session = client.startSession();

// Transacción
try {
  session.startTransaction();
  
  await db.collection('cuentas').updateOne(
    { _id: cuenta1Id },
    { $inc: { saldo: -100 } },
    { session }
  );
  
  await db.collection('cuentas').updateOne(
    { _id: cuenta2Id },
    { $inc: { saldo: 100 } },
    { session }
  );
  
  await db.collection('transacciones').insertOne(
    {
      from: cuenta1Id,
      to: cuenta2Id,
      monto: 100,
      fecha: new Date()
    },
    { session }
  );
  
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  console.error('Error en la transacción:', error);
} finally {
  session.endSession();
}
```

### 12B.2.3 Mongoose: ODM para MongoDB

Mongoose es un Object Document Mapper (ODM) para MongoDB y Node.js que proporciona una solución basada en esquemas para modelar los datos.

#### Definición de esquemas

```javascript
const mongoose = require('mongoose');

// Definir esquema
const usuarioSchema = new mongoose.Schema({
  nombre: {
    type: String,
    required: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
  },
  contraseña: {
    type: String,
    required: true,
    minlength: 8
  },
  activo: {
    type: Boolean,
    default: true
  },
  rol: {
    type: String,
    enum: ['usuario', 'admin', 'editor'],
    default: 'usuario'
  },
  direccion: {
    calle: String,
    ciudad: String,
    codigo_postal: String
  },
  fecha_registro: {
    type: Date,
    default: Date.now
  }
});

// Añadir métodos al esquema
usuarioSchema.methods.nombreCompleto = function() {
  return `${this.nombre.nombre} ${this.nombre.apellido}`;
};

// Middleware (hooks)
usuarioSchema.pre('save', async function(next) {
  if (this.isModified('contraseña')) {
    this.contraseña = await bcrypt.hash(this.contraseña, 8);
  }
  next();
});

// Crear modelo
const Usuario = mongoose.model('Usuario', usuarioSchema);

module.exports = Usuario;
```

#### Operaciones básicas con Mongoose

```javascript
// Conexión
mongoose.connect('mongodb://localhost:27017/miapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true
});

// Crear documento
async function crearUsuario() {
  try {
    const usuario = new Usuario({
      nombre: 'María López',
      email: 'maria@ejemplo.com',
      contraseña: 'contraseña123',
      direccion: {
        calle: 'Calle Secundaria 456',
        ciudad: 'Barcelona',
        codigo_postal: '08001'
      }
    });
    
    await usuario.save();
    console.log('Usuario creado:', usuario);
  } catch (error) {
    console.error('Error al crear usuario:', error);
  }
}

// Buscar documentos
async function buscarUsuarios() {
  try {
    // Encontrar todos
    const usuarios = await Usuario.find();
    
    // Filtrar
    const usuariosActivos = await Usuario
      .find({ activo: true })
      .select('nombre email')
      .sort({ fecha_registro: -1 })
      .limit(10);
    
    // Buscar por ID
    const usuario = await Usuario.findById('123456789');
    
    // Buscar uno
    const admin = await Usuario.findOne({ rol: 'admin' });
    
    return usuariosActivos;
  } catch (error) {
    console.error('Error al buscar usuarios:', error);
  }
}

// Actualizar documento
async function actualizarUsuario(id, datos) {
  try {
    const usuario = await Usuario.findByIdAndUpdate(
      id, 
      datos, 
      { new: true, runValidators: true }
    );
    return usuario;
  } catch (error) {
    console.error('Error al actualizar usuario:', error);
  }
}

// Eliminar documento
async function eliminarUsuario(id) {
  try {
    await Usuario.findByIdAndDelete(id);
    console.log('Usuario eliminado');
  } catch (error) {
    console.error('Error al eliminar usuario:', error);
  }
}
```

#### Relaciones en Mongoose

```javascript
// Referencias
const articuloSchema = new mongoose.Schema({
  titulo: String,
  contenido: String,
  autor: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Usuario',
    required: true
  },
  tags: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Tag'
  }]
});

// Población (similar a JOIN)
async function getArticulosConAutor() {
  const articulos = await Articulo
    .find()
    .populate('autor', 'nombre email')
    .populate('tags', 'nombre');
  
  return articulos;
}

// Subdocumentos (documentos embebidos)
const productoSchema = new mongoose.Schema({
  nombre: String,
  precio: Number,
  reviews: [{
    usuario: String,
    estrellas: Number,
    comentario: String,
    fecha: {
      type: Date,
      default: Date.now
    }
  }]
});

// Añadir un subdocumento
async function addReview(productoId, review) {
  const producto = await Producto.findById(productoId);
  producto.reviews.push(review);
  await producto.save();
}
```

### 12B.2.4 Firebase Firestore

Firestore es una base de datos documental en la nube, parte de la plataforma Firebase de Google, diseñada para aplicaciones móviles y web.

#### Características principales

- Sin servidor (serverless)
- Sincronización en tiempo real
- Modo offline
- Seguridad basada en reglas declarativas
- Escalado automático

#### Estructura de datos

```javascript
// Colección "usuarios"
// Documento "user123"
{
  "nombre": "Laura Martínez",
  "email": "laura@ejemplo.com",
  "nivel": 2,
  "premium": false,
  "ultimoAcceso": Timestamp(1625097600, 0)
}

// Colección "usuarios/user123/pedidos"
// Documento "order456"
{
  "producto": "Laptop",
  "cantidad": 1,
  "precio": 1200.00,
  "estado": "enviado",
  "fecha": Timestamp(1625097600, 0)
}
```

#### Operaciones básicas con Firestore (JavaScript)

```javascript
// Importación
import { initializeApp } from "firebase/app";
import { getFirestore, collection, doc, getDoc, setDoc, updateDoc, deleteDoc, query, where, orderBy, limit } from "firebase/firestore";

// Inicialización
const firebaseConfig = {
  apiKey: "...",
  authDomain: "miapp.firebaseapp.com",
  projectId: "miapp",
  // ...
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// Crear o sobrescribir un documento
async function crearUsuario(id, datos) {
  try {
    await setDoc(doc(db, "usuarios", id), datos);
    console.log("Usuario creado con ID:", id);
  } catch (error) {
    console.error("Error al crear usuario:", error);
  }
}

// Leer un documento
async function obtenerUsuario(id) {
  try {
    const docRef = doc(db, "usuarios", id);
    const docSnap = await getDoc(docRef);
    
    if (docSnap.exists()) {
      return docSnap.data();
    } else {
      console.log("No existe el documento");
      return null;
    }
  } catch (error) {
    console.error("Error al obtener usuario:", error);
  }
}

// Actualizar un documento
async function actualizarUsuario(id, datos) {
  try {
    const docRef = doc(db, "usuarios", id);
    await updateDoc(docRef, datos);
    console.log("Usuario actualizado");
  } catch (error) {
    console.error("Error al actualizar usuario:", error);
  }
}

// Eliminar un documento
async function eliminarUsuario(id) {
  try {
    await deleteDoc(doc(db, "usuarios", id));
    console.log("Usuario eliminado");
  } catch (error) {
    console.error("Error al eliminar usuario:", error);
  }
}

// Consultas
async function consultarUsuarios() {
  try {
    const usuariosRef = collection(db, "usuarios");
    
    // Consulta simple
    const q = query(
      usuariosRef,
      where("premium", "==", true),
      orderBy("nivel", "desc"),
      limit(10)
    );
    
    const querySnapshot = await getDocs(q);
    const usuarios = [];
    
    querySnapshot.forEach((doc) => {
      usuarios.push({ id: doc.id, ...doc.data() });
    });
    
    return usuarios;
  } catch (error) {
    console.error("Error en la consulta:", error);
  }
}
```

#### Escucha en tiempo real

```javascript
// Escuchar cambios en un documento
const unsub = onSnapshot(doc(db, "usuarios", "user123"), (doc) => {
  console.log("Datos actuales:", doc.data());
});

// Detener escucha
// unsub();

// Escuchar cambios en una colección
const q = query(
  collection(db, "usuarios"),
  where("premium", "==", true)
);

const unsubscribe = onSnapshot(q, (snapshot) => {
  snapshot.docChanges().forEach((change) => {
    if (change.type === "added") {
      console.log("Nuevo usuario:", change.doc.data());
    }
    if (change.type === "modified") {
      console.log("Usuario modificado:", change.doc.data());
    }
    if (change.type === "removed") {
      console.log("Usuario eliminado:", change.doc.id);
    }
  });
});
```

#### Reglas de seguridad

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Autenticación requerida para todos los documentos
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
    
    // Reglas específicas para la colección usuarios
    match /usuarios/{userId} {
      // Los usuarios solo pueden leer/escribir sus propios documentos
      allow read, update, delete: if request.auth.uid == userId;
      allow create: if request.auth != null;
      
      // Validar datos en actualizaciones
      allow update: if request.resource.data.nombre is string &&
                       request.resource.data.nombre.size() <= 100;
    }
    
    // Reglas para subcollecciones
    match /usuarios/{userId}/pedidos/{orderId} {
      allow read, write: if request.auth.uid == userId;
    }
    
    // Reglas para documentos públicos
    match /productos/{productoId} {
      allow read: if true;  // Lectura pública
      allow write: if request.auth != null && 
                      get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.admin == true;
    }
  }
}
```

### 12B.2.5 CouchDB

Apache CouchDB es una base de datos documental que utiliza JSON para documentos, HTTP para API y JavaScript para mapeo y reducción.

#### Características principales

- Modelo de consistencia eventual
- Replicación master-master
- HTTP REST API
- MapReduce para vistas
- Cambios incrementales (feeds)

#### Operaciones básicas (HTTP API)

```javascript
// Crear base de datos
fetch('http://localhost:5984/miapp', { 
  method: 'PUT',
  headers: { 'Authorization': 'Basic ' + btoa('admin:password') }
})
.then(res => res.json())
.then(data => console.log(data));

// Crear documento
fetch('http://localhost:5984/miapp/doc1', { 
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Basic ' + btoa('admin:password')
  },
  body: JSON.stringify({
    nombre: 'Luisa Gómez',
    email: 'luisa@ejemplo.com',
    edad: 28
  })
})
.then(res => res.json())
.then(data => console.log(data));

// Obtener documento
fetch('http://localhost:5984/miapp/doc1', {
  headers: { 'Authorization': 'Basic ' + btoa('admin:password') }
})
.then(res => res.json())
.then(data => console.log(data));

// Actualizar documento
fetch('http://localhost:5984/miapp/doc1?rev=1-2902191555', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Basic ' + btoa('admin:password')
  },
  body: JSON.stringify({
    nombre: 'Luisa Gómez',
    email: 'luisa.nueva@ejemplo.com',
    edad: 29,
    _rev: '1-2902191555'  // Revisión actual
  })
})
.then(res => res.json())
.then(data => console.log(data));
```

#### Vistas y MapReduce

```javascript
// Definir una vista
fetch('http://localhost:5984/miapp/_design/usuarios', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Basic ' + btoa('admin:password')
  },
  body: JSON.stringify({
    views: {
      por_edad: {
        map: function(doc) {
          if (doc.edad) {
            emit(doc.edad, doc.nombre);
          }
        }.toString()
      },
      conteo_por_dominio: {
        map: function(doc) {
          if (doc.email) {
            var dominio = doc.email.split('@')[1];
            emit(dominio, 1);
          }
        }.toString(),
        reduce: function(keys, values, rereduce) {
          return sum(values);
        }.toString()
      }
    }
  })
})
.then(res => res.json())
.then(data => console.log(data));

// Consultar vista
fetch('http://localhost:5984/miapp/_design/usuarios/_view/por_edad?startkey=20&endkey=30', {
  headers: { 'Authorization': 'Basic ' + btoa('admin:password') }
})
.then(res => res.json())
.then(data => console.log(data));
```

## 12B.3 Conclusión

Las bases de datos de documentos ofrecen un equilibrio entre flexibilidad y estructura que las hace ideales para muchas aplicaciones modernas. Su capacidad para manejar datos semiestructurados, combinada con escalabilidad horizontal y características avanzadas como consultas complejas e índices, las convierte en una opción popular para desarrollo web y móvil.

MongoDB se ha establecido como el líder en este espacio, con una comunidad activa y un ecosistema robusto, mientras que alternativas como Firestore ofrecen soluciones gestionadas en la nube que simplifican el despliegue y la administración.

Al elegir una base de datos documental, es importante considerar factores como el modelo de consistencia requerido, la escalabilidad necesaria, y las características específicas que mejor se adapten a los patrones de acceso de datos de tu aplicación.
