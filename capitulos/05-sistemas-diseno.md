# Capítulo 5: Sistemas de Diseño (Design Systems)

## 5.1 Fundamentos de los Sistemas de Diseño

Un sistema de diseño es una colección de componentes reutilizables, guiados por estándares claros, que pueden ensamblarse para construir cualquier número de aplicaciones. Va más allá de una simple biblioteca de componentes o guía de estilo, representando un enfoque holístico para el diseño de productos digitales.

### 5.1.1 Componentes Clave de un Sistema de Diseño

1. **Principios de Diseño**: Valores fundamentales que guían las decisiones de diseño.
2. **Tokens de Diseño**: Variables que almacenan valores visuales como colores, espaciado, tipografía.
3. **Componentes**: Elementos de UI reutilizables que implementan los tokens y siguen los principios.
4. **Patrones**: Soluciones probadas a problemas recurrentes de UI/UX.
5. **Documentación**: Guías sobre uso, implementación y desarrollo.

### 5.1.2 Beneficios de los Sistemas de Diseño

- **Consistencia**: Experiencia de usuario coherente a través de productos y plataformas.
- **Eficiencia**: Reducción significativa del tiempo de desarrollo.
- **Escalabilidad**: Facilita el crecimiento del producto y del equipo.
- **Calidad**: Componentes probados y refinados que cumplen estándares de accesibilidad y usabilidad.
- **Colaboración**: Mejora la comunicación entre diseñadores y desarrolladores.

### 5.1.3 Ejemplos de Sistemas de Diseño Populares

- **Material Design (Google)**: Enfoque en el diseño basado en papel con animaciones y efectos de profundidad.
- **Human Interface Guidelines (Apple)**: Principios para crear interfaces que funcionan armoniosamente con los dispositivos Apple.
- **Fluent Design System (Microsoft)**: Énfasis en movimiento, profundidad, materiales y luz.
- **Atlassian Design System**: Centrado en la claridad y la eficiencia para productos empresariales.
- **Lightning Design System (Salesforce)**: Componentes empresariales con accesibilidad incorporada.

## 5.2 Tokens de Diseño

Los tokens de diseño son variables con nombre que almacenan valores visuales y se utilizan en lugar de valores hardcodeados. Actúan como la "fuente única de verdad" para los valores de diseño.

### 5.2.1 Tipos de Tokens

```javascript
// tokens.js - Ejemplo de definición de tokens de diseño

// 1. Tokens Globales (Primitivos)
const globalTokens = {
  // Paleta de colores
  colors: {
    blue100: '#DAECFF',
    blue200: '#B6DAFF',
    blue300: '#84CAFF',
    blue400: '#53B1FD',
    blue500: '#2E90FA',
    blue600: '#1570EF',
    blue700: '#175CD3',
    blue800: '#1849A9',
    
    gray100: '#F2F4F7',
    gray200: '#E4E7EC',
    gray300: '#D0D5DD',
    gray400: '#98A2B3',
    gray500: '#667085',
    gray600: '#475467',
    gray700: '#344054',
    gray800: '#1D2939',
    
    white: '#FFFFFF',
    black: '#000000',
  },
  
  // Tipografía
  fontFamilies: {
    sans: "'Inter', -apple-system, BlinkMacSystemFont, sans-serif",
    serif: "Georgia, serif",
    mono: "'IBM Plex Mono', monospace",
  },
  
  // Tamaños de fuente (en rem para mejor accesibilidad)
  fontSizes: {
    xs: '0.75rem',    // 12px
    sm: '0.875rem',   // 14px
    md: '1rem',       // 16px
    lg: '1.125rem',   // 18px
    xl: '1.25rem',    // 20px
    '2xl': '1.5rem',  // 24px
    '3xl': '1.875rem', // 30px
    '4xl': '2.25rem', // 36px
    '5xl': '3rem',    // 48px
  },
  
  // Pesos de fuente
  fontWeights: {
    regular: 400,
    medium: 500,
    semiBold: 600,
    bold: 700,
  },
  
  // Espaciado
  spacing: {
    '0': '0',
    '1': '0.25rem',  // 4px
    '2': '0.5rem',   // 8px
    '3': '0.75rem',  // 12px
    '4': '1rem',     // 16px
    '5': '1.25rem',  // 20px
    '6': '1.5rem',   // 24px
    '8': '2rem',     // 32px
    '10': '2.5rem',  // 40px
    '12': '3rem',    // 48px
    '16': '4rem',    // 64px
    '20': '5rem',    // 80px
  },
  
  // Radios de borde
  borderRadius: {
    none: '0',
    sm: '0.125rem',  // 2px
    md: '0.375rem',  // 6px
    lg: '0.5rem',    // 8px
    xl: '0.75rem',   // 12px
    '2xl': '1rem',   // 16px
    full: '9999px',
  },
  
  // Sombras
  shadows: {
    sm: '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
    md: '0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06)',
    lg: '0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05)',
    xl: '0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04)',
  },
};

// 2. Tokens Semánticos (Alias que referencian a los globales)
const semanticTokens = {
  colors: {
    // Colores de marca
    primary: globalTokens.colors.blue600,
    primaryLight: globalTokens.colors.blue400,
    primaryDark: globalTokens.colors.blue800,
    
    // Colores de interfaz
    background: globalTokens.colors.white,
    backgroundAlt: globalTokens.colors.gray100,
    backgroundInverse: globalTokens.colors.gray800,
    
    // Colores de texto
    text: globalTokens.colors.gray800,
    textMuted: globalTokens.colors.gray500,
    textInverse: globalTokens.colors.white,
    
    // Colores de estado
    success: '#12B76A',
    warning: '#F79009',
    error: '#F04438',
    info: '#53B1FD',
  },
  
  // Configuraciones específicas de componentes
  buttons: {
    primary: {
      background: semanticTokens.colors.primary,
      text: globalTokens.colors.white,
      hoverBackground: globalTokens.colors.blue700,
      borderRadius: globalTokens.borderRadius.md,
    },
    secondary: {
      background: globalTokens.colors.white,
      text: semanticTokens.colors.primary,
      border: semanticTokens.colors.primary,
      hoverBackground: globalTokens.colors.blue100,
      borderRadius: globalTokens.borderRadius.md,
    },
  }
};

export { globalTokens, semanticTokens };
```

### 5.2.2 Implementación con CSS Variables

```css
/* tokens.css - Implementación con variables CSS */
:root {
  /* Colores */
  --color-blue-100: #DAECFF;
  --color-blue-500: #2E90FA;
  --color-blue-700: #175CD3;
  
  --color-gray-100: #F2F4F7;
  --color-gray-500: #667085;
  --color-gray-800: #1D2939;
  
  /* Colores semánticos */
  --color-primary: var(--color-blue-500);
  --color-background: #FFFFFF;
  --color-text: var(--color-gray-800);
  
  /* Espaciado */
  --space-1: 0.25rem;
  --space-4: 1rem;
  --space-8: 2rem;
  
  /* Tipografía */
  --font-family-base: 'Inter', -apple-system, sans-serif;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  
  /* Componentes */
  --button-padding: var(--space-1) var(--space-4);
  --button-border-radius: 0.375rem;
}

/* Uso de variables */
.button-primary {
  background-color: var(--color-primary);
  color: var(--color-background);
  padding: var(--button-padding);
  border-radius: var(--button-border-radius);
  font-family: var(--font-family-base);
}

.button-primary:hover {
  background-color: var(--color-blue-700);
}
```

## 5.3 Componentes Reutilizables

Los componentes son bloques de construcción para la interfaz de usuario que implementan los tokens de diseño y siguen los patrones definidos en el sistema.

### 5.3.1 Anatomía de un Componente de Diseño

- **Propiedades (Props)**: Configuran el aspecto y comportamiento.
- **Estados**: Diferentes apariencias basadas en la interacción (hover, focus, disabled).
- **Variantes**: Versiones predefinidas para diferentes contextos.
- **Documentación**: Guía de uso y ejemplos.
- **Pruebas**: Garantías de funcionamiento y accesibilidad.

### 5.3.2 Ejemplo de Componente Button en React

```jsx
// Button.tsx
import React from 'react';
import styled from 'styled-components';

// Tipos para las props
export interface ButtonProps {
  /**
   * Contenido del botón
   */
  children: React.ReactNode;
  
  /**
   * Variante que determina la apariencia visual
   */
  variant?: 'primary' | 'secondary' | 'tertiary' | 'danger';
  
  /**
   * Tamaño del botón
   */
  size?: 'small' | 'medium' | 'large';
  
  /**
   * Si el botón está deshabilitado
   */
  disabled?: boolean;
  
  /**
   * Si el botón ocupa todo el ancho disponible
   */
  fullWidth?: boolean;
  
  /**
   * Función a ejecutar al hacer clic
   */
  onClick?: () => void;
  
  /**
   * Atributo type del botón HTML
   */
  type?: 'button' | 'submit' | 'reset';
}

// Estilos usando tokens a través de styled-components
const StyledButton = styled.button<ButtonProps>`
  /* Estilos base */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--button-border-radius);
  font-family: var(--font-family-base);
  font-weight: var(--font-weight-medium);
  transition: all 0.2s ease;
  cursor: pointer;
  
  /* Tamaños */
  ${props => {
    switch(props.size) {
      case 'small':
        return `
          padding: var(--space-1) var(--space-2);
          font-size: var(--font-size-sm);
        `;
      case 'large':
        return `
          padding: var(--space-3) var(--space-6);
          font-size: var(--font-size-lg);
        `;
      default:
        return `
          padding: var(--space-2) var(--space-4);
          font-size: var(--font-size-base);
        `;
    }
  }}
  
  /* Variantes */
  ${props => {
    switch(props.variant) {
      case 'secondary':
        return `
          background-color: transparent;
          color: var(--color-primary);
          border: 1px solid var(--color-primary);
          
          &:hover:not(:disabled) {
            background-color: var(--color-blue-100);
          }
        `;
      case 'tertiary':
        return `
          background-color: transparent;
          color: var(--color-primary);
          border: none;
          
          &:hover:not(:disabled) {
            background-color: var(--color-blue-100);
          }
        `;
      case 'danger':
        return `
          background-color: var(--color-error);
          color: white;
          border: none;
          
          &:hover:not(:disabled) {
            background-color: var(--color-error-dark);
          }
        `;
      default:
        return `
          background-color: var(--color-primary);
          color: white;
          border: none;
          
          &:hover:not(:disabled) {
            background-color: var(--color-blue-700);
          }
        `;
    }
  }}
  
  /* Estado deshabilitado */
  ${props => props.disabled && `
    opacity: 0.5;
    cursor: not-allowed;
  `}
  
  /* Ancho completo */
  ${props => props.fullWidth && `
    width: 100%;
  `}
  
  &:focus {
    outline: none;
    box-shadow: 0 0 0 3px var(--color-blue-200);
  }
`;

// Componente Button
export const Button = ({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  fullWidth = false,
  onClick,
  type = 'button',
  ...rest
}: ButtonProps) => {
  return (
    <StyledButton
      variant={variant}
      size={size}
      disabled={disabled}
      fullWidth={fullWidth}
      onClick={onClick}
      type={type}
      {...rest}
    >
      {children}
    </StyledButton>
  );
};

export default Button;
```

### 5.3.3 Documentación del Componente (Ejemplo con Storybook)

```jsx
// Button.stories.tsx
import React from 'react';
import { Meta, Story } from '@storybook/react';
import Button, { ButtonProps } from './Button';

export default {
  title: 'Components/Button',
  component: Button,
  argTypes: {
    variant: {
      control: {
        type: 'select',
        options: ['primary', 'secondary', 'tertiary', 'danger'],
      },
    },
    size: {
      control: {
        type: 'select',
        options: ['small', 'medium', 'large'],
      },
    },
    disabled: {
      control: 'boolean',
    },
    fullWidth: {
      control: 'boolean',
    },
    onClick: { action: 'clicked' },
  },
} as Meta;

const Template: Story<ButtonProps> = (args) => <Button {...args} />;

export const Primary = Template.bind({});
Primary.args = {
  variant: 'primary',
  children: 'Primary Button',
};

export const Secondary = Template.bind({});
Secondary.args = {
  variant: 'secondary',
  children: 'Secondary Button',
};

export const Tertiary = Template.bind({});
Tertiary.args = {
  variant: 'tertiary',
  children: 'Tertiary Button',
};

export const Danger = Template.bind({});
Danger.args = {
  variant: 'danger',
  children: 'Danger Button',
};

export const Small = Template.bind({});
Small.args = {
  size: 'small',
  children: 'Small Button',
};

export const Large = Template.bind({});
Large.args = {
  size: 'large',
  children: 'Large Button',
};

export const Disabled = Template.bind({});
Disabled.args = {
  disabled: true,
  children: 'Disabled Button',
};

export const FullWidth = Template.bind({});
FullWidth.args = {
  fullWidth: true,
  children: 'Full Width Button',
};
```

## 5.4 Implementación de un Sistema de Diseño

### 5.4.1 Estrategia de Implementación

1. **Auditoría de la Interfaz**: Analizar los componentes y patrones existentes.
2. **Definir Tokens**: Establecer variables para colores, espaciado, tipografía, etc.
3. **Crear Componentes Fundamentales**: Empezar con componentes base como botones, tarjetas, formularios.
4. **Documentación**: Establecer guías de uso y ejemplos interactivos.
5. **Integración Gradual**: Incorporar el sistema en productos existentes de manera incremental.
6. **Mantenimiento y Evolución**: Actualizar el sistema basado en feedback y nuevas necesidades.

### 5.4.2 Herramientas para Sistemas de Diseño

- **Figma/Sketch/Adobe XD**: Para crear y mantener los activos de diseño.
- **Storybook**: Documentación de componentes y entorno de desarrollo.
- **Styled Components / Emotion**: CSS-in-JS para implementar estilos basados en tokens.
- **Chromatic**: Pruebas visuales y revisión de cambios.
- **Zeroheight/Notion/Docusaurus**: Para documentación más completa.
- **Theme UI / Chakra UI**: Frameworks que facilitan la implementación.

### 5.4.3 Caso de Estudio: Implementación Gradual

```javascript
// Ejemplo de migración gradual usando styled-components y ThemeProvider

// 1. Definir tema basado en tokens
const theme = {
  colors: {
    primary: '#2E90FA',
    secondary: '#667085',
    background: '#FFFFFF',
    text: '#1D2939',
    // ...otros colores
  },
  fontSizes: {
    small: '0.875rem',
    medium: '1rem',
    large: '1.25rem',
    // ...otros tamaños
  },
  space: [0, '0.25rem', '0.5rem', '1rem', '2rem', '4rem'],
  // ...otras propiedades
};

// 2. Proveer el tema a toda la aplicación
import { ThemeProvider } from 'styled-components';
import App from './App';

ReactDOM.render(
  <ThemeProvider theme={theme}>
    <App />
  </ThemeProvider>,
  document.getElementById('root')
);

// 3. Crear componentes que usan el tema
import styled from 'styled-components';

const Button = styled.button`
  background-color: ${props => props.theme.colors.primary};
  color: white;
  padding: ${props => `${props.theme.space[2]} ${props.theme.space[3]}`};
  border-radius: 4px;
  border: none;
  font-size: ${props => props.theme.fontSizes.medium};
  
  &:hover {
    background-color: ${props => darken(0.1, props.theme.colors.primary)};
  }
`;

// 4. Utilizar en los componentes existentes
const LegacyComponent = () => {
  return (
    <div className="legacy-container">
      {/* Componente legacy */}
      <button className="old-button">Botón Antiguo</button>
      
      {/* Nuevo componente del sistema de diseño */}
      <Button>Botón del Sistema de Diseño</Button>
    </div>
  );
};
```

## 5.5 Mejores Prácticas y Consideraciones

### 5.5.1 Accesibilidad

Los sistemas de diseño deben incorporar la accesibilidad desde el principio, no como una característica adicional.

```jsx
// Ejemplo de componente accesible
const AccessibleButton = styled.button`
  /* Estilos visuales */
  background-color: var(--color-primary);
  color: white;
  padding: var(--space-2) var(--space-4);
  border-radius: var(--button-border-radius);
  
  /* Accesibilidad */
  &:focus-visible {
    outline: 3px solid var(--color-blue-300);
    outline-offset: 2px;
  }
  
  /* Alto contraste para usuarios con baja visión */
  @media (forced-colors: active) {
    border: 1px solid ButtonText;
  }
`;

// Componente con soporte para teclado y ARIA
const DropdownMenu = ({ label, children }) => {
  const [isOpen, setIsOpen] = useState(false);
  const menuRef = useRef(null);
  
  return (
    <div>
      <button 
        aria-haspopup="true"
        aria-expanded={isOpen}
        onClick={() => setIsOpen(!isOpen)}
      >
        {label}
      </button>
      
      {isOpen && (
        <ul 
          ref={menuRef}
          role="menu"
          tabIndex={-1}
        >
          {React.Children.map(children, (child, index) => (
            <li role="menuitem" tabIndex={0}>
              {child}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

### 5.5.2 Rendimiento

Los sistemas de diseño deben optimizarse para garantizar que las aplicaciones resultantes sean eficientes.

```javascript
// Ejemplo: Optimización de bundle con tree-shaking
import { Button } from './components'; // Importa solo lo necesario

// En lugar de:
// import * as Components from './components'; // Importa todo
```

### 5.5.3 Versionado y Cambios

La gestión de versiones es crucial para mantener la estabilidad mientras se evoluciona el sistema.

```json
// package.json
{
  "name": "mi-design-system",
  "version": "2.3.1",
  "description": "Sistema de diseño corporativo",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "lint": "eslint src --ext .ts,.tsx",
    "storybook": "start-storybook -p 6006",
    "build-storybook": "build-storybook",
    "release": "standard-version"
  },
  "peerDependencies": {
    "react": "^17.0.0 || ^18.0.0",
    "react-dom": "^17.0.0 || ^18.0.0"
  },
  // ...otras dependencias
}
```

### 5.5.4 Documentación Efectiva

Una documentación clara y completa es esencial para la adopción del sistema de diseño.

```jsx
// Ejemplo de documentación para un componente
/**
 * Avatar component displays a user's profile picture or their initials if no image is available.
 * 
 * @component
 * @example
 * ```jsx
 * <Avatar
 *   src="https://example.com/avatar.jpg"
 *   alt="Jane Smith"
 *   size="medium"
 * />
 * ```
 */
export interface AvatarProps {
  /**
   * URL de la imagen del avatar
   */
  src?: string;
  
  /**
   * Texto alternativo para accesibilidad
   */
  alt: string;
  
  /**
   * Tamaño del avatar
   * @default "medium"
   */
  size?: 'small' | 'medium' | 'large';
  
  /**
   * Si el avatar debe mostrar un indicador de presencia
   * @default false
   */
  showPresence?: boolean;
  
  /**
   * Estado de presencia del usuario
   * @default "offline"
   */
  presenceStatus?: 'online' | 'busy' | 'away' | 'offline';
}

const Avatar = ({ 
  src, 
  alt,
  size = 'medium',
  showPresence = false,
  presenceStatus = 'offline'
}: AvatarProps) => {
  // Implementación
};
```

## 5.6 Accesibilidad (A11y) en Sistemas de Diseño

La accesibilidad digital o A11y (abreviatura que representa la primera letra, seguida de 11 letras intermedias y la última letra de "accessibility") es un componente fundamental de cualquier sistema de diseño moderno. Un sistema verdaderamente eficiente debe ser utilizable por todas las personas, independientemente de sus capacidades o contexto de uso.

### 5.6.1 Principios de Accesibilidad WCAG

Los Sistemas de Diseño deben adherirse a las Pautas de Accesibilidad para el Contenido Web (WCAG), organizadas en cuatro principios fundamentales:

1. **Perceptible**: La información debe ser presentable a los usuarios de manera que puedan percibirla.
2. **Operable**: Los componentes de la interfaz deben ser operables por todos los usuarios.
3. **Comprensible**: La información y el manejo de la interfaz deben ser comprensibles.
4. **Robusto**: El contenido debe ser suficientemente robusto para funcionar con tecnologías actuales y futuras.

### 5.6.2 Implementación de A11y en Sistemas de Diseño

```tsx
// Ejemplo: Componente de botón accesible
import React, { forwardRef } from 'react';
import styled from 'styled-components';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'tertiary';
  size?: 'small' | 'medium' | 'large';
  isLoading?: boolean;
}

const StyledButton = styled.button<ButtonProps>`
  /* Estilos base */
  cursor: pointer;
  border: none;
  font-family: var(--font-family);
  font-weight: var(--font-weight-medium);
  border-radius: var(--border-radius-md);
  transition: all 0.2s ease-in-out;
  position: relative;
  
  /* Estados de foco visibles para navegación por teclado */
  &:focus-visible {
    outline: 3px solid var(--color-focus-ring);
    outline-offset: 2px;
  }
  
  /* Alto ratio de contraste para visibilidad */
  &.primary {
    background-color: var(--color-primary);
    color: white;
    /* Aseguramos contraste mínimo de 4.5:1 para texto normal */
  }
  
  /* Estados de hover/foco para personas que usan ratones y teclados */
  &:hover:not(:disabled),
  &:focus:not(:disabled) {
    background-color: var(--color-primary-dark);
  }
  
  /* Indicador claro para estado deshabilitado */
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
  
  /* Soporte para usuarios con modo de alto contraste en sistema operativo */
  @media (forced-colors: active) {
    border: 1px solid ButtonText;
  }
`;

export const Button = forwardRef<HTMLButtonElement, ButtonProps>((
  {
    children,
    variant = 'primary',
    size = 'medium',
    isLoading = false,
    disabled,
    type = 'button',
    className,
    ...props
  },
  ref
) => {
  // Merge classes para mantener las clases pasadas por props
  const buttonClass = `${className || ''} ${variant}`;
  
  return (
    <StyledButton
      ref={ref}
      className={buttonClass}
      size={size}
      disabled={disabled || isLoading}
      type={type}
      variant={variant}
      aria-busy={isLoading ? 'true' : 'false'}
      {...props}
    >
      {isLoading ? (
        <>
          <span className="loading-spinner" aria-hidden="true" />
          <span className="visually-hidden">Cargando...</span>
          {children}
        </>
      ) : (
        children
      )}
    </StyledButton>
  );
});

Button.displayName = 'Button';
```

### 5.6.3 Lista de Verificación de Accesibilidad para Componentes

#### Estructura Semántica

- Usar elementos HTML semánticos (`button`, `a`, `nav`, `header`, etc.)
- Implementar correctamente el flujo de encabezados (`h1` - `h6`)
- Asegurar una estructura lógica para lectores de pantalla

#### Navegación y Operabilidad

- Todos los elementos interactivos deben ser accesibles por teclado
- Proporcionar indicadores visibles de foco
- Implementar atajos de teclado cuando sea apropiado
- Orden de tabulación lógico (`tabindex`)

```tsx
// Ejemplo: Menú desplegable accesible
import React, { useState, useRef, useEffect } from 'react';

export const Dropdown = ({ label, children }) => {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);
  const buttonRef = useRef<HTMLButtonElement>(null);
  const menuRef = useRef<HTMLUListElement>(null);
  
  // Cerrar al hacer clic fuera
  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    };
    
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);
  
  // Manejo de navegación por teclado
  const handleKeyDown = (event: React.KeyboardEvent) => {
    if (!isOpen) {
      if (event.key === 'Enter' || event.key === ' ' || event.key === 'ArrowDown') {
        event.preventDefault();
        setIsOpen(true);
        // Enfocar primer elemento del menú una vez abierto
        setTimeout(() => {
          const items = menuRef.current?.querySelectorAll('[role="menuitem"]');
          if (items?.length) (items[0] as HTMLElement).focus();
        }, 10);
      }
    } else {
      if (event.key === 'Escape') {
        setIsOpen(false);
        buttonRef.current?.focus();
      }
    }
  };
  
  return (
    <div ref={dropdownRef} className="dropdown-container">
      <button
        ref={buttonRef}
        aria-haspopup="true"
        aria-expanded={isOpen}
        aria-controls="dropdown-menu"
        onClick={() => setIsOpen(!isOpen)}
        onKeyDown={handleKeyDown}
      >
        {label}
        <span aria-hidden="true" className="dropdown-arrow">
          &#9660;
        </span>
      </button>
      
      {isOpen && (
        <ul
          id="dropdown-menu"
          ref={menuRef}
          role="menu"
          className="dropdown-menu"
          aria-labelledby="dropdown-button"
        >
          {React.Children.map(children, (child, index) => (
            <li
              role="menuitem"
              tabIndex={0}
              onKeyDown={(e) => {
                if (e.key === 'Enter' || e.key === ' ') {
                  e.preventDefault();
                  (e.target as HTMLElement).click();
                } else if (e.key === 'ArrowDown') {
                  e.preventDefault();
                  const items = menuRef.current?.querySelectorAll('[role="menuitem"]');
                  const nextIndex = (index + 1) % (items?.length || 1);
                  (items?.[nextIndex] as HTMLElement)?.focus();
                } else if (e.key === 'ArrowUp') {
                  e.preventDefault();
                  const items = menuRef.current?.querySelectorAll('[role="menuitem"]');
                  const total = items?.length || 1;
                  const nextIndex = (index - 1 + total) % total;
                  (items?.[nextIndex] as HTMLElement)?.focus();
                }
              }}
            >
              {child}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

#### Texto y Legibilidad

- Usar suficiente contraste de color (mínimo 4.5:1 para texto normal, 3:1 para texto grande)
- Permitir zoom y escalado de texto sin pérdida de funcionalidad
- Evitar texto en imágenes
- Implementar clases de utilidad para texto oculto visualmente pero disponible para lectores de pantalla

```css
/* Clase de utilidad para texto accesible solo para lectores de pantalla */
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

#### Imágenes y Multimedia

- Proporcionar texto alternativo (`alt`) para todas las imágenes
- Incluir subtítulos y transcripciones para contenido de audio y video
- Evitar contenido con destellos que puedan causar convulsiones

#### Formularios

- Asociar etiquetas (`label`) con campos de entrada
- Agrupar campos relacionados con `fieldset` y `legend`
- Proporcionar mensajes de error claros y accesibles
- Validar formularios tanto del lado del cliente como del servidor

```tsx
// Ejemplo: Campo de formulario accesible
const FormField = ({ id, label, error, ...props }) => (
  <div className="form-field">
    <label htmlFor={id}>{label}</label>
    <input 
      id={id} 
      aria-invalid={!!error} 
      aria-describedby={error ? `${id}-error` : undefined} 
      {...props} 
    />
    {error && (
      <div id={`${id}-error`} className="error-message" role="alert">
        {error}
      </div>
    )}
  </div>
);
```

### 5.6.4 Herramientas de Prueba de Accesibilidad

- **Auditorías manuales**: Navegación por teclado, pruebas con lectores de pantalla (NVDA, JAWS, VoiceOver)
- **Herramientas automatizadas**: 
  - Lighthouse (Chrome DevTools)
  - axe-core y sus plugins para navegadores
  - Pa11y para pruebas de CI/CD

```javascript
// Ejemplo: Prueba de accesibilidad automatizada con Jest y axe-core
import React from 'react';
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { Button } from './Button';

expect.extend(toHaveNoViolations);

describe('Button component', () => {
  it('should not have accessibility violations', async () => {
    const { container } = render(
      <Button onClick={() => console.log('clicked')}>Click me</Button>
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

### 5.6.5 Integración de A11y en el Flujo de Trabajo

- Incluir requisitos de accesibilidad en los criterios de aceptación
- Incorporar pruebas de accesibilidad en pipelines de CI/CD
- Realizar revisiones de accesibilidad durante las revisiones de código
- Capacitar a los equipos en buenas prácticas de accesibilidad

```yaml
# Ejemplo: Configuración de GitHub Actions para pruebas de accesibilidad
name: Accessibility Tests

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Install dependencies
        run: npm ci
      - name: Build project
        run: npm run build
      - name: Run accessibility tests
        run: npm run test:a11y
```

## 5.7 Storybook como Herramienta para Design Systems

Storybook es una herramienta de código abierto para desarrollar componentes de UI de forma aislada. Es ideal para sistemas de diseño porque facilita la documentación, pruebas y desarrollo de componentes.

### 5.7.1 Configuración Básica de Storybook

```bash
# Instalar Storybook en un proyecto existente
npx storybook@latest init
```

```javascript
// .storybook/main.js - Configuración principal
module.exports = {
  stories: [
    '../src/**/*.stories.mdx',
    '../src/**/*.stories.@(js|jsx|ts|tsx)'
  ],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',  // Addon para accesibilidad
  ],
  framework: {
    name: '@storybook/react-webpack5',
    options: {}
  },
  docs: {
    autodocs: true
  }
};
```

```javascript
// .storybook/preview.js - Configuración de previsualización
import '../src/styles/globals.css';
import { ThemeProvider } from '../src/contexts/ThemeContext';

// Parámetros globales para todas las historias
export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
  backgrounds: {
    default: 'light',
    values: [
      { name: 'light', value: '#FFFFFF' },
      { name: 'dark', value: '#1D2939' },
    ],
  },
  // Configuración del addon de accesibilidad
  a11y: {
    element: '#root',
    manual: false,
    config: {
      rules: [
        {
          id: 'color-contrast',
          reviewOnFail: true,
        },
      ],
    },
  },
};

// Decorador global para envolver todas las historias con el ThemeProvider
export const decorators = [
  (Story) => (
    <ThemeProvider>
      <div style={{ margin: '2rem' }}>
        <Story />
      </div>
    </ThemeProvider>
  ),
];
```

### 5.7.2 Estructura de un Story File

```tsx
// src/components/Button/Button.stories.tsx
import React from 'react';
import { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Design System/Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['primary', 'secondary', 'tertiary'],
      description: 'Estilo visual del botón'
    },
    size: {
      control: { type: 'select' },
      options: ['small', 'medium', 'large'],
      description: 'Tamaño del botón'
    },
    disabled: {
      control: 'boolean',
      description: 'Estado deshabilitado del botón'
    },
    onClick: { action: 'clicked' }
  },
  parameters: {
    componentSubtitle: 'Botón interactivo para acciones principales y secundarias',
    docs: {
      description: {
        component: 'Los botones permiten a los usuarios ejecutar acciones con un solo clic.'
      }
    },
    // Configuración de diseño responsivo
    viewport: {
      defaultViewport: 'responsive'
    },
    // Configuración de accesibilidad para este componente
    a11y: {
      config: {
        rules: [
          {
            id: 'button-name',
            reviewOnFail: true,
          },
        ],
      },
    },
  },
  // Valores predeterminados para todos los Stories
  args: {
    children: 'Botón',
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    variant: 'primary',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
  },
};

export const Tertiary: Story = {
  args: {
    variant: 'tertiary',
  },
};

export const Small: Story = {
  args: {
    size: 'small',
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
  },
};

export const WithIcon: Story = {
  args: {
    children: (
      <>
        <span className="icon" aria-hidden="true">+</span>
        Añadir
      </>
    ),
  },
};

// Historia que muestra un estado de error de accesibilidad
export const NoText: Story = {
  args: {
    children: null,
    'aria-label': undefined, // Omitir intencionalmente para mostrar error a11y
  },
  parameters: {
    docs: {
      description: {
        story: 'Este ejemplo muestra un error de accesibilidad: botón sin texto ni aria-label.'
      }
    },
  },
};
```

### 5.7.3 Documentación Avanzada con MDX

```jsx
// src/components/Button/Button.stories.mdx
import { Canvas, Meta, Story, ArgsTable } from '@storybook/addon-docs';
import { Button } from './Button';

<Meta 
  title="Design System/Components/Button/Docs" 
  component={Button} 
/>

# Botón

El componente Button es fundamental en nuestro sistema de diseño y se utiliza para acciones primarias y secundarias.

## Uso

Importe el componente Button y utilícelo con las props necesarias:

```jsx
import { Button } from './components/Button';

<Button variant="primary" onClick={() => console.log('Clicked!')}>Click me</Button>
```

## Propiedades

<ArgsTable of={Button} />

## Ejemplos

### Variantes

<Canvas>
  <Story id="design-system-components-button--primary" />
  <Story id="design-system-components-button--secondary" />
  <Story id="design-system-components-button--tertiary" />
</Canvas>

### Tamaños

<Canvas>
  <Story id="design-system-components-button--small" />
  <Button>Medium (Default)</Button>
  <Button size="large">Large</Button>
</Canvas>

### Estados

<Canvas>
  <Button>Habilitado</Button>
  <Story id="design-system-components-button--disabled" />
  <Button variant="primary" isLoading>Cargando</Button>
</Canvas>

## Accesibilidad

El componente Button está diseñado siguiendo las mejores prácticas de accesibilidad:

- Utiliza el elemento HTML nativo `<button>` para funcionalidad completa de teclado
- Implementa estados de foco visibles para navegación por teclado
- Incluye estados apropiados de aria-disabled y aria-busy
- Mantiene un ratio de contraste adecuado para todos los estados y variantes
```

### 5.7.4 Addons Útiles para Design Systems

- **@storybook/addon-a11y**: Pruebas de accesibilidad
- **@storybook/addon-designs**: Integrar diseños de Figma/Sketch
- **@storybook/addon-controls**: Interactuar con props de componentes
- **@storybook/addon-viewport**: Probar componentes en diferentes tamaños de pantalla
- **@storybook/addon-docs**: Documentación autogenerada
- **storybook-addon-themes**: Alternar entre temas claros y oscuros

```javascript
// Ejemplo de instalación de addons
npm install --save-dev @storybook/addon-a11y @storybook/addon-designs storybook-addon-themes
```

### 5.7.5 Pruebas con Storybook

```javascript
// src/components/Button/Button.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import { composeStories } from '@storybook/testing-react';
import * as stories from './Button.stories';

// Componer stories para pruebas
const { Primary, Disabled } = composeStories(stories);

describe('Button', () => {
  it('llama a onClick cuando se hace clic', () => {
    const onClickMock = jest.fn();
    const { getByRole } = render(<Primary onClick={onClickMock} />);
    
    fireEvent.click(getByRole('button'));
    expect(onClickMock).toHaveBeenCalledTimes(1);
  });
  
  it('no llama a onClick cuando está deshabilitado', () => {
    const onClickMock = jest.fn();
    const { getByRole } = render(<Disabled onClick={onClickMock} />);
    
    fireEvent.click(getByRole('button'));
    expect(onClickMock).not.toHaveBeenCalled();
  });
});
```

### 5.7.6 Organización para Design Systems

La estructura de un sistema de diseño en Storybook generalmente sigue un patrón jerárquico:

```
src/
├── foundations/          # Elementos fundamentales del sistema
│   ├── colors/
│   │   ├── Colors.stories.tsx
│   │   └── Colors.mdx
│   ├── typography/
│   │   └── Typography.stories.mdx
│   └── spacing/
│       └── Spacing.stories.mdx
├── components/           # Componentes atómicos individuales
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.stories.tsx
│   │   └── Button.test.tsx
│   └── Input/
│       ├── Input.tsx
│       └── Input.stories.tsx
└── patterns/            # Combinaciones de componentes
    └── Form/
        ├── Form.tsx
        └── Form.stories.tsx
```

### 5.7.7 Integración con Figma y Herramientas de Diseño

```tsx
// src/components/Button/Button.stories.tsx (fragmento)
import { Meta } from '@storybook/react';
import { Button } from './Button';

export default {
  title: 'Design System/Components/Button',
  component: Button,
  parameters: {
    design: {
      type: 'figma',
      url: 'https://www.figma.com/file/LKQ4FJ4bTnCSjedbRpk931/Sample-File?node-id=0%3A1',
    },
  },
} as Meta;
```

### 5.7.8 Publicación del Design System con Storybook

```bash
# Construir Storybook para producción
npm run build-storybook

# Desplegar en GitHub Pages con storybook-deployer
npm i --save-dev @storybook/storybook-deployer
```

En package.json:
```json
{
  "scripts": {
    "deploy-storybook": "storybook-to-ghpages"
  }
}
```

```yaml
# Ejemplo de workflow de GitHub Actions para publicación automática
name: Deploy Storybook

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Install dependencies
        run: npm ci
      - name: Build Storybook
        run: npm run build-storybook
      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@4.1.4
        with:
          branch: gh-pages
          folder: storybook-static
```

## 5.8 Conclusiones

Un sistema de diseño bien implementado proporciona consistencia, eficiencia y escalabilidad. No es solo una biblioteca de componentes, sino un conjunto completo de principios, patrones y prácticas que guían el desarrollo de productos digitales. La inversión en un sistema de diseño robusto genera retornos significativos a largo plazo, especialmente para organizaciones con múltiples productos o equipos grandes.

El éxito de un sistema de diseño depende tanto de factores técnicos como culturales. Es crucial obtener el apoyo de los interesados, capacitar a los equipos y establecer procesos claros para su mantenimiento y evolución. Con el enfoque adecuado, un sistema de diseño puede convertirse en un activo estratégico que impulsa la innovación y la calidad del producto.
