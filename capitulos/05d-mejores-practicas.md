# Capítulo 5: Sistemas de Diseño (Design Systems)

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
    "prepublishOnly": "npm run build && npm run test"
  },
  "peerDependencies": {
    "react": "^17.0.0",
    "react-dom": "^17.0.0"
  }
}
```

### 5.5.4 Documentación y Guías de Contribución

Una buena documentación facilita la adopción del sistema de diseño y mantiene la coherencia.

```jsx
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
}
```

### 5.5.5 Internacionalización (i18n)

Considerar el soporte multilenguaje desde el inicio.

```jsx
// Ejemplo: Componente con soporte para diferentes direcciones de texto
const CardComponent = styled.div`
  padding: 1rem;
  text-align: ${props => props.theme.direction === 'rtl' ? 'right' : 'left'};
  margin-left: ${props => props.theme.direction === 'rtl' ? 'auto' : '0'};
  margin-right: ${props => props.theme.direction === 'rtl' ? '0' : 'auto'};
`;
```

### 5.5.6 Accesibilidad Avanzada

Los Sistemas de Diseño deben adherirse a las Pautas de Accesibilidad para el Contenido Web (WCAG), organizadas en cuatro principios fundamentales:

1. **Perceptible**: La información debe ser presentable a los usuarios de manera que puedan percibirla.
2. **Operable**: Los componentes de la interfaz deben ser operables por todos los usuarios.
3. **Comprensible**: La información y el manejo de la interfaz deben ser comprensibles.
4. **Robusto**: El contenido debe ser lo suficientemente robusto para funcionar con tecnologías actuales y futuras.

```tsx
// Implementación de un botón accesible con React
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
  transition: all 0.2s;
  position: relative;
  
  /* Asegurarse que el botón es operable */
  min-height: 44px; /* WCAG 2.5.5 Target Size */
  
  /* Asegurarse que el foco es claramente visible */
  &:focus-visible {
    outline: 2px solid var(--color-focus-ring);
    outline-offset: 2px;
  }
  
  /* Asegurarse que el hover tiene suficiente contraste */
  &:hover:not(:disabled) {
    filter: brightness(0.9);
  }
  
  /* Estado de carga con indicador para usuarios de lectores de pantalla */
  ${props => props.isLoading && `
    &:before {
      content: '';
      position: absolute;
      /* Spinner styles */
    }
    
    &:after {
      content: 'Cargando...';
      position: absolute;
      width: 1px;
      height: 1px;
      padding: 0;
      margin: -1px;
      overflow: hidden;
      clip: rect(0, 0, 0, 0);
      white-space: nowrap;
      border-width: 0;
    }
  `}
`;

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  (
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
      variant={variant}
      disabled={disabled || isLoading}
      aria-busy={isLoading}
      type={type}
      {...props}
    >
      {children}
    </StyledButton>
  );
});

Button.displayName = 'Button';
```

#### Navegación con Teclado

Elementos importantes para soportar navegación con teclado:

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
  
  // Cerrar dropdown cuando se hace click fuera
  useEffect(() => {
    function handleClickOutside(event: MouseEvent) {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    }
    
    document.addEventListener('mousedown', handleClickOutside);
    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, []);
  
  // Manejar navegación con teclado
  const handleKeyDown = (event: React.KeyboardEvent) => {
    if (!isOpen) {
      if (event.key === 'ArrowDown' || event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        setIsOpen(true);
        
        // Focus primer elemento después de abrir
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
              className="dropdown-item"
              onKeyDown={(event) => {
                const items = menuRef.current?.querySelectorAll('[role="menuitem"]');
                const total = items?.length || 0;
                
                if (event.key === 'ArrowDown') {
                  event.preventDefault();
                  const nextIndex = (index + 1) % total;
                  (items?.[nextIndex] as HTMLElement)?.focus();
                } else if (event.key === 'ArrowUp') {
                  event.preventDefault();
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

```css
/* Ejemplo: Tipografía accesible */
:root {
  --font-size-base: 1rem; /* 16px por defecto, respeta las preferencias del usuario */
  --line-height-base: 1.5;
  --font-family-base: system-ui, -apple-system, sans-serif;
}

body {
  font-family: var(--font-family-base);
  font-size: var(--font-size-base);
  line-height: var(--line-height-base);
  color: var(--color-text); /* Asegurarse que tiene suficiente contraste */
}

/* Usar em/rem para permitir escalado */
h1 { font-size: 2rem; }
h2 { font-size: 1.5rem; }

/* Soportar modo oscuro o preferencias de contraste */
@media (prefers-color-scheme: dark) {
  :root {
    --color-background: #121212;
    --color-text: #FFFFFF;
  }
}

/* Soportar animaciones reducidas */
@media (prefers-reduced-motion) {
  * {
    animation: none !important;
    transition: none !important;
  }
}
```

#### Formularios Accesibles

- Asociar labels con campos de formulario
- Incluir instrucciones claras
- Proporcionar feedback de errores
- Agrupar campos relacionados con `fieldset` y `legend`
- Proporcionar mensajes de error claros y accesibles
- Validar formularios tanto del lado del cliente como del servidor

```tsx
// Ejemplo: Campo de formulario accesible
import React, { useState } from 'react';

export const TextField = ({ 
  id, 
  label, 
  required, 
  helperText,
  errorText,
  ...props 
}) => {
  const [value, setValue] = useState('');
  const [touched, setTouched] = useState(false);
  const hasError = touched && errorText;
  
  const describedBy = [];
  if (helperText) describedBy.push(`${id}-helper`);
  if (hasError) describedBy.push(`${id}-error`);
  
  return (
    <div className="form-field">
      <label htmlFor={id}>
        {label}
        {required && <span aria-hidden="true"> *</span>}
        {required && <span className="sr-only"> (obligatorio)</span>}
      </label>
      
      {helperText && (
        <div id={`${id}-helper`} className="helper-text">
          {helperText}
        </div>
      )}
      
      <input
        id={id}
        value={value}
        onChange={(e) => setValue(e.target.value)}
        onBlur={() => setTouched(true)}
        aria-invalid={!!hasError}
        aria-required={required}
        aria-describedby={describedBy.length ? describedBy.join(' ') : undefined}
        {...props}
      />
      
      {hasError && (
        <div id={`${id}-error`} className="error-text" role="alert">
          {errorText}
        </div>
      )}
    </div>
  );
};
```
