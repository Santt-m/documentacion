# Capítulo 5: Sistemas de Diseño (Design Systems)

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
```

### 5.3.3 Documentación de Componentes con Storybook

```jsx
// Button.stories.tsx
import { Meta, Story } from '@storybook/react';
import { Button, ButtonProps } from './Button';

export default {
  title: 'Design System/Components/Button',
  component: Button,
  argTypes: {
    variant: { 
      control: 'select', 
      options: ['primary', 'secondary', 'tertiary', 'danger'],
    },
    size: { 
      control: 'select', 
      options: ['small', 'medium', 'large'],
    },
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
