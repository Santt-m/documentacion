# 8.2 Funciones y Clases en TypeScript

## Introducción

Las funciones y clases son componentes fundamentales de cualquier lenguaje de programación orientado a objetos, y TypeScript eleva estos elementos a un nuevo nivel gracias a su sistema de tipado estático. En este capítulo, exploraremos cómo TypeScript mejora la definición y uso de funciones y clases, proporcionando herramientas que facilitan la creación de código más robusto, mantenible y autoexplicativo.

En un entorno de desarrollo moderno, donde las aplicaciones crecen en complejidad y tamaño, la capacidad de TypeScript para añadir restricciones de tipo a funciones y clases se convierte en una ventaja competitiva significativa. No solo ayuda a prevenir errores comunes, sino que también mejora la documentación del código y facilita el trabajo colaborativo en equipos grandes.

Veremos cómo TypeScript permite definir firmas de funciones precisas, trabajar con parámetros opcionales y por defecto, implementar funciones genéricas, y estructurar clases con modificadores de acceso y características avanzadas de la programación orientada a objetos. Todo esto manteniendo la flexibilidad y expresividad que caracteriza a JavaScript.

## 8.2.1 Funciones en TypeScript

Las funciones son bloques fundamentales en cualquier aplicación JavaScript. TypeScript añade anotaciones de tipo a parámetros y valores de retorno, proporcionando claridad y seguridad adicionales.

### Definición de funciones tipadas

```typescript
// Función con tipos
function add(a: number, b: number): number {
  return a + b;
}

// Expresiones de función
const subtract = function(a: number, b: number): number {
  return a - b;
};

// Arrow functions
const multiply = (a: number, b: number): number => a * b;

// Tipo de función
type MathFunction = (a: number, b: number) => number;

const divide: MathFunction = (a, b) => {
  if (b === 0) throw new Error("No se puede dividir por cero");
  return a / b;
};
```

### Parámetros de función

TypeScript ofrece varias formas de definir parámetros de función para mayor flexibilidad:

```typescript
// Parámetros opcionales
function greet(name: string, greeting?: string): string {
  // El parámetro opcional debe venir después de los requeridos
  return greeting ? `${greeting}, ${name}!` : `Hello, ${name}!`;
}

// Parámetros con valores predeterminados
function createPoint(x: number = 0, y: number = 0): [number, number] {
  return [x, y];
}

// Parámetros rest
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

// Desestructuración de parámetros
function printUserInfo({ name, age }: { name: string; age: number }): void {
  console.log(`${name} is ${age} years old`);
}

// Sobrecarga de funciones
function process(x: number): number;
function process(x: string): string;
function process(x: number | string): number | string {
  if (typeof x === "number") {
    return x * 2;
  } else {
    return `Processed: ${x}`;
  }
}
```

### Contexto de `this` en funciones

Uno de los problemas comunes en JavaScript es la referencia ambigua de `this`. TypeScript permite especificar explícitamente el tipo de `this` en funciones:

```typescript
interface User {
  id: number;
  name: string;
  greet(this: User): void;
}

const user: User = {
  id: 1,
  name: "Alice",
  greet() {
    console.log(`Hello, my name is ${this.name}`);
  }
};

// Arrow functions y preservación del contexto
class Calculator {
  value: number = 0;
  
  // La arrow function "captura" el this del contexto donde se define
  add = (n: number): void => {
    this.value += n;
  }
  
  // Método regular - el this depende de cómo se llame
  subtract(n: number): void {
    this.value -= n;
  }
}

const calc = new Calculator();
calc.add(5); // this.value = 5
const fn = calc.subtract;
fn(2); // Error: this es undefined en este contexto
```

## 8.2.2 Clases en TypeScript

Las clases en TypeScript extienden la sintaxis de ES6 con anotaciones de tipo y características adicionales como modificadores de acceso, propiedades estáticas y abstractas.

### Definición de clases

```typescript
class Person {
  // Propiedades con modificadores de acceso
  private id: number;
  public name: string;
  protected age: number;
  readonly birthDate: Date;
  
  // Constructor
  constructor(id: number, name: string, age: number, birthDate: Date) {
    this.id = id;
    this.name = name;
    this.age = age;
    this.birthDate = birthDate;
  }
  
  // Métodos
  public greet(): string {
    return `Hello, my name is ${this.name} and I'm ${this.age} years old`;
  }
  
  // Método privado
  private generateId(): string {
    return `person_${this.id}`;
  }
  
  // Método protegido
  protected canVote(): boolean {
    return this.age >= 18;
  }
}
```

### Modificadores de acceso

TypeScript proporciona tres modificadores de acceso que controlan la visibilidad de propiedades y métodos:

- **public**: Accesible desde cualquier lugar (default)
- **private**: Solo accesible dentro de la clase
- **protected**: Accesible dentro de la clase y subclases

```typescript
class BankAccount {
  public owner: string;
  private balance: number;
  protected accountNumber: string;
  
  constructor(owner: string, initialBalance: number, accountNumber: string) {
    this.owner = owner;
    this.balance = initialBalance;
    this.accountNumber = accountNumber;
  }
  
  public deposit(amount: number): void {
    if (amount <= 0) throw new Error("Amount must be positive");
    this.balance += amount;
  }
  
  public withdraw(amount: number): boolean {
    if (amount > this.balance) return false;
    this.balance -= amount;
    return true;
  }
  
  public getBalance(): number {
    return this.balance;
  }
}

class SavingsAccount extends BankAccount {
  private interestRate: number;
  
  constructor(owner: string, initialBalance: number, accountNumber: string, interestRate: number) {
    super(owner, initialBalance, accountNumber);
    this.interestRate = interestRate;
  }
  
  public addInterest(): void {
    const interest = this.getBalance() * this.interestRate;
    this.deposit(interest);
  }
  
  // Puede acceder a accountNumber (protected)
  public getLastFourDigits(): string {
    return this.accountNumber.slice(-4);
  }
  
  // No puede acceder a balance (private)
  // public printBalance(): void {
  //   console.log(this.balance); // Error: Property 'balance' is private
  // }
}
```

### Parámetros de propiedad

TypeScript permite una sintaxis abreviada para definir y asignar propiedades a través del constructor:

```typescript
class User {
  // Los parámetros con modificadores de acceso se convierten automáticamente en propiedades
  constructor(
    public name: string, 
    private id: number, 
    protected email: string,
    readonly createdAt: Date = new Date()
  ) {}
  
  public getId(): number {
    return this.id;
  }
}

// Equivalente a:
class UserLongForm {
  public name: string;
  private id: number;
  protected email: string;
  readonly createdAt: Date;
  
  constructor(name: string, id: number, email: string, createdAt: Date = new Date()) {
    this.name = name;
    this.id = id;
    this.email = email;
    this.createdAt = createdAt;
  }
  
  public getId(): number {
    return this.id;
  }
}
```

### Propiedades y métodos estáticos

Los miembros estáticos pertenecen a la clase misma, no a instancias individuales:

```typescript
class MathUtility {
  static readonly PI: number = 3.14159;
  
  static square(x: number): number {
    return x * x;
  }
  
  static distanceBetweenPoints(x1: number, y1: number, x2: number, y2: number): number {
    return Math.sqrt(MathUtility.square(x2 - x1) + MathUtility.square(y2 - y1));
  }
}

// Uso sin instanciar la clase
console.log(MathUtility.PI); // 3.14159
console.log(MathUtility.square(5)); // 25
```

### Getters y setters

Los accessors permiten controlar el acceso a propiedades de clase:

```typescript
class Temperature {
  private _celsius: number = 0;
  
  get celsius(): number {
    return this._celsius;
  }
  
  set celsius(value: number) {
    if (value < -273.15) {
      throw new Error("Temperature below absolute zero is not possible");
    }
    this._celsius = value;
  }
  
  get fahrenheit(): number {
    return this._celsius * 9/5 + 32;
  }
  
  set fahrenheit(value: number) {
    this.celsius = (value - 32) * 5/9;
  }
}

const temp = new Temperature();
temp.celsius = 25;
console.log(temp.fahrenheit); // 77
temp.fahrenheit = 68;
console.log(temp.celsius); // 20
```

## 8.2.3 Clases abstractas e interfaces

TypeScript permite definir clases abstractas que no pueden ser instanciadas directamente y deben ser extendidas por subclases.

```typescript
// Clase abstracta
abstract class Vehicle {
  constructor(protected brand: string, protected model: string) {}
  
  abstract drive(): void; // Método abstracto que debe ser implementado
  
  honk(): void {
    console.log("Beep beep!");
  }
  
  getInfo(): string {
    return `${this.brand} ${this.model}`;
  }
}

// Implementación de clase abstracta
class Car extends Vehicle {
  constructor(brand: string, model: string, private numDoors: number) {
    super(brand, model);
  }
  
  drive(): void {
    console.log(`Driving ${this.brand} ${this.model} car...`);
  }
  
  getDoorsInfo(): string {
    return `This car has ${this.numDoors} doors`;
  }
}

// No se puede instanciar una clase abstracta
// const vehicle = new Vehicle("Generic", "Vehicle"); // Error

// Se puede instanciar una implementación concreta
const myCar = new Car("Toyota", "Corolla", 4);
myCar.drive();
myCar.honk();
console.log(myCar.getInfo());
```

### Implementación de interfaces

Las clases pueden implementar interfaces para asegurar que cumplen con un contrato específico:

```typescript
// Definir una interfaz
interface Drivable {
  start(): void;
  stop(): void;
  accelerate(speed: number): void;
}

interface Chargeable {
  charge(duration: number): void;
  getBatteryLevel(): number;
}

// Implementar múltiples interfaces
class ElectricCar implements Drivable, Chargeable {
  private isRunning: boolean = false;
  private currentSpeed: number = 0;
  private batteryLevel: number = 0;
  
  start(): void {
    this.isRunning = true;
    console.log("Car started silently");
  }
  
  stop(): void {
    this.isRunning = false;
    this.currentSpeed = 0;
    console.log("Car stopped");
  }
  
  accelerate(speed: number): void {
    if (!this.isRunning) {
      throw new Error("Cannot accelerate. Car is not running.");
    }
    this.currentSpeed += speed;
    this.batteryLevel -= speed / 10;
    console.log(`Accelerating to ${this.currentSpeed} km/h`);
  }
  
  charge(duration: number): void {
    const chargeAmount = duration * 10;
    this.batteryLevel = Math.min(100, this.batteryLevel + chargeAmount);
    console.log(`Battery charged to ${this.batteryLevel}%`);
  }
  
  getBatteryLevel(): number {
    return this.batteryLevel;
  }
}
```

## 8.2.4 Genéricos en Funciones y Clases

Los genéricos proporcionan un medio para crear componentes reutilizables que pueden trabajar con una variedad de tipos en lugar de un solo tipo.

### Funciones genéricas

```typescript
// Función genérica simple
function identity<T>(arg: T): T {
  return arg;
}

const num = identity<number>(42); // Explícitamente number
const str = identity("hello");    // Inferido como string

// Función genérica con restricciones
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "John", age: 30 };
const name = getProperty(user, "name"); // OK
// const height = getProperty(user, "height"); // Error: "height" no está en keyof user

// Función genérica para mapear arrays
function map<T, U>(array: T[], callback: (item: T) => U): U[] {
  return array.map(callback);
}

const numbers = [1, 2, 3, 4, 5];
const doubledNumbers = map(numbers, n => n * 2);        // [2, 4, 6, 8, 10]
const numberStrings = map(numbers, n => n.toString());  // ["1", "2", "3", "4", "5"]
```

### Clases genéricas

```typescript
// Clase genérica
class Box<T> {
  private content: T;
  
  constructor(value: T) {
    this.content = value;
  }
  
  getValue(): T {
    return this.content;
  }
  
  setValue(value: T): void {
    this.content = value;
  }
}

const numberBox = new Box<number>(123);
const value = numberBox.getValue(); // type is number

const stringBox = new Box("Hello world");
const greeting = stringBox.getValue(); // type is string

// Clase genérica con restricciones
interface Sizeable {
  size: number;
}

class Collection<T extends Sizeable> {
  private items: T[] = [];
  
  add(item: T): void {
    this.items.push(item);
  }
  
  getTotalSize(): number {
    return this.items.reduce((sum, item) => sum + item.size, 0);
  }
  
  getItems(): T[] {
    return [...this.items];
  }
}

const files = new Collection<{name: string, size: number}>();
files.add({name: "document.pdf", size: 1024});
files.add({name: "image.jpg", size: 2048});
console.log(files.getTotalSize()); // 3072
```

### Interfaces genéricas

```typescript
// Interfaz genérica
interface Repository<T> {
  getById(id: number): T | undefined;
  getAll(): T[];
  create(item: T): void;
  update(id: number, item: Partial<T>): boolean;
  delete(id: number): boolean;
}

// Implementación
interface User {
  id: number;
  name: string;
  email: string;
}

class UserRepository implements Repository<User> {
  private users: User[] = [];
  
  getById(id: number): User | undefined {
    return this.users.find(user => user.id === id);
  }
  
  getAll(): User[] {
    return [...this.users];
  }
  
  create(user: User): void {
    if (this.getById(user.id)) {
      throw new Error(`User with id ${user.id} already exists`);
    }
    this.users.push(user);
  }
  
  update(id: number, userData: Partial<User>): boolean {
    const index = this.users.findIndex(user => user.id === id);
    if (index === -1) return false;
    
    this.users[index] = { ...this.users[index], ...userData };
    return true;
  }
  
  delete(id: number): boolean {
    const initialLength = this.users.length;
    this.users = this.users.filter(user => user.id !== id);
    return this.users.length !== initialLength;
  }
}
```

## 8.2.5 Herencia vs. Composición

TypeScript soporta herencia a través de clases, pero también facilita la composición, que a menudo es una alternativa más flexible.

### Herencia

```typescript
// Jerarquía de herencia
class Animal {
  constructor(protected name: string) {}
  
  move(distance: number = 0): void {
    console.log(`${this.name} moved ${distance}m.`);
  }
}

class Dog extends Animal {
  constructor(name: string, private breed: string) {
    super(name);
  }
  
  bark(): void {
    console.log("Woof! Woof!");
  }
  
  // Sobrescribe el método de la clase padre
  move(distance: number = 5): void {
    console.log(`${this.name} is running...`);
    super.move(distance);
  }
  
  getBreed(): string {
    return this.breed;
  }
}

const dog = new Dog("Rex", "German Shepherd");
dog.move(10); // "Rex is running..." "Rex moved 10m."
dog.bark();   // "Woof! Woof!"
```

### Composición

```typescript
// Composición de comportamientos
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG]: ${message}`);
  }
}

class DatabaseService {
  private logger: Logger;
  
  constructor(logger: Logger) {
    this.logger = logger;
  }
  
  save(data: any): void {
    this.logger.log("Saving data to database");
    // Implementación para guardar datos
  }
  
  query(filter: any): void {
    this.logger.log(`Querying database with filter: ${JSON.stringify(filter)}`);
    // Implementación para consultar datos
  }
}

// Uso
const logger = new ConsoleLogger();
const dbService = new DatabaseService(logger);
dbService.save({ id: 1, name: "Test" });
```

### Mixin Pattern

Los mixins ofrecen una forma de reutilizar código sin herencia múltiple:

```typescript
// Definir tipos para el patrón mixin
type Constructor<T = {}> = new (...args: any[]) => T;

// Mixin para añadir funcionalidad de timestamp
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = new Date();
    
    getTimestamp() {
      return this.timestamp;
    }
  };
}

// Mixin para añadir funcionalidad de activación
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    
    activate() {
      this.isActive = true;
      return this;
    }
    
    deactivate() {
      this.isActive = false;
      return this;
    }
  };
}

// Clase base
class Person {
  constructor(public name: string) {}
}

// Aplicar mixins
const TimestampedPerson = Timestamped(Person);
const ActivatablePerson = Activatable(Person);
const EnhancedPerson = Activatable(Timestamped(Person));

// Uso
const person = new EnhancedPerson("John");
person.activate();
console.log(`${person.name} was created at ${person.getTimestamp()} and is ${person.isActive ? 'active' : 'inactive'}`);
```

## Conclusión

Las funciones y clases en TypeScript representan una evolución significativa respecto a JavaScript estándar, ofreciendo herramientas poderosas para estructurar código de manera más robusta y mantenible. El sistema de tipos aplicado a estos componentes fundamentales permite detectar errores comunes durante el desarrollo, proporciona documentación automática y mejora significativamente la experiencia de desarrollo.

La capacidad de definir interfaces, trabajar con genéricos, implementar modificadores de acceso y aplicar patrones de diseño avanzados como mixins sitúa a TypeScript como una herramienta indispensable para proyectos de mediana y gran escala. Estas características no solo mejoran la calidad del código, sino que también facilitan el trabajo en equipo al establecer contratos claros entre diferentes partes del sistema.

Es importante recordar que TypeScript no reemplaza buenas prácticas de diseño orientado a objetos. La decisión entre herencia y composición, la correcta aplicación de principios SOLID, y el uso juicioso de patrones de diseño siguen siendo responsabilidad del desarrollador. TypeScript simplemente proporciona las herramientas para implementar estas prácticas de manera más segura y expresiva.

En el próximo capítulo, exploraremos patrones avanzados y utilidades de tipos que llevan las capacidades de TypeScript al siguiente nivel, permitiendo transformaciones de tipos complejas y facilitando la integración con frameworks modernos de desarrollo.
