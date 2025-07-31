# 6.2 APIs RESTful y GraphQL

En el backend moderno, las APIs son el punto de comunicación principal entre diferentes sistemas. Este capítulo explora las APIs RESTful y GraphQL, dos paradigmas dominantes para el diseño de interfaces de programación de aplicaciones.

## 6.2.1 Principios de REST

REST (Representational State Transfer) es un estilo arquitectónico para sistemas distribuidos que ha dominado el diseño de APIs web durante años.

### Principios clave de REST:

1. **Arquitectura Cliente-Servidor**: Separación clara de responsabilidades.
2. **Sin Estado (Stateless)**: Cada solicitud contiene toda la información necesaria.
3. **Cacheable**: Las respuestas deben indicar si se pueden almacenar en caché.
4. **Interfaz Uniforme**: Recursos identificados en las solicitudes, manipulación a través de representaciones.
5. **Sistema de Capas**: El cliente no sabe si está conectado directamente al servidor final.

### Niveles de Madurez REST (Modelo de Richardson)

| Nivel | Descripción | Características |
|-------|-------------|----------------|
| 0 | Pantano de POX (Plain Old XML) | Una URL, un método HTTP (generalmente POST) |
| 1 | Recursos | Múltiples URLs (recursos), un método HTTP |
| 2 | Verbos HTTP | Uso adecuado de métodos HTTP (GET, POST, PUT, DELETE) |
| 3 | HATEOAS | Hipermedia como motor del estado de la aplicación |

## 6.2.2 Diseño de APIs RESTful

Una API RESTful bien diseñada es fundamental para facilitar la integración y el uso por parte de los clientes.

### Mejores Prácticas

#### 1. Usar Sustantivos para Recursos (no Verbos)

```
✅ Bueno:
GET /users
GET /users/123

❌ Malo:
GET /getUsers
GET /getUserById/123
```

#### 2. Colecciones en Plural, Entidades en Singular

```
✅ Bueno:
GET /users (colección)
GET /users/123 (elemento específico)

❌ Malo:
GET /user (colección)
GET /users/user/123 (elemento específico)
```

#### 3. Uso Adecuado de los Métodos HTTP

- **GET**: Recuperar recursos (no debe tener efectos secundarios)
- **POST**: Crear nuevos recursos
- **PUT**: Actualizar recursos existentes (actualización completa)
- **PATCH**: Actualización parcial de recursos
- **DELETE**: Eliminar recursos

#### 4. Códigos de Estado HTTP Correctos

| Código | Significado | Uso Típico |
|--------|-------------|------------|
| 200 | OK | Operación exitosa (GET, PUT, PATCH) |
| 201 | Created | Recurso creado exitosamente (POST) |
| 204 | No Content | Operación exitosa sin contenido de retorno (DELETE) |
| 400 | Bad Request | La solicitud no se pudo procesar (error de validación) |
| 401 | Unauthorized | Autenticación requerida |
| 403 | Forbidden | Cliente autenticado pero sin permisos |
| 404 | Not Found | Recurso no encontrado |
| 409 | Conflict | Conflicto con el estado actual del recurso |
| 500 | Internal Server Error | Error inesperado en el servidor |

#### 5. Versionado de API

```
✅ Opciones:
URL: /api/v1/users
Encabezado: Accept: application/vnd.company.v1+json
Parámetro de consulta: /api/users?version=1
```

### Ejemplo de Implementación RESTful

```typescript
// Ejemplo con Express (Node.js/TypeScript)
import express from 'express';
import { Router } from 'express';

const router: Router = express.Router();

// Colección de recursos (Listar y Crear)
router.get('/products', (req, res) => {
  // Parámetros de consulta para filtrado y paginación
  const { category, limit = 10, page = 1 } = req.query;
  
  // En una aplicación real, estos datos vendrían de una base de datos
  const products = [
    { id: 1, name: 'Product A', category: 'electronics', price: 99.99 },
    { id: 2, name: 'Product B', category: 'books', price: 19.99 }
  ];
  
  // Filtrado condicional
  const filteredProducts = category 
    ? products.filter(p => p.category === category) 
    : products;
  
  // Paginación
  const startIndex = (Number(page) - 1) * Number(limit);
  const paginatedProducts = filteredProducts.slice(startIndex, startIndex + Number(limit));
  
  // Incluir metadatos de paginación
  res.json({
    data: paginatedProducts,
    meta: {
      total: filteredProducts.length,
      page: Number(page),
      limit: Number(limit),
      pages: Math.ceil(filteredProducts.length / Number(limit))
    }
  });
});

router.post('/products', (req, res) => {
  const { name, category, price } = req.body;
  
  // Validación básica
  if (!name || !category || price === undefined) {
    return res.status(400).json({ 
      error: 'Validation failed', 
      details: 'Name, category, and price are required' 
    });
  }
  
  // En una aplicación real, aquí guardaríamos en la base de datos
  const newProduct = {
    id: Date.now(),  // Simplificado; en producción usaríamos un mejor generador de IDs
    name,
    category,
    price
  };
  
  // Respuesta con código 201 Created y ubicación del nuevo recurso
  res.status(201)
     .location(`/api/products/${newProduct.id}`)
     .json(newProduct);
});

// Recurso específico (Obtener, Actualizar, Eliminar)
router.get('/products/:id', (req, res) => {
  const productId = parseInt(req.params.id);
  
  // En una aplicación real, buscaríamos en la base de datos
  const product = { id: productId, name: 'Product A', category: 'electronics', price: 99.99 };
  
  if (!product) {
    return res.status(404).json({ error: 'Product not found' });
  }
  
  res.json(product);
});

router.put('/products/:id', (req, res) => {
  const productId = parseInt(req.params.id);
  const { name, category, price } = req.body;
  
  // Validación para actualización completa
  if (!name || !category || price === undefined) {
    return res.status(400).json({ 
      error: 'Validation failed', 
      details: 'For PUT requests, name, category, and price are required' 
    });
  }
  
  // En una aplicación real, aquí actualizaríamos en la base de datos
  const updatedProduct = {
    id: productId,
    name,
    category,
    price
  };
  
  res.json(updatedProduct);
});

router.patch('/products/:id', (req, res) => {
  const productId = parseInt(req.params.id);
  
  // Simulación: obtener producto existente (en realidad vendría de la BD)
  let product = { id: productId, name: 'Product A', category: 'electronics', price: 99.99 };
  
  // Actualización parcial: solo actualiza los campos proporcionados
  const updates = req.body;
  product = { ...product, ...updates };
  
  res.json(product);
});

router.delete('/products/:id', (req, res) => {
  const productId = parseInt(req.params.id);
  
  // En una aplicación real, aquí eliminaríamos de la base de datos
  
  // Respuesta sin contenido para operaciones DELETE exitosas
  res.status(204).end();
});

export default router;
```

## 6.2.3 Introducción a GraphQL

GraphQL es un lenguaje de consulta y manipulación de datos para APIs, que permite a los clientes solicitar exactamente los datos que necesitan.

### Características Principales de GraphQL

1. **Consultas Precisas**: Los clientes especifican exactamente qué campos necesitan.
2. **Una Sola Endpoint**: A diferencia de REST con múltiples endpoints.
3. **Tipado Fuerte**: Sistema de tipos que describe los datos disponibles.
4. **Introspección**: Las APIs pueden consultarse a sí mismas para documentación.

### Comparación: REST vs GraphQL

| Aspecto | REST | GraphQL |
|---------|------|---------|
| Endpoints | Múltiples | Una sola |
| Formato de datos | Servidor decide | Cliente decide |
| Versioning | Explícito (v1, v2, etc.) | Evolución continua |
| Over-fetching | Común | Eliminado |
| Under-fetching | Común (múltiples requests) | Eliminado |
| Caching | Simple (nivel HTTP) | Complejo (nivel cliente) |
| Implementación | Sencilla | Requiere más configuración |

## 6.2.4 Implementación de GraphQL

### Definición del Esquema

```graphql
# Schema GraphQL
type Product {
  id: ID!
  name: String!
  description: String
  price: Float!
  category: Category!
  inStock: Boolean!
  reviews: [Review!]
}

type Category {
  id: ID!
  name: String!
  products: [Product!]!
}

type Review {
  id: ID!
  text: String!
  rating: Int!
  author: User!
  product: Product!
}

type User {
  id: ID!
  username: String!
  email: String!
  reviews: [Review!]!
}

type Query {
  product(id: ID!): Product
  products(category: ID, limit: Int, offset: Int): [Product!]!
  categories: [Category!]!
  user(id: ID!): User
}

type Mutation {
  createProduct(name: String!, description: String, price: Float!, categoryId: ID!): Product!
  updateProduct(id: ID!, name: String, description: String, price: Float): Product!
  deleteProduct(id: ID!): Boolean!
  createReview(productId: ID!, text: String!, rating: Int!): Review!
}
```

### Implementación con Apollo Server (Node.js)

```typescript
// Ejemplo con Apollo Server y Express (Node.js/TypeScript)
import { ApolloServer } from 'apollo-server-express';
import express from 'express';
import { typeDefs } from './schema';

// Resolvers: implementan la lógica para obtener los datos
const resolvers = {
  Query: {
    product: async (_, { id }) => {
      // En una app real, buscaríamos en la base de datos
      return { id, name: 'Sample Product', price: 99.99 };
    },
    products: async (_, { category, limit = 10, offset = 0 }) => {
      // Lógica para buscar productos, posiblemente filtrar por categoría
      // y aplicar paginación (limit/offset)
      return [
        { id: '1', name: 'Product 1', price: 19.99 },
        { id: '2', name: 'Product 2', price: 29.99 },
      ];
    },
  },
  Mutation: {
    createProduct: async (_, { name, description, price, categoryId }) => {
      // Validación y lógica para crear un producto
      const newProduct = {
        id: Date.now().toString(),
        name,
        description,
        price,
        categoryId,
        inStock: true
      };
      
      // En una app real, guardaríamos en la base de datos
      
      return newProduct;
    },
  },
  // Resolvers para relaciones entre tipos
  Product: {
    category: async (parent) => {
      // Buscar la categoría asociada al producto
      return { id: parent.categoryId, name: 'Sample Category' };
    },
    reviews: async (parent) => {
      // Buscar reseñas asociadas al producto
      return [
        { id: '101', text: 'Great product!', rating: 5 },
      ];
    },
  },
};

async function startServer() {
  const app = express();
  
  // Crear instancia de Apollo Server
  const server = new ApolloServer({
    typeDefs,
    resolvers,
    context: ({ req }) => {
      // Aquí podrías extraer el token de auth y proporcionar
      // el contexto para todos los resolvers
      const user = getUserFromToken(req.headers.authorization);
      return { user };
    },
  });
  
  await server.start();
  
  // Aplicar middleware a Express
  server.applyMiddleware({ app });
  
  // Iniciar el servidor
  app.listen({ port: 4000 }, () =>
    console.log(`🚀 Server ready at http://localhost:4000${server.graphqlPath}`)
  );
}

startServer();

// Función auxiliar (simplificada) para extraer usuario del token
function getUserFromToken(authHeader) {
  if (!authHeader) return null;
  // En una implementación real verificaríamos y decodificaríamos el JWT
  return { id: '1', username: 'test_user' };
}
```

### Ejemplo de Consultas GraphQL

#### Consulta Básica

```graphql
# Consulta para obtener un producto específico con datos relacionados
query GetProduct {
  product(id: "1") {
    id
    name
    price
    category {
      id
      name
    }
    reviews {
      id
      rating
      text
      author {
        username
      }
    }
  }
}
```

#### Consulta con Variables

```graphql
# Consulta parametrizada
query GetFilteredProducts($categoryId: ID, $limit: Int) {
  products(category: $categoryId, limit: $limit) {
    id
    name
    price
    inStock
  }
}

# Variables (enviadas separadamente)
{
  "categoryId": "5",
  "limit": 5
}
```

#### Mutación

```graphql
# Creación de un nuevo producto
mutation CreateNewProduct {
  createProduct(
    name: "New Gadget",
    description: "This is a fantastic new gadget",
    price: 149.99,
    categoryId: "5"
  ) {
    id
    name
    price
  }
}
```

## 6.2.5 ¿Cuándo Usar REST vs GraphQL?

### Escenarios Favorables para REST

- Aplicaciones con pocos tipos de recursos y estructuras de datos simples
- APIs públicas que serán consumidas por muchos clientes diferentes
- Cuando el caching HTTP es importante
- Cuando quieres aprovechar la infraestructura web existente (proxies, CDNs)
- Servicios principalmente CRUD con pocos enlaces entre datos

### Escenarios Favorables para GraphQL

- Aplicaciones móviles que necesitan optimizar las solicitudes de datos
- Interfaces con muchos estados y vistas diferentes
- Cuando hay muchas relaciones entre entidades
- Cuando los clientes necesitan diferentes subconjuntos de datos
- Equipos de desarrollo frontend que necesitan iterar rápidamente sin depender del backend
