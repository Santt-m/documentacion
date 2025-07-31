# 6.4 Autenticación y Autorización

La seguridad es un aspecto fundamental de cualquier aplicación backend moderna. Este capítulo explora los enfoques y técnicas para implementar autenticación y autorización efectivas.

## 6.4.1 Conceptos Fundamentales

### Diferencia entre Autenticación y Autorización

- **Autenticación**: Verifica la identidad del usuario ("¿Quién eres?")
- **Autorización**: Determina qué recursos puede acceder el usuario ("¿Qué puedes hacer?")

### Factores de Autenticación

- **Algo que sabes**: Contraseñas, PIN
- **Algo que tienes**: Token físico, teléfono móvil
- **Algo que eres**: Huella dactilar, reconocimiento facial
- **Ubicación**: Geolocalización
- **Comportamiento**: Patrones de uso, biometría conductual

## 6.4.2 Sistemas de Autenticación

### Autenticación Basada en JWT

JSON Web Tokens (JWT) es un estándar abierto (RFC 7519) que define una forma compacta y autónoma para transmitir información de manera segura entre partes como un objeto JSON.

**Estructura de un JWT**:
```
header.payload.signature
```

**Ejemplo de implementación con Node.js**:

```javascript
// auth.service.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const JWT_SECRET = process.env.JWT_SECRET || 'tu-clave-secreta'; // En producción, siempre usar variable de entorno
const JWT_EXPIRES_IN = '24h';

class AuthService {
  // Generar token después de autenticación exitosa
  static generateToken(user) {
    const payload = {
      sub: user.id,
      email: user.email,
      role: user.role,
      iat: Math.floor(Date.now() / 1000)
    };
    
    return jwt.sign(payload, JWT_SECRET, { expiresIn: JWT_EXPIRES_IN });
  }
  
  // Verificar token
  static verifyToken(token) {
    try {
      return jwt.verify(token, JWT_SECRET);
    } catch (error) {
      throw new Error('Token inválido o expirado');
    }
  }
  
  // Autenticar usuario
  static async authenticate(email, password, userRepository) {
    // Buscar usuario por email
    const user = await userRepository.findByEmail(email);
    if (!user) {
      throw new Error('Credenciales inválidas');
    }
    
    // Verificar contraseña
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      throw new Error('Credenciales inválidas');
    }
    
    // Generar token si las credenciales son válidas
    return {
      user: {
        id: user.id,
        email: user.email,
        role: user.role
      },
      token: this.generateToken(user)
    };
  }
}

module.exports = AuthService;
```

**Middleware de autenticación**:

```javascript
// auth.middleware.js
const AuthService = require('../services/auth.service');

function authenticateJWT(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ 
      error: 'Unauthorized', 
      message: 'Authentication token required' 
    });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    // Verificar y decodificar token
    const decodedToken = AuthService.verifyToken(token);
    
    // Agregar payload decodificado a la solicitud para uso posterior
    req.user = decodedToken;
    
    next();
  } catch (error) {
    return res.status(401).json({ 
      error: 'Unauthorized', 
      message: error.message 
    });
  }
}

module.exports = { authenticateJWT };
```

**Uso en rutas**:

```javascript
// routes/user.routes.js
const express = require('express');
const router = express.Router();
const { authenticateJWT } = require('../middleware/auth.middleware');
const UserController = require('../controllers/user.controller');

// Rutas públicas
router.post('/login', UserController.login);
router.post('/register', UserController.register);

// Rutas protegidas
router.get('/profile', authenticateJWT, UserController.getProfile);
router.put('/profile', authenticateJWT, UserController.updateProfile);

module.exports = router;
```

### OAuth 2.0 y OpenID Connect

**OAuth 2.0** es un framework de autorización que permite a una aplicación obtener acceso limitado a una cuenta de usuario en un servicio HTTP.

**OpenID Connect** es una capa de identidad construida sobre OAuth 2.0 que permite a los clientes verificar la identidad del usuario.

**Flujos comunes de OAuth 2.0**:

1. **Código de Autorización**: Para aplicaciones con servidor
2. **Implícito**: Para aplicaciones de una sola página (SPA)
3. **Credenciales de cliente**: Para acceso API a API
4. **Contraseña del propietario del recurso**: Para aplicaciones de confianza

**Ejemplo de integración con Passport.js**:

```javascript
// oauth.js
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const User = require('../models/user.model');

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: "/auth/google/callback"
  },
  async function(accessToken, refreshToken, profile, done) {
    try {
      // Buscar usuario existente
      let user = await User.findOne({ googleId: profile.id });
      
      if (user) {
        return done(null, user);
      }
      
      // Crear nuevo usuario si no existe
      user = await User.create({
        googleId: profile.id,
        email: profile.emails[0].value,
        name: profile.displayName,
        avatar: profile.photos[0].value
      });
      
      return done(null, user);
    } catch (error) {
      return done(error, null);
    }
  }
));

passport.serializeUser((user, done) => {
  done(null, user.id);
});

passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (error) {
    done(error, null);
  }
});
```

**Configuración de rutas OAuth**:

```javascript
// auth.routes.js
const express = require('express');
const router = express.Router();
const passport = require('passport');
const AuthController = require('../controllers/auth.controller');

// Rutas de autenticación Google
router.get('/google',
  passport.authenticate('google', { scope: ['profile', 'email'] }));

router.get('/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    // Autenticación exitosa, redirigir
    res.redirect('/dashboard');
  }
);

// Otras rutas de autenticación
router.post('/login', AuthController.login);
router.post('/register', AuthController.register);
router.get('/logout', (req, res) => {
  req.logout();
  res.redirect('/');
});

module.exports = router;
```

### Autenticación de Dos Factores (2FA)

La autenticación de dos factores añade una capa adicional de seguridad requiriendo dos métodos de autenticación.

**Implementación básica con TOTP**:

```javascript
// two-factor.service.js
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');

class TwoFactorService {
  // Generar secreto para 2FA
  static generateSecret(username) {
    const secret = speakeasy.generateSecret({
      name: `MyApp:${username}`
    });
    
    return {
      otpauthUrl: secret.otpauth_url,
      base32: secret.base32
    };
  }
  
  // Generar QR code para la aplicación autenticadora
  static async generateQRCode(otpauthUrl) {
    try {
      const qrCodeUrl = await QRCode.toDataURL(otpauthUrl);
      return qrCodeUrl;
    } catch (error) {
      throw new Error('Error generating QR code');
    }
  }
  
  // Verificar token TOTP
  static verifyToken(token, secret) {
    return speakeasy.totp.verify({
      secret: secret,
      encoding: 'base32',
      token: token,
      window: 1 // Permite ventana de 1 intervalo para manejar desincronización
    });
  }
}

module.exports = TwoFactorService;
```

**Integración con el proceso de autenticación**:

```javascript
// auth.controller.js
const AuthService = require('../services/auth.service');
const TwoFactorService = require('../services/two-factor.service');
const User = require('../models/user.model');

exports.setup2FA = async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    
    // Generar secreto para 2FA
    const { otpauthUrl, base32 } = TwoFactorService.generateSecret(user.email);
    
    // Guardar secreto temporalmente (hasta verificación)
    user.tempTwoFactorSecret = base32;
    await user.save();
    
    // Generar código QR
    const qrCode = await TwoFactorService.generateQRCode(otpauthUrl);
    
    res.json({ 
      message: 'Two-factor authentication setup initiated',
      qrCode
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.verify2FA = async (req, res) => {
  try {
    const { token } = req.body;
    const user = await User.findById(req.user.id);
    
    // Verificar token con el secreto temporal
    const isValid = TwoFactorService.verifyToken(
      token, 
      user.tempTwoFactorSecret
    );
    
    if (!isValid) {
      return res.status(400).json({ message: 'Invalid token' });
    }
    
    // Activar 2FA y guardar secreto permanentemente
    user.isTwoFactorEnabled = true;
    user.twoFactorSecret = user.tempTwoFactorSecret;
    user.tempTwoFactorSecret = undefined;
    await user.save();
    
    res.json({ message: 'Two-factor authentication enabled successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.login = async (req, res) => {
  try {
    const { email, password, token } = req.body;
    
    // Autenticar con credenciales
    const user = await User.findOne({ email });
    if (!user || !(await user.comparePassword(password))) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    // Si 2FA está habilitado, verificar token
    if (user.isTwoFactorEnabled) {
      if (!token) {
        return res.status(401).json({ 
          message: '2FA token required',
          requires2FA: true
        });
      }
      
      const isValidToken = TwoFactorService.verifyToken(
        token, 
        user.twoFactorSecret
      );
      
      if (!isValidToken) {
        return res.status(401).json({ message: 'Invalid 2FA token' });
      }
    }
    
    // Generar JWT
    const authToken = AuthService.generateToken(user);
    
    res.json({
      message: 'Authentication successful',
      token: authToken,
      user: {
        id: user._id,
        email: user.email,
        name: user.name
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

## 6.4.3 Autorización

### Control de Acceso Basado en Roles (RBAC)

RBAC asigna permisos a roles, y los roles a usuarios.

**Implementación básica**:

```javascript
// rbac.middleware.js
function checkRole(roles) {
  return (req, res, next) => {
    // req.user se establece en el middleware authenticateJWT
    if (!req.user) {
      return res.status(401).json({ 
        error: 'Unauthorized', 
        message: 'Authentication required' 
      });
    }
    
    // Verificar si el usuario tiene el rol requerido
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ 
        error: 'Forbidden', 
        message: 'Insufficient privileges' 
      });
    }
    
    next();
  };
}

module.exports = { checkRole };
```

**Uso en rutas**:

```javascript
// admin.routes.js
const express = require('express');
const router = express.Router();
const { authenticateJWT } = require('../middleware/auth.middleware');
const { checkRole } = require('../middleware/rbac.middleware');
const AdminController = require('../controllers/admin.controller');

// Todas las rutas requieren autenticación y rol de admin
router.use(authenticateJWT);
router.use(checkRole(['admin']));

router.get('/users', AdminController.listUsers);
router.get('/users/:id', AdminController.getUserDetails);
router.put('/users/:id', AdminController.updateUser);
router.delete('/users/:id', AdminController.deleteUser);

module.exports = router;
```

### Control de Acceso Basado en Atributos (ABAC)

ABAC es más flexible que RBAC, permitiendo decisiones de acceso basadas en atributos del usuario, recurso, acción y contexto.

**Implementación con políticas de acceso**:

```javascript
// policy.service.js
class PolicyService {
  static canAccessResource(user, resource, action, context = {}) {
    // Políticas para entidad "article"
    if (resource.type === 'article') {
      switch (action) {
        case 'read':
          // Cualquier usuario puede leer artículos publicados
          if (resource.isPublished) return true;
          
          // Los autores pueden leer sus artículos no publicados
          if (resource.authorId === user.id) return true;
          
          // Administradores y editores pueden leer cualquier artículo
          if (['admin', 'editor'].includes(user.role)) return true;
          
          return false;
          
        case 'update':
          // El autor puede actualizar dentro de 24 horas de creación
          if (resource.authorId === user.id && 
              Date.now() - resource.createdAt < 24 * 60 * 60 * 1000) {
            return true;
          }
          
          // Editores pueden actualizar cualquier artículo
          if (user.role === 'editor') return true;
          
          // Administradores pueden actualizar cualquier cosa
          if (user.role === 'admin') return true;
          
          return false;
          
        case 'delete':
          // Solo administradores pueden eliminar artículos
          return user.role === 'admin';
          
        default:
          return false;
      }
    }
    
    // Políticas para otros tipos de recursos...
    
    return false;
  }
}

module.exports = PolicyService;
```

**Middleware para verificar políticas**:

```javascript
// policy.middleware.js
const PolicyService = require('../services/policy.service');

function checkPolicy(resourceType, actionType) {
  return async (req, res, next) => {
    try {
      // req.user se establece en el middleware authenticateJWT
      if (!req.user) {
        return res.status(401).json({ 
          error: 'Unauthorized', 
          message: 'Authentication required' 
        });
      }
      
      // Obtener el recurso según su tipo
      let resource;
      
      switch (resourceType) {
        case 'article':
          // Obtener artículo desde base de datos
          resource = await Article.findById(req.params.id);
          if (!resource) {
            return res.status(404).json({ message: 'Resource not found' });
          }
          break;
          
        // Otros tipos de recursos...
        
        default:
          return res.status(400).json({ message: 'Invalid resource type' });
      }
      
      // Verificar política de acceso
      const canAccess = PolicyService.canAccessResource(
        req.user,
        { ...resource.toObject(), type: resourceType },
        actionType,
        { ip: req.ip, time: new Date() }
      );
      
      if (!canAccess) {
        return res.status(403).json({ 
          error: 'Forbidden', 
          message: 'You do not have permission to perform this action' 
        });
      }
      
      // Adjuntar recurso a req para uso posterior
      req.resource = resource;
      next();
    } catch (error) {
      next(error);
    }
  };
}

module.exports = { checkPolicy };
```

**Uso en rutas**:

```javascript
// article.routes.js
const express = require('express');
const router = express.Router();
const { authenticateJWT } = require('../middleware/auth.middleware');
const { checkPolicy } = require('../middleware/policy.middleware');
const ArticleController = require('../controllers/article.controller');

// Rutas públicas
router.get('/', ArticleController.listPublishedArticles);
router.get('/:id', ArticleController.getPublishedArticle);

// Rutas protegidas con autenticación
router.get('/drafts', 
  authenticateJWT, 
  ArticleController.listDrafts);

router.get('/:id/edit', 
  authenticateJWT, 
  checkPolicy('article', 'update'),
  ArticleController.getArticleForEditing);

router.put('/:id', 
  authenticateJWT, 
  checkPolicy('article', 'update'),
  ArticleController.updateArticle);

router.delete('/:id', 
  authenticateJWT, 
  checkPolicy('article', 'delete'),
  ArticleController.deleteArticle);

router.post('/', 
  authenticateJWT, 
  checkPolicy('article', 'create'),
  ArticleController.createArticle);

module.exports = router;
```

## 6.4.4 Buenas Prácticas de Seguridad

### Almacenamiento Seguro de Contraseñas

- **Nunca almacenar contraseñas en texto plano**
- **Usar algoritmos de hash modernos** (bcrypt, Argon2, scrypt)
- **Salting** para proteger contra ataques de diccionario y rainbow tables
- **Factor de trabajo/costo** ajustado para equilibrar seguridad y rendimiento

**Ejemplo con bcrypt**:

```javascript
// password.service.js
const bcrypt = require('bcrypt');

class PasswordService {
  static async hash(plainPassword, saltRounds = 12) {
    return bcrypt.hash(plainPassword, saltRounds);
  }
  
  static async verify(plainPassword, hashedPassword) {
    return bcrypt.compare(plainPassword, hashedPassword);
  }
}

module.exports = PasswordService;
```

### Protección contra Ataques Comunes

#### Rate Limiting

```javascript
// rate-limit.middleware.js
const rateLimit = require('express-rate-limit');

// Limitar intentos de login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 5, // 5 intentos
  message: {
    error: 'Too many login attempts',
    message: 'Too many failed login attempts, please try again later.'
  },
  standardHeaders: true,
  legacyHeaders: false,
});

// Limitar solicitudes a la API general
const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minuto
  max: 60, // 60 solicitudes por minuto
  message: {
    error: 'Too many requests',
    message: 'Too many requests, please try again later.'
  },
  standardHeaders: true,
  legacyHeaders: false,
});

module.exports = { loginLimiter, apiLimiter };
```

#### CSRF Protection

```javascript
// app.js
const express = require('express');
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser());

// Configurar CSRF protection
const csrfProtection = csrf({ cookie: true });

// Aplicar a rutas que manejan formularios
app.get('/form', csrfProtection, (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

app.post('/process', csrfProtection, (req, res) => {
  // El middleware csrf verificará automáticamente el token
  // Si es inválido, generará un error
  res.send('Form processed');
});

// Middleware de manejo de errores para CSRF
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).json({
      error: 'CSRF error',
      message: 'Form has been tampered with'
    });
  }
  next(err);
});
```

### Principio de Privilegio Mínimo

Siempre otorgar los mínimos permisos necesarios para realizar una tarea.

```javascript
// Ejemplo de diferentes niveles de autorización por endpoint
router.get('/users',
  authenticateJWT,
  checkRole(['admin']), // Solo administradores
  UserController.listAllUsers
);

router.get('/users/:id',
  authenticateJWT,
  (req, res, next) => {
    // El usuario puede ver su propio perfil, o un admin puede ver cualquiera
    if (req.params.id === req.user.id || req.user.role === 'admin') {
      next();
    } else {
      res.status(403).json({ message: 'Forbidden' });
    }
  },
  UserController.getUserProfile
);
```

### Logging y Auditoría

```javascript
// audit.middleware.js
const AuditLog = require('../models/audit-log.model');

async function auditLog(req, res, next) {
  // Capturar el tiempo de inicio
  const startTime = Date.now();
  
  // Almacenar detalles originales de la respuesta
  const originalSend = res.send;
  
  // Sobrescribir método send para capturar la respuesta
  res.send = function(body) {
    res.body = body;
    return originalSend.call(this, body);
  };
  
  // Continuar con la solicitud
  next();
  
  // Cuando la respuesta completa, registrar la actividad
  res.on('finish', async () => {
    try {
      const endTime = Date.now();
      const duration = endTime - startTime;
      
      // Crear entrada de auditoría
      await AuditLog.create({
        userId: req.user?.id || null,
        action: `${req.method} ${req.originalUrl}`,
        resource: req.originalUrl,
        status: res.statusCode,
        ip: req.ip,
        userAgent: req.get('User-Agent'),
        timestamp: new Date(),
        duration: duration,
        details: {
          method: req.method,
          query: req.query,
          body: req.method !== 'GET' ? sanitizeRequestBody(req.body) : undefined,
          statusCode: res.statusCode
        }
      });
    } catch (error) {
      console.error('Error logging audit trail:', error);
    }
  });
}

// Eliminar datos sensibles del cuerpo de la solicitud
function sanitizeRequestBody(body) {
  if (!body) return undefined;
  
  const sanitized = { ...body };
  
  // Eliminar campos sensibles
  const sensitiveFields = ['password', 'passwordConfirm', 'token', 'credit_card', 'ssn'];
  
  for (const field of sensitiveFields) {
    if (sanitized[field]) {
      sanitized[field] = '[REDACTED]';
    }
  }
  
  return sanitized;
}

module.exports = { auditLog };
```
