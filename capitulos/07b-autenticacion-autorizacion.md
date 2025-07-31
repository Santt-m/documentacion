# 7.2 Autenticación y Autorización

La autenticación y autorización son pilares fundamentales en la seguridad de aplicaciones modernas, permitiendo verificar la identidad de usuarios y controlar su acceso a recursos específicos. Este capítulo explora métodos robustos para implementar estos sistemas en aplicaciones backend.

## Índice

- [7.2.1 Fundamentos de Autenticación y Autorización](#721-fundamentos-de-autenticación-y-autorización)
- [7.2.2 Autenticación con JSON Web Tokens (JWT)](#722-autenticación-con-json-web-tokens-jwt)
- [7.2.3 Implementación de Refresh Tokens](#723-implementación-de-refresh-tokens)
- [7.2.4 OAuth 2.0 y OpenID Connect](#724-oauth-20-y-openid-connect)
- [7.2.5 Autenticación con Proveedores Externos](#725-autenticación-con-proveedores-externos)
- [7.2.6 Sistemas de Roles y Permisos](#726-sistemas-de-roles-y-permisos)
- [7.2.7 Mejores Prácticas de Seguridad](#727-mejores-prácticas-de-seguridad)

## 7.2.1 Fundamentos de Autenticación y Autorización

### Diferencias entre Autenticación y Autorización

- **Autenticación**: Proceso de verificar la identidad de un usuario o sistema.
- **Autorización**: Proceso de determinar si un usuario autenticado tiene permiso para acceder a un recurso o realizar una acción específica.

### Flujos Básicos

```
+----------------+          +----------------+          +----------------+
|                |  1. Solicita    |                |  3. Verifica   |                |
|    Cliente     | ------------>   |    Servidor    | ------------>   |     Base      |
|                | credenciales    |                | credenciales    |   de Datos    |
|                |                 |                |                 |                |
|                |  2. Proporciona |                |  4. Resultado   |                |
|                | <------------   |                | <------------   |                |
+----------------+  token/sesión   +----------------+  verificación   +----------------+
       ^                                    |
       |                                    |
       |  5. Solicita recurso               |
       |  con token/sesión                  |
       +------------------------------------+
                       |
                       v
                  6. Autoriza y
                   devuelve recurso
```

### Factores de Autenticación

1. **Algo que sabes**: Contraseña, PIN, respuesta a pregunta de seguridad.
2. **Algo que tienes**: Token físico, smartphone (para recibir códigos).
3. **Algo que eres**: Huella digital, reconocimiento facial, biometría.

La autenticación multifactor (MFA) combina dos o más de estos factores para aumentar la seguridad.

## 7.2.2 Autenticación con JSON Web Tokens (JWT)

JSON Web Tokens (JWT) es un estándar abierto (RFC 7519) que define un método compacto y autónomo para transmitir información de forma segura entre partes como un objeto JSON. Esta información puede ser verificada y confiable porque está firmada digitalmente.

### Estructura de un JWT

Un JWT consiste en tres partes separadas por puntos:

```
header.payload.signature
```

1. **Header (Cabecera)**: Contiene el tipo de token y el algoritmo de firma.
   ```json
   {
     "alg": "HS256",
     "typ": "JWT"
   }
   ```

2. **Payload (Carga útil)**: Contiene las afirmaciones (claims) sobre la entidad (usuario) y metadata.
   ```json
   {
     "sub": "1234567890",
     "name": "John Doe",
     "admin": true,
     "iat": 1516239022,
     "exp": 1516242622
   }
   ```

3. **Signature (Firma)**: Creada a partir del header codificado, payload codificado, una clave secreta y el algoritmo especificado.
   ```
   HMACSHA256(
     base64UrlEncode(header) + "." +
     base64UrlEncode(payload),
     secret)
   ```

### Claims (Afirmaciones) Comunes

- **Registradas**: Conjunto de claims predefinidos que no son obligatorios pero sí recomendados.
  - **iss** (issuer): Emisor del JWT
  - **sub** (subject): Sujeto del token (normalmente el ID de usuario)
  - **exp** (expiration time): Tiempo de expiración
  - **iat** (issued at): Tiempo de emisión
  - **nbf** (not before): Tiempo antes del cual el token no debe ser aceptado
  - **jti** (JWT ID): Identificador único del token
  - **aud** (audience): Audiencia del token

- **Públicas**: Claims definidas a gusto, pero para evitar colisiones deben estar registradas en el IANA JSON Web Token Registry.

- **Privadas**: Claims personalizadas creadas para compartir información entre las partes.

### Implementación Básica de JWT en Node.js

A continuación se muestra cómo implementar autenticación JWT con Express, usando la biblioteca `jsonwebtoken`:

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const app = express();
app.use(express.json());

// En producción, usar una base de datos real
const users = [];

// Clave secreta - en producción guardar en variables de entorno
const JWT_SECRET = process.env.JWT_SECRET || 'your_jwt_secret';

// Ruta para registro de usuario
app.post('/api/register', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    // Comprobar si el usuario ya existe
    if (users.find(u => u.username === username)) {
      return res.status(400).json({ message: 'Usuario ya existe' });
    }
    
    // Hash de contraseña
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Crear nuevo usuario
    const user = {
      id: users.length + 1,
      username,
      password: hashedPassword
    };
    
    users.push(user);
    
    res.status(201).json({ message: 'Usuario creado exitosamente' });
  } catch (error) {
    res.status(500).json({ message: 'Error en el servidor' });
  }
});

// Ruta para login
app.post('/api/login', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    // Buscar usuario
    const user = users.find(u => u.username === username);
    if (!user) {
      return res.status(401).json({ message: 'Credenciales inválidas' });
    }
    
    // Verificar contraseña
    const validPassword = await bcrypt.compare(password, user.password);
    if (!validPassword) {
      return res.status(401).json({ message: 'Credenciales inválidas' });
    }
    
    // Generar token JWT
    const token = jwt.sign(
      { id: user.id, username: user.username },
      JWT_SECRET,
      { expiresIn: '2h' }
    );
    
    res.json({ token });
  } catch (error) {
    res.status(500).json({ message: 'Error en el servidor' });
  }
});

// Middleware de autenticación
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
  
  if (!token) return res.status(401).json({ message: 'Token no proporcionado' });
  
  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) return res.status(403).json({ message: 'Token inválido' });
    
    req.user = user;
    next();
  });
}

// Ruta protegida
app.get('/api/profile', authenticateToken, (req, res) => {
  res.json({ message: 'Ruta protegida', user: req.user });
});
```

### Flujo completo de autenticación con JWT

1. **El usuario se autentica** con credenciales (nombre usuario/contraseña)
2. **El servidor verifica** las credenciales contra la base de datos
3. **Si son correctas**, el servidor genera un JWT firmado con información del usuario
4. **El token se envía al cliente**, que lo almacena (localStorage, cookie, etc.)
5. **En peticiones posteriores**, el cliente incluye el token en el header de autorización
6. **El servidor valida** el token en cada petición a rutas protegidas
7. **Si el token es válido**, se permite el acceso a los recursos solicitados

### Mejores prácticas de seguridad con JWT

1. **Usar HTTPS** para todas las comunicaciones que involucren JWT
2. **Almacenar secretos de forma segura** usando variables de entorno
3. **Establecer tiempos de expiración cortos** y usar refresh tokens si es necesario
4. **Validar claims**: emisor, audiencia, tiempo de expiración, etc.
5. **Minimizar datos sensibles** en el payload (el JWT no está encriptado, solo firmado)
6. **Implementar revocación de tokens** para casos de compromiso o logout
7. **Usar algoritmos de firma seguros** (RS256 en lugar de HS256 para sistemas a gran escala)
8. **Incluir fingerprint** o hash único de dispositivo para prevenir robo de tokens
9. **Protección XSS**: Almacenar tokens en cookies HttpOnly con la bandera Secure

## 7.2.3 Implementación de Refresh Tokens

Los refresh tokens son una parte fundamental de la arquitectura de autenticación JWT, permitiendo mantener sesiones de larga duración sin comprometer la seguridad.

### ¿Qué son los Refresh Tokens?

Un refresh token es un tipo especial de token que se utiliza para obtener un nuevo access token (JWT) cuando este ha expirado, sin necesidad de que el usuario vuelva a autenticarse con sus credenciales.

### Ventajas de los Refresh Tokens

1. **Mejora de seguridad**: Permite que los access tokens tengan una vida útil muy corta (minutos)
2. **Experiencia de usuario mejorada**: El usuario no necesita volver a iniciar sesión constantemente
3. **Revocación efectiva**: Permite invalidar sesiones activas cuando sea necesario
4. **Control de sesión**: Facilita la gestión de múltiples dispositivos y sesiones

### Flujo de Autenticación con Refresh Tokens

```
+----------------+          +----------------+          +----------------+
|                |  1. Login     |                |          |                |
|    Cliente     | ------------>|    Servidor    |--------->|     Base      |
|                |               |                |          |   de Datos    |
|                |  2. Access +  |                |          |                |
|                |<-------------|                |<---------|                |
+----------------+ Refresh Token +----------------+          +----------------+
      |  |                                                       ^
      |  |                                                       |
      |  | 3. Solicitud con                                      |
      |  |    Access Token                                       |
      |  |    (hasta expirar)                                    |
      |  v                                                       |
+----------------+                                               |
|                |                                               |
|    API         |                                               |
|   Recursos     |                                               |
|                |                                               |
+----------------+                                               |
      ^                                                         |
      |  4. Access Token                                        |
      |     expirado                                            |
      |                                                         |
      |  5. Solicitud de                                        |
      |     nuevo token                                         |
      |     con Refresh Token                                   |
      |                                                         |
+----------------+          +----------------+                   |
|                |          |                |                   |
|    Cliente     |--------->|    Servidor    |-------------------+
|                | Refresh  |                | Verifica validez
|                |<---------|                | del Refresh Token
+----------------+ Nuevo    +----------------+
                  Access Token
```

### Implementación de Refresh Tokens en Node.js

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const { v4: uuidv4 } = require('uuid');
require('dotenv').config();

const app = express();
app.use(express.json());

// En producción usaremos una base de datos real
const users = [];
const refreshTokens = {}; // Almacén de refresh tokens

// Constantes de configuración (en producción usaríamos variables de entorno)
const JWT_SECRET = process.env.JWT_SECRET || 'your_jwt_secret';
const REFRESH_SECRET = process.env.REFRESH_SECRET || 'your_refresh_secret';
const ACCESS_TOKEN_EXPIRY = '15m';  // Corta duración
const REFRESH_TOKEN_EXPIRY = '7d';  // Larga duración

// Ruta para login con refresh token
app.post('/api/login', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    // Buscar usuario
    const user = users.find(u => u.username === username);
    if (!user) {
      return res.status(401).json({ message: 'Credenciales inválidas' });
    }
    
    // Verificar contraseña
    const validPassword = await bcrypt.compare(password, user.password);
    if (!validPassword) {
      return res.status(401).json({ message: 'Credenciales inválidas' });
    }
    
    // Generar tokens
    const accessToken = generateAccessToken(user);
    const refreshToken = generateRefreshToken(user);
    
    // Almacenar refresh token (en producción sería en base de datos)
    refreshTokens[refreshToken] = {
      userId: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 días
    };
    
    res.json({ 
      accessToken,
      refreshToken,
      expiresIn: 15 * 60, // segundos
      user: { id: user.id, username: user.username }
    });
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: 'Error en el servidor' });
  }
});

// Ruta para obtener nuevo access token usando refresh token
app.post('/api/refresh-token', (req, res) => {
  const { refreshToken } = req.body;
  
  if (!refreshToken || !refreshTokens[refreshToken]) {
    return res.status(401).json({ message: 'Refresh token inválido' });
  }
  
  try {
    // Verificar refresh token
    const decoded = jwt.verify(refreshToken, REFRESH_SECRET);
    const storedToken = refreshTokens[refreshToken];
    
    if (decoded.id !== storedToken.userId || new Date() > new Date(storedToken.expiresAt)) {
      throw new Error('Token expirado o inválido');
    }
    
    // Encontrar usuario
    const user = users.find(u => u.id === decoded.id);
    if (!user) {
      throw new Error('Usuario no encontrado');
    }
    
    // Generar nuevo access token
    const accessToken = generateAccessToken(user);
    
    res.json({ 
      accessToken,
      expiresIn: 15 * 60 // segundos
    });
  } catch (error) {
    delete refreshTokens[refreshToken]; // Invalidar token comprometido
    return res.status(403).json({ message: 'Refresh token inválido o expirado' });
  }
});

// Ruta para logout
app.post('/api/logout', (req, res) => {
  const { refreshToken } = req.body;
  
  // Eliminar refresh token
  if (refreshToken && refreshTokens[refreshToken]) {
    delete refreshTokens[refreshToken];
  }
  
  res.json({ message: 'Logout exitoso' });
});

// Funciones auxiliares para generar tokens
function generateAccessToken(user) {
  return jwt.sign(
    { 
      id: user.id, 
      username: user.username,
      type: 'access'
    },
    JWT_SECRET,
    { expiresIn: ACCESS_TOKEN_EXPIRY }
  );
}

function generateRefreshToken(user) {
  return jwt.sign(
    { 
      id: user.id,
      type: 'refresh',
      tokenId: uuidv4() // ID único para poder revocar tokens específicos
    },
    REFRESH_SECRET,
    { expiresIn: REFRESH_TOKEN_EXPIRY }
  );
}
```

### Mejores Prácticas para Refresh Tokens

1. **Almacenamiento seguro**: Los refresh tokens deben almacenarse en una base de datos (no en memoria)
2. **Tiempo de expiración**: Establecer periodos más largos que los access tokens pero con límite
3. **Rotación de tokens**: Generar un nuevo refresh token cada vez que se utiliza
4. **Identificadores únicos**: Cada refresh token debe tener un ID único
5. **Revocación efectiva**: Mantener una lista negra de tokens revocados
6. **Validación de fingerprint**: Asociar el token a una huella del dispositivo
7. **Limitación por usuario**: Limitar el número de refresh tokens activos por usuario

## 7.2.4 OAuth 2.0 y OpenID Connect

OAuth 2.0 es un protocolo de autorización que permite a aplicaciones de terceros obtener acceso limitado a un servicio sin exponer las credenciales del usuario. OpenID Connect (OIDC) es una capa de identidad construida sobre OAuth 2.0 que agrega funcionalidades de autenticación.

### Conceptos clave de OAuth 2.0

- **Resource Owner**: El usuario que posee los recursos protegidos.
- **Client**: La aplicación que solicita acceso a los recursos protegidos.
- **Authorization Server**: El servidor que emite tokens de acceso al cliente.
- **Resource Server**: El servidor que aloja los recursos protegidos.
- **Access Token**: Token que se utiliza para acceder a recursos protegidos.
- **Scope**: Permisos específicos que se solicitan y otorgan.

### Flujos de OAuth 2.0

#### 1. Authorization Code Flow

El flujo más completo y seguro, ideal para aplicaciones con backend:

```
+--------+                               +---------------+
|        |--(1)- Autorización ---------> |               |
|        |                               |  Servidor de  |
|        <--(2)- Código Autorización --- |  Autorización |
|        |                               |               |
| Cliente|--(3)- Código + Secret ------> |               |
|        |                               |               |
|        <--(4)- Access Token ---------- |               |
+--------+                               +---------------+
    |                                           ^
    |                                           |
    v                                           |
+--------+                               +---------------+
|        |--(5)- Access Token --------> |               |
| Recurso|                               |  Servidor de  |
|        <--(6)- Recurso Protegido ---- |   Recursos    |
+--------+                               +---------------+
```

#### 2. Implicit Flow

Flujo simplificado para aplicaciones cliente (obsoleto en favor de PKCE):

```
+--------+                               +---------------+
|        |--(1)- Autorización ---------> |               |
| Cliente|                               |  Servidor de  |
|        <--(2)- Token directamente ---- |  Autorización |
+--------+                               +---------------+
    |                                           ^
    |                                           |
    v                                           |
+--------+                               +---------------+
|        |--(3)- Access Token --------> |               |
| Recurso|                               |  Servidor de  |
|        <--(4)- Recurso Protegido ---- |   Recursos    |
+--------+                               +---------------+
```

#### 3. Client Credentials Flow

Para autenticación entre servicios (sin usuario):

```
+--------+                               +---------------+
|        |--(1)- Client ID + Secret --> |               |
| Cliente|                               |  Servidor de  |
|        <--(2)- Access Token ---------- |  Autorización |
+--------+                               +---------------+
    |                                           ^
    |                                           |
    v                                           |
+--------+                               +---------------+
|        |--(3)- Access Token --------> |               |
| Recurso|                               |  Servidor de  |
|        <--(4)- Recurso Protegido ---- |   Recursos    |
+--------+                               +---------------+
```

### OpenID Connect (OIDC)

OpenID Connect extiende OAuth 2.0 con funcionalidades de autenticación:

- **ID Token**: Token JWT que contiene información de identidad del usuario
- **UserInfo Endpoint**: Endpoint para obtener información adicional del usuario
- **Discovery**: Documento de configuración para descubrir endpoints

### Implementación de OAuth 2.0 con Passport.js

```javascript
const express = require('express');
const session = require('express-session');
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

const app = express();

// Configuración de sesiones (en producción usar almacenamiento seguro como Redis)
app.use(session({
  secret: process.env.SESSION_SECRET || 'tu_secreto_seguro',
  resave: false,
  saveUninitialized: false,
  cookie: { secure: process.env.NODE_ENV === 'production', maxAge: 24 * 60 * 60 * 1000 } // 24 horas
}));

// Inicializar Passport
app.use(passport.initialize());
app.use(passport.session());

// Configurar estrategia de Google OAuth 2.0
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: 'http://localhost:3000/auth/google/callback',
  scope: ['profile', 'email']
},
function(accessToken, refreshToken, profile, done) {
  // En producción, guardar/actualizar usuario en base de datos
  const user = {
    id: profile.id,
    displayName: profile.displayName,
    email: profile.emails[0].value,
    avatar: profile.photos[0].value,
    accessToken: accessToken,
    refreshToken: refreshToken
  };
  
  return done(null, user);
}));

// Serialización/deserialización de usuario para sesión
passport.serializeUser((user, done) => {
  done(null, user);
});

passport.deserializeUser((user, done) => {
  // En producción, recuperar usuario desde base de datos
  done(null, user);
});

// Middleware para verificar autenticación
function isAuthenticated(req, res, next) {
  if (req.isAuthenticated()) {
    return next();
  }
  res.redirect('/login');
}

// Rutas de autenticación
app.get('/auth/google', passport.authenticate('google'));

app.get('/auth/google/callback', 
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/dashboard');
  }
);

app.get('/logout', (req, res) => {
  req.logout((err) => {
    if (err) { return next(err); }
    res.redirect('/');
  });
});

// Ruta protegida
app.get('/dashboard', isAuthenticated, (req, res) => {
  res.json({ message: 'Dashboard protegido', user: req.user });
});

// Ruta de perfil que utiliza el access token para acceder a la API de Google
app.get('/profile', isAuthenticated, async (req, res) => {
  try {
    // Ejemplo: acceder a información adicional del usuario usando el token
    const response = await fetch('https://www.googleapis.com/oauth2/v3/userinfo', {
      headers: { Authorization: `Bearer ${req.user.accessToken}` }
    });
    
    if (!response.ok) {
      throw new Error('Error al obtener perfil');
    }
    
    const userData = await response.json();
    res.json(userData);
  } catch (error) {
    res.status(500).json({ message: 'Error al acceder al perfil', error: error.message });
  }
});
```

### Implementación de PKCE (Proof Key for Code Exchange)

El flujo PKCE mejora la seguridad en aplicaciones públicas como SPAs y aplicaciones móviles:

```javascript
// En el cliente (frontend)
const crypto = require('crypto');

// 1. Generar un code_verifier aleatorio
function generateCodeVerifier() {
  return base64URLEncode(crypto.randomBytes(32));
}

// 2. Crear code_challenge a partir del code_verifier
function generateCodeChallenge(codeVerifier) {
  return base64URLEncode(
    crypto.createHash('sha256').update(codeVerifier).digest()
  );
}

// Función auxiliar para codificación base64URL
function base64URLEncode(buffer) {
  return buffer.toString('base64')
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=/g, '');
}

// 3. Almacenar el code_verifier para usarlo más tarde
const codeVerifier = generateCodeVerifier();
const codeChallenge = generateCodeChallenge(codeVerifier);

// 4. Redirigir al usuario al endpoint de autorización con el code_challenge
const authorizationUrl = new URL('https://authorization-server.com/auth');
authorizationUrl.searchParams.append('client_id', 'tu_client_id');
authorizationUrl.searchParams.append('redirect_uri', 'https://tu-app.com/callback');
authorizationUrl.searchParams.append('response_type', 'code');
authorizationUrl.searchParams.append('code_challenge', codeChallenge);
authorizationUrl.searchParams.append('code_challenge_method', 'S256');
window.location = authorizationUrl.toString();

// 5. En el callback, intercambiar el código y el code_verifier por tokens
async function exchangeCodeForTokens(code) {
  const tokenResponse = await fetch('https://authorization-server.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      client_id: 'tu_client_id',
      grant_type: 'authorization_code',
      code: code,
      redirect_uri: 'https://tu-app.com/callback',
      code_verifier: codeVerifier
    })
  });
  
  return tokenResponse.json();
}
```

### Comparativa entre JWT y OAuth 2.0

| Característica | JWT | OAuth 2.0 |
|---------------|-----|----------|
| **Propósito** | Formato de token para transmitir información | Protocolo de autorización |
| **Uso típico** | Autenticación y autorización en APIs | Delegación de acceso a terceros |
| **Estándares relacionados** | JWS, JWE, JWA | OpenID Connect |
| **Complejidad** | Baja a Media | Media a Alta |
| **Estado** | Stateless | Generalmente stateful |
| **Revocación** | Difícil (requiere lista negra) | Fácil (revocar tokens en el servidor) |
| **Privacidad de datos** | Contenido visible (solo firmado) | Opaco (el cliente no ve el contenido) |
| **Escalabilidad** | Alta (no requiere almacenamiento en servidor) | Media (requiere persistencia de tokens) |

### Mejores Prácticas para OAuth 2.0

1. **Usar HTTPS** para todas las comunicaciones
2. **Implementar PKCE** incluso para aplicaciones confidenciales
3. **Validar todos los parámetros** recibidos en redirecciones
4. **Limitar scopes** al mínimo necesario
5. **Establecer tiempos de expiración cortos** para access tokens
6. **No usar Implicit Flow** en nuevos desarrollos
7. **Validar tokens** antes de aceptarlos
8. **Implementar validación de estado (state parameter)** para prevenir CSRF
9. **Almacenar client_secrets de forma segura**
10. **Realizar auditorías regulares** de los permisos otorgados

## 7.2.5 Autenticación con Proveedores Externos

Integrar proveedores externos de autenticación (como Google, Facebook, GitHub, etc.) en aplicaciones permite ofrecer una experiencia de registro e inicio de sesión conveniente para los usuarios. Esta sección muestra cómo implementar autenticación social en aplicaciones Node.js.

### Ventajas de la Autenticación con Proveedores Externos

1. **Experiencia de usuario mejorada**: Elimina la necesidad de crear y recordar nuevas credenciales
2. **Mayor seguridad**: Delega la responsabilidad de la autenticación a proveedores especializados
3. **Datos verificados**: Acceso a información de usuario previamente verificada (email, foto, etc.)
4. **Reducción de abandonos**: Simplifica el proceso de registro, aumentando las conversiones
5. **Acceso a APIs**: Posibilidad de interactuar con APIs del proveedor (con permiso del usuario)

### Implementación Multi-proveedor con Passport.js

Passport.js facilita la integración de múltiples proveedores de autenticación social:

```javascript
const express = require('express');
const session = require('express-session');
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const FacebookStrategy = require('passport-facebook').Strategy;
const GitHubStrategy = require('passport-github2').Strategy;

const app = express();
app.use(express.json());

// Configuración de sesiones
app.use(session({
  secret: process.env.SESSION_SECRET || 'secret_session_key',
  resave: false,
  saveUninitialized: false,
  cookie: { secure: process.env.NODE_ENV === 'production' }
}));

// Inicializar Passport
app.use(passport.initialize());
app.use(passport.session());

// Base de datos simulada (en producción usaríamos una BD real)
const users = [];

// Función para encontrar o crear usuario
async function findOrCreateUser(profile, provider) {
  let user = users.find(u => 
    u.providerId === profile.id && u.provider === provider
  );
  
  if (!user) {
    user = {
      id: users.length + 1,
      providerId: profile.id,
      provider: provider,
      displayName: profile.displayName,
      email: profile.emails?.[0]?.value,
      photo: profile.photos?.[0]?.value,
      // Información adicional dependiendo del proveedor
    };
    users.push(user);
  }
  
  return user;
}

// Serialización/Deserialización
passport.serializeUser((user, done) => {
  done(null, user.id);
});

passport.deserializeUser((id, done) => {
  const user = users.find(u => u.id === id);
  done(null, user || null);
});

// Configurar estrategia Google
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback',
  scope: ['profile', 'email']
}, async (accessToken, refreshToken, profile, done) => {
  try {
    const user = await findOrCreateUser(profile, 'google');
    // Guardar tokens para posible uso posterior con APIs de Google
    user.googleTokens = { accessToken, refreshToken };
    return done(null, user);
  } catch (err) {
    return done(err);
  }
}));

// Configurar estrategia Facebook
passport.use(new FacebookStrategy({
  clientID: process.env.FACEBOOK_APP_ID,
  clientSecret: process.env.FACEBOOK_APP_SECRET,
  callbackURL: '/auth/facebook/callback',
  profileFields: ['id', 'displayName', 'photos', 'email']
}, async (accessToken, refreshToken, profile, done) => {
  try {
    const user = await findOrCreateUser(profile, 'facebook');
    user.facebookTokens = { accessToken, refreshToken };
    return done(null, user);
  } catch (err) {
    return done(err);
  }
}));

// Configurar estrategia GitHub
passport.use(new GitHubStrategy({
  clientID: process.env.GITHUB_CLIENT_ID,
  clientSecret: process.env.GITHUB_CLIENT_SECRET,
  callbackURL: '/auth/github/callback',
  scope: ['user:email']
}, async (accessToken, refreshToken, profile, done) => {
  try {
    const user = await findOrCreateUser(profile, 'github');
    user.githubTokens = { accessToken, refreshToken };
    return done(null, user);
  } catch (err) {
    return done(err);
  }
}));

// Rutas de autenticación para Google
app.get('/auth/google', passport.authenticate('google'));
app.get('/auth/google/callback', 
  passport.authenticate('google', { 
    failureRedirect: '/login',
    successRedirect: '/profile'
  })
);

// Rutas de autenticación para Facebook
app.get('/auth/facebook', passport.authenticate('facebook'));
app.get('/auth/facebook/callback',
  passport.authenticate('facebook', {
    failureRedirect: '/login',
    successRedirect: '/profile'
  })
);

// Rutas de autenticación para GitHub
app.get('/auth/github', passport.authenticate('github'));
app.get('/auth/github/callback',
  passport.authenticate('github', {
    failureRedirect: '/login',
    successRedirect: '/profile'
  })
);

// Ruta para logout
app.get('/logout', (req, res) => {
  req.logout((err) => {
    if (err) { return next(err); }
    res.redirect('/');
  });
});

// Middleware de autenticación
function ensureAuthenticated(req, res, next) {
  if (req.isAuthenticated()) {
    return next();
  }
  res.status(401).json({ message: 'No autenticado' });
}

// Ruta protegida para perfil
app.get('/profile', ensureAuthenticated, (req, res) => {
  res.json({ user: req.user });
});

// Inicio de sesión unificado con selector de proveedor
app.get('/login', (req, res) => {
  res.send(`
    <h1>Iniciar sesión</h1>
    <a href="/auth/google">Iniciar sesión con Google</a><br>
    <a href="/auth/facebook">Iniciar sesión con Facebook</a><br>
    <a href="/auth/github">Iniciar sesión con GitHub</a>
  `);
});
```

### Vinculación de Múltiples Proveedores a una Cuenta

Permitir que los usuarios vinculen varias identidades sociales a su cuenta principal:

```javascript
// Middleware para verificar si el usuario ya está autenticado
function checkAccountLinking(req, res, next) {
  // Almacenar en sesión que estamos en modo vinculación si el usuario ya está autenticado
  if (req.isAuthenticated()) {
    req.session.accountLinking = true;
    req.session.currentUserId = req.user.id;
  }
  next();
}

// Ejemplo para Google (aplicar lógica similar para otros proveedores)
app.get('/auth/google/link', ensureAuthenticated, checkAccountLinking, 
  passport.authenticate('google')
);

app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    // Comprobar si estamos en modo vinculación
    if (req.session.accountLinking) {
      const currentUserId = req.session.currentUserId;
      const currentUser = users.find(u => u.id === currentUserId);
      
      // Vincular la nueva identidad al usuario actual
      if (currentUser) {
        currentUser.linkedAccounts = currentUser.linkedAccounts || [];
        currentUser.linkedAccounts.push({
          provider: 'google',
          providerId: req.user.providerId,
          email: req.user.email
        });
        
        // Limpiar variables de sesión
        delete req.session.accountLinking;
        delete req.session.currentUserId;
        
        // Establecer el usuario original en la sesión
        req.login(currentUser, (err) => {
          if (err) return next(err);
          return res.redirect('/profile?linked=true');
        });
      }
    } else {
      // Flujo normal de autenticación
      res.redirect('/profile');
    }
  }
);
```

### Estrategias Personalizadas para APIs Propias

Crear una estrategia personalizada para un sistema de autenticación propio:

```javascript
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcrypt');

// Base de datos simulada
const localUsers = [
  {
    id: 1,
    username: 'usuario',
    password: '$2b$10$X/4QLCaIQkEYAZLGiUcT4.WN0idWhYgKjJZ5gKk/mJhX/.RMB.JLG', // 'contraseña'
    email: 'usuario@ejemplo.com'
  }
];

// Configurar estrategia local
passport.use(new LocalStrategy(
  async (username, password, done) => {
    try {
      // Buscar usuario
      const user = localUsers.find(u => u.username === username);
      if (!user) {
        return done(null, false, { message: 'Usuario no encontrado' });
      }
      
      // Verificar contraseña
      const validPassword = await bcrypt.compare(password, user.password);
      if (!validPassword) {
        return done(null, false, { message: 'Contraseña incorrecta' });
      }
      
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));

// Rutas para autenticación local
app.post('/auth/local', passport.authenticate('local', {
  failureRedirect: '/login',
  successRedirect: '/profile'
}));
```

### Mejores Prácticas para Autenticación con Proveedores Externos

1. **Almacenar información mínima**: Guardar solo los datos necesarios del perfil del usuario
2. **Ofrecer alternativas**: Siempre proporcionar métodos alternativos de autenticación
3. **Gestionar desconexiones**: Manejar casos donde el proveedor no esté disponible
4. **Revisar permisos solicitados**: Pedir solo los permisos estrictamente necesarios
5. **Mantener tokens actualizados**: Implementar renovación de tokens para accesos de larga duración
6. **Configurar callbacks seguros**: Usar URLs de callback en dominios verificados
7. **Validar datos recibidos**: No confiar ciegamente en la información proporcionada
8. **Permitir desvincular cuentas**: Dar opción a los usuarios de desconectar proveedores
9. **Manejar conflictos de identidad**: Definir estrategias para cuando un email ya existe en el sistema
10. **Cumplir normativas**: Asegurar cumplimiento de normativas de privacidad como GDPR

## 7.2.6 Sistemas de Roles y Permisos

Una vez autenticado un usuario, es crucial controlar a qué recursos puede acceder y qué acciones puede realizar. Los sistemas de roles y permisos son fundamentales para implementar una autorización robusta en aplicaciones.

### Conceptos Fundamentales

- **Usuario**: Entidad que interactúa con el sistema.
- **Rol**: Conjunto de permisos agrupados bajo un nombre (Administrador, Editor, Lector...).
- **Permiso**: Autorización para realizar una acción específica sobre un recurso.
- **Recurso**: Elemento del sistema al que se controla el acceso (documentos, funciones, endpoints...).

### Modelos de Control de Acceso

#### 1. Control de Acceso Basado en Roles (RBAC)

Los permisos se asignan a roles y los roles se asignan a usuarios.

```
+------------+      +---------+      +-----------+
|            |      |         |      |           |
|  Usuarios  +----->+  Roles  +----->+ Permisos  |
|            |      |         |      |           |
+------------+      +---------+      +-----------+
```

#### 2. Control de Acceso Basado en Atributos (ABAC)

Las decisiones de acceso se basan en atributos (del usuario, recurso, acción, contexto).

```
+------------+
|            |
|  Atributos +---+
|  Usuario   |   |
+------------+   |     +----------------+      +----------+
                 |     |                |      |          |
+------------+   +---->+    Políticas   +----->+  Decisión |
|            |   |     |    de Acceso   |      |          |
|  Atributos +---+     |                |      +----------+
|  Recurso   |   |     +----------------+
+------------+   |
                 |
+------------+   |
|            |   |
|  Atributos +---+
|  Contexto  |
+------------+
```

### Implementación de RBAC en Express

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');

const app = express();
app.use(express.json());

// En producción, usar una base de datos real
const users = [
  { id: 1, username: 'admin', roles: ['admin'] },
  { id: 2, username: 'editor', roles: ['editor'] },
  { id: 3, username: 'user', roles: ['user'] }
];

// Definición de roles y permisos
const roles = {
  admin: ['read:any', 'create:any', 'update:any', 'delete:any'],
  editor: ['read:any', 'create:own', 'update:own'],
  user: ['read:own']
};

// Middleware de autenticación (similar al visto anteriormente)
function authenticateToken(req, res, next) {
  // ... código de verificación de token ...
}

// Middleware para verificar roles
function checkRole(requiredRoles) {
  return (req, res, next) => {
    const userRoles = req.user.roles;
    
    // Verificar si el usuario tiene al menos uno de los roles requeridos
    const hasRequiredRole = requiredRoles.some(role => userRoles.includes(role));
    
    if (!hasRequiredRole) {
      return res.status(403).json({ message: 'Acceso prohibido: rol insuficiente' });
    }
    
    next();
  };
}

// Middleware para verificar permisos específicos
function checkPermission(action, resource) {
  return (req, res, next) => {
    const userRoles = req.user.roles;
    let hasPermission = false;
    
    // Comprobar si alguno de los roles del usuario tiene el permiso requerido
    for (const role of userRoles) {
      const permissions = roles[role] || [];
      
      if (permissions.includes(`${action}:any`)) {
        // Permiso para cualquier recurso de este tipo
        hasPermission = true;
        break;
      } else if (permissions.includes(`${action}:own`)) {
        // Permiso solo para recursos propios
        // Verificar si el recurso pertenece al usuario
        if (isOwnResource(req.user.id, resource, req.params.id)) {
          hasPermission = true;
          break;
        }
      }
    }
    
    if (!hasPermission) {
      return res.status(403).json({ message: 'Acceso prohibido: permiso insuficiente' });
    }
    
    next();
  };
}

// Función auxiliar para verificar propiedad de un recurso
function isOwnResource(userId, resourceType, resourceId) {
  // En producción, consultaríamos la base de datos
  // Este es un ejemplo simplificado
  const resources = {
    articles: [
      { id: 1, authorId: 2 },
      { id: 2, authorId: 3 }
    ]
  };
  
  const resource = resources[resourceType]?.find(r => r.id === parseInt(resourceId));
  return resource && resource.authorId === userId;
}

// Rutas protegidas con diferentes niveles de autorización

// Ruta que requiere rol específico
app.get('/admin/dashboard', 
  authenticateToken, 
  checkRole(['admin']), 
  (req, res) => {
    res.json({ message: 'Panel de administración', user: req.user });
  }
);

// Ruta que requiere permisos específicos
app.get('/articles/:id', 
  authenticateToken,
  (req, res, next) => {
    // Para lectura de artículos, verificamos permisos dentro del handler
    const userRoles = req.user.roles;
    const articleId = req.params.id;
    let canAccess = false;
    
    // Comprobar permisos por cada rol
    for (const role of userRoles) {
      const permissions = roles[role] || [];
      
      if (permissions.includes('read:any')) {
        // Puede leer cualquier artículo
        canAccess = true;
        break;
      } else if (permissions.includes('read:own')) {
        // Solo puede leer sus propios artículos
        if (isOwnResource(req.user.id, 'articles', articleId)) {
          canAccess = true;
          break;
        }
      }
    }
    
    if (!canAccess) {
      return res.status(403).json({ message: 'No tienes permiso para ver este artículo' });
    }
    
    // Simulación de recuperación de artículo
    const article = { id: parseInt(articleId), title: 'Artículo de prueba' };
    res.json(article);
  }
);

app.put('/articles/:id', 
  authenticateToken, 
  checkPermission('update', 'articles'), 
  (req, res) => {
    res.json({ message: 'Artículo actualizado' });
  }
);

app.delete('/articles/:id', 
  authenticateToken, 
  checkPermission('delete', 'articles'), 
  (req, res) => {
    res.json({ message: 'Artículo eliminado' });
  }
);
```

### Implementación de una capa ACL (Lista de Control de Acceso)

Para aplicaciones más complejas, podemos usar bibliotecas como `accesscontrol`:

```javascript
const { AccessControl } = require('accesscontrol');

// Definir granularidad de permisos
const ac = new AccessControl();

ac.grant('user')
  .readOwn('profile')
  .updateOwn('profile')
  .readOwn('article')

ac.grant('editor')
  .extend('user')
  .createOwn('article')
  .updateOwn('article')
  .readAny('article')

ac.grant('admin')
  .extend('editor')
  .createAny('article')
  .updateAny('article')
  .deleteAny('article')
  .readAny('profile')

// Middleware para verificar permisos usando accesscontrol
function checkPermission(action, resource) {
  return (req, res, next) => {
    const role = req.user.role;
    const permission = ac.can(role)[action](resource);
    
    if (permission.granted) {
      // Opcionalmente filtrar campos permitidos
      if (action === 'read') {
        res.locals.permittedFields = permission.attributes;
      }
      next();
    } else {
      res.status(403).json({ message: 'Acceso prohibido' });
    }
  };
}

app.get('/articles/:id', 
  authenticateToken,
  (req, res, next) => {
    const isOwn = isOwnResource(req.user.id, 'articles', req.params.id);
    const action = isOwn ? 'readOwn' : 'readAny';
    const permission = ac.can(req.user.role)[action]('article');
    
    if (permission.granted) {
      // Simular recuperación de artículo
      const article = { id: parseInt(req.params.id), title: 'Artículo de prueba', content: 'Contenido...', secretData: 'Datos confidenciales' };
      
      // Filtrar campos según los permisos
      const filteredArticle = permission.filter(article);
      res.json(filteredArticle);
    } else {
      res.status(403).json({ message: 'Acceso prohibido' });
    }
  }
);
```

### Autorización a Nivel de Base de Datos

Para aplicaciones con MongoDB:

```javascript
const mongoose = require('mongoose');

// Esquema con campo de propietario
const articleSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  }
});

// Middleware pre-find para aplicar restricciones por rol
articleSchema.pre('find', function() {
  const user = this.getOptions().user;
  if (!user) return;
  
  if (user.roles.includes('admin')) {
    // Los administradores pueden ver todo, no aplicar filtros
    return;
  } else if (user.roles.includes('editor')) {
    // Los editores pueden ver todos los artículos, pero no hay datos privados
    this.select('-privateData');
  } else {
    // Los usuarios normales solo pueden ver sus propios artículos
    this.where({ author: user._id }).select('-privateData');
  }
});

const Article = mongoose.model('Article', articleSchema);

app.get('/articles', authenticateToken, async (req, res) => {
  try {
    // Pasar usuario al contexto de la consulta
    const articles = await Article.find().setOptions({ user: req.user });
    res.json(articles);
  } catch (error) {
    res.status(500).json({ message: 'Error al recuperar artículos', error: error.message });
  }
});
```

### Mejores Prácticas para Sistemas de Roles y Permisos

1. **Principio de mínimo privilegio**: Otorgar solo los permisos estrictamente necesarios
2. **Separación de responsabilidades**: Dividir tareas críticas entre diferentes roles
3. **Herencia de roles**: Implementar jerarquías de roles para simplificar la gestión
4. **Auditoría de acciones**: Registrar todas las operaciones sensibles con el usuario que las realizó
5. **Revisar permisos periódicamente**: Actualizar roles y permisos según evolucione la aplicación
6. **Verificar permisos en todas las capas**: No confiar solo en la interfaz de usuario
7. **Permisos dinámicos**: Permitir cambios en tiempo de ejecución según el contexto
8. **Evitar permisos duplicados**: Consolidar permisos similares para reducir complejidad
9. **Documentar el sistema de permisos**: Mantener clara la estructura para el equipo de desarrollo

## 7.2.7 Mejores Prácticas de Seguridad

La implementación de sistemas de autenticación y autorización requiere un enfoque cuidadoso para garantizar la seguridad. A continuación, se presentan las mejores prácticas generales que deben aplicarse a cualquier sistema.

### Almacenamiento Seguro de Credenciales

1. **Nunca almacenar contraseñas en texto plano**
   - Utilizar algoritmos de hash diseñados específicamente para contraseñas (bcrypt, Argon2, scrypt)
   - Aplicar "salting" para prevenir ataques con tablas rainbow

```javascript
const bcrypt = require('bcrypt');

async function hashPassword(plainPassword) {
  // El salt se genera automáticamente y se almacena como parte del hash
  // El 10 representa el coste computacional (factor de trabajo)
  const saltRounds = 10;
  return await bcrypt.hash(plainPassword, saltRounds);
}

async function verifyPassword(plainPassword, hashedPassword) {
  return await bcrypt.compare(plainPassword, hashedPassword);
}

// Ejemplo de uso
async function registerUser(username, password) {
  try {
    const hashedPassword = await hashPassword(password);
    // Guardar usuario con contraseña hasheada
    // db.users.insert({ username, password: hashedPassword });
    return true;
  } catch (error) {
    console.error('Error al registrar usuario:', error);
    return false;
  }
}
```

2. **Protección contra fuerza bruta**
   - Limitar intentos de inicio de sesión
   - Implementar captchas
   - Usar retrasos incrementales (backoff exponencial)

```javascript
const loginAttempts = {}; // En producción, usar Redis o similar

function limitLoginAttempts(username, req, res, next) {
  // Inicializar o recuperar intentos
  loginAttempts[username] = loginAttempts[username] || {
    count: 0,
    lastAttempt: 0,
    blocked: false
  };
  
  const attempt = loginAttempts[username];
  const now = Date.now();
  
  // Verificar si está bloqueado
  if (attempt.blocked) {
    // Desbloquear después de 30 minutos
    if (now - attempt.lastAttempt > 30 * 60 * 1000) {
      attempt.blocked = false;
      attempt.count = 0;
    } else {
      return res.status(429).json({ 
        message: 'Demasiados intentos fallidos. Cuenta bloqueada temporalmente.' 
      });
    }
  }
  
  // Si han pasado más de 15 minutos desde el último intento, reiniciar contador
  if (attempt.count > 0 && now - attempt.lastAttempt > 15 * 60 * 1000) {
    attempt.count = 0;
  }
  
  // Aplicar retraso exponencial
  if (attempt.count > 0) {
    const delaySeconds = Math.pow(2, attempt.count - 1);
    const elapsedSeconds = (now - attempt.lastAttempt) / 1000;
    
    if (elapsedSeconds < delaySeconds) {
      return res.status(429).json({
        message: `Intente nuevamente en ${Math.ceil(delaySeconds - elapsedSeconds)} segundos`
      });
    }
  }
  
  // Actualizar último intento
  attempt.lastAttempt = now;
  
  // Almacenar en req para usar en el handler de login
  req.loginAttempt = attempt;
  next();
}
```

### Seguridad en Comunicaciones

1. **Usar HTTPS en todas las comunicaciones**
   - Configurar TLS/SSL en el servidor
   - Redirigir automáticamente de HTTP a HTTPS
   - Implementar HTTP Strict Transport Security (HSTS)

```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// Aplicar políticas de seguridad con helmet
app.use(helmet());

// Configurar HSTS
app.use(helmet.hsts({
  maxAge: 15552000, // 180 días en segundos
  includeSubDomains: true,
  preload: true
}));

// Middleware para redireccionar HTTP a HTTPS
function requireHTTPS(req, res, next) {
  if (!req.secure && req.get('x-forwarded-proto') !== 'https' && process.env.NODE_ENV !== 'development') {
    return res.redirect('https://' + req.get('host') + req.url);
  }
  next();
}

app.use(requireHTTPS);
```

2. **Protección contra vulnerabilidades comunes**
   - Cross-Site Scripting (XSS)
   - Cross-Site Request Forgery (CSRF)
   - Inyección SQL
   - Server-Side Request Forgery (SSRF)

```javascript
const csrf = require('csurf');

// Proteger contra CSRF
const csrfProtection = csrf({ cookie: true });

// Aplicar en rutas que manejan formularios/envío de datos
app.post('/api/change-password', csrfProtection, (req, res) => {
  // Verificación ya realizada por el middleware csrf
  // Lógica de cambio de contraseña...
});

// Generar token para el cliente
app.get('/form', csrfProtection, (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});
```

### Manejo de Sesiones y Tokens

1. **Configuración segura de cookies**
   - Flags: HttpOnly, Secure, SameSite
   - Tiempo de expiración adecuado

```javascript
app.use(session({
  secret: process.env.SESSION_SECRET,
  name: '_session_id', // Evitar nombres default como 'connect.sid'
  cookie: {
    httpOnly: true,     // No accesible via JavaScript
    secure: process.env.NODE_ENV === 'production', // Solo HTTPS en producción
    sameSite: 'lax',   // Protección contra CSRF
    maxAge: 3600000    // 1 hora en milisegundos
  },
  resave: false,
  saveUninitialized: false,
  // En producción usar almacenamiento persistente
  store: productionStore() // Redis, MongoDB, etc.
}));
```

2. **Revocación de tokens y sesiones**
   - Implementar mecanismo de logout efectivo
   - Mantener lista negra de tokens revocados

```javascript
const revokedTokens = new Set(); // En producción usar Redis o similar

// Middleware para verificar si un token ha sido revocado
function checkIfTokenRevoked(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (token && revokedTokens.has(token)) {
    return res.status(401).json({ message: 'Token revocado o inválido' });
  }
  
  next();
}

// Aplicar antes del middleware de autenticación
app.use(checkIfTokenRevoked);

// Ruta para logout (revocación de token)
app.post('/api/logout', authenticateToken, (req, res) => {
  const token = req.headers.authorization.split(' ')[1];
  
  // Agregar a lista de tokens revocados
  revokedTokens.add(token);
  
  // Programar eliminación del token (limpieza)
  setTimeout(() => {
    revokedTokens.delete(token);
  }, 24 * 60 * 60 * 1000); // 24 horas (o usar TTL en Redis)
  
  res.json({ message: 'Logout exitoso' });
});
```

### Gestión de Secretos y Configuraciones

1. **Manejo seguro de secretos**
   - No hardcodear secretos en el código
   - Usar variables de entorno o sistemas de manejo de secretos

```javascript
// Archivo .env (nunca subir a repositorios)
// JWT_SECRET=tu_secreto_muy_seguro_y_aleatorio
// DB_PASSWORD=tu_password_de_base_de_datos

// Archivo de configuración (config.js)
require('dotenv').config();

module.exports = {
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: '2h'
  },
  db: {
    url: `mongodb://${process.env.DB_USER}:${process.env.DB_PASSWORD}@${process.env.DB_HOST}/${process.env.DB_NAME}`
  },
  // Validar que existan las variables necesarias
  validateEnv: () => {
    const required = ['JWT_SECRET', 'DB_PASSWORD', 'DB_HOST'];
    for (const variable of required) {
      if (!process.env[variable]) {
        throw new Error(`Variable de entorno ${variable} no configurada`);
      }
    }
    return true;
  }
};

// Validar al inicio de la aplicación
const config = require('./config');
try {
  config.validateEnv();
} catch (error) {
  console.error('Error en configuración:', error.message);
  process.exit(1);
}
```

### Auditoria y Monitoreo

1. **Registro de eventos de seguridad**
   - Loguear todos los eventos relacionados con autenticación y autorización
   - Almacenar logs de forma segura

```javascript
const winston = require('winston');

// Configurar logger
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'auth-service' },
  transports: [
    new winston.transports.File({ filename: 'logs/security-events.log' })
  ]
});

// Middleware para registro de eventos de autenticación
function logAuthEvent(eventType, userId, success, details = {}) {
  logger.info({
    eventType,
    userId,
    success,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    ...details,
    timestamp: new Date().toISOString()
  });
}

app.post('/api/login', (req, res) => {
  // Lógica de autenticación...
  
  if (authSuccess) {
    logAuthEvent('login', user.id, true);
    // Continuar con el flujo normal...
  } else {
    logAuthEvent('login', req.body.username, false, { reason: 'invalid_credentials' });
    // Manejar error...
  }
});
```

### Conclusión

La seguridad en sistemas de autenticación y autorización debe ser una prioridad absoluta en cualquier aplicación. Siguiendo estas mejores prácticas, podemos:  

- Proteger la información de los usuarios
- Prevenir accesos no autorizados
- Mantener la integridad del sistema
- Cumplir con regulaciones y estándares de seguridad

Recuerda que la seguridad es un proceso continuo, no un producto final. Debes mantenerte actualizado sobre nuevas vulnerabilidades y técnicas de ataque, realizar auditorías de seguridad periódicas y actualizar tus prácticas según evolucionen los estándares de la industria.
