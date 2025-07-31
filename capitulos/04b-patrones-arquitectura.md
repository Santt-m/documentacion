# Capítulo 4: Patrones y Arquitecturas de Software (Continuación)

## 4.2.2 Patrones Estructurales

Los patrones estructurales se ocupan de la composición de clases u objetos para formar estructuras más grandes y flexibles.

### Patrón Adapter (Adaptador)

El patrón Adapter permite que interfaces incompatibles trabajen juntas, convirtiendo la interfaz de una clase en otra que el cliente espera.

```typescript
// Patrón Adapter en TypeScript

// Interfaz objetivo que el cliente espera usar
interface MediaPlayer {
  play(audioType: string, fileName: string): void;
}

// Interfaz incompatible
interface AdvancedMediaPlayer {
  playMp4(fileName: string): void;
  playAvi(fileName: string): void;
}

// Implementación concreta de la interfaz incompatible
class Mp4Player implements AdvancedMediaPlayer {
  playMp4(fileName: string): void {
    console.log(`Reproduciendo archivo MP4: ${fileName}`);
  }
  
  playAvi(fileName: string): void {
    // No hace nada, no soporta AVI
  }
}

class AviPlayer implements AdvancedMediaPlayer {
  playMp4(fileName: string): void {
    // No hace nada, no soporta MP4
  }
  
  playAvi(fileName: string): void {
    console.log(`Reproduciendo archivo AVI: ${fileName}`);
  }
}

// Adaptador que hace que AdvancedMediaPlayer sea compatible con MediaPlayer
class MediaAdapter implements MediaPlayer {
  private advancedPlayer: AdvancedMediaPlayer;
  
  constructor(audioType: string) {
    if (audioType === 'mp4') {
      this.advancedPlayer = new Mp4Player();
    } else if (audioType === 'avi') {
      this.advancedPlayer = new AviPlayer();
    }
  }
  
  play(audioType: string, fileName: string): void {
    if (audioType === 'mp4') {
      this.advancedPlayer.playMp4(fileName);
    } else if (audioType === 'avi') {
      this.advancedPlayer.playAvi(fileName);
    }
  }
}

// Clase cliente que utiliza la interfaz MediaPlayer
class AudioPlayer implements MediaPlayer {
  private mediaAdapter: MediaAdapter;
  
  play(audioType: string, fileName: string): void {
    // Soporte nativo para archivos mp3
    if (audioType === 'mp3') {
      console.log(`Reproduciendo archivo MP3: ${fileName}`);
    }
    // Soporte mediante adaptador para otros formatos
    else if (audioType === 'mp4' || audioType === 'avi') {
      this.mediaAdapter = new MediaAdapter(audioType);
      this.mediaAdapter.play(audioType, fileName);
    }
    else {
      console.log(`Formato ${audioType} no soportado`);
    }
  }
}

// Uso
const audioPlayer = new AudioPlayer();
audioPlayer.play('mp3', 'cancion.mp3');
audioPlayer.play('mp4', 'video.mp4');
audioPlayer.play('avi', 'pelicula.avi');
audioPlayer.play('wav', 'audio.wav');
```

### Patrón Composite

El patrón Composite permite componer objetos en estructuras de árbol para representar jerarquías parte-todo. Permite a los clientes tratar objetos individuales y composiciones de objetos de manera uniforme.

```typescript
// Patrón Composite en TypeScript

// Componente base que define operaciones comunes
abstract class FileSystemComponent {
  protected name: string;
  protected parent: Directory | null = null;
  
  constructor(name: string) {
    this.name = name;
  }
  
  public setParent(parent: Directory): void {
    this.parent = parent;
  }
  
  public getParent(): Directory | null {
    return this.parent;
  }
  
  public getName(): string {
    return this.name;
  }
  
  // Métodos abstractos que deben implementarse
  public abstract getSize(): number;
  public abstract print(indent: string): void;
}

// Clase Hoja (Leaf) que representa un archivo
class File extends FileSystemComponent {
  private size: number;
  
  constructor(name: string, size: number) {
    super(name);
    this.size = size;
  }
  
  public getSize(): number {
    return this.size;
  }
  
  public print(indent: string): void {
    console.log(`${indent}File: ${this.name} (${this.size} bytes)`);
  }
}

// Clase Compuesta (Composite) que representa un directorio
class Directory extends FileSystemComponent {
  private children: FileSystemComponent[] = [];
  
  constructor(name: string) {
    super(name);
  }
  
  public add(component: FileSystemComponent): void {
    this.children.push(component);
    component.setParent(this);
  }
  
  public remove(component: FileSystemComponent): void {
    const index = this.children.indexOf(component);
    this.children.splice(index, 1);
    component.setParent(null);
  }
  
  public getSize(): number {
    return this.children.reduce((sum, child) => sum + child.getSize(), 0);
  }
  
  public print(indent: string): void {
    console.log(`${indent}Directory: ${this.name} (${this.getSize()} bytes)`);
    this.children.forEach(child => {
      child.print(indent + '  ');
    });
  }
}

// Uso
const root = new Directory('root');
const documents = new Directory('documents');
const pictures = new Directory('pictures');

root.add(documents);
root.add(pictures);

const file1 = new File('document1.txt', 100);
const file2 = new File('document2.txt', 200);
documents.add(file1);
documents.add(file2);

const file3 = new File('photo1.jpg', 1000);
const file4 = new File('photo2.jpg', 1500);
pictures.add(file3);
pictures.add(file4);

// Imprimir estructura completa
root.print('');

// Obtener tamaño total
console.log(`Tamaño total: ${root.getSize()} bytes`);
```

### Patrón Proxy

El patrón Proxy proporciona un sustituto o representante de otro objeto para controlar el acceso a este.

```typescript
// Patrón Proxy en TypeScript

// Interfaz común para el sujeto real y el proxy
interface Image {
  display(): void;
}

// Sujeto real
class RealImage implements Image {
  private fileName: string;
  
  constructor(fileName: string) {
    this.fileName = fileName;
    this.loadFromDisk();
  }
  
  private loadFromDisk(): void {
    console.log(`Cargando imagen: ${this.fileName}`);
  }
  
  public display(): void {
    console.log(`Mostrando imagen: ${this.fileName}`);
  }
}

// Proxy
class ProxyImage implements Image {
  private realImage: RealImage | null = null;
  private fileName: string;
  
  constructor(fileName: string) {
    this.fileName = fileName;
  }
  
  public display(): void {
    // Carga la imagen real solo cuando es necesario
    if (this.realImage === null) {
      this.realImage = new RealImage(this.fileName);
    }
    this.realImage.display();
  }
}

// Uso
const images: Image[] = [
  new ProxyImage("foto1.jpg"),
  new ProxyImage("foto2.jpg"),
  new ProxyImage("foto3.jpg")
];

// La imagen no se carga hasta que se llama a display()
console.log("Inicialización completa, no se ha cargado ninguna imagen aún");

// La primera imagen se carga y muestra
images[0].display();

// La primera imagen ya está cargada, solo se muestra
images[0].display();

// Las otras imágenes se cargan por primera vez cuando se muestran
images[1].display();
images[2].display();
```

## 4.2.3 Patrones de Comportamiento

Los patrones de comportamiento se ocupan de la comunicación entre objetos y cómo se distribuyen responsabilidades.

### Patrón Observer (Observador)

El patrón Observer define una dependencia uno-a-muchos entre objetos, de modo que cuando un objeto cambia su estado, todos sus dependientes son notificados y actualizados automáticamente.

```typescript
// Patrón Observer en TypeScript

// Interfaz para el observador
interface Observer {
  update(message: string): void;
}

// Clase concreta del observador
class ConcreteObserver implements Observer {
  private name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  public update(message: string): void {
    console.log(`${this.name} recibió notificación: ${message}`);
  }
}

// Sujeto observado
class Subject {
  private observers: Observer[] = [];
  private state: string = "";
  
  public attach(observer: Observer): void {
    const isExist = this.observers.includes(observer);
    if (!isExist) {
      this.observers.push(observer);
      console.log('Observador añadido');
    }
  }
  
  public detach(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index !== -1) {
      this.observers.splice(index, 1);
      console.log('Observador eliminado');
    }
  }
  
  public notify(message: string): void {
    console.log('Notificando a todos los observadores...');
    for (const observer of this.observers) {
      observer.update(message);
    }
  }
  
  public setState(state: string): void {
    this.state = state;
    this.notify(`Nuevo estado: ${this.state}`);
  }
}

// Uso
const subject = new Subject();

const observer1 = new ConcreteObserver('Observador 1');
const observer2 = new ConcreteObserver('Observador 2');
const observer3 = new ConcreteObserver('Observador 3');

subject.attach(observer1);
subject.attach(observer2);
subject.attach(observer3);

// Cambiar el estado notificará a todos los observadores
subject.setState('Estado A');

// Eliminar un observador
subject.detach(observer2);

// Este cambio solo notificará a los observadores 1 y 3
subject.setState('Estado B');
```

### Patrón Strategy (Estrategia)

El patrón Strategy define una familia de algoritmos, encapsula cada uno de ellos y los hace intercambiables. Permite que el algoritmo varíe independientemente de los clientes que lo utilizan.

```typescript
// Patrón Strategy en TypeScript

// Interfaz de estrategia
interface PaymentStrategy {
  pay(amount: number): boolean;
}

// Implementaciones concretas de estrategias
class CreditCardStrategy implements PaymentStrategy {
  private cardNumber: string;
  private cvv: string;
  private dateOfExpiry: string;
  private name: string;
  
  constructor(cardNumber: string, cvv: string, dateOfExpiry: string, name: string) {
    this.cardNumber = cardNumber;
    this.cvv = cvv;
    this.dateOfExpiry = dateOfExpiry;
    this.name = name;
  }
  
  public pay(amount: number): boolean {
    console.log(`Pagando ${amount}€ con tarjeta de crédito`);
    console.log(`Nombre: ${this.name}, Número: ${this.cardNumber.substring(0, 4)}XXXXXXXX`);
    // Lógica real de procesamiento de pago aquí
    return true;
  }
}

class PayPalStrategy implements PaymentStrategy {
  private email: string;
  private password: string;
  
  constructor(email: string, password: string) {
    this.email = email;
    this.password = password;
  }
  
  public pay(amount: number): boolean {
    console.log(`Pagando ${amount}€ con PayPal`);
    console.log(`Email: ${this.email}`);
    // Lógica real de procesamiento de pago aquí
    return true;
  }
}

class CashOnDeliveryStrategy implements PaymentStrategy {
  private address: string;
  
  constructor(address: string) {
    this.address = address;
  }
  
  public pay(amount: number): boolean {
    console.log(`Pagando ${amount}€ contra reembolso`);
    console.log(`Dirección de entrega: ${this.address}`);
    // Lógica real de procesamiento de pago aquí
    return true;
  }
}

// Clase contexto que utiliza la estrategia
class ShoppingCart {
  private items: { name: string; price: number }[] = [];
  
  public addItem(name: string, price: number): void {
    this.items.push({ name, price });
  }
  
  public calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
  
  public checkout(paymentStrategy: PaymentStrategy): boolean {
    const amount = this.calculateTotal();
    return paymentStrategy.pay(amount);
  }
}

// Uso
const cart = new ShoppingCart();
cart.addItem('Laptop', 1200);
cart.addItem('Auriculares', 150);
cart.addItem('Ratón', 50);

// Cliente elige el método de pago
console.log(`Total a pagar: ${cart.calculateTotal()}€`);

// Pago con tarjeta de crédito
const creditCardPayment = new CreditCardStrategy('1234567890123456', '123', '12/2024', 'John Doe');
cart.checkout(creditCardPayment);

// O pago con PayPal
const paypalPayment = new PayPalStrategy('john.doe@example.com', 'password');
cart.checkout(paypalPayment);

// O pago contra reembolso
const codPayment = new CashOnDeliveryStrategy('Calle Principal 123, Ciudad');
cart.checkout(codPayment);
```

### Patrón Chain of Responsibility (Cadena de Responsabilidad)

El patrón Chain of Responsibility pasa una solicitud a lo largo de una cadena de manejadores. Al recibir una solicitud, cada manejador decide si la procesa o la pasa al siguiente manejador en la cadena.

```typescript
// Patrón Chain of Responsibility en TypeScript

// Clase base para los manejadores
abstract class RequestHandler {
  private nextHandler: RequestHandler | null = null;
  
  public setNext(handler: RequestHandler): RequestHandler {
    this.nextHandler = handler;
    return handler;
  }
  
  public handleRequest(request: string, amount: number): string {
    if (this.nextHandler) {
      return this.nextHandler.handleRequest(request, amount);
    }
    return 'Fin de la cadena. Solicitud no procesada.';
  }
}

// Manejadores concretos
class AuthorizationHandler extends RequestHandler {
  public handleRequest(request: string, amount: number): string {
    if (request === 'authorizar') {
      return `Solicitud autorizada para: ${amount}€`;
    }
    console.log('AuthorizationHandler pasa la solicitud al siguiente manejador');
    return super.handleRequest(request, amount);
  }
}

class ValidationHandler extends RequestHandler {
  public handleRequest(request: string, amount: number): string {
    if (amount <= 0) {
      return 'Error: El monto debe ser positivo';
    }
    console.log('ValidationHandler pasa la solicitud al siguiente manejador');
    return super.handleRequest(request, amount);
  }
}

class LimitHandler extends RequestHandler {
  private limit: number;
  
  constructor(limit: number) {
    super();
    this.limit = limit;
  }
  
  public handleRequest(request: string, amount: number): string {
    if (amount > this.limit) {
      return `Error: El monto excede el límite de ${this.limit}€`;
    }
    console.log('LimitHandler pasa la solicitud al siguiente manejador');
    return super.handleRequest(request, amount);
  }
}

// Uso
const auth = new AuthorizationHandler();
const validation = new ValidationHandler();
const limit = new LimitHandler(1000);

// Configurar la cadena
auth.setNext(validation).setNext(limit);

// Procesar solicitudes
console.log(auth.handleRequest('autorizar', 500));  // Debería ser aprobada
console.log(auth.handleRequest('autorizar', 1500)); // Debería ser rechazada por límite
console.log(auth.handleRequest('autorizar', -100)); // Debería ser rechazada por validación
console.log(auth.handleRequest('desconocido', 500)); // Debería llegar al final de la cadena
```

## 4.3 Arquitectura de Aplicaciones Frontend

### 4.3.1 MVC (Modelo-Vista-Controlador)

El patrón MVC divide una aplicación en tres componentes interconectados: Modelo (datos), Vista (interfaz de usuario) y Controlador (lógica de negocio).

```
graph TD
    A[Usuario] -->|Interactúa con| B[Vista]
    B -->|Actualiza| A
    B -->|Envía eventos a| C[Controlador]
    C -->|Manipula| D[Modelo]
    D -->|Notifica cambios a| B
```

#### Componentes:

- **Modelo**: Representa los datos y la lógica de negocio.
- **Vista**: Representa la interfaz de usuario y cómo se muestran los datos.
- **Controlador**: Actúa como intermediario, procesando las entradas del usuario y actualizando el modelo y la vista.

#### Implementación en JavaScript:

```javascript
// MVC simple en JavaScript

// Modelo
class TodoModel {
  constructor() {
    this.todos = [];
    this.listeners = [];
  }
  
  addTodo(todoText) {
    const todo = {
      id: Date.now(),
      text: todoText,
      completed: false
    };
    this.todos.push(todo);
    this.notifyListeners();
  }
  
  removeTodo(id) {
    this.todos = this.todos.filter(todo => todo.id !== id);
    this.notifyListeners();
  }
  
  toggleTodo(id) {
    this.todos = this.todos.map(todo => 
      todo.id === id ? {...todo, completed: !todo.completed} : todo
    );
    this.notifyListeners();
  }
  
  subscribe(listener) {
    this.listeners.push(listener);
  }
  
  notifyListeners() {
    this.listeners.forEach(listener => listener(this.todos));
  }
}

// Vista
class TodoView {
  constructor() {
    this.app = document.getElementById('app');
    this.todoList = document.createElement('ul');
    this.app.appendChild(this.todoList);
    
    this.input = document.createElement('input');
    this.input.type = 'text';
    this.app.appendChild(this.input);
    
    this.addButton = document.createElement('button');
    this.addButton.textContent = 'Añadir';
    this.app.appendChild(this.addButton);
  }
  
  render(todos) {
    this.todoList.innerHTML = '';
    todos.forEach(todo => {
      const li = document.createElement('li');
      li.textContent = todo.text;
      li.style.textDecoration = todo.completed ? 'line-through' : 'none';
      li.dataset.id = todo.id;
      this.todoList.appendChild(li);
    });
  }
  
  bindAddTodo(handler) {
    this.addButton.addEventListener('click', () => {
      if (this.input.value) {
        handler(this.input.value);
        this.input.value = '';
      }
    });
  }
  
  bindToggleTodo(handler) {
    this.todoList.addEventListener('click', event => {
      if (event.target.tagName === 'LI') {
        handler(parseInt(event.target.dataset.id));
      }
    });
  }
}

// Controlador
class TodoController {
  constructor(model, view) {
    this.model = model;
    this.view = view;
    
    // Vincular view y model
    this.model.subscribe(todos => this.view.render(todos));
    this.view.bindAddTodo(this.handleAddTodo.bind(this));
    this.view.bindToggleTodo(this.handleToggleTodo.bind(this));
    
    // Renderizado inicial
    this.view.render(this.model.todos);
  }
  
  handleAddTodo(todoText) {
    this.model.addTodo(todoText);
  }
  
  handleToggleTodo(id) {
    this.model.toggleTodo(id);
  }
}

// Uso
const app = new TodoController(new TodoModel(), new TodoView());
```

### 4.3.2 MVVM (Modelo-Vista-ViewModel)

El patrón MVVM facilita la separación de la interfaz de usuario de la lógica de negocio y el comportamiento. Es popular en frameworks como Vue.js y Angular.

```
graph TD
    A[Usuario] -->|Interactúa con| B[Vista]
    B <-->|Binding bidireccional| C[ViewModel]
    C -->|Actualiza| D[Modelo]
    D -->|Notifica cambios a| C
```

#### Componentes:

- **Modelo**: Datos y lógica de negocio.
- **Vista**: Interfaz de usuario, principalmente declarativa.
- **ViewModel**: Expone datos y comandos del modelo a la vista. Implementa enlace de datos bidireccional.

#### Implementación en Vue.js:

```html
<!-- Componente MVVM simple en Vue.js -->
<!DOCTYPE html>
<html>
<head>
  <title>MVVM con Vue.js</title>
  <script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
</head>
<body>
  <div id="app">
    <h1>Lista de Tareas ({{ todos.length }})</h1>
    
    <div>
      <input v-model="newTodo" @keyup.enter="addTodo" placeholder="Nueva tarea...">
      <button @click="addTodo">Añadir</button>
    </div>
    
    <ul>
      <li v-for="todo in todos" :key="todo.id" @click="toggleTodo(todo)" 
          :style="{ textDecoration: todo.completed ? 'line-through' : 'none' }">
        {{ todo.text }}
      </li>
    </ul>
    
    <div>
      <button @click="clearCompleted">Limpiar completadas</button>
    </div>
  </div>

  <script>
    // Modelo: Representado implícitamente a través de los datos en el ViewModel
    
    // ViewModel
    new Vue({
      el: '#app',
      data: {
        todos: [
          { id: 1, text: 'Aprender MVVM', completed: false },
          { id: 2, text: 'Estudiar Vue.js', completed: true }
        ],
        newTodo: ''
      },
      methods: {
        addTodo() {
          if (this.newTodo.trim()) {
            this.todos.push({
              id: Date.now(),
              text: this.newTodo,
              completed: false
            });
            this.newTodo = '';
          }
        },
        toggleTodo(todo) {
          todo.completed = !todo.completed;
        },
        clearCompleted() {
          this.todos = this.todos.filter(todo => !todo.completed);
        }
      }
    });
    
    // Vista: Representada por el HTML con directivas Vue
  </script>
</body>
</html>
```

### 4.3.3 Flux y Redux

Flux es un patrón de arquitectura para la gestión del estado de la aplicación, popularizado por Facebook. Redux es una implementación específica de Flux con algunas diferencias clave.

```
graph LR
    A[Acción] -->|Dispara| B[Dispatcher]
    B -->|Actualiza| C[Store]
    C -->|Notifica| D[Vista]
    D -->|Genera| A
```

#### Componentes:

- **Actions**: Descripciones de los cambios que se deben realizar.
- **Dispatcher**: Encargado de enviar acciones a los stores.
- **Stores**: Contienen el estado y la lógica de la aplicación.
- **Views**: Componentes que renderizan la interfaz basada en el estado.

#### Redux en JavaScript:

```javascript
// Redux básico con React

// Acciones
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';

// Creadores de acciones
function addTodo(text) {
  return { type: ADD_TODO, text };
}

function toggleTodo(id) {
  return { type: TOGGLE_TODO, id };
}

// Reducer
function todosReducer(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          id: Date.now(),
          text: action.text,
          completed: false
        }
      ];
    case TOGGLE_TODO:
      return state.map(todo =>
        todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
      );
    default:
      return state;
  }
}

// Store
const store = Redux.createStore(todosReducer);

// Componente React
function TodoApp() {
  const [todos, setTodos] = React.useState(store.getState());
  const [text, setText] = React.useState('');
  
  // Suscripción al store
  React.useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setTodos(store.getState());
    });
    return unsubscribe;
  }, []);
  
  const handleAddTodo = () => {
    if (text.trim()) {
      store.dispatch(addTodo(text));
      setText('');
    }
  };
  
  const handleToggleTodo = (id) => {
    store.dispatch(toggleTodo(id));
  };
  
  return (
    <div>
      <h1>Lista de Tareas ({todos.length})</h1>
      
      <div>
        <input
          value={text}
          onChange={e => setText(e.target.value)}
          onKeyPress={e => e.key === 'Enter' && handleAddTodo()}
        />
        <button onClick={handleAddTodo}>Añadir</button>
      </div>
      
      <ul>
        {todos.map(todo => (
          <li
            key={todo.id}
            onClick={() => handleToggleTodo(todo.id)}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```
