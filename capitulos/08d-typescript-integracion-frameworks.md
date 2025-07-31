# 8.4 Integración de TypeScript con Frameworks

## Introducción

El verdadero poder de TypeScript se manifiesta cuando se integra con los frameworks y bibliotecas más populares del ecosistema JavaScript. En este capítulo, exploraremos cómo TypeScript mejora el desarrollo de aplicaciones en diversos frameworks frontend y backend, proporcionando seguridad de tipos, mejor documentación y una experiencia de desarrollo superior.

A medida que las aplicaciones web modernas se vuelven más complejas, la necesidad de un código robusto y mantenible se vuelve crítica. TypeScript aborda esta necesidad al añadir verificación de tipos estáticos a los frameworks que ya conocemos y amamos, desde React y Vue en el frontend hasta Express y NestJS en el backend.

En este capítulo, veremos ejemplos prácticos de cómo TypeScript se integra con estos frameworks, las mejores prácticas para cada uno, y cómo aprovechar al máximo las características de tipado para crear aplicaciones más seguras y mantenibles. También exploraremos las herramientas y configuraciones específicas que facilitan esta integración.

## 8.4.1 TypeScript con React

React es una de las bibliotecas más populares para construir interfaces de usuario, y TypeScript añade una capa de seguridad de tipos que mejora significativamente la experiencia de desarrollo.

### Configuración de un proyecto React con TypeScript

```bash
# Usando Create React App
npx create-react-app my-app --template typescript

# O usando Vite
npm create vite@latest my-app -- --template react-ts
```

### Componentes funcionales tipados

```tsx
// Definición de props con interface
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  children?: React.ReactNode;
}

// Componente funcional tipado
const Button: React.FC<ButtonProps> = ({ 
  text, 
  onClick, 
  variant = 'primary', 
  disabled = false,
  children 
}) => {
  return (
    <button 
      className={`button ${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {text}
      {children}
    </button>
  );
};

// Uso
const App: React.FC = () => {
  return (
    <Button 
      text="Click Me" 
      onClick={() => console.log("Button clicked!")}
      variant="primary"
    >
      <span>🚀</span>
    </Button>
  );
};
```

### Componentes de clase con TypeScript

```tsx
// Componente de clase tipado
interface CounterState {
  count: number;
}

interface CounterProps {
  initialCount: number;
  step?: number;
}

class Counter extends React.Component<CounterProps, CounterState> {
  // Valores por defecto para props
  static defaultProps: Partial<CounterProps> = {
    step: 1
  };
  
  constructor(props: CounterProps) {
    super(props);
    this.state = {
      count: props.initialCount
    };
  }
  
  increment = () => {
    this.setState(prevState => ({
      count: prevState.count + (this.props.step || 1)
    }));
  };
  
  decrement = () => {
    this.setState(prevState => ({
      count: prevState.count - (this.props.step || 1)
    }));
  };
  
  render() {
    return (
      <div>
        <h2>Count: {this.state.count}</h2>
        <button onClick={this.increment}>+</button>
        <button onClick={this.decrement}>-</button>
      </div>
    );
  }
}

// Uso
<Counter initialCount={10} step={2} />
```

### Hooks tipados

```tsx
// useState con tipos
const [count, setCount] = useState<number>(0);

// useReducer con tipos
type CounterAction = 
  | { type: 'INCREMENT'; payload: number }
  | { type: 'DECREMENT'; payload: number }
  | { type: 'RESET' };

interface CounterState {
  count: number;
}

const counterReducer = (state: CounterState, action: CounterAction): CounterState => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + action.payload };
    case 'DECREMENT':
      return { count: state.count - action.payload };
    case 'RESET':
      return { count: 0 };
    default:
      return state;
  }
};

const CounterWithReducer: React.FC = () => {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({ type: 'INCREMENT', payload: 1 })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT', payload: 1 })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
};

// useRef con tipos
const inputRef = useRef<HTMLInputElement>(null);

useEffect(() => {
  if (inputRef.current) {
    inputRef.current.focus();
  }
}, []);

// useCallback con tipado
const handleSubmit = useCallback((event: React.FormEvent) => {
  event.preventDefault();
  // lógica
}, []);

// Custom hook tipado
interface UseToggleOutput {
  value: boolean;
  toggle: () => void;
  setTrue: () => void;
  setFalse: () => void;
}

function useToggle(initialValue = false): UseToggleOutput {
  const [value, setValue] = useState<boolean>(initialValue);
  
  const toggle = useCallback(() => {
    setValue(v => !v);
  }, []);
  
  const setTrue = useCallback(() => {
    setValue(true);
  }, []);
  
  const setFalse = useCallback(() => {
    setValue(false);
  }, []);
  
  return { value, toggle, setTrue, setFalse };
}

// Uso del custom hook
const { value: isModalOpen, toggle: toggleModal, setTrue: openModal, setFalse: closeModal } = useToggle();
```

### Context API con TypeScript

```tsx
// Definir el tipo para el contexto
interface ThemeContextType {
  isDarkMode: boolean;
  toggleTheme: () => void;
}

// Crear contexto con un valor por defecto
const ThemeContext = React.createContext<ThemeContextType | undefined>(undefined);

// Hook personalizado para usar el contexto
function useTheme(): ThemeContextType {
  const context = React.useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// Proveedor de contexto
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [isDarkMode, setIsDarkMode] = useState(false);
  
  const toggleTheme = useCallback(() => {
    setIsDarkMode(prev => !prev);
  }, []);
  
  // Memoizar el valor del contexto para evitar renderizaciones innecesarias
  const value = React.useMemo(() => {
    return { isDarkMode, toggleTheme };
  }, [isDarkMode, toggleTheme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

// Componente que consume el contexto
const ThemedButton: React.FC<{ label: string; onClick: () => void }> = ({ label, onClick }) => {
  const { isDarkMode } = useTheme();
  
  return (
    <button 
      className={isDarkMode ? 'dark-button' : 'light-button'}
      onClick={onClick}
    >
      {label}
    </button>
  );
};
```

### Tipando eventos

```tsx
// Eventos de formulario
const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
  event.preventDefault();
  // Lógica para manejar el envío
};

// Eventos de input
const handleInputChange = (event: React.ChangeEvent<HTMLInputElement>) => {
  const { name, value } = event.target;
  // Actualizar estado
};

// Eventos de teclado
const handleKeyDown = (event: React.KeyboardEvent<HTMLInputElement>) => {
  if (event.key === 'Enter') {
    // Lógica para manejo de tecla Enter
  }
};

// Eventos de ratón
const handleMouseOver = (event: React.MouseEvent<HTMLDivElement>) => {
  // Lógica para hover
};

// Eventos genéricos
const handleEvent = <T extends Element>(event: React.SyntheticEvent<T>) => {
  // Manejo genérico de eventos
};
```

## 8.4.2 TypeScript con Express

Express es un framework minimalista para Node.js que puede beneficiarse enormemente de la seguridad de tipos que ofrece TypeScript.

### Configuración de un proyecto Express con TypeScript

```bash
# Inicializar proyecto
npm init -y
npm install express
npm install --save-dev typescript @types/node @types/express ts-node nodemon

# Crear tsconfig.json
npx tsc --init
```

```json
// tsconfig.json configuración básica para Express
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"]
}
```

### Aplicación básica de Express tipada

```typescript
// src/app.ts
import express, { Request, Response, NextFunction } from 'express';

// Definir un tipo para los datos de usuario
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

// Arreglo tipado de usuarios
let users: User[] = [
  { id: 1, name: 'John Doe', email: 'john@example.com', role: 'admin' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com', role: 'user' },
];

// Middleware tipado
const loggerMiddleware = (req: Request, res: Response, next: NextFunction) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.path}`);
  next();
};

app.use(loggerMiddleware);

// GET endpoint para obtener todos los usuarios
app.get('/users', (req: Request, res: Response) => {
  res.json(users);
});

// GET endpoint para obtener un usuario por ID
app.get('/users/:id', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  const user = users.find(u => u.id === id);
  
  if (!user) {
    return res.status(404).json({ message: 'User not found' });
  }
  
  res.json(user);
});

// POST endpoint para crear un usuario
app.post('/users', (req: Request, res: Response) => {
  const { name, email, role } = req.body;
  
  // Validación básica
  if (!name || !email || !role || !['admin', 'user'].includes(role)) {
    return res.status(400).json({ message: 'Invalid user data' });
  }
  
  const newUser: User = {
    id: users.length > 0 ? Math.max(...users.map(u => u.id)) + 1 : 1,
    name,
    email,
    role: role as 'admin' | 'user'
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Extendiendo el tipo Request

A menudo necesitamos añadir propiedades personalizadas al objeto Request, como información de usuario después de la autenticación:

```typescript
// src/types/express/index.d.ts
import { User } from '../../models/user';

declare global {
  namespace Express {
    interface Request {
      user?: User;
      token?: string;
    }
  }
}

// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { User } from '../models/user';

export const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return res.status(401).json({ message: 'Authentication required' });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET as string) as { userId: number };
    
    // Buscar usuario en la base de datos
    const user = findUserById(decoded.userId);
    
    if (!user) {
      return res.status(401).json({ message: 'Invalid token' });
    }
    
    // Añadir usuario y token al objeto request
    req.user = user;
    req.token = token;
    
    next();
  } catch (error) {
    res.status(401).json({ message: 'Invalid token' });
  }
};

// Uso en rutas
app.get('/profile', authMiddleware, (req: Request, res: Response) => {
  // req.user está disponible gracias a la extensión del tipo
  res.json({ user: req.user });
});
```

### Middleware genérico con TypeScript

```typescript
// Middleware para validación de datos
import { Request, Response, NextFunction } from 'express';
import Joi from 'joi';

export const validate = <T>(schema: Joi.ObjectSchema<T>) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const { error } = schema.validate(req.body);
    
    if (error) {
      return res.status(400).json({ 
        message: 'Validation error', 
        details: error.details.map(d => d.message)
      });
    }
    
    next();
  };
};

// Esquema de validación
const userSchema = Joi.object({
  name: Joi.string().required().min(2),
  email: Joi.string().required().email(),
  role: Joi.string().valid('admin', 'user').required()
});

// Uso en rutas
app.post('/users', validate(userSchema), (req: Request, res: Response) => {
  // El cuerpo ya está validado
  // ...
});
```

### Controladores tipados

```typescript
// src/controllers/user.controller.ts
import { Request, Response } from 'express';
import { UserService } from '../services/user.service';

export class UserController {
  private userService: UserService;
  
  constructor() {
    this.userService = new UserService();
  }
  
  getAll = async (req: Request, res: Response): Promise<void> => {
    try {
      const users = await this.userService.findAll();
      res.json(users);
    } catch (error) {
      res.status(500).json({ message: 'Internal server error' });
    }
  };
  
  getById = async (req: Request, res: Response): Promise<void> => {
    try {
      const id = parseInt(req.params.id);
      const user = await this.userService.findById(id);
      
      if (!user) {
        res.status(404).json({ message: 'User not found' });
        return;
      }
      
      res.json(user);
    } catch (error) {
      res.status(500).json({ message: 'Internal server error' });
    }
  };
  
  create = async (req: Request, res: Response): Promise<void> => {
    try {
      const user = await this.userService.create(req.body);
      res.status(201).json(user);
    } catch (error) {
      res.status(500).json({ message: 'Internal server error' });
    }
  };
}

// src/routes/user.routes.ts
import { Router } from 'express';
import { UserController } from '../controllers/user.controller';

const router = Router();
const userController = new UserController();

router.get('/', userController.getAll);
router.get('/:id', userController.getById);
router.post('/', userController.create);

export default router;
```

## 8.4.3 TypeScript con Angular

Angular es un framework que ya está construido con TypeScript, lo que proporciona una integración perfecta.

### Componentes de Angular tipados

```typescript
// Componente básico de Angular
import { Component, Input, Output, EventEmitter } from '@angular/core';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  selector: 'app-user-card',
  template: `
    <div class="user-card" [ngClass]="{ 'active': isActive }">
      <h2>{{ user.name }}</h2>
      <p>{{ user.email }}</p>
      <button (click)="onSelect()">Select</button>
    </div>
  `,
  styleUrls: ['./user-card.component.css']
})
export class UserCardComponent {
  @Input() user!: User;
  @Input() isActive = false;
  @Output() selected = new EventEmitter<User>();
  
  onSelect(): void {
    this.selected.emit(this.user);
  }
}
```

### Servicios tipados

```typescript
// Servicio Angular con HttpClient
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { map, catchError } from 'rxjs/operators';

export interface User {
  id: number;
  name: string;
  email: string;
}

export interface ApiResponse<T> {
  data: T;
  message: string;
  status: number;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'https://api.example.com/users';
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<ApiResponse<User[]>>(this.apiUrl).pipe(
      map(response => response.data)
    );
  }
  
  getUserById(id: number): Observable<User> {
    return this.http.get<ApiResponse<User>>(`${this.apiUrl}/${id}`).pipe(
      map(response => response.data)
    );
  }
  
  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<ApiResponse<User>>(this.apiUrl, user).pipe(
      map(response => response.data)
    );
  }
  
  updateUser(user: User): Observable<User> {
    return this.http.put<ApiResponse<User>>(`${this.apiUrl}/${user.id}`, user).pipe(
      map(response => response.data)
    );
  }
  
  deleteUser(id: number): Observable<void> {
    return this.http.delete<ApiResponse<void>>(`${this.apiUrl}/${id}`).pipe(
      map(response => response.data)
    );
  }
}
```

## 8.4.4 TypeScript con Vue

Vue 3 fue reescrito completamente en TypeScript, lo que facilita enormemente la integración.

### Componentes de Vue con TypeScript (usando Composition API)

```typescript
// Componente Vue 3 con TypeScript y Composition API
<script lang="ts">
import { defineComponent, ref, computed } from 'vue';

interface User {
  id: number;
  name: string;
  email: string;
}

export default defineComponent({
  name: 'UserList',
  
  props: {
    initialUsers: {
      type: Array as () => User[],
      required: true
    },
    maxVisible: {
      type: Number,
      default: 10
    }
  },
  
  setup(props, { emit }) {
    // Referencias tipadas
    const users = ref<User[]>(props.initialUsers);
    const searchTerm = ref<string>('');
    
    // Valores calculados tipados
    const filteredUsers = computed<User[]>(() => {
      return users.value.filter(user => 
        user.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      ).slice(0, props.maxVisible);
    });
    
    // Funciones tipadas
    const selectUser = (user: User) => {
      emit('user-selected', user);
    };
    
    const addUser = (newUser: Omit<User, 'id'>) => {
      const id = Math.max(0, ...users.value.map(u => u.id)) + 1;
      users.value.push({
        ...newUser,
        id
      });
    };
    
    return {
      users,
      searchTerm,
      filteredUsers,
      selectUser,
      addUser
    };
  }
});
</script>
```

### Componentes de Vue con Class API

```typescript
// Componente Vue con Class API y TypeScript
<script lang="ts">
import { Component, Prop, Watch, Emit, Vue } from 'vue-property-decorator';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  name: 'user-profile'
})
export default class UserProfile extends Vue {
  @Prop({ required: true }) user!: User;
  @Prop({ default: false }) editable!: boolean;
  
  isEditing = false;
  editedName = '';
  
  @Watch('user', { immediate: true, deep: true })
  onUserChanged(newUser: User) {
    this.editedName = newUser.name;
  }
  
  startEditing() {
    if (!this.editable) return;
    this.isEditing = true;
  }
  
  @Emit('user-updated')
  saveChanges(): User {
    this.isEditing = false;
    const updatedUser = { ...this.user, name: this.editedName };
    return updatedUser;
  }
}
</script>
```

## Conclusión

La integración de TypeScript con frameworks modernos de JavaScript no solo mejora la seguridad y la confiabilidad del código, sino que también proporciona una experiencia de desarrollo superior. A través de la tipificación estática, los desarrolladores pueden detectar errores tempranamente, mejorar la documentación del código y facilitar el mantenimiento a largo plazo.

Cada framework tiene sus propias particularidades en cuanto a la integración con TypeScript:

- **React** se beneficia enormemente de TypeScript para la validación de props, el tipado de estados y el manejo seguro de eventos, lo que resulta en componentes más robustos y reutilizables.
  
- **Express** gana en claridad y seguridad al definir rutas, middleware y controladores con tipos estáticos, reduciendo errores comunes en aplicaciones backend.
  
- **Angular**, siendo un framework que ya utiliza TypeScript de forma nativa, aprovecha al máximo sus características para proporcionar una experiencia de desarrollo cohesiva y bien estructurada.
  
- **Vue** ha evolucionado para ofrecer un excelente soporte de TypeScript, especialmente en su versión 3 con la Composition API, permitiendo crear componentes más mantenibles y seguros.

La decisión de utilizar TypeScript con estos frameworks representa una inversión inicial en términos de tiempo de aprendizaje y configuración, pero los beneficios a largo plazo son sustanciales. La detección temprana de errores, la mejora en la experiencia de desarrollo y la facilidad de mantenimiento hacen que TypeScript sea una elección estratégica para proyectos de cualquier tamaño, pero especialmente para aquellos que escalan rápidamente o requieren mantenimiento a largo plazo.

A medida que el ecosistema de JavaScript continúa evolucionando, TypeScript se mantiene como una herramienta fundamental para garantizar la calidad del código en aplicaciones modernas, brindando la seguridad de un tipado estático sin sacrificar la flexibilidad y expresividad que hacen de JavaScript un lenguaje tan poderoso y versátil.
