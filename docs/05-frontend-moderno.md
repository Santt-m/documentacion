# Capítulo 5: Frontend Moderno

El desarrollo frontend ha evolucionado significativamente en los últimos años, impulsado por frameworks modernos, nuevas APIs del navegador y prácticas de desarrollo más estructuradas. Este capítulo explora las tecnologías y metodologías actuales que definen el frontend moderno.

## 5.1 Frameworks Frontend Modernos

### 5.1.1 React

React es una biblioteca de JavaScript para construir interfaces de usuario, desarrollada y mantenida por Facebook (ahora Meta). Se centra en el desarrollo basado en componentes y utiliza un DOM virtual para mejorar el rendimiento.

#### Características Principales de React

- **Componentes**: Unidades reutilizables que encapsulan lógica y UI.
- **DOM Virtual**: Representación en memoria del DOM real para optimizar actualizaciones.
- **Flujo de datos unidireccional**: Datos que fluyen de padres a hijos.
- **JSX**: Extensión sintáctica que permite escribir HTML en JavaScript.

#### Componente Funcional con Hooks

```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchUserData() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
          throw new Error('Failed to fetch user data');
        }
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchUserData();
  }, [userId]); // Dependencia: se ejecuta cuando cambia userId

  if (loading) return <div className="loading">Loading user data...</div>;
  if (error) return <div className="error">{error}</div>;
  if (!user) return <div>No user data available</div>;

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>Member since: {new Date(user.createdAt).toLocaleDateString()}</p>
      {user.bio && <section className="bio">{user.bio}</section>}
    </div>
  );
}

export default UserProfile;
```

### 5.1.2 Vue.js

Vue.js es un framework progresivo para construir interfaces de usuario, diseñado para ser adoptado incrementalmente.

#### Características Principales de Vue

- **Reactividad**: Sistema de reactividad eficiente para actualizaciones automáticas.
- **Componentes de un solo archivo (SFC)**: Combinan template, script y estilo.
- **Directivas**: Extensiones HTML especiales como v-if, v-for, v-model.
- **Composition API**: API moderna para organizar la lógica por funcionalidad.

#### Componente Vue con Composition API

```vue
<template>
  <div class="user-profile">
    <div v-if="loading" class="loading">Loading user data...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else-if="!user" class="no-data">No user data available</div>
    <div v-else>
      <h1>{{ user.name }}</h1>
      <p>Email: {{ user.email }}</p>
      <p>Member since: {{ formatDate(user.createdAt) }}</p>
      <section v-if="user.bio" class="bio">{{ user.bio }}</section>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const props = defineProps({
  userId: {
    type: String,
    required: true
  }
});

const user = ref(null);
const loading = ref(true);
const error = ref(null);

function formatDate(dateString) {
  return new Date(dateString).toLocaleDateString();
}

async function fetchUserData() {
  try {
    loading.value = true;
    const response = await fetch(`/api/users/${props.userId}`);
    if (!response.ok) {
      throw new Error('Failed to fetch user data');
    }
    user.value = await response.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
}

onMounted(fetchUserData);
</script>

<style scoped>
.user-profile {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.loading, .error, .no-data {
  padding: 20px;
  text-align: center;
}

.error {
  color: #e74c3c;
}

.bio {
  margin-top: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 5px;
}
</style>
```

### 5.1.3 Angular

Angular es un framework completo desarrollado por Google, que proporciona herramientas para crear aplicaciones frontend complejas.

#### Características Principales de Angular

- **Arquitectura basada en componentes**: Similar a React y Vue.
- **Inyección de dependencias**: Sistema robusto para gestionar dependencias.
- **TypeScript por defecto**: Tipado estático para detectar errores tempranamente.
- **RxJS**: Programación reactiva para manejar eventos asíncronos.
- **Directivas estructurales**: Como *ngIf y *ngFor.

## 5.2 React Hooks y Componentes Avanzados

Los hooks de React permiten usar estado y otras características de React sin escribir clases. Se introdujeron en React 16.8 y han transformado la forma en que se escriben componentes.

### 5.2.1 Hooks Básicos y Personalizados

#### useState

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

#### useEffect

```jsx
function DocumentTitle({ title }) {
  useEffect(() => {
    document.title = title;
    
    // Cleanup function (optional)
    return () => {
      document.title = 'React App';
    };
  }, [title]); // Dependencia: ejecuta efecto cuando title cambia
  
  return <div>The document title is now: {title}</div>;
}
```

#### Hook Personalizado

```jsx
// Hook personalizado para manejar formularios
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues({ ...values, [name]: value });
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched({ ...touched, [name]: true });
  };
  
  const resetForm = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  };
  
  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    resetForm,
    setValues,
    setErrors
  };
}

// Uso del hook personalizado
function SignupForm() {
  const { 
    values, 
    handleChange, 
    handleBlur,
    errors,
    touched,
    resetForm 
  } = useForm({
    email: '',
    password: ''
  });
  
  const handleSubmit = (e) => {
    e.preventDefault();
    // Lógica de envío
    resetForm();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
        />
        {touched.email && errors.email && (
          <div className="error">{errors.email}</div>
        )}
      </div>
      
      <div>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Password"
        />
        {touched.password && errors.password && (
          <div className="error">{errors.password}</div>
        )}
      </div>
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

### 5.2.2 useMemo y useCallback

Estos hooks ayudan a optimizar el rendimiento evitando cálculos innecesarios.

```jsx
function ExpensiveComponent({ data, filter }) {
  // useMemo memoriza el resultado de un cálculo costoso
  const filteredData = useMemo(() => {
    console.log('Filtering data...');
    return data.filter(item => item.includes(filter));
  }, [data, filter]); // Recalcula solo si data o filter cambian
  
  // useCallback memoriza una función
  const handleItemClick = useCallback((item) => {
    console.log('Item clicked:', item);
    // Lógica adicional...
  }, []); // Array vacío significa que la función nunca cambia
  
  return (
    <div>
      {filteredData.map((item, index) => (
        <div key={index} onClick={() => handleItemClick(item)}>
          {item}
        </div>
      ))}
    </div>
  );
}
```

### 5.2.3 Portals y Error Boundaries

#### Portals

Los portals permiten renderizar elementos hijos en un nodo DOM fuera de la jerarquía del componente padre.

```jsx
import React, { useState } from 'react';
import ReactDOM from 'react-dom';

function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return ReactDOM.createPortal(
    <div className="modal-overlay">
      <div className="modal">
        <button className="close-button" onClick={onClose}>
          &times;
        </button>
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>,
    document.getElementById('modal-root') // Nodo fuera de la jerarquía normal
  );
}

function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  return (
    <div className="app">
      <h1>Portal Example</h1>
      <button onClick={() => setIsModalOpen(true)}>
        Open Modal
      </button>
      
      <Modal isOpen={isModalOpen} onClose={() => setIsModalOpen(false)}>
        <h2>Modal Content</h2>
        <p>This is rendered outside the parent component's DOM hierarchy.</p>
      </Modal>
    </div>
  );
}
```

#### Error Boundaries

Las Error Boundaries son componentes de React que capturan errores de JavaScript en cualquier parte de su árbol de componentes hijos.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    // Actualiza el estado para mostrar la UI de fallback
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // Puedes registrar el error en un servicio
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      // Puedes renderizar cualquier UI de fallback
      return <h1>Something went wrong.</h1>;
    }
    
    return this.props.children;
  }
}

// Uso
function App() {
  return (
    <div>
      <ErrorBoundary>
        <ComponentThatMightError />
      </ErrorBoundary>
    </div>
  );
}
```

## 5.3 Gestión de Estado en Aplicaciones Frontend

La gestión del estado es uno de los aspectos más importantes y complejos del desarrollo frontend moderno.

### 5.3.1 Enfoques de Gestión de Estado

#### Estado Local con Hooks

Para componentes individuales o pequeños árboles de componentes.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return (/* ... */);
}
```

#### Contexto de React

Para compartir estado entre componentes sin prop drilling.

```jsx
// Crear contexto
const ThemeContext = React.createContext('light');

// Proveedor en componente padre
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <MainContent />
      <Footer />
    </ThemeContext.Provider>
  );
}

// Consumir en componente hijo (en cualquier nivel)
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      style={{
        background: theme === 'light' ? '#ffffff' : '#333333',
        color: theme === 'light' ? '#333333' : '#ffffff'
      }}
    >
      Toggle Theme
    </button>
  );
}
```

#### Redux

Biblioteca para gestión de estado global, predecible y centralizado.

```jsx
// actions.js
export const increment = () => ({
  type: 'INCREMENT'
});

export const decrement = () => ({
  type: 'DECREMENT'
});

// reducer.js
const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// store.js
import { createStore } from 'redux';
import counterReducer from './reducer';

const store = createStore(counterReducer);
export default store;

// Counter.js (con React-Redux)
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from './actions';

function Counter() {
  // Obtener estado del store
  const count = useSelector(state => state.count);
  // Obtener dispatch function
  const dispatch = useDispatch();
  
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
}
```

## 5.4 Estilos en React: Múltiples Enfoques

### 5.4.1 CSS Tradicional

```jsx
// En un archivo .css separado
import './Button.css';

function Button({ children }) {
  return <button className="custom-button">{children}</button>;
}
```

### 5.4.2 Módulos CSS

```jsx
// Button.module.css
import styles from './Button.module.css';

function Button({ children }) {
  return <button className={styles.button}>{children}</button>;
}
```

### 5.4.3 CSS-in-JS con Styled Components

```jsx
import styled from 'styled-components';

const StyledButton = styled.button`
  background-color: ${props => props.primary ? 'blue' : 'white'};
  color: ${props => props.primary ? 'white' : 'blue'};
  padding: 10px 15px;
  border: 2px solid blue;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    background-color: ${props => props.primary ? 'darkblue' : 'lightblue'};
  }
`;

function Button({ primary, children }) {
  return <StyledButton primary={primary}>{children}</StyledButton>;
}
```

### 5.4.4 Utility-First CSS con Tailwind

```jsx
// Con Tailwind CSS
function Button({ children, primary }) {
  const buttonClasses = primary
    ? "bg-blue-500 text-white hover:bg-blue-700"
    : "bg-white text-blue-500 border-blue-500 hover:bg-blue-100";
    
  return (
    <button className={`px-4 py-2 rounded border-2 ${buttonClasses}`}>
      {children}
    </button>
  );
}
```

## 5.5 Routing en Aplicaciones Frontend

El routing en aplicaciones de página única (SPA) permite navegar entre diferentes "páginas" sin recargar el navegador.

### 5.5.1 React Router

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Link, useParams } from 'react-router-dom';

// Componente de página principal
function Home() {
  return (
    <div>
      <h1>Home Page</h1>
      <p>Welcome to our application!</p>
    </div>
  );
}

// Componente de página de usuario
function UserProfile() {
  const { userId } = useParams();
  
  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {userId}</p>
    </div>
  );
}

// Componente de navegación
function Navigation() {
  return (
    <nav>
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/users/1">User 1</Link></li>
        <li><Link to="/users/2">User 2</Link></li>
      </ul>
    </nav>
  );
}

// Componente principal con rutas
function App() {
  return (
    <BrowserRouter>
      <Navigation />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users/:userId" element={<UserProfile />} />
        <Route path="*" element={<div>Page not found</div>} />
      </Routes>
    </BrowserRouter>
  );
}
```

## 5.6 Optimización de Rendimiento

### 5.6.1 Técnicas de Optimización

#### Code Splitting

Divide el bundle de JavaScript en chunks más pequeños para cargar solo lo necesario.

```jsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Importaciones lazy para code splitting
const Home = lazy(() => import('./pages/Home'));
const UserProfile = lazy(() => import('./pages/UserProfile'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/users/:userId" element={<UserProfile />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

#### Memoización

```jsx
// Componente memoizado que solo se rerenderiza cuando sus props cambian
const MemoizedComponent = React.memo(function MyComponent(props) {
  return (
    <div>
      {/* Contenido del componente */}
    </div>
  );
});
```

### 5.6.2 Herramientas de Análisis de Rendimiento

- **Lighthouse**: Para medir rendimiento, accesibilidad, SEO, etc.
- **React Profiler**: Para identificar componentes que se renderizando innecesariamente.
- **Chrome DevTools Performance tab**: Para análisis detallado de rendimiento.

## 5.7 Patrones de Diseño en Frontend

### 5.7.1 Container/Presentational Pattern

Separa la lógica de los componentes visuales.

```jsx
// Componente presentacional (UI pura)
function UserCard({ name, email, avatar }) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}

// Componente contenedor (lógica)
function UserCardContainer({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    async function fetchUser() {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    }
    
    fetchUser();
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return <UserCard {...user} />;
}
```

### 5.7.2 Render Props

Permite compartir código entre componentes usando una prop cuyo valor es una función.

```jsx
// Componente que proporciona funcionalidad de mouse tracking
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    function handleMouseMove(event) {
      setPosition({
        x: event.clientX,
        y: event.clientY
      });
    }
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);
  
  return render(position);
}

// Uso del componente
function App() {
  return (
    <div>
      <h1>Mouse Tracker Example</h1>
      <MouseTracker
        render={position => (
          <div>
            <p>Current mouse position:</p>
            <p>
              X: {position.x}, Y: {position.y}
            </p>
          </div>
        )}
      />
    </div>
  );
}
```

## 5.8 Testing en Frontend

### 5.8.1 Testing con Jest y React Testing Library

```jsx
// UserProfile.jsx
import React from 'react';

function UserProfile({ user }) {
  if (!user) return <div>No user data</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      {user.bio && <p className="bio">{user.bio}</p>}
    </div>
  );
}

export default UserProfile;

// UserProfile.test.jsx
import { render, screen } from '@testing-library/react';
import UserProfile from './UserProfile';

describe('UserProfile component', () => {
  test('renders no user data message when user is null', () => {
    render(<UserProfile user={null} />);
    expect(screen.getByText(/no user data/i)).toBeInTheDocument();
  });
  
  test('renders user info when user is provided', () => {
    const testUser = {
      name: 'Jane Doe',
      email: 'jane@example.com'
    };
    
    render(<UserProfile user={testUser} />);
    
    expect(screen.getByText('Jane Doe')).toBeInTheDocument();
    expect(screen.getByText(/email: jane@example.com/i)).toBeInTheDocument();
  });
  
  test('renders bio when provided', () => {
    const testUser = {
      name: 'Jane Doe',
      email: 'jane@example.com',
      bio: 'Software developer with 5 years experience.'
    };
    
    render(<UserProfile user={testUser} />);
    
    expect(screen.getByText(testUser.bio)).toBeInTheDocument();
    expect(screen.getByText(testUser.bio)).toHaveClass('bio');
  });
  
  test('does not render bio section when bio is not provided', () => {
    const testUser = {
      name: 'Jane Doe',
      email: 'jane@example.com'
    };
    
    const { container } = render(<UserProfile user={testUser} />);
    
    expect(container.querySelector('.bio')).not.toBeInTheDocument();
  });
});
```

## 5.9 State of the Art: Next.js y SSR/SSG

Next.js es un framework React que permite renderizado del lado del servidor (SSR) y generación de sitios estáticos (SSG), entre otras funcionalidades.

### 5.9.1 Generación Estática con Next.js (SSG)

```jsx
// pages/blog/[slug].js
import { getPostBySlug, getAllPosts } from '../../lib/api';
import markdownToHtml from '../../lib/markdownToHtml';

export default function Post({ post }) {
  return (
    <div className="post">
      <h1>{post.title}</h1>
      <div className="post-meta">
        <time>{post.date}</time>
        <span className="author">By {post.author}</span>
      </div>
      <div className="content" dangerouslySetInnerHTML={{ __html: post.content }} />
    </div>
  );
}

// Esta función se ejecuta en tiempo de build
export async function getStaticProps({ params }) {
  const post = getPostBySlug(params.slug);
  const content = await markdownToHtml(post.content || '');
  
  return {
    props: {
      post: {
        ...post,
        content
      }
    }
  };
}

// Esta función genera todas las rutas en tiempo de build
export async function getStaticPaths() {
  const posts = getAllPosts();
  
  return {
    paths: posts.map(post => ({
      params: {
        slug: post.slug
      }
    })),
    fallback: false // 404 para rutas no generadas en build
  };
}
```

### 5.9.2 Renderizado del Lado del Servidor (SSR)

```jsx
// pages/dashboard.js
import { useSession } from 'next-auth/react';

export default function Dashboard({ serverData }) {
  const { data: session } = useSession();
  
  return (
    <div>
      <h1>Dashboard</h1>
      {session ? (
        <>
          <p>Welcome, {session.user.name}!</p>
          <div className="data-container">
            <h2>Server Data</h2>
            <pre>{JSON.stringify(serverData, null, 2)}</pre>
          </div>
        </>
      ) : (
        <p>Please sign in to view your dashboard.</p>
      )}
    </div>
  );
}

// Esta función se ejecuta en cada request
export async function getServerSideProps(context) {
  // Se podría verificar auth en el servidor también
  try {
    // Obtener datos de una API o base de datos
    const res = await fetch('https://api.example.com/dashboard-data');
    const serverData = await res.json();
    
    return {
      props: { serverData }
    };
  } catch (error) {
    return {
      props: { serverData: null }
    };
  }
}
```

## 5.10 Conclusiones sobre Frontend Moderno

El desarrollo frontend moderno se caracteriza por:

- **Componentización**: Interfaces construidas con componentes reutilizables.
- **Gestión de Estado**: Patrones y herramientas para manejar estado de forma predecible.
- **Optimización de Rendimiento**: Técnicas como code splitting y lazy loading.
- **Renderizado Híbrido**: Combinación de CSR, SSR y SSG según necesidades.
- **Herramientas para Desarrolladores**: Ecosistema robusto de testing, linting, etc.

La elección de tecnologías debe basarse en los requisitos específicos del proyecto, el equipo y las necesidades del usuario final.
