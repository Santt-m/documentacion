# Capítulo 7: Desarrollo Backend

## 7.1 Fundamentos de Desarrollo Backend

El desarrollo backend se refiere a la creación de la lógica del servidor, las bases de datos y las APIs que permiten el funcionamiento de las aplicaciones web modernas. Es la parte "oculta" para el usuario final pero fundamental para el funcionamiento de cualquier aplicación.

### 7.1.1 Arquitectura Cliente-Servidor

La arquitectura cliente-servidor es el modelo fundamental en el que se basan las aplicaciones web:

- **Cliente**: La aplicación frontend que se ejecuta en el navegador del usuario.
- **Servidor**: El software que se ejecuta en un servidor remoto y proporciona servicios al cliente.
- **Comunicación**: Generalmente se realiza mediante el protocolo HTTP/HTTPS.

```
+------------+         Solicitudes HTTP         +------------+
|            | <-------------------------->     |            |
|  Cliente   |                                  |  Servidor  |
|            |          Respuestas HTTP         |            |
+------------+ <-------------------------->     +------------+
   (Browser)                                    (API/Backend)
```

### 7.1.2 Protocolos de Comunicación

#### HTTP/HTTPS

HTTP (Hypertext Transfer Protocol) es el protocolo estándar para la comunicación web.

```
GET /api/users HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Códigos de estado HTTP comunes:

- **2xx**: Éxito (200 OK, 201 Created)
- **3xx**: Redirección (301 Moved Permanently, 304 Not Modified)
- **4xx**: Error del cliente (400 Bad Request, 401 Unauthorized, 404 Not Found)
- **5xx**: Error del servidor (500 Internal Server Error)

#### REST (Representational State Transfer)

REST es un estilo arquitectónico que define un conjunto de restricciones para crear servicios web:

- **Recursos identificados mediante URIs**: `/users`, `/products/42`
- **Operaciones mediante métodos HTTP**:
  - GET: Recuperar datos
  - POST: Crear nuevos datos
  - PUT/PATCH: Actualizar datos
  - DELETE: Eliminar datos
- **Representación de recursos**: JSON, XML, HTML, etc.
- **Sin estado**: Cada solicitud contiene toda la información necesaria

Ejemplo de API RESTful:

```
# Obtener todos los usuarios
GET /api/users

# Obtener un usuario específico
GET /api/users/123

# Crear un nuevo usuario
POST /api/users
Content-Type: application/json
{ "name": "Juan", "email": "juan@example.com" }

# Actualizar un usuario
PUT /api/users/123
Content-Type: application/json
{ "name": "Juan Pérez", "email": "juan@example.com" }

# Eliminar un usuario
DELETE /api/users/123
```

#### GraphQL

GraphQL es una alternativa a REST que permite a los clientes solicitar exactamente los datos que necesitan.

```graphql
# Consulta GraphQL
query {
  user(id: "123") {
    name
    email
    posts {
      title
      createdAt
    }
  }
}

# Respuesta JSON
{
  "data": {
    "user": {
      "name": "Juan Pérez",
      "email": "juan@example.com",
      "posts": [
        {
          "title": "Introducción a GraphQL",
          "createdAt": "2023-05-15"
        },
        {
          "title": "REST vs GraphQL",
          "createdAt": "2023-06-10"
        }
      ]
    }
  }
}
```

### 7.1.3 Node.js y Express

Node.js es un entorno de ejecución para JavaScript en el servidor, y Express es el framework más popular para crear APIs.

```javascript
// Servidor básico con Express
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Middleware para parsear JSON
app.use(express.json());

// Ruta simple
app.get('/', (req, res) => {
  res.send('¡Hola mundo!');
});

// Ruta para obtener usuarios
app.get('/api/users', (req, res) => {
  // En un caso real, aquí consultaríamos la base de datos
  const users = [
    { id: 1, name: 'Ana' },
    { id: 2, name: 'Carlos' }
  ];
  res.json(users);
});

// Ruta para crear un usuario
app.post('/api/users', (req, res) => {
  const { name } = req.body;
  
  if (!name) {
    return res.status(400).json({ error: 'El nombre es obligatorio' });
  }
  
  // En un caso real, aquí guardaríamos en la base de datos
  const newUser = {
    id: 3,
    name
  };
  
  res.status(201).json(newUser);
});

// Iniciar el servidor
app.listen(PORT, () => {
  console.log(`Servidor ejecutándose en http://localhost:${PORT}`);
});
```

### 7.1.4 Middleware

Los middleware son funciones que tienen acceso al objeto de solicitud (req), al objeto de respuesta (res) y a la función siguiente en el ciclo solicitud-respuesta.

```javascript
// Middleware de registro
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
  next(); // Continúa al siguiente middleware
});

// Middleware de autenticación
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization;
  
  if (!token || !token.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Token no proporcionado' });
  }
  
  try {
    // En un caso real, aquí verificaríamos el JWT
    const decoded = jwt.verify(
      token.split(' ')[1], 
      process.env.JWT_SECRET
    );
    req.user = decoded; // Guarda el usuario en la solicitud
    next(); // Continúa al siguiente middleware
  } catch (error) {
    return res.status(401).json({ error: 'Token inválido' });
  }
};

// Aplicar middleware a rutas específicas
app.get('/api/profile', authMiddleware, (req, res) => {
  res.json({ message: 'Perfil protegido', user: req.user });
});
```

## 7.2 Bases de Datos y ORM

### 7.2.1 Tipos de Bases de Datos

#### Bases de Datos Relacionales (SQL)

Organizan datos en tablas relacionadas mediante claves primarias y foráneas.

```sql
-- Ejemplo de SQL para crear tablas
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  content TEXT,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Consulta con JOIN
SELECT posts.title, users.name
FROM posts
JOIN users ON posts.user_id = users.id
WHERE users.name = 'Juan';
```

Principales sistemas: PostgreSQL, MySQL, SQL Server, Oracle.

#### Bases de Datos NoSQL

Almacenan datos en formatos no relacionales, como documentos, clave-valor, columnas o grafos.

```javascript
// Ejemplo de documento en MongoDB
{
  "_id": ObjectId("5f8a3d25b2e90f4a1c1e9d5a"),
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "posts": [
    {
      "title": "Introducción a MongoDB",
      "content": "MongoDB es una base de datos documental...",
      "created_at": ISODate("2023-05-15T10:30:00Z")
    },
    {
      "title": "NoSQL vs SQL",
      "content": "Existen diferentes enfoques para el almacenamiento...",
      "created_at": ISODate("2023-06-10T14:15:00Z")
    }
  ],
  "created_at": ISODate("2023-01-15T08:30:00Z")
}
```

Principales sistemas: MongoDB, Redis, Cassandra, DynamoDB.

### 7.2.2 ORM (Object-Relational Mapping)

Los ORM permiten trabajar con bases de datos utilizando objetos en el código, abstrayendo el SQL.

#### Sequelize (para bases de datos SQL con Node.js)

```javascript
// Configuración de Sequelize
const { Sequelize, DataTypes } = require('sequelize');

// Conexión a la base de datos
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres', // o mysql, sqlite, mariadb, mssql
  logging: false
});

// Definición del modelo Usuario
const User = sequelize.define('User', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true
    }
  }
});

// Definición del modelo Post
const Post = sequelize.define('Post', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  title: {
    type: DataTypes.STRING,
    allowNull: false
  },
  content: {
    type: DataTypes.TEXT
  }
});

// Definición de relaciones
User.hasMany(Post);
Post.belongsTo(User);

// Uso de los modelos
async function main() {
  // Sincroniza modelos con la base de datos
  await sequelize.sync();
  
  // Crear un usuario
  const user = await User.create({
    name: 'Ana',
    email: 'ana@example.com'
  });
  
  // Crear un post asociado al usuario
  const post = await Post.create({
    title: 'Mi primer post',
    content: 'Contenido del post',
    UserId: user.id
  });
  
  // Consultar con relaciones
  const userWithPosts = await User.findOne({
    where: { id: user.id },
    include: Post
  });
  
  console.log(userWithPosts.toJSON());
}

main().catch(console.error);
```

#### Mongoose (para MongoDB con Node.js)

```javascript
const mongoose = require('mongoose');

// Conexión a MongoDB
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Definición del esquema de Usuario
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    match: /^\S+@\S+\.\S+$/
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Definición del esquema de Post
const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true
  },
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Crear modelos a partir de los esquemas
const User = mongoose.model('User', userSchema);
const Post = mongoose.model('Post', postSchema);

// Uso de los modelos
async function main() {
  // Crear un usuario
  const user = await User.create({
    name: 'Carlos',
    email: 'carlos@example.com'
  });
  
  // Crear un post asociado al usuario
  const post = await Post.create({
    title: 'MongoDB y Mongoose',
    content: 'Mongoose es un ODM para MongoDB...',
    author: user._id
  });
  
  // Consultar con populate (similar a JOIN)
  const populatedPost = await Post.findById(post._id)
    .populate('author');
  
  console.log(populatedPost);
}

main().catch(console.error);
```

### 7.2.3 Migraciones y Gestión de Esquemas

Las migraciones permiten versionar y gestionar los cambios en la estructura de la base de datos.

```javascript
// Migración con Sequelize CLI (archivo: migrations/XXXXXX-create-users.js)
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true
      },
      password: {
        type: Sequelize.STRING,
        allowNull: false
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Users');
  }
};
```

## 7.3 Autenticación y Autorización

### 7.3.1 Autenticación con JWT

JSON Web Tokens (JWT) es un estándar para crear tokens de acceso que permiten la autenticación entre dos partes.

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const app = express();

app.use(express.json());

// Base de datos simulada
const users = [];

// Ruta para registro
app.post('/api/register', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    // Verificar si el usuario ya existe
    if (users.find(u => u.username === username)) {
      return res.status(400).json({ message: 'El usuario ya existe' });
    }
    
    // Encriptar contraseña
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Guardar usuario
    const user = {
      id: users.length + 1,
      username,
      password: hashedPassword
    };
    
    users.push(user);
    
    res.status(201).json({ message: 'Usuario creado exitosamente' });
  } catch (error) {
    res.status(500).json({ message: 'Error en el servidor' });
  }
});

// Ruta para login
app.post('/api/login', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    // Buscar usuario
    const user = users.find(u => u.username === username);
    if (!user) {
      return res.status(401).json({ message: 'Credenciales inválidas' });
    }
    
    // Verificar contraseña
    const validPassword = await bcrypt.compare(password, user.password);
    if (!validPassword) {
      return res.status(401).json({ message: 'Credenciales inválidas' });
    }
    
    // Generar token JWT
    const token = jwt.sign(
      { id: user.id, username: user.username },
      'your_jwt_secret',  // En producción, usar una variable de entorno
      { expiresIn: '2h' }
    );
    
    res.json({ token });
  } catch (error) {
    res.status(500).json({ message: 'Error en el servidor' });
  }
});

// Middleware de autenticación
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (token == null) return res.status(401).json({ message: 'Token no proporcionado' });
  
  jwt.verify(token, 'your_jwt_secret', (err, user) => {
    if (err) return res.status(403).json({ message: 'Token inválido' });
    
    req.user = user;
    next();
  });
}

// Ruta protegida
app.get('/api/profile', authenticateToken, (req, res) => {
  res.json({ message: 'Ruta protegida', user: req.user });
});

app.listen(3000, () => {
  console.log('Servidor iniciado en puerto 3000');
});
```

### 7.3.2 OAuth 2.0 y Autenticación con Proveedores Externos

OAuth 2.0 permite a las aplicaciones acceder a recursos en nombre del usuario sin compartir credenciales.

```javascript
const express = require('express');
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const session = require('express-session');

const app = express();

// Configuración de sesión
app.use(session({
  secret: 'your_session_secret',
  resave: false,
  saveUninitialized: false
}));

app.use(passport.initialize());
app.use(passport.session());

// Configuración de Passport con Google OAuth
passport.use(new GoogleStrategy({
    clientID: 'YOUR_GOOGLE_CLIENT_ID',
    clientSecret: 'YOUR_GOOGLE_CLIENT_SECRET',
    callbackURL: 'http://localhost:3000/auth/google/callback'
  },
  function(accessToken, refreshToken, profile, done) {
    // En un caso real, buscarías o crearías el usuario en la base de datos
    // Aquí simplemente devolvemos el perfil
    return done(null, profile);
  }
));

// Serialización y deserialización del usuario
passport.serializeUser((user, done) => {
  done(null, user);
});

passport.deserializeUser((user, done) => {
  done(null, user);
});

// Middleware de autenticación
function isAuthenticated(req, res, next) {
  if (req.isAuthenticated()) {
    return next();
  }
  res.redirect('/login');
}

// Rutas de autenticación
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] }));

app.get('/auth/google/callback', 
  passport.authenticate('google', { failureRedirect: '/login' }),
  function(req, res) {
    // Autenticación exitosa
    res.redirect('/dashboard');
  });

app.get('/login', (req, res) => {
  res.send('Página de login <a href="/auth/google">Iniciar sesión con Google</a>');
});

app.get('/dashboard', isAuthenticated, (req, res) => {
  res.send(`Bienvenido, ${req.user.displayName}! <a href="/logout">Cerrar sesión</a>`);
});

app.get('/logout', (req, res) => {
  req.logout(function(err) {
    if (err) { return next(err); }
    res.redirect('/');
  });
});

app.get('/', (req, res) => {
  res.send('Página principal');
});

app.listen(3000, () => {
  console.log('Servidor iniciado en puerto 3000');
});
```
