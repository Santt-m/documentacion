# Capítulo 2: Testing: Asegurando la Calidad del Software

El testing es una fase crucial en el ciclo de vida del desarrollo de software. No solo ayuda a identificar y corregir defectos, sino que también valida que la aplicación cumple con los requisitos y funciona como se espera. Un enfoque de testing robusto es fundamental para entregar software de alta calidad, mantener la confianza del usuario y reducir los costos de mantenimiento a largo plazo.

## Tipos de Pruebas

Existen diferentes niveles y tipos de pruebas, cada uno con un propósito específico y un alcance distinto.

### Pruebas Unitarias

Las pruebas unitarias son el nivel más bajo de pruebas y se enfocan en verificar la funcionalidad de las unidades de código más pequeñas y aisladas, como funciones, métodos o clases individuales.

**Propósito**: Asegurar que cada unidad de código funciona correctamente de forma independiente.

**Características**: Son rápidas de ejecutar, fáciles de escribir y depurar, y proporcionan retroalimentación inmediata sobre el comportamiento del código.

**Herramientas comunes (JavaScript/TypeScript)**: Jest, Vitest, Mocha, Jasmine.

```typescript
// src/utils/math.ts
export function sum(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

// tests/unit/math.test.ts (usando Jest/Vitest)
import { sum, multiply } from '../src/utils/math';

describe('Math Utilities', () => {
  test('sum should correctly add two numbers', () => {
    expect(sum(1, 2)).toBe(3);
    expect(sum(-1, 1)).toBe(0);
    expect(sum(0, 0)).toBe(0);
  });

  test('multiply should correctly multiply two numbers', () => {
    expect(multiply(2, 3)).toBe(6);
    expect(multiply(-2, 4)).toBe(-8);
    expect(multiply(5, 0)).toBe(0);
  });
});
```

### Pruebas de Integración

Las pruebas de integración verifican cómo diferentes unidades o módulos de la aplicación interactúan entre sí. Se centran en las interfaces y los flujos de datos entre componentes.

**Propósito**: Detectar problemas en la comunicación y el contrato entre diferentes partes del sistema.

**Características**: Son más lentas que las unitarias pero más rápidas que las E2E. Requieren que los componentes interactúen, a veces con dependencias reales (como una base de datos de prueba).

**Herramientas comunes (Frontend)**: React Testing Library, Vue Test Utils.

**Herramientas comunes (Backend)**: Supertest (para APIs Express), Jest (con mocks para dependencias externas).

```jsx
// components/UserDisplay.tsx (Componente React)
import React from 'react';

interface UserDisplayProps {
  user: {
    name: string;
    email: string;
  };
}

const UserDisplay: React.FC<UserDisplayProps> = ({ user }) => {
  return (
    <div className="user-display">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};

export default UserDisplay;

// tests/integration/UserDisplay.test.tsx (usando React Testing Library)
import React from 'react';
import { render, screen } from '@testing-library/react';
import UserDisplay from '../src/components/UserDisplay';

describe('UserDisplay Component', () => {
  test('renders user name and email correctly', () => {
    const testUser = { name: 'Alice Smith', email: 'alice@example.com' };
    render(<UserDisplay user={testUser} />);

    // Verifica que el nombre y el email del usuario estén presentes en el documento
    expect(screen.getByText(/Alice Smith/i)).toBeInTheDocument();
    expect(screen.getByText(/alice@example.com/i)).toBeInTheDocument();
  });
});
```

```javascript
// tests/integration/api.test.ts (usando Supertest para una API Node.js)
import request from 'supertest';
import express from 'express'; // Asumiendo que tu app Express está configurada

// Mini aplicación Express para la prueba
const app = express();
app.use(express.json());
app.get('/api/users/:id', (req, res) => {
  if (req.params.id === '1') {
    return res.status(200).json({ id: 1, name: 'Test User', email: 'test@example.com' });
  }
  res.status(404).json({ message: 'User not found' });
});

describe('User API Integration', () => {
  test('GET /api/users/:id should return a user if found', async () => {
    const res = await request(app).get('/api/users/1');
    expect(res.statusCode).toEqual(200);
    expect(res.body).toHaveProperty('name', 'Test User');
    expect(res.body).toHaveProperty('email', 'test@example.com');
  });

  test('GET /api/users/:id should return 404 if user not found', async () => {
    const res = await request(app).get('/api/users/999');
    expect(res.statusCode).toEqual(404);
    expect(res.body).toHaveProperty('message', 'User not found');
  });
});
```

### Pruebas End-to-End (E2E)

Las pruebas End-to-End (E2E) simulan el comportamiento de un usuario real interactuando con la aplicación completa, desde la interfaz de usuario hasta la base de datos y los servicios externos.

**Propósito**: Validar que los flujos de usuario críticos funcionan correctamente en el sistema integrado.

**Características**: Son las más lentas y costosas de mantener, pero ofrecen la mayor confianza en la funcionalidad general de la aplicación.

**Herramientas comunes**: Playwright, Cypress, Selenium.

```typescript
// tests/e2e/login.spec.ts (usando Playwright)
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('should allow a user to log in successfully', async ({ page }) => {
    // Navegar a la página de login
    await page.goto('http://localhost:3000/login');

    // Rellenar el formulario de login
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'Password123!');

    // Hacer clic en el botón de login
    await page.click('button[type="submit"]');

    // Esperar a que la URL cambie a la página del dashboard (o similar)
    await page.waitForURL('http://localhost:3000/dashboard');

    // Verificar que el usuario está logueado (ej. por un elemento en el dashboard)
    await expect(page.locator('h1')).toHaveText('Bienvenido al Dashboard');
    await expect(page.locator('text=test@example.com')).toBeVisible();
  });

  test('should show an error message for invalid credentials', async ({ page }) => {
    await page.goto('http://localhost:3000/login');

    await page.fill('input[name="email"]', 'wrong@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');

    await page.click('button[type="submit"]');

    // Esperar a que aparezca el mensaje de error
    await expect(page.locator('text=Credenciales inválidas')).toBeVisible();
    // Asegurarse de que no se redirigió al dashboard
    await expect(page).toHaveURL('http://localhost:3000/login');
  });
});
```

### Pruebas de Rendimiento/Carga

Las pruebas de rendimiento y carga evalúan la capacidad de respuesta, estabilidad, escalabilidad y uso de recursos de una aplicación bajo una carga de trabajo específica.

**Propósito**: Identificar cuellos de botella, verificar la capacidad del sistema para manejar un número esperado de usuarios y asegurar que cumple con los requisitos de rendimiento.

**Herramientas comunes**: JMeter, K6, LoadRunner.

### Pruebas de Seguridad

Las pruebas de seguridad buscan vulnerabilidades en la aplicación que podrían ser explotadas por atacantes.

**Propósito**: Identificar debilidades como inyecciones, XSS, configuraciones erróneas, fallos de autenticación, etc.

**Herramientas comunes**: OWASP ZAP, Burp Suite, Nessus.

## Filosofías de Testing

La forma en que se distribuyen los diferentes tipos de pruebas en un proyecto se describe a menudo mediante modelos visuales.

### Pirámide de Pruebas

La Pirámide de Pruebas es un concepto propuesto por Mike Cohn que sugiere una estrategia de testing eficiente:

- **Base (más grande)**: Pruebas Unitarias: La mayoría de las pruebas deben ser unitarias. Son rápidas, baratas y proporcionan retroalimentación inmediata.

- **Medio**: Pruebas de Integración: Un número menor de pruebas de integración. Son más lentas pero verifican las interacciones clave entre componentes.

- **Cima (más pequeña)**: Pruebas End-to-End: El menor número de pruebas deben ser E2E. Son las más lentas y costosas, y se reservan para los flujos de usuario más críticos.

Esta pirámide enfatiza la importancia de la automatización y la detección temprana de errores, ya que los bugs son más baratos de corregir cuanto antes se detectan.

```
       /\       
      /E2E\     
     /-----\    
    /  Int  \   
   /---------\  
  /  Unitarias \ 
 /-------------\
```

### Ice Cream Cone Anti-Pattern

El Anti-Patrón del Cono de Helado es lo opuesto a la pirámide de pruebas. Se caracteriza por tener:

- Pocas pruebas unitarias.
- Un número moderado de pruebas de integración.
- Una gran cantidad de pruebas E2E.

Este anti-patrón es ineficiente porque las pruebas E2E son lentas, frágiles y costosas de mantener. Un fallo en una prueba E2E puede ser difícil de depurar, ya que podría indicar un problema en cualquier parte del sistema.

```
    /-----\    
   / E2E  \   
  /--------\  
 /    Int   \ 
/------------\
\__ Unit ___/
```

## Automatización de Pruebas

La automatización de pruebas es fundamental en el desarrollo moderno. Integrar las pruebas en el pipeline de Integración Continua/Despliegue Continuo (CI/CD) asegura que cada cambio de código se valide automáticamente.

**Integración Continua (CI)**: Cada vez que un desarrollador sube código al repositorio, se ejecutan automáticamente las pruebas (unitarias, integración). Esto ayuda a detectar problemas tempranamente y a mantener la base de código estable.

**Herramientas de CI**: GitHub Actions, GitLab CI, Jenkins, CircleCI.

## Mejores Prácticas en Testing

- **Escribir Pruebas Claras y Legibles**: Las pruebas deben ser fáciles de entender, casi como documentación del código.

- **Probar Casos de Éxito y de Error**: No solo pruebes el camino feliz; asegúrate de que tu código maneje correctamente los errores y las entradas inválidas.

- **Mantener las Pruebas Actualizadas**: Las pruebas deben evolucionar junto con el código de producción. Las pruebas desactualizadas son una carga.

- **Independencia de las Pruebas**: Cada prueba debe ser independiente de las demás. El orden de ejecución no debe afectar el resultado.

- **Integrar Pruebas en el Pipeline de CI/CD**: Asegura que las pruebas se ejecuten automáticamente y que los problemas se detecten de inmediato.

- **Cobertura de Código**: Apunta a una alta cobertura de código, pero no la uses como el único métrica de calidad. Una alta cobertura no garantiza que el código sea correcto, solo que se ha ejecutado.
