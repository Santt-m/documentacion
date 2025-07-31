# Capítulo 3: Seguridad en el Desarrollo de Software

## 3.2 Seguridad Backend

### 3.2.1 Anatomía de una Solicitud HTTP Segura

Cuando un usuario realiza una solicitud GET, la aplicación debe seguir una serie de pasos de seguridad para garantizar que la solicitud es legítima y que el usuario tiene permiso para acceder a los datos solicitados.

```
graph TD
    A[Cliente: Envía Solicitud GET] --> B{Servidor: Firewall / WAF};
    B -- Pasa Filtros --> C{Servidor: Balanceador de Carga};
    C --> D[Servidor: API Gateway / Reverse Proxy];
    D --> E{Servidor: Middleware de Autenticación};
    E -- Token Válido --> F{Servidor: Middleware de Autorización};
    F -- Usuario Autorizado --> G[Servidor: Controlador / Ruta];
    G --> H{Servidor: Validación de Entrada};
    H -- Entrada Válida --> I[Servidor: Lógica de Negocio];
    I --> J[Servidor: Base de Datos];
    J -- Datos Recuperados --> K[Servidor: Validación de Datos];
    K --> L[Servidor: Serialización de Respuesta];
    L --> M[Servidor: Envía Respuesta];
    M --> N[Cliente: Recibe Respuesta];
    
    B -- Bloqueada --> O[Respuesta: 403 Forbidden];
    E -- Token Inválido --> P[Respuesta: 401 Unauthorized];
    F -- Usuario No Autorizado --> Q[Respuesta: 403 Forbidden];
    H -- Entrada Inválida --> R[Respuesta: 400 Bad Request];
```

#### Flujo Detallado:

**Cliente: Envía Solicitud GET:**

Lógica: El cliente (navegador, aplicación móvil, etc.) envía una solicitud HTTP GET, incluyendo normalmente una cabecera de autorización con un token.

Herramientas: Fetch API, Axios, XMLHttpRequest.

**Servidor: Firewall / WAF (Web Application Firewall):**

Lógica: Filtra el tráfico entrante, bloqueando solicitudes maliciosas conocidas como inyecciones SQL, XSS, etc.

Herramientas: ModSecurity, AWS WAF, Cloudflare WAF.

**Servidor: Balanceador de Carga:**

Lógica: Distribuye las solicitudes entrantes entre múltiples instancias del servidor para asegurar la disponibilidad y el rendimiento, y puede realizar terminación SSL/TLS.

Herramientas: Nginx, HAProxy, AWS ELB, Google Cloud Load Balancing.

**Servidor: API Gateway / Reverse Proxy:**

Lógica: Actúa como punto de entrada unificado para las APIs, puede realizar rate limiting, caching, y enrutamiento de solicitudes.

Herramientas: Kong, Amazon API Gateway, Express Gateway.

**Servidor: Middleware de Autenticación:**

Lógica: Verifica la identidad del usuario mediante la validación del token de autorización, asegurando que sea válido, no haya expirado y esté firmado correctamente.

Herramientas: Librerías de Passport.js (Node.js), middlewares personalizados, JWT libraries.

**Servidor: Controlador / Ruta (Validación de Entrada):**

Lógica: Valida que los parámetros de la solicitud (query params, path params) cumplan con los requisitos y restricciones esperadas.

Herramientas: Zod, Joi, class-validator, express-validator.

**Servidor: Lógica de Negocio:**

Lógica: Implementa la funcionalidad principal de la aplicación, respetando las reglas de negocio y asegurando que se apliquen los controles de acceso adecuados.

Herramientas: Frameworks como Express, NestJS, Spring Boot.

**Servidor: Base de Datos:**

Lógica: Ejecuta consultas para recuperar o modificar datos, aplicando medidas de seguridad como el uso de consultas parametrizadas para prevenir inyecciones SQL.

Herramientas: ORMs como Sequelize, TypeORM, Prisma, Mongoose.

**Servidor: Validación de Datos:**

Lógica: Verifica que los datos recuperados cumplen con los requisitos de negocio y no contienen información sensible no autorizada.

Herramientas: Zod, TypeScript, validadores personalizados.

**Servidor: Serialización de Respuesta:**

Lógica: Convierte los datos internos en un formato adecuado para la respuesta HTTP (generalmente JSON), asegurando que no se filtren datos sensibles o internos.

Herramientas: JSON.stringify, librerías de serialización/deserialización.

**Servidor: Envía Respuesta:** El servidor envía la respuesta HTTP al cliente, idealmente sobre una conexión HTTPS segura.

### 3.2.2 Autenticación y Autorización Robustas

#### Hashing Seguro de Contraseñas

El hashing de contraseñas es fundamental para la seguridad de un sistema, ya que almacena las contraseñas de manera que no puedan ser recuperadas en texto claro, incluso si la base de datos se ve comprometida.

```typescript
// utils/password.ts
import * as bcrypt from 'bcrypt';

// Cantidad de rondas para el algoritmo bcrypt
const saltRounds = 12; // Ajustar según las necesidades de seguridad y rendimiento

/**
 * Hashea una contraseña utilizando bcrypt
 * @param password Contraseña en texto claro
 * @returns Contraseña hasheada
 */
async function hashPassword(password: string): Promise<string> {
  const hashedPassword = await bcrypt.hash(password, saltRounds);
  console.log('Contraseña hasheada:', hashedPassword);
  return hashedPassword;
}

/**
 * Compara una contraseña en texto claro con su versión hasheada
 * @param password Contraseña en texto claro
 * @param hashedPassword Contraseña hasheada almacenada
 * @returns Booleano que indica si la contraseña es correcta
 */
async function comparePassword(password: string, hashedPassword: string): Promise<boolean> {
  const isMatch = await bcrypt.compare(password, hashedPassword);
  console.log('Coincide la contraseña:', isMatch);
  return isMatch;
}

// Uso
const password = 'ContraseñaSegura123!';
hashPassword(password).then(hashedPassword => {
  console.log('Hash para almacenamiento:', hashedPassword);
  
  // Simular verificación posterior
  comparePassword(password, hashedPassword).then(isMatch => {
    console.log('Verificación exitosa:', isMatch); // Debería ser true
  });
  
  // Simular verificación con contraseña incorrecta
  comparePassword('ContraseñaIncorrecta', hashedPassword).then(isMatch => {
    console.log('Verificación fallida:', isMatch); // Debería ser false
  });
});
```

#### JSON Web Tokens (JWT) para Autenticación Segura

Los JWT son una forma estándar de representar claims de manera segura entre dos partes. Son comunes para autenticación y autorización en aplicaciones modernas.

```
graph TD
    A[Cliente: Envía Credenciales] --> B{Servidor de Autenticación: Valida Credenciales};
    B -- Credenciales Válidas --> C[Servidor de Autenticación];
    B -- Genera JWT --> C[Servidor de Autenticación];
    C -- Envía JWT (HttpOnly Cookie) --> A;
    A -- Solicitud con JWT --> D{Servidor de Recursos};
    D -- Verifica JWT --> E[Servidor de Recursos];
    E -- Acceso Concedido/Denegado --> A;
```

#### OAuth 2.0 y OpenID Connect:

**OAuth 2.0** es un protocolo de autorización que permite a aplicaciones acceder a recursos en nombre del usuario sin compartir credenciales.

**OpenID Connect (OIDC)** es una capa de identidad encima de OAuth 2.0 que proporciona información de autenticación estándar.

#### Tipos de Concesión (Grant Types) en OAuth 2.0:

1. **Authorization Code Flow:**
   - Más seguro para aplicaciones con backend
   - El cliente recibe un código de autorización que intercambia por un token de acceso
   - Soporta tokens de actualización

2. **Implicit Flow:**
   - Para aplicaciones SPA (desuso debido a consideraciones de seguridad)
   - El token de acceso se devuelve directamente al cliente
   - No hay tokens de actualización

3. **Client Credentials:**
   - Para autenticación de servidor a servidor
   - El cliente usa sus propias credenciales para obtener un token
   - No involucra a un usuario final

4. **Resource Owner Password Credentials:**
   - El cliente recolecta credenciales del usuario y las envía al servidor
   - Menos seguro, se utiliza solo cuando hay una alta confianza entre el cliente y el servidor

### 3.2.3 Modelos de Control de Acceso

#### RBAC (Role-Based Access Control):

El control de acceso basado en roles asigna permisos a roles específicos y luego asigna esos roles a los usuarios.

**Implementación:**

```typescript
// Definición de Roles y Permisos
enum Permission {
  READ_USERS = 'read:users',
  WRITE_USERS = 'write:users',
  DELETE_USERS = 'delete:users',
  READ_PRODUCTS = 'read:products',
  WRITE_PRODUCTS = 'write:products',
  DELETE_PRODUCTS = 'delete:products',
  ADMIN_PANEL = 'admin:panel'
}

interface Role {
  name: string;
  permissions: Permission[];
}

// Definición de roles predefinidos
const roles: Record<string, Role> = {
  ADMIN: {
    name: 'Administrator',
    permissions: [
      Permission.READ_USERS,
      Permission.WRITE_USERS,
      Permission.DELETE_USERS,
      Permission.READ_PRODUCTS,
      Permission.WRITE_PRODUCTS,
      Permission.DELETE_PRODUCTS,
      Permission.ADMIN_PANEL
    ]
  },
  EDITOR: {
    name: 'Content Editor',
    permissions: [
      Permission.READ_USERS,
      Permission.READ_PRODUCTS,
      Permission.WRITE_PRODUCTS
    ]
  },
  VIEWER: {
    name: 'Viewer',
    permissions: [
      Permission.READ_USERS,
      Permission.READ_PRODUCTS
    ]
  }
};

// Helper para verificar permisos
function hasPermission(userRoles: string[], permission: Permission): boolean {
  return userRoles.some(roleName => {
    const role = roles[roleName];
    return role && role.permissions.includes(permission);
  });
}

// Ejemplo de uso
const userRoles = ['EDITOR', 'VIEWER']; // Este usuario tiene dos roles

console.log('¿Puede leer usuarios?', hasPermission(userRoles, Permission.READ_USERS)); // true
console.log('¿Puede escribir productos?', hasPermission(userRoles, Permission.WRITE_PRODUCTS)); // true
console.log('¿Puede eliminar usuarios?', hasPermission(userRoles, Permission.DELETE_USERS)); // false
```

**Ventajas de RBAC:**
- Simple de entender e implementar
- Fácil gestión de permisos a través de roles
- Adecuado para organizaciones con estructura jerárquica clara

**Desventajas:**
- Puede volverse complejo con muchos roles y permisos
- No es ideal para permisos basados en atributos específicos del contexto

#### ABAC (Attribute-Based Access Control):

El control de acceso basado en atributos utiliza políticas que combinan atributos del usuario, recurso, acción y entorno para tomar decisiones de autorización.

**Implementación Conceptual:**

```typescript
// Definición de tipos para ABAC
interface User {
  id: string;
  department: string;
  level: number;
  region: string;
}

interface Resource {
  id: string;
  type: string;
  owner: string;
  confidentiality: 'public' | 'internal' | 'confidential' | 'restricted';
}

interface Action {
  type: 'read' | 'write' | 'delete' | 'approve';
}

interface Environment {
  time: Date;
  ipAddress: string;
  deviceType: 'mobile' | 'desktop' | 'unknown';
}

// Función de decisión de política
function evaluatePolicy(user: User, resource: Resource, action: Action, environment: Environment): boolean {
  // Ejemplo de política: Los usuarios pueden ver documentos públicos
  if (resource.confidentiality === 'public' && action.type === 'read') {
    return true;
  }
  
  // Ejemplo de política: Los usuarios pueden ver documentos internos de su propio departamento
  if (resource.confidentiality === 'internal' && action.type === 'read' && 
      resource.type === 'document' && resource.owner === user.department) {
    return true;
  }
  
  // Ejemplo de política: Solo usuarios de nivel 3+ pueden editar documentos confidenciales
  if (resource.confidentiality === 'confidential' && action.type === 'write' && 
      user.level >= 3) {
    return true;
  }
  
  // Ejemplo de política: Documentos restringidos solo pueden ser accedidos durante horas laborales
  if (resource.confidentiality === 'restricted') {
    const hour = environment.time.getHours();
    if (hour < 9 || hour > 17) {
      return false;
    }
    
    // Y solo por usuarios de nivel 5+ de la misma región que el documento
    if (user.level >= 5 && user.region === resource.owner) {
      return true;
    }
  }
  
  // Por defecto, denegar acceso
  return false;
}

// Ejemplo de uso
const user: User = {
  id: 'user123',
  department: 'engineering',
  level: 4,
  region: 'europe'
};

const resource: Resource = {
  id: 'doc456',
  type: 'document',
  owner: 'engineering',
  confidentiality: 'confidential'
};

const action: Action = {
  type: 'write'
};

const environment: Environment = {
  time: new Date(), // Hora actual
  ipAddress: '192.168.1.1',
  deviceType: 'desktop'
};

const hasAccess = evaluatePolicy(user, resource, action, environment);
console.log('¿Tiene acceso?', hasAccess);
```

**Ventajas de ABAC:**
- Muy granular y flexible.
- Puede manejar decisiones complejas basadas en múltiples factores.
- Adaptable a diferentes contextos y situaciones.

**Desventaja:** Mayor complejidad de implementación y gestión.

### Middleware para Autorización:

En el backend, se utilizan middlewares para interceptar las solicitudes y aplicar las reglas de autorización antes de que la solicitud llegue a la lógica de negocio.

```typescript
// Ejemplo de middleware de autorización en Express (Node.js)
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

// Middleware para verificar JWT
export function authenticateJWT(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader) {
    return res.status(401).json({ message: 'Token de autenticación no proporcionado' });
  }
  
  const token = authHeader.split(' ')[1]; // Formato: "Bearer TOKEN"
  
  jwt.verify(token, process.env.JWT_SECRET as string, (err, decoded) => {
    if (err) {
      return res.status(403).json({ message: 'Token inválido o expirado' });
    }
    
    // Añadir la información del usuario al objeto request para usarlo en las rutas
    req.user = decoded;
    next();
  });
}

// Middleware para verificar permisos basados en roles
export function authorize(allowedRoles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Usuario no autenticado' });
    }
    
    const { roles } = req.user;
    
    // Verificar si el usuario tiene al menos uno de los roles permitidos
    const hasPermission = roles.some((role: string) => allowedRoles.includes(role));
    
    if (!hasPermission) {
      return res.status(403).json({ 
        message: 'No tienes permiso para acceder a este recurso' 
      });
    }
    
    next();
  };
}

// Uso en rutas
// app.get('/users', authenticateJWT, authorize(['admin']), (req, res) => {
//   res.send('Lista de usuarios');
// });
// app.get('/admin', authenticateJWT, authorize(['admin']), (req, res) => {
//   res.send('Bienvenido al panel de administración.');
// });
// app.post('/products', authenticateJWT, authorize(['admin', 'editor']), (req, res) => {
//   res.send('Producto creado.');
// });
```
