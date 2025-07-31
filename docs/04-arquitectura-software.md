# Capítulo 4: Arquitectura de Software

La arquitectura de software es la estructura fundamental de un sistema, compuesta por componentes, sus propiedades externas y las relaciones entre ellos. Una buena arquitectura proporciona un marco sólido para el desarrollo, mantenimiento y evolución de los sistemas de software.

## 4.1 Principios Arquitectónicos Fundamentales

Los principios arquitectónicos son guías que ayudan a los desarrolladores a crear sistemas robustos, mantenibles y escalables.

### 4.1.1 Separación de Responsabilidades

Cada componente del sistema debe tener una responsabilidad única y bien definida. Esto facilita la comprensión, el mantenimiento y la evolución del software.

### 4.1.2 Alta Cohesión y Bajo Acoplamiento

- **Alta Cohesión**: Los componentes deben tener elementos fuertemente relacionados entre sí.
- **Bajo Acoplamiento**: Los componentes deben ser lo más independientes posible entre sí.

### 4.1.3 Abstracción

Ocultar los detalles de implementación detrás de interfaces bien definidas, permitiendo que los componentes se comuniquen sin conocer los detalles internos de cada uno.

### 4.1.4 Modularidad

Organizar el sistema en módulos o componentes que puedan ser desarrollados, probados, desplegados y mantenidos de forma independiente.

## 4.2 Patrones Arquitectónicos Comunes

### 4.2.1 Arquitectura en Capas

Organiza el sistema en capas horizontales, donde cada capa tiene un rol específico y solo puede comunicarse con las capas adyacentes.

```
+------------------+
|  Presentación    |
+------------------+
|  Lógica Negocio  |
+------------------+
|  Acceso a Datos  |
+------------------+
|  Base de Datos   |
+------------------+
```

**Ventajas**:
- Clara separación de responsabilidades
- Facilita la comprensión y el mantenimiento
- Permite la sustitución de capas enteras

**Desventajas**:
- Puede introducir overhead de rendimiento
- Cambios que atraviesan capas pueden ser complejos

### 4.2.2 Modelo-Vista-Controlador (MVC)

Divide la aplicación en tres componentes principales:

- **Modelo**: Representa los datos y la lógica de negocio.
- **Vista**: Presenta la información al usuario.
- **Controlador**: Gestiona las entradas del usuario y actualiza el modelo y la vista.

```
+-------------+        +-------------+
|    Vista    |<-------|Controlador  |
+-------------+        +-------------+
      ^                      |
      |                      v
      |                +-------------+
      +----------------|   Modelo    |
                       +-------------+
```

**Ventajas**:
- Separación clara entre la presentación y la lógica
- Facilita las pruebas unitarias
- Permite múltiples vistas para los mismos datos

**Ejemplo en React**:

```jsx
// Model
class UserModel {
  constructor() {
    this.users = [];
    this.onUserListChanged = null;
  }

  addUser(user) {
    this.users.push(user);
    this.onUserListChanged(this.users);
  }

  getUsers() {
    return this.users;
  }
}

// Controller
class UserController {
  constructor(model, view) {
    this.model = model;
    this.view = view;

    // Enlaza la vista con el controlador
    this.view.onAddUser = this.addUser.bind(this);
    
    // Enlaza el modelo con la vista
    this.model.onUserListChanged = this.view.updateUserList.bind(this.view);
  }

  addUser(name, email) {
    if (name && email) {
      this.model.addUser({ name, email });
    }
  }
}

// View (componente React)
function UserView({ onAddUser, users }) {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onAddUser(name, email);
    setName('');
    setEmail('');
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input 
          value={name} 
          onChange={(e) => setName(e.target.value)} 
          placeholder="Name" 
        />
        <input 
          value={email} 
          onChange={(e) => setEmail(e.target.value)} 
          placeholder="Email" 
        />
        <button type="submit">Add User</button>
      </form>
      
      <ul>
        {users.map((user, index) => (
          <li key={index}>
            {user.name} ({user.email})
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 4.2.3 Arquitectura Hexagonal (Puertos y Adaptadores)

También conocida como "Ports and Adapters", esta arquitectura aísla el núcleo de la aplicación (lógica de negocio) de las dependencias externas como UI, bases de datos o servicios.

## 4.3 Arquitectura Hexagonal en Detalle

La arquitectura hexagonal (o de puertos y adaptadores) busca crear aplicaciones que:

1. Sean independientes de frameworks y herramientas externas
2. Sean fácilmente testables sin UI, base de datos o servicios externos
3. Sean independientes del mecanismo de entrega (web, CLI, API, etc.)

### 4.3.1 Componentes Principales

- **Núcleo de la aplicación**: Contiene la lógica de negocio y las reglas de dominio.
- **Puertos**: Interfaces que definen cómo el núcleo interactúa con el exterior.
  - **Puertos de entrada (primarios)**: Permiten que el exterior use el núcleo.
  - **Puertos de salida (secundarios)**: Permiten que el núcleo use servicios externos.
- **Adaptadores**: Implementaciones concretas de los puertos.
  - **Adaptadores primarios**: Transforman las solicitudes externas en llamadas al núcleo.
  - **Adaptadores secundarios**: Conectan el núcleo con servicios externos.

### 4.3.2 Beneficios

- **Testabilidad**: El núcleo puede probarse independientemente de las dependencias externas.
- **Flexibilidad**: Las implementaciones externas (DB, UI, etc.) pueden cambiarse sin afectar el núcleo.
- **Mantenibilidad**: La separación clara de responsabilidades facilita el mantenimiento.

## 4.4 Diagramas de la Arquitectura Hexagonal

```
graph TD
    subgraph Exterior
        A[Interfaz de Usuario / API REST] -- Usa --> B(Adaptador Primario);
        C[Base de Datos] -- Usa --> D(Adaptador Secundario);
        E[Servicio Externo] -- Usa --> F(Adaptador Secundario);
    end

    subgraph Núcleo de la Aplicación
        G[Puerto de Entrada] -- Implementado por --> B;
        H[Puerto de Salida] -- Implementado por --> D;
        I[Puerto de Salida] -- Implementado por --> F;
        J[Lógica de Negocio / Casos de Uso] -- Llama a --> H;
        J -- Llama a --> I;
        G -- Llama a --> J;
    end

    B -- Llama a --> G;
    D -- Implementa --> H;
    F -- Implementa --> I;
```

## 4.5 Ejemplos de Código (Node.js/TypeScript)

### Entidad de Dominio

```typescript
// src/domain/entities/User.ts
export interface User {
  id: string;
  name: string;
  email: string;
  passwordHash: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### Puertos (Interfaces)

```typescript
// src/domain/ports/UserRepository.ts
import { User } from '../entities/User';

export interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<boolean>;
}

// src/domain/ports/PasswordHasher.ts
export interface PasswordHasher {
  hash(password: string): Promise<string>;
  verify(password: string, hash: string): Promise<boolean>;
}
```

### Servicios de Aplicación (Casos de Uso)

```typescript
// src/application/use-cases/UserService.ts
import { User } from '../../domain/entities/User';
import { UserRepository } from '../../domain/ports/UserRepository';
import { PasswordHasher } from '../../domain/ports/PasswordHasher';

export class UserService {
  constructor(
    private userRepository: UserRepository,
    private passwordHasher: PasswordHasher
  ) {}

  async registerUser(name: string, email: string, password: string): Promise<User> {
    // Verificar si el usuario ya existe
    const existingUser = await this.userRepository.findByEmail(email);
    if (existingUser) {
      throw new Error('User with this email already exists');
    }

    // Hash de la contraseña
    const passwordHash = await this.passwordHasher.hash(password);

    // Crear y guardar el usuario
    const user: User = {
      id: this.generateId(),
      name,
      email,
      passwordHash,
      createdAt: new Date(),
      updatedAt: new Date()
    };

    return this.userRepository.save(user);
  }

  private generateId(): string {
    return Math.random().toString(36).substring(2) + Date.now().toString(36);
  }
}
```

### Adaptadores Secundarios (Implementaciones de los Puertos)

```typescript
// src/infrastructure/repositories/MongoUserRepository.ts
import { User } from '../../domain/entities/User';
import { UserRepository } from '../../domain/ports/UserRepository';
import { MongoClient, Collection } from 'mongodb';

export class MongoUserRepository implements UserRepository {
  private collection: Collection;

  constructor(private client: MongoClient) {
    this.collection = client.db('app').collection('users');
  }

  async findById(id: string): Promise<User | null> {
    const user = await this.collection.findOne({ id });
    return user as User | null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.collection.findOne({ email });
    return user as User | null;
  }

  async save(user: User): Promise<User> {
    await this.collection.updateOne(
      { id: user.id },
      { $set: { ...user, updatedAt: new Date() } },
      { upsert: true }
    );
    return user;
  }

  async delete(id: string): Promise<boolean> {
    const result = await this.collection.deleteOne({ id });
    return result.deletedCount === 1;
  }
}

// src/infrastructure/security/BcryptPasswordHasher.ts
import { PasswordHasher } from '../../domain/ports/PasswordHasher';
import bcrypt from 'bcrypt';

export class BcryptPasswordHasher implements PasswordHasher {
  constructor(private saltRounds: number = 10) {}

  async hash(password: string): Promise<string> {
    return bcrypt.hash(password, this.saltRounds);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}
```

### Adaptadores Primarios (Controllers)

```typescript
// src/infrastructure/web/controllers/UserController.ts
import { Request, Response } from 'express';
import { UserService } from '../../../application/use-cases/UserService';

export class UserController {
  constructor(private userService: UserService) {}

  async registerUser(req: Request, res: Response): Promise<Response> {
    try {
      const { name, email, password } = req.body;
      
      if (!name || !email || !password) {
        return res.status(400).json({ message: 'Missing required fields' });
      }
      
      const user = await this.userService.registerUser(name, email, password);
      
      // No devolver el hash de la contraseña ni información sensible
      const { passwordHash, ...safeUser } = user;
      
      return res.status(201).json(safeUser);
    } catch (error: any) {
      return res.status(500).json({ message: error.message });
    }
  }
}
```

### Orquestación de la Aplicación

```typescript
// src/index.ts
import express from 'express';
import { MongoClient } from 'mongodb';
import { UserController } from './infrastructure/web/controllers/UserController';
import { UserService } from './application/use-cases/UserService';
import { MongoUserRepository } from './infrastructure/repositories/MongoUserRepository';
import { BcryptPasswordHasher } from './infrastructure/security/BcryptPasswordHasher';

async function bootstrap() {
  const app = express();
  app.use(express.json());
  
  // Configurar MongoDB
  const client = new MongoClient('mongodb://localhost:27017');
  await client.connect();
  
  // Crear instancias de adaptadores
  const userRepository = new MongoUserRepository(client);
  const passwordHasher = new BcryptPasswordHasher();
  
  // Crear servicio (caso de uso)
  const userService = new UserService(userRepository, passwordHasher);
  
  // Crear controller
  const userController = new UserController(userService);
  
  // Definir rutas
  app.post('/api/users', (req, res) => userController.registerUser(req, res));
  
  // Iniciar servidor
  const PORT = process.env.PORT || 3000;
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}

bootstrap().catch(console.error);
```

## 4.6 Microservicios vs. Arquitectura Monolítica

### 4.6.1 Arquitectura Monolítica

Una arquitectura monolítica es aquella donde todos los componentes de la aplicación están integrados en una única unidad de despliegue.

**Ventajas**:
- Desarrollo inicial más rápido y sencillo
- Depuración más fácil
- Despliegue simplificado
- Mejor rendimiento en aplicaciones pequeñas

**Desventajas**:
- Escalabilidad limitada
- Dificultad para mantener y evolucionar a largo plazo
- Acoplamiento entre componentes
- Adopción tecnológica restringida

### 4.6.2 Microservicios

Los microservicios dividen la aplicación en servicios pequeños e independientes, cada uno con su propia base de código, ciclo de desarrollo y despliegue.

**Ventajas**:
- Escalabilidad selectiva de servicios
- Adopción flexible de tecnologías
- Equipos autónomos y paralelos
- Fallos aislados y mayor resiliencia

**Desventajas**:
- Mayor complejidad operativa
- Desafíos de comunicación entre servicios
- Pruebas end-to-end más complejas
- Overhead de red
