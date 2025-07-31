### 3.4.4 Herramientas para el Modelado de Amenazas

Existen diversas herramientas que facilitan el proceso de modelado de amenazas, adaptándose a diferentes necesidades y contextos de desarrollo:

#### Microsoft Threat Modeling Tool

Herramienta visual gratuita que permite crear diagramas de flujo de datos, identificar amenazas automáticamente utilizando STRIDE y generar informes detallados.

```typescript
// Ejemplo de vulnerabilidades identificadas por Microsoft Threat Modeling Tool
// y su implementación de mitigación en código

// Vulnerabilidad: Suplantación en la API de autenticación
// Mitigación: Implementación de tokens anti-CSRF

class AuthController {
  login(req: Request, res: Response) {
    // Generar token CSRF único para este usuario y sesión
    const csrfToken = crypto.randomBytes(64).toString('hex');
    
    // Almacenar en la sesión del servidor
    req.session.csrfToken = csrfToken;
    
    // Enviar token al cliente en una cookie segura
    res.cookie('XSRF-TOKEN', csrfToken, {
      httpOnly: false, // Necesita ser accesible por JavaScript
      secure: true,
      sameSite: 'strict'
    });
    
    // Resto de la lógica de login
    // ...
  }
  
  // Middleware para verificar token CSRF
  validateCsrf(req: Request, res: Response, next: NextFunction) {
    const csrfCookie = req.cookies['XSRF-TOKEN'];
    const csrfHeader = req.headers['x-xsrf-token'];
    
    if (!csrfCookie || !csrfHeader || csrfCookie !== csrfHeader || 
        csrfCookie !== req.session.csrfToken) {
      return res.status(403).json({ error: 'CSRF verification failed' });
    }
    
    next();
  }
}

// Uso del middleware en rutas sensibles
router.post('/api/user/profile', authController.validateCsrf, userController.updateProfile);
router.post('/api/payment', authController.validateCsrf, paymentController.processPayment);
```

#### OWASP Threat Dragon

Herramienta de código abierto de OWASP para la creación de modelos de amenazas con un enfoque en la sencillez y la colaboración en equipos.

```typescript
// Ejemplo de mitigación basada en amenaza identificada en OWASP Threat Dragon
// Vulnerabilidad: Exposición de información sensible en logs y errores
// Mitigación: Implementación de filtrado y sanitización

class LoggingService {
  // Lista de campos que nunca deben aparecer en logs
  private sensitiveFields = [
    'password', 'token', 'secret', 'apiKey', 'cardNumber', 
    'cvv', 'ssn', 'passport', 'creditCard'
  ];
  
  // Método para sanitizar objetos antes de loggearlos
  sanitizeForLogging(data: any): any {
    if (!data) return data;
    
    // Si es string, verificar si contiene patrones sensibles
    if (typeof data === 'string') {
      return this.maskSensitivePatterns(data);
    }
    
    // Si es un array, sanitizar cada elemento
    if (Array.isArray(data)) {
      return data.map(item => this.sanitizeForLogging(item));
    }
    
    // Si es un objeto, sanitizar recursivamente
    if (typeof data === 'object') {
      const sanitized = { ...data };
      
      for (const key in sanitized) {
        // Verificar si es un campo sensible
        if (this.sensitiveFields.some(field => key.toLowerCase().includes(field))) {
          sanitized[key] = '[REDACTADO]';
        } else {
          // Sanitizar recursivamente
          sanitized[key] = this.sanitizeForLogging(sanitized[key]);
        }
      }
      
      return sanitized;
    }
    
    return data;
  }
  
  // Método para sanitizar y loggear datos
  log(level: 'info' | 'warn' | 'error', message: string, data?: any): void {
    const sanitizedData = data ? this.sanitizeForLogging(data) : undefined;
    
    // Ahora podemos loggear de forma segura
    console[level](message, sanitizedData);
    
    // También se podría enviar a un servicio de logging
    // this.logstashClient.send({
    //   level,
    //   message,
    //   data: sanitizedData,
    //   timestamp: new Date().toISOString()
    // });
  }
  
  // Sanitizar patrones sensibles en strings (tarjetas, tokens, etc.)
  private maskSensitivePatterns(input: string): string {
    // Máscara para números de tarjeta de crédito
    let masked = input.replace(
      /(\d{4})\d{8,12}(\d{4})/g, 
      '$1********$2'
    );
    
    // Máscara para tokens Bearer
    masked = masked.replace(
      /(Bearer\s+)[A-Za-z0-9\-_=]+\.[A-Za-z0-9\-_=]+\.?[A-Za-z0-9\-_.+/=]*/gi,
      '$1[REDACTADO]'
    );
    
    return masked;
  }
}

// Uso del servicio de logging seguro
const logger = new LoggingService();

try {
  // Alguna operación que puede fallar
  processPayment(userDetails);
} catch (error) {
  // Log seguro que no expondrá datos sensibles
  logger.log('error', 'Error procesando pago', {
    userId: user.id,
    error: error.message,
    cardDetails: user.paymentInfo,  // Será sanitizado automáticamente
    requestId: requestContext.id
  });
}
```

#### ThreatSpec

Herramienta de "modelado de amenazas como código" que permite integrar el modelado directamente en el código fuente mediante comentarios especiales.

```typescript
/**
 * @threat Spoofing de identidad vía man-in-the-middle
 * @mitigates Spoofing mediante uso de HTTPS y certificados pinning
 */
class ApiClient {
  constructor() {
    this.axios = axios.create({
      baseURL: 'https://api.payment-service.com',
      httpsAgent: new https.Agent({
        rejectUnauthorized: true,
        ca: fs.readFileSync(path.join(__dirname, '../certs/ca.pem')),
        cert: fs.readFileSync(path.join(__dirname, '../certs/cert.pem')),
        key: fs.readFileSync(path.join(__dirname, '../certs/key.pem')),
      })
    });
  }
  
  /**
   * @threat Tampering de datos durante la transmisión
   * @mitigates Tampering mediante firma digital de las solicitudes
   */
  async sendPaymentRequest(paymentData) {
    const signature = this.generateSignature(paymentData);
    
    return this.axios.post('/payments', {
      ...paymentData,
      signature,
      timestamp: Date.now()
    });
  }
  
  /**
   * @threat Repudio de transacciones
   * @mitigates Repudio mediante generación de firmas digitales y logging inmutable
   */
  generateSignature(data) {
    const payload = JSON.stringify(data);
    return crypto
      .createHmac('sha256', this.apiSecret)
      .update(payload)
      .digest('hex');
  }
}
```

#### IriusRisk

Plataforma comercial que ofrece modelado de amenazas automatizado, análisis de riesgos y generación de requisitos de seguridad, con una amplia biblioteca de conocimientos y plantillas predefinidas para distintos tipos de aplicaciones.

```typescript
// Ejemplo de implementación basada en los requisitos generados por IriusRisk
// para un servicio de pagos

// Requisito: Implementar endurecimiento de cabeceras HTTP
import helmet from 'helmet';

// Aplicar cabeceras de seguridad recomendadas por IriusRisk
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", 'trusted-cdn.com'],
      styleSrc: ["'self'", "'unsafe-inline'", 'trusted-cdn.com'],
      imgSrc: ["'self'", 'data:', 'trusted-cdn.com'],
      connectSrc: ["'self'", 'api.payment-processor.com'],
      formAction: ["'self'"],
      frameAncestors: ["'none'"]
    }
  },
  hsts: {
    maxAge: 31536000, // 1 año
    includeSubDomains: true,
    preload: true
  }
}));

// Requisito: Implementar protección contra ataques de fuerza bruta
class BruteForceProtection {
  constructor(redisClient) {
    this.redis = redisClient;
    this.maxAttempts = 5;
    this.blockDuration = 30 * 60; // 30 minutos en segundos
  }
  
  async check(identifier) {
    const key = `bruteforce:${identifier}`;
    const attempts = await this.redis.incr(key);
    
    // Primera vez que se ve este identificador
    if (attempts === 1) {
      await this.redis.expire(key, 3600); // Expirar después de 1 hora
    }
    
    if (attempts > this.maxAttempts) {
      const blockKey = `blocked:${identifier}`;
      const isBlocked = await this.redis.exists(blockKey);
      
      if (!isBlocked) {
        // Bloquear por el tiempo especificado
        await this.redis.set(blockKey, '1');
        await this.redis.expire(blockKey, this.blockDuration);
      }
      
      return {
        allowed: false,
        remainingAttempts: 0,
        blockedFor: await this.redis.ttl(blockKey)
      };
    }
    
    return {
      allowed: true,
      remainingAttempts: this.maxAttempts - attempts
    };
  }
  
  async reset(identifier) {
    const key = `bruteforce:${identifier}`;
    const blockKey = `blocked:${identifier}`;
    
    await Promise.all([
      this.redis.del(key),
      this.redis.del(blockKey)
    ]);
  }
}

// Requisito: Implementar validación de entrada estricta basada en esquemas
import { z } from 'zod';

// Esquema de validación para solicitudes de pago
const paymentSchema = z.object({
  amount: z.number().positive().lt(1000000), // Monto máximo de $1M
  currency: z.string().length(3).regex(/^[A-Z]{3}$/), // ISO 4217 (e.j. USD, EUR)
  cardToken: z.string().regex(/^tok_[a-zA-Z0-9]{24}$/), // Formato específico de token
  description: z.string().max(255).optional(),
  metadata: z.record(z.string(), z.string()).optional()
});

function validatePaymentRequest(req, res, next) {
  const result = paymentSchema.safeParse(req.body);
  
  if (!result.success) {
    return res.status(400).json({
      error: 'Validation error',
      details: result.error.issues.map(issue => ({
        path: issue.path.join('.'),
        message: issue.message
      }))
    });
  }
  
  // La validación fue exitosa, continuar
  req.validatedData = result.data;
  next();
}
```

#### Consideraciones para la Selección de Herramientas

La selección de herramientas de modelado de amenazas debe considerar:

1. **Tamaño y complejidad** del proyecto
2. **Experiencia del equipo** en seguridad
3. **Integración** con el proceso de desarrollo existente
4. **Costo** y recursos disponibles
5. **Necesidades de colaboración** y documentación

Las organizaciones más maduras suelen implementar un enfoque híbrido, combinando herramientas formales de modelado con procesos integrados en el desarrollo y revisiones de seguridad manuales.
