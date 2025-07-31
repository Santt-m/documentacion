# Capítulo 5: Sistemas de Diseño (Design Systems)

## 5.6 Creación y Mantenimiento de Documentación

### 5.6.1 Herramientas para Documentación

Para una documentación efectiva de un sistema de diseño, se pueden utilizar diversas herramientas:

- **Storybook**: Herramienta principal para documentar componentes con ejemplos interactivos.
- **Docusaurus**: Generador de sitios de documentación basado en React y Markdown.
- **Zeroheight**: Plataforma especializada para documentación de sistemas de diseño.
- **Notion**: Para documentación interna y colaboración en equipo.
- **Figma**: Para documentación visual y alineación con el diseño.
- **MDX**: Combinación de Markdown y JSX para documentación interactiva.

### 5.6.2 Estructura de una Documentación Efectiva

Una documentación completa de un sistema de diseño debería incluir:

1. **Introducción y filosofía**
   - Principios de diseño
   - Visión y objetivos
   - Cómo contribuir

2. **Fundamentos**
   - Tokens de diseño
   - Guía de estilo
   - Tipografía
   - Espaciado
   - Colores

3. **Componentes**
   - Uso y propósito
   - API y propiedades
   - Estados y variantes
   - Buenas prácticas y anti-patrones
   - Ejemplos de código

4. **Patrones y composiciones**
   - Combinaciones comunes de componentes
   - Soluciones a problemas específicos

5. **Guías de uso**
   - Para diseñadores
   - Para desarrolladores
   - Accesibilidad
   - Rendimiento

### 5.6.3 Mantenimiento a Largo Plazo

Para mantener la documentación actualizada:

- Establecer un proceso de revisión y actualización
- Automatizar la sincronización entre código y documentación
- Designar responsables de cada sección
- Recopilar feedback de los usuarios de la documentación
- Establecer KPIs para medir la calidad de la documentación

## 5.7 Storybook como Herramienta Central

### 5.7.1 Configuración Básica

```bash
# Crear un nuevo proyecto de Storybook
npx sb init

# Iniciar el servidor de desarrollo
npm run storybook
```

```javascript
// .storybook/main.js
module.exports = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y'
  ],
  framework: '@storybook/react',
  core: {
    builder: '@storybook/builder-webpack5',
  },
};
```

```javascript
// .storybook/preview.js
import { themes } from '@storybook/theming';
import { ThemeProvider } from 'styled-components';
import { myTheme } from '../src/theme';
import '../src/styles/global.css';

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
  docs: {
    theme: themes.light,
  },
};

export const decorators = [
  (Story) => (
    <ThemeProvider theme={myTheme}>
      <Story />
    </ThemeProvider>
  ),
];
```

### 5.7.2 Creación de Historias

```jsx
// Button.stories.jsx
import { Button } from './Button';

export default {
  title: 'Design System/Components/Button',
  component: Button,
  argTypes: {
    variant: {
      options: ['primary', 'secondary', 'tertiary'],
      control: { type: 'select' },
    },
    size: {
      options: ['small', 'medium', 'large'],
      control: { type: 'radio' },
    },
    onClick: { action: 'clicked' },
  },
};

// Template básico
const Template = (args) => <Button {...args} />;

// Historia primaria
export const Primary = Template.bind({});
Primary.args = {
  variant: 'primary',
  children: 'Button',
};

// Variaciones
export const Secondary = Template.bind({});
Secondary.args = {
  ...Primary.args,
  variant: 'secondary',
  children: 'Secondary Button',
};

export const Large = Template.bind({});
Large.args = {
  ...Primary.args,
  size: 'large',
};

export const Small = Template.bind({});
Small.args = {
  ...Primary.args,
  size: 'small',
};
```

### 5.7.3 Documentación Avanzada con MDX

```mdx
import { Meta, Story, Canvas, ArgsTable } from '@storybook/addon-docs';
import { Button } from './Button';

<Meta title="Design System/Components/Button" component={Button} />

# Button

Buttons are used to initialize an action.

## Usage

Use buttons to trigger actions or navigation.

<Canvas>
  <Story name="Primary">
    <Button variant="primary">Primary Button</Button>
  </Story>
</Canvas>

## API

<ArgsTable of={Button} />

## Variants

<Canvas>
  <Story name="Variants">
    <div style={{ display: 'flex', gap: '8px' }}>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="tertiary">Tertiary</Button>
      <Button variant="danger">Danger</Button>
    </div>
  </Story>
</Canvas>

## Sizes

<Canvas>
  <Story name="Sizes">
    <div style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
      <Button size="small">Small</Button>
      <Button size="medium">Medium</Button>
      <Button size="large">Large</Button>
    </div>
  </Story>
</Canvas>

## States

<Canvas>
  <Story name="States">
    <div style={{ display: 'flex', gap: '8px' }}>
      <Button>Default</Button>
      <Button disabled>Disabled</Button>
    </div>
  </Story>
</Canvas>

## Accessibility

The Button component follows WAI-ARIA best practices:

- Uses native button element for keyboard accessibility
- Provides visual feedback for focus state
- Ensures proper contrast ratios
- Supports screen readers through appropriate labels
```

### 5.7.4 Addons Esenciales

- **@storybook/addon-essentials**: Incluye Controls, Actions, Docs, Viewport, Backgrounds, y Toolbars.
- **@storybook/addon-a11y**: Para pruebas de accesibilidad.
- **@storybook/addon-links**: Para crear enlaces entre historias.
- **@storybook/addon-interactions**: Para pruebas de interacción.
- **@storybook/addon-viewport**: Para probar en diferentes tamaños de pantalla.

```javascript
// .storybook/main.js
module.exports = {
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y',
    '@storybook/addon-links',
    '@storybook/addon-interactions',
    'storybook-addon-designs',
    'storybook-dark-mode',
    'storybook-mobile',
  ],
};
```

### 5.7.5 Pruebas con Storybook

```javascript
// Button.test.js
import { render, screen, fireEvent } from '@testing-library/react';
import { composeStories } from '@storybook/testing-react';
import * as stories from './Button.stories';

// Compone todas las historias para usarlas en pruebas
const { Primary, Disabled } = composeStories(stories);

describe('Button', () => {
  test('renders primary button', () => {
    render(<Primary />);
    const buttonElement = screen.getByText(/button/i);
    expect(buttonElement).toBeInTheDocument();
    expect(buttonElement).toHaveClass('btn-primary');
  });

  test('disabled button cannot be clicked', () => {
    const onClick = jest.fn();
    render(<Disabled onClick={onClick} />);
    const buttonElement = screen.getByText(/disabled/i);
    
    fireEvent.click(buttonElement);
    expect(onClick).not.toHaveBeenCalled();
    expect(buttonElement).toBeDisabled();
  });
});
```

### 5.7.6 Organización para Design Systems

Estructura recomendada de archivos:

```
design-system/
├── .storybook/          # Configuración de Storybook
├── src/
│   ├── tokens/          # Design tokens
│   │   ├── colors/
│   │   │   └── Colors.stories.mdx
│   │   ├── typography/
│   │   │   └── Typography.stories.mdx
│   │   └── spacing/
│   │       └── Spacing.stories.mdx
│   ├── components/      # Componentes atómicos individuales
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── Button.test.tsx
│   │   └── Input/
│   │       ├── Input.tsx
│   │       └── Input.stories.tsx
│   └── patterns/        # Combinaciones de componentes
│       └── Form/
│           ├── Form.tsx
│           └── Form.stories.tsx
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
