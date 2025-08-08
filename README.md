
# Sidekick Integration and Module Federation Overview

Sidekick can integrate custom Kicks through module federation. This capability requires a few specific libraries and configuration nuances, so use this documentation to ensure that your custom Kick is properly configured for use in Sidekick. We will start with an overview of how to package and deploy a custom Kick for integration with Sidekick. Then there will be more detailed information on how to actually develop a custom Kick.

## Module federation overview

Module federation allows two independent JavaScript applications to share code and load remote modules at runtime. Sidekick uses the JavaScript packaging library Vite to provide this capability. With Vite, this is possible via the [`vite-plugin-federation`](https://github.com/originjs/vite-plugin-federation) plugin. Both of these are required dependencies of any custom Kick project.

### Key concepts of module federation:

- **Host App (Sidekick)**: The application that loads components or modules from remote sources.
- **Remote App (Custom Kick)**: The application that exposes specific components, hooks, or utilities to the host app.
- **Shared dependencies (or shared modules)**: Libraries, like React or Vue, that are shared between both the host and remote apps to avoid duplication and conflicts.
  - Example: Both the host app and the remote app might be able to use the host app’s instance of React, as long as the host app exposes it for shared use.

## Building a Custom Kick Project

### Prerequisites
- Vite v4+
- `vite-plugin-federation` plugin

### Remote (Customer Library) Configuration
In `vite.config.ts`:

```ts
/**
 * Sidekick uses Vite as a bundler in order to support module federation,
 * a powerful feature that enables Sidekick to load Kicks from external
 * servers.
 *
 * Read more about this config file at https://vite.dev/config/
 */
import { defineConfig } from 'vite'
import federation from '@originjs/vite-plugin-federation'

/**
 * This file exports a configuration object which will require the
 * `build` and `plugins` properties, in order to configure Vite to
 * properly build the Kick for Sidekick.
 */
export default defineConfig({
 // Here, we will configure Vite to expect the latest version of EcmaScript (JavaScript).
  build: {
    target: 'esnext'
  },
  /**
   * Here, we will tell Vite that we are using React by loading the `react()` plugin.
   * We will also use the `federation()` plugin to expose our code to Sidekick.
   * This configuration will tell Vite to find our `/src/index.js` file and expose it
   * as `kicks.js` when loaded by Sidekick. The exposed component must be named `./Kicks`.
   *
   * This configuration also states that the `react` and `react-dom` modules should be
   * shared by the host application (Sidekick), such that they do not need to be part
   * of the Kick's own bundle.
   */
  plugins: [
    federation({
      name: 'kicks',
      filename: 'kicks.js',
      exposes: {
        './Kicks': './src/components/index.jsx',
      },
      shared: ['react', 'react-dom']
    })
  ]
})
```

### Build process
To prepare the custom Kick for deployment, run the following command:

```bash
  vite build
```

This generates the production-ready files in the dist/ directory. The contents of the dist/ folder will include:

- `kicks.js` (the entry point file for your custom Kick)

- Federated JavaScript chunks

- Static assets (such as styles or images)


## Deploying your custom Kick

### What You Deploy:
- `dist/` directory (the module federation entry point and all dependent JS/CSS chunks)

### Where to Deploy:
- A public CDN or static hosting with a publicly-accessible URL (for example, AWS S3 with CloudFront).
  - Example URL: ```https://example.com/assets/kicks.js```

### Integrating with Sidekick
For Sidekick to successfully integrate your custom Kick, MadCap Xyleme must have a public URL to your deployed federated module (for example, ```https://example.com/assets/kicks.js```). This URL will be used to dynamically load your custom Kicks into Sidekick directly within the Xyleme platform.

> [!IMPORTANT]
> To provide MadCap Xyleme with your URL, please open a ticket with the [Customer Request Center](https://madcapsoftware.atlassian.net/servicedesk/customer/portal/6).

## Creating a custom Kick

In order to create a custom Kick that can be integrated with Sidekick, you must first create a compatible Vite project. You can do this by either cloning one of the example projects and modifying them as needed. To create a new project from scratch, follow these steps:

### Step 1: Initialize Your project
#### Step 1.1: Create a Vite project

If you don't have a Vite project yet, create one with the following commands:

```bash
  npm create vite@latest my-lib --template react-ts
  cd my-lib
  npm install
```

> [!NOTE]
> If you’d like to use Typescript, swap react for react-ts.

> [!NOTE]
> These examples will use NPM, but Yarn is also a viable alternative.

#### Step 1.2: Install the required dependencies

Run the following command to install the required dependencies:

```bash
  npm install @originjs/vite-plugin-federation --save-dev
```

### Step 2: Define the Required Structure for Your Library

To be compatible with the host application, your custom Kick must use ES Module definitions and must export a function named `registerKicks` returning a function, like the following:

```ts
export const registerKicks = () => {
    return {
      kicks: [
        // Your custom Kick(s)
      ]
   }
}
```

This function should return an object containing an array property named kicks. This array will hold the objects describing your custom Kicks that should be registered to Sidekick. Each object in the array will be an object containing the metadata and React component for each of your custom Kicks.

```ts
export const registerKicks = () => {
    return {
      kicks: [
        {
          // The React component of your Kick
          component: customKickComponent;
          description: "string";
          id: "string";
          name: "string";
          position: number;
          userRoles: ["roleName"]
        }
      ]
   }
}
```

You may return several Kicks at once. Each Kick should have a React component and additional information like name, description, id, and sorting position within Sidekick.

The React component that handles the functionality of your Kick should accept the following props:

```ts
const SimpleKickComponent = ({onReady, onCloseKick, xyFetch, theme}) => {...}
```

### Props

| Prop        | Type                                 | Required | Description |
|-------------|--------------------------------------|----------|-------------| 
| onReady     |(result: RegisterKickResult) => void  | Yes      | The callback function invoked when the Kick is ready to use, returning an object with an `execute` function, which runs when the user activates the Kick |
| onCloseKick | () => void                           | No       | The callback function invoked when the Kick is closed or cancelled |
| xyFetch     | XyFetch                              | No       | The network request utility used for making API requests to Xyleme APIs, works like the `fetch` utility |
| theme       | Theme                                | No       | Xyleme’s MUI theme which can be passed to your own MUI implementation, allowing for seamless presentation of custom kicks |
| userRoles   | string[]                             | No       | An array of “Role” names (strings) that a user must have in order to have the Kick available to them. If a user has any of the defined roles assigned to their account, they will be able to see this Kick in Sidekick. |

Usage Example

```tsx
import {useCallback, useEffect} from 'react';

const KickComponent = ({onReady, theme, xyFetch, onCloseKick}) => {
  /**
   * This function is what will be wrapped in an object and passed
   * to the `onReady` prop, which will be run when the Kick is
   * activated in Sidekick.
   *
   * This function can accept the `items` argument which will be a list
   * of items selected by the user in Sidekick when they activate this Kick.
   *
   * This function must be wrapped in React's `useCallback` hook.
   *
   * This particular function will simply log the list of items that
   * were selected by the user in Sidekick to the browser's DevTools.
   */
  const handleExecute = useCallback((items) => {
    console.log('Kick executed:', items);
  }, []);

  /**
     * This call to React's `useEffect` hook is what binds our `execute` function
     * to Sidekick for use.
     * It registers our `execute` function with Sidekick by wrapping the function
     * in an object, with the function being the value of the `execute` property,
     * and then passing to the `onReady` prop we received above.
     *
     * React's hooks system requires that any dependencies be declared explicitly as an array in
     * the second argument to the hook. In this case, we need both the `onReady` prop from our
     * component, as well as the `handleExecut` function we declared above, so we must provide
     * `[onReady, handleExecute]` as the second argument to the `useEffect` hook.
     *
     */
  useEffect(() => {
    onReady({
      execute: handleExecute
    })
  }, [onReady, handleExecute]);

  return (
    <div>Kick Sample</div>
  );
};
```

The `onReady` function should be called once your component is ready with an object containing the following properties:

| Prop       | Type                               | Required | Description                                                                               |
|------------|------------------------------------|----------|-------------------------------------------------------------------------------------------| 
| execute    | (items: SidekickItem[]) => void    | Yes      | The callback function invoked when the Kick is executed by the end user. Will be provided an array of items currently selected in Sidekick as an argument |
| canExecute | (items: SidekickItem[]) => boolean | No       | The callback function invoked to check whether the Kick can be executed for selected items in Sidekick. You can use this to ensure the Kick is only available when only compatible items are selected by the end user. |

Usage Example

```tsx
import {useCallback, useEffect} from 'react';

const KickComponent = ({onReady, theme, xyFetch, onCloseKick}) => {
  /**
   * This function is what will be wrapped in an object and passed
   * to the `onReady` prop, which will be run when the Kick is
   * activated in Sidekick.
   *
   * This function can accept the `items` argument which will be a list
   * of items selected by the user in Sidekick when they activate this Kick.
   *
   * This function must be wrapped in React's `useCallback` hook.
   *
   * This particular function will simply log the list of items that
   * were selected by the user in Sidekick to the browser's DevTools.
   */
  const handleExecute = useCallback((items) => {
    console.log('Kick executed:', items);
  }, []);

  /**
   * Similar to the "handleExecute" callback function above. Used to determine
   * whether the currently-selected items in Sidekick are all compatible with
   * this particular Kick.
   *
   * Should return a boolean. "true" if the Kick should be enabled, "false" if not.
   *
   * This function can accept the `items` argument which will be a list
   * of items selected by the user in Sidekick when they activate this Kick.
   *
   * This function must be wrapped in React's `useCallback` hook.
   *
   * This particular example will only enable the current Kick of there is
   * only a single item selected in Sidekick.
   */
  const handleCanExecute = useCallback((items) => {
    // can be executed only for single selection;
    return items.length === 1;
  }, []);

  /**
     * This call to React's `useEffect` hook is what binds our `execute` function
     * to Sidekick for use.
     * It registers our `execute` function with Sidekick by wrapping the function
     * in an object, with the function being the value of the `execute` property,
     * and then passing to the `onReady` prop we received above.
     *
     * React's hooks system requires that any dependencies be declared explicitly as an array in
     * the second argument to the hook. In this case, we need the `onReady` prop from our
     * component, as well as the `handleExecut` function we declared above, and finally 
     * the `handleCanExectue` from above, so we must provide `[onReady, handleExecute]` as the
     * second argument to the `useEffect` hook.
     *
     */
  useEffect(() => {
    onReady({
      execute: handleExecute,
      canExecute: handleCanExecute
    })
  }, [onReady, handleExecute, handleCanExecutue]);

  return (
    <div>Kick Sample</div>
  );
};
```

### Step 3: Configure Vite Module Federation

In `vite.config.js`:

```ts
/**
 * Sidekick uses Vite as a bundler in order to support module federation,
 * a powerful feature that enables Sidekick to load Kicks from external
 * servers.
 *
 * Read more about this config file at https://vite.dev/config/
 */
import {defineConfig} from 'vite'
import federation from '@originjs/vite-plugin-federation'
import react from '@vitejs/plugin-react'

/**
 * This file exports a configuration object which will require the
 * `build` and `plugins` properties, in order to configure Vite to
 * properly build the Kick for Sidekick.
 */
export default defineConfig({
    // Here we will configure Vite to expect the latest version of EcmaScript (JavaScript)
    build: {
        target: 'esnext'
    },
    /**
     * Here we will tell Vite that we are using React by loading the `react()` plugin,
     * we will also leverage the `federation()` plugin to expose our code to Sidekick.
     * This configuration will tell Vite to find our `/src/index.jsx` file and expose it
     * as `kicks.js` when loaded by Sidekick.
     *
     * The component name being exposed must be `"./Kicks"`.
     *
     * This configuration also states that the `react` and `react-dom` modules should be
     * shared by the host application (Sidekick), such that they do not need to be part
     * of the Kick's own bundle.
     */
    plugins: [
        react(),
        federation({
            name: 'kicks',
            filename: 'kicks.js',
            exposes: {
                './Kicks': './src/index.jsx',
            },
            shared: ['react', 'react-dom']
        })
    ],
})
```

### Step 4: Build the Custom Kick

Run the Vite build command to generate a production-ready bundle:

```bash
  vite build
```

This will create a `dist/` directory containing:
`
- `kicks.js`
- All JavaScript/CSS chunks
- Other assets

### Step 5: Deploy the dist/ Folder

You should deploy the contents of `dist/` to a public URL such as:

- AWS S3 + CloudFront
- Azure Blob Storage
- GitHub Pages
- Your own CDN

Make sure the URL to `kicks.js` is accessible by Sidekick (i.e. it should be public). For example:
https://example.com/assets/kicks.js

### Step 5: Configure your Sidekick instance

Once your custom Kick module is deployed and available via public URL, contact your Xyleme CSM and provide them with the appropriate URL to your Kick. Once your Xyleme platform is configured to point to this custom Kick’s URL, it should automatically become available within Sidekick itself.