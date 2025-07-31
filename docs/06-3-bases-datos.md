# 6.3 Bases de Datos y Persistencia

La gestión de datos es una parte fundamental del desarrollo backend. Este capítulo explora las diferentes opciones de bases de datos disponibles para aplicaciones modernas, patrones de acceso a datos y consideraciones de diseño.

## 6.3.1 Panorama de Bases de Datos Modernas

El ecosistema de bases de datos ha evolucionado más allá del tradicional modelo relacional, ofreciendo soluciones adaptadas a diferentes casos de uso.

### Clasificación Principal

1. **Bases de Datos Relacionales (SQL)**
   - PostgreSQL, MySQL, SQLite, SQL Server, Oracle

2. **Bases de Datos NoSQL**
   - **Documentales**: MongoDB, Firestore, CouchDB
   - **Clave-Valor**: Redis, DynamoDB, etcd
   - **Columnares**: Cassandra, HBase, ClickHouse
   - **Grafos**: Neo4j, ArangoDB, Amazon Neptune

3. **Bases de Datos NewSQL**
   - CockroachDB, Google Spanner, TiDB
   
4. **Bases de Datos de Búsqueda**
   - Elasticsearch, Solr, MeiliSearch

### Comparativa de Modelos

| Tipo | Puntos Fuertes | Casos de Uso | Limitaciones |
|------|----------------|--------------|-------------|
| Relacional | Integridad referencial, transacciones ACID, consultas complejas | Aplicaciones financieras, ERP, CRM, sistemas con relaciones complejas | Escalabilidad horizontal más compleja |
| Documental | Esquema flexible, desarrollo rápido, fácil escalabilidad | Contenido, CMS, aplicaciones web, catálogos | Consultas relacionales menos eficientes, consistencia eventual |
| Clave-Valor | Rendimiento extremo, simplicidad | Caché, colas, configuración, sesiones | Consultas limitadas, sin esquema |
| Columnar | Análisis eficiente, alto rendimiento en escritura | Big Data, análisis en tiempo real, telemetría | Complejidad de implementación |
| Grafo | Relaciones complejas, consultas de recorrido | Redes sociales, recomendaciones, detección de fraude | Curva de aprendizaje, menos soporte ecosistémico |

## 6.3.2 Bases de Datos SQL: Diseño y Buenas Prácticas

### Normalización

La normalización es el proceso de organizar los datos para reducir la redundancia y mejorar la integridad.

**Formas normales básicas**:
- **1NF**: Valores atómicos (no múltiples valores en una celda)
- **2NF**: 1NF + sin dependencias parciales
- **3NF**: 2NF + sin dependencias transitivas

**Ejemplo de normalización**:

**No normalizado**:
```
PEDIDOS(id_pedido, fecha, id_cliente, nombre_cliente, email_cliente, productos, precios)
```

**Normalizado (3NF)**:
```
CLIENTES(id_cliente, nombre_cliente, email_cliente)
PEDIDOS(id_pedido, fecha, id_cliente)
DETALLES_PEDIDO(id_pedido, id_producto, cantidad, precio)
PRODUCTOS(id_producto, nombre_producto, precio_unitario)
```

### Relaciones y Claves

**Tipos de relaciones**:
- **Uno a uno (1:1)**: Un registro en la tabla A corresponde exactamente a uno en la tabla B.
- **Uno a muchos (1:N)**: Un registro en la tabla A corresponde a varios en la tabla B.
- **Muchos a muchos (N:M)**: Requiere tabla de unión (join table).

**Ejemplo de relación muchos a muchos**:

```sql
-- Tabla de estudiantes
CREATE TABLE estudiantes (
    id_estudiante INT PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

-- Tabla de cursos
CREATE TABLE cursos (
    id_curso INT PRIMARY KEY,
    nombre VARCHAR(100),
    descripcion TEXT
);

-- Tabla de unión (inscripciones)
CREATE TABLE inscripciones (
    id_estudiante INT,
    id_curso INT,
    fecha_inscripcion DATE,
    calificacion DECIMAL(3,1),
    PRIMARY KEY (id_estudiante, id_curso),
    FOREIGN KEY (id_estudiante) REFERENCES estudiantes(id_estudiante),
    FOREIGN KEY (id_curso) REFERENCES cursos(id_curso)
);
```

### Optimización de Consultas

**Índices**:
Los índices mejoran dramáticamente el rendimiento de las consultas pero tienen un costo en escrituras y almacenamiento.

**Tipos de índices comunes**:
- **Índice B-Tree**: Índice general para operaciones `=`, `<`, `>`, BETWEEN, etc.
- **Índice Hash**: Excelente para operaciones de igualdad exacta.
- **Índice de Texto Completo**: Para búsqueda en campos de texto.
- **Índice Espacial**: Para datos geoespaciales.
- **Índice Multicampo**: Para consultas que filtran por múltiples columnas.

```sql
-- Ejemplo de creación de índices
CREATE INDEX idx_usuarios_apellido ON usuarios(apellido);
CREATE UNIQUE INDEX idx_productos_codigo ON productos(codigo);
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos(id_cliente, fecha_pedido);
```

**EXPLAIN**: La mayoría de bases de datos SQL ofrecen un comando EXPLAIN para analizar el plan de ejecución de consultas.

```sql
EXPLAIN SELECT * FROM usuarios 
WHERE apellido = 'García' AND fecha_registro > '2023-01-01';
```

## 6.3.3 MongoDB y NoSQL Documental

MongoDB es una de las bases de datos NoSQL documentales más populares, que almacena datos en documentos similares a JSON (BSON).

### Estructura de Datos

```javascript
// Ejemplo de un documento en MongoDB (colección 'usuarios')
{
  _id: ObjectId("60d21b4667d0d8992e610c85"),
  nombre: "Ana López",
  email: "ana@ejemplo.com",
  rol: "admin",
  preferencias: {
    notificaciones: true,
    tema: "oscuro"
  },
  intereses: ["tecnología", "viajes", "fotografía"],
  fecha_registro: ISODate("2023-01-15T14:12:00Z"),
  ultima_sesion: ISODate("2023-06-20T08:45:12Z")
}
```

### Consultas Básicas

```javascript
// Encontrar un usuario por email
db.usuarios.findOne({ email: "ana@ejemplo.com" });

// Filtrar usuarios por rol con proyección (seleccionar campos)
db.usuarios.find(
  { rol: "admin" }, 
  { nombre: 1, email: 1 }
);

// Consulta con condiciones múltiples
db.productos.find({
  precio: { $gt: 100, $lt: 500 },
  categorias: "electrónica",
  disponible: true
});

// Actualizar documento
db.usuarios.updateOne(
  { email: "ana@ejemplo.com" },
  { $set: { "preferencias.tema": "claro" } }
);

// Agregaciones
db.pedidos.aggregate([
  { $match: { estado: "completado" } },
  { $group: { 
      _id: "$id_cliente", 
      total_gastado: { $sum: "$total" },
      cantidad_pedidos: { $sum: 1 }
    }
  },
  { $sort: { total_gastado: -1 } },
  { $limit: 5 }
]);
```

### Modelado de Datos en MongoDB

**Enfoque incrustado (embedding)**:
Almacenar datos relacionados en un solo documento.

```javascript
// Documento con datos incrustados
{
  _id: ObjectId("..."),
  titulo: "MongoDB Basics",
  autor: {
    nombre: "Carlos Ruiz",
    email: "carlos@ejemplo.com",
    bio: "Desarrollador y escritor técnico..."
  },
  comentarios: [
    {
      usuario: "maria",
      texto: "Gran artículo!",
      fecha: ISODate("2023-05-15...")
    },
    {
      usuario: "pedro",
      texto: "Muy útil, gracias!",
      fecha: ISODate("2023-05-16...")
    }
  ]
}
```

**Enfoque referencial**:
Similar a las claves foráneas en SQL, almacenando referencias entre documentos.

```javascript
// Colección 'libros'
{
  _id: ObjectId("libro1"),
  titulo: "MongoDB Basics",
  autor_id: ObjectId("autor1"),
  categorias: ["database", "programming"]
}

// Colección 'autores'
{
  _id: ObjectId("autor1"),
  nombre: "Carlos Ruiz",
  email: "carlos@ejemplo.com",
  bio: "Desarrollador y escritor técnico..."
}

// Colección 'comentarios'
{
  _id: ObjectId("comentario1"),
  libro_id: ObjectId("libro1"),
  usuario: "maria",
  texto: "Gran artículo!",
  fecha: ISODate("2023-05-15...")
}
```

### Consideraciones de Diseño

- **Embedding vs. Referencias**: Incrustación para datos que se acceden juntos frecuentemente; referencias para relaciones complejas o datos muy grandes.
- **Evitar documentos muy grandes**: Límite de 16MB por documento.
- **Evitar arrays muy grandes**: Considerar separar en colecciones separadas si pueden crecer indefinidamente.
- **Desnormalizar con cuidado**: La redundancia puede mejorar el rendimiento de lectura pero complica las actualizaciones.

## 6.3.4 Redis: Clave-Valor y Estructuras de Datos

Redis es una base de datos en memoria que funciona como almacén de clave-valor pero soporta estructuras de datos complejas.

### Estructuras de Datos Principales

**Strings**:
```
SET user:1:name "Alice Smith"
GET user:1:name
```

**Hashes** (maps):
```
HSET user:1 name "Alice" email "alice@example.com" age 28
HGET user:1 name
HGETALL user:1
```

**Lists**:
```
LPUSH notifications:user:1 "New message"
LPUSH notifications:user:1 "Friend request"
LRANGE notifications:user:1 0 -1  # obtener todos los elementos
```

**Sets**:
```
SADD interests:user:1 "programming" "music" "hiking"
SMEMBERS interests:user:1
```

**Sorted Sets**:
```
ZADD leaderboard 1000 "player1"
ZADD leaderboard 2500 "player2"
ZADD leaderboard 1800 "player3"
ZREVRANGE leaderboard 0 2 WITHSCORES  # top 3 players
```

### Casos de Uso Comunes

1. **Caché**:
```
SET product:10 "{\"id\":10,\"name\":\"Laptop\",\"price\":999}" EX 3600  # expira en 1 hora
```

2. **Contadores**:
```
INCR page:views
GET page:views
```

3. **Limitador de tasa (Rate Limiting)**:
```
INCR rate:ip:192.168.1.1:minute
EXPIRE rate:ip:192.168.1.1:minute 60
```

4. **Colas de mensajes**:
```
# Productor
LPUSH task_queue '{"id": 1, "task": "process_image"}'

# Consumidor
BRPOP task_queue 30  # esperar hasta 30 segundos por una tarea
```

5. **Sesiones**:
```
HMSET session:abc123 user_id 42 login_time "2023-06-10T15:30:00Z" data "{\"cart\":[1,2,3]}"
EXPIRE session:abc123 1800  # expira en 30 minutos
```

## 6.3.5 ORMs y Capas de Acceso a Datos

Los Object-Relational Mappers (ORMs) facilitan la interacción con bases de datos relacionando objetos en el código con tablas en la base de datos.

### Ventajas y Desventajas

**Ventajas**:
- Abstracción del SQL puro
- Mapeo automático entre objetos y tablas
- Mayor productividad del desarrollador
- Seguridad contra inyección SQL

**Desventajas**:
- Posible sobrecarga de rendimiento
- Menor control sobre consultas específicas
- Curva de aprendizaje adicional

### Ejemplos de ORMs Populares

**Sequelize (Node.js/JavaScript)**:

```javascript
// Definir modelo
const User = sequelize.define('User', {
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
  },
  email: {
    type: DataTypes.STRING,
    unique: true,
    validate: {
      isEmail: true
    }
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false
  },
  role: {
    type: DataTypes.ENUM('user', 'admin'),
    defaultValue: 'user'
  }
});

// Consultas
async function findUserById(id) {
  return await User.findByPk(id);
}

async function findAdminUsers() {
  return await User.findAll({
    where: {
      role: 'admin'
    }
  });
}

async function createUser(userData) {
  return await User.create(userData);
}
```

**TypeORM (TypeScript)**:

```typescript
// Definir entidad
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 100 })
  firstName: string;

  @Column({ length: 100 })
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column({
    type: "enum",
    enum: ["user", "admin"],
    default: "user"
  })
  role: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

// Repositorio y consultas
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  async findOne(id: number): Promise<User> {
    return this.userRepository.findOne({
      where: { id },
      relations: ['posts']
    });
  }

  async findByEmail(email: string): Promise<User> {
    return this.userRepository.findOne({ where: { email } });
  }

  async create(userData: Partial<User>): Promise<User> {
    const user = this.userRepository.create(userData);
    return await this.userRepository.save(user);
  }
}
```

**Mongoose (MongoDB con Node.js)**:

```javascript
// Definir esquema
const userSchema = new mongoose.Schema({
  firstName: { 
    type: String, 
    required: true 
  },
  lastName: String,
  email: {
    type: String,
    required: true,
    unique: true,
    match: /^\S+@\S+\.\S+$/
  },
  password: {
    type: String,
    required: true
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Método del esquema
userSchema.methods.fullName = function() {
  return `${this.firstName} ${this.lastName}`;
};

// Método estático
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email });
};

// Middleware pre-save (hook)
userSchema.pre('save', async function(next) {
  // Hash password si ha cambiado
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

// Crear modelo
const User = mongoose.model('User', userSchema);

// Uso
async function createUser(userData) {
  const user = new User(userData);
  return await user.save();
}

async function findUserByEmail(email) {
  return await User.findByEmail(email);
}
```

## 6.3.6 Patrones de Acceso a Datos

### Patrón Repositorio

Encapsula la lógica para acceder a datos, proporcionando una interfaz orientada a objetos.

```typescript
// Definición de la interfaz del repositorio
interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(options?: QueryOptions): Promise<User[]>;
  create(userData: Partial<User>): Promise<User>;
  update(id: string, userData: Partial<User>): Promise<User | null>;
  delete(id: string): Promise<boolean>;
}

// Implementación para MongoDB
class MongoUserRepository implements UserRepository {
  constructor(private readonly db: Db) {}

  async findById(id: string): Promise<User | null> {
    const user = await this.db.collection('users').findOne({ _id: new ObjectId(id) });
    return user ? this.mapToUser(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.db.collection('users').findOne({ email });
    return user ? this.mapToUser(user) : null;
  }

  async findAll(options?: QueryOptions): Promise<User[]> {
    const query = options?.filter || {};
    const limit = options?.limit || 100;
    const skip = options?.skip || 0;
    
    const users = await this.db.collection('users')
      .find(query)
      .limit(limit)
      .skip(skip)
      .toArray();
    
    return users.map(user => this.mapToUser(user));
  }

  async create(userData: Partial<User>): Promise<User> {
    const result = await this.db.collection('users').insertOne({
      ...userData,
      createdAt: new Date()
    });
    
    return this.findById(result.insertedId.toString()) as Promise<User>;
  }

  // Métodos update y delete omitidos por brevedad

  private mapToUser(doc: any): User {
    return {
      id: doc._id.toString(),
      firstName: doc.firstName,
      lastName: doc.lastName,
      email: doc.email,
      role: doc.role,
      createdAt: doc.createdAt
    };
  }
}
```

### Patrón Unit of Work

Mantiene una lista de objetos afectados por una transacción de negocio y coordina los cambios.

```typescript
class UnitOfWork {
  private readonly userRepository: UserRepository;
  private readonly postRepository: PostRepository;
  private readonly commentRepository: CommentRepository;
  
  constructor(private readonly db: Db) {
    this.userRepository = new MongoUserRepository(db);
    this.postRepository = new MongoPostRepository(db);
    this.commentRepository = new MongoCommentRepository(db);
  }
  
  getUserRepository(): UserRepository {
    return this.userRepository;
  }
  
  getPostRepository(): PostRepository {
    return this.postRepository;
  }
  
  getCommentRepository(): CommentRepository {
    return this.commentRepository;
  }
  
  async beginTransaction() {
    this.session = this.db.client.startSession();
    this.session.startTransaction();
  }
  
  async commitTransaction() {
    if (this.session) {
      await this.session.commitTransaction();
      await this.session.endSession();
    }
  }
  
  async rollbackTransaction() {
    if (this.session) {
      await this.session.abortTransaction();
      await this.session.endSession();
    }
  }
}

// Uso del patrón
async function createPostWithComments(postData, comments) {
  const unitOfWork = new UnitOfWork(db);
  
  try {
    await unitOfWork.beginTransaction();
    
    // Usar repositorios
    const post = await unitOfWork.getPostRepository().create(postData);
    
    for (const commentData of comments) {
      commentData.postId = post.id;
      await unitOfWork.getCommentRepository().create(commentData);
    }
    
    await unitOfWork.commitTransaction();
    return post;
  } catch (error) {
    await unitOfWork.rollbackTransaction();
    throw error;
  }
}
```

## 6.3.7 Bases de Datos Políglota y Selección de Base de Datos

En arquitecturas modernas, es común usar diferentes bases de datos para diferentes componentes de la aplicación, aprovechando las fortalezas de cada una.

### Criterios de Selección

Al elegir una base de datos, considera:

1. **Estructura de los datos**: Relacionales, documentos, grafos, series temporales
2. **Patrón de acceso**: Lecturas vs. escrituras, consultas complejas vs. simples
3. **Escalabilidad requerida**: Vertical vs. horizontal
4. **Consistencia vs. disponibilidad**: Según teorema CAP
5. **Rendimiento requerido**: Latencia, throughput
6. **Costos y recursos**: Licencias, hardware, conocimiento del equipo
7. **Ecosistema y soporte**: Herramientas, bibliotecas, comunidad

### Ejemplo de Arquitectura Políglota

Para una aplicación de comercio electrónico:

1. **PostgreSQL**: Datos transaccionales (pedidos, pagos)
2. **MongoDB**: Catálogo de productos con atributos variables
3. **Redis**: Carrito de compras, sesiones, caché
4. **Elasticsearch**: Búsqueda de productos
5. **Neo4j**: Sistema de recomendaciones basado en relaciones entre productos y usuarios

### Ejemplo de Implementación

```typescript
// Service que usa múltiples bases de datos
class ProductService {
  constructor(
    private readonly postgresClient: Pool,
    private readonly mongoClient: MongoClient,
    private readonly redisClient: Redis,
    private readonly elasticClient: Client
  ) {}
  
  async getProduct(id: string) {
    // Intentar primero en caché
    const cachedProduct = await this.redisClient.get(`product:${id}`);
    if (cachedProduct) {
      return JSON.parse(cachedProduct);
    }
    
    // Buscar en la base de datos principal
    const product = await this.mongoClient
      .db('store')
      .collection('products')
      .findOne({ _id: new ObjectId(id) });
    
    if (product) {
      // Guardar en caché para futuros accesos
      await this.redisClient.set(
        `product:${id}`,
        JSON.stringify(product),
        'EX', 3600 // expirar en 1 hora
      );
    }
    
    return product;
  }
  
  async searchProducts(query: string) {
    // Usar Elasticsearch para búsqueda eficiente
    const result = await this.elasticClient.search({
      index: 'products',
      body: {
        query: {
          multi_match: {
            query,
            fields: ['name', 'description', 'categories']
          }
        }
      }
    });
    
    return result.hits.hits.map(hit => hit._source);
  }
  
  async createOrder(order) {
    // Usar PostgreSQL para datos transaccionales
    const client = await this.postgresClient.connect();
    
    try {
      await client.query('BEGIN');
      
      // Insertar en múltiples tablas con integridad referencial
      const orderResult = await client.query(
        'INSERT INTO orders(user_id, total, status) VALUES($1, $2, $3) RETURNING id',
        [order.userId, order.total, 'pending']
      );
      
      const orderId = orderResult.rows[0].id;
      
      for (const item of order.items) {
        await client.query(
          'INSERT INTO order_items(order_id, product_id, quantity, price) VALUES($1, $2, $3, $4)',
          [orderId, item.productId, item.quantity, item.price]
        );
      }
      
      await client.query('COMMIT');
      
      // Eliminar el carrito de compras de Redis
      await this.redisClient.del(`cart:${order.userId}`);
      
      return { orderId };
    } catch (e) {
      await client.query('ROLLBACK');
      throw e;
    } finally {
      client.release();
    }
  }
}
```
