# Capítulo 3: Seguridad en el Desarrollo de Software

## 3.2 Seguridad Backend

## Introducción

La seguridad backend representa el núcleo de la defensa de los sistemas de software modernos. Mientras que el frontend protege la experiencia del usuario y la interfaz visible, el backend protege los activos más críticos de una organización: sus datos, lógica de negocio e infraestructura. 

En 2023, el costo promedio global de una violación de datos alcanzó los 4,45 millones de dólares, según el informe de IBM sobre el costo de las violaciones de datos. Más alarmante aún, el tiempo medio para detectar y contener una brecha fue de 277 días (197 días para identificarla y 80 días adicionales para contenerla). Estas estadísticas subrayan la importancia crítica de implementar medidas de seguridad robustas en el backend de cualquier aplicación.

La seguridad backend debe abordar múltiples capas:

1. **Infraestructura:** Servidores, redes, contenedores y servicios en la nube.
2. **Aplicación:** Lógica de negocio, APIs, manejo de solicitudes y respuestas.
3. **Datos:** Almacenamiento, procesamiento, transmisión y acceso.
4. **Identidad:** Autenticación, autorización y control de acceso.

El paradigma actual de seguridad backend adopta el principio de "defensa en profundidad", donde múltiples capas de seguridad se implementan para proteger los sistemas. Este enfoque reconoce que ninguna medida de seguridad es infalible, y que la redundancia en los mecanismos de protección es esencial.

Este capítulo explora los componentes fundamentales de la seguridad backend, desde la arquitectura de sistemas seguros hasta la implementación de modelos de autorización robustos, con enfoque especial en aplicaciones web y APIs modernas desarrolladas con tecnologías como Node.js, Express y TypeScript.

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

### 3.2.3 Seguridad de Datos en el Backend

La protección de los datos es una de las responsabilidades más críticas del backend, abarcando tanto su almacenamiento como su transmisión y procesamiento.

#### Cifrado de Datos

El cifrado garantiza que la información sensible permanezca protegida incluso si los sistemas de seguridad perimetral fallan.

```typescript
// utils/encryption.ts - Implementación de cifrado/descifrado usando crypto
import crypto from 'crypto';

// Configuración para cifrado simétrico AES-256-GCM
const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY as string; // Debe ser una clave de 32 bytes (256 bits)
const IV_LENGTH = 16; // Para AES, el IV siempre es de 16 bytes
const AUTH_TAG_LENGTH = 16; // Length of auth tag in GCM mode

/**
 * Cifra datos sensibles usando AES-256-GCM
 * @param text Texto a cifrar
 * @returns Texto cifrado en formato: iv:authTag:encrypted
 */
export function encrypt(text: string): string {
  // Generar un IV aleatorio para cada operación de cifrado
  const iv = crypto.randomBytes(IV_LENGTH);
  
  // Crear el cifrador con la clave, el IV y el algoritmo
  const cipher = crypto.createCipheriv('aes-256-gcm', Buffer.from(ENCRYPTION_KEY, 'hex'), iv);
  
  // Cifrar el texto
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  // Obtener el authentication tag (importante para GCM)
  const authTag = cipher.getAuthTag().toString('hex');
  
  // Devolver IV + authTag + datos cifrados en formato hexadecimal
  return iv.toString('hex') + ':' + authTag + ':' + encrypted;
}

/**
 * Descifra datos cifrados con AES-256-GCM
 * @param encryptedText Texto cifrado en formato: iv:authTag:encrypted
 * @returns Texto original descifrado
 */
export function decrypt(encryptedText: string): string {
  // Separar los componentes
  const parts = encryptedText.split(':');
  if (parts.length !== 3) {
    throw new Error('Formato de texto cifrado inválido');
  }
  
  const iv = Buffer.from(parts[0], 'hex');
  const authTag = Buffer.from(parts[1], 'hex');
  const encrypted = parts[2];
  
  // Crear el descifrador
  const decipher = crypto.createDecipheriv('aes-256-gcm', Buffer.from(ENCRYPTION_KEY, 'hex'), iv);
  
  // Establecer el authentication tag
  decipher.setAuthTag(authTag);
  
  // Descifrar
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

// Ejemplo de uso
export function encryptionExample() {
  // Dato sensible a proteger
  const sensitiveData = "Número de tarjeta: 4111-1111-1111-1111, CVV: 123";
  console.log('Datos originales:', sensitiveData);
  
  // Cifrar los datos
  const encryptedData = encrypt(sensitiveData);
  console.log('Datos cifrados:', encryptedData);
  
  // Descifrar los datos
  const decryptedData = decrypt(encryptedData);
  console.log('Datos descifrados:', decryptedData);
  
  // Verificar que los datos descifrados coinciden con los originales
  console.log('Verificación correcta:', sensitiveData === decryptedData);
}
```

#### Protección de Datos en Reposo

Los datos en reposo (almacenados en bases de datos, sistemas de archivos, etc.) requieren protección adicional:

1. **Cifrado a nivel de campo**: Para datos especialmente sensibles como números de tarjetas de crédito o información médica.

2. **Cifrado de toda la base de datos**: Proporciona una capa adicional de protección.

3. **Técnicas de tokenización**: Reemplazan datos sensibles con tokens sin significado intrínseco.

```typescript
// services/userDataService.ts
import { encrypt, decrypt } from '../utils/encryption';

interface User {
  id: string;
  name: string;
  email: string;
  // Campos sensibles que serán cifrados
  phoneNumber?: string;
  healthInfo?: string;
}

class UserDataService {
  // Prepara un usuario para almacenamiento en base de datos
  // cifrando los campos sensibles
  prepareForStorage(user: User): any {
    // Clonar el usuario para no modificar el original
    const preparedUser: any = { ...user };
    
    // Cifrar campos sensibles si existen
    if (user.phoneNumber) {
      preparedUser.phoneNumber = encrypt(user.phoneNumber);
    }
    
    if (user.healthInfo) {
      preparedUser.healthInfo = encrypt(user.healthInfo);
    }
    
    // Agregar metadatos para saber que campos están cifrados
    preparedUser.encryptedFields = [];
    if (user.phoneNumber) preparedUser.encryptedFields.push('phoneNumber');
    if (user.healthInfo) preparedUser.encryptedFields.push('healthInfo');
    
    return preparedUser;
  }
  
  // Restaura un usuario desde la base de datos
  // descifrando los campos sensibles
  restoreFromStorage(storedUser: any): User {
    // Clonar el usuario para no modificar el original
    const user: User = { ...storedUser };
    
    // Verificar qué campos están cifrados
    const encryptedFields = storedUser.encryptedFields || [];
    
    // Descifrar los campos necesarios
    if (encryptedFields.includes('phoneNumber') && storedUser.phoneNumber) {
      user.phoneNumber = decrypt(storedUser.phoneNumber);
    }
    
    if (encryptedFields.includes('healthInfo') && storedUser.healthInfo) {
      user.healthInfo = decrypt(storedUser.healthInfo);
    }
    
    // Eliminar el campo de metadatos
    delete user.encryptedFields;
    
    return user;
  }
}
```

#### Protección contra Inyección de SQL

Las inyecciones SQL siguen siendo una de las vulnerabilidades más comunes y peligrosas en aplicaciones backend.

```typescript
// ❌ INSEGURO: Vulnerable a inyección SQL
function getUser(id: string) {
  const query = `SELECT * FROM users WHERE id = ${id}`;
  // El atacante podría enviar: "1; DROP TABLE users;--"
  return db.execute(query);
}

// ✅ SEGURO: Consultas parametrizadas con valores escapados
function getUserSafe(id: string) {
  return db.execute('SELECT * FROM users WHERE id = ?', [id]);
}

// ✅ SEGURO: Uso de ORM que maneja automáticamente la parametrización
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function getUserWithORM(id: string) {
  return prisma.user.findUnique({
    where: {
      id: id
    }
  });
}
```

### 3.2.4 Mejores Prácticas de Seguridad Backend

#### Protección contra Ataques de Fuerza Bruta

Los atacantes pueden intentar adivinar contraseñas o tokens mediante intentos repetidos. Las defensas incluyen:

```typescript
// middleware/rateLimiter.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';

// Cliente Redis para almacenamiento distribuido
const redisClient = new Redis({
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || '6379'),
});

// Limitador general para todas las rutas (evita DDoS)
export const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 100, // Límite de 100 solicitudes por ventana por IP
  standardHeaders: true,
  message: 'Demasiadas solicitudes desde esta IP, por favor intente de nuevo después de 15 minutos',
  // Uso de Redis para entornos con múltiples instancias
  store: new RedisStore({
    sendCommand: (...args: string[]) => redisClient.call(...args)
  })
});

// Limitador estricto para rutas sensibles (login, registro)
export const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hora
  max: 10, // 10 intentos por hora
  standardHeaders: true,
  message: 'Demasiados intentos de autenticación. Cuenta bloqueada temporalmente.',
  // Aumenta el tiempo de bloqueo con cada intento
  // para mitigar ataques sostenidos
  skipSuccessfulRequests: false,
  store: new RedisStore({
    sendCommand: (...args: string[]) => redisClient.call(...args),
    // La clave puede incluir identificadores específicos como nombre de usuario
    // para bloquear intentos sobre cuentas específicas
    prefix: 'rl:auth:'
  })
});
```

#### Cabeceras de Seguridad

Configura cabeceras HTTP para reforzar la seguridad de la aplicación:

```typescript
// middleware/securityHeaders.ts
import helmet from 'helmet';
import { Express } from 'express';

export function configureSecurityHeaders(app: Express) {
  // Aplicar helmet con configuración personalizada
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'", 'cdn.jsdelivr.net', 'ajax.googleapis.com'],
        styleSrc: ["'self'", "'unsafe-inline'", 'cdn.jsdelivr.net', 'fonts.googleapis.com'],
        imgSrc: ["'self'", 'data:', 'cdn.example.com'],
        fontSrc: ["'self'", 'fonts.gstatic.com'],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: [],
      },
    },
    // Establecer X-XSS-Protection para navegadores antiguos
    xssFilter: true,
    // Prevenir MIME sniffing
    noSniff: true,
    // Configuración de HSTS
    hsts: {
      maxAge: 31536000, // 1 año en segundos
      includeSubDomains: true,
      preload: true
    },
    // Prevenir clickjacking
    frameguard: { 
      action: 'deny' 
    },
    // Restringir información de referrer
    referrerPolicy: { 
      policy: 'same-origin' 
    }
  }));
  
  // Cabeceras personalizadas adicionales
  app.use((req, res, next) => {
    // Ejemplo: Cabecera para prevenir cacheo de datos sensibles
    res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate, proxy-revalidate');
    res.setHeader('Pragma', 'no-cache');
    res.setHeader('Expires', '0');
    next();
  });
}
```

#### Validación de Entrada Robusta

Toda entrada del usuario debe ser validada rigurosamente:

```typescript
// controllers/orderController.ts
import { z } from 'zod';
import { Request, Response } from 'express';

// Esquema para validación de creación de órdenes
const CreateOrderSchema = z.object({
  productId: z.string().uuid({ message: "ID de producto inválido" }),
  quantity: z.number().int().positive().max(100, { 
    message: "La cantidad debe ser entre 1 y 100" 
  }),
  shippingAddress: z.object({
    street: z.string().min(5, { message: "Calle demasiado corta" }),
    city: z.string().min(2, { message: "Ciudad inválida" }),
    postalCode: z.string().regex(/^\d{5}(-\d{4})?$/, { 
      message: "Código postal inválido" 
    }),
    country: z.string().min(2, { message: "País inválido" })
  }),
  paymentMethod: z.enum(['credit_card', 'paypal', 'bank_transfer'], {
    errorMap: () => ({ message: "Método de pago no soportado" })
  }),
  // Campos condicionales
  creditCardInfo: z.object({
    cardNumber: z.string().regex(/^\d{16}$/, { 
      message: "Número de tarjeta inválido" 
    }),
    expiryDate: z.string().regex(/^(0[1-9]|1[0-2])\/\d{2}$/, { 
      message: "Fecha de expiración inválida (MM/YY)" 
    }),
    cvv: z.string().regex(/^\d{3}$/, { 
      message: "CVV inválido" 
    })
  }).optional().nullable()
    .refine(
      (data) => data !== null || data !== undefined, 
      { message: "Información de tarjeta requerida para pagos con tarjeta" }
    ),
});

// Tipo derivado del esquema para TypeScript
type CreateOrderRequest = z.infer<typeof CreateOrderSchema>;

// Controlador para crear una nueva orden
export async function createOrder(req: Request, res: Response) {
  try {
    // Validar los datos de entrada
    const validationResult = CreateOrderSchema.safeParse(req.body);
    
    if (!validationResult.success) {
      // Devolver errores de validación formateados
      return res.status(400).json({
        success: false,
        message: "Datos de orden inválidos",
        errors: validationResult.error.format()
      });
    }
    
    // Datos validados y tipados correctamente
    const orderData: CreateOrderRequest = validationResult.data;
    
    // Procesamiento adicional de seguridad
    // Por ejemplo, sanitización de datos o verificación de permisos
    
    // Procesar la orden con los datos validados
    const order = await orderService.createOrder(orderData, req.user.id);
    
    return res.status(201).json({
      success: true,
      message: "Orden creada exitosamente",
      data: {
        orderId: order.id,
        estimatedDelivery: order.estimatedDeliveryDate
      }
    });
  } catch (error) {
    console.error("Error al crear orden:", error);
    return res.status(500).json({
      success: false,
      message: "Error interno del servidor",
      // No revelar detalles del error en producción
      ...(process.env.NODE_ENV !== 'production' && { error: error.message })
    });
  }
}
```

## Conclusiones

La seguridad backend es un campo en constante evolución que requiere vigilancia continua y actualización de conocimientos. Los principios fundamentales que deben guiar el desarrollo de sistemas backend seguros incluyen:

1. **Defensa en profundidad:** Implementar múltiples capas de seguridad para que si una falla, otras sigan protegiendo el sistema.

2. **Principio del mínimo privilegio:** Otorgar solo los permisos estrictamente necesarios a usuarios, procesos y servicios.

3. **Validación completa:** Nunca confiar en los datos de entrada, independientemente de su origen. Toda entrada debe ser validada, filtrada y sanitizada.

4. **Gestión adecuada de secretos:** Las claves, tokens y credenciales nunca deben estar en el código fuente ni repositorios. Utilizar sistemas de gestión de secretos como HashiCorp Vault o AWS Secrets Manager.

5. **Monitorización y logging:** Mantener registros detallados de eventos de seguridad y establecer sistemas de alerta para detectar comportamientos anómalos.

6. **Actualizaciones y parcheo:** Mantener todas las dependencias y sistemas actualizados con los últimos parches de seguridad.

La implementación efectiva de estas prácticas requiere un enfoque holístico que considere la seguridad desde las primeras etapas del diseño hasta el despliegue y mantenimiento continuo. En un entorno donde las amenazas evolucionan constantemente, la seguridad backend no es un producto final, sino un proceso iterativo de mejora continua.
