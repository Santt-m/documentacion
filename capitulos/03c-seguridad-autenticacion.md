# Capítulo 3: Seguridad en el Desarrollo de Software

## 3.3 Autenticación y Gestión de Sesiones

### 3.3.1 Gestión Segura de Sesiones

La gestión de sesiones es fundamental para mantener el estado de un usuario autenticado a lo largo de múltiples solicitudes. Una gestión insegura de sesiones puede llevar a ataques como el secuestro de sesión.

#### Cookies HttpOnly y Secure:

Las cookies que almacenan identificadores de sesión o tokens de autenticación deben configurarse con los flags adecuados:

- **HttpOnly:** Impide que el JavaScript del lado del cliente acceda a la cookie, mitigando los ataques XSS.
- **Secure:** Asegura que la cookie solo se envíe a través de conexiones HTTPS, protegiendo contra ataques de intermediarios.
- **SameSite:** Controla cuándo se envían las cookies en solicitudes entre sitios, mitigando ataques CSRF.
- **Path y Domain:** Limitan el alcance de la cookie a URLs específicas, reduciendo la superficie de ataque.

```typescript
// Configuración de cookie segura en Express (Node.js)
import express from 'express';
import session from 'express-session';
import connectRedis from 'connect-redis';
import Redis from 'ioredis';

const app = express();
const RedisStore = connectRedis(session);
const redisClient = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: Number(process.env.REDIS_PORT) || 6379,
});

// Configuración de sesión segura
app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET || 'secret-value-for-development-only',
  name: 'app.sid', // Cambia el nombre predeterminado de la cookie para no revelar que usas Express
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true, // Previene acceso desde JavaScript del lado del cliente
    secure: process.env.NODE_ENV === 'production', // Solo HTTPS en producción
    maxAge: 1000 * 60 * 60 * 24, // 24 horas
    sameSite: 'strict', // Controla el envío de cookies en solicitudes cross-site
    path: '/' // Limita la cookie a este path
  }
}));
```

#### Protección contra Cross-Site Request Forgery (CSRF):

CSRF es un ataque que obliga a un usuario final a ejecutar acciones no deseadas en una aplicación web en la que ya está autenticado.

```typescript
// Implementación de protección CSRF en Express (Node.js)
import express from 'express';
import csrf from 'csurf';
import cookieParser from 'cookie-parser';

const app = express();

// Middleware necesario para csurf
app.use(cookieParser());
app.use(express.urlencoded({ extended: false }));

// Configuración de protección CSRF
const csrfProtection = csrf({ cookie: { httpOnly: true, secure: true, sameSite: 'strict' } });

// Aplicar protección CSRF a rutas que manejan formularios o modificaciones
app.get('/form', csrfProtection, (req, res) => {
  // Pasar el token al frontend
  res.render('form', { csrfToken: req.csrfToken() });
});

app.post('/process', csrfProtection, (req, res) => {
  // Si llegamos aquí, el token CSRF ha sido validado
  // Procesar el formulario
  res.send('Formulario procesado correctamente');
});
```

### 3.3.2 Cabeceras HTTP de Seguridad

Las cabeceras HTTP relacionadas con la seguridad proporcionan capas adicionales de protección contra diversos ataques:

#### HSTS (HTTP Strict Transport Security):

Lógica: Una cabecera de respuesta HTTP que le indica al navegador que solo debe interactuar con el sitio web usando conexiones HTTPS, incluso si el usuario intenta acceder a través de HTTP. Ayuda a prevenir ataques de "SSL stripping".

```typescript
// Implementación en Express
import express from 'express';
import helmet from 'helmet';

const app = express();

// Configuración de HSTS
app.use(helmet.hsts({
  maxAge: 31536000, // 1 año en segundos
  includeSubDomains: true, // Aplicar a subdominios
  preload: true // Indicar elegibilidad para precargar en navegadores
}));

// También se puede configurar manualmente
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
  next();
});
```

#### CORS (Cross-Origin Resource Sharing):

Lógica: Mecanismo que permite que un servidor restrinja los recursos que pueden ser solicitados desde un dominio diferente. Fundamental para proteger APIs y recursos web.

```typescript
// Implementación de CORS en Express
import express from 'express';
import cors from 'cors';

const app = express();

// Configuración básica de CORS (permisiva para desarrollo)
app.use(cors());

// Configuración de CORS restrictiva para producción
app.use(cors({
  origin: ['https://miapp.com', 'https://admin.miapp.com'], // Solo permitir estos orígenes
  methods: ['GET', 'POST', 'PUT', 'DELETE'], // Métodos permitidos
  allowedHeaders: ['Content-Type', 'Authorization'], // Cabeceras permitidas
  credentials: true // Permite el envío de cookies y cabeceras de autorización
}));

// ... tus rutas y lógica de API
```

### 3.3.3 Cifrado de Datos en Reposo (Encryption at Rest):

Lógica: Cifrar los datos cuando están almacenados en la base de datos o en el sistema de archivos. Esto protege los datos en caso de una brecha de seguridad que comprometa el almacenamiento físico.

```typescript
// Ejemplo de cifrado y descifrado de datos sensibles
import * as crypto from 'crypto';

/**
 * Clase para manejar el cifrado y descifrado de datos sensibles
 */
class DataEncryptor {
  private algorithm: string;
  private key: Buffer;
  private iv: Buffer;

  /**
   * Constructor
   * @param encryptionKey Clave de cifrado (debe ser de 32 bytes para AES-256)
   */
  constructor(encryptionKey: string) {
    this.algorithm = 'aes-256-cbc';
    // Crear una clave de 32 bytes (256 bits) a partir de la clave proporcionada
    this.key = crypto.createHash('sha256').update(encryptionKey).digest();
    // El IV debe ser único para cada operación de cifrado
    this.iv = crypto.randomBytes(16);
  }

  /**
   * Cifra un texto
   * @param text Texto a cifrar
   * @returns Objeto con el texto cifrado en hexadecimal y el IV utilizado
   */
  encrypt(text: string): { encryptedData: string; iv: string } {
    // Generar un nuevo IV para cada operación de cifrado
    const iv = crypto.randomBytes(16);
    
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return {
      encryptedData: encrypted,
      iv: iv.toString('hex')
    };
  }

  /**
   * Descifra un texto cifrado
   * @param encryptedData Texto cifrado en hexadecimal
   * @param iv IV utilizado para cifrar, en hexadecimal
   * @returns Texto descifrado
   */
  decrypt(encryptedData: string, iv: string): string {
    const decipher = crypto.createDecipheriv(
      this.algorithm, 
      this.key, 
      Buffer.from(iv, 'hex')
    );
    
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}

// Ejemplo de uso
const encryptor = new DataEncryptor(process.env.ENCRYPTION_KEY || 'clave-segura-para-desarrollo');

// Cifrar datos sensibles
const sensitiveData = "Número de tarjeta de crédito: 1234-5678-9012-3456";
const { encryptedData, iv } = encryptor.encrypt(sensitiveData);

console.log('Datos cifrados:', encryptedData);
console.log('IV:', iv);

// Más tarde, descifrar los datos
const decryptedData = encryptor.decrypt(encryptedData, iv);
console.log('Datos descifrados:', decryptedData);
```

## 3.4 Seguridad en la Cadena de Suministro de Software

### 3.4.1 Ataques a la Cadena de Suministro (Supply Chain Attacks):

Lógica: Un atacante compromete una dependencia popular (ej. inyecta código malicioso en una librería de npm) para luego atacar a todas las aplicaciones que la utilizan.

#### Mitigación:

- Revisa cuidadosamente las dependencias antes de adoptarlas.
- Usa bloqueos de versiones (package-lock.json, yarn.lock) para asegurar que las instalaciones sean consistentes.
- Implementa políticas de seguridad de contenido (CSP) para reducir el impacto de scripts de terceros.

### 3.4.2 Seguridad en Pipelines CI/CD:

Lógica: Integrar análisis de seguridad automatizados en los pipelines CI/CD para detectar vulnerabilidades antes del despliegue.

```yaml
# Ejemplo de configuración de GitHub Actions con análisis de seguridad
name: Security Checks

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run npm audit
      run: npm audit --audit-level=high
      
    - name: Run ESLint
      run: npm run lint
      
    - name: Run SAST with SonarCloud
      uses: SonarSource/sonarcloud-github-action@v1.8
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      
    - name: Run dependency vulnerability scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### 3.4.3 Seguridad en Contenedores (Docker/Kubernetes):

Lógica: Los entornos contenerizados introducen nuevos vectores de ataque que requieren consideraciones de seguridad específicas.

#### Mitigación:

- Usa imágenes base mínimas.
- Evita ejecutar contenedores como root.
- Escanea las imágenes de Docker en busca de vulnerabilidades.
- Configura políticas de red y seguridad en Kubernetes (Network Policies, RBAC).

## 3.5 Monitoreo y Respuesta a Incidentes

### 3.5.1 Logging Seguro:

Lógica: Implementar un sistema de logging que capture eventos de seguridad relevantes sin registrar información sensible.

```typescript
// Ejemplo de configuración de logging seguro con Winston
import winston from 'winston';
import Transport from 'winston-transport';

// Crear un transport personalizado que sanitiza información sensible
class SanitizingTransport extends Transport {
  constructor(opts?: Transport.TransportStreamOptions) {
    super(opts);
  }
  
  log(info: any, callback: () => void) {
    // Sanitizar datos sensibles antes de loggear
    if (info.password) {
      info.password = '[REDACTADO]';
    }
    if (info.creditCard) {
      info.creditCard = '[REDACTADO]';
    }
    
    // Puedes sanitizar datos que coincidan con patrones (ej. números de tarjetas)
    if (typeof info.message === 'string') {
      info.message = info.message.replace(/\b(?:\d[ -]*?){13,16}\b/g, '[TARJETA REDACTADA]');
    }
    
    this.emit('logged', info);
    callback();
  }
}

// Configurar Winston
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'api-service' },
  transports: [
    new SanitizingTransport(),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// En producción, también loggear a la consola (formato legible para humanos)
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }));
}

// Uso
logger.info('Usuario ha iniciado sesión', { userId: 'user123', timestamp: new Date() });
logger.error('Error al procesar pago', { 
  error: 'Payment rejected', 
  user: 'user123',
  creditCard: '1234-5678-9012-3456' // Será sanitizado automáticamente
});
```

### 3.5.2 Detección de Intrusiones:

Lógica: Implementar sistemas para detectar y alertar sobre comportamientos sospechosos que podrían indicar un intento de intrusión.

```typescript
// Ejemplo conceptual de detección de intrusiones básica
import { Request, Response, NextFunction } from 'express';
import redis from 'redis';
import { promisify } from 'util';

// Cliente Redis para almacenar contadores y estado
const redisClient = redis.createClient({
  host: process.env.REDIS_HOST,
  port: Number(process.env.REDIS_PORT)
});

const incrAsync = promisify(redisClient.incr).bind(redisClient);
const expireAsync = promisify(redisClient.expire).bind(redisClient);
const getAsync = promisify(redisClient.get).bind(redisClient);
const setAsync = promisify(redisClient.set).bind(redisClient);

/**
 * Middleware para detectar intentos de fuerza bruta en login
 */
export async function bruteForceDetection(req: Request, res: Response, next: NextFunction) {
  const ip = req.ip;
  const username = req.body.username;
  
  // Clave única para este IP + usuario
  const key = `login-attempts:${ip}:${username}`;
  
  try {
    // Incrementar contador de intentos
    const attempts = await incrAsync(key);
    
    // Establecer tiempo de expiración si es el primer intento
    if (attempts === 1) {
      await expireAsync(key, 3600); // Expira después de 1 hora
    }
    
    // Si hay demasiados intentos, bloquear
    if (attempts > 5) {
      // Registrar el evento de seguridad
      const blockKey = `blocked:${ip}`;
      await setAsync(blockKey, 'true');
      await expireAsync(blockKey, 3600 * 24); // Bloquear por 24 horas
      
      // Enviar alerta a un sistema de monitoreo
      sendSecurityAlert({
        type: 'brute_force_attempt',
        ip,
        username,
        attempts,
        timestamp: new Date()
      });
      
      return res.status(429).json({
        error: 'Demasiados intentos de inicio de sesión. Cuenta bloqueada temporalmente.'
      });
    }
    
    // Si llegamos aquí, permitir que continúe
    next();
  } catch (error) {
    console.error('Error en detección de fuerza bruta:', error);
    // En caso de error, permitir continuar para no bloquear la funcionalidad
    next();
  }
}

/**
 * Middleware para detectar comportamiento anómalo basado en patrones
 */
export async function anomalyDetection(req: Request, res: Response, next: NextFunction) {
  try {
    const ip = req.ip;
    const endpoint = req.path;
    const method = req.method;
    
    // Verificar si el IP está bloqueado
    const isBlocked = await getAsync(`blocked:${ip}`);
    if (isBlocked) {
      return res.status(403).json({ error: 'Acceso bloqueado' });
    }
    
    // Ejemplos de patrones sospechosos:
    
    // 1. Demasiadas solicitudes en poco tiempo
    const requestKey = `requests:${ip}:${Date.now().toString().substr(0, 10)}`; // Por cada 10 segundos
    const requestCount = await incrAsync(requestKey);
    await expireAsync(requestKey, 10); // Expira después de 10 segundos
    
    if (requestCount > 50) { // Umbral arbitrario
      sendSecurityAlert({
        type: 'rate_limit_exceeded',
        ip,
        endpoint,
        requestCount,
        timestamp: new Date()
      });
      return res.status(429).json({ error: 'Demasiadas solicitudes' });
    }
    
    // 2. Acceso a endpoints sensibles
    const sensitiveEndpoints = ['/admin', '/config', '/users/all', '/api/keys'];
    if (sensitiveEndpoints.some(e => endpoint.includes(e))) {
      // Registrar acceso a endpoint sensible
      sendSecurityAlert({
        type: 'sensitive_endpoint_access',
        ip,
        endpoint,
        method,
        user: req.user?.id || 'anonymous',
        timestamp: new Date()
      });
      
      // No bloqueamos, solo monitoreamos en este caso
    }
    
    // 3. Patrones de ataque conocidos en URL o parámetros
    const urlPattern = req.url.toLowerCase();
    const suspiciousPatterns = ['../..', 'cmd=', 'exec=', 'select+from', 'union+select', '<script>'];
    
    if (suspiciousPatterns.some(pattern => urlPattern.includes(pattern))) {
      sendSecurityAlert({
        type: 'suspicious_request_pattern',
        ip,
        url: req.url,
        pattern: suspiciousPatterns.find(p => urlPattern.includes(p)),
        timestamp: new Date()
      });
      return res.status(400).json({ error: 'Solicitud inválida' });
    }
    
    next();
  } catch (error) {
    console.error('Error en detección de anomalías:', error);
    next();
  }
}

// Función para enviar alertas a sistemas de monitoreo
function sendSecurityAlert(alert: any) {
  // En implementación real, esto podría enviar a:
  // - Sistema SIEM (Security Information and Event Management)
  // - Servicio de monitoreo como Datadog, New Relic, etc.
  // - Sistema de notificaciones (email, Slack, etc.)
  console.warn('ALERTA DE SEGURIDAD:', JSON.stringify(alert));
  
  // Ejemplo: Enviar a webhook
  fetch(process.env.SECURITY_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(alert)
  }).catch(err => console.error('Error al enviar alerta:', err));
}
```

### 3.5.3 Plan de Respuesta a Incidentes:

Lógica: Ten un plan claro y documentado sobre cómo responder a una brecha de seguridad o a un incidente. Esto incluye roles y responsabilidades, pasos para la contención, erradicación, recuperación y análisis post-incidente.

**Simulacros:** Realiza simulacros de incidentes periódicamente para probar y mejorar el plan.

### 3.5.4 Métricas de Seguridad y Reporting:

Lógica: Establecer métricas que permitan evaluar y reportar el estado de la seguridad del sistema a lo largo del tiempo:

- **Tiempo Medio de Detección (MTTD):** Cuánto tiempo pasa desde que ocurre una brecha hasta que se detecta.
- **Tiempo Medio de Respuesta (MTTR):** Cuánto tiempo pasa desde la detección hasta la resolución.
- **Vulnerabilidades Identificadas vs. Corregidas:** Seguimiento del progreso en la corrección de vulnerabilidades conocidas.
- **Intentos de Intrusión Bloqueados:** Número de ataques detectados y bloqueados por los sistemas de seguridad.

### 3.5.5 Actualizaciones y Parcheo:

Lógica: Implementar un proceso consistente para aplicar actualizaciones de seguridad a todas las dependencias y componentes del sistema. La mayoría de las brechas de seguridad explotan vulnerabilidades conocidas que ya tienen parches disponibles.

```typescript
// Ejemplo de script para verificar dependencias obsoletas y con vulnerabilidades
const { execSync } = require('child_process');
const fs = require('fs');

// Función para ejecutar un comando y capturar su salida
function run(command) {
  try {
    return execSync(command, { encoding: 'utf8' });
  } catch (error) {
    console.error(`Error ejecutando: ${command}`);
    console.error(error.stdout || error.message);
    return null;
  }
}

// Verificar si hay actualizaciones disponibles
console.log('Verificando actualizaciones disponibles...');
const outdated = run('npm outdated --json');

if (outdated) {
  const outdatedJSON = JSON.parse(outdated);
  const outdatedCount = Object.keys(outdatedJSON).length;
  
  console.log(`Se encontraron ${outdatedCount} dependencias desactualizadas:`);
  for (const [pkg, info] of Object.entries(outdatedJSON)) {
    console.log(`- ${pkg}: ${info.current} -> ${info.latest} (wanted: ${info.wanted})`);
  }
}

// Ejecutar auditoría de seguridad
console.log('\nEjecutando auditoría de seguridad...');
const audit = run('npm audit --json');

if (audit) {
  const auditJSON = JSON.parse(audit);
  const vulnerabilities = auditJSON.vulnerabilities;
  
  if (vulnerabilities) {
    const total = 
      (vulnerabilities.low || 0) + 
      (vulnerabilities.moderate || 0) + 
      (vulnerabilities.high || 0) + 
      (vulnerabilities.critical || 0);
    
    console.log(`Se encontraron ${total} vulnerabilidades:`);
    console.log(`- Críticas: ${vulnerabilities.critical || 0}`);
    console.log(`- Altas: ${vulnerabilities.high || 0}`);
    console.log(`- Moderadas: ${vulnerabilities.moderate || 0}`);
    console.log(`- Bajas: ${vulnerabilities.low || 0}`);
    
    // Guardar reporte detallado
    fs.writeFileSync('security-report.json', audit);
    console.log('Reporte detallado guardado en security-report.json');
    
    // Para vulnerabilidades críticas y altas, recomendar actualización inmediata
    if ((vulnerabilities.critical || 0) > 0 || (vulnerabilities.high || 0) > 0) {
      console.log('\n⚠️ ACCIÓN REQUERIDA: Se encontraron vulnerabilidades críticas o altas.');
      console.log('Ejecuta `npm audit fix` para corregir automáticamente los problemas que se puedan.');
      console.log('Para vulnerabilidades que requieren actualizaciones mayores:');
      console.log('1. Revisa el reporte detallado');
      console.log('2. Planifica las actualizaciones necesarias');
      console.log('3. Prueba exhaustivamente después de actualizar');
    }
  } else {
    console.log('No se encontraron vulnerabilidades. ¡Buen trabajo!');
  }
}
```
