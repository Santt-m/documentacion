# Capítulo 6: React y Next.js

## 6.1 Fundamentos de React

React es una biblioteca de JavaScript para construir interfaces de usuario, especialmente aplicaciones de una sola página donde la experiencia del usuario es crucial. Desarrollada y mantenida por Facebook, React ha revolucionado el desarrollo frontend con su enfoque en componentes reutilizables y un flujo de datos unidireccional.

### 6.1.1 Conceptos Fundamentales

#### Componentes

Los componentes son los bloques de construcción de cualquier aplicación React. Permiten dividir la UI en piezas independientes y reutilizables.

```jsx
// Componente Funcional
function Welcome(props) {
  return <h1>Hola, {props.name}</h1>;
}

// Componente de Clase (enfoque más antiguo)
class Welcome extends React.Component {
  render() {
    return <h1>Hola, {this.props.name}</h1>;
  }
}

// Uso
<Welcome name="Sara" />
```

#### JSX

JSX es una extensión de sintaxis para JavaScript que permite escribir marcado HTML dentro de JavaScript.

```jsx
const element = <h1>Hola, mundo!</h1>;

// Con expresiones JavaScript
const name = 'Josh Perez';
const element = <h1>Hola, {name}</h1>;

// Con atributos
const element = <img src={user.avatarUrl} alt={user.name} />;

// Con hijos
const element = (
  <div>
    <h1>Hola!</h1>
    <h2>Bienvenido a React.</h2>
  </div>
);
```

#### Props

Las props son la forma de pasar datos de un componente padre a un componente hijo.

```jsx
// Definición del componente
function Welcome(props) {
  return <h1>Hola, {props.name}</h1>;
}

// Uso
<Welcome name="Sara" />
```

#### Estado (State)

El estado es un objeto que determina cómo se comporta y renderiza un componente. A diferencia de las props, el estado es privado y completamente controlado por el componente.

```jsx
// Usando Hooks (React moderno)
import { useState } from 'react';

function Counter() {
  // Declara una variable de estado "count" con valor inicial 0
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Has hecho clic {count} veces</p>
      <button onClick={() => setCount(count + 1)}>
        Haz clic
      </button>
    </div>
  );
}

// Usando Clases (React clásico)
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  render() {
    return (
      <div>
        <p>Has hecho clic {this.state.count} veces</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Haz clic
        </button>
      </div>
    );
  }
}
```

#### Ciclo de Vida (Lifecycle)

En los componentes de clase, React proporciona métodos que permiten ejecutar código en momentos específicos del ciclo de vida de un componente.

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  // Se ejecuta después de que el componente se monte
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  // Se ejecuta antes de que el componente se desmonte
  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hola, mundo!</h1>
        <h2>Son las {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

En componentes funcionales, el Hook `useEffect` reemplaza varios métodos del ciclo de vida.

```jsx
import { useState, useEffect } from 'react';

function Clock() {
  const [date, setDate] = useState(new Date());
  
  useEffect(() => {
    const timer = setInterval(() => {
      setDate(new Date());
    }, 1000);
    
    // Función de limpieza (equivalente a componentWillUnmount)
    return () => {
      clearInterval(timer);
    };
  }, []); // Array vacío significa "ejecutar solo una vez"
  
  return (
    <div>
      <h1>Hola, mundo!</h1>
      <h2>Son las {date.toLocaleTimeString()}.</h2>
    </div>
  );
}
```

### 6.1.2 Hooks

Los Hooks son una adición en React 16.8 que permiten usar estado y otras características de React sin escribir una clase.

#### useState

Permite añadir estado a los componentes funcionales.

```jsx
import { useState } from 'react';

function Example() {
  // Declara una variable de estado llamada "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Has hecho clic {count} veces</p>
      <button onClick={() => setCount(count + 1)}>
        Haz clic
      </button>
    </div>
  );
}
```

#### useEffect

Permite ejecutar efectos secundarios en componentes funcionales.

```jsx
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar a componentDidMount y componentDidUpdate
  useEffect(() => {
    // Actualiza el título del documento usando la API del navegador
    document.title = `Has hecho clic ${count} veces`;
    
    // Opcional: función de limpieza (similar a componentWillUnmount)
    return () => {
      document.title = 'React App';
    };
  }, [count]); // Solo se ejecuta si count cambia

  return (
    <div>
      <p>Has hecho clic {count} veces</p>
      <button onClick={() => setCount(count + 1)}>
        Haz clic
      </button>
    </div>
  );
}
```

#### useContext

Acepta un objeto de contexto y devuelve el valor actual del contexto.

```jsx
import { createContext, useContext } from 'react';

// Crear un contexto
const ThemeContext = createContext('light');

function App() {
  // Proporciona un valor de contexto
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  // Consume el contexto
  const theme = useContext(ThemeContext);
  return <button className={theme}>Botón con Tema</button>;
}
```

#### useReducer

Una alternativa a useState que es más adecuada para la lógica de estado compleja.

```jsx
import { useReducer } from 'react';

// Reducer
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  // state contiene el estado actual, dispatch es la función para enviar acciones
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      Contador: {state.count}
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
    </>
  );
}
```

#### useMemo y useCallback

Estos hooks ayudan a optimizar el rendimiento evitando cálculos y renderizados innecesarios.

```jsx
import { useState, useMemo, useCallback } from 'react';

function ExpensiveCalculation({ items, query }) {
  // useMemo memoriza el resultado de un cálculo costoso
  const filteredItems = useMemo(() => {
    console.log('Filtrando items...');
    return items.filter(item => item.name.includes(query));
  }, [items, query]); // Solo recalcula si items o query cambian

  // useCallback memoriza una función
  const handleClick = useCallback(() => {
    console.log('Elemento clickeado!');
  }, []); // La función no cambia entre renderizados

  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id} onClick={handleClick}>
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

### 6.1.3 Renderizado Condicional y Listas

#### Renderizado Condicional

Permite mostrar diferentes elementos basados en ciertas condiciones.

```jsx
function UserGreeting(props) {
  const isLoggedIn = props.isLoggedIn;
  
  // Usando el operador ternario
  return (
    <h1>
      {isLoggedIn 
        ? 'Bienvenido de nuevo!' 
        : 'Por favor, inicia sesión.'}
    </h1>
  );
  
  // Alternativa con AND lógico
  return (
    <div>
      <h1>Bienvenido!</h1>
      {isLoggedIn && <p>Nos alegra verte de nuevo.</p>}
    </div>
  );
}
```

#### Renderizado de Listas

React permite renderizar colecciones de elementos de manera eficiente.

```jsx
function NumberList(props) {
  const numbers = props.numbers;
  
  // Crea una lista de elementos <li> a partir de un array
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  
  return (
    <ul>{listItems}</ul>
  );
}

// Uso
<NumberList numbers={[1, 2, 3, 4, 5]} />
```

### 6.1.4 Manejo de Formularios

React proporciona un enfoque para manejar formularios llamado "componentes controlados", donde el estado de React controla los valores de los elementos del formulario.

```jsx
import { useState } from 'react';

function NameForm() {
  const [value, setValue] = useState('');

  const handleChange = (event) => {
    setValue(event.target.value);
  };

  const handleSubmit = (event) => {
    alert('Se envió un nombre: ' + value);
    event.preventDefault();
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Nombre:
        <input type="text" value={value} onChange={handleChange} />
      </label>
      <input type="submit" value="Enviar" />
    </form>
  );
}
```

#### Formularios más Complejos

Para formularios más complejos, se pueden usar bibliotecas como Formik o React Hook Form.

```jsx
// Ejemplo con React Hook Form
import { useForm } from 'react-hook-form';

function SignupForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();
  
  const onSubmit = data => {
    console.log(data);
    // Enviar datos al servidor
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Nombre:</label>
        <input 
          {...register("name", { 
            required: "El nombre es obligatorio",
            minLength: { value: 2, message: "El nombre debe tener al menos 2 caracteres" }
          })} 
        />
        {errors.name && <p>{errors.name.message}</p>}
      </div>
      
      <div>
        <label>Email:</label>
        <input 
          {...register("email", { 
            required: "El email es obligatorio",
            pattern: { 
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: "Email inválido"
            }
          })} 
        />
        {errors.email && <p>{errors.email.message}</p>}
      </div>
      
      <div>
        <label>Contraseña:</label>
        <input 
          type="password" 
          {...register("password", { 
            required: "La contraseña es obligatoria",
            minLength: { value: 8, message: "La contraseña debe tener al menos 8 caracteres" }
          })} 
        />
        {errors.password && <p>{errors.password.message}</p>}
      </div>
      
      <button type="submit">Registrarse</button>
    </form>
  );
}
```

### 6.1.5 Comunicación entre Componentes

#### Props (Padre a Hijo)

La forma más básica de comunicación es pasar props de un componente padre a un hijo.

```jsx
function Parent() {
  const message = "Mensaje desde el padre";
  return <Child message={message} />;
}

function Child(props) {
  return <p>{props.message}</p>;
}
```

#### Prop Function (Hijo a Padre)

Para que un hijo comunique datos a un padre, el padre puede pasar una función como prop que el hijo puede llamar.

```jsx
function Parent() {
  const [message, setMessage] = useState("");
  
  const handleMessageFromChild = (childMessage) => {
    setMessage(childMessage);
  };
  
  return (
    <div>
      <p>Mensaje del hijo: {message}</p>
      <Child onMessageSend={handleMessageFromChild} />
    </div>
  );
}

function Child(props) {
  const sendMessageToParent = () => {
    props.onMessageSend("Hola desde el hijo!");
  };
  
  return (
    <button onClick={sendMessageToParent}>
      Enviar mensaje al padre
    </button>
  );
}
```

#### Context (Para Componentes Profundamente Anidados)

Context permite pasar datos a través del árbol de componentes sin tener que pasar props manualmente en cada nivel.

```jsx
import { createContext, useContext, useState } from 'react';

// Crear un contexto
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: "John", role: "Admin" });
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Main />
    </UserContext.Provider>
  );
}

function Main() {
  return (
    <div>
      <Header />
      <Content />
    </div>
  );
}

function Header() {
  const { user } = useContext(UserContext);
  return <h1>Bienvenido, {user.name}!</h1>;
}

function Content() {
  return (
    <div>
      <UserProfile />
    </div>
  );
}

function UserProfile() {
  const { user, setUser } = useContext(UserContext);
  
  return (
    <div>
      <p>Nombre: {user.name}</p>
      <p>Rol: {user.role}</p>
      <button onClick={() => setUser({ ...user, role: "User" })}>
        Cambiar a Usuario
      </button>
    </div>
  );
}
```

### 6.1.6 Componentes de Orden Superior (HOCs) y Render Props

Estos son patrones avanzados para reutilizar lógica entre componentes.

#### HOCs

Un componente de orden superior es una función que toma un componente y devuelve un nuevo componente.

```jsx
// HOC que añade funcionalidad de rastreo
function withTracking(WrappedComponent) {
  return function WithTracking(props) {
    // Lógica de rastreo
    const trackEvent = (eventName) => {
      console.log(`Event: ${eventName}`);
      // Aquí iría la lógica real de analytics
    };
    
    // Devuelve el componente original con props adicionales
    return <WrappedComponent {...props} trackEvent={trackEvent} />;
  }
}

// Componente base
function Button({ label, onClick, trackEvent }) {
  const handleClick = () => {
    trackEvent(`Button "${label}" clicked`);
    onClick();
  };
  
  return <button onClick={handleClick}>{label}</button>;
}

// Componente mejorado con tracking
const ButtonWithTracking = withTracking(Button);

// Uso
<ButtonWithTracking 
  label="Comprar Ahora" 
  onClick={() => console.log('Comprando...')} 
/>
```

#### Render Props

Un componente con render props toma una función que devuelve un elemento React y la llama en lugar de implementar su propia lógica de renderizado.

```jsx
// Componente que proporciona datos de posición del mouse
class MouseTracker extends React.Component {
  state = { x: 0, y: 0 };
  
  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  };
  
  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        {/* Llama a la prop render pasando el estado como argumento */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Uso
<MouseTracker 
  render={({ x, y }) => (
    <div>
      <h1>Mueve el mouse!</h1>
      <p>La posición actual del mouse es ({x}, {y})</p>
    </div>
  )}
/>
```
