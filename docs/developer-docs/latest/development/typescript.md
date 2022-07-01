---
title: Typescript - Strapi Developer Docs
description: Learn how you can use Typescript for your Strapi application.
canonicalUrl: https://docs.strapi.io/developer-docs/latest/development/typescript.html
---

# TypeScript development

::: callout 🚧  TypeScript documentation
This section is still a work in progress and will continue to be updated and improved. Migrating existing Strapi applications written in JavaScript is not currently recommended. In the meantime, feel free to ask for help on the [forum](https://forum.strapi.io/) or on the community [Discord](https://discord.strapi.io).
:::

TypeScript adds an additional type system layer above JavaScript, which means that existing JavaScript code is also TypeScript code. Strapi supports TypeScript in new projects on v4.2.0 and above. TypeScript-enabled projects allow developing plugins with TypeScript as well as using TypeScript typings.

::: strapi Getting started with TypeScript
To start developing in TypeScript, use the [CLI documentation](/developer-docs/latest/setup-deployment-guides/installation/cli.md) to create a new TypeScript project. Additionally, the [project structure](/developer-docs/latest/setup-deployment-guides/file-structure.md) and [TypeScript configuration](/developer-docs/latest/setup-deployment-guides/configurations/optional/typescript.md) sections have TypeScript-specific resources for understanding and configuring an application.
:::

## Start developing in TypeScript

Starting the development environment for a TypeScript-enabled project requires building the admin panel prior to starting the server. In development mode, the application source code is compiled to the `./dist/build` directory and recompiled with each change in the Content-type Builder. To start the application, run the following commands in the root directory:

<code-group>

<code-block title="NPM">

```sh
npm run build
npm run develop
```

</code-block>

 <code-block title="YARN">

```sh
yarn build
yarn develop
```

</code-block>

</code-group>

## Use TypeScript typings

Strapi provides typings on the `Strapi` class to improve the TypeScript developing experience. These typings come with an autocomplete feature that automatically offers suggestions while developing.

To experience TypeScript-based autocomplete while developing Strapi applications, you could try the following:

1. From your code editor, open the `./src/index.ts` file.
2. In the `register` method, declare the `strapi` argument as of type `Strapi`:

    ```js
    // path: ./src/index.ts

    import '@strapi/strapi';

    export default {
      register( { strapi }: { strapi: Strapi }) {
        // ...
      },
    };
    ```

2. Within the body of the `register` method, start typing `strapi.` and use keyboard arrows to browse the available properties.
3. Choose `runLifecyclesfunction` from the list.
4. When the `strapi.runLifecyclesFunctions` method is added, a list of available lifecycle types (i.e. `register`, `bootstrap` and `destroy`) are returned by the code editor. Use keyboard arrows to choose one of the lifecycles and the code will autocomplete.

## Develop a plugin using TypeScript

New plugins can be generated following the [plugins development documentation](/developer-docs/latest/development/plugins-development.md). There are 2 important distinctions for TypeScript applications:

- After creating the plugin, run `yarn install` or `npm run install` in the plugin directory `src/admin/plugins/[my-plugin-name]` to install the dependencies for the plugin.
- Run `yarn build` or `npm run build` in the plugin directory `src/admin/plugins/[my-plugin-name]` to build the admin panel including the plugin.

::: note
It is not necessary to repeat the `yarn install` or `npm run install` command after the initial installation. The `yarn build` or `npm run build` command is necessary to implement any plugin development that affects the admin panel.
:::

## Start Strapi programmatically

Starting Strapi programmatically in a TypeScript project requires additional configurations to load everything correctly. The primary difference for TypeScript programmatic use is that the codebase and compiled code are stored in separate directories, whereas the same directory is used to read and write in native JavaScript.

### Understand programmatic use

<!-- NOTE TO SELF: add callout here about the typical programmatic use config for TS with what not to do. Also move most of this content elsewhere-->

When instantiating Strapi programmatically using the default export of `@strapi/strapi`, different parameters such as the directories containing the codebase and the compiled codebase can be passed. The important information for TypeScript applications is that there should be a separate directory for compiling the codebase.

::: caution
Do not set `appDir` to `build` or `dist` directories as it could cause issues when the app tries to write certain files.
:::

For programmatic use of Strapi with TypeScript use the  `strapi()` function. The function accepts 2 parameters, `appDir` and `distDir`, to set the path of the root directory and the compiled codebase directory respectively. For TypeScript projects Strapi should be instantiated with a specified `distDir` directory, and specifying `appDir` is optional. The two possible configurations for TypeScript projects are as follows:

| `appDir` parameter value | `distDir` parameter value | Actual `app` directory    | Actual `dist` directory   |
| ------------------------ | ------------------------- | ----------------------    | -----------------------   | 
| -                        | `./dist`                  | current working directory | `./dist`                  | 
| `./app`                  | `./dist`                  | `./app`                   | `./dist`                  | 


::: note
The [public directory](/developer-docs/latest/setup-deployment-guides/configurations/optional/public-assets.md) is considered static and thus ignores the  `app` and `dist` directories.
:::

::: details Examples of how Strapi resolves directories based on passed parameters for TypeScript projects:

The default values for the `app` and `dist` directories are transformed and assigned using one of the following options:

```js
const resolveWorkingDirectories = opts => {

  const distDir = opts.distDir ? path.resolve(cwd, opts.distDir) : appDir; // Only distDir is defined, appDir is set to process.cwd().

//NOTE TO SELF: example to set both here

  return { appDir, distDir };
}
```

Start Strapi using a custom `dist` directory:

```js

const strapi = require('@strapi/strapi');

strapi({ distDir: './dist' });
// appDir => process.cwd() | distDir => './dist'
```

Start Strapi using custom `app` and `dist` directories:

```js

const strapi = require('@strapi/strapi');

strapi({ appDir: './app', distDir: './dist' });
// appDir => './app' | distDir => './dist'

```

:::

### Use both JavaScript and TypeScript codebases when starting Strapi programmatically

To allow JavaScript and TypeScript codebases to be used programmatically together. A common use is for creating command-line interface tools or developing a plugin. The XXXX checks the codebase for JavaScript and TypeScript files. If the project contains TypeScript files the `dist` directory is fetched from the property `outDir` in the `tsconfig.json` file and assigned to the property `distDir`. To add and implement the `@strapi/typescript-utils` package use the following code snippet:

```js

const strapi = require('@strapi/strapi');

const appContext = await strapi.compile();
// Start the app by providing the app and dist directories:
const app = await strapi(appContext).load(); 

```