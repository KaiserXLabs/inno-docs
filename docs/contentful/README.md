# Inno Contentful

A monorepo for all Contentful related code. This monorepo is managed with [lerna](https://lerna.js.org/).

## Structure 

```
  @inno-contentful
  |
  | - /cli
  | - /components
  | - /core-app
  | - /lib
  | - /migrations
  | ... 
```

### @inno-contentful/cli
A collection of custom CLI toolings. Only accesible on your local machine. There are some prerequisites to it, please check the [README](./packages/cli/README.md) file for further information on how to set up the package. 

### @inno-contentful/components
A collection of shared components for all the apps we are going to provide. This package is going to be a component library hosted on the [internal registry server of Kaiser X Labs](https://registry.kxlabs.io/). If you need access to this registry, check 1Password. 

### @inno-contentful/core-app
The core application used on Conteful to enrich the editor experience. It has a set of different modules which are bundled by this app. It has two depencies, which is both the `@inno-contentful/components` and `@inno-contentufl/lib` package. You can always check the depencies within this monorepo by running the graph command provided by `nx`. To do so, simply run `npx nx graph`. This eventually will spin up an UI with which you can identify the dependencies created. 

### @inno-contentful/lib
Anything else relevant within the context of Contentful is going to be wrapped by that package. By the time this text is written, this is a shared data model as well as shared react hooks. Same as for components, this package is going to built as a library and eventually published to the Kaiser X Labs registry server. For the local development, you are going to be use the local reference, which speeds up the development flow. 

### @inno-contentful/migrations
A collection of migration scripts. Check the package [README](./packages/migrations/README.md) for futher information on how to work with this package. There are currently two ways on how to execute those scripts. One is locally, secondly we do have those scripts automatically executed by our CI pipeline (GitHub action).

## Development

If you want to run scoped commands from the root of the project, you simply use the package name as scope parameters. Useful commands are going to be defined in the root `package.json`. For example, if you want to run the `@inno-contentful/core-app` in development mode with HMR enabled, including other services like the `@inno-contentful/components` and the `@inno-contentful/lib` up and running the same time, simply use this command: 

```
npm run start:dev
```
This will crawl through the `/packages` folder and execute the `dev` command in parallel. 

