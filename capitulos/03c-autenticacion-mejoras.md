### 3.5.6 Autenticación Multifactor (MFA)

La autenticación multifactor (MFA) representa una de las medidas más efectivas para proteger las cuentas de usuario contra ataques de credenciales comprometidas. Según Microsoft, la MFA puede bloquear más del 99.9% de los intentos de compromiso de cuentas.

#### Tipos de Factores de Autenticación

Los factores de autenticación se dividen en tres categorías principales:

1. **Algo que sabes**: Contraseñas, PINs, respuestas a preguntas de seguridad.
2. **Algo que tienes**: Dispositivos físicos como tokens de seguridad, smartphones, tarjetas inteligentes.
3. **Algo que eres**: Factores biométricos como huellas dactilares, reconocimiento facial o de iris.

Una implementación robusta de MFA debería combinar al menos dos categorías diferentes.

#### Implementación de Autenticación por Tiempo (TOTP)

Uno de los métodos más populares de MFA es el uso de contraseñas de un solo uso basadas en tiempo (TOTP), implementadas en aplicaciones como Google Authenticator, Microsoft Authenticator o Authy.

```typescript
// Implementación de TOTP en Node.js con otplib
import { authenticator } from 'otplib';

class TOTPService {
  // Genera un nuevo secreto para un usuario
  generateSecret(userName: string): { secret: string; uri: string } {
    // Generar un secreto aleatorio
    const secret = authenticator.generateSecret();
    
    // Crear una URI que pueda ser usada para generar un código QR
    const uri = authenticator.keyuri(userName, 'MiAplicaciónSegura', secret);
    
    return { secret, uri };
  }
  
  // Verificar un código TOTP proporcionado por el usuario
  verifyToken(token: string, secret: string): boolean {
    try {
      // La verificación comprueba automáticamente que el token sea válido
      // para la ventana de tiempo actual (30 segundos por defecto)
      return authenticator.verify({ token, secret });
    } catch (error) {
      console.error('Error al verificar token TOTP:', error);
      return false;
    }
  }
}

// Ejemplo de uso en un flujo de registro
async function registrarMFA(req: Request, res: Response) {
  const userId = req.user.id;
  const totpService = new TOTPService();
  
  // Generar un secreto para el usuario
  const { secret, uri } = totpService.generateSecret(req.user.email);
  
  // Almacenar el secreto en la base de datos asociado al usuario
  await userModel.update(userId, { totpSecret: secret, mfaEnabled: false });
  
  // Generar un código QR usando el URI
  const qrCode = await generateQRCode(uri);
  
  // Enviar la imagen del código QR al cliente
  res.json({
    qrCode,
    message: 'Escanea este código QR con tu aplicación de autenticación'
  });
}

// Ejemplo de verificación en el proceso de activación de MFA
async function activarMFA(req: Request, res: Response) {
  const userId = req.user.id;
  const { token } = req.body;
  
  // Obtener el secreto del usuario desde la base de datos
  const user = await userModel.findById(userId);
  
  if (!user || !user.totpSecret) {
    return res.status(400).json({ message: 'MFA no configurado para este usuario' });
  }
  
  const totpService = new TOTPService();
  
  // Verificar el token proporcionado
  if (totpService.verifyToken(token, user.totpSecret)) {
    // Activar MFA para el usuario
    await userModel.update(userId, { mfaEnabled: true });
    return res.json({ message: 'MFA activado correctamente' });
  }
  
  return res.status(400).json({ message: 'Código incorrecto o expirado' });
}

// Ejemplo de uso en el proceso de inicio de sesión
async function verificarMFA(req: Request, res: Response) {
  const { userId, token } = req.body;
  
  // Obtener el usuario y su secreto TOTP
  const user = await userModel.findById(userId);
  
  if (!user || !user.mfaEnabled || !user.totpSecret) {
    return res.status(400).json({ message: 'MFA no habilitado o mal configurado' });
  }
  
  const totpService = new TOTPService();
  
  // Verificar el token
  if (totpService.verifyToken(token, user.totpSecret)) {
    // Generar sesión o token de autenticación completa
    const sessionToken = generateSessionToken(user);
    return res.json({ token: sessionToken });
  }
  
  // Token inválido
  return res.status(401).json({ message: 'Código de verificación incorrecto' });
}
```

#### Implementación de WebAuthn / FIDO2

WebAuthn es un estándar web que permite la autenticación sin contraseñas usando llaves de seguridad físicas o la biometría integrada en dispositivos.

```typescript
// Ejemplo simplificado de implementación de WebAuthn con Node.js
import { 
  generateRegistrationOptions,
  verifyRegistrationResponse,
  generateAuthenticationOptions,
  verifyAuthenticationResponse
} from '@simplewebauthn/server';
import { isoUint8Array } from '@simplewebauthn/server/helpers';
import type { 
  RegistrationResponseJSON,
  AuthenticationResponseJSON 
} from '@simplewebauthn/typescript-types';

const rpName = 'Mi Aplicación Segura';
const rpID = 'miapp.com';
const origin = `https://${rpID}`;

// Registro de una nueva credencial WebAuthn
async function generarOpcionesRegistro(userId: string, userName: string) {
  // Obtener credenciales existentes para este usuario
  const user = await userModel.findById(userId);
  const existingCredentials = user?.credentials || [];
  
  // Generar opciones para el cliente
  const options = generateRegistrationOptions({
    rpName,
    rpID,
    userID: userId,
    userName,
    // Evitar registros duplicados
    excludeCredentials: existingCredentials.map(cred => ({
      id: isoUint8Array.fromHex(cred.credentialID),
      type: 'public-key',
      transports: cred.transports || [],
    })),
    authenticatorSelection: {
      // Solicitar verificación de usuario (biometría, PIN, etc.)
      userVerification: 'preferred',
      // Adjuntar o desconectado (llaves de seguridad físicas)
      authenticatorAttachment: 'platform',
    },
    supportedAlgorithmIDs: [-7, -257], // ES256, RS256
  });
  
  // Guardar el desafío para verificar la respuesta
  await userModel.update(userId, { 
    currentChallenge: options.challenge 
  });
  
  return options;
}

// Verificar la respuesta de registro
async function verificarRegistro(userId: string, response: RegistrationResponseJSON) {
  // Obtener el desafío guardado previamente
  const user = await userModel.findById(userId);
  const expectedChallenge = user?.currentChallenge;
  
  if (!expectedChallenge) {
    throw new Error('No se encontró desafío para este usuario');
  }
  
  // Verificar la respuesta
  const verification = await verifyRegistrationResponse({
    response,
    expectedChallenge,
    expectedOrigin: origin,
    expectedRPID: rpID,
  });
  
  if (verification.verified && verification.registrationInfo) {
    // Guardar la credencial en la base de datos
    const newCredential = {
      credentialID: isoUint8Array.toHex(verification.registrationInfo.credentialID),
      publicKey: isoUint8Array.toHex(verification.registrationInfo.credentialPublicKey),
      counter: verification.registrationInfo.counter,
      transports: response.response.transports,
    };
    
    // Añadir la nueva credencial a la lista del usuario
    const credentials = [...(user.credentials || []), newCredential];
    await userModel.update(userId, { 
      credentials,
      currentChallenge: null,
      webAuthnEnabled: true
    });
    
    return { verified: true };
  }
  
  throw new Error('Fallo en la verificación del registro');
}

// Generar opciones para autenticación
async function generarOpcionesAutenticacion(userId: string) {
  // Obtener credenciales del usuario
  const user = await userModel.findById(userId);
  const credentials = user?.credentials || [];
  
  if (credentials.length === 0) {
    throw new Error('Usuario sin credenciales WebAuthn registradas');
  }
  
  // Generar opciones para el cliente
  const options = generateAuthenticationOptions({
    rpID,
    // Solo permitir credenciales registradas para este usuario
    allowCredentials: credentials.map(cred => ({
      id: isoUint8Array.fromHex(cred.credentialID),
      type: 'public-key',
      transports: cred.transports || [],
    })),
    userVerification: 'preferred',
  });
  
  // Guardar el desafío
  await userModel.update(userId, { 
    currentChallenge: options.challenge 
  });
  
  return options;
}

// Verificar la respuesta de autenticación
async function verificarAutenticacion(userId: string, response: AuthenticationResponseJSON) {
  // Obtener usuario y sus credenciales
  const user = await userModel.findById(userId);
  const expectedChallenge = user?.currentChallenge;
  
  if (!expectedChallenge || !user?.credentials) {
    throw new Error('No se encontró desafío o credenciales para este usuario');
  }
  
  // Encontrar la credencial específica usada
  const credentialID = response.id;
  const credential = user.credentials.find(
    cred => cred.credentialID === credentialID
  );
  
  if (!credential) {
    throw new Error('Credencial no reconocida');
  }
  
  // Verificar la autenticación
  const verification = await verifyAuthenticationResponse({
    response,
    expectedChallenge,
    expectedOrigin: origin,
    expectedRPID: rpID,
    authenticator: {
      credentialID: isoUint8Array.fromHex(credential.credentialID),
      credentialPublicKey: isoUint8Array.fromHex(credential.publicKey),
      counter: credential.counter,
    },
  });
  
  if (verification.verified) {
    // Actualizar el contador de la credencial
    const credentials = user.credentials.map(cred => 
      cred.credentialID === credentialID
        ? { ...cred, counter: verification.authenticationInfo.newCounter }
        : cred
    );
    
    await userModel.update(userId, { 
      credentials,
      currentChallenge: null,
    });
    
    // Generar sesión o token de autenticación completa
    const sessionToken = generateSessionToken(user);
    
    return { verified: true, token: sessionToken };
  }
  
  throw new Error('Fallo en la verificación de la autenticación');
}
```

### 3.5.7 JSON Web Tokens (JWT)

Los JSON Web Tokens (JWT) son un estándar de la industria (RFC 7519) para representar claims (afirmaciones) de forma segura entre dos partes. Se han convertido en el mecanismo preferido para implementar autenticación y autorización en APIs modernas debido a su naturaleza sin estado (stateless).

#### Estructura de un JWT

Un JWT consta de tres partes separadas por puntos (`.`):

1. **Header (Cabecera)**: Contiene el tipo de token y el algoritmo de firma.
2. **Payload (Carga útil)**: Contiene las claims o afirmaciones sobre la entidad (usuario) y metadatos adicionales.
3. **Signature (Firma)**: Asegura que el token no ha sido alterado durante su transmisión.

```typescript
// Ejemplo de implementación de autenticación JWT con Node.js
import jwt from 'jsonwebtoken';
import { Request, Response, NextFunction } from 'express';

const JWT_SECRET = process.env.JWT_SECRET || 'clave-muy-secreta-y-larga-para-desarrollo';
const JWT_EXPIRES_IN = '1h'; // 1 hora

// Generar un token JWT al iniciar sesión
export function generarToken(usuario: any) {
  // Payload con información relevante pero no sensible
  const payload = {
    sub: usuario.id, // subject (identificador único)
    name: usuario.nombre,
    email: usuario.email,
    roles: usuario.roles,
    permissions: usuario.permissions,
    // Evitar incluir datos sensibles como contraseñas o PII no necesaria
  };
  
  // Opciones del token
  const options = {
    expiresIn: JWT_EXPIRES_IN, // Tiempo de vida del token
    issuer: 'mi-app-segura', // Emisor del token
    audience: 'mi-api', // Audiencia prevista
    jwtid: generarUUID(), // ID único para este token (para revocación)
    notBefore: 0, // Token válido inmediatamente
  };
  
  // Generar y firmar el token
  return jwt.sign(payload, JWT_SECRET, options);
}

// Middleware para verificar token en rutas protegidas
export function verificarToken(req: Request, res: Response, next: NextFunction) {
  // Obtener el token de la cabecera de autorización
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ message: 'No se proporcionó token de autenticación' });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    // Verificar el token
    const decoded = jwt.verify(token, JWT_SECRET);
    
    // Verificar si el token ha sido revocado
    // (esto requiere una implementación personalizada, generalmente usando Redis)
    if (isTokenRevoked(decoded.jti)) {
      return res.status(401).json({ message: 'Token revocado' });
    }
    
    // Agregar la información decodificada a la solicitud
    req.user = decoded;
    
    next();
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      return res.status(401).json({ message: 'Token expirado' });
    } else if (error instanceof jwt.JsonWebTokenError) {
      return res.status(401).json({ message: 'Token inválido' });
    }
    
    // Cualquier otro error
    return res.status(500).json({ message: 'Error al verificar token' });
  }
}

// Sistema de renovación de tokens (refresh tokens)
export function generarRefreshToken(usuario: any) {
  // El refresh token debe tener una vigencia más larga
  return jwt.sign(
    { sub: usuario.id, tipo: 'refresh' },
    process.env.JWT_REFRESH_SECRET || 'clave-refresh-separada-y-segura',
    { expiresIn: '7d', jwtid: generarUUID() } // 7 días
  );
}

// Endpoint para renovar tokens usando un refresh token
export async function renovarToken(req: Request, res: Response) {
  const { refreshToken } = req.body;
  
  if (!refreshToken) {
    return res.status(400).json({ message: 'Se requiere refresh token' });
  }
  
  try {
    // Verificar el refresh token
    const decoded = jwt.verify(
      refreshToken, 
      process.env.JWT_REFRESH_SECRET || 'clave-refresh-separada-y-segura'
    );
    
    // Verificar que sea de tipo refresh
    if (decoded.tipo !== 'refresh') {
      return res.status(401).json({ message: 'Tipo de token inválido' });
    }
    
    // Comprobar si el token ha sido revocado
    if (isRefreshTokenRevoked(decoded.jti)) {
      return res.status(401).json({ message: 'Refresh token revocado' });
    }
    
    // Obtener el usuario asociado
    const usuario = await userModel.findById(decoded.sub);
    
    if (!usuario) {
      return res.status(401).json({ message: 'Usuario no encontrado' });
    }
    
    // Generar un nuevo token de acceso
    const nuevoToken = generarToken(usuario);
    
    // Opcionalmente, generar un nuevo refresh token y revocar el anterior
    // (rotación de refresh tokens, mejora la seguridad)
    const nuevoRefreshToken = generarRefreshToken(usuario);
    revocarRefreshToken(decoded.jti);
    
    return res.json({
      token: nuevoToken,
      refreshToken: nuevoRefreshToken
    });
  } catch (error) {
    console.error('Error al renovar token:', error);
    return res.status(401).json({ message: 'Refresh token inválido o expirado' });
  }
}

// Sistema de revocación de tokens
let tokenRevocationList = new Set(); // En producción, usar Redis o similar

// Revocar un token de acceso
export function revocarToken(jti: string) {
  tokenRevocationList.add(jti);
  // En una implementación real, establecer un TTL igual a la expiración del token
}

// Verificar si un token ha sido revocado
export function isTokenRevoked(jti: string) {
  return tokenRevocationList.has(jti);
}

// Implementaciones para refresh tokens (similares)
let refreshTokenRevocationList = new Set();
export function revocarRefreshToken(jti: string) {
  refreshTokenRevocationList.add(jti);
}
export function isRefreshTokenRevoked(jti: string) {
  return refreshTokenRevocationList.has(jti);
}

// Función de ayuda para generar UUIDs
function generarUUID() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
    const r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}
```

#### Mejores Prácticas con JWT

1. **Usar algoritmos seguros**: Preferir RS256 (asimétrico) sobre HS256 (simétrico) para sistemas en producción.

2. **Establecer expiración corta**: Limitar la vida útil de los tokens de acceso (15-60 minutos).

3. **Implementar un sistema de renovación**: Usar refresh tokens con mayor duración pero almacenados de forma segura.

4. **Mecanismo de revocación**: Implementar una "lista negra" de tokens revocados, normalmente con Redis.

5. **No almacenar datos sensibles**: Los JWT no están cifrados por defecto, solo firmados.

6. **Validar todas las reclamaciones**: Verificar issuer, audience, expiration, etc., no solo la firma.

7. **Protección contra algoritmo "none"**: Configurar explícitamente los algoritmos aceptados.

## Conclusiones

La autenticación y gestión de sesiones seguras son componentes fundamentales de cualquier sistema de software moderno. A lo largo de este capítulo, hemos explorado diversas técnicas y estrategias para implementar estos mecanismos de forma robusta:

1. **Defensa en profundidad**: La combinación de múltiples capas de seguridad, como cookies seguras, protección CSRF, cabeceras HTTP de seguridad y autenticación multifactor, proporciona una protección más efectiva que cualquier medida individual.

2. **Balance entre seguridad y usabilidad**: Al implementar mecanismos como los refresh tokens y la autenticación progresiva, podemos mantener un alto nivel de seguridad sin sacrificar significativamente la experiencia del usuario.

3. **Actualización constante**: Las amenazas de seguridad evolucionan rápidamente, por lo que es esencial mantenerse al día con las últimas vulnerabilidades y mejores prácticas, realizando auditorías y pruebas de seguridad de forma regular.

4. **Monitoreo y respuesta**: Un sistema de detección de anomalías y un plan claro de respuesta a incidentes son tan importantes como las medidas preventivas.

5. **Estándares abiertos**: El uso de estándares como JWT, OAuth 2.0, OpenID Connect y WebAuthn proporciona soluciones probadas y revisadas por la comunidad de seguridad.

Recordemos que la seguridad perfecta no existe, pero mediante la implementación consistente de estas prácticas y el mantenimiento de una mentalidad de "seguridad por diseño", podemos reducir significativamente el riesgo y proteger eficazmente los datos y la privacidad de nuestros usuarios.
