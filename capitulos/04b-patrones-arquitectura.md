# Capítulo 4: Patrones y Arquitecturas de Software (Continuación)

## 4.2.2 Patrones Estructurales

Los patrones estructurales constituyen una categoría fundamental en el diseño de software orientado a objetos, centrándose en cómo se componen las clases y objetos para formar estructuras más complejas y funcionales. A diferencia de los patrones creacionales que se enfocan en la instanciación, los patrones estructurales nos ayudan a organizar y ensamblar componentes para conseguir nuevas funcionalidades sin cambiar las partes individuales.

Estos patrones son particularmente valiosos cuando necesitamos integrar sistemas incompatibles, proporcionar interfaces simplificadas a subsistemas complejos, o crear jerarquías de objetos que puedan crecer dinámicamente. Al implementar patrones estructurales adecuadamente, logramos mayor flexibilidad, modularidad y reutilización en nuestros sistemas.

En esta sección, exploraremos algunos de los patrones estructurales más relevantes y aplicados en el desarrollo de software moderno, desde el clásico patrón Adapter que permite la colaboración entre interfaces incompatibles, hasta patrones como Composite que facilitan el tratamiento uniforme de objetos individuales y sus composiciones.

### Patrón Adapter (Adaptador)

El patrón Adapter aborda uno de los desafíos más comunes en el desarrollo de software: la integración de componentes con interfaces incompatibles. Este patrón actúa como un intermediario que traduce la interfaz de un componente existente en otra interfaz que los clientes esperan, permitiendo que clases con interfaces incompatibles colaboren efectivamente.

En esencia, el Adapter encapsula un objeto existente con una nueva interfaz, facilitando su uso en contextos donde originalmente no encajaría. Este patrón resulta particularmente valioso durante la integración de bibliotecas de terceros, la migración gradual de sistemas legacy, o cuando se necesita reutilizar código existente sin modificarlo.

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

El patrón Composite proporciona una solución elegante para trabajar con estructuras jerárquicas donde los objetos individuales y sus composiciones deben tratarse de manera uniforme. Este patrón organiza objetos en estructuras de árbol para representar jerarquías parte-todo, permitiendo a los clientes ignorar las diferencias entre objetos simples y compuestos.

La esencia del patrón Composite radica en definir una clase abstracta o interfaz que declare operaciones comunes tanto para componentes simples (hojas) como para compuestos (contenedores). Esto permite al código cliente trabajar con ambos tipos de objetos a través de la misma interfaz, sin preocuparse por sus diferencias estructurales subyacentes.

Este enfoque es ampliamente utilizado en interfaces gráficas de usuario (donde los controles simples y los contenedores siguen la misma interfaz), sistemas de archivos (donde archivos y directorios se manipulan de forma similar), y cualquier otro escenario donde exista una estructura jerárquica natural.

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

El patrón Proxy introduce un nivel de indirección al proporcionar un objeto sustituto que actúa como intermediario entre un cliente y un objeto real. Este sustituto controla el acceso al objeto original, permitiendo realizar operaciones antes o después de que las solicitudes lleguen al objeto real.

A diferencia del patrón Adapter que cambia la interfaz de un objeto, el Proxy mantiene la misma interfaz pero introduce comportamientos adicionales. Esta característica lo convierte en una herramienta poderosa para implementar aspectos como el control de acceso, el registro de operaciones, la carga perezosa (lazy loading), el almacenamiento en caché o la gestión de recursos remotos.

El patrón Proxy se implementa especialmente cuando necesitamos optimizar el rendimiento, mejorar la seguridad o simplificar la interacción con objetos complejos o remotos sin modificar su código original.

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

Los patrones de comportamiento constituyen un grupo fundamental de soluciones de diseño que abordan la interacción dinámica entre objetos y la asignación de responsabilidades. Estos patrones se centran en los algoritmos y la comunicación entre objetos, definiendo cómo colaboran y se distribuyen las responsabilidades para realizar tareas específicas.

A diferencia de los patrones creacionales y estructurales, los patrones de comportamiento se enfocan en el flujo de control y las relaciones dinámicas entre objetos durante la ejecución. Su implementación adecuada nos permite crear sistemas más flexibles, mantenibles y con un mayor desacoplamiento entre componentes.

Los patrones de comportamiento son particularmente valiosos cuando necesitamos definir interacciones complejas entre objetos sin crear dependencias fuertes, gestionar el flujo de trabajo de una aplicación, o implementar mecanismos de comunicación escalables entre componentes de un sistema. Estos patrones nos ayudan a escribir código más limpio, modular y adaptable a futuros cambios.

### Patrón Observer (Observador)

El patrón Observer establece un mecanismo de suscripción que permite a múltiples objetos recibir notificaciones sobre cambios en el estado de otro objeto, sin crear un acoplamiento fuerte entre ellos. Este patrón define una relación uno-a-muchos entre objetos, donde el cambio de estado en un objeto (el sujeto) desencadena la actualización automática de todos los objetos dependientes (los observadores).

Este patrón resulta fundamental en la implementación del principio de diseño de bajo acoplamiento, ya que el sujeto solo conoce que tiene una lista de observadores que implementan una interfaz común, pero no necesita conocer las clases concretas de estos observadores ni los detalles de cómo procesan las actualizaciones.

El patrón Observer es ampliamente utilizado en interfaces gráficas de usuario (donde los cambios en el modelo actualizan múltiples vistas), sistemas de eventos, arquitecturas reactivas, y cualquier escenario donde un cambio en un componente deba propagarse a otros componentes relacionados sin crear dependencias estrechas entre ellos.

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

El patrón Strategy encapsula algoritmos en clases separadas y los hace intercambiables, permitiendo que la selección del algoritmo varíe en tiempo de ejecución sin afectar al código cliente. Este patrón define una familia de algoritmos, donde cada uno se encapsula en una clase independiente que implementa una interfaz común.

La esencia de este patrón radica en la separación entre el contexto que utiliza el algoritmo y la implementación concreta del mismo. Esta separación permite cambiar la estrategia utilizada sin modificar el código cliente, facilitando la extensibilidad y el mantenimiento del sistema.

El patrón Strategy es particularmente valioso cuando tenemos múltiples formas de realizar una operación, cuando necesitamos seleccionar algoritmos en tiempo de ejecución según ciertas condiciones, o cuando queremos evitar una proliferación de condicionales complejos para seleccionar comportamientos. Este enfoque promueve el principio Open/Closed de SOLID, permitiendo extender el sistema con nuevos algoritmos sin modificar el código existente.

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

El patrón Chain of Responsibility establece un mecanismo elegante para procesar solicitudes secuencialmente a través de una serie de objetos procesadores. Este patrón crea una cadena de objetos receptores donde cada uno tiene la oportunidad de procesar una solicitud o pasarla al siguiente en la cadena, evitando acoplar el emisor de la solicitud con sus receptores específicos.

La clave de este patrón está en la flexibilidad que proporciona: tanto el orden de procesamiento como la composición de la cadena pueden modificarse dinámicamente en tiempo de ejecución. Además, cada manejador se centra exclusivamente en su responsabilidad específica, siguiendo el principio de responsabilidad única.

Este patrón es especialmente útil cuando una solicitud debe ser procesada por múltiples objetos sin especificar de antemano qué objeto la manejará, cuando el conjunto de objetos que pueden procesar una solicitud debe determinarse dinámicamente, o cuando se desea desacoplar el emisor y los receptores de una solicitud. Lo encontramos implementado en sistemas de manejo de excepciones, middleware en frameworks web, validación en cascada, y filtros de procesamiento secuencial.

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

La arquitectura frontend ha experimentado una evolución significativa en la última década, transformándose desde simples scripts que manipulaban el DOM hasta sofisticados sistemas que gestionan complejos estados de aplicación y flujos de datos. Esta evolución ha sido impulsada por la creciente complejidad de las aplicaciones web modernas, que ahora manejan interacciones ricas, actualizaciones en tiempo real y estados de aplicación complejos.

Una arquitectura frontend bien diseñada proporciona beneficios cruciales: mejora la experiencia del usuario mediante interfaces más receptivas, facilita el desarrollo paralelo por parte de equipos, simplifica el mantenimiento a largo plazo y mejora la testabilidad del código. En esta sección, exploraremos los patrones arquitectónicos más influyentes que han dado forma al desarrollo frontend moderno.

### 4.3.1 MVC (Modelo-Vista-Controlador)

El patrón arquitectónico MVC ha sido uno de los más influyentes en el desarrollo de software, especialmente en aplicaciones con interfaces gráficas. Este patrón divide una aplicación interactiva en tres componentes interconectados pero claramente separados: el Modelo que gestiona los datos y la lógica de negocio, la Vista que se encarga de la presentación y la interfaz de usuario, y el Controlador que actúa como intermediario procesando las entradas del usuario y coordinando el Modelo y la Vista.

Originalmente concebido en los años 70 para el lenguaje Smalltalk, MVC ha evolucionado y se ha adaptado a numerosos entornos y frameworks, convirtiéndose en la base conceptual de muchos frameworks web tanto en el backend como en el frontend. Su principio fundamental de separación de responsabilidades facilita el desarrollo paralelo, mejora la mantenibilidad y permite reutilizar componentes en diferentes partes de la aplicación.

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

El patrón MVVM (Modelo-Vista-ViewModel) surgió como una evolución del patrón MVC, especialmente diseñado para aprovechar las capacidades de enlace de datos (data binding) presentes en las tecnologías modernas de interfaz de usuario. Desarrollado inicialmente por Microsoft para sus tecnologías WPF y Silverlight, MVVM ha ganado amplia adopción en el desarrollo web frontend gracias a frameworks como Vue.js, Angular y Knockout.js.

La característica distintiva de MVVM es el ViewModel, que actúa como un convertidor especializado que transforma los datos del Modelo en información que la Vista puede presentar fácilmente, y viceversa. A diferencia del Controlador en MVC, el ViewModel no tiene conocimiento directo de la Vista, sino que expone propiedades y comandos a los que la Vista se enlaza mediante mecanismos de binding, generalmente declarativos.

Este patrón facilita una verdadera separación de responsabilidades, permite una mejor testabilidad del código (especialmente el ViewModel), y aprovecha al máximo las capacidades de enlace bidireccional de datos que muchos frameworks modernos ofrecen. MVVM brilla especialmente en aplicaciones con interfaces complejas donde el estado de la UI debe sincronizarse constantemente con los datos subyacentes.

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

Flux representa un cambio paradigmático en la gestión del estado de aplicaciones frontend, introducido por Facebook para abordar los desafíos de mantenibilidad y previsibilidad en aplicaciones complejas. A diferencia de los patrones tradicionales como MVC donde el flujo de datos puede ser bidireccional, Flux impone un flujo de datos unidireccional que hace que los cambios de estado sean más predecibles y fáciles de rastrear.

Este patrón surgió como respuesta a los problemas de cascadas de actualizaciones y dificultades de depuración que Facebook experimentó con aplicaciones React de gran escala. El concepto fundamental de Flux es que los datos fluyen en una sola dirección: las acciones son despachadas por vistas o servicios, procesadas por un dispatcher central, que actualiza los stores (almacenes de estado), los cuales finalmente notifican a las vistas para que se actualicen.

Redux, por su parte, emergió como una implementación refinada de los principios de Flux, introduciendo conceptos adicionales como un único store para toda la aplicación, reducers puros para transformar el estado, y middleware para manejar efectos secundarios. Desarrollado por Dan Abramov, Redux ha ganado enorme popularidad en el ecosistema React y más allá, gracias a su enfoque simple pero potente para la gestión predecible del estado de la aplicación.

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

## Conclusiones sobre Patrones de Diseño y Arquitecturas Frontend

A lo largo de este capítulo, hemos explorado una amplia gama de patrones de diseño y arquitecturas que constituyen la columna vertebral del desarrollo de software moderno. Estos patrones representan décadas de experiencia colectiva y mejores prácticas en la industria, refinadas para abordar desafíos recurrentes en el diseño y construcción de sistemas complejos.

Los patrones estructurales como Adapter, Composite y Proxy nos ofrecen mecanismos elegantes para organizar objetos y clases en estructuras flexibles y mantenibles. Los patrones de comportamiento como Observer, Strategy y Chain of Responsibility nos permiten definir interacciones sofisticadas entre componentes minimizando el acoplamiento. Y finalmente, las arquitecturas frontend como MVC, MVVM y Flux/Redux proporcionan frameworks conceptuales para estructurar aplicaciones completas de manera coherente.

Sin embargo, es fundamental recordar que los patrones de diseño son herramientas, no objetivos en sí mismos. La implementación de un patrón debe siempre estar justificada por las necesidades específicas del sistema y el contexto del problema. Como señaló el famoso arquitecto Christopher Alexander, cuyo trabajo inspiró el movimiento de patrones de diseño en software: "Un patrón es una solución recurrente a un problema en un contexto".

La verdadera maestría en la aplicación de patrones de diseño no consiste en memorizar su estructura, sino en reconocer cuándo un problema particular puede beneficiarse de un patrón específico y cómo adaptarlo a las circunstancias únicas de cada proyecto. Con este conocimiento, podemos crear sistemas más flexibles, mantenibles y robustos que resistan la prueba del tiempo y la evolución de los requisitos.
```
