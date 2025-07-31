### 3.4.8 Caso de Estudio: Modelado de Amenazas en una API de Pagos

Para ilustrar la aplicación práctica de las técnicas y metodologías de modelado de amenazas, analizaremos un caso de estudio completo de un servicio de API de pagos.

#### Contexto del Sistema

Consideremos una API de procesamiento de pagos con las siguientes características:

- Procesa transacciones de tarjetas de crédito/débito
- Almacena información limitada de clientes
- Se integra con proveedores externos de procesamiento de pagos
- Expone endpoints REST para aplicaciones cliente
- Emite recibos y comprobantes digitales

#### Paso 1: Diagrama de Flujo de Datos (DFD)

```
+-------------+      +---------------+      +------------------+
|             |      |               |      |                  |
| Cliente Web |----->| API Gateway   |----->| Autenticación    |
|             |      |               |      |                  |
+-------------+      +---------------+      +------------------+
                           |                      |
                           v                      v
                    +---------------+      +------------------+
                    |               |      |                  |
                    | API de Pagos  |<---->| Base de Datos    |
                    |               |      |                  |
                    +---------------+      +------------------+
                           |                       
                           v                       
                    +---------------+       +------------------+
                    | Proveedor de  |       |                  |
                    | Pagos Externo |       | Servicio de      |
                    |               |       | Notificaciones   |
                    +---------------+       +------------------+
                                                    ^
                                                    |
                    +---------------+                |
                    |               |----------------+
                    | Generador de  |
                    | Recibos       |
                    |               |
                    +---------------+
```

#### Paso 2: Identificación de Activos Críticos

| Activo | Descripción | Impacto si se compromete |
|--------|------------|---------------------------|
| Datos de tarjetas | Información de pago de clientes | Alto - Pérdida financiera y reputacional |
| Credenciales API | Claves para acceso a APIs externas | Alto - Potencial fraude masivo |
| Datos de cliente | Información personal de usuarios | Medio - Violación de privacidad |
| Historial de transacciones | Registro de pagos procesados | Medio - Información sensible de negocio |
| Recibos digitales | Comprobantes de pago | Bajo-Medio - Alteración de registros contables |

#### Paso 3: Análisis STRIDE por Componente

Veamos algunas amenazas identificadas usando STRIDE para cada componente:

**API Gateway**

```typescript
// S: Suplantación - Un atacante podría suplantar al API Gateway
threatModel.components.apiGateway.threats.push({
  category: 'Spoofing',
  description: 'Suplantación del API Gateway mediante intercepción de tráfico',
  mitigations: [
    'Implementar TLS mutuo (mTLS)',
    'Validación de certificados',
    'IP Whitelisting para servidores conocidos'
  ],
  riskLevel: 'HIGH'
});

// T: Manipulación - Modificación de los datos de transacción en tránsito
threatModel.components.apiGateway.threats.push({
  category: 'Tampering',
  description: 'Modificación de los datos de la transacción durante la comunicación',
  mitigations: [
    'Uso de HTTPS con TLS 1.3',
    'Implementación de firmas digitales para verificar integridad',
    'Validación de estructuras de datos'
  ],
  riskLevel: 'HIGH'
});
```

**API de Pagos**

```typescript
// I: Divulgación de Información - Fuga de datos de tarjetas
threatModel.components.paymentsApi.threats.push({
  category: 'Information Disclosure',
  description: 'Exposición de datos de tarjetas en logs o respuestas de error',
  mitigations: [
    'Sanitización de logs para datos sensibles',
    'Manejo estandarizado de errores sin datos sensibles',
    'Tokenización de datos de tarjetas',
    'Implementación de políticas PCI-DSS'
  ],
  riskLevel: 'CRITICAL'
});

// D: Denegación de Servicio - Sobrecarga de solicitudes de pago
threatModel.components.paymentsApi.threats.push({
  category: 'Denial of Service',
  description: 'Ataque DoS mediante múltiples solicitudes de procesamiento',
  mitigations: [
    'Implementar rate limiting por IP y por cliente',
    'Uso de sistema de colas para procesamiento',
    'Escalado automático de recursos',
    'Implementación de timeout en integraciones externas'
  ],
  riskLevel: 'HIGH'
});
```

**Base de Datos**

```typescript
// E: Elevación de Privilegios - Acceso no autorizado a la base de datos
threatModel.components.database.threats.push({
  category: 'Elevation of Privilege',
  description: 'Escalada de privilegios mediante inyección SQL',
  mitigations: [
    'Uso de ORM con consultas parametrizadas',
    'Principio de mínimo privilegio en cuentas de BD',
    'Segmentación de base de datos',
    'Auditoría de cambios en esquemas y privilegios'
  ],
  riskLevel: 'CRITICAL'
});

// R: Repudio - Modificación no registrada de datos de transacciones
threatModel.components.database.threats.push({
  category: 'Repudiation',
  description: 'Modificación de registros de transacciones sin auditoría',
  mitigations: [
    'Registros de auditoría inmutables',
    'Firmas digitales en registros críticos',
    'Separación de responsabilidades para cambios en datos'
  ],
  riskLevel: 'MEDIUM'
});
```

#### Paso 4: Implementación de Mitigaciones

Veamos la implementación de algunas mitigaciones clave:

**1. Tokenización de datos de tarjetas**

```typescript
import { TokenizationService } from './services/tokenization';

class PaymentProcessor {
  private tokenizationService: TokenizationService;
  
  constructor(tokenizationService: TokenizationService) {
    this.tokenizationService = tokenizationService;
  }
  
  async processPayment(paymentData: PaymentRequest): Promise<PaymentResult> {
    try {
      // Tokenizar datos de tarjeta inmediatamente al recibir
      const cardToken = await this.tokenizationService.tokenize(paymentData.cardDetails);
      
      // Eliminar datos sensibles de memoria lo antes posible
      const sanitizedPaymentData = {
        ...paymentData,
        cardDetails: undefined, // Eliminar datos originales
        cardToken // Usar solo el token en adelante
      };
      
      // Procesar el pago con el token
      const result = await this.paymentProvider.processWithToken(sanitizedPaymentData);
      
      // Almacenar solo información segura
      await this.transactionRepository.save({
        transactionId: result.transactionId,
        amount: sanitizedPaymentData.amount,
        cardTokenLastFour: cardToken.lastFour,
        status: result.status,
        timestamp: new Date()
      });
      
      return result;
    } catch (error) {
      // Manejo seguro de errores
      this.logSafeError('Error procesando pago', error);
      throw new PaymentProcessingError('No se pudo procesar el pago');
    }
  }
}
```

**2. Implementación de rate limiting**

```typescript
import Redis from 'redis';
import { promisify } from 'util';

class RateLimiter {
  private readonly redis: Redis.RedisClient;
  private readonly getAsync: (key: string) => Promise<string | null>;
  private readonly setAsync: (key: string, value: string) => Promise<unknown>;
  private readonly expireAsync: (key: string, seconds: number) => Promise<number>;

  constructor() {
    // Configurar cliente Redis
    this.redis = Redis.createClient({
      host: process.env.REDIS_HOST,
      password: process.env.REDIS_PASSWORD,
      tls: process.env.NODE_ENV === 'production' ? {} : undefined
    });
    
    // Promisify métodos Redis
    this.getAsync = promisify(this.redis.get).bind(this.redis);
    this.setAsync = promisify(this.redis.set).bind(this.redis);
    this.expireAsync = promisify(this.redis.expire).bind(this.redis);
  }

  // Middleware para Express
  createMiddleware(options: {
    windowMs: number; // Ventana de tiempo en ms
    maxRequests: number; // Número máximo de solicitudes
    keyGenerator: (req: Request) => string; // Generador de clave (IP, usuario, etc)
  }) {
    return async (req: Request, res: Response, next: NextFunction) => {
      try {
        const key = `rate-limit:${options.keyGenerator(req)}`;
        
        // Obtener conteo actual
        const current = await this.getAsync(key);
        const count = current ? parseInt(current, 10) : 0;
        
        // Verificar límite
        if (count >= options.maxRequests) {
          // Registrar intento de exceso
          this.logRateLimitExceeded(req, count);
          
          // Cabeceras informativas
          res.setHeader('X-RateLimit-Limit', options.maxRequests);
          res.setHeader('X-RateLimit-Remaining', 0);
          res.setHeader('Retry-After', Math.ceil(options.windowMs / 1000));
          
          return res.status(429).json({
            error: 'Too Many Requests',
            message: 'Has excedido el límite de solicitudes permitidas'
          });
        }
        
        // Incrementar contador
        await this.setAsync(key, (count + 1).toString());
        
        // Establecer tiempo de expiración si es nuevo
        if (count === 0) {
          await this.expireAsync(key, Math.ceil(options.windowMs / 1000));
        }
        
        // Cabeceras informativas
        res.setHeader('X-RateLimit-Limit', options.maxRequests);
        res.setHeader('X-RateLimit-Remaining', options.maxRequests - count - 1);
        
        next();
      } catch (error) {
        // En caso de error, permitir la solicitud
        console.error('Error en rate limiting:', error);
        next();
      }
    };
  }
  
  private logRateLimitExceeded(req: Request, count: number) {
    const clientIp = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
    console.warn(`Rate limit exceeded: IP=${clientIp}, count=${count}, endpoint=${req.path}`);
  }
}

// Uso en la aplicación
const rateLimiter = new RateLimiter();

// Aplicar límites específicos por ruta
app.use('/api/payments', rateLimiter.createMiddleware({
  windowMs: 60 * 1000, // 1 minuto
  maxRequests: 10, // 10 solicitudes por minuto
  keyGenerator: (req) => {
    // Usar clientId si está autenticado, o IP si no
    return req.user?.clientId || req.ip;
  }
}));
```

**3. Implementación de auditoría inmutable**

```typescript
import { createHmac } from 'crypto';

class AuditLogger {
  private readonly secret: string;
  private readonly externalLogService: ExternalLogService;
  
  constructor() {
    // Usar secreto diferente al de la aplicación
    this.secret = process.env.AUDIT_SECRET_KEY;
    this.externalLogService = new ExternalLogService();
  }
  
  async logTransactionEvent(event: TransactionEvent): Promise<string> {
    // Generar entrada de auditoría con todos los detalles relevantes
    const auditEntry = {
      eventId: this.generateEventId(),
      timestamp: new Date().toISOString(),
      transactionId: event.transactionId,
      eventType: event.type,
      actor: {
        userId: event.userId,
        ipAddress: event.ipAddress,
        userAgent: event.userAgent
      },
      details: this.sanitizeDetails(event.details),
      previousState: event.previousState,
      newState: event.newState
    };
    
    // Calcular hash para detección de manipulación
    const entryJson = JSON.stringify(auditEntry);
    const hmac = createHmac('sha256', this.secret)
      .update(entryJson)
      .digest('hex');
    
    // Añadir firma al registro
    const signedEntry = {
      ...auditEntry,
      signature: hmac
    };
    
    // Almacenar en múltiples ubicaciones para inmutabilidad
    await Promise.all([
      // Base de datos local
      this.localAuditRepository.insert(signedEntry),
      
      // Servicio externo para almacenamiento inmutable
      this.externalLogService.store(signedEntry),
      
      // Si es crítico, enviar a sistema blockchain de auditoría
      event.isCritical ? this.blockchainAuditService.record(signedEntry) : Promise.resolve()
    ]);
    
    return signedEntry.eventId;
  }
  
  // Verificar la integridad de un registro de auditoría
  async verifyAuditEntry(entry: SignedAuditEntry): Promise<boolean> {
    // Extraer firma
    const { signature, ...auditData } = entry;
    
    // Recalcular HMAC
    const expectedSignature = createHmac('sha256', this.secret)
      .update(JSON.stringify(auditData))
      .digest('hex');
    
    // Comparar firmas
    return signature === expectedSignature;
  }
  
  private generateEventId(): string {
    // Generar ID único para el evento
    return `evt_${Date.now()}_${crypto.randomBytes(8).toString('hex')}`;
  }
  
  private sanitizeDetails(details: any): any {
    // Eliminar o enmascarar datos sensibles antes de registrar
    const sanitized = { ...details };
    
    // Eliminar datos sensibles conocidos
    if (sanitized.cardNumber) sanitized.cardNumber = this.maskCardNumber(sanitized.cardNumber);
    if (sanitized.cvv) delete sanitized.cvv;
    if (sanitized.password) delete sanitized.password;
    
    return sanitized;
  }
  
  private maskCardNumber(cardNumber: string): string {
    // Mostrar solo los últimos 4 dígitos
    return cardNumber.replace(/\d(?=\d{4})/g, '*');
  }
}
```

**4. Implementación de validación y sanitización de entrada**

```typescript
import { z } from 'zod';
import { sanitizeHtml } from 'sanitize-html';

// Esquema de validación para la solicitud de pago
const paymentRequestSchema = z.object({
  amount: z.number().positive().max(100000), // Limitar monto máximo
  currency: z.enum(['USD', 'EUR', 'GBP']), // Monedas permitidas
  description: z.string().max(500).transform(val => sanitizeHtml(val)), // Sanitizar HTML
  cardDetails: z.object({
    number: z.string()
      .regex(/^\d{13,19}$/, 'Número de tarjeta inválido') // Validación básica
      .refine(num => luhnCheck(num), 'Número de tarjeta inválido'), // Algoritmo de Luhn
    expiryMonth: z.number().min(1).max(12),
    expiryYear: z.number().min(new Date().getFullYear()),
    cvv: z.string().regex(/^\d{3,4}$/, 'CVV inválido')
  }),
  billingAddress: z.object({
    line1: z.string().max(100),
    line2: z.string().max(100).optional(),
    city: z.string().max(100),
    postalCode: z.string().max(20),
    country: z.string().length(2) // ISO country code
  })
});

// Middleware de validación para Express
function validatePaymentRequest(req: Request, res: Response, next: NextFunction) {
  try {
    // Validar y sanitizar datos de entrada
    const validatedData = paymentRequestSchema.parse(req.body);
    
    // Reemplazar cuerpo de la solicitud con datos validados
    req.body = validatedData;
    
    next();
  } catch (error) {
    if (error instanceof z.ZodError) {
      // Estructurar errores de validación
      const formattedErrors = error.errors.map(err => ({
        path: err.path.join('.'),
        message: err.message
      }));
      
      // Registrar intento con datos inválidos
      console.warn('Datos de pago inválidos:', {
        clientIp: req.ip,
        userId: req.user?.id,
        errors: formattedErrors
      });
      
      return res.status(400).json({
        error: 'Validation Error',
        details: formattedErrors
      });
    }
    
    next(error);
  }
}

// Función auxiliar para validación de tarjetas con algoritmo de Luhn
function luhnCheck(cardNumber: string): boolean {
  // Implementación del algoritmo de Luhn
  const digits = cardNumber.split('').map(Number);
  let sum = 0;
  let alternate = false;
  
  for (let i = digits.length - 1; i >= 0; i--) {
    let digit = digits[i];
    
    if (alternate) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }
    
    sum += digit;
    alternate = !alternate;
  }
  
  return sum % 10 === 0;
}

// Aplicar el middleware de validación
app.post('/api/payments', validatePaymentRequest, paymentController.processPayment);
```

#### Paso 5: Evaluación y Priorización de Riesgos

Utilizamos la metodología DREAD para evaluar y priorizar los riesgos identificados:

| Amenaza | D | R | E | A | D | Total | Prioridad |
|---------|---|---|---|---|---|-------|----------|
| Exposición datos de tarjetas | 3 | 2 | 3 | 3 | 2 | 13 | Alta |
| Inyección SQL | 3 | 3 | 2 | 3 | 2 | 13 | Alta |
| Manipulación de transacciones | 3 | 2 | 2 | 3 | 1 | 11 | Media-Alta |
| Ataque DoS | 2 | 3 | 2 | 3 | 1 | 11 | Media-Alta |
| Suplantación de API | 2 | 1 | 2 | 2 | 1 | 8 | Media |
| Modificación sin auditoría | 2 | 1 | 1 | 1 | 1 | 6 | Baja |

#### Paso 6: Integración con el Ciclo de Desarrollo

Finalmente, este modelo de amenazas se integra en el ciclo de desarrollo:

1. **Backlog de Seguridad**: Las mitigaciones se convierten en historias de usuario en el backlog de desarrollo, priorizadas según su evaluación DREAD.

2. **Pruebas Automatizadas**: Se implementan pruebas específicas para verificar las mitigaciones:

```typescript
describe('PaymentProcessor - Seguridad', () => {
  it('no debe almacenar datos completos de tarjetas en logs', async () => {
    const logSpy = jest.spyOn(console, 'log');
    const paymentProcessor = new PaymentProcessor(mockTokenizationService);
    
    await paymentProcessor.processPayment(mockPaymentData);
    
    // Verificar que los logs no contengan datos sensibles
    expect(logSpy).not.toHaveBeenCalledWith(
      expect.stringContaining(mockPaymentData.cardDetails.number)
    );
  });
  
  it('debe tokenizar datos de tarjetas antes de procesamiento', async () => {
    const tokenizeSpy = jest.spyOn(mockTokenizationService, 'tokenize');
    const paymentProcessor = new PaymentProcessor(mockTokenizationService);
    
    await paymentProcessor.processPayment(mockPaymentData);
    
    // Verificar que se llamó al servicio de tokenización
    expect(tokenizeSpy).toHaveBeenCalledWith(mockPaymentData.cardDetails);
    expect(tokenizeSpy).toHaveBeenCalledBefore(mockPaymentProvider.process);
  });
  
  it('debe validar transacciones contra límites configurados', async () => {
    const paymentProcessor = new PaymentProcessor(mockTokenizationService);
    const data = { ...mockPaymentData, amount: 1000000 }; // Monto extremadamente alto
    
    // Debería rechazar la transacción
    await expect(paymentProcessor.processPayment(data))
      .rejects.toThrow('El monto excede los límites permitidos');
  });
  
  it('debe crear registros de auditoría inmutables para transacciones', async () => {
    const auditSpy = jest.spyOn(mockAuditLogger, 'logTransactionEvent');
    const paymentProcessor = new PaymentProcessor(
      mockTokenizationService, 
      mockPaymentProvider,
      mockAuditLogger
    );
    
    await paymentProcessor.processPayment(mockPaymentData);
    
    // Verificar que se creó registro de auditoría
    expect(auditSpy).toHaveBeenCalled();
    expect(auditSpy.mock.calls[0][0]).toHaveProperty('signature');
  });
});
```

3. **Monitoreo Continuo**: Implementación de alertas y monitoreo para detectar potenciales explotaciones de las vulnerabilidades identificadas:

```typescript
// Configuración de monitoreo y alertas
const securityMonitoring = {
  // Alertas para posibles explotaciones de vulnerabilidades
  rulesets: [
    {
      name: 'Detección de fugas de datos',
      rules: [
        {
          id: 'card-data-leak',
          description: 'Posible fuga de datos de tarjetas',
          pattern: /\b(?:\d{4}[-\s]?){3}\d{4}\b/,
          severity: 'CRITICAL',
          sources: ['http-response', 'log-output'],
          action: 'BLOCK_AND_ALERT'
        }
      ]
    },
    {
      name: 'Detección de ataques de inyección',
      rules: [
        {
          id: 'sql-injection',
          description: 'Posible intento de inyección SQL',
          patterns: [
            /(\%27)|(\')|(\-\-)|(\%23)|(#)/i,
            /((\%3D)|(=))[^\n]*((\%27)|(\')|(\-\-)|(\%3B)|(;))/i,
            /\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/i,
            /exec(\s|\+)+(s|x)p\w+/i
          ],
          severity: 'HIGH',
          sources: ['http-request', 'query-params'],
          action: 'BLOCK_AND_ALERT'
        }
      ]
    },
    {
      name: 'Detección de ataques DoS',
      rules: [
        {
          id: 'rate-limit-exceeded',
          description: 'Posible ataque DoS - Límites de tasa excedidos',
          condition: {
            metric: 'request-rate',
            threshold: 100,
            timeWindow: '1m',
            groupBy: ['client-ip']
          },
          severity: 'MEDIUM',
          action: 'RATE_LIMIT_AND_ALERT'
        }
      ]
    }
  ],
  
  // Configuración de alertas
  alertChannels: [
    {
      type: 'email',
      recipients: ['security-team@example.com'],
      minSeverity: 'MEDIUM'
    },
    {
      type: 'slack',
      webhook: process.env.SLACK_SECURITY_WEBHOOK,
      channel: '#security-alerts',
      minSeverity: 'HIGH'
    }
  ],
  
  // Dashboard con métricas clave
  metrics: [
    {
      name: 'auth-failures',
      description: 'Intentos fallidos de autenticación',
      thresholds: {
        warning: 10,
        critical: 50
      }
    },
    {
      name: 'rate-limit-hits',
      description: 'Solicitudes bloqueadas por rate limiting',
      thresholds: {
        warning: 50,
        critical: 200
      }
    },
    {
      name: 'transaction-validation-failures',
      description: 'Fallos de validación en transacciones',
      thresholds: {
        warning: 20,
        critical: 100
      }
    }
  ]
};
```

#### Paso 7: Revisión y Evolución Continua

Para garantizar que el modelo de amenazas se mantenga relevante a medida que el sistema evoluciona:

1. **Revisiones periódicas**: Programar revisiones trimestrales del modelo de amenazas.

2. **Actualización basada en incidentes**: Incorporar lecciones aprendidas de incidentes de seguridad.

3. **Adaptación a nuevos vectores de ataque**: Mantenerse actualizado sobre nuevas técnicas de ataque.

4. **Automatización**: Implementar herramientas que verifiquen continuamente que las mitigaciones están en funcionamiento.

5. **Feedback de seguridad**: Establecer un proceso para incorporar hallazgos de pruebas de penetración y auditorías.

Este caso de estudio muestra cómo el modelado de amenazas se aplica de forma práctica en un sistema de pagos, desde la identificación de activos críticos hasta la implementación y verificación de controles de seguridad, asegurando una protección efectiva contra las amenazas más relevantes.
