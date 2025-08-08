# Xyleme Sidekick

## Overview

This project provides examples of how to create your own custom Kicks for use with Sidekick, available within MadCap Xyleme Create. You may use these examples as a base for your own Kicks, or use them as reference for creating your own Kicks from scratch.

## What is Sidekick?

Sidekick serves as an extensible toolkit within the Create platform, allowing for organizations to provide their content teams with the tools they need, directly within the flow of work. This means that native custom tooling is available to content teams instead of requiring them to use multiple external tools to meet their goals.

### The role of Kicks

Kicks are the vehicles for individual sets of functionality, you could also think of them as tools or actions. Kicks provide a user with a new action they can use from within Sidekick when selecting appropriate content. The functionality of a given Kick can be as simple or as complex as is required by the given situation. Kicks can perform functions directly, call the Create APIs, or call external APIs.

A Kick is built in JavaScript as a React component, with some additional definition that allows it to be recognized by Sidekick. More information about the technical requirements of Kicks is found below.

A Kick may also be governed by role-based access, ensuring only appropriate users can leverage the Kick’s functionality.

## Example projects

Custom Kicks are simple React components wrapped in a Vite build package. This configuration leverages module federation to allow organizations to retain ownership of their custom Kicks. Organizations control hosting of their custom Kick code, while the custom Kicks load natively into the Sidekick module. This means that custom Kicks are treated equally within Create rather than acting as an external module without access to the core Sidekick functionality.

### How example projects are structured

The following examples of custom Kicks are broken into separate projects for easy reference. The examples, though they get progressively more substantial, strive to be as simple as possible and are usually composed of a minimal set of files:

- `src/index.jsx` - The main JavaScript module that holds the React component and all the kick functionality.
    - This can be split into many different files, if desired. Our examples will generally be within this single file.

- `vite.config.js` - This is the Vite configuration file, the packaging framework leveraged by Sidekick.

    - This file is extremely important as it contains the configuration for module federation, the mechanism that exposes the custom Kick to Sidekick itself.

- `package.json` - This is the manifest file for Node Package Manager (NPM) and is where you will define all of your project dependencies. 

    - Dependencies may include React, Vite, or anything else needed for your Kick’s functionality.

All example projects contain ample code comments to lead you through the functionality of the Kick.

### Example projects

The following projects act as models for you to build your own custom Kicks. Each example project builds upon the previously listed projects.

- [**simple-kick**](https://github.com/xyleme-sidekick/simple-kick): The most basic custom Kick possible. This Kick registers itself with Sidekick and prints the currently selected items to the browser dialog.
- [**xy-fetch-kick**](https://github.com/xyleme-sidekick/xy-fetch-kick): This Kick uses the Create APIs to request information about the selected items. This example applies the xy-fetch function provided by Sidekick to custom Kicks through module federation.
- [**integrated-kick**](https://github.com/xyleme-sidekick/integrated-kick): This Kick Builds upon the previous xy-fetch-kick example, but extends it to leverage:
    - The native Create MUI theme, provided by Sidekick to custom Kicks through module federation, to look seamless when viewed within Create.
    - Create permissions, to ensure that only users with the proper role assignment can access the Kick.
    - Sidekick item filters using the canExecute function, to ensure that the Kick is only enabled when compatible Sidekick items are selected.
- [**get-info-kick**](https://github.com/xyleme-sidekick/get-info-kick): This is an example of an advanced custom Kick. This Kick collects and displays document information for the selected items.

## Getting started

Developing, running, building, and deploying your custom Kick requires Node.js v18+ and either NPM or Yarn, all other dependencies are found in each project’s package.json file. These dependencies can be installed with either npm install or yarn install, depending on which package manager you choose.

Each project repository will have instructions on how to clone, run, test, and build the example Kick.

## Overview of creating your own Kick

In order to build a custom Kick and make it available to your Create instance, there are a few key tasks to complete:

1. Create the Kick project by either cloning one of the example project repositories or creating your own.
2. Install all project dependencies.
3. Create your custom Kick’s React component.
4. Register your custom Kick by exporting the registerKicks function referencing your custom Kick’s component.
5. Configure your Vite file to expose your main output file as './Kicks' for module federation.
6. Build your project into a production build and serve it via a publicly-accessible URL (for example, from a CDN).
7. Provide your Xyleme CSM with the public URL to your custom Kick. 

Once your Create instance is configured to point to this custom Kick’s URL, it will automatically become available within Sidekick itself.

## License

All example projects are [MIT Licensed](https://github.com/xyleme-sidekick/.github/blob/main/LICENSE).

## Additional resources 

- [Sidekick Integration & Module Federation Overview](https://github.com/xyleme-sidekick/.github/blob/main/README.md)

- [Sidekick user guide](https://training.bravais.com/s/2419/sidekick-user-guide?page=Item1)

- [Create User Reference Guide](https://training.bravais.com/s/2300/create-user-reference-guide?page=Item1)

- [Create Administrator User Guide](https://training.bravais.com/s/2307/create-administrator-user-guide?page=Item1)

- [Create API documentation](https://api-docs.xyleme.com/docs/create/a06cbm00k0uug-xyleme-create-api)