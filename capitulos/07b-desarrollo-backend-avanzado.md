# Capítulo 7: Desarrollo Backend (Continuación)

## 7.4 APIs Avanzadas

### 7.4.1 GraphQL en Profundidad

GraphQL es un lenguaje de consulta y un tiempo de ejecución que permite a los clientes solicitar exactamente los datos que necesitan.

```javascript
// Configuración básica de GraphQL con Apollo Server
const { ApolloServer, gql } = require('apollo-server');

// Definición del esquema
const typeDefs = gql`
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
  }

  type Query {
    users: [User!]!
    user(id: ID!): User
    posts: [Post!]!
    post(id: ID!): Post
  }

  type Mutation {
    createUser(name: String!, email: String!): User!
    createPost(title: String!, content: String, authorId: ID!): Post!
  }
`;

// Datos simulados
const users = [
  { id: '1', name: 'John', email: 'john@example.com' },
  { id: '2', name: 'Jane', email: 'jane@example.com' },
];

const posts = [
  { id: '1', title: 'GraphQL Intro', content: 'Content...', authorId: '1', createdAt: '2023-01-15' },
  { id: '2', title: 'Apollo Server', content: 'Content...', authorId: '1', createdAt: '2023-02-20' },
  { id: '3', title: 'REST vs GraphQL', content: 'Content...', authorId: '2', createdAt: '2023-03-10' },
];

// Resolvers
const resolvers = {
  Query: {
    users: () => users,
    user: (_, { id }) => users.find(user => user.id === id),
    posts: () => posts,
    post: (_, { id }) => posts.find(post => post.id === id),
  },
  User: {
    posts: (parent) => posts.filter(post => post.authorId === parent.id),
  },
  Post: {
    author: (parent) => users.find(user => user.id === parent.authorId),
  },
  Mutation: {
    createUser: (_, { name, email }) => {
      const newUser = { 
        id: String(users.length + 1), 
        name, 
        email 
      };
      users.push(newUser);
      return newUser;
    },
    createPost: (_, { title, content, authorId }) => {
      const newPost = {
        id: String(posts.length + 1),
        title,
        content,
        authorId,
        createdAt: new Date().toISOString().split('T')[0]
      };
      posts.push(newPost);
      return newPost;
    }
  }
};

// Crear y iniciar servidor
const server = new ApolloServer({ typeDefs, resolvers });

server.listen().then(({ url }) => {
  console.log(`🚀 Servidor GraphQL listo en ${url}`);
});
```

### 7.4.2 WebSockets y Comunicación en Tiempo Real

Los WebSockets permiten una comunicación bidireccional entre cliente y servidor.

```javascript
// Servidor WebSocket con Socket.io
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.get('/', (req, res) => {
  res.sendFile(__dirname + '/index.html');
});

// Manejo de conexiones WebSocket
io.on('connection', (socket) => {
  console.log('Un usuario se ha conectado');
  
  // Enviar mensaje al cliente que se conectó
  socket.emit('welcome', { message: 'Bienvenido al chat!' });
  
  // Notificar a todos los demás clientes
  socket.broadcast.emit('userJoined', { message: 'Un nuevo usuario se ha unido' });
  
  // Manejar mensajes de chat
  socket.on('chatMessage', (msg) => {
    console.log('Mensaje recibido:', msg);
    // Reenviar mensaje a todos los clientes
    io.emit('chatMessage', msg);
  });
  
  // Manejar desconexiones
  socket.on('disconnect', () => {
    console.log('Un usuario se ha desconectado');
    io.emit('userLeft', { message: 'Un usuario ha salido del chat' });
  });
});

server.listen(3000, () => {
  console.log('Servidor escuchando en http://localhost:3000');
});
```

Cliente HTML básico para WebSockets:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Chat en Tiempo Real</title>
  <style>
    #messages { list-style-type: none; margin: 0; padding: 0; }
    #messages li { padding: 5px 10px; }
    #messages li:nth-child(odd) { background: #eee; }
  </style>
</head>
<body>
  <ul id="messages"></ul>
  <form id="form">
    <input id="input" autocomplete="off" /><button>Enviar</button>
  </form>
  
  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io();
    const form = document.getElementById('form');
    const input = document.getElementById('input');
    const messages = document.getElementById('messages');
    
    // Mostrar mensajes de sistema
    socket.on('welcome', (data) => {
      const item = document.createElement('li');
      item.textContent = data.message;
      item.style.color = 'blue';
      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
    });
    
    socket.on('userJoined', (data) => {
      const item = document.createElement('li');
      item.textContent = data.message;
      item.style.color = 'green';
      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
    });
    
    socket.on('userLeft', (data) => {
      const item = document.createElement('li');
      item.textContent = data.message;
      item.style.color = 'red';
      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
    });
    
    // Manejar envío de formulario
    form.addEventListener('submit', (e) => {
      e.preventDefault();
      if (input.value) {
        // Enviar mensaje al servidor
        socket.emit('chatMessage', {
          text: input.value,
          timestamp: new Date().toISOString()
        });
        input.value = '';
      }
    });
    
    // Recibir mensajes de chat
    socket.on('chatMessage', (msg) => {
      const item = document.createElement('li');
      item.textContent = `${new Date(msg.timestamp).toLocaleTimeString()}: ${msg.text}`;
      messages.appendChild(item);
      window.scrollTo(0, document.body.scrollHeight);
    });
  </script>
</body>
</html>
```

## 7.5 Arquitecturas de Backend Escalables

### 7.5.1 Microservicios

La arquitectura de microservicios descompone una aplicación en servicios pequeños e independientes.

```javascript
// Microservicio de Usuarios (users-service.js)
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3001;

app.use(express.json());

// Base de datos simulada
const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
];

// Endpoints
app.get('/api/users', (req, res) => {
  res.json(users);
});

app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ message: 'Usuario no encontrado' });
  res.json(user);
});

app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  const newUser = { id: users.length + 1, name, email };
  users.push(newUser);
  res.status(201).json(newUser);
});

app.listen(PORT, () => {
  console.log(`Servicio de usuarios ejecutándose en puerto ${PORT}`);
});

// Microservicio de Productos (products-service.js)
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3002;

app.use(express.json());

// Base de datos simulada
const products = [
  { id: 1, name: 'Laptop', price: 999.99 },
  { id: 2, name: 'Smartphone', price: 699.99 },
];

// Endpoints
app.get('/api/products', (req, res) => {
  res.json(products);
});

app.get('/api/products/:id', (req, res) => {
  const product = products.find(p => p.id === parseInt(req.params.id));
  if (!product) return res.status(404).json({ message: 'Producto no encontrado' });
  res.json(product);
});

app.post('/api/products', (req, res) => {
  const { name, price } = req.body;
  const newProduct = { id: products.length + 1, name, price };
  products.push(newProduct);
  res.status(201).json(newProduct);
});

app.listen(PORT, () => {
  console.log(`Servicio de productos ejecutándose en puerto ${PORT}`);
});
```

### 7.5.2 API Gateway

Un API Gateway actúa como punto de entrada único para múltiples microservicios.

```javascript
// API Gateway (gateway.js)
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const app = express();
const PORT = process.env.PORT || 3000;

// Configuración de rutas y sus proxies
app.use('/api/users', createProxyMiddleware({
  target: 'http://localhost:3001',
  changeOrigin: true,
  pathRewrite: {
    '^/api/users': '/api/users'
  }
}));

app.use('/api/products', createProxyMiddleware({
  target: 'http://localhost:3002',
  changeOrigin: true,
  pathRewrite: {
    '^/api/products': '/api/products'
  }
}));

// Middleware para logging de solicitudes
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.originalUrl}`);
  next();
});

app.get('/', (req, res) => {
  res.send('API Gateway');
});

app.listen(PORT, () => {
  console.log(`API Gateway ejecutándose en puerto ${PORT}`);
});
```

### 7.5.3 Colas y Sistemas de Mensajería

Las colas permiten una comunicación asíncrona entre servicios.

```javascript
// Ejemplo con RabbitMQ
const amqp = require('amqplib');

// Productor (servicio que envía mensajes)
async function producer() {
  try {
    // Conectar al servidor RabbitMQ
    const connection = await amqp.connect('amqp://localhost');
    const channel = await connection.createChannel();
    
    // Declarar la cola
    const queue = 'tasks_queue';
    await channel.assertQueue(queue, { durable: true });
    
    // Enviar mensajes
    for (let i = 1; i <= 5; i++) {
      const message = { 
        id: i, 
        task: `Tarea ${i}`,
        timestamp: new Date().toISOString()
      };
      
      channel.sendToQueue(
        queue,
        Buffer.from(JSON.stringify(message)),
        { persistent: true }
      );
      
      console.log(`[x] Enviado ${JSON.stringify(message)}`);
      
      // Esperar un segundo entre mensajes
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
    
    // Cerrar la conexión
    setTimeout(() => {
      connection.close();
    }, 500);
  } catch (error) {
    console.error('Error en el productor:', error);
  }
}

// Consumidor (servicio que procesa mensajes)
async function consumer() {
  try {
    // Conectar al servidor RabbitMQ
    const connection = await amqp.connect('amqp://localhost');
    const channel = await connection.createChannel();
    
    // Declarar la cola
    const queue = 'tasks_queue';
    await channel.assertQueue(queue, { durable: true });
    
    // Configurar prefetch para no recibir más de un mensaje a la vez
    channel.prefetch(1);
    
    console.log('[*] Esperando mensajes. Para salir presiona CTRL+C');
    
    // Consumir mensajes
    channel.consume(queue, (msg) => {
      if (msg !== null) {
        const content = JSON.parse(msg.content.toString());
        console.log(`[x] Recibido ${JSON.stringify(content)}`);
        
        // Simular procesamiento
        setTimeout(() => {
          console.log(`[x] Procesado ${content.task}`);
          
          // Confirmar procesamiento
          channel.ack(msg);
        }, 2000);
      }
    }, { noAck: false });
  } catch (error) {
    console.error('Error en el consumidor:', error);
  }
}

// Ejecutar productor y consumidor
producer();
consumer();
```

## 7.6 Almacenamiento y Caché

### 7.6.1 Sistemas de Caché

El caché mejora el rendimiento almacenando temporalmente datos a los que se accede con frecuencia.

```javascript
// Ejemplo con Redis como caché
const express = require('express');
const redis = require('redis');
const { promisify } = require('util');
const app = express();
const PORT = process.env.PORT || 3000;

// Crear cliente Redis
const redisClient = redis.createClient({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379
});

// Convertir métodos de callback a promesas
const getAsync = promisify(redisClient.get).bind(redisClient);
const setAsync = promisify(redisClient.set).bind(redisClient);

// Manejar errores de Redis
redisClient.on('error', (err) => {
  console.error('Error en Redis:', err);
});

// Middleware para parsear JSON
app.use(express.json());

// Función simulada para obtener datos (ej. desde una DB)
async function fetchDataFromDatabase(id) {
  // Simular latencia de base de datos
  await new Promise(resolve => setTimeout(resolve, 1000));
  
  return {
    id,
    name: `Item ${id}`,
    data: `Datos del item ${id}`,
    timestamp: new Date().toISOString()
  };
}

// Endpoint con caché
app.get('/api/items/:id', async (req, res) => {
  const { id } = req.params;
  const cacheKey = `item:${id}`;
  
  try {
    // Intentar obtener datos del caché
    const cachedData = await getAsync(cacheKey);
    
    if (cachedData) {
      console.log('Datos obtenidos desde el caché');
      return res.json(JSON.parse(cachedData));
    }
    
    // Si no está en caché, obtener de la base de datos
    console.log('Obteniendo datos desde la base de datos');
    const data = await fetchDataFromDatabase(id);
    
    // Guardar en caché (expiración de 60 segundos)
    await setAsync(cacheKey, JSON.stringify(data), 'EX', 60);
    
    return res.json(data);
  } catch (error) {
    console.error('Error al obtener datos:', error);
    res.status(500).json({ error: 'Error al obtener datos' });
  }
});

// Iniciar servidor
app.listen(PORT, () => {
  console.log(`Servidor ejecutándose en puerto ${PORT}`);
});
```

### 7.6.2 Almacenamiento de Archivos

Gestión de archivos en servidores y servicios de almacenamiento en la nube.

```javascript
// Ejemplo con Multer y AWS S3
const express = require('express');
const multer = require('multer');
const { S3Client, PutObjectCommand, GetObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
const app = express();
const PORT = process.env.PORT || 3000;

// Configuración de AWS S3
const s3Client = new S3Client({
  region: process.env.AWS_REGION || 'us-east-1',
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  }
});

const BUCKET_NAME = process.env.AWS_BUCKET_NAME || 'my-app-bucket';

// Configuración de Multer para almacenamiento temporal
const upload = multer({
  storage: multer.memoryStorage(),
  limits: {
    fileSize: 5 * 1024 * 1024 // Límite de 5MB
  }
});

app.use(express.json());

// Endpoint para subir archivo
app.post('/api/upload', upload.single('file'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No se proporcionó ningún archivo' });
    }
    
    // Crear nombre único para el archivo
    const fileName = `${Date.now()}-${req.file.originalname}`;
    
    // Configurar parámetros para S3
    const params = {
      Bucket: BUCKET_NAME,
      Key: fileName,
      Body: req.file.buffer,
      ContentType: req.file.mimetype
    };
    
    // Subir archivo a S3
    await s3Client.send(new PutObjectCommand(params));
    
    // Generar URL firmada para acceso
    const getCommand = new GetObjectCommand({
      Bucket: BUCKET_NAME,
      Key: fileName
    });
    
    const signedUrl = await getSignedUrl(s3Client, getCommand, {
      expiresIn: 3600 // URL válida por 1 hora
    });
    
    // Responder con información del archivo
    res.status(201).json({
      message: 'Archivo subido exitosamente',
      file: {
        name: req.file.originalname,
        type: req.file.mimetype,
        size: req.file.size,
        url: signedUrl
      }
    });
  } catch (error) {
    console.error('Error al subir archivo:', error);
    res.status(500).json({ error: 'Error al subir el archivo' });
  }
});

app.listen(PORT, () => {
  console.log(`Servidor ejecutándose en puerto ${PORT}`);
});
```

## 7.7 Contenedores y Despliegue

### 7.7.1 Docker y Contenerización

Docker permite empaquetar aplicaciones y sus dependencias en contenedores.

```dockerfile
# Dockerfile para una aplicación Node.js
FROM node:16-alpine

# Crear directorio de la aplicación
WORKDIR /app

# Copiar archivos de configuración y dependencias
COPY package*.json ./
RUN npm ci --only=production

# Copiar el código fuente
COPY . .

# Exponer puerto
EXPOSE 3000

# Comando para iniciar la aplicación
CMD ["node", "server.js"]
```

Script para construir y ejecutar el contenedor:

```bash
# Construir la imagen
docker build -t myapp .

# Ejecutar el contenedor
docker run -p 3000:3000 -d --name myapp-container myapp
```

### 7.7.2 Docker Compose para Múltiples Servicios

Docker Compose permite definir y ejecutar aplicaciones multi-contenedor.

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Aplicación principal
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGO_URI=mongodb://mongo:27017/myapp
      - REDIS_HOST=redis
    depends_on:
      - mongo
      - redis

  # Base de datos MongoDB
  mongo:
    image: mongo:4.4
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  # Caché con Redis
  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

volumes:
  mongo-data:
  redis-data:
```

Comandos para gestionar la aplicación multi-contenedor:

```bash
# Iniciar todos los servicios
docker-compose up -d

# Ver logs de un servicio específico
docker-compose logs app

# Detener todos los servicios
docker-compose down

# Detener y eliminar volúmenes
docker-compose down -v
```

### 7.7.3 CI/CD para Backend

Integración y despliegue continuos para aplicaciones backend.

```yaml
# .github/workflows/ci-cd.yml (GitHub Actions)
name: Node.js CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [14.x, 16.x]
        
    steps:
    - uses: actions/checkout@v2
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run linter
      run: npm run lint
      
    - name: Run tests
      run: npm test
      
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Use Node.js 16.x
      uses: actions/setup-node@v2
      with:
        node-version: 16.x
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Build application
      run: npm run build
      
    - name: Deploy to production
      uses: amondnet/vercel-action@v20
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
        vercel-args: '--prod'
```

## 7.8 Monitoreo y Logging

### 7.8.1 Logging Estructurado

El logging estructurado mejora la capacidad de análisis y búsqueda de logs.

```javascript
// Ejemplo con Winston
const winston = require('winston');
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Configuración del logger
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'user-service' },
  transports: [
    // Escribir logs a la consola
    new winston.transports.Console(),
    // Escribir logs a archivos
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Middleware para logging de solicitudes HTTP
app.use((req, res, next) => {
  const start = Date.now();
  
  // Una vez que se envía la respuesta
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    logger.info('HTTP Request', {
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      duration,
      ip: req.ip,
      userAgent: req.get('User-Agent')
    });
  });
  
  next();
});

// Middleware para errores
app.use((err, req, res, next) => {
  logger.error('Application error', {
    error: {
      message: err.message,
      stack: err.stack
    },
    request: {
      method: req.method,
      url: req.originalUrl,
      ip: req.ip
    }
  });
  
  res.status(500).json({ error: 'Error interno del servidor' });
});

// Rutas de ejemplo
app.get('/', (req, res) => {
  logger.debug('Procesando solicitud a la ruta principal');
  res.send('¡Hola mundo!');
});

app.get('/error', (req, res, next) => {
  try {
    // Simular un error
    throw new Error('Ejemplo de error');
  } catch (error) {
    next(error);
  }
});

app.listen(PORT, () => {
  logger.info(`Servidor iniciado en puerto ${PORT}`);
});
```
