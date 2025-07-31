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
  
  // Verificar que el mensaje no sea demasiado antiguo (prevenir replay attacks)
  const now = Date.now();
  if (now - timestamp > MAX_MESSAGE_AGE_MS) {
    return false;
  }
  
  // Recrear la firma y comparar
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(`${payload}${timestamp}${nonce}`)
    .digest('hex');
  
  // Comparación segura (tiempo constante para evitar timing attacks)
  return crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expectedSignature, 'hex')
  );
}

// R: Repudio - Implementar auditoría y logging
class AuditLogger {
  async logAction(userId: string, action: string, resource: string, success: boolean, metadata?: any) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      userId,
      action,
      resource,
      success,
      clientIp: this.getClientIp(),
      userAgent: this.getUserAgent(),
      requestId: this.getRequestId(),
      metadata
    };
    
    // Asegurar inmutabilidad del log (idealmente usando servicios externos o blockchain)
    const signedLog = this.signLogEntry(logEntry);
    
    // Almacenar en múltiples ubicaciones para redundancia y preservación
    await Promise.all([
      this.dbLogger.insert(signedLog),
      this.fileLogger.append(signedLog),
      this.securityMonitoringService.send(signedLog)
    ]);
    
    return logEntry.timestamp;
  }
}

// I: Divulgación de Información - Implementar filtros y controles de acceso
function sanitizeErrorResponse(error: any): SafeErrorResponse {
  // No exponer detalles internos, stacktraces o mensajes de sistema
  const safeError = {
    message: 'Se produjo un error procesando la solicitud',
    code: generateErrorCode(error),
    requestId: generateRequestId()
  };
  
  // Registrar el error completo internamente
  logger.error('Error en solicitud', {
    requestId: safeError.requestId,
    error: error.message,
    stack: error.stack,
    context: getRequestContext()
  });
  
  // Solo devolver información segura al cliente
  return safeError;
}

// D: Denegación de Servicio - Implementar límites y protecciones
class RateLimiter {
  async checkLimit(key: string, limit: number, windowSeconds: number): Promise<RateLimitResult> {
    const now = Date.now();
    const windowStart = now - (windowSeconds * 1000);
    
    // Implementar sliding window para conteo de solicitudes
    const requestCount = await this.cache.zcount(`ratelimit:${key}`, windowStart, now);
    
    if (requestCount >= limit) {
      // Límite excedido
      const oldestRequest = await this.cache.zrange(`ratelimit:${key}`, 0, 0, 'WITHSCORES');
      const resetTime = parseInt(oldestRequest[1]) + (windowSeconds * 1000);
      const retryAfterSeconds = Math.ceil((resetTime - now) / 1000);
      
      return {
        allowed: false,
        remaining: 0,
        retryAfterSeconds,
        limit
      };
    }
    
    // Registrar esta solicitud
    await this.cache.zadd(`ratelimit:${key}`, now, `${now}-${randomBytes(8).toString('hex')}`);
    // Establecer expiración para limpieza automática
    await this.cache.expire(`ratelimit:${key}`, windowSeconds * 2);
    
    return {
      allowed: true,
      remaining: limit - requestCount - 1,
      retryAfterSeconds: 0,
      limit
    };
  }
}

// E: Elevación de Privilegios - Implementar control de acceso riguroso
function requirePermission(requiredPermission: string) {
  return function(req: Request, res: Response, next: NextFunction) {
    // Verificar que el usuario esté autenticado
    if (!req.user) {
      return res.status(401).json({
        error: 'No autenticado',
        message: 'Se requiere autenticación para acceder a este recurso'
      });
    }
    
    // Verificar que el usuario tenga el permiso requerido
    if (!req.user.permissions.includes(requiredPermission)) {
      // Registrar intento de acceso no autorizado
      auditLogger.logAction(
        req.user.id,
        'ACCESS_DENIED',
        req.path,
        false,
        { requiredPermission, userPermissions: req.user.permissions }
      );
      
      return res.status(403).json({
        error: 'Acceso denegado',
        message: 'No tienes permisos suficientes para realizar esta operación'
      });
    }
    
    next();
  };
}
```
