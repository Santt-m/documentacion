# Capítulo 8: TypeScript - El Superhéroe del Desarrollo JavaScript

## 8.1 Fundamentos de TypeScript

TypeScript es un superconjunto de JavaScript desarrollado por Microsoft que añade tipado estático opcional al lenguaje. TypeScript se compila a JavaScript plano, lo que permite su ejecución en cualquier navegador, host o motor de JavaScript.

### 8.1.1 Ventajas de TypeScript

- **Tipado estático**: Detecta errores durante la fase de desarrollo antes de ejecutar el código
- **Mejor documentación**: El sistema de tipos actúa como documentación viva
- **Mejor autocompletado e IntelliSense**: Los editores pueden proporcionar sugerencias más precisas
- **Refactorización más segura**: Los cambios se pueden realizar con mayor confianza
- **Integración con frameworks modernos**: Funciona perfectamente con React, Angular, Vue, etc.

### 8.1.2 Configuración básica

```json
// tsconfig.json - Configuración básica de TypeScript
{
  "compilerOptions": {
    "target": "es5",           // Versión de ECMAScript de salida
    "module": "commonjs",      // Sistema de módulos
    "strict": true,           // Habilitar todas las comprobaciones estrictas de tipo
    "esModuleInterop": true,  // Permitir imports de CommonJS de forma más natural
    "skipLibCheck": true,     // Saltarse la comprobación de tipos en archivos .d.ts
    "forceConsistentCasingInFileNames": true, // Asegurar consistencia en nombres de archivos
    "outDir": "./dist",       // Directorio de salida
    "rootDir": "./src"        // Directorio raíz
  },
  "include": ["src/**/*"],    // Archivos a incluir
  "exclude": ["node_modules", "**/*.test.ts"] // Archivos a excluir
}
```

## 8.2 Sistema de Tipos

El sistema de tipos de TypeScript es uno de sus aspectos más poderosos, permitiendo detectar errores comunes durante el desarrollo.

### 8.2.1 Tipos básicos

```typescript
// Tipos primitivos
let isDone: boolean = false;
let decimal: number = 6;
let color: string = "blue";
let list: number[] = [1, 2, 3];
let tuple: [string, number] = ["hello", 10];

// Enums
enum Color {Red, Green, Blue}
let c: Color = Color.Green;

// Any (evitar cuando sea posible)
let notSure: any = 4;
notSure = "maybe a string instead";

// Void (para funciones sin retorno)
function warnUser(): void {
  console.log("This is a warning message");
}

// Null y Undefined
let u: undefined = undefined;
let n: null = null;

// Never (para funciones que nunca retornan o siempre lanzan excepciones)
function error(message: string): never {
  throw new Error(message);
}
```

### 8.2.2 Interfaces

Las interfaces son una forma poderosa de definir contratos dentro de tu código y con código externo.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Propiedad opcional
  readonly createdAt: Date; // Propiedad de solo lectura
}

// Implementación de la interfaz
const newUser: User = {
  id: 1,
  name: "John Doe",
  email: "john@example.com",
  createdAt: new Date()
};

// Interfaces para funciones
interface SearchFunction {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunction = (src, sub) => src.includes(sub);

// Extensión de interfaces
interface Employee extends User {
  employeeId: string;
  department: string;
}
```

### 8.2.3 Tipos

TypeScript también permite crear alias de tipo con `type`, que pueden ser usados de manera similar a las interfaces pero con algunas diferencias importantes.

```typescript
// Alias de tipo básico
type ID = string | number;

// Tipo objeto
type Point = {
  x: number;
  y: number;
};

// Union types
type Result = "success" | "failure" | "error";

// Intersection types
type Admin = User & { permissions: string[] };

// Type con generics
type Container<T> = { value: T };

// Diferencia clave con interfaces: los types no se pueden extender después de ser definidos
```

## 8.3 Funciones en TypeScript

### 8.3.1 Definición de funciones

```typescript
// Función con tipos
function add(a: number, b: number): number {
  return a + b;
}

// Parámetros opcionales
function buildName(firstName: string, lastName?: string): string {
  return lastName ? `${firstName} ${lastName}` : firstName;
}

// Parámetros por defecto
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}!`;
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, val) => acc + val, 0);
}

// Function overloads
function processInput(input: string): string;
function processInput(input: number): number;
function processInput(input: string | number): string | number {
  if (typeof input === "string") {
    return input.toUpperCase();
  } else {
    return input * 2;
  }
}
```

### 8.3.2 Funciones de flecha

```typescript
// Función de flecha con tipo
const multiply = (a: number, b: number): number => a * b;

// Con inferencia de tipo
const divide = (a: number, b: number) => a / b;

// Contexto de this preservado
class Calculator {
  value: number = 0;
  
  add = (n: number): void => {
    this.value += n; // 'this' se refiere a la instancia de Calculator
  }
}
```

## 8.4 Clases en TypeScript

### 8.4.1 Definición de clases

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
  greet(): string {
    return `Hello, my name is ${this.name}`;
  }
  
  // Getters y setters
  get personAge(): number {
    return this.age;
  }
  
  set personAge(value: number) {
    if (value >= 0) {
      this.age = value;
    }
  }
}

// Herencia
class Employee extends Person {
  position: string;
  
  constructor(id: number, name: string, age: number, birthDate: Date, position: string) {
    super(id, name, age, birthDate); // Llamada al constructor de la clase padre
    this.position = position;
  }
  
  // Sobrescritura de métodos
  greet(): string {
    return `${super.greet()}. I work as a ${this.position}`;
  }
}
```

### 8.4.2 Sintaxis abreviada para propiedades

```typescript
// Versión corta para definir y asignar propiedades
class User {
  constructor(
    public id: number,
    public name: string,
    private email: string,
    protected role: string = "user"
  ) {}
  
  // Los parámetros con modificadores de acceso se convierten automáticamente en propiedades
}
```

### 8.4.3 Clases abstractas e interfaces

```typescript
// Clase abstracta
abstract class Vehicle {
  constructor(protected brand: string) {}
  
  abstract drive(): void; // Método abstracto que debe ser implementado
  
  honk(): void {
    console.log("Beep beep!");
  }
}

// Implementación de clase abstracta
class Car extends Vehicle {
  drive(): void {
    console.log(`Driving a ${this.brand} car`);
  }
}

// Implementación de interfaces en clases
interface Loggable {
  log(): void;
}

class Product implements Loggable {
  constructor(public name: string, public price: number) {}
  
  log(): void {
    console.log(`Product: ${this.name}, Price: $${this.price}`);
  }
}
```

## 8.5 Genéricos

Los genéricos permiten crear componentes reutilizables que pueden trabajar con varios tipos en lugar de un único tipo.

### 8.5.1 Funciones genéricas

```typescript
// Función genérica básica
function identity<T>(arg: T): T {
  return arg;
}

// Uso
let output = identity<string>("myString");
let output2 = identity(42); // Inferencia de tipo

// Múltiples parámetros de tipo
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const p = pair<string, number>("hello", 42);
```

### 8.5.2 Interfaces y clases genéricas

```typescript
// Interfaz genérica
interface Repository<T> {
  getById(id: number): T;
  getAll(): T[];
  create(item: T): void;
}

// Implementación
class UserRepository implements Repository<User> {
  private users: User[] = [];
  
  getById(id: number): User {
    return this.users.find(user => user.id === id);
  }
  
  getAll(): User[] {
    return [...this.users];
  }
  
  create(user: User): void {
    this.users.push(user);
  }
}

// Clase genérica
class Queue<T> {
  private data: T[] = [];
  
  push(item: T): void {
    this.data.push(item);
  }
  
  pop(): T | undefined {
    return this.data.shift();
  }
}

const numberQueue = new Queue<number>();
numberQueue.push(10);
```

### 8.5.3 Restricciones de tipo

```typescript
// Restricción para asegurar que T tiene una propiedad 'length'
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // Ahora podemos acceder a .length
  return arg;
}

// Funciona
logLength("hello");
logLength([1, 2, 3]);

// Error: los números no tienen propiedad 'length'
// logLength(123);

// Restricciones usando keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: "John", email: "john@example.com" };
const userName = getProperty(user, "name"); // OK
// const userAge = getProperty(user, "age"); // Error: "age" no es key de user
```

## 8.6 Utilidades avanzadas de tipo

TypeScript incluye varios tipos utilitarios que facilitan las transformaciones de tipo comunes.

### 8.6.1 Tipos utilitarios incorporados

```typescript
// Partial: hace todas las propiedades opcionales
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

// Required: hace todas las propiedades requeridas
type RequiredUser = Required<{ name?: string; email?: string }>;
// Equivalente a: { name: string; email: string }

// Readonly: hace todas las propiedades de solo lectura
const readonlyUser: Readonly<User> = {
  id: 1,
  name: "John",
  email: "john@example.com",
  age: 30
};
// readonlyUser.name = "Jane"; // Error: no se puede asignar a propiedades de solo lectura

// Pick: selecciona un subconjunto de propiedades
type UserCredentials = Pick<User, "email" | "id">;
// Equivalente a: { email: string; id: number }

// Omit: excluye ciertas propiedades
type UserWithoutId = Omit<User, "id">;
// Equivalente a: { name: string; email: string; age: number }

// Record: crea un objeto con claves de un tipo y valores de otro
type UserRoles = Record<string, User>;
const roles: UserRoles = {
  admin: { id: 1, name: "Admin User", email: "admin@example.com", age: 35 },
  editor: { id: 2, name: "Editor", email: "editor@example.com", age: 28 }
};
```

### 8.6.2 Mapped types

Los mapped types permiten transformar cada propiedad de un tipo de manera uniforme.

```typescript
// Básico mapped type
type Nullable<T> = { [P in keyof T]: T[P] | null };
const nullableUser: Nullable<User> = {
  id: 1,
  name: null,
  email: "email@example.com",
  age: null
};

// Con modificadores de propiedad
type Optional<T> = { [P in keyof T]?: T[P] };
type Readonly<T> = { readonly [P in keyof T]: T[P] };

// Eliminación de modificadores
type Mutable<T> = { -readonly [P in keyof T]: T[P] };
type Required<T> = { [P in keyof T]-?: T[P] };
```

### 8.6.3 Conditional types

Los conditional types permiten seleccionar tipos basados en condiciones, como una expresión if-else para tipos.

```typescript
type IsString<T> = T extends string ? true : false;

// Uso
type Yes = IsString<"hello">; // true
type No = IsString<42>; // false

// Con generics e infer
type UnpackArray<T> = T extends Array<infer U> ? U : T;

type Item = UnpackArray<string[]>; // string
type Same = UnpackArray<number>; // number

// Extraer tipo de retorno de una función
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet() { return "hello"; }
type GreetReturn = ReturnType<typeof greet>; // string
```

## 8.7 Patrones avanzados

### 8.7.1 Mixins

```typescript
// Base constructor type
type Constructor<T = {}> = new (...args: any[]) => T;

// Mixin function para añadir funcionalidad timestamp
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
    }
    
    deactivate() {
      this.isActive = false;
    }
  };
}

// Clase base
class User {
  constructor(public name: string) {}
}

// Aplicar mixins
const TimestampedUser = Timestamped(User);
const ActivatableTimestampedUser = Activatable(TimestampedUser);

// Uso
const user = new ActivatableTimestampedUser("John");
console.log(user.name); // John
console.log(user.getTimestamp()); // Date object
user.activate();
console.log(user.isActive); // true
```

### 8.7.2 Decorator Pattern

```typescript
// Ejemplo de decoradores de método (requiere experimentalDecorators: true en tsconfig)
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with args: ${JSON.stringify(args)}`);
    const result = originalMethod.apply(this, args);
    console.log(`Result: ${result}`);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(1, 2); 
// Logs:
// "Calling add with args: [1,2]"
// "Result: 3"
```

### 8.7.3 Builder Pattern

```typescript
class ProductBuilder {
  private product: { name?: string; price?: number; category?: string } = {};
  
  setName(name: string): ProductBuilder {
    this.product.name = name;
    return this;
  }
  
  setPrice(price: number): ProductBuilder {
    this.product.price = price;
    return this;
  }
  
  setCategory(category: string): ProductBuilder {
    this.product.category = category;
    return this;
  }
  
  build(): Product {
    // Verificar que tenemos todos los campos requeridos
    if (!this.product.name || !this.product.price) {
      throw new Error("Product requires name and price");
    }
    
    return new Product(
      this.product.name,
      this.product.price,
      this.product.category || "uncategorized"
    );
  }
}

class Product {
  constructor(
    public name: string,
    public price: number,
    public category: string
  ) {}
}

// Uso
const laptop = new ProductBuilder()
  .setName("Laptop")
  .setPrice(999)
  .setCategory("Electronics")
  .build();
```

## 8.8 Tipado avanzado en React

### 8.8.1 Componentes funcionales

```tsx
import React, { FC, ReactNode } from 'react';

// Props interface
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  children?: ReactNode;
}

// Componente funcional tipado
const Button: FC<ButtonProps> = ({ 
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
const App = () => {
  return (
    <Button 
      text="Click Me" 
      onClick={() => console.log('clicked')}
      variant="secondary"
    />
  );
};
```

### 8.8.2 Hooks con TypeScript

```tsx
import React, { useState, useEffect, useRef, useCallback } from 'react';

// useState con type inference
const [count, setCount] = useState(0);
// o explícito
const [user, setUser] = useState<User | null>(null);

// useRef con tipado
const inputRef = useRef<HTMLInputElement>(null);

// useEffect con dependencias tipadas
useEffect(() => {
  // operaciones con inputRef.current
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
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.log(error);
      return initialValue;
    }
  });

  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.log(error);
    }
  };

  return [storedValue, setValue];
}

// Uso del custom hook
const [name, setName] = useLocalStorage<string>('name', 'John');
```

## 8.9 Manejo de módulos y namespaces

### 8.9.1 Importación y exportación

```typescript
// math.ts - Exportación de módulos
export function add(x: number, y: number): number {
  return x + y;
}

export const PI = 3.14159;

// Exportación por defecto
export default class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// app.ts - Importación
import Calculator, { add, PI as MathPI } from './math';

// Importación del namespace completo
import * as MathUtils from './math';
MathUtils.add(1, 2);
```

### 8.9.2 Declaraciones de módulos

```typescript
// Declarar módulos para librerías sin tipos
declare module 'untyped-module' {
  export function doSomething(value: string): number;
  export class Helper {
    constructor(options: { debug: boolean });
    execute(): void;
  }
}

// Uso
import { doSomething } from 'untyped-module';
doSomething("test");
```

### 8.9.3 Namespaces

```typescript
// Namespaces (menos comunes en proyectos modernos, pero útiles en ciertos escenarios)
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
  
  export class LettersOnlyValidator implements StringValidator {
    isValid(s: string): boolean {
      return /^[A-Za-z]+$/.test(s);
    }
  }
  
  export class ZipCodeValidator implements StringValidator {
    isValid(s: string): boolean {
      return /^\d{5}(-\d{4})?$/.test(s);
    }
  }
}

// Uso
let validators: { [s: string]: Validation.StringValidator } = {};
validators["zip"] = new Validation.ZipCodeValidator();
validators["letters"] = new Validation.LettersOnlyValidator();
```

## 8.10 Mejores prácticas y patrones

### 8.10.1 Guía de estilo

```typescript
// Usar interfaces para objetos públicos API
interface UserService {
  getUser(id: number): Promise<User>;
}

// Usar type para uniones, intersecciones y utilidades
type UserOrAdmin = User | Admin;
type UserWithRoles = User & { roles: string[] };

// Nombres de interfaces sin prefijo "I"
// Bien
interface User { /* ... */ }
// Evitar
// interface IUser { /* ... */ }

// Consistencia en la posición de readonly
interface Config {
  readonly apiKey: string;
  readonly timeout: number;
}

// Tipo de retorno explícito para funciones públicas
function calculateTotal(items: CartItem[]): number {
  return items.reduce((total, item) => total + item.price, 0);
}

// Evitar any siempre que sea posible
// En lugar de any, usar unknown o genéricos
function parse<T>(text: string): T {
  return JSON.parse(text) as T;
}
```

### 8.10.2 Type guards

```typescript
// User-defined type guards
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

// Type guard usando type predicate
function isFish(pet: Bird | Fish): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Bird | Fish) {
  if (isFish(pet)) {
    // TypeScript sabe que pet es un Fish aquí
    pet.swim();
  } else {
    // TypeScript sabe que pet es un Bird aquí
    pet.fly();
  }
}

// Type guards con typeof
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;
  }
  return padding + value;
}

// Type guards con instanceof
class Animal {}
class Dog extends Animal {
  woof() {}
}

function makeNoise(animal: Animal) {
  if (animal instanceof Dog) {
    animal.woof();
  } else {
    console.log("unknown animal");
  }
}
```

### 8.10.3 Aserciones de tipo seguras

```typescript
// Aserciones de tipo con as
const input = document.getElementById("input") as HTMLInputElement;

// Alternativa más segura: type assertion con verificación
function assertIsHTMLInputElement(element: Element): asserts element is HTMLInputElement {
  if (!(element instanceof HTMLInputElement)) {
    throw new Error("Element is not an HTMLInputElement");
  }
}

const element = document.getElementById("input");
assertIsHTMLInputElement(element);
// Ahora TypeScript sabe que element es HTMLInputElement
element.value = "hello";

// Non-null assertion operator (!)
// Usar con precaución
function getNonNullValue(value: string | null): string {
  // Sólo usar cuando estamos seguros que value no es null
  return value!;
}
```

## 8.11 Herramientas y ecosistema

### 8.11.1 Compilación y bundling

```json
// tsconfig.json avanzado
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "lib": ["es2020", "dom"],
    "jsx": "react",
    "declaration": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "moduleResolution": "node",
    "baseUrl": "./src",
    "paths": {
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"]
    },
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```

### 8.11.2 Linting y formateo

```json
// .eslintrc.json para TypeScript
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/explicit-function-return-type": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }]
  }
}
```

### 8.11.3 Testing en TypeScript

```typescript
// Ejemplo con Jest
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  collectCoverage: true,
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts'
  ]
};

// Test file
import { sum } from '../math';

describe('sum function', () => {
  it('adds two numbers correctly', () => {
    expect(sum(1, 2)).toBe(3);
  });
  
  it('handles negative numbers', () => {
    expect(sum(-1, -2)).toBe(-3);
  });
});
```

## 8.12 Conclusiones

TypeScript se ha convertido en una herramienta esencial en el desarrollo moderno de JavaScript, proporcionando seguridad de tipos, mejor documentación y una experiencia de desarrollo superior. Aunque tiene una curva de aprendizaje, los beneficios a largo plazo en términos de mantenibilidad del código y prevención de errores hacen que valga la pena la inversión.

La adopción de TypeScript sigue creciendo, y su integración con frameworks populares como React, Angular y Vue lo convierte en una habilidad valiosa para cualquier desarrollador de JavaScript.

A medida que el ecosistema de JavaScript continúa evolucionando, TypeScript se adapta constantemente, añadiendo nuevas características y mejorando las existentes para proporcionar la mejor experiencia de desarrollo posible.
