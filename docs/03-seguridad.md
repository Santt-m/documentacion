# Capítulo 3: Seguridad en Desarrollo Full-Stack: Un Enfoque Integral

Este capítulo aborda la importancia crítica de la seguridad en el desarrollo de aplicaciones full-stack, destacando cómo proteger tanto el frontend como el backend de las amenazas comunes. Se presentarán principios de seguridad fundamentales y ejemplos de implementación práctica.

## 3.1 Principios OWASP Aplicados y Amenazas Comunes

La Open Web Application Security Project (OWASP) es una fundación sin fines de lucro que trabaja para mejorar la seguridad del software. Sus guías y principios son esenciales para construir aplicaciones robustas y seguras.

### 3.1.1 Frontend (Next.js/React)

La seguridad en el frontend se centra en proteger al usuario y prevenir ataques que puedan comprometer la integridad de la aplicación en el navegador.

```jsx
// pages/_document.tsx - Cabeceras de seguridad
import Document, { Html, Head, Main, NextScript } from 'next/document';

class MyDocument extends Document {
  render() {
    return (
      <Html lang="es">
        <Head>
          {/* Content Security Policy (CSP): Ayuda a prevenir ataques XSS y de inyección de datos. */}
          {/* Restringe las fuentes de contenido (scripts, estilos, imágenes, etc.) que el navegador puede cargar. */}
          <meta httpEquiv="Content-Security-Policy" 
                content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'" />
          
          {/* X-Content-Type-Options: Previene ataques de "MIME-sniffing". */}
          {/* Asegura que el navegador interprete los tipos de contenido tal como los declara el servidor, evitando que ejecute scripts maliciosos. */}
          <meta httpEquiv="X-Content-Type-Options" content="nosniff" />
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}
```

### 3.1.2 Backend (Node.js)

La seguridad en el backend es crucial para proteger los datos sensibles, la lógica de negocio y los recursos del servidor.

```javascript
// middleware/security.ts
import express from 'express';
import helmet from 'helmet'; // Colección de middlewares para seguridad HTTP
import rateLimit from 'express-rate-limit'; // Middleware para limitar el número de solicitudes

const app = express(); // Asumiendo que 'app' es tu instancia de Express

// Helmet: Configura cabeceras HTTP de seguridad para proteger la aplicación de vulnerabilidades comunes.
app.use(helmet({
  contentSecurityPolicy: false, // Deshabilitado aquí porque se personaliza en el frontend (_document.tsx)
  // HTTP Strict Transport Security (HSTS): Fuerza a los navegadores a usar HTTPS para futuras solicitudes al dominio.
  hsts: { maxAge: 63072000 }   // 2 años en segundos
}));

// Rate Limiting: Limita el número de solicitudes de una misma IP en un período de tiempo dado.
// Ayuda a prevenir ataques de fuerza bruta y DDoS.
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,    // 15 minutos (tiempo de la ventana)
  max: 100                      // 100 solicitudes permitidas por IP dentro de la ventana
}));

// Exportar 'app' o el middleware si se usa en un archivo de configuración separado
export default app; // O exporta los middlewares individualmente
```

### 3.1.3 Principales Amenazas y Mitigaciones (Ampliado)

#### Inyecciones SQL

Las inyecciones SQL ocurren cuando un atacante inserta código SQL malicioso en una entrada de datos de la aplicación, que luego es ejecutado por la base de datos.

**Mitigación Clave**: Utilizar ORMs (Object-Relational Mappers) o consultas parametrizadas. Estas herramientas separan los datos de los comandos SQL, evitando que la entrada del usuario sea interpretada como código ejecutable.

**Validación de Entradas**: Implementar una validación estricta de todas las entradas del usuario en el backend. Librerías como Zod o Joi en TypeScript/Node.js son excelentes para definir esquemas y asegurar que los datos recibidos cumplen con el formato y tipo esperados.

```typescript
import { z } from 'zod'; // Importar Zod para la validación de esquemas

// Definición de un esquema de usuario con Zod
// Cada propiedad tiene reglas de validación específicas
const UserSchema = z.object({
  email: z.string().email({ message: "Formato de email inválido." }), // Valida que sea un email válido
  password: z.string()
              .min(8, { message: "La contraseña debe tener al menos 8 caracteres." }) // Longitud mínima
              .regex(/[A-Z]/, { message: "La contraseña debe contener al menos una mayúscula." }) // Al menos una mayúscula
              .regex(/[a-z]/, { message: "La contraseña debe contener al menos una minúscula." }) // Al menos una minúscula
              .regex(/[0-9]/, { message: "La contraseña debe contener al menos un número." }) // Al menos un número
              .regex(/[^a-zA-Z0-9]/, { message: "La contraseña debe contener al menos un carácter especial." }), // Al menos un carácter especial
  username: z.string().min(3).max(20).optional(), // Opcional, con longitud
  age: z.number().int().positive().optional(), // Número entero positivo opcional
});

// Ejemplo de uso de la validación
type NewUser = z.infer<typeof UserSchema>; // Infiere el tipo TypeScript del esquema

function validateAndProcessUser(userData: any) {
  try {
    // Intenta parsear y validar los datos de entrada contra el esquema
    const validatedUser = UserSchema.parse(userData);
    console.log("Datos de usuario válidos:", validatedUser);
    // Aquí se continuaría con la lógica de negocio, como guardar en la base de datos
    return { success: true, data: validatedUser };
  } catch (error: any) {
    console.error("Error de validación:", error.errors);
    return { success: false, errors: error.errors };
  }
}

// Ejemplo de datos válidos
validateAndProcessUser({
  email: "test@example.com",
  password: "Password123!",
  username: "devUser",
  age: 30
});

// Ejemplo de datos inválidos (email incorrecto, contraseña débil)
validateAndProcessUser({
  email: "invalid-email",
  password: "short",
});
```

#### Cross-Site Scripting (XSS)

Los ataques XSS ocurren cuando un atacante inyecta scripts maliciosos en páginas web vistas por otros usuarios. Estos scripts pueden robar cookies, tokens de sesión o redirigir a los usuarios a sitios fraudulentos.

**Mitigación Clave**:

- **Sanitizar Inputs**: Siempre sanitizar cualquier entrada de usuario antes de renderizarla en el DOM. Librerías como DOMPurify son efectivas para limpiar HTML y prevenir la inyección de scripts.

- **Escape de Salidas**: Escapar el contenido generado por el usuario antes de mostrarlo en la interfaz. Frameworks como React realizan un auto-escape en JSX, lo que significa que el contenido incrustado en {} se convierte en texto seguro por defecto.

```jsx
// Ejemplo de auto-escape en React (JSX)
import React from 'react';

interface UserCommentProps {
  comment: string;
}

const UserComment: React.FC<UserCommentProps> = ({ comment }) => {
  // Si 'comment' contiene HTML o scripts maliciosos (ej. "<script>alert('XSS')</script>"),
  // React los escapará automáticamente, mostrándolos como texto plano en lugar de ejecutarlos.
  return (
    <div className="comment-box">
      <p>Comentario del usuario:</p>
      <div className="comment-content">
        {comment} {/* React escapa automáticamente el contenido aquí */}
      </div>
    </div>
  );
};

// Ejemplo de uso:
function AppXSSExample(): JSX.Element {
  const maliciousComment = "<img src='x' onerror='alert(\"XSS Attack!\")' />";
  const safeComment = "¡Hola, este es un comentario normal!";

  return (
    <div>
      <UserComment comment={maliciousComment} />
      <UserComment comment={safeComment} />
    </div>
  );
}
```

## 3.2 Flujo de Seguridad en una Solicitud GET (Ejemplo Detallado)

Cuando un usuario realiza una solicitud GET, la aplicación debe seguir una serie de pasos de seguridad para garantizar que la solicitud es legítima y que el usuario tiene permiso para acceder a los datos solicitados.

```
graph TD
    A[Cliente: Envía Solicitud GET] --> B{Servidor: Firewall / WAF};
    B -- Pasa Filtros --> C{Servidor: Balanceador de Carga};
    C --> D[Servidor: API Gateway / Reverse Proxy];
    D -- Autenticación/Autorización --> E[Servidor: Middleware de Seguridad];
    E -- Validar Entrada --> F[Servidor: Controlador / Ruta];
    F -- Lógica de Negocio / Acceso a Datos --> G[Servidor: Servicio / Repositorio];
    G --> H[Base de Datos / Almacenamiento];
```

### Desglose del Flujo:

1. **Cliente**: El navegador del usuario o una aplicación cliente envía una solicitud GET a una URL específica.

2. **Firewall / Web Application Firewall (WAF)**: Primera línea de defensa que filtra solicitudes maliciosas conocidas (inyecciones, XSS, etc.). Servicios como Cloudflare o AWS WAF pueden proporcionar esta protección.

3. **Balanceador de Carga**: Distribuye el tráfico entre varios servidores para mejorar la disponibilidad y resistencia a ataques DDoS.

4. **API Gateway / Reverse Proxy**: Enruta solicitudes a los servicios adecuados y puede aplicar políticas de throttling (limitación de frecuencia), validación de solicitudes, etc.

5. **Middleware de Seguridad**: Verifica la autenticación (¿el usuario es quien dice ser?) y la autorización (¿el usuario tiene permiso para este recurso?).

6. **Controlador / Ruta**: Valida y sanitiza parámetros de entrada antes de procesarlos.

7. **Servicio / Repositorio**: Aplica reglas de negocio y restricciones de acceso a datos.

8. **Base de Datos / Almacenamiento**: Accede a los datos con permisos mínimos necesarios (principio de privilegio mínimo).

## 3.3 Implementación de JWT para Autenticación Segura

JSON Web Tokens (JWT) es un estándar abierto (RFC 7519) que define una forma compacta y autónoma para transmitir información de manera segura entre partes como un objeto JSON. Esta información puede ser verificada y confiable porque está firmada digitalmente.

### 3.3.1 Estructura de un JWT

Un JWT consiste en tres partes separadas por puntos (`.`):

1. **Header**: Indica el algoritmo usado para la firma.
2. **Payload**: Contiene las afirmaciones (claims) sobre la entidad (generalmente el usuario) y datos adicionales.
3. **Signature**: Verifica que el token no ha sido alterado.

### 3.3.2 Implementación en Node.js (Backend)

```typescript
// auth/jwt.service.ts
import jwt from 'jsonwebtoken';
import { User } from '../models/user.model';

// En una aplicación real, estos valores estarían en variables de entorno
const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key'; // Nunca uses un secreto hardcodeado en producción
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN || '1h';

export class JwtService {
  /**
   * Genera un token JWT para un usuario autenticado
   */
  static generateToken(user: User): string {
    const payload = {
      sub: user.id, // subject (a quien pertenece el token)
      email: user.email,
      role: user.role,
      iat: Date.now() / 1000, // issued at (cuándo se emitió)
    };

    return jwt.sign(payload, JWT_SECRET, {
      expiresIn: JWT_EXPIRES_IN,
    });
  }

  /**
   * Verifica un token JWT
   */
  static verifyToken(token: string): any {
    try {
      return jwt.verify(token, JWT_SECRET);
    } catch (error) {
      throw new Error('Token inválido o expirado');
    }
  }
}
```

### 3.3.3 Middleware de Autenticación

```typescript
// middleware/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { JwtService } from '../auth/jwt.service';

// Extendemos la interfaz Request para incluir el usuario
declare global {
  namespace Express {
    interface Request {
      user?: any;
    }
  }
}

export function authenticate(req: Request, res: Response, next: NextFunction) {
  try {
    // 1. Extraer token del header de autorización
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ message: 'No se proporcionó token de autenticación' });
    }
    
    const token = authHeader.split(' ')[1];
    
    // 2. Verificar token
    const payload = JwtService.verifyToken(token);
    
    // 3. Adjuntar información del usuario a la solicitud
    req.user = payload;
    
    // 4. Continuar con la siguiente función en la cadena
    next();
  } catch (error) {
    return res.status(401).json({ message: 'Token inválido o expirado' });
  }
}

// Middleware adicional para verificar roles (autorización)
export function authorize(roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ message: 'No autenticado' });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ message: 'No tienes permiso para acceder a este recurso' });
    }

    next();
  };
}
```

### 3.3.4 Uso en las Rutas API

```typescript
// routes/user.routes.ts
import express from 'express';
import { authenticate, authorize } from '../middleware/auth.middleware';
import * as userController from '../controllers/user.controller';

const router = express.Router();

// Rutas públicas
router.post('/login', userController.login);
router.post('/register', userController.register);

// Rutas protegidas - requieren autenticación
router.get('/profile', authenticate, userController.getProfile);

// Rutas protegidas con roles específicos
router.get('/admin/users', 
  authenticate, 
  authorize(['admin']), 
  userController.getAllUsers);

export default router;
```

### 3.3.5 Almacenamiento Seguro de JWT en el Frontend

```typescript
// services/auth.service.ts (Frontend - React / Next.js)
export class AuthService {
  // Almacena el token en localStorage (opción menos segura pero común)
  static saveToken(token: string): void {
    localStorage.setItem('auth_token', token);
  }

  // Mejor opción: almacenar en HttpOnly Cookie (requiere configuración del servidor)
  // En este caso, el servidor establecería la cookie en la respuesta HTTP
  
  static getToken(): string | null {
    return localStorage.getItem('auth_token');
  }
  
  static removeToken(): void {
    localStorage.removeItem('auth_token');
  }
  
  static isLoggedIn(): boolean {
    const token = this.getToken();
    if (!token) return false;
    
    // Decodificar token para verificar expiración
    // (sin verificar firma ya que eso ocurre en el servidor)
    try {
      const base64Url = token.split('.')[1];
      const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
      const payload = JSON.parse(window.atob(base64));
      
      // Verificar si el token ha expirado
      return payload.exp > Date.now() / 1000;
    } catch (error) {
      return false;
    }
  }
}
```

### 3.3.6 Uso en Componentes React

```jsx
// components/ProtectedRoute.tsx
import React from 'react';
import { Navigate } from 'react-router-dom';
import { AuthService } from '../services/auth.service';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRole?: string;
}

const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ children, requiredRole }) => {
  const isLoggedIn = AuthService.isLoggedIn();
  
  // Verificar si el usuario está autenticado
  if (!isLoggedIn) {
    return <Navigate to="/login" replace />;
  }
  
  // Si se especifica un rol requerido, verificarlo
  if (requiredRole) {
    // En una implementación real, verificaríamos el rol del usuario
    // Esto podría requerir decodificar el token o hacer una solicitud al servidor
    const userRole = 'user'; // Este valor debería venir del token o del estado de la aplicación
    
    if (requiredRole !== userRole) {
      return <Navigate to="/unauthorized" replace />;
    }
  }
  
  return <>{children}</>;
};

export default ProtectedRoute;
```

## 3.4 Encriptación y Hashing: Protegiendo Datos Sensibles

### 3.4.1 Hashing de Contraseñas con Bcrypt

```javascript
// services/password.service.js
const bcrypt = require('bcrypt');

class PasswordService {
  // Número de rondas para el algoritmo de hashing
  // Mayor número = más seguro, pero más lento
  static SALT_ROUNDS = 12;
  
  /**
   * Genera un hash para una contraseña en texto plano
   */
  static async hash(plainPassword) {
    return bcrypt.hash(plainPassword, this.SALT_ROUNDS);
  }
  
  /**
   * Verifica si una contraseña en texto plano coincide con un hash
   */
  static async verify(plainPassword, hashedPassword) {
    return bcrypt.compare(plainPassword, hashedPassword);
  }
}

module.exports = PasswordService;
```

### 3.4.2 Encriptación de Datos Sensibles

```javascript
// services/encryption.service.js
const crypto = require('crypto');

class EncryptionService {
  // Estos valores deberían estar en variables de entorno en una app real
  static ENCRYPTION_KEY = process.env.ENCRYPTION_KEY || '12345678901234567890123456789012'; // 32 bytes
  static IV_LENGTH = 16; // 16 bytes para AES
  static ALGORITHM = 'aes-256-cbc';
  
  /**
   * Encripta datos sensibles (ej. número de tarjeta, SSN)
   */
  static encrypt(text) {
    const iv = crypto.randomBytes(this.IV_LENGTH);
    const cipher = crypto.createCipheriv(
      this.ALGORITHM, 
      Buffer.from(this.ENCRYPTION_KEY), 
      iv
    );
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    // Devolvemos IV + datos encriptados para poder desencriptar después
    return `${iv.toString('hex')}:${encrypted}`;
  }
  
  /**
   * Desencripta datos previamente encriptados
   */
  static decrypt(text) {
    const textParts = text.split(':');
    const iv = Buffer.from(textParts[0], 'hex');
    const encryptedText = textParts[1];
    
    const decipher = crypto.createDecipheriv(
      this.ALGORITHM, 
      Buffer.from(this.ENCRYPTION_KEY), 
      iv
    );
    
    let decrypted = decipher.update(encryptedText, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}

module.exports = EncryptionService;
```

### 3.4.3 Uso en una Entidad de Usuario

```javascript
// models/User.js (con Mongoose)
const mongoose = require('mongoose');
const PasswordService = require('../services/password.service');
const EncryptionService = require('../services/encryption.service');

const userSchema = new mongoose.Schema({
  email: { 
    type: String, 
    required: true, 
    unique: true 
  },
  passwordHash: { 
    type: String, 
    required: true 
  },
  firstName: String,
  lastName: String,
  // Campo sensible que necesita ser encriptado/desencriptado
  socialSecurityNumber: {
    type: String,
    // Encriptar SSN antes de guardar
    set: function(val) {
      if (!val) return;
      // Guardamos la versión encriptada en la base de datos
      return EncryptionService.encrypt(val);
    },
    // Desencriptar SSN al acceder
    get: function(val) {
      if (!val) return;
      // Desencriptamos cuando se accede al valor
      return EncryptionService.decrypt(val);
    }
  },
  createdAt: { 
    type: Date, 
    default: Date.now 
  },
  lastLogin: Date
});

// Método pre-save para hacer hash de la contraseña
userSchema.pre('save', async function(next) {
  const user = this;
  
  // Solo hash la contraseña si ha sido modificada o es nueva
  if (!user.isModified('passwordHash')) return next();
  
  try {
    // Reemplaza la contraseña en texto plano con el hash
    user.passwordHash = await PasswordService.hash(user.passwordHash);
    next();
  } catch (error) {
    next(error);
  }
});

// Método para comparar contraseñas
userSchema.methods.comparePassword = async function(candidatePassword) {
  return PasswordService.verify(candidatePassword, this.passwordHash);
};

const User = mongoose.model('User', userSchema);

module.exports = User;
```

## 3.5 Lista de Verificación de Seguridad para Despliegue

Antes de desplegar una aplicación a producción, es fundamental realizar una serie de verificaciones de seguridad. Esta lista no es exhaustiva, pero cubre puntos clave:

### 3.5.1 Configuración del Servidor
- [ ] Todos los secretos (API keys, contraseñas) están en variables de entorno, no en el código
- [ ] Firewall configurado para exponer solo los puertos necesarios (80, 443)
- [ ] Implementado HTTPS con certificados válidos
- [ ] Configuradas cabeceras de seguridad (HSTS, CSP, X-Content-Type-Options, etc.)
- [ ] Deshabilitados listados de directorios
- [ ] Servidor configurado para mostrar páginas de error personalizadas (no información de debug)

### 3.5.2 Aplicación
- [ ] Input validation implementada en todas las entradas de usuario
- [ ] Autenticación y autorización probadas exhaustivamente
- [ ] Las contraseñas se almacenan con algoritmos de hashing seguros (bcrypt, Argon2)
- [ ] Implementado rate limiting para prevenir ataques de fuerza bruta
- [ ] Habilitado CSRF protection en formularios
- [ ] Configurada gestión segura de sesiones (timeout, HttpOnly, Secure flags)
- [ ] Las APIs solo exponen los datos mínimos necesarios

### 3.5.3 Base de Datos
- [ ] Credenciales seguras y únicas para la base de datos
- [ ] Base de datos no accesible directamente desde Internet
- [ ] Privilegios mínimos para el usuario de aplicación
- [ ] Consultas parametrizadas o ORM utilizado en lugar de concatenación de strings
- [ ] Backups regulares configurados y probados

### 3.5.4 Monitorización y Respuesta
- [ ] Logging de eventos de seguridad configurado
- [ ] Sistema de alertas implementado para actividades sospechosas
- [ ] Plan de respuesta a incidentes documentado
- [ ] Escaneo regular de vulnerabilidades programado
