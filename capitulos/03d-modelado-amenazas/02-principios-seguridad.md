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

#### Defensa en Profundidad

Este principio consiste en implementar múltiples capas de seguridad para que si una falla, otras sigan protegiendo el sistema.

```typescript
// Ejemplo de implementación de defensa en profundidad para una API de transacciones

// 1. Primera capa: Validación de entrada
function validateTransactionRequest(req: TransactionRequest): ValidationResult {
  const schema = z.object({
    amount: z.number().positive(),
    currency: z.string().length(3),
    accountId: z.string().uuid(),
    description: z.string().max(255).optional(),
    // Más validaciones...
  });
  
  const result = schema.safeParse(req);
  if (!result.success) {
    return {
      valid: false,
      errors: result.error.issues
    };
  }
  
  return { valid: true };
}

// 2. Segunda capa: Control de acceso
async function checkTransactionPermissions(userId: string, accountId: string): Promise<boolean> {
  // Verificar en base de datos si el usuario tiene acceso a esta cuenta
  const permissions = await db.userAccountPermissions.findUnique({
    where: {
      userId_accountId: {
        userId,
        accountId
      }
    }
  });
  
  return permissions !== null && permissions.canInitiateTransactions;
}

// 3. Tercera capa: Límites de transacción
async function checkTransactionLimits(userId: string, amount: number): Promise<LimitCheckResult> {
  // Obtener límites configurados para el usuario
  const userLimits = await limitsService.getUserLimits(userId);
  
  // Obtener historial de transacciones recientes
  const recentTransactions = await transactionRepository.getRecentByUser(userId, {
    timeframe: '24h'
  });
  
  // Calcular total gastado en las últimas 24h
  const totalSpent = recentTransactions.reduce((sum, tx) => sum + tx.amount, 0);
  
  if (totalSpent + amount > userLimits.dailyLimit) {
    return {
      allowed: false,
      reason: 'DAILY_LIMIT_EXCEEDED',
      currentUsage: totalSpent,
      limit: userLimits.dailyLimit
    };
  }
  
  return { allowed: true };
}

// 4. Cuarta capa: Detección de fraude en tiempo real
async function performFraudCheck(transaction: Transaction): Promise<FraudCheckResult> {
  // Combinar múltiples señales para detectar patrones sospechosos
  const signals = await Promise.all([
    geoLocationService.checkIpLocation(transaction.clientIp),
    deviceFingerprintService.validateDevice(transaction.deviceFingerprint),
    behaviorAnalysisService.analyzeUserPattern(transaction.userId, transaction),
    recentActivityService.checkForUnusualPatterns(transaction.userId)
  ]);
  
  // Calcular puntuación de riesgo
  const riskScore = fraudScoringModel.calculateScore(signals, transaction);
  
  if (riskScore > FRAUD_THRESHOLD_HIGH) {
    return {
      approved: false,
      requiresVerification: true,
      riskScore,
      reasons: ['UNUSUAL_LOCATION', 'PATTERN_DEVIATION']
    };
  } else if (riskScore > FRAUD_THRESHOLD_MEDIUM) {
    return {
      approved: true,
      requiresVerification: true,
      riskScore,
      reasons: ['VELOCITY_CHECK']
    };
  }
  
  return {
    approved: true,
    requiresVerification: false,
    riskScore
  };
}

// 5. Quinta capa: Auditoría y monitoreo post-transacción
async function recordTransactionAudit(transaction: Transaction, outcome: TransactionOutcome): Promise<void> {
  // Crear registro de auditoría inmutable
  const auditEntry = {
    transactionId: transaction.id,
    timestamp: new Date().toISOString(),
    userId: transaction.userId,
    accountId: transaction.accountId,
    amount: transaction.amount,
    currency: transaction.currency,
    outcome: outcome.status,
    ipAddress: transaction.clientIp,
    deviceFingerprint: transaction.deviceFingerprint,
    geoLocation: transaction.geoLocation,
    processingTime: outcome.processingTime
  };
  
  // Firmar digitalmente para prevenir manipulación
  const signedEntry = auditService.signEntry(auditEntry);
  
  // Almacenar en sistema de auditoría
  await auditService.store(signedEntry);
  
  // Analizar en tiempo real para detección de anomalías
  await anomalyDetectionService.analyze(signedEntry);
}
```

#### Principle of Least Privilege (Principio de Mínimo Privilegio)

Cada componente, proceso o usuario debe operar utilizando el conjunto mínimo de privilegios necesarios para realizar su tarea.

```typescript
// Ejemplo de implementación del principio de mínimo privilegio en una API
import { NextFunction, Request, Response } from 'express';
import jwt from 'jsonwebtoken';

// Definición de roles con permisos específicos
const ROLES = {
  GUEST: {
    permissions: ['read:public']
  },
  USER: {
    permissions: ['read:public', 'read:own', 'write:own']
  },
  EDITOR: {
    permissions: ['read:public', 'read:own', 'write:own', 'read:all', 'write:submitted']
  },
  ADMIN: {
    permissions: ['read:public', 'read:own', 'write:own', 'read:all', 'write:all', 'delete:all']
  }
};

// Middleware para verificar permisos específicos
function requirePermission(permission: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      // Verificar token JWT
      const token = req.headers.authorization?.split(' ')[1];
      if (!token) {
        return res.status(401).json({ message: 'Se requiere autenticación' });
      }
      
      // Verificar y decodificar token
      const decoded = jwt.verify(token, process.env.JWT_SECRET!) as {
        userId: string;
        role: string;
      };
      
      // Obtener rol del usuario
      const userRole = ROLES[decoded.role as keyof typeof ROLES];
      if (!userRole) {
        return res.status(403).json({ message: 'Rol no válido' });
      }
      
      // Verificar si el rol tiene el permiso requerido
      if (!userRole.permissions.includes(permission)) {
        return res.status(403).json({ message: 'Permiso insuficiente' });
      }
      
      // Añadir información de usuario y permisos al request
      req.user = {
        id: decoded.userId,
        role: decoded.role,
        permissions: userRole.permissions
      };
      
      next();
    } catch (error) {
      return res.status(401).json({ message: 'Token no válido' });
    }
  };
}

// Ejemplo de uso en rutas con permisos específicos
router.get('/articles', requirePermission('read:public'), articleController.getPublicArticles);
router.get('/articles/all', requirePermission('read:all'), articleController.getAllArticles);
router.post('/articles', requirePermission('write:own'), articleController.createArticle);
router.delete('/articles/:id', requirePermission('delete:all'), articleController.deleteArticle);

// Implementación de permisos más granulares con contexto
function requireContextualPermission(permission: string, contextExtractor: (req: Request) => string) {
  return async (req: Request, res: Response, next: NextFunction) => {
    // Autenticación igual que antes...
    
    // Extraer el contexto (ej: ID del recurso)
    const contextId = contextExtractor(req);
    
    // Para permisos sobre recursos propios, verificar propiedad
    if (permission === 'write:own' || permission === 'read:own') {
      const isOwner = await resourceOwnershipService.checkOwnership(
        req.user.id,
        contextId
      );
      
      if (!isOwner && !req.user.permissions.includes(`${permission.split(':')[0]}:all`)) {
        return res.status(403).json({ message: 'No tienes permiso para este recurso' });
      }
    }
    
    next();
  };
}

// Ejemplo de uso contextual
router.put(
  '/articles/:id',
  requireContextualPermission('write:own', (req) => req.params.id),
  articleController.updateArticle
);
```

#### Fail Secure (Fallo Seguro)

Cuando un sistema falla, debe hacerlo de manera que no comprometa la seguridad. Es mejor denegar acceso que permitirlo accidentalmente.

```typescript
// Ejemplo de manejo de fallos seguros en autenticación
async function authenticateUser(username: string, password: string): Promise<AuthResult> {
  try {
    // Buscar usuario en la base de datos
    const user = await userRepository.findByUsername(username);
    
    // Si no existe, mismo flujo que si existe (para prevenir timing attacks)
    if (!user) {
      // Realizar una comparación de contraseña falsa para mantener tiempo constante
      await bcrypt.compare(password, DUMMY_PASSWORD_HASH);
      return {
        success: false,
        error: 'INVALID_CREDENTIALS',
        // No especificar que el usuario no existe
        message: 'Credenciales inválidas'
      };
    }
    
    // Verificar si la cuenta está bloqueada
    if (user.lockedUntil && user.lockedUntil > new Date()) {
      return {
        success: false,
        error: 'ACCOUNT_LOCKED',
        message: 'Cuenta temporalmente bloqueada. Intente más tarde.'
      };
    }
    
    // Verificar contraseña
    const passwordValid = await bcrypt.compare(password, user.passwordHash);
    
    if (!passwordValid) {
      // Incrementar contador de intentos fallidos
      await userRepository.incrementFailedAttempts(user.id);
      
      // Verificar si se debe bloquear la cuenta
      const updatedUser = await userRepository.findById(user.id);
      if (updatedUser.failedAttempts >= MAX_FAILED_ATTEMPTS) {
        // Bloquear cuenta por 15 minutos
        const lockUntil = new Date(Date.now() + 15 * 60 * 1000);
        await userRepository.lockAccount(user.id, lockUntil);
      }
      
      return {
        success: false,
        error: 'INVALID_CREDENTIALS',
        message: 'Credenciales inválidas'
      };
    }
    
    // Resetear contador de intentos fallidos
    await userRepository.resetFailedAttempts(user.id);
    
    // Generar token
    const token = generateAuthToken(user);
    
    return {
      success: true,
      token,
      user: {
        id: user.id,
        username: user.username,
        role: user.role
      }
    };
  } catch (error) {
    // Registrar error para investigación interna
    logger.error('Error en autenticación', { error, username });
    
    // No revelar detalles del error al cliente
    return {
      success: false,
      error: 'AUTHENTICATION_ERROR',
      message: 'Error procesando la autenticación. Intente nuevamente.'
    };
  }
}

// Ejemplo de fallo seguro en sistema de pagos
async function processPayment(paymentRequest: PaymentRequest): Promise<PaymentResult> {
  try {
    // Validar solicitud
    const validationResult = validatePaymentRequest(paymentRequest);
    if (!validationResult.valid) {
      return {
        success: false,
        status: 'VALIDATION_ERROR',
        message: 'Datos de pago inválidos'
      };
    }
    
    // Procesar el pago con el proveedor externo
    const paymentResult = await paymentGateway.processTransaction(paymentRequest);
    
    // Verificar resultado
    if (paymentResult.status === 'success') {
      // Registrar transacción exitosa
      await transactionRepository.save({
        ...paymentRequest,
        status: 'completed',
        gatewayReference: paymentResult.transactionId,
        completedAt: new Date()
      });
      
      return {
        success: true,
        transactionId: paymentResult.transactionId,
        status: 'COMPLETED'
      };
    } else {
      // Registrar transacción fallida
      await transactionRepository.save({
        ...paymentRequest,
        status: 'failed',
        failureReason: paymentResult.errorCode,
        gatewayReference: paymentResult.transactionId || null,
        completedAt: null
      });
      
      return {
        success: false,
        status: 'PAYMENT_FAILED',
        message: 'No se pudo procesar el pago',
        errorCode: paymentResult.errorCode
      };
    }
  } catch (error) {
    // Registrar error para investigación
    logger.error('Error crítico en procesamiento de pago', {
      error,
      paymentRequest: {
        ...paymentRequest,
        // Enmascarar datos sensibles en logs
        cardNumber: maskCardNumber(paymentRequest.cardNumber)
      }
    });
    
    // IMPORTANTE: verificar si el pago se completó realmente a pesar del error
    try {
      const paymentStatus = await paymentGateway.checkTransactionStatus({
        merchantReference: paymentRequest.merchantReference
      });
      
      if (paymentStatus.completed) {
        // ¡El pago se completó a pesar del error!
        await transactionRepository.save({
          ...paymentRequest,
          status: 'completed',
          gatewayReference: paymentStatus.transactionId,
          completedAt: new Date(),
          notes: 'Transacción verificada tras error de comunicación'
        });
        
        return {
          success: true,
          transactionId: paymentStatus.transactionId,
          status: 'COMPLETED',
          message: 'Pago procesado correctamente'
        };
      }
    } catch (verificationError) {
      // No se pudo verificar el estado - asumir fallo seguro
      logger.error('Error en verificación de estado de pago', { verificationError });
    }
    
    // Devolver respuesta segura
    return {
      success: false,
      status: 'SYSTEM_ERROR',
      message: 'Error temporal en el procesamiento. No se ha realizado ningún cargo.'
    };
  }
}
```
