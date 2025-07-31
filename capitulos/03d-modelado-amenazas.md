# Capítulo 3: Seguridad en el Desarrollo de Software (Continuación)

## 3.4 Modelado de Amenazas y Principios de Seguridad

### 3.4.1 Modelado de Amenazas: Enfoque Proactivo

El modelado de amenazas es un enfoque estructurado para identificar, cuantificar y abordar los riesgos de seguridad asociados con una aplicación. Este proceso debe realizarse en las etapas iniciales del desarrollo, idealmente durante la fase de diseño, para asegurar que las medidas de seguridad se integren desde el principio.

#### El Modelo STRIDE

STRIDE es uno de los modelos más utilizados para la identificación sistemática de amenazas, creado por Microsoft. Cada letra representa una categoría de amenaza:

| Letra | Amenaza | Descripción | Ejemplos | Contramedidas |
|-------|---------|-------------|----------|---------------|
| **S** | **S**poofing (Suplantación) | Hacerse pasar por otra entidad | Phishing, falsificación de tokens | Autenticación fuerte, MFA |
| **T** | **T**ampering (Manipulación) | Modificación maliciosa de datos | Alteración de datos en tránsito | Integridad de datos, firmas digitales |
| **R** | **R**epudiation (Repudio) | Negar haber realizado una acción | Usuario que niega haber hecho una transacción | Logging seguro, firmas digitales |
| **I** | **I**nformation Disclosure (Divulgación de Información) | Exposición de información sensible | Fugas de datos, error logs con información sensible | Cifrado, control de acceso, sanitización de salidas |
| **D** | **D**enial of Service (Denegación de Servicio) | Hacer un sistema inaccesible | Ataques DDoS, consumo excesivo de recursos | Rate limiting, escalado automático, arquitectura resiliente |
| **E** | **E**levation of Privilege (Elevación de Privilegios) | Obtener más permisos de los asignados | Vulnerabilidades que permiten acceso administrativo | Principio de mínimo privilegio, segregación de funciones |

#### Implementación Práctica del Modelado STRIDE

```typescript
// Ejemplo de diagrama para modelado de amenazas usando mermaid-js
// Este código podría utilizarse para generar un diagrama de flujo de datos
// que posteriormente se analizaría con STRIDE

/**
 * graph TD
 *   U[Usuario] -->|Envía credenciales| A[API de Autenticación]
 *   A -->|Verifica| DB[(Base de Datos)]
 *   A -->|Genera token| U
 *   U -->|Solicita con token| R[API de Recursos]
 *   R -->|Verifica token| A
 *   R -->|Consulta datos| DB
 *   R -->|Devuelve datos| U
 * 
 * subgraph "Zona de Confianza"
 *   A
 *   DB
 *   R
 * end
 */

// Implementación de contramedidas para amenazas identificadas

// S: Suplantación - Implementar autenticación robusta
class AuthenticationService {
  async authenticate(username: string, password: string, mfaCode?: string) {
    // Verificar credenciales contra base de datos
    const user = await this.userRepository.findByUsername(username);
    if (!user) return { success: false, message: 'Usuario no encontrado' };
    
    // Verificar contraseña con bcrypt (nunca usar comparación directa)
    const isPasswordValid = await bcrypt.compare(password, user.passwordHash);
    if (!isPasswordValid) {
      // Registrar intento fallido (para detectar ataques)
      await this.logFailedAttempt(username, 'INVALID_PASSWORD');
      return { success: false, message: 'Credenciales inválidas' };
    }
    
    // Si MFA está habilitado, verificar código
    if (user.mfaEnabled) {
      if (!mfaCode) return { success: false, message: 'Se requiere código MFA' };
      
      const isMfaValid = this.verifyMfaCode(user.mfaSecret, mfaCode);
      if (!isMfaValid) {
        await this.logFailedAttempt(username, 'INVALID_MFA');
        return { success: false, message: 'Código MFA inválido' };
      }
    }
    
    // Generar token con expiración corta
    const token = this.generateSecureToken(user);
    
    // Registrar inicio de sesión exitoso
    await this.logSuccessfulLogin(user.id);
    
    return {
      success: true,
      token,
      expiresIn: TOKEN_EXPIRATION_SECONDS
    };
  }
}

// T: Manipulación - Asegurar la integridad de los datos
function createSignedPayload(data: any, secret: string): SignedPayload {
  const payload = JSON.stringify(data);
  const timestamp = Date.now();
  const nonce = crypto.randomBytes(16).toString('hex');
  
  // Crear firma HMAC para verificar integridad
  const signature = crypto
    .createHmac('sha256', secret)
    .update(`${payload}${timestamp}${nonce}`)
    .digest('hex');
  
  return {
    payload,
    timestamp,
    nonce,
    signature
  };
}

function verifySignedPayload(signedPayload: SignedPayload, secret: string): boolean {
  const { payload, timestamp, nonce, signature } = signedPayload;
  
  // Verificar que no haya expirado
  const now = Date.now();
  if (now - timestamp > MAX_SIGNATURE_AGE_MS) {
    return false; // Expirado
  }
  
  // Recalcular firma para verificar
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(`${payload}${timestamp}${nonce}`)
    .digest('hex');
  
  // Comparación segura contra timing attacks
  return crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expectedSignature, 'hex')
  );
}

// R: Repudio - Implementar logging seguro
class SecureLogger {
  private readonly logger: Logger;
  
  constructor() {
    this.logger = this.initializeLogger();
  }
  
  private initializeLogger(): Logger {
    // Configurar logger seguro con tamper-evident logs
    return createLogger({
      level: process.env.LOG_LEVEL || 'info',
      format: combine(
        timestamp(),
        json(),
        // Añadir hash para detectar manipulación de logs
        format((info) => {
          const logWithoutHash = { ...info };
          delete logWithoutHash.hash;
          const serialized = JSON.stringify(logWithoutHash);
          info.hash = crypto
            .createHash('sha256')
            .update(serialized + process.env.LOG_SECRET)
            .digest('hex');
          return info;
        })()
      ),
      transports: [
        new transports.Console(),
        new transports.File({ filename: 'secure-app.log' })
      ]
    });
  }
  
  logSecurityEvent(userId: string, action: string, details: Record<string, any>) {
    this.logger.info({
      userId,
      action,
      details,
      timestamp: new Date().toISOString(),
      ip: getCurrentRequestIP(),
      userAgent: getCurrentUserAgent()
    });
  }
  
  // Método para verificar integridad de logs
  static verifyLogIntegrity(logEntry: string): boolean {
    try {
      const entry = JSON.parse(logEntry);
      const { hash, ...logWithoutHash } = entry;
      const serialized = JSON.stringify(logWithoutHash);
      const expectedHash = crypto
        .createHash('sha256')
        .update(serialized + process.env.LOG_SECRET)
        .digest('hex');
      
      return expectedHash === hash;
    } catch (error) {
      return false;
    }
  }
}

// I: Divulgación de Información - Control de acceso granular
class ResourceController {
  async getResource(resourceId: string, user: AuthenticatedUser) {
    // Verificar existencia del recurso
    const resource = await this.resourceRepository.findById(resourceId);
    if (!resource) {
      throw new NotFoundError('Recurso no encontrado');
    }
    
    // Verificar permisos con sistema RBAC o ABAC
    const canAccess = await this.accessControlService.hasPermission(
      user, 
      'read', 
      resource
    );
    
    if (!canAccess) {
      // Importante: Loggear intentos de acceso no autorizados
      this.secureLogger.logSecurityEvent(
        user.id, 
        'UNAUTHORIZED_ACCESS_ATTEMPT', 
        { resourceId, resourceType: resource.type }
      );
      
      throw new ForbiddenError('No tiene permisos para acceder a este recurso');
    }
    
    // Filtrar información sensible según el rol del usuario
    const sanitizedResource = this.resourceSanitizer.sanitizeForUser(
      resource, 
      user.roles
    );
    
    // Registrar acceso exitoso
    this.secureLogger.logSecurityEvent(
      user.id, 
      'RESOURCE_ACCESS', 
      { resourceId, resourceType: resource.type }
    );
    
    return sanitizedResource;
  }
}

// D: Denegación de Servicio - Implementar rate limiting y timeout
class RateLimitingMiddleware {
  constructor(
    private readonly redis: Redis,
    private readonly options: {
      windowMs: number;
      maxRequests: number;
      keyGenerator: (req: Request) => string;
    }
  ) {}
  
  middleware() {
    return async (req: Request, res: Response, next: NextFunction) => {
      const key = `ratelimit:${this.options.keyGenerator(req)}`;
      
      try {
        // Obtener contador actual
        const currentCount = await this.redis.get(key) || 0;
        
        if (Number(currentCount) >= this.options.maxRequests) {
          // Límite excedido
          return res.status(429).json({
            error: 'Too Many Requests',
            message: 'Has excedido el límite de solicitudes. Intenta más tarde.'
          });
        }
        
        // Incrementar contador
        if (currentCount === 0) {
          // Primer request en la ventana
          await this.redis.multi()
            .incr(key)
            .pexpire(key, this.options.windowMs)
            .exec();
        } else {
          // Incrementar contador existente
          await this.redis.incr(key);
        }
        
        // Añadir headers de límite
        res.setHeader('X-RateLimit-Limit', this.options.maxRequests);
        res.setHeader('X-RateLimit-Remaining', this.options.maxRequests - Number(currentCount) - 1);
        
        next();
      } catch (error) {
        // En caso de error, permitir la solicitud
        console.error('Error en rate limiting:', error);
        next();
      }
    };
  }
}

// E: Elevación de Privilegios - Implementar principio de mínimo privilegio
class AdminController {
  async promoteUserToAdmin(userId: string, requestingAdmin: AuthenticatedUser) {
    // Verificar que el solicitante tiene permisos de super-admin
    if (!requestingAdmin.roles.includes('SUPER_ADMIN')) {
      this.secureLogger.logSecurityEvent(
        requestingAdmin.id,
        'PRIVILEGE_ESCALATION_ATTEMPT',
        { targetUserId: userId, attemptedAction: 'PROMOTE_TO_ADMIN' }
      );
      
      throw new ForbiddenError('No tienes permisos para promover usuarios a administradores');
    }
    
    // Verificar que existen procesos de aprobación múltiple para acciones críticas
    const pendingApproval = await this.approvalRepository.create({
      type: 'ROLE_CHANGE',
      requestedBy: requestingAdmin.id,
      targetUser: userId,
      targetRole: 'ADMIN',
      status: 'PENDING',
      createdAt: new Date()
    });
    
    // Notificar a otros super-admins para aprobación
    await this.notificationService.notifySuperAdmins(
      'APPROVAL_REQUIRED',
      `Aprobación requerida para promover a ${userId} a rol de ADMIN`,
      pendingApproval.id
    );
    
    return {
      message: 'Solicitud de promoción enviada para aprobación',
      approvalId: pendingApproval.id
    };
  }
}
```

### 3.4.2 Principios Fundamentales de Seguridad por Defecto

La seguridad no debe ser una característica adicional, sino un elemento integrado en cada aspecto del desarrollo de software.

#### Secure Defaults (Seguridad por Defecto)

Este principio establece que las configuraciones predeterminadas deben ser las más seguras posibles. Los usuarios y desarrolladores deberían tener que hacer un esfuerzo consciente para debilitar la seguridad, no para fortalecerla.

```typescript
// Ejemplo de configuración segura por defecto para cookies en Express
import session from 'express-session';

app.use(session({
  secret: process.env.SESSION_SECRET,
  name: 'app_session', // No usar el nombre predeterminado
  cookie: {
    httpOnly: true,     // No accesible mediante JavaScript (por defecto)
    secure: true,       // Solo HTTPS (por defecto en producción)
    sameSite: 'strict', // Restricción de envío cross-site (por defecto)
    maxAge: 3600000     // Expiración corta de 1 hora (por defecto)
  },
  resave: false,
  saveUninitialized: false,
  // Almacenamiento seguro por defecto (no MemoryStore)
  store: new RedisStore({
    client: redisClient,
    prefix: 'session:'
  })
}));

// Ejemplo de configuración segura de Helmet en Next.js
// En next.config.js
const securityHeaders = [
  // X-DNS-Prefetch-Control: controlar prefetching de DNS
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on'
  },
  // X-Frame-Options: prevenir clickjacking
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN'
  },
  // X-Content-Type-Options: prevenir MIME sniffing
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  // Referrer-Policy: controlar información del referrer
  {
    key: 'Referrer-Policy',
    value: 'origin-when-cross-origin'
  },
  // Permissions-Policy: limitar características del navegador
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=(), interest-cohort=()'
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: '/:path*', // Aplicar a todas las rutas
        headers: securityHeaders,
      },
    ];
  },
};
```

#### Least Privilege (Mínimo Privilegio)

Este principio dicta que cualquier usuario, proceso o sistema debe tener solamente los privilegios mínimos necesarios para desempeñar su función. La restricción de acceso reduce la superficie de ataque y minimiza el daño potencial de brechas de seguridad.

```typescript
// Ejemplo de implementación del principio de menor privilegio en una API REST

// 1. Definición de permisos específicos y granulares
enum Permission {
  USER_READ = 'user:read',
  USER_CREATE = 'user:create',
  USER_UPDATE = 'user:update',
  USER_DELETE = 'user:delete',
  
  POST_READ = 'post:read',
  POST_CREATE = 'post:create',
  POST_UPDATE = 'post:update',
  POST_DELETE = 'post:delete',
  
  COMMENT_READ = 'comment:read',
  COMMENT_CREATE = 'comment:create',
  COMMENT_UPDATE = 'comment:update',
  COMMENT_DELETE = 'comment:delete',
}

// 2. Definición de roles con permisos específicos
const ROLES = {
  PUBLIC: [
    Permission.POST_READ,
    Permission.COMMENT_READ,
  ],
  USER: [
    Permission.POST_READ,
    Permission.POST_CREATE,
    Permission.COMMENT_READ,
    Permission.COMMENT_CREATE,
    Permission.USER_READ,
    // Usuarios pueden actualizar y eliminar sólo SUS PROPIOS recursos
  ],
  MODERATOR: [
    Permission.POST_READ,
    Permission.POST_UPDATE,
    Permission.COMMENT_READ,
    Permission.COMMENT_UPDATE,
    Permission.COMMENT_DELETE,
    Permission.USER_READ,
  ],
  ADMIN: [
    // Los administradores tienen todos los permisos excepto eliminar usuarios
    ...Object.values(Permission).filter(p => p !== Permission.USER_DELETE),
  ],
  SUPER_ADMIN: [
    // Super admins tienen todos los permisos
    ...Object.values(Permission),
  ],
};

// 3. Middleware para verificar permisos
function requirePermission(permission: Permission) {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user; // Usuario autenticado de middleware previo
    
    if (!user) {
      return res.status(401).json({ message: 'No autenticado' });
    }
    
    // Verificar si el usuario tiene el permiso requerido
    const hasPermission = user.permissions.includes(permission);
    
    if (!hasPermission) {
      // Registrar intento de acceso no autorizado
      secureLogger.log({
        level: 'warn',
        message: 'Intento de acceso no autorizado',
        userId: user.id,
        requiredPermission: permission,
        endpoint: req.originalUrl,
        method: req.method,
      });
      
      return res.status(403).json({ message: 'Permiso denegado' });
    }
    
    next();
  };
}

// 4. Middleware para restricciones de propiedad (usuarios pueden editar solo sus propios recursos)
function requireOwnership(resourceType: 'post' | 'comment' | 'user') {
  return async (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;
    const resourceId = req.params.id;
    
    if (!user || !resourceId) {
      return res.status(401).json({ message: 'No autenticado' });
    }
    
    try {
      let isOwner = false;
      
      switch (resourceType) {
        case 'post':
          const post = await PostModel.findById(resourceId);
          isOwner = post?.authorId === user.id;
          break;
        case 'comment':
          const comment = await CommentModel.findById(resourceId);
          isOwner = comment?.authorId === user.id;
          break;
        case 'user':
          // Un usuario solo puede modificarse a sí mismo
          isOwner = resourceId === user.id;
          break;
      }
      
      // Si no es dueño y no tiene permisos de administrador
      if (!isOwner && !user.permissions.includes(`${resourceType}:admin`)) {
        secureLogger.log({
          level: 'warn',
          message: 'Intento de modificación de recurso ajeno',
          userId: user.id,
          resourceType,
          resourceId,
          endpoint: req.originalUrl,
          method: req.method,
        });
        
        return res.status(403).json({ message: 'No eres propietario de este recurso' });
      }
      
      next();
    } catch (error) {
      next(error);
    }
  };
}

// 5. Aplicación en rutas de la API
router.get('/users', requirePermission(Permission.USER_READ), userController.listUsers);
router.post('/users', requirePermission(Permission.USER_CREATE), userController.createUser);
router.put('/users/:id', 
  requirePermission(Permission.USER_UPDATE), 
  requireOwnership('user'), 
  userController.updateUser
);
router.delete('/users/:id',
  requirePermission(Permission.USER_DELETE),
  // Solo super admins pueden eliminar usuarios, no se requiere verificación de propiedad
  userController.deleteUser
);

// Para publicaciones, permitir edición solo al autor o moderadores/admins
router.put('/posts/:id',
  requirePermission(Permission.POST_UPDATE),
  async (req, res, next) => {
    // Los moderadores y admins pueden saltarse la verificación de propiedad
    if (req.user.roles.some(role => ['MODERATOR', 'ADMIN', 'SUPER_ADMIN'].includes(role))) {
      return next();
    }
    // Para usuarios regulares, verificar propiedad
    requireOwnership('post')(req, res, next);
  },
  postController.updatePost
);
```

### 3.4.3 Integración en el Ciclo de Desarrollo

Para ser efectivo, el modelado de amenazas y los principios de seguridad deben integrarse en todo el ciclo de desarrollo:

1. **Fase de Diseño**: Realizar modelado de amenazas STRIDE para identificar riesgos temprano.
2. **Fase de Desarrollo**: Aplicar principios de Secure by Default y Least Privilege.
3. **Fase de Pruebas**: Incluir pruebas de seguridad específicas para las amenazas identificadas.
4. **Fase de Despliegue**: Verificar configuraciones seguras y aplicar hardening.
5. **Fase de Mantenimiento**: Monitoreo continuo, actualizaciones de seguridad y revisión periódica de amenazas.

Este enfoque proactivo y continuo asegura que la seguridad no sea un componente añadido al final, sino una consideración integral a lo largo de todo el proceso de desarrollo de software.
