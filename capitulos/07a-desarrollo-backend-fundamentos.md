# Capítulo 7: Desarrollo Backend

## 7.1 Fundamentos de Desarrollo Backend

### Introducción

El desarrollo backend constituye el núcleo invisible pero esencial de cualquier aplicación web moderna. Mientras que el frontend se encarga de la interfaz con la que interactúan los usuarios, el backend es el responsable de toda la lógica de negocio, procesamiento de datos, autenticación, autorización y comunicación con bases de datos y otros servicios externos.

En este capítulo, exploraremos los componentes fundamentales que conforman la columna vertebral de las aplicaciones web: desde la arquitectura cliente-servidor, los protocolos de comunicación, las tecnologías más utilizadas como Node.js y Express, hasta los sistemas de gestión de bases de datos y mecanismos de autenticación.

El backend es similar al motor y el sistema eléctrico de un automóvil: no es visible para los pasajeros, pero sin él, el vehículo no funcionaría. Su correcto diseño e implementación garantizan la escalabilidad, rendimiento y seguridad de las aplicaciones, aspectos críticos en el entorno digital actual.

### 7.1.1 Arquitectura Cliente-Servidor

La arquitectura cliente-servidor es el modelo fundamental en el que se basan las aplicaciones web modernas. Este paradigma define la forma en que se distribuyen las responsabilidades entre los diferentes componentes de un sistema.

#### Componentes principales

- **Cliente**: Es la aplicación frontend que se ejecuta en el dispositivo del usuario (navegador web, aplicación móvil, etc.). Se encarga de la interfaz de usuario, captura de datos y presentación de información. Ejemplos incluyen aplicaciones React, Angular o aplicaciones móviles nativas.

- **Servidor**: Es el software que se ejecuta en un entorno remoto y centralizado, responsable de procesar solicitudes, ejecutar lógica de negocio compleja, gestionar bases de datos y proporcionar datos y servicios al cliente. Ejemplos incluyen servidores Node.js, Django, Spring Boot, etc.

- **Comunicación**: El intercambio de información entre cliente y servidor se realiza principalmente mediante el protocolo HTTP/HTTPS, aunque existen otros protocolos como WebSockets para comunicación bidireccional en tiempo real.

```
+----------------+                              +----------------+
|                |     1. Solicitud HTTP       |                |
|    CLIENTE     | ---------------------------> |    SERVIDOR    |
| (React, Vue.js,|                              | (Node.js, Java,|
|  Angular, etc.)|     2. Respuesta HTTP       |  Python, etc.) |
|                | <--------------------------- |                |
+----------------+                              +----------------+
                                                       |
                                                       | 3. Interacción
                                                       v
                                               +----------------+
                                               |                |
                                               |   BASE DE      |
                                               |    DATOS       |
                                               |                |
                                               +----------------+
```

#### Ventajas de la arquitectura cliente-servidor

- **Separación de responsabilidades**: Permite que equipos distintos trabajen en el frontend y backend de forma independiente.
- **Escalabilidad**: Los servidores pueden escalarse horizontalmente para manejar más solicitudes sin afectar a los clientes.
- **Seguridad**: La lógica crítica y los datos sensibles pueden mantenerse en el servidor, lejos del entorno del cliente.
- **Centralización**: Facilita actualizaciones y mantenimiento centralizados del backend sin necesidad de actualizar los clientes.

#### Desafíos comunes

- **Latencia**: La necesidad de comunicación constante entre cliente y servidor puede introducir retrasos.
- **Disponibilidad**: Si el servidor falla, todo el sistema puede dejar de funcionar.
- **Complejidad**: Requiere gestionar protocolos de comunicación, seguridad y estado entre componentes distribuidos.

### 7.1.2 Protocolos de Comunicación

#### HTTP/HTTPS

HTTP (Hypertext Transfer Protocol) es el protocolo estándar para la comunicación web, basado en un modelo de solicitud-respuesta entre un cliente y un servidor. HTTPS es su versión segura, que implementa encriptación SSL/TLS para proteger la información intercambiada.

##### Estructura de una solicitud HTTP

```
GET /api/users HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Content-Type: application/json

{"filter": {"active": true}}
```

La solicitud HTTP anterior incluye:
- **Método HTTP**: `GET` (otros incluyen POST, PUT, DELETE, PATCH, OPTIONS)
- **Ruta**: `/api/users`
- **Versión**: `HTTP/1.1`
- **Cabeceras**: Información adicional como tipo de contenido esperado, autorización, etc.
- **Cuerpo**: Datos adicionales enviados al servidor (común en POST/PUT)

##### Estructura de una respuesta HTTP

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600
Date: Wed, 31 Jul 2025 07:55:36 GMT
Content-Length: 357

{
  "data": [
    { "id": 1, "name": "Ana López", "role": "developer" },
    { "id": 2, "name": "Carlos Gómez", "role": "designer" }
  ],
  "meta": {
    "total": 2,
    "page": 1
  }
}
```

La respuesta HTTP anterior incluye:
- **Código de estado**: `200 OK`
- **Cabeceras**: Información sobre el tipo de contenido, caché, etc.
- **Cuerpo**: Datos devueltos por el servidor (generalmente en formato JSON)

##### Códigos de estado HTTP

Los códigos de estado HTTP indican el resultado de una solicitud:

- **1xx**: Informativo
  - `100 Continue`: Indica que el cliente debe continuar con la solicitud

- **2xx**: Éxito
  - `200 OK`: La solicitud ha tenido éxito
  - `201 Created`: Recurso creado exitosamente
  - `204 No Content`: Éxito, pero sin contenido para devolver

- **3xx**: Redirección
  - `301 Moved Permanently`: El recurso ha sido movido definitivamente
  - `304 Not Modified`: El recurso no ha cambiado desde la última solicitud (caché)
  - `307 Temporary Redirect`: Redirección temporal

- **4xx**: Error del cliente
  - `400 Bad Request`: Solicitud mal formada o inválida
  - `401 Unauthorized`: Autenticación requerida o fallida
  - `403 Forbidden`: Acceso denegado aún con autenticación correcta
  - `404 Not Found`: Recurso no encontrado
  - `422 Unprocessable Entity`: La solicitud está bien formada pero no puede procesarse

- **5xx**: Error del servidor
  - `500 Internal Server Error`: Error genérico en el servidor
  - `502 Bad Gateway`: El servidor actua como gateway y recibió respuesta inválida
  - `503 Service Unavailable`: Servidor temporalmente no disponible
  - `504 Gateway Timeout`: Tiempo de espera agotado en comunicación con servidor upstream

##### Ventajas de HTTPS sobre HTTP

- **Encriptación**: Los datos intercambiados están cifrados, protegiendo información sensible
- **Autenticación**: Verifica la identidad del servidor al que se conecta el cliente
- **Integridad**: Garantiza que los datos no sean modificados durante la transmisión
- **SEO**: Mejor posicionamiento en motores de búsqueda como Google
- **Confianza**: Genera mayor confianza en los usuarios al mostrar el icono de candado en el navegador

#### REST (Representational State Transfer)

REST es un estilo arquitectónico introducido por Roy Fielding en su tesis doctoral en 2000. Define un conjunto de restricciones y principios para crear servicios web escalables, mantenibles e interoperables.

##### Principios fundamentales de REST

1. **Arquitectura Cliente-Servidor**: Separación clara de responsabilidades entre cliente y servidor.

2. **Sin Estado (Stateless)**: Cada solicitud del cliente al servidor contiene toda la información necesaria para entender y procesar la petición. El servidor no almacena contexto del cliente entre solicitudes.

3. **Cacheable**: Las respuestas deben definir de manera explícita si pueden ser almacenadas en caché para mejorar la eficiencia y escalabilidad.

4. **Sistema por Capas**: Un cliente no puede determinar si está conectado directamente al servidor final o a un intermediario, lo que permite insertar proxies, balanceadores de carga, etc.

5. **Interfaz Uniforme**: Simplifica la arquitectura general mediante una interfaz estandarizada que se compone de:
   - **Identificación de recursos**: Cada recurso tiene un identificador único (URI)
   - **Manipulación de recursos a través de representaciones**: Los clientes trabajan con representaciones de recursos (JSON, XML)
   - **Mensajes autodescriptivos**: Cada mensaje incluye suficiente información para ser procesado
   - **HATEOAS** (Hypermedia as the Engine of Application State): Los clientes interactúan con la aplicación enteramente a través de hipermedios proporcionados dinámicamente por el servidor

##### Recursos y URIs

Un concepto clave en REST es el recurso, que puede ser cualquier información nombrable como una entidad o colección:

- **Recursos individuales**: `/api/users/123`, `/api/products/42`
- **Colecciones de recursos**: `/api/users`, `/api/products`
- **Recursos anidados**: `/api/users/123/orders`, `/api/departments/5/employees`

##### Métodos HTTP y su semántica en REST

- **GET**: Recupera la representación de un recurso sin modificarlo (idempotente)
- **POST**: Crea un nuevo recurso o envía información para su procesamiento
- **PUT**: Actualiza un recurso completo o lo crea si no existe (idempotente)
- **PATCH**: Actualiza parcialmente un recurso
- **DELETE**: Elimina un recurso (idempotente)
- **OPTIONS**: Devuelve los métodos HTTP permitidos para un recurso
- **HEAD**: Similar a GET pero sólo devuelve cabeceras, sin cuerpo

##### Ejemplo de API RESTful completo

```
# 1. Obtener todos los usuarios (posiblemente paginados)
GET /api/users?page=1&limit=20
Accept: application/json

# Respuesta
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    { "id": 123, "name": "Juan", "email": "juan@example.com", "role": "user" },
    { "id": 124, "name": "Ana", "email": "ana@example.com", "role": "admin" }
  ],
  "meta": {
    "total": 45,
    "page": 1,
    "limit": 20,
    "pages": 3
  },
  "links": {
    "self": "/api/users?page=1&limit=20",
    "next": "/api/users?page=2&limit=20",
    "last": "/api/users?page=3&limit=20"
  }
}

# 2. Obtener un usuario específico
GET /api/users/123
Accept: application/json

# Respuesta
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Juan",
  "email": "juan@example.com",
  "role": "user",
  "created_at": "2023-05-15T10:30:00Z",
  "links": {
    "self": "/api/users/123",
    "posts": "/api/users/123/posts"
  }
}

# 3. Crear un nuevo usuario
POST /api/users
Content-Type: application/json

{
  "name": "Carlos",
  "email": "carlos@example.com",
  "password": "contraseñaSegura123"
}

# Respuesta
HTTP/1.1 201 Created
Location: /api/users/125
Content-Type: application/json

{
  "id": 125,
  "name": "Carlos",
  "email": "carlos@example.com",
  "role": "user",
  "created_at": "2025-07-31T07:55:36Z"
}

# 4. Actualizar un usuario completamente
PUT /api/users/123
Content-Type: application/json

{
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "role": "editor"
}

# Respuesta
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "role": "editor",
  "updated_at": "2025-07-31T07:56:15Z"
}

# 5. Actualizar parcialmente un usuario
PATCH /api/users/123
Content-Type: application/json

{
  "role": "admin"
}

# Respuesta
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "role": "admin",
  "updated_at": "2025-07-31T07:57:22Z"
}

# 6. Eliminar un usuario
DELETE /api/users/123

# Respuesta
HTTP/1.1 204 No Content
```

##### Buenas prácticas en el diseño de APIs RESTful

1. **Usar sustantivos en plural para colecciones**: `/users` en lugar de `/user`
2. **Evitar verbos en las URLs**: Usar `/posts/123` en vez de `/getPosts/123`
3. **Usar parámetros de consulta para filtrado y paginación**: `/products?category=electronics&page=2`
4. **Versionado de APIs**: `/api/v1/users` permite evolucionar la API sin romper clientes existentes
5. **Respuestas consistentes**: Estructura uniforme para éxitos y errores
6. **Códigos HTTP apropiados**: Usar correctamente los códigos de estado HTTP
7. **HATEOAS**: Incluir enlaces para navegar entre recursos relacionados
8. **Documentación**: Proporcionar documentación clara con Swagger/OpenAPI

##### Herramientas para desarrollo de APIs RESTful

- **Swagger/OpenAPI**: Documentación y diseño de APIs
- **Postman/Insomnia**: Pruebas y depuración de APIs
- **Frameworks**: Express (Node.js), Spring Boot (Java), Django Rest Framework (Python)

#### GraphQL

GraphQL es un lenguaje de consulta y una especificación para APIs desarrollado por Facebook en 2015 y publicado como proyecto de código abierto en 2018. A diferencia de REST, GraphQL permite a los clientes solicitar exactamente los datos que necesitan, ofreciendo mayor flexibilidad y eficiencia en la comunicación entre cliente y servidor.

##### Conceptos fundamentales de GraphQL

1. **Schema**: Define todos los tipos de datos disponibles y las operaciones permitidas en la API.
   
2. **Types**: Define la estructura de los datos disponibles:
   - **Scalar Types**: Int, Float, String, Boolean, ID
   - **Object Types**: Tipos personalizados como User, Post, Comment
   - **Input Types**: Tipos usados en mutaciones para entrada de datos
   - **Enum Types**: Conjunto predefinido de valores permitidos
   - **Interface & Union Types**: Para tipos polimórficos

3. **Operaciones**:
   - **Query**: Obtener datos (similar a GET en REST)
   - **Mutation**: Modificar datos (similar a POST, PUT, DELETE)
   - **Subscription**: Recibir actualizaciones en tiempo real

4. **Resolvers**: Funciones en el servidor que resuelven cómo obtener los datos para cada campo solicitado.

##### Ejemplo de schema GraphQL

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String
  author: User!
  createdAt: String!
  comments: [Comment!]!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
  createdAt: String!
}

type Query {
  user(id: ID!): User
  users: [User!]!
  post(id: ID!): Post
  posts: [Post!]!
}

type Mutation {
  createUser(name: String!, email: String!): User!
  createPost(title: String!, content: String, authorId: ID!): Post!
  createComment(text: String!, postId: ID!, authorId: ID!): Comment!
}
```

##### Ejemplo de consulta GraphQL y su respuesta

```graphql
# Consulta GraphQL para obtener un usuario y sus posts
query {
  user(id: "123") {
    name
    email
    posts {
      title
      createdAt
      comments {
        text
        author {
          name
        }
      }
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
          "createdAt": "2023-05-15",
          "comments": [
            {
              "text": "Excelente artículo!",
              "author": {
                "name": "Ana Gómez"
              }
            }
          ]
        },
        {
          "title": "REST vs GraphQL",
          "createdAt": "2023-06-10",
          "comments": []
        }
      ]
    }
  }
}
```

##### Ejemplo de mutación GraphQL

```graphql
# Mutación para crear un nuevo post
mutation {
  createPost(
    title: "Ventajas de GraphQL", 
    content: "GraphQL ofrece muchas ventajas como...", 
    authorId: "123"
  ) {
    id
    title
    author {
      name
    }
    createdAt
  }
}

# Respuesta JSON
{
  "data": {
    "createPost": {
      "id": "456",
      "title": "Ventajas de GraphQL",
      "author": {
        "name": "Juan Pérez"
      },
      "createdAt": "2025-07-31T08:15:20Z"
    }
  }
}
```

##### GraphQL vs REST: Comparativa

| Característica | GraphQL | REST |
|-----------------|---------|------|
| **Solicitud de datos** | Flexible: el cliente especifica exactamente qué datos necesita | Fija: el servidor determina qué datos devolver |
| **Endpoints** | Un único endpoint para todas las operaciones | Múltiples endpoints para diferentes recursos |
| **Overfetching/Underfetching** | Minimizado, ya que el cliente solicita sólo lo necesario | Común, a menudo se obtienen datos de más o de menos |
| **Versionado** | No requiere versionado explícito | A menudo requiere versionado de API |
| **Caché** | Más complejo de implementar | Más sencillo con HTTP estándar |
| **Curva de aprendizaje** | Moderada a alta | Baja a moderada |
| **Documentación** | Auto-documentado con introspection | Requiere documentación externa (Swagger, etc.) |
| **Control de errores** | Estructura de errores unificada | Depende de la implementación |

##### Cuándo usar GraphQL

- **Aplicaciones con necesidades de datos variables**: Interfaces que muestran diferentes vistas de los mismos datos
- **Aplicaciones móviles**: Donde el ancho de banda puede ser limitado y es crucial minimizar la cantidad de datos transmitidos
- **Agregación de múltiples fuentes**: Cuando necesitas combinar datos de diversas APIs o servicios
- **Interfaces complejas**: Donde diferentes componentes requieren diferentes partes de los mismos datos

##### Implementaciones populares de GraphQL

- **Apollo Server/Client**: Ecosistema completo para GraphQL en JavaScript
- **Relay**: Framework de Facebook para React con optimizaciones específicas
- **Hasura**: Genera automáticamente APIs GraphQL desde bases de datos
- **Prisma**: ORM moderno que facilita la creación de APIs GraphQL
- **GraphQL Java**: Implementación para Java y la JVM
- **Strawberry**: Framework GraphQL para Python

### 7.1.3 Node.js y Express

#### Node.js: La plataforma de servidor JavaScript

Node.js es un entorno de ejecución para JavaScript del lado del servidor basado en el motor V8 de Chrome. Creado en 2009 por Ryan Dahl, revolucionó el desarrollo backend al permitir usar JavaScript fuera del navegador, ofreciendo un modelo de E/S no bloqueante y orientado a eventos.

##### Características clave de Node.js

- **Modelo de E/S no bloqueante**: Permite manejar muchas conexiones simultáneas sin necesidad de crear hilos adicionales.
- **Event Loop**: El bucle de eventos permite operaciones asíncronas eficientes.
- **JavaScript en todas partes**: Unifica el lenguaje de programación en cliente y servidor.
- **NPM (Node Package Manager)**: Uno de los ecosistemas de paquetes más grandes, con más de un millón de paquetes.
- **Uso eficiente de memoria**: Ideal para aplicaciones en tiempo real y microservicios.

##### Ejemplo básico de servidor HTTP con Node.js puro

```javascript
// Servidor HTTP básico con Node.js nativo
const http = require('http');

const server = http.createServer((req, res) => {
  const { url, method } = req;
  
  // Establecer cabeceras de respuesta
  res.setHeader('Content-Type', 'application/json');
  
  // Manejar diferentes rutas
  if (url === '/') {
    res.statusCode = 200;
    res.end(JSON.stringify({ message: '¡Hola mundo!' }));
  } 
  else if (url === '/api/users' && method === 'GET') {
    const users = [
      { id: 1, name: 'Ana' },
      { id: 2, name: 'Carlos' }
    ];
    res.statusCode = 200;
    res.end(JSON.stringify(users));
  } 
  else {
    res.statusCode = 404;
    res.end(JSON.stringify({ error: 'Ruta no encontrada' }));
  }
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Servidor ejecutándose en http://localhost:${PORT}`);
});
```

#### Express: Framework web para Node.js

Express.js es el framework web más popular para Node.js. Proporciona una capa de abstracción sobre el módulo HTTP de Node.js, facilitando la creación de APIs y aplicaciones web con una sintaxis más limpia y funcionalidades avanzadas.

##### Ventajas de Express

- **Minimalista pero potente**: Núcleo ligero que se puede extender con middleware.
- **Sistema de middleware**: Permite procesar solicitudes de forma modular.
- **Routing avanzado**: Sistema de enrutamiento flexible y expresivo.
- **Templates**: Soporte para múltiples motores de plantillas.
- **Performance**: Optimizado para aplicaciones de alto rendimiento.
- **Ecosistema**: Amplia comunidad y disponibilidad de plugins.

##### Ejemplo completo de API RESTful con Express

```javascript
// Servidor completo con Express
const express = require('express');
const morgan = require('morgan'); // Middleware de logging
const helmet = require('helmet'); // Middleware de seguridad
const cors = require('cors'); // Middleware para CORS

const app = express();
const PORT = process.env.PORT || 3000;

// Configuración de middleware
app.use(express.json()); // Para parsear JSON en el body
app.use(express.urlencoded({ extended: true })); // Para parsear datos de formularios
app.use(morgan('dev')); // Logging de solicitudes HTTP
app.use(helmet()); // Seguridad con cabeceras HTTP
app.use(cors()); // Habilitar CORS

// Base de datos simulada
let users = [
  { id: 1, name: 'Ana López', email: 'ana@example.com', role: 'admin' },
  { id: 2, name: 'Carlos Gómez', email: 'carlos@example.com', role: 'user' }
];

// Middleware personalizado para verificar API key
const verifyApiKey = (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey || apiKey !== 'tu-api-key-secreta') {
    return res.status(401).json({ error: 'API key inválida o no proporcionada' });
  }
  next();
};

// Middleware para validar existencia de usuario
const validateUserExists = (req, res, next) => {
  const userId = parseInt(req.params.id);
  const userExists = users.some(user => user.id === userId);
  
  if (!userExists) {
    return res.status(404).json({ error: `Usuario con id ${userId} no encontrado` });
  }
  next();
};

// Middleware para validar datos de usuario
const validateUserData = (req, res, next) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Nombre y email son obligatorios' });
  }
  
  if (email && !email.includes('@')) {
    return res.status(400).json({ error: 'Formato de email inválido' });
  }
  
  next();
};

// Agrupar rutas con Router
const userRouter = express.Router();

// CRUD para usuarios
// GET - Obtener todos los usuarios
userRouter.get('/', (req, res) => {
  // Soporte para filtrado mediante query params
  const { role } = req.query;
  
  if (role) {
    const filteredUsers = users.filter(user => user.role === role);
    return res.json(filteredUsers);
  }
  
  res.json(users);
});

// GET - Obtener un usuario por ID
userRouter.get('/:id', validateUserExists, (req, res) => {
  const userId = parseInt(req.params.id);
  const user = users.find(user => user.id === userId);
  res.json(user);
});

// POST - Crear un nuevo usuario
userRouter.post('/', validateUserData, (req, res) => {
  const { name, email, role = 'user' } = req.body;
  
  // Validar si el email ya existe
  const emailExists = users.some(user => user.email === email);
  if (emailExists) {
    return res.status(409).json({ error: 'El email ya está en uso' });
  }
  
  // Generar nuevo ID
  const newId = users.length > 0 ? Math.max(...users.map(user => user.id)) + 1 : 1;
  
  // Crear usuario
  const newUser = {
    id: newId,
    name,
    email,
    role,
    createdAt: new Date().toISOString()
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT - Actualizar un usuario completamente
userRouter.put('/:id', validateUserExists, validateUserData, (req, res) => {
  const userId = parseInt(req.params.id);
  const { name, email, role } = req.body;
  
  // Actualizar usuario
  users = users.map(user => {
    if (user.id === userId) {
      return {
        ...user,
        name,
        email,
        role: role || user.role,
        updatedAt: new Date().toISOString()
      };
    }
    return user;
  });
  
  const updatedUser = users.find(user => user.id === userId);
  res.json(updatedUser);
});

// DELETE - Eliminar un usuario
userRouter.delete('/:id', validateUserExists, (req, res) => {
  const userId = parseInt(req.params.id);
  users = users.filter(user => user.id !== userId);
  res.status(204).send();
});

// Registrar router de usuarios
app.use('/api/users', verifyApiKey, userRouter);

// Middleware para manejo centralizado de errores
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Error interno del servidor' });
});

// Manejar rutas no encontradas
app.use((req, res) => {
  res.status(404).json({ error: 'Ruta no encontrada' });
});

// Iniciar el servidor
app.listen(PORT, () => {
  console.log(`Servidor Express ejecutándose en http://localhost:${PORT}`);
});
```

##### Estructura de proyecto Express recomendada

Para aplicaciones medianas o grandes, es recomendable estructurar el proyecto de manera modular:

```
project-root/
├── node_modules/
├── src/
│   ├── controllers/
│   │   ├── userController.js
│   │   └── productController.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── validation.js
│   ├── models/
│   │   ├── userModel.js
│   │   └── productModel.js
│   ├── routes/
│   │   ├── userRoutes.js
│   │   └── productRoutes.js
│   ├── services/
│   │   ├── userService.js
│   │   └── productService.js
│   ├── utils/
│   │   ├── logger.js
│   │   └── helpers.js
│   ├── config/
│   │   ├── database.js
│   │   └── environment.js
│   └── app.js
├── .env
├── .gitignore
├── package.json
├── README.md
└── server.js
```

##### Alternativas a Express

Existen otras opciones para el desarrollo backend en Node.js:

- **Fastify**: Más rápido que Express, con un sistema de plugins y esquemas JSON.
- **Koa**: Desarrollado por el equipo de Express, más ligero y enfocado en middleware asíncronos con async/await.
- **Nest.js**: Framework inspirado en Angular, con arquitectura modular y TypeScript.
- **Hapi**: Enfocado en servicios API modulares con configuración rica.
- **Sails.js**: Framework MVC inspirado en Ruby on Rails.
```

### 7.1.4 Middleware

Los middleware son funciones que se ejecutan durante el ciclo de vida de una solicitud HTTP en una aplicación web. Actúan como "capas" intermedias entre la solicitud del cliente y la respuesta del servidor, permitiendo manipular, procesar, modificar, validar o enriquecer tanto la solicitud como la respuesta.

#### Concepto y flujo del middleware

En su esencia, un middleware es una función que recibe tres parámetros:
- `req`: El objeto de solicitud HTTP
- `res`: El objeto de respuesta HTTP
- `next`: Una función que ejecuta el siguiente middleware en la cadena

El flujo de procesamiento de middleware puede visualizarse como una "tubería" por la que pasa cada solicitud:

```
Solicitud HTTP → Middleware 1 → Middleware 2 → ... → Controlador → ... → Middleware N → Respuesta HTTP
```

Cada middleware puede:
1. Ejecutar código arbitrario
2. Modificar los objetos de solicitud y respuesta
3. Finalizar el ciclo solicitud-respuesta
4. Llamar al siguiente middleware en la pila

#### Tipos de middleware

1. **Middleware de aplicación**: Se aplica a toda la aplicación
   ```javascript
   app.use(middlewareFunction);
   ```

2. **Middleware de ruta**: Se aplica solo a rutas específicas
   ```javascript
   app.use('/api/users', middlewareFunction);
   ```

3. **Middleware de manejo de errores**: Tiene un parámetro adicional para el error
   ```javascript
   app.use((err, req, res, next) => { /* código */ });
   ```

4. **Middleware incorporado**: Proporcionado por el framework (Express, etc.)
   ```javascript
   app.use(express.json());
   ```

5. **Middleware de terceros**: Desarrollado por la comunidad
   ```javascript
   app.use(helmet());
   ```

#### Ejemplos prácticos de middleware

##### Middleware de autenticación

```javascript
// Middleware de autenticación con JWT
const jwt = require('jsonwebtoken');

const authMiddleware = (req, res, next) => {
  // Obtener el token del encabezado de autorización
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ 
      error: 'Acceso denegado', 
      message: 'Token de autenticación no proporcionado o formato inválido'
    });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    // Verificar token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Añadir información del usuario al objeto de solicitud
    req.user = decoded;
    
    // Continuar con la siguiente función en la pila
    next();
  } catch (error) {
    return res.status(401).json({ 
      error: 'Token inválido', 
      message: error.message 
    });
  }
};

// Aplicar middleware a rutas específicas
app.get('/api/profile', authMiddleware, (req, res) => {
  res.json({ message: 'Perfil protegido', user: req.user });
});
```

##### Middleware de registro (logging)

```javascript
// Middleware personalizado para registro de solicitudes
const requestLogger = (req, res, next) => {
  const start = Date.now();
  const { method, url, ip } = req;
  
  // Ejecutar código cuando se envió la respuesta (usando el evento 'finish')
  res.on('finish', () => {
    const responseTime = Date.now() - start;
    const { statusCode } = res;
    
    console.log(`${new Date().toISOString()} | ${method} ${url} | Status: ${statusCode} | IP: ${ip} | ${responseTime}ms`);
  });
  
  next();
};
```

##### Middleware de validación

```javascript
// Middleware para validar datos de producto usando express-validator
const { body, validationResult } = require('express-validator');

const validateProduct = [
  // Validación de campos
  body('name')
    .trim()
    .notEmpty().withMessage('El nombre es obligatorio')
    .isLength({ min: 3, max: 100 }).withMessage('El nombre debe tener entre 3 y 100 caracteres'),
  
  body('price')
    .notEmpty().withMessage('El precio es obligatorio')
    .isFloat({ min: 0.01 }).withMessage('El precio debe ser un número positivo mayor a 0'),
  
  body('category')
    .trim()
    .notEmpty().withMessage('La categoría es obligatoria')
    .isIn(['electrónica', 'ropa', 'hogar', 'alimentos']).withMessage('Categoría no válida'),
  
  // Middleware que procesa los resultados de la validación
  (req, res, next) => {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    
    next();
  }
];
```

##### Middleware de compresión

```javascript
// Middleware de compresión para reducir el tamaño de las respuestas
const compression = require('compression');

// Comprimir todas las respuestas
app.use(compression());

// O configurar opciones específicas
app.use(compression({
  level: 6, // Nivel de compresión (0-9)
  threshold: 1024, // Umbral mínimo en bytes para comprimir
  filter: (req, res) => {
    // No comprimir respuestas ya comprimidas o imágenes
    if (req.headers['content-type']?.includes('image/')) {
      return false;
    }
    // Comprimir por defecto
    return compression.filter(req, res);
  }
}));
```

##### Middleware de limitación de tasa (rate limiting)

```javascript
// Middleware para limitar solicitudes y prevenir ataques DoS
const rateLimit = require('express-rate-limit');

// Configuración básica
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 100, // Límite de 100 solicitudes por ventana de tiempo
  standardHeaders: true, // Devolver info de límite en cabeceras X-RateLimit-*
  legacyHeaders: false, // Deshabilitar cabeceras 'X-RateLimit-*'
  message: 'Demasiadas solicitudes, por favor intente nuevamente más tarde.',
  handler: (req, res, next, options) => {
    return res.status(429).json({
      error: 'Límite de tasa excedido',
      message: options.message,
      retryAfter: Math.ceil(options.windowMs / 1000 / 60) // Minutos para reintentar
    });
  }
});

// Aplicar limiter a todas las solicitudes que usan la API
app.use('/api/', apiLimiter);

// Limiter más estricto para rutas de autenticación
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hora
  max: 5, // Límite de 5 intentos por hora
  message: 'Demasiados intentos fallidos, intente nuevamente en una hora.'
});

app.use('/api/auth/login', authLimiter);
```

#### Cadenas de middleware

Se pueden encadenar múltiples middlewares para una ruta específica:

```javascript
app.post('/api/articles',
  authMiddleware,         // Primero verifica la autenticación
  requestLogger,          // Luego registra la solicitud
  validateArticleData,    // Después valida los datos
  uploadMiddleware,       // Procesa subida de archivos si hay
  (req, res) => {         // Finalmente, maneja la solicitud
    // Lógica para crear el artículo
  }
);
```

#### Mejores prácticas para middleware

1. **Mantener middlewares pequeños y enfocados**: Cada middleware debe tener una única responsabilidad.

2. **Ordenar correctamente**: Coloca middlewares en el orden correcto, ya que se ejecutan secuencialmente.

3. **Manejo adecuado de errores**: Utiliza middleware de errores para capturar excepciones.

4. **Documentar middleware**: Especialmente para middleware compartido entre diferentes partes de la aplicación.

5. **Evitar middleware asíncronos sin manejo adecuado**: Si un middleware usa operaciones asíncronas, asegúrate de manejar correctamente errores y llamar a `next()`.

```javascript
// ❌ Mal - Middleware asincrónico sin manejo adecuado
app.use(async (req, res, next) => {
  const result = await someAsyncOperation(); // Si falla, next() nunca se llama
  req.result = result;
  next();
});

// ✅ Bien - Middleware asincrónico con manejo de errores
app.use(async (req, res, next) => {
  try {
    const result = await someAsyncOperation();
    req.result = result;
    next();
  } catch (error) {
    next(error); // Pasar error al middleware de errores
  }
});
```

6. **Usar middleware incorporado y de terceros cuando sea posible**: Evita reinventar la rueda para funcionalidades comunes.

7. **Modularizar middleware complejo**: Divide middlewares complejos en componentes más pequeños y reutilizables.

8. **Considerar el rendimiento**: Algunos middlewares pueden afectar significativamente el rendimiento, especialmente los que realizan operaciones pesadas.

## 7.2 Bases de Datos y ORM

Las bases de datos son componentes esenciales en el desarrollo de aplicaciones backend, ya que proporcionan mecanismos para almacenar, recuperar, actualizar y gestionar datos de forma persistente. Por otro lado, los ORM (Object-Relational Mapping) son herramientas que facilitan la interacción entre el código orientado a objetos y las bases de datos relacionales.

### 7.2.1 Tipos de Bases de Datos

Existe una amplia variedad de bases de datos, cada una con características distintivas que las hacen adecuadas para diferentes casos de uso. Podemos clasificarlas en dos grandes categorías principales:

#### Bases de Datos Relacionales (SQL)

Las bases de datos relacionales organizan la información en tablas estructuradas (relaciones) con filas y columnas, donde los datos están conectados entre sí mediante relaciones definidas por claves primarias y foráneas. Siguen el modelo relacional propuesto por E.F. Codd en 1970 y utilizan SQL (Structured Query Language) como lenguaje estándar para consultas.

##### Características principales

- **Estructura predefinida**: Requieren un esquema fijo con tipos de datos y restricciones definidas previamente.
- **Transacciones ACID**: Garantizan Atomicidad, Consistencia, Aislamiento y Durabilidad.
- **Relaciones**: Permiten establecer conexiones entre diferentes tablas mediante claves.
- **Normalización**: Proceso para eliminar redundancias y dependencias indeseadas.
- **JOIN**: Capacidad para combinar datos de múltiples tablas en consultas.

##### Ejemplos populares

- **PostgreSQL**: Sistema de código abierto con excelente rendimiento y funcionalidades avanzadas.
- **MySQL/MariaDB**: Ampliamente utilizado, especialmente en aplicaciones web.
- **SQLite**: Base de datos ligera basada en archivos, ideal para aplicaciones móviles.
- **SQL Server**: Solución robusta de Microsoft con buen soporte empresarial.
- **Oracle Database**: Sistema con gran escalabilidad para entornos empresariales.

##### Ejemplo de definición de esquema SQL

```sql
-- Ejemplo de SQL para crear tablas relacionadas
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  role VARCHAR(20) DEFAULT 'user',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  content TEXT,
  status VARCHAR(20) DEFAULT 'draft',
  user_id INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  content TEXT NOT NULL,
  post_id INTEGER NOT NULL,
  user_id INTEGER NOT NULL,
  parent_comment_id INTEGER,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (parent_comment_id) REFERENCES comments(id) ON DELETE SET NULL
);

-- Índice para optimizar búsquedas por email de usuario
CREATE INDEX idx_users_email ON users(email);

-- Índice compuesto para búsquedas de posts por usuario y estado
CREATE INDEX idx_posts_user_status ON posts(user_id, status);
```

##### Ejemplo de operaciones CRUD en SQL

```sql
-- CREATE: Insertar un nuevo usuario
INSERT INTO users (name, email, role) 
VALUES ('Ana García', 'ana@ejemplo.com', 'admin');

-- READ: Seleccionar usuarios con sus posts
SELECT u.name, p.title, p.created_at 
FROM users u
JOIN posts p ON u.id = p.user_id
WHERE u.role = 'admin' AND p.status = 'published'
ORDER BY p.created_at DESC
LIMIT 10;

-- UPDATE: Actualizar el rol de un usuario
UPDATE users
SET role = 'editor', updated_at = CURRENT_TIMESTAMP
WHERE email = 'ana@ejemplo.com';

-- DELETE: Eliminar posts antiguos
DELETE FROM posts
WHERE status = 'draft' AND created_at < NOW() - INTERVAL '1 year';
```

##### Ventajas de las bases de datos relacionales

- **Integridad de datos**: Garantizan la consistencia mediante restricciones y validaciones.
- **Transacciones seguras**: Operaciones atómicas que mantienen la coherencia de los datos.
- **Consultas complejas**: Capacidad para realizar consultas sofisticadas con JOIN, agregaciones, etc.
- **Madurez y soporte**: Sistemas probados durante décadas con amplia documentación.

##### Desventajas y limitaciones

- **Escalabilidad horizontal**: Pueden ser más difíciles de escalar horizontalmente que soluciones NoSQL.
- **Esquemas rígidos**: Los cambios en la estructura requieren migraciones.
- **Rendimiento**: En algunos escenarios específicos, pueden ser menos eficientes que alternativas NoSQL.

#### Bases de Datos NoSQL

Las bases de datos NoSQL ("Not Only SQL") son sistemas de almacenamiento que proporcionan mecanismos para almacenar y recuperar datos modelados de formas diferentes a las tablas relacionales tradicionales. Surgieron como respuesta a las limitaciones de las bases de datos relacionales para ciertas cargas de trabajo modernas, especialmente en entornos web de gran escala, aplicaciones en tiempo real y big data.

##### Tipos principales de bases de datos NoSQL

1. **Bases de datos de documentos**
   - Almacenan datos en documentos semiestructurados (JSON, BSON, XML)
   - Cada documento puede tener una estructura diferente
   - Ejemplos: MongoDB, CouchDB, Firebase Firestore

2. **Bases de datos clave-valor**
   - Modelo de datos más simple: cada elemento se almacena como un par clave-valor
   - Alta eficiencia para operaciones sencillas de lectura/escritura
   - Ejemplos: Redis, Amazon DynamoDB, Riak

3. **Bases de datos columnares**
   - Optimizadas para consultas en grandes conjuntos de datos
   - Almacenan datos en columnas en lugar de filas
   - Ejemplos: Cassandra, HBase, Google Bigtable

4. **Bases de datos de grafos**
   - Optimizadas para datos interconectados y relaciones complejas
   - Utilizan nodos, propiedades y aristas
   - Ejemplos: Neo4j, ArangoDB, Amazon Neptune

##### Ejemplo de base de datos de documentos (MongoDB)

```javascript
// Ejemplo de documento en MongoDB representando un usuario con posts anidados
{
  "_id": ObjectId("5f8a3d25b2e90f4a1c1e9d5a"),
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "role": "author",
  "profile": {
    "bio": "Desarrollador fullstack especializado en JavaScript",
    "location": "Barcelona",
    "social": {
      "twitter": "@juanperez",
      "github": "juanperezdev"
    }
  },
  "posts": [
    {
      "title": "Introducción a NoSQL",
      "content": "Las bases de datos NoSQL ofrecen flexibilidad...",
      "tags": ["nosql", "databases", "mongodb"],
      "published_at": ISODate("2023-05-15T10:30:00Z"),
      "comments": [
        { 
          "text": "Excelente artículo, muy claro", 
          "author": "Ana López", 
          "created_at": ISODate("2023-05-16T08:45:00Z") 
        },
        { 
          "text": "Me gustaría ver más ejemplos prácticos", 
          "author": "Carlos Gómez", 
          "created_at": ISODate("2023-05-17T14:22:00Z") 
        }
      ]
    },
    {
      "title": "Patrones de diseño en MongoDB",
      "content": "A la hora de modelar datos en MongoDB...",
      "tags": ["mongodb", "design patterns"],
      "published_at": ISODate("2023-06-20T09:15:00Z"),
      "comments": []
    }
  ],
  "last_login": ISODate("2023-07-01T11:42:15Z"),
  "created_at": ISODate("2023-01-15T08:30:00Z")
}
```

##### Ejemplo de base de datos clave-valor (Redis)

```bash
# Almacenar un string simple
SET user:1000:name "Juan Pérez"

# Almacenar un hash con múltiples campos
HMSET user:1000 name "Juan Pérez" email "juan@example.com" role "author"

# Obtener un valor
GET user:1000:name

# Obtener todos los campos del hash
HGETALL user:1000

# Listas para almacenar colecciones ordenadas
RPUSH user:1000:posts "post:5001" "post:5002"

# Sets para colecciones únicas no ordenadas
SADD user:1000:skills "javascript" "node.js" "mongodb"

# Sorted Sets para elementos ordenados con puntuación
ZADD trending_posts 120 "post:5001" 94 "post:5002" 65 "post:4892"
```

##### Ejemplo de base de datos grafo (Neo4j - Cypher Query Language)

```cypher
// Crear nodos de usuarios
CREATE (juan:User {name: 'Juan Pérez', email: 'juan@example.com'})
CREATE (ana:User {name: 'Ana López', email: 'ana@example.com'})
CREATE (carlos:User {name: 'Carlos Gómez', email: 'carlos@example.com'})

// Crear nodos de posts
CREATE (post1:Post {title: 'Introducción a NoSQL', content: 'Las bases de datos NoSQL...'}) 
CREATE (post2:Post {title: 'Bases de datos de grafos', content: 'Neo4j es una base de datos...'}) 

// Crear relaciones
CREATE (juan)-[:AUTHORED]->(post1)
CREATE (ana)-[:COMMENTED {date: '2023-05-16'}]->(post1)
CREATE (carlos)-[:COMMENTED {date: '2023-05-17'}]->(post1)
CREATE (carlos)-[:AUTHORED]->(post2)
CREATE (juan)-[:FOLLOWS]->(ana)
CREATE (ana)-[:FOLLOWS]->(juan)
CREATE (ana)-[:FOLLOWS]->(carlos)

// Consulta: Encontrar todos los comentarios en posts escritos por Juan
MATCH (u:User {name: 'Juan Pérez'})-[:AUTHORED]->(p:Post)<-[c:COMMENTED]-(commenter)
RETURN p.title AS post, commenter.name AS commenter, c.date AS date
```

##### Ventajas de las bases de datos NoSQL

- **Esquemas flexibles**: Adaptables a cambios en los requerimientos sin necesidad de migraciones complejas.
- **Escalabilidad horizontal**: Diseñadas para distribuirse fácilmente entre múltiples servidores.
- **Alto rendimiento**: Optimizadas para patrones de acceso específicos.
- **Manejo de grandes volúmenes**: Adecuadas para big data y casos de uso con muchos datos.
- **Disponibilidad**: Muchas implementaciones priorizan disponibilidad sobre consistencia inmediata (teorema CAP).

##### Desventajas y limitaciones

- **Consistencia eventual**: Algunas soluciones sacrifican consistencia inmediata por disponibilidad.
- **Falta de estándares**: No hay un lenguaje de consulta estándar como SQL.
- **Menor soporte para transacciones**: Las operaciones ACID complejas pueden ser más difíciles de implementar.
- **Curva de aprendizaje**: Requiere nuevos patrones de diseño y mentalidad diferente al modelo relacional.

##### Cuándo usar NoSQL vs SQL

| Caso de uso | SQL | NoSQL |
|-------------|-----|-------|
| **Datos estructurados con esquema fijo** | ✅ Ideal | ❌ No óptimo |
| **Transacciones complejas y joins** | ✅ Excelente | ❌ Limitado |
| **Aplicaciones con datos variables** | ❌ Requiere migraciones | ✅ Flexible |
| **Aplicaciones de gran escala** | ❌ Escalamiento vertical costoso | ✅ Escalamiento horizontal nativo |
| **Desarrollo ágil e iterativo** | ❌ Cambios de esquema complejos | ✅ Esquema dinámico |
| **Análisis de relaciones complejas** | ✅ Bueno (con joins) | ✅ Excelente con bases de grafos |

### 7.2.2 ORM (Object-Relational Mapping)

El mapeo objeto-relacional (ORM) es una técnica de programación que permite la conversión de datos entre sistemas de tipos incompatibles, específicamente entre el mundo orientado a objetos de los lenguajes de programación y el mundo relacional de las bases de datos. Los ORM actúan como una capa de abstracción entre el código de la aplicación y la base de datos, permitiendo a los desarrolladores trabajar con objetos en lugar de con sentencias SQL directas.

#### Conceptos fundamentales de los ORM

1. **Mapeo de objetos a tablas**: Un ORM mapea clases a tablas y objetos a filas, facilitando la persistencia de datos.

2. **Abstracción de la base de datos**: El ORM maneja las diferencias entre distintos sistemas de bases de datos, lo que facilita la migración entre ellos.

3. **Reducción del código repetitivo**: Elimina la necesidad de escribir consultas SQL básicas, permitiendo centrarse en la lógica de negocio.

4. **Carga perezosa (lazy loading)**: Capacidad para cargar relaciones solo cuando realmente se necesitan, optimizando el rendimiento.

5. **Transacciones**: Los ORM suelen proporcionar abstracciones para gestionar transacciones de base de datos.

6. **Validaciones**: Incluyen mecanismos para validar datos antes de persistirlos en la base de datos.

#### Ventajas y desventajas de usar ORM

##### Ventajas

- **Productividad**: Reducción significativa de código repetitivo y simplificación del acceso a datos.
- **Mantenibilidad**: Código más limpio, organizado y fácil de mantener.
- **Portabilidad**: Facilita el cambio entre diferentes sistemas de bases de datos.
- **Seguridad**: Ayuda a prevenir inyecciones SQL al parametrizar automáticamente las consultas.

##### Desventajas

- **Curva de aprendizaje**: Requiere tiempo para aprender el funcionamiento y las peculiaridades del ORM.
- **Rendimiento**: En operaciones complejas, el SQL generado automáticamente puede ser menos eficiente que consultas manuales optimizadas.
- **Abstracción excesiva**: Puede hacer que los desarrolladores pierdan conocimiento sobre el funcionamiento real de la base de datos.
- **Complejidad adicional**: Para casos muy simples, usar un ORM podría añadir complejidad innecesaria.

#### ORM populares para diferentes lenguajes

| Lenguaje | ORM populares |
|----------|---------------|
| JavaScript/Node.js | Sequelize, TypeORM, Prisma, Objection.js |
| Python | SQLAlchemy, Django ORM, Peewee |
| Java | Hibernate, EclipseLink, MyBatis |
| PHP | Doctrine, Eloquent (Laravel), Propel |
| Ruby | Active Record (Rails), Sequel |
| C# | Entity Framework, Dapper, NHibernate |
| Go | GORM, SQLBoiler |

#### Sequelize (para bases de datos SQL con Node.js)

Sequelize es uno de los ORM más populares para Node.js, compatible con múltiples bases de datos relacionales como PostgreSQL, MySQL, SQLite y SQL Server. Proporciona una API robusta para definir modelos, establecer relaciones, realizar consultas complejas y gestionar migraciones.

##### Instalación y configuración inicial

```bash
# Instalación del paquete principal y el driver para PostgreSQL
npm install sequelize pg pg-hstore

# Para otros drivers:
# npm install mysql2       # MySQL
# npm install sqlite3      # SQLite
# npm install tedious      # SQL Server
```

##### Configuración y conexión a la base de datos

```javascript
// Importación de módulos
const { Sequelize, DataTypes } = require('sequelize');

// Conexión a la base de datos
const sequelize = new Sequelize({
  database: process.env.DB_NAME || 'my_database',
  username: process.env.DB_USER || 'db_user',
  password: process.env.DB_PASSWORD || 'db_password',
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 5432,
  dialect: 'postgres', // o 'mysql', 'sqlite', 'mariadb', 'mssql'
  logging: false, // Desactiva logs de SQL (true para activarlos)
  pool: {
    max: 5,           // Máximo de conexiones en el pool
    min: 0,           // Mínimo de conexiones en el pool
    acquire: 30000,   // Tiempo máximo (ms) para obtener una conexión
    idle: 10000       // Tiempo máximo (ms) que una conexión puede estar inactiva
  },
});

// Comprobar conexión
async function testConnection() {
  try {
    await sequelize.authenticate();
    console.log('Conexión establecida correctamente.');
  } catch (error) {
    console.error('Error al conectar con la base de datos:', error);
  }
}

testConnection();
```

##### Definición de modelos y relaciones

```javascript
// Definición del modelo Usuario
const User = sequelize.define('User', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  name: {
    type: DataTypes.STRING(100),
    allowNull: false
  },
  email: {
    type: DataTypes.STRING(100),
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true
    }
  },
  role: {
    type: DataTypes.ENUM('user', 'admin', 'editor'),
    defaultValue: 'user'
  },
  status: {
    type: DataTypes.BOOLEAN,
    defaultValue: true
  },
  lastLogin: {
    type: DataTypes.DATE
  }
}, {
  // Opciones del modelo
  tableName: 'users',        // Nombre personalizado para la tabla
  timestamps: true,          // Crea campos createdAt y updatedAt
  paranoid: true,            // Soft deletes (añade deletedAt)
  underscored: true,         // Usa snake_case para nombres de columnas
  indexes: [
    { unique: true, fields: ['email'] },
    { fields: ['role', 'status'] }
  ],
  hooks: {
    // Hook ejecutado antes de crear un usuario
    beforeCreate: async (user) => {
      if (user.password) {
        // Ejemplo: hash de contraseña
        // user.password = await bcrypt.hash(user.password, 10);
      }
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
    type: DataTypes.STRING(200),
    allowNull: false
  },
  content: {
    type: DataTypes.TEXT
  },
  status: {
    type: DataTypes.ENUM('draft', 'published', 'archived'),
    defaultValue: 'draft'
  },
  publishedAt: {
    type: DataTypes.DATE
  }
}, {
  tableName: 'posts',
  timestamps: true,
  underscored: true
});

// Definición del modelo Comment
const Comment = sequelize.define('Comment', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  content: {
    type: DataTypes.TEXT,
    allowNull: false
  }
}, {
  tableName: 'comments',
  timestamps: true,
  underscored: true
});

// Definición de relaciones

// Un usuario puede tener muchos posts (1:N)
User.hasMany(Post, {
  foreignKey: {
    name: 'userId',
    allowNull: false
  },
  onDelete: 'CASCADE'
});
Post.belongsTo(User, { foreignKey: 'userId' });

// Un post puede tener muchos comentarios (1:N)
Post.hasMany(Comment, {
  foreignKey: {
    name: 'postId',
    allowNull: false
  },
  onDelete: 'CASCADE'
});
Comment.belongsTo(Post, { foreignKey: 'postId' });

// Un usuario puede escribir muchos comentarios (1:N)
User.hasMany(Comment, {
  foreignKey: {
    name: 'userId',
    allowNull: false
  }
});
Comment.belongsTo(User, { foreignKey: 'userId' });

// Relación muchos a muchos (N:M) entre usuarios (seguidores y seguidos)
User.belongsToMany(User, {
  through: 'Followers',
  as: 'followers',
  foreignKey: 'followedId'
});

User.belongsToMany(User, {
  through: 'Followers',
  as: 'following',
  foreignKey: 'followerId'
});
```

##### Operaciones CRUD y consultas avanzadas

```javascript
// Uso de los modelos
async function ejemplosSequelize() {
  try {
    // Sincroniza modelos con la base de datos (solo en desarrollo)
    // En producción es mejor usar migraciones
    await sequelize.sync({ force: false, alter: true });
    
    // CREATE: Crear un usuario
    const user = await User.create({
      name: 'Ana García',
      email: 'ana@ejemplo.com',
      role: 'editor'
    });
    
    // CREATE: Crear un post asociado al usuario
    const post = await Post.create({
      title: 'Guía completa de Sequelize',
      content: 'Sequelize es un ORM para Node.js que soporta...',
      status: 'published',
      publishedAt: new Date(),
      userId: user.id  // Asociación con el usuario
    });
    
    // Crear comentarios
    await Comment.create({
      content: 'Excelente artículo, muy útil!',
      userId: user.id,
      postId: post.id
    });
    
    // READ: Búsqueda simple
    const foundUser = await User.findByPk(user.id);
    console.log('Usuario encontrado:', foundUser.toJSON());
    
    // READ: Búsqueda con condiciones
    const editors = await User.findAll({
      where: {
        role: 'editor',
        status: true
      },
      order: [['createdAt', 'DESC']],
      limit: 10
    });
    
    // READ: Búsqueda con operadores
    const { Op } = require('sequelize');
    const recentPosts = await Post.findAll({
      where: {
        status: 'published',
        publishedAt: {
          [Op.gte]: new Date(new Date() - 7 * 24 * 60 * 60 * 1000) // Últimos 7 días
        },
        title: {
          [Op.like]: '%Sequelize%' // Títulos que contengan "Sequelize"
        }
      }
    });
    
    // READ: Consultas con JOIN (eager loading)
    const userWithPosts = await User.findOne({
      where: { id: user.id },
      include: [
        {
          model: Post,
          where: { status: 'published' }, // Filtro en la relación
          required: false, // LEFT JOIN (false) vs INNER JOIN (true)
          include: [{ model: Comment }]  // Anidación de relaciones
        },
        {
          model: User,
          as: 'followers'
        }
      ]
    });
    
    // UPDATE: Actualizar un registro
    await user.update({ lastLogin: new Date() });
    
    // UPDATE: Actualización masiva
    await Post.update(
      { status: 'archived' },
      { 
        where: { 
          publishedAt: {
            [Op.lt]: new Date(new Date() - 365 * 24 * 60 * 60 * 1000) // Posts de hace más de un año
          }
        } 
      }
    );
    
    // DELETE: Eliminar un registro (soft delete si paranoid: true)
    await post.destroy();
    
    // DELETE: Eliminación masiva
    await Comment.destroy({
      where: {
        createdAt: {
          [Op.lt]: new Date(new Date() - 30 * 24 * 60 * 60 * 1000) // Comentarios de hace más de 30 días
        }
      }
    });
    
    // Transacciones
    const t = await sequelize.transaction();
    try {
      const newUser = await User.create({
        name: 'Carlos Rodríguez',
        email: 'carlos@ejemplo.com'
      }, { transaction: t });
      
      const newPost = await Post.create({
        title: 'Trabajando con transacciones',
        content: 'Las transacciones garantizan...',
        userId: newUser.id
      }, { transaction: t });
      
      await t.commit();
    } catch (error) {
      await t.rollback();
      throw error;
    }
    
    // Consultas raw SQL
    const [results, metadata] = await sequelize.query(
      "SELECT u.name, COUNT(p.id) as post_count FROM users u LEFT JOIN posts p ON u.id = p.user_id GROUP BY u.name",
      { type: sequelize.QueryTypes.SELECT }
    );
    
    console.log('Estadísticas de usuarios:', results);
    
  } catch (error) {
    console.error('Error en operaciones Sequelize:', error);
  }
}

ejemplosSequelize();
```

#### Mongoose (para MongoDB con Node.js)

Mongoose es un ODM (Object Document Mapper) para MongoDB y Node.js que proporciona una solución elegante y basada en esquemas para modelar los datos de la aplicación. A diferencia de los ORM tradicionales, Mongoose está diseñado específicamente para trabajar con las bases de datos orientadas a documentos de MongoDB.

##### Instalación y configuración inicial

```bash
# Instalación de Mongoose
npm install mongoose
```

##### Conexión a MongoDB

```javascript
const mongoose = require('mongoose');

// Conexión a MongoDB (con variables de entorno para mayor seguridad)
const mongoURI = process.env.MONGO_URI || 'mongodb://localhost:27017/mi_aplicacion';

// Opciones de conexión
const options = {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  serverSelectionTimeoutMS: 5000,
  // Si necesitas autenticación:
  // user: process.env.MONGO_USER,
  // pass: process.env.MONGO_PASSWORD,
};

// Conexión a la base de datos
mongoose.connect(mongoURI, options)
  .then(() => console.log('Conexión exitosa a MongoDB'))
  .catch(err => console.error('Error conectando a MongoDB:', err));

// Eventos de conexión
mongoose.connection.on('connected', () => console.log('Mongoose conectado'));
mongoose.connection.on('error', (err) => console.error('Error de conexión:', err));
mongoose.connection.on('disconnected', () => console.log('Mongoose desconectado'));

// Cerrar conexión cuando la aplicación termina
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  console.log('Conexión a MongoDB cerrada');
  process.exit(0);
});
```

##### Definición de esquemas y modelos

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

// Schema de usuario con validaciones y opciones avanzadas
const userSchema = new Schema({
  name: {
    type: String,
    required: [true, 'El nombre es obligatorio'],
    trim: true,
    minlength: [2, 'El nombre debe tener al menos 2 caracteres'],
    maxlength: [50, 'El nombre no puede exceder los 50 caracteres']
  },
  email: {
    type: String,
    required: [true, 'El email es obligatorio'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Por favor ingresa un email válido']
  },
  password: {
    type: String,
    required: [true, 'La contraseña es obligatoria'],
    minlength: [6, 'La contraseña debe tener al menos 6 caracteres'],
    select: false // No incluir en consultas por defecto (seguridad)
  },
  role: {
    type: String,
    enum: {
      values: ['user', 'admin', 'editor'],
      message: '{VALUE} no es un rol válido'
    },
    default: 'user'
  },
  avatar: String,
  bio: {
    type: String,
    maxlength: [500, 'La biografía no puede exceder los 500 caracteres']
  },
  isActive: {
    type: Boolean,
    default: true
  },
  lastLogin: Date,
  skills: [String], // Array de strings
  metadata: Schema.Types.Mixed, // Campo flexible para cualquier tipo de datos
  location: {
    country: String,
    city: String,
    coordinates: {
      lat: Number,
      lng: Number
    }
  }
}, {
  // Opciones del schema
  timestamps: true, // Crea campos createdAt y updatedAt automáticamente
  collection: 'usuarios', // Nombre personalizado para la colección (opcional)
  toJSON: { virtuals: true }, // Incluir virtuales cuando se convierte a JSON
  toObject: { virtuals: true } // Incluir virtuales cuando se convierte a objeto
});

// Índices para optimizar consultas
userSchema.index({ email: 1 }); // Índice simple
userSchema.index({ name: 1, role: 1 }); // Índice compuesto

// Campo virtual (calculado, no almacenado en la BD)
userSchema.virtual('fullName').get(function() {
  return `${this.name} - ${this.email}`;
});

// Método personalizado de instancia
userSchema.methods.isAdmin = function() {
  return this.role === 'admin';
};

// Método personalizado estático
userSchema.statics.findByRole = function(role) {
  return this.find({ role: role });
};

// Middleware (hooks) - pre y post
userSchema.pre('save', async function(next) {
  // Solo hash la contraseña si ha sido modificada o es nueva
  if (!this.isModified('password')) return next();
  
  try {
    // Ejemplo de cómo usar bcrypt (requiere npm install bcrypt)
    // const salt = await bcrypt.genSalt(10);
    // this.password = await bcrypt.hash(this.password, salt);
    this.password = `hashed_${this.password}`; // Solo simulación para el ejemplo
    next();
  } catch (error) {
    next(error);
  }
});

// Schema para posts
const postSchema = new Schema({
  title: {
    type: String,
    required: true,
    trim: true
  },
  slug: {
    type: String,
    unique: true,
    lowercase: true
  },
  content: {
    type: String,
    required: true
  },
  summary: String,
  status: {
    type: String,
    enum: ['draft', 'published', 'archived'],
    default: 'draft'
  },
  category: {
    type: Schema.Types.ObjectId,
    ref: 'Category'
  },
  tags: [String],
  author: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  featuredImage: String,
  viewCount: {
    type: Number,
    default: 0
  },
  comments: [
    {
      content: { type: String, required: true },
      author: { type: Schema.Types.ObjectId, ref: 'User' },
      createdAt: { type: Date, default: Date.now }
    }
  ], // Array de subdocumentos
  publishedAt: Date
}, { timestamps: true });

// Generar slug automáticamente a partir del título
postSchema.pre('save', function(next) {
  if (this.isNew || this.isModified('title')) {
    // En una aplicación real, usaríamos una librería como slugify
    this.slug = this.title
      .toLowerCase()
      .replace(/[^\w ]+/g, '')
      .replace(/ +/g, '-');
  }
  next();
});

// Schema para categorías
const categorySchema = new Schema({
  name: {
    type: String,
    required: true,
    unique: true
  },
  description: String,
  parentCategory: {
    type: Schema.Types.ObjectId,
    ref: 'Category'
  }
}, { timestamps: true });

// Crear modelos a partir de los schemas
const User = mongoose.model('User', userSchema);
const Post = mongoose.model('Post', postSchema);
const Category = mongoose.model('Category', categorySchema);
```

##### Operaciones CRUD y consultas avanzadas

```javascript
async function ejemplosMongoose() {
  try {
    // CREATE: Crear un usuario
    const user = await User.create({
      name: 'Laura Martínez',
      email: 'laura@ejemplo.com',
      password: 'contraseña123', // Se hará hash automáticamente por el hook pre-save
      role: 'editor',
      location: {
        country: 'España',
        city: 'Madrid'
      },
      skills: ['javascript', 'mongodb', 'express']
    });
    
    // Crear una categoría
    const category = await Category.create({
      name: 'Desarrollo Web',
      description: 'Artículos sobre desarrollo web frontend y backend'
    });
    
    // Crear un post asociado al usuario y categoría
    const post = await Post.create({
      title: 'Guía completa de MongoDB y Mongoose',
      content: 'MongoDB es una base de datos NoSQL orientada a documentos...',
      summary: 'Aprende a usar MongoDB y Mongoose en aplicaciones Node.js',
      author: user._id,
      category: category._id,
      tags: ['mongodb', 'mongoose', 'nodejs', 'database'],
      status: 'published',
      publishedAt: new Date()
    });
    
    // Añadir un comentario al post
    post.comments.push({
      content: 'Excelente artículo! Muy completo.',
      author: user._id
    });
    await post.save();
    
    // READ: Búsqueda simple
    const foundUser = await User.findById(user._id);
    console.log('Usuario encontrado:', foundUser);
    
    // READ: Búsqueda por criterios
    const editors = await User.find({ 
      role: 'editor',
      isActive: true
    }).select('name email role'); // Proyección (seleccionar campos específicos)
    
    // READ: Búsqueda con filtros avanzados
    const activePosts = await Post.find({
      status: 'published',
      publishedAt: { $gte: new Date(new Date().setDate(new Date().getDate() - 30)) }, // Últimos 30 días
      tags: { $in: ['mongodb', 'database'] } // Posts que contengan alguna de estas etiquetas
    })
    .sort({ publishedAt: -1 }) // Ordenar por fecha de publicación descendente
    .limit(10);
    
    // READ: Aggregation pipeline (operaciones complejas como GROUP BY)
    const categorySummary = await Post.aggregate([
      { $match: { status: 'published' } },
      { $group: {
          _id: '$category',
          count: { $sum: 1 },
          avgViews: { $avg: '$viewCount' },
          titles: { $push: '$title' }
        }
      },
      { $sort: { count: -1 } },
      { $limit: 5 }
    ]);
    
    // READ: Populate (similar a JOIN)
    const populatedPost = await Post.findById(post._id)
      .populate('author', 'name email') // Solo traer algunos campos del autor
      .populate('category')
      .populate('comments.author');
    
    console.log('Post con relaciones populadas:', JSON.stringify(populatedPost, null, 2));
    
    // READ: Encadenamiento de consultas
    const recentEditorPosts = await Post.find()
      .where('status').equals('published')
      .where('author').in(await User.find({ role: 'editor' }).select('_id'))
      .select('title publishedAt viewCount')
      .sort('-publishedAt')
      .limit(5)
      .lean(); // Retorna objetos JavaScript planos (más rápido pero sin métodos de documento)
    
    // UPDATE: Actualizar un documento
    await User.findByIdAndUpdate(
      user._id,
      { 
        $set: { lastLogin: new Date() },
        $push: { skills: 'mongoose' } // Añadir a un array
      },
      { new: true, runValidators: true } // Opciones: devolver documento actualizado y ejecutar validaciones
    );
    
    // UPDATE: Actualizar múltiples documentos
    await Post.updateMany(
      { viewCount: { $lt: 10 } },
      { $set: { status: 'archived' } }
    );
    
    // DELETE: Eliminar un documento
    await Post.findByIdAndDelete(post._id);
    
    // DELETE: Eliminar múltiples documentos
    await Post.deleteMany({ status: 'archived', publishedAt: { $lt: new Date(new Date().setFullYear(new Date().getFullYear() - 1)) } });
    
    // Transacción (requiere MongoDB 4.0+)
    const session = await mongoose.startSession();
    session.startTransaction();
    try {
      const newCategory = await Category.create([{
        name: 'Programación Funcional',
        description: 'Conceptos y aplicaciones de la programación funcional'
      }], { session });
      
      const newPost = await Post.create([{
        title: 'Introducción a la programación funcional',
        content: 'La programación funcional es un paradigma...',
        author: user._id,
        category: newCategory[0]._id,
        status: 'draft'
      }], { session });
      
      await session.commitTransaction();
      console.log('Transacción completada con éxito');
    } catch (error) {
      await session.abortTransaction();
      console.error('Error en la transacción:', error);
    } finally {
      session.endSession();
    }
    
  } catch (error) {
    console.error('Error en operaciones Mongoose:', error);
  }
}

ejemplosMongoose();
```

##### Plugins y extensiones

Mongoose puede extenderse con plugins para añadir funcionalidades comunes:

```javascript
// Ejemplo de un plugin para paginación (mongoose-paginate-v2)
const mongoosePaginate = require('mongoose-paginate-v2');
postSchema.plugin(mongoosePaginate);

// Uso de la paginación
const options = {
  page: 2,
  limit: 10,
  sort: { publishedAt: -1 },
  populate: ['author', 'category']
};

const result = await Post.paginate({ status: 'published' }, options);
// result contiene: docs, totalDocs, limit, page, totalPages, etc.
```

Mongoose es una herramienta poderosa para trabajar con MongoDB que facilita la modelación de datos, validaciones, relaciones y consultas complejas, manteniendo la flexibilidad y ventajas inherentes a las bases de datos NoSQL.

### 7.2.3 Migraciones y Gestión de Esquemas

Las migraciones son una forma de gestionar cambios incrementales y versionados en la estructura de las bases de datos. Funcionan como un sistema de control de versiones para la estructura de la base de datos, permitiendo a los equipos de desarrollo:

- **Versionar los cambios**: Cada modificación queda registrada cronológicamente.
- **Colaborar eficientemente**: Múltiples desarrolladores pueden aplicar cambios sin conflictos.
- **Automatizar despliegues**: Facilitan la configuración de entornos consistentes.
- **Realizar rollbacks controlados**: Permiten revertir cambios a versiones anteriores.

#### Migraciones en Sequelize

Sequelize proporciona un sistema de migraciones a través de su CLI (Command Line Interface):

##### Configuración inicial

```bash
# Instalar Sequelize CLI globalmente
npm install -g sequelize-cli

# O como dependencia de desarrollo
npm install --save-dev sequelize-cli

# Inicializar proyecto con Sequelize
npx sequelize-cli init
```

Esto creará la estructura de carpetas:
```
./config            # Configuración de la base de datos
./migrations        # Archivos de migración
./models            # Definiciones de modelos
./seeders           # Datos iniciales de prueba
```

##### Crear una migración

```bash
# Crear migración para una nueva tabla
npx sequelize-cli migration:generate --name create-users
```

##### Ejemplo de migración

```javascript
// Archivo: migrations/XXXXXX-create-users.js
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
        type: Sequelize.STRING(100),
        allowNull: false
      },
      email: {
        type: Sequelize.STRING(100),
        allowNull: false,
        unique: true
      },
      password: {
        type: Sequelize.STRING,
        allowNull: false
      },
      role: {
        type: Sequelize.ENUM('user', 'admin', 'editor'),
        defaultValue: 'user'
      },
      isActive: {
        type: Sequelize.BOOLEAN,
        defaultValue: true
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

    // Añadir un índice para optimizar búsquedas por email
    await queryInterface.addIndex('Users', ['email']);
  },

  down: async (queryInterface, Sequelize) => {
    // Revertir cambios (rollback)
    await queryInterface.dropTable('Users');
  }
};
```

##### Migración para modificar una tabla existente

```javascript
// migrations/YYYYYY-add-profile-to-users.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('Users', 'profileImage', {
      type: Sequelize.STRING,
      allowNull: true
    });

    await queryInterface.addColumn('Users', 'bio', {
      type: Sequelize.TEXT,
      allowNull: true
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn('Users', 'profileImage');
    await queryInterface.removeColumn('Users', 'bio');
  }
};
```

##### Ejecutar migraciones

```bash
# Aplicar todas las migraciones pendientes
npx sequelize-cli db:migrate

# Revertir la última migración
npx sequelize-cli db:migrate:undo

# Revertir todas las migraciones
npx sequelize-cli db:migrate:undo:all
```

#### Migraciones con MongoDB/Mongoose

A diferencia de las bases de datos relacionales, MongoDB es flexible con los esquemas, pero aún así se pueden gestionar cambios estructurales. Hay varias estrategias:

##### 1. Usando mongoose-migrate

```bash
npm install mongoose-migrate
```

```javascript
// migrations/20230601-add-status-to-users.js
export async function up() {
  // Obtiene acceso al modelo
  const User = this.connection.model('User');
  
  // Actualiza todos los documentos
  await User.updateMany(
    { status: { $exists: false } },
    { $set: { status: 'active' } }
  );
}

export async function down() {
  const User = this.connection.model('User');
  
  // Elimina el campo status de todos los documentos
  await User.updateMany(
    {},
    { $unset: { status: '' } }
  );
}
```

##### 2. Usando migrate-mongo

```bash
npm install -g migrate-mongo
migrate-mongo init
```

```javascript
// migrations/20230601-add-status-field.js
module.exports = {
  async up(db) {
    await db.collection('users').updateMany(
      { status: { $exists: false } },
      { $set: { status: 'active' } }
    );
  },

  async down(db) {
    await db.collection('users').updateMany(
      {},
      { $unset: { status: '' } }
    );
  }
};
```

#### Mejores prácticas para migraciones

1. **Idempotencia**: Las migraciones deben poder ejecutarse múltiples veces sin efectos secundarios.

2. **Migraciones atómicas**: Cada migración debe hacer un solo cambio lógico.

3. **Pruebas**: Probar las migraciones en entornos de desarrollo antes de aplicarlas en producción.

4. **Documentación**: Incluir comentarios que expliquen el propósito de cada migración.

5. **Metadatos**: Mantener una tabla de metadatos para rastrear qué migraciones se han aplicado.

6. **Seguridad**: Asegurar que las migraciones no comprometan datos existentes.

7. **Rendimiento**: Considerar el impacto en el rendimiento para tablas grandes, posiblemente programando migraciones en horarios de bajo tráfico.

#### Seeds (Datos iniciales)

Los seeds son datos iniciales que se cargan en la base de datos para pruebas o inicialización de ambientes.

##### Seeds con Sequelize

```bash
# Crear un seeder
npx sequelize-cli seed:generate --name demo-users
```

```javascript
// seeders/XXXXXX-demo-users.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.bulkInsert('Users', [{
      name: 'Admin User',
      email: 'admin@example.com',
      password: 'hashedPassword123', // En producción usaríamos un hash real
      role: 'admin',
      createdAt: new Date(),
      updatedAt: new Date()
    },
    {
      name: 'Demo User',
      email: 'demo@example.com',
      password: 'hashedPassword456',
      role: 'user',
      createdAt: new Date(),
      updatedAt: new Date()
    }]);
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.bulkDelete('Users', null, {});
  }
};
```

##### Seeds con MongoDB

```javascript
// scripts/seed-mongo.js
const mongoose = require('mongoose');
const User = require('../models/User');
const Post = require('../models/Post');

async function seedDatabase() {
  try {
    // Conectar a MongoDB
    await mongoose.connect('mongodb://localhost:27017/mi_aplicacion');
    
    // Limpiar colecciones existentes
    await User.deleteMany({});
    await Post.deleteMany({});
    
    // Crear usuarios
    const admin = await User.create({
      name: 'Admin',
      email: 'admin@ejemplo.com',
      password: 'adminpass',
      role: 'admin'
    });
    
    const author = await User.create({
      name: 'Autor Demo',
      email: 'autor@ejemplo.com',
      password: 'autorpass',
      role: 'editor'
    });
    
    // Crear posts
    await Post.create([
      {
        title: 'Primer post',
        content: 'Contenido del primer post...',
        author: author._id,
        status: 'published',
        publishedAt: new Date()
      },
      {
        title: 'Segundo post',
        content: 'Contenido del segundo post...',
        author: author._id,
        status: 'draft'
      }
    ]);
    
    console.log('Base de datos inicializada correctamente');
  } catch (error) {
    console.error('Error al inicializar la base de datos:', error);
  } finally {
    await mongoose.connection.close();
  }
}

seedDatabase();
```

### 7.2.4 Patrones de Acceso a Datos

Los patrones de acceso a datos son estructuras de diseño que ayudan a organizar la interacción con la base de datos, promoviendo la separación de responsabilidades, la reutilización y el mantenimiento de código. Estos patrones son fundamentales para aplicaciones robustas y escalables.

#### Patrón Repository

El patrón Repository actuáa como una capa intermedia entre la lógica de negocio y la capa de datos, encapsulando la lógica para acceder a la fuente de datos.

```javascript
// Implementación del patrón Repository con Sequelize
class UserRepository {
  constructor(userModel) {
    this.userModel = userModel;
  }

  async findAll(options = {}) {
    return await this.userModel.findAll(options);
  }

  async findById(id, options = {}) {
    return await this.userModel.findByPk(id, options);
  }

  async findByEmail(email) {
    return await this.userModel.findOne({ where: { email } });
  }

  async create(userData) {
    return await this.userModel.create(userData);
  }

  async update(id, userData) {
    const [updated] = await this.userModel.update(userData, {
      where: { id }
    });
    
    if (updated) {
      return this.findById(id);
    }
    
    throw new Error('Usuario no encontrado');
  }

  async delete(id) {
    const deleted = await this.userModel.destroy({ 
      where: { id } 
    });
    
    if (!deleted) {
      throw new Error('Usuario no encontrado');
    }
    
    return { id };
  }

  // Métodos adicionales específicos para usuarios
  async findByRole(role) {
    return await this.userModel.findAll({ 
      where: { role } 
    });
  }

  async updateLastLogin(id) {
    return await this.update(id, { lastLogin: new Date() });
  }
}

// Uso del repositorio
const userRepository = new UserRepository(User);

// En un controlador o servicio
async function getUserProfile(req, res) {
  try {
    const user = await userRepository.findById(req.params.id, {
      include: 'posts' // Cargar relaciones si es necesario
    });
    
    if (!user) {
      return res.status(404).json({ message: 'Usuario no encontrado' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
}
```

#### Patrón DAO (Data Access Object)

Similar al patrón Repository, el DAO aísla la lógica de acceso a datos del resto de la aplicación, pero típicamente está más orientado a operaciones CRUD específicas.

```javascript
// Implementación del patrón DAO con MongoDB/Mongoose
class PostDAO {
  constructor(postModel) {
    this.postModel = postModel;
  }

  async create(postData) {
    const post = new this.postModel(postData);
    return await post.save();
  }

  async findById(id) {
    return await this.postModel.findById(id).populate('author');
  }

  async findAll(filter = {}) {
    return await this.postModel.find(filter).sort({ createdAt: -1 });
  }

  async findPaginated(filter = {}, page = 1, limit = 10) {
    const skip = (page - 1) * limit;
    
    const posts = await this.postModel.find(filter)
      .sort({ createdAt: -1 })
      .skip(skip)
      .limit(limit)
      .populate('author', 'name email');
    
    const total = await this.postModel.countDocuments(filter);
    
    return {
      posts,
      totalPages: Math.ceil(total / limit),
      currentPage: page,
      total
    };
  }

  async update(id, postData) {
    return await this.postModel.findByIdAndUpdate(
      id, 
      postData, 
      { new: true, runValidators: true }
    );
  }

  async delete(id) {
    return await this.postModel.findByIdAndDelete(id);
  }

  // Métodos específicos para posts
  async findByAuthor(authorId) {
    return await this.postModel.find({ author: authorId });
  }

  async findByCategory(categoryId) {
    return await this.postModel.find({ category: categoryId });
  }

  async findByTags(tags) {
    return await this.postModel.find({ 
      tags: { $in: Array.isArray(tags) ? tags : [tags] } 
    });
  }

  async incrementViewCount(id) {
    return await this.postModel.findByIdAndUpdate(
      id,
      { $inc: { viewCount: 1 } },
      { new: true }
    );
  }
}
```

#### Patrón Service (Servicio)

El patrón Service encapsula la lógica de negocio y utiliza los repositorios/DAOs para acceder a los datos. Esto añade una capa adicional de abstracción.

```javascript
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async registerUser(userData) {
    // Validar si el email ya existe
    const existingUser = await this.userRepository.findByEmail(userData.email);
    if (existingUser) {
      throw new Error('El email ya está en uso');
    }

    // Hash de la contraseña
    const salt = await bcrypt.genSalt(10);
    const hashedPassword = await bcrypt.hash(userData.password, salt);

    // Crear usuario
    const user = await this.userRepository.create({
      ...userData,
      password: hashedPassword
    });

    // No devolver la contraseña en la respuesta
    const { password, ...userResponse } = user.toJSON();
    return userResponse;
  }

  async authenticateUser(email, password) {
    // Buscar usuario por email
    const user = await this.userRepository.findByEmail(email);
    if (!user) {
      throw new Error('Credenciales inválidas');
    }

    // Verificar contraseña
    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      throw new Error('Credenciales inválidas');
    }

    // Actualizar último login
    await this.userRepository.updateLastLogin(user.id);

    // Generar token JWT
    const token = jwt.sign(
      { id: user.id, email: user.email, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '1d' }
    );

    return { token, user: { id: user.id, email: user.email, name: user.name, role: user.role } };
  }

  // Otros métodos para la lógica de negocio
}
```

#### Patrón Unit of Work

El patrón Unit of Work coordina la realización de múltiples operaciones de base de datos dentro de una sola transacción.

```javascript
class UnitOfWork {
  constructor(sequelize) {
    this.sequelize = sequelize;
    this.transaction = null;
  }

  async begin() {
    this.transaction = await this.sequelize.transaction();
    return this.transaction;
  }

  async commit() {
    if (!this.transaction) throw new Error('No hay una transacción activa');
    await this.transaction.commit();
    this.transaction = null;
  }

  async rollback() {
    if (!this.transaction) throw new Error('No hay una transacción activa');
    await this.transaction.rollback();
    this.transaction = null;
  }

  // Método para ejecutar operaciones dentro de una transacción
  async executeInTransaction(callback) {
    try {
      await this.begin();
      const result = await callback(this.transaction);
      await this.commit();
      return result;
    } catch (error) {
      if (this.transaction) await this.rollback();
      throw error;
    }
  }
}

// Uso del Unit of Work
const unitOfWork = new UnitOfWork(sequelize);

async function createUserWithPosts(userData, postsData) {
  return await unitOfWork.executeInTransaction(async (transaction) => {
    const user = await userRepository.create(userData, { transaction });
    
    const posts = [];
    for (const postData of postsData) {
      const post = await postRepository.create(
        { ...postData, userId: user.id },
        { transaction }
      );
      posts.push(post);
    }
    
    return { user, posts };
  });
}
```

#### Patrón Active Record

En el patrón Active Record, el modelo tiene conocimiento sobre cómo persistirse a sí mismo en la base de datos, combinando lógica de negocio y acceso a datos. Muchos ORM como Sequelize implementan este patrón por defecto.

```javascript
// Ejemplo de Active Record con Sequelize
const User = sequelize.define('User', {
  name: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true
  },
  // Otros campos...
}, {
  // Métodos de instancia (métodos para cada registro)
  instanceMethods: {
    async getPosts() {
      return await Post.findAll({ where: { userId: this.id } });
    },
    async updateProfile(data) {
      Object.assign(this, data);
      return await this.save();
    }
  },
  // Métodos de clase (métodos para el modelo)
  classMethods: {
    async findByEmail(email) {
      return await this.findOne({ where: { email } });
    }
  }
});

// Uso de Active Record
const user = await User.findByPk(1);
user.name = 'Nuevo Nombre';
await user.save(); // El objeto conoce cómo persistirse

const posts = await user.getPosts(); // Método personalizado de instancia
```

#### Arquitectura en Capas

Una arquitectura completa de acceso a datos generalmente consiste en varias capas:

1. **Capa de Modelos**: Definición de entidades y esquemas (Sequelize Models, Mongoose Schemas).
2. **Capa de Acceso a Datos**: Repositorios o DAOs que encapsulan operaciones de base de datos.
3. **Capa de Servicios**: Lógica de negocio que utiliza los repositorios.
4. **Capa de Controladores**: Maneja las solicitudes HTTP y utiliza los servicios.

```javascript
// Estructura de carpetas para una arquitectura en capas
```
```
/src
  /models            # Definiciones de modelos
    User.js
    Post.js
  /repositories      # Implementaciones de repositorios
    userRepository.js
    postRepository.js
  /services          # Lógica de negocio
    userService.js
    postService.js
  /controllers       # Controladores para rutas API
    userController.js
    postController.js
  /routes            # Definición de rutas
    userRoutes.js
    postRoutes.js
  /config            # Configuración (database, etc.)
  /utils             # Utilidades generales
  /middleware        # Middleware personalizado
  app.js             # Punto de entrada principal
```

#### Ejemplo Integrado

Veamos un ejemplo completo que integra estos patrones:

```javascript
// controllers/userController.js
class UserController {
  constructor(userService) {
    this.userService = userService;
  }

  // Controlador para registro de usuarios
  async register(req, res) {
    try {
      const userData = req.body;
      const result = await this.userService.registerUser(userData);
      res.status(201).json(result);
    } catch (error) {
      res.status(400).json({ message: error.message });
    }
  }

  // Controlador para login
  async login(req, res) {
    try {
      const { email, password } = req.body;
      const result = await this.userService.authenticateUser(email, password);
      res.json(result);
    } catch (error) {
      res.status(401).json({ message: error.message });
    }
  }

  // Otros controladores...
}

// routes/userRoutes.js
const express = require('express');
const router = express.Router();

module.exports = (userController) => {
  router.post('/register', userController.register.bind(userController));
  router.post('/login', userController.login.bind(userController));
  // Otras rutas...
  
  return router;
};

// app.js (configuración)
const express = require('express');
const app = express();
const sequelize = require('./config/database');

// Modelos
const User = require('./models/User');
// Otros modelos...

// Repositorios
const UserRepository = require('./repositories/userRepository');
const userRepository = new UserRepository(User);

// Servicios
const UserService = require('./services/userService');
const userService = new UserService(userRepository);

// Controladores
const UserController = require('./controllers/userController');
const userController = new UserController(userService);

// Rutas
app.use('/api/users', require('./routes/userRoutes')(userController));

// Iniciar servidor
sequelize.sync().then(() => {
  app.listen(3000, () => {
    console.log('Servidor en ejecución en el puerto 3000');
  });
});
```

Utilizar patrones de acceso a datos adecuados proporciona múltiples beneficios:

- **Separación de responsabilidades**: Cada componente tiene un rol claramente definido.
- **Pruebas unitarias más fáciles**: Se pueden probar componentes de forma aislada.
- **Mantenibilidad mejorada**: Los cambios en una capa no afectan necesariamente a otras.
- **Reutilización de código**: Los repositorios y servicios pueden reutilizarse en diferentes controladores.
- **Flexibilidad para cambios**: La fuente de datos puede cambiarse sin afectar a la lógica de negocio.

## 7.3 Autenticación y Autorización

### 7.3.1 Autenticación con JWT

JSON Web Tokens (JWT) es un estándar abierto (RFC 7519) que define un método compacto y autónomo para transmitir información de forma segura entre partes como un objeto JSON. Esta información puede ser verificada y confiable porque está firmada digitalmente.

#### Estructura de un JWT

Un JWT consiste en tres partes separadas por puntos:

```
xxxxxx.yyyyyy.zzzzzz
```

- **Header**: Contiene el tipo de token y el algoritmo de firma (ej: HMAC SHA256, RSA).
- **Payload**: Contiene las claims (afirmaciones) sobre una entidad (usuario) y metadata.
- **Signature**: Firma que verifica la integridad del token.

#### Claims (Afirmaciones) comunes:

- **iss**: Emisor del JWT
- **sub**: Sujeto del token (normalmente el ID de usuario)
- **exp**: Tiempo de expiración
- **iat**: Tiempo de emisión
- **nbf**: Tiempo antes del cual el token no debe ser aceptado
- **jti**: Identificador único del token
- **aud**: Audiencia del token

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
