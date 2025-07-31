# Capítulo 12B-3: Bases de Datos NoSQL - Grafos y Tendencias Actuales

## 12B.7 Bases de Datos de Grafos

Las bases de datos de grafos están diseñadas para representar y almacenar relaciones complejas entre datos. Utilizan estructuras de grafos con nodos, aristas y propiedades para representar y almacenar información.

### 12B.7.1 Conceptos clave

**Componentes principales:**
- **Nodos (vértices)**: Entidades con propiedades
- **Relaciones (aristas)**: Conexiones entre nodos, con dirección, tipo y propiedades
- **Propiedades**: Atributos tanto de nodos como de relaciones
- **Etiquetas**: Clasificadores para nodos y relaciones

**Características principales:**
- Rendimiento constante para recorridos de grafos, independiente del tamaño total
- Almacenamiento y consulta nativos de relaciones (sin JOINs)
- Modelo de datos intuitivo y cercano a la estructura real de muchos dominios
- Flexibilidad para añadir nuevos tipos de relaciones sin afectar la estructura existente

**Casos de uso ideales:**
- Redes sociales
- Sistemas de recomendación
- Detección de fraude
- Gestión de conocimiento
- Mapeo de redes
- Análisis de impacto
- Ruteo y logística

### 12B.7.2 Neo4j

Neo4j es la base de datos de grafos más popular, ofreciendo un motor de grafos nativo con ACID completo.

#### Modelo de datos

![Modelo de datos Neo4j](https://neo4j.com/docs/getting-started/current/graphdb-concepts/graphdb-concepts.png)

#### Lenguaje de consulta: Cypher

Cypher es un lenguaje de consulta declarativo para grafos, similar en concepto a SQL pero optimizado para estructuras de grafos.

```cypher
// Crear nodos
CREATE (p:Persona {nombre: 'Juan', edad: 30})
CREATE (e:Empresa {nombre: 'TechCorp', sector: 'Tecnología'})

// Crear relaciones
MATCH
  (p:Persona {nombre: 'Juan'}),
  (e:Empresa {nombre: 'TechCorp'})
CREATE (p)-[:TRABAJA_EN {desde: 2020, cargo: 'Desarrollador'}]->(e)

// Consulta simple
MATCH (p:Persona)-[r:TRABAJA_EN]->(e:Empresa)
RETURN p.nombre, r.cargo, e.nombre

// Consulta con filtros
MATCH (p:Persona)-[r:TRABAJA_EN]->(e:Empresa)
WHERE p.edad > 25 AND e.sector = 'Tecnología'
RETURN p.nombre, r.cargo, e.nombre, p.edad

// Consulta de caminos (hasta 3 niveles)
MATCH (p1:Persona {nombre: 'Juan'})-[*1..3]-(p2:Persona)
RETURN p1.nombre, p2.nombre
```

#### Patrones comunes de consulta

1. **Búsqueda de nodos**:
```cypher
MATCH (p:Persona {nombre: 'Juan'})
RETURN p
```

2. **Relaciones directas**:
```cypher
MATCH (p:Persona {nombre: 'Juan'})-[r:CONOCE]->(amigo)
RETURN amigo.nombre
```

3. **Caminos variables**:
```cypher
MATCH path = (:Persona {nombre: 'Juan'})-[:CONOCE*1..3]->(:Persona {nombre: 'Ana'})
RETURN path
```

4. **Recomendaciones (amigos de amigos)**:
```cypher
MATCH (p:Persona {nombre: 'Juan'})-[:CONOCE]->(amigo)-[:CONOCE]->(amigo_de_amigo)
WHERE NOT (p)-[:CONOCE]->(amigo_de_amigo)
AND p <> amigo_de_amigo
RETURN amigo_de_amigo.nombre, count(amigo) as conexiones
ORDER BY conexiones DESC
```

5. **Agrupación y conteo**:
```cypher
MATCH (p:Persona)-[:VIVE_EN]->(c:Ciudad)
RETURN c.nombre, count(p) as habitantes
ORDER BY habitantes DESC
```

#### Ejemplo con Neo4j JavaScript Driver

```javascript
const neo4j = require('neo4j-driver');

// Crear un driver
const driver = neo4j.driver(
  'bolt://localhost:7687',
  neo4j.auth.basic('neo4j', 'password')
);

async function ejemploNeo4j() {
  const session = driver.session();
  
  try {
    // Crear datos
    await session.run(`
      CREATE (p:Producto {nombre: 'Smartphone X', precio: 599.99})
      CREATE (c:Categoría {nombre: 'Electrónica'})
      CREATE (p)-[:PERTENECE_A]->(c)
    `);
    
    // Consultar datos
    const result = await session.run(`
      MATCH (p:Producto)-[:PERTENECE_A]->(c:Categoría)
      RETURN p.nombre AS producto, p.precio AS precio, c.nombre AS categoría
    `);
    
    // Procesar resultados
    result.records.forEach(record => {
      console.log(
        `Producto: ${record.get('producto')}, ` +
        `Precio: ${record.get('precio')}, ` +
        `Categoría: ${record.get('categoría')}`
      );
    });
    
    // Análisis de grafos
    const centralidad = await session.run(`
      MATCH (p:Persona)
      RETURN p.nombre AS nombre,
             size((p)-[:CONOCE]->()) AS salientes,
             size((p)<-[:CONOCE]-()) AS entrantes
      ORDER BY entrantes DESC
    `);
    
    console.log('Análisis de centralidad:');
    centralidad.records.forEach(record => {
      console.log(
        `${record.get('nombre')}: ` +
        `${record.get('entrantes')} conexiones entrantes, ` +
        `${record.get('salientes')} conexiones salientes`
      );
    });
    
  } finally {
    await session.close();
  }
  
  // Cerrar el driver al finalizar la aplicación
  await driver.close();
}

ejemploNeo4j().catch(error => {
  console.error('Error:', error);
  process.exit(1);
});
```

### 12B.7.3 Modelado de datos con grafos

#### Principios de modelado

1. **Nodos para entidades, relaciones para conexiones**
   - Los nodos representan objetos/conceptos
   - Las relaciones expresan cómo se conectan

2. **Propiedades para información**
   - Información simple como propiedades de nodos
   - Información compleja posiblemente como nodos propios

3. **Múltiples etiquetas para categorización**
   - Un nodo puede tener varias etiquetas para distintas categorizaciones

4. **Relaciones direccionales y con propiedades**
   - Las relaciones tienen dirección semántica
   - Las propiedades de relación capturan metadatos de la conexión

#### Ejemplos de modelado

**Red social:**
```cypher
// Personas y amistades
CREATE (alice:Persona {nombre: 'Alice', edad: 28})
CREATE (bob:Persona {nombre: 'Bob', edad: 32})
CREATE (charlie:Persona {nombre: 'Charlie', edad: 25})
CREATE (alice)-[:AMIGO_DE {desde: '2020-01-15'}]->(bob)
CREATE (bob)-[:AMIGO_DE {desde: '2020-01-15'}]->(alice)
CREATE (alice)-[:AMIGO_DE {desde: '2019-08-10'}]->(charlie)

// Publicaciones y comentarios
CREATE (post:Publicación {
  contenido: 'Me encanta Neo4j!',
  fecha: '2023-05-20'
})
CREATE (alice)-[:PUBLICÓ]->(post)
CREATE (comentario:Comentario {
  contenido: 'Totalmente de acuerdo!',
  fecha: '2023-05-20'
})
CREATE (bob)-[:COMENTÓ]->(comentario)
CREATE (comentario)-[:EN]->(post)

// Etiquetas e intereses
CREATE (tech:Interés {nombre: 'Tecnología'})
CREATE (travel:Interés {nombre: 'Viajes'})
CREATE (alice)-[:INTERESADO_EN]->(tech)
CREATE (alice)-[:INTERESADO_EN]->(travel)
CREATE (bob)-[:INTERESADO_EN]->(tech)
CREATE (post)-[:ETIQUETADO_CON]->(tech)
```

**Sistema de recomendación de productos:**
```cypher
// Productos y categorías
CREATE (laptop:Producto {id: 'p1', nombre: 'Laptop Pro', precio: 1299.99})
CREATE (phone:Producto {id: 'p2', nombre: 'Smartphone X', precio: 699.99})
CREATE (tech:Categoría {nombre: 'Electrónica'})
CREATE (laptop)-[:EN_CATEGORÍA]->(tech)
CREATE (phone)-[:EN_CATEGORÍA]->(tech)

// Usuarios y compras
CREATE (user:Usuario {id: 'u1', nombre: 'Juan'})
CREATE (user)-[:COMPRÓ {fecha: '2023-04-15', valoración: 5}]->(laptop)
CREATE (user)-[:VIO {veces: 3}]->(phone)

// Recomendaciones
MATCH (u:Usuario {id: 'u1'})-[:COMPRÓ]->(p:Producto)-[:EN_CATEGORÍA]->(c:Categoría)
MATCH (rec:Producto)-[:EN_CATEGORÍA]->(c)
WHERE NOT (u)-[:COMPRÓ]->(rec)
RETURN rec.nombre, count(*) as relevancia
ORDER BY relevancia DESC
```

### 12B.7.4 Otras bases de datos de grafos

#### Amazon Neptune

Base de datos de grafos gestionada por AWS, compatible con los estándares de la industria:

- Soporta TinkerPop Gremlin y SPARQL
- Altamente disponible y escalable
- Integración con servicios AWS

```java
// Ejemplo con Gremlin
Cluster cluster = Cluster.build()
  .addContactPoint("neptune-endpoint")
  .port(8182)
  .enableSsl(true)
  .create();

GraphTraversalSource g = traversal().withRemote(
  DriverRemoteConnection.using(cluster)
);

// Crear vértices
Vertex alice = g.addV("persona").property("nombre", "Alice").next();
Vertex bob = g.addV("persona").property("nombre", "Bob").next();

// Crear aristas
g.V(alice).addE("conoce").to(bob).property("desde", "2020").iterate();

// Consultas
List<Map<String, Object>> result = g.V()
  .has("nombre", "Alice")
  .out("conoce")
  .valueMap(true)
  .toList();
```

#### ArangoDB

Base de datos multi-modelo que combina documentos, clave-valor y grafos:

- API unificada para todos los modelos de datos
- AQL (ArangoDB Query Language)
- Clústers distribuidos

```javascript
// Ejemplo de AQL
db._query(`
  FOR persona IN Personas
    FILTER persona.ciudad == "Madrid"
    FOR amigo IN OUTBOUND persona Amistades
      COLLECT ciudad = amigo.ciudad
      WITH COUNT INTO total
      RETURN { ciudad, total }
`);
```

## 12B.8 Tendencias y Patrones en NoSQL

### 12B.8.1 Persistencia políglota

La **persistencia políglota** se refiere al uso de múltiples tecnologías de almacenamiento de datos en una misma aplicación, eligiendo la más adecuada para cada tipo de dato y caso de uso.

**Beneficios:**
- Optimización por caso de uso
- Mayor flexibilidad
- Mejor rendimiento

**Ejemplo de arquitectura políglota:**

```
                 +------------------+
                 |    Aplicación    |
                 +------------------+
                         |
           +-------------+-------------+
           |             |             |
+----------v---+ +-------v------+ +----v---------+
| MongoDB      | | Redis        | | Neo4j        |
| (Documentos) | | (Caché, KV)  | | (Relaciones) |
+------------+-+ +-+------------+ +------+-------+
           |       |                    |
           +-------+--------------------+
                   |
           +-------v----------+
           | PostgreSQL       |
           | (Datos críticos) |
           +------------------+
```

**Implementación:**
```javascript
// Ejemplo conceptual en Node.js
const mongoose = require('mongoose');
const redis = require('redis');
const neo4j = require('neo4j-driver');
const { Pool } = require('pg');

// Conexiones a diferentes bases de datos
const mongoClient = mongoose.connect('mongodb://localhost/app');
const redisClient = redis.createClient();
const neo4jDriver = neo4j.driver('bolt://localhost:7687');
const pgPool = new Pool({
  connectionString: 'postgresql://user:pass@localhost/app'
});

async function getUserProfile(userId) {
  // Intentar obtener de caché primero
  const cachedProfile = await redisClient.get(`user:${userId}`);
  if (cachedProfile) return JSON.parse(cachedProfile);
  
  // Si no está en caché, obtener datos básicos de MongoDB
  const userDoc = await User.findById(userId);
  
  // Obtener relaciones sociales de Neo4j
  const neo4jSession = neo4jDriver.session();
  const connections = await neo4jSession.run(`
    MATCH (u:User {id: $userId})-[:KNOWS]->(friend)
    RETURN friend.id, friend.name
    LIMIT 5
  `, { userId });
  
  // Obtener transacciones de PostgreSQL
  const { rows: transactions } = await pgPool.query(
    'SELECT * FROM transactions WHERE user_id = $1 ORDER BY date DESC LIMIT 5',
    [userId]
  );
  
  // Construir perfil completo
  const profile = {
    ...userDoc.toJSON(),
    connections: connections.records.map(r => ({ 
      id: r.get('friend.id'),
      name: r.get('friend.name')
    })),
    recentTransactions: transactions
  };
  
  // Guardar en caché
  await redisClient.set(`user:${userId}`, JSON.stringify(profile), 'EX', 300);
  
  return profile;
}
```

### 12B.8.2 Patrones de consistencia

#### Consistencia eventual

La mayoría de sistemas NoSQL utilizan consistencia eventual, garantizando que los datos serán consistentes en algún momento futuro.

**Implementación con versionado:**
```javascript
// Ejemplo conceptual de objeto con versionado
function updateUserProfile(userId, updates) {
  // Obtener versión actual
  const currentDoc = db.users.findOne({ _id: userId });
  const newVersion = currentDoc.version + 1;
  
  // Actualizar con nueva versión
  db.users.updateOne(
    { _id: userId, version: currentDoc.version },
    { $set: { ...updates, version: newVersion } }
  );
  
  // Publicar evento para sincronizar servicios
  publish('user.updated', {
    userId,
    updates,
    version: newVersion,
    timestamp: new Date()
  });
}
```

#### CQRS (Command Query Responsibility Segregation)

Separar las operaciones de lectura (queries) de las operaciones de escritura (commands).

**Implementación:**
```javascript
// Servicio de escritura
class UserCommandService {
  async createUser(userData) {
    // Validar comando
    validateUserData(userData);
    
    // Generar ID
    const userId = generateUuid();
    
    // Almacenar en BD optimizada para escritura
    await writeDb.users.insertOne({
      _id: userId,
      ...userData,
      createdAt: new Date(),
      version: 1
    });
    
    // Publicar evento
    await eventBus.publish('UserCreated', {
      userId,
      data: userData,
      timestamp: new Date()
    });
    
    return userId;
  }
}

// Servicio de lectura
class UserQueryService {
  async getUserProfile(userId) {
    // Leer de BD optimizada para lectura
    return await readDb.userProfiles.findOne({ userId });
  }
  
  async searchUsers(criteria) {
    return await readDb.userProfiles.find(criteria).limit(20).toArray();
  }
}

// Proyector (sincroniza write -> read)
class UserProfileProjector {
  async handleUserCreated(event) {
    await readDb.userProfiles.insertOne({
      userId: event.userId,
      ...event.data,
      lastUpdated: event.timestamp
    });
  }
  
  async handleUserUpdated(event) {
    await readDb.userProfiles.updateOne(
      { userId: event.userId },
      { 
        $set: { 
          ...event.updates,
          lastUpdated: event.timestamp 
        } 
      }
    );
  }
}
```

### 12B.8.3 Diseño guiado por consultas

En NoSQL, a diferencia de las bases de datos relacionales donde normalizamos primero, debemos modelar nuestros datos basándonos en los patrones de consulta de la aplicación.

**Pasos:**
1. Identificar los casos de uso principales
2. Analizar patrones de consulta
3. Desarrollar modelo de datos optimizado para estos patrones
4. Aceptar desnormalización cuando sea necesario

**Ejemplo práctico:**

```javascript
// Modelado guiado por consulta para un blog en MongoDB

// Consulta frecuente: "Obtener los últimos 10 artículos con autor"
// Modelado: Embeber información del autor en cada artículo

db.articulos.insertOne({
  titulo: "Introducción a NoSQL",
  contenido: "...",
  fecha: new Date("2023-05-20"),
  autor: {
    id: "auth123",
    nombre: "Ana García",
    avatar: "https://ejemplo.com/avatar.jpg"
  },
  tags: ["nosql", "bases de datos", "mongodb"]
});

// Consulta: "Obtener todos los comentarios de un artículo"
// Modelado: Mantener comentarios en colección separada con referencia

db.comentarios.insertMany([
  {
    articulo_id: "art123",
    usuario: {
      id: "user456",
      nombre: "Juan Pérez"
    },
    texto: "¡Gran artículo!",
    fecha: new Date("2023-05-21")
  },
  {
    articulo_id: "art123",
    usuario: {
      id: "user789",
      nombre: "María López"
    },
    texto: "Muy informativo",
    fecha: new Date("2023-05-22")
  }
]);

// Consulta: "Obtener estadísticas por autor"
// Modelado: Colección separada para estadísticas agregadas

db.estadisticas_autor.insertOne({
  autor_id: "auth123",
  total_articulos: 24,
  total_vistas: 153700,
  comentarios_recibidos: 843,
  ultimo_articulo: new Date("2023-05-20"),
  articulos_por_mes: [
    { mes: "2023-05", cantidad: 3 },
    { mes: "2023-04", cantidad: 5 },
    { mes: "2023-03", cantidad: 4 }
  ]
});
```

### 12B.8.4 Tendencias actuales en NoSQL

1. **Bases de datos NewSQL**
   - CockroachDB, Google Spanner
   - Escalabilidad horizontal + ACID
   - Reconciliación de mundos SQL y NoSQL

2. **Serverless Databases**
   - Modelos de pago por uso
   - Escalado automático
   - DynamoDB, Firestore, CosmosDB, FaunaDB

3. **Multi-modelo**
   - Una única base de datos para múltiples modelos (documentos, grafos, clave-valor)
   - ArangoDB, FaunaDB, OrientDB

4. **Edge Computing**
   - Bases de datos para dispositivos edge
   - SQLite, PouchDB, RxDB

5. **Time Series**
   - Optimizadas para datos secuenciales temporales
   - InfluxDB, TimescaleDB

6. **Graph + AI**
   - Bases de datos de grafos con capacidades ML integradas
   - Neo4j Graph Data Science

7. **APIs universales**
   - GraphQL como interfaz unificada
   - Hasura, Prisma, FaunaDB

## 12B.9 Conclusiones sobre NoSQL

Las bases de datos NoSQL han revolucionado el panorama del almacenamiento de datos, ofreciendo alternativas especializadas al modelo relacional tradicional. A modo de resumen:

- **Flexibilidad y especialización**: Cada tipo de base de datos NoSQL está optimizada para casos de uso específicos.

- **Escalabilidad**: Diseñadas para escalar horizontalmente, permitiendo manejar volúmenes masivos de datos.

- **Modelos de consistencia**: Ofrecen opciones que van desde consistencia eventual hasta ACID completo, según necesidades.

- **Evolución continua**: El ecosistema NoSQL sigue evolucionando, con tendencias hacia la convergencia con SQL y nuevos paradigmas.

- **Persistencia políglota**: La tendencia actual es utilizar múltiples bases de datos especializadas para diferentes aspectos de una aplicación.

La elección entre SQL, NoSQL o una combinación de ambas debe basarse en:

1. Estructura de los datos
2. Patrones de consulta previstos
3. Requisitos de escalabilidad
4. Necesidades de consistencia
5. Modelo de desarrollo y habilidades del equipo

En última instancia, la mejor solución suele ser una combinación pragmática de tecnologías, seleccionadas según las necesidades específicas de cada componente de la aplicación.
