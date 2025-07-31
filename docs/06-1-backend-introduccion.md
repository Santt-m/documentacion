# 6.1 Backend Moderno: Introducción

El desarrollo backend ha evolucionado significativamente en los últimos años, adaptándose a las necesidades cambiantes del mercado y aprovechando nuevas tecnologías. Este capítulo introduce los fundamentos y tendencias actuales en el desarrollo backend moderno.

## 6.1.1 Evolución del Backend

El backend ha pasado por varias fases de evolución:

1. **Aplicaciones monolíticas tradicionales**: Todo el código backend se ejecutaba como una única aplicación.
2. **Arquitecturas orientadas a servicios (SOA)**: División en servicios con responsabilidades específicas.
3. **Microservicios**: Descomposición en servicios más pequeños e independientes.
4. **Serverless**: Ejecución de código sin administrar la infraestructura subyacente.

### Características del Backend Moderno

- **Escalabilidad horizontal**: Capacidad de escalar añadiendo más instancias.
- **Alta disponibilidad**: Sistemas diseñados para minimizar el tiempo de inactividad.
- **Resiliencia**: Capacidad de recuperarse automáticamente de fallos.
- **Observabilidad**: Facilidad para monitorear y diagnosticar problemas.
- **API-first**: Diseño centrado en APIs como interfaz principal.
- **Cloud-native**: Aprovechamiento máximo de servicios en la nube.

## 6.1.2 Principales Tecnologías y Lenguajes

El ecosistema de backend moderno incluye una variedad de lenguajes y frameworks, cada uno con sus propias fortalezas:

### Node.js y JavaScript/TypeScript

Node.js ha revolucionado el desarrollo backend permitiendo usar JavaScript/TypeScript en el servidor.

**Ventajas**:
- Mismo lenguaje en frontend y backend
- Modelo de E/S no bloqueante eficiente para operaciones con alta concurrencia
- Ecosistema NPM extenso

**Frameworks populares**:
- Express.js: Minimalista y flexible
- NestJS: Inspirado en Angular, con arquitectura modular
- Fastify: Enfocado en rendimiento

**Ejemplo de API REST con Express**:

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

// Middleware de logging simple
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
});

// Datos de ejemplo (en una app real usaríamos una base de datos)
let users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' }
];

// Rutas CRUD
app.get('/api/users', (req, res) => {
  res.json(users);
});

app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ message: 'User not found' });
  res.json(user);
});

app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  if (!name || !email) {
    return res.status(400).json({ message: 'Name and email are required' });
  }
  
  const newUser = {
    id: users.length + 1,
    name,
    email
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ message: 'User not found' });
  
  const { name, email } = req.body;
  if (name) user.name = name;
  if (email) user.email = email;
  
  res.json(user);
});

app.delete('/api/users/:id', (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  if (userIndex === -1) return res.status(404).json({ message: 'User not found' });
  
  const deletedUser = users[userIndex];
  users = users.filter(u => u.id !== parseInt(req.params.id));
  
  res.json(deletedUser);
});

// Manejo de errores
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Something went wrong!' });
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

### Python

Python es ampliamente utilizado en backend, especialmente para aplicaciones que requieren procesamiento de datos o integración con machine learning.

**Frameworks populares**:
- Django: Completo, con "baterías incluidas"
- Flask: Ligero y minimalista
- FastAPI: Moderno, rápido y con soporte de tipado

### Java y el Ecosistema JVM

Java sigue siendo una opción sólida para aplicaciones empresariales, junto con otros lenguajes JVM como Kotlin y Scala.

**Frameworks populares**:
- Spring Boot: Simplifica el desarrollo con Spring
- Quarkus: Optimizado para aplicaciones nativas en la nube
- Micronaut: Diseñado para microservicios y aplicaciones serverless

### Go (Golang)

Go es cada vez más popular para servicios en la nube y microservicios por su rendimiento y simplicidad.

**Ventajas**:
- Compilación rápida
- Ejecución eficiente
- Soporte nativo para concurrencia

### Rust

Rust está ganando tracción para aplicaciones que requieren alto rendimiento y seguridad de memoria.

**Ventajas**:
- Control de memoria sin recolección de basura
- Prevención de errores en tiempo de compilación
- Rendimiento comparable a C/C++

## 6.1.3 Comparativa: ¿Cuándo Usar Qué Tecnología?

| Lenguaje/Framework | Casos de uso ideales |
|-------------------|---------------------|
| Node.js/Express   | APIs con alta concurrencia, aplicaciones en tiempo real, cuando el equipo ya conoce JavaScript |
| Python/Django     | Desarrollo rápido, aplicaciones con ML/AI, procesamiento de datos |
| Java/Spring Boot  | Sistemas empresariales, aplicaciones de misión crítica |
| Go                | Microservicios, herramientas de infraestructura, alto rendimiento |
| Rust              | Sistemas de bajo nivel, seguridad crítica, máximo rendimiento |

## 6.1.4 Consideraciones para Elegir una Stack de Backend

Al seleccionar tecnologías para un proyecto backend, considera:

1. **Requisitos del proyecto**: Características, escalabilidad, rendimiento
2. **Experiencia del equipo**: Conocimientos existentes y curva de aprendizaje
3. **Ecosistema**: Disponibilidad de bibliotecas y herramientas
4. **Soporte a largo plazo**: Estabilidad y comunidad activa
5. **Rendimiento**: Necesidades específicas de latencia y throughput
6. **Escalabilidad**: Capacidad para crecer con la demanda
7. **Tiempo de desarrollo**: Velocidad para llegar al mercado
