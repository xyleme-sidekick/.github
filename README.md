
# Module Federation Integration & Library Usage Documentation (Vite-based)

## 1. Module Federation Overview

Module Federation allows separately built applications to share code and load remote modules at runtime. With Vite, this is enabled via [`vite-plugin-federation`](https://github.com/originjs/vite-plugin-federation), which brings Webpack-style federation capabilities to Vite-based projects.

### Key Concepts:
- **Host App**: Loads components or modules from remote sources.
- **Remote App** (our library): Exposes specific components, hooks, or utilities to the host.
- **Shared Dependencies**: Libraries like React or Vue are shared between apps to avoid duplication.

---

## 2. Build Process Requirements

### Prerequisites
- Vite v4+ installed in both host and remote projects
- `vite-plugin-federation` installed

### Remote (Customer Library) Configuration
In `vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import federation from '@originjs/vite-plugin-federation'

export default defineConfig({
  plugins: [
    federation({
      name: 'kicks',
      filename: 'kicks.js',
      exposes: {
        './Kicks': './src/components/sampleKick.tsx',
      },
      shared: ['react', 'react-dom']
    })
  ],
  build: {
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  }
})
```

### Build Process
To prepare the remote module for deployment, run the following command in the remote project:

```bash
  vite build
```

This generates the production-ready files in the `dist/` directory. The contents of the `dist/` folder include:

- `kicks.js`
- Federated JavaScript chunks
- Static assets (e.g., styles, images)

---

## 3. Deployment Details

### What You Deploy:
- `dist/` directory (the module federation entry point and all dependent JS/CSS chunks)

### Where It’s Deployed:
- Public CDN or static hosting (e.g., AWS S3 + CloudFront)
- Example path:
  ```
  https://cdn.customer.com/assets/kicks.js
  ```
---

## 4. What the Customer Provides

To successfully consume your library we need to have:
- a public URL to your deployed federated module (for example: https://customer.com/assets/kicks.js). This URL will be used to dynamically load your components into the Create application.

## 5. How to Create Your Library

This section describes the steps the customer needs to follow to create a remote library that can be consumed via Module Federation using Vite.

### Step 1: Initialize Your Project

If you don't have a Vite project yet, create one:

```bash
  npm create vite@latest my-lib --template react-ts
  cd my-lib
  npm install
```

Install required dependencies:
```bash
  npm install @originjs/vite-plugin-federation --save-dev
```

### Step 2: Define the Required Structure for Your Library

To be compatible with the host application, your library must export a function named registerKicks with the following signature:

```ts
interface LibraryReturnType {
  registerKicks: () => RegisteredKicks
}
```

This function should return a RegisteredKicks object containing metadata and React components to be rendered.
```ts
import { FC } from 'react'

export interface RegisteredKicks {
  kicks: RegisteredKick[];
}

export interface RegisteredKick {
  component: FC<KickProps>;
  description: string;
  id: string;
  name: string;
  position: number;
}
```
You may return several Kicks at once. Each Kick should have a React component and additional information like
name, description, id, and position in the Sidekick.

A React component that handles kick-related functionality should work the following props:

```ts
export interface KickProps {
  onCloseKick?: () => void;
  onReady: (result: RegisterKickResult) => void;
  theme: Theme;
  xyFetch: XyFetch;
}

export type SidekickExecuteFuncType = (items: SidekickItem[]) => void;
export type SidekickCanExecuteFuncType = (items: SidekickItem[]) => boolean;

export interface RegisterKickResult {
  canExecute?: SidekickCanExecuteFuncType;
  execute: SidekickExecuteFuncType;
}
```

### Props

| Prop        | Type                                 | Required | Description                                                                   |
|-------------|--------------------------------------|----------|-------------------------------------------------------------------------------| 
| onCloseKick | () => void                           | No       | Callback function invoked when the Kick is closed or cancelled                |
| onReady     | (result: RegisterKickResult) => void | Yes      | Callback function invoked when the Kick is ready to use, returning the result |
| theme       | Theme                                | Yes      | The MUI Theme object used to style the MUI components                         |
| xyFetch     | XyFetch                              | Yes      | The fetch utility used for making API requests to Create API                  |

Usage Example

```tsx
import {useCallback, useEffect} from 'react';
import { Kick } from './Kick';
import { Theme, XyFetch } from 'your-library';
import type { RegisterKickResult } from 'your-types';

const KickComponent: FC<KickProps> = ({onReady, theme, xyFetch, onCloseKick}) => {
  const handleExecute = useCallback((items: SidekickItems[]) => {
    console.log('Kick executed:', items);
  }, []);

  const handleCloseKick = useCallback(() => {
    onCloseKick?.();
  }, [onCloseKick]);
  
  useEffect(() => {
    onReady({
      execute: handleExecute
    })
  }, [handleExecute]);

  return (
    <div>Kick Sample</div>
  );
};
```

`onReady` should be called once your component is ready with an object containing:

| Prop       | Type                               | Required | Description                                                                               |
|------------|------------------------------------|----------|-------------------------------------------------------------------------------------------| 
| execute    | (items: SidekickItem[]) => void    | Yes      | Callback function invoked when the Kick is executed                                       |
| canExecute | (items: SidekickItem[]) => boolean | No       | Callback function invoked to check if Kick can be executed for selected items in Sidekick |

Usage Example

```tsx
import {useCallback, useEffect} from 'react';
import { Kick } from './Kick';
import { Theme, XyFetch } from 'your-library';
import type { RegisterKickResult } from 'your-types';

const KickComponent: FC<KickProps> = ({onReady, theme, xyFetch, onCloseKick}) => {
  const handleExecute = useCallback((items: SidekickItems[]) => {
    console.log('Kick executed:', items);
  }, []);

  const handleCanExecute = useCallback((items: SidekickItems[]) => {
    // can be executed only for single selection;
    return items.length === 1;
  }, []);

  const handleCloseKick = useCallback(() => {
    onCloseKick?.();
  }, [onCloseKick]);
  
  useEffect(() => {
    onReady({
      execute: handleExecute,
      canExecute: handleCanExecute
    })
  }, [handleExecute]);

  return (
    <div>Kick Sample</div>
  );
};
```

### Step 3: Configure Module Federation in vite.config.ts

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import federation from '@originjs/vite-plugin-federation'

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'customerKicks',
      filename: 'kicks.js',
      exposes: {
        './Kicks': './src/components/MyKicks.ts'
      },
      shared: ['react', 'react-dom']
    })
  ],
  build: {
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  }
})
```

### Step 4: Build the Library

Run the Vite build command to generate a production-ready bundle:

```bash
  vite build
```

This will create a `dist/` directory containing:
- remoteEntry.js
- All JavaScript/CSS chunks
- Other assets

### Step 5: Deploy the dist/ Folder

You should deploy the contents of `dist/` to a public location such as:
- AWS S3 + CloudFront
- Azure Blob Storage
- GitHub Pages
- Your custom CDN

Make sure the URL to `remoteEntry.js` is accessible by the host app. For example:

https://customer.com/widgets/remoteEntry.js

### Step 5: Share the Remote URL

Once deployed, provide the public URL of remoteEntry.js to the host application team so they can consume your library using Module Federation.