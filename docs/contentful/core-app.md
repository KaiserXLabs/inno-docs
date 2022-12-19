# @inno-contentful/core-app

Consolidation of all core modules used to enrich the editor experience on Contentful. This react app is bundled and shipped to a static webserver. It can be used on any designated space or enviroment. It is built in a way to work as a replacement for the built in `Editor`. For all default fields, where no custom editor should be applied to, it will simply use the [Field Editor Library](https://github.com/contentful/field-editors/) provided by Contentful to render the field.

For all custom fields, which in our case are complex objects when speaking about the data type, the app is going to render the corresponding module. All the data storage is based on the interface definition of the sdk which is on `sdk.entry.field` level. This is a change of paradigm, as we were using the `sdk.field` approach when we were applying all the custom `apps` on field level rather than on Entry Editor level.

## Development

Run a dev server with hot module reloading (HMR) enabled with the following command:

```bash
# Start Vite dev server
npm run dev
```

## Styling / Components

Prevent using custom style sheets and stick to the utility classes provided by `@contentful/f36-*`. That keep's the code base clean and consistent. The same rule applies to the component template. Please stick to what the component library has to offer and only use those components. An example is the so called `Box` which should use rather the `div`. You might still see parts of the repo where there are stylesheets. This is for the old components where no refactoring has been taken place. If you stumble across one of them, feel free to change them accordingly to the appraoch I mentioned above.

If you are going to create custom styling to some component, a dedicated review is needed. Involve editors as early possible to prevent ending up with some component which is hard to use. This will harm the editor experience, and that's exactly the opposite what we would like to achieve with this approach. In the end the editor experience should be enhanced with the changes and adaptions we provide.

## Testing / Documentation

We use [Storybook](https://storybook.js.org/) for both testing and documenation. With this tool we are able to provide an overview on the look & feel as well as the API to it. As all components rely on the [Contentful App SDK](https://www.contentful.com/developers/docs/extensibility/app-framework/sdk/) we need to make sure that the sdk itself is mocked in order to run our tests. There is a custom Provider which will do that. Each component should have it's own story to both describe it's purpose, it's functionality as well as to demonstrate how it is intended to be used.

### Write a story

Each module should have two parts included:

- A markdown file to document the module
- Dedicated component stories to showcase and test the component

Put them into this folder structure:

```
  src
  |
  | - /stories
  | --  /examples
  | --  /<module>
  | ---    <module>.stories.mdx       // General module documentation
  | ---    <component>.stories.tsx    // Showcase, test and describe component
```

## Available Scripts

In the project directory, you can run:

#### `npm start`

Creates or updates your app definition in Contentful, and runs the app in development mode.
Open your app to view it in the browser.

The page will reload if you make edits.
You will also see any lint errors in the console.

#### `npm run build`

Builds the app for production to the `build` folder.
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.
Your app is ready to be deployed!

#### `npm run upload`

Uploads the build folder to contentful and creates a bundle that is automatically activated.
The command guides you through the deployment process and asks for all required arguments.
Read [here](https://www.contentful.com/developers/docs/extensibility/app-framework/create-contentful-app/#deploy-with-contentful) for more information about the deployment process.

#### `npm run upload-ci`

Similar to `npm run upload` it will upload your app to contentful and activate it. The only difference is  
that with this command all required arguments are read from the environment variables, for example when you add
the upload command to your CI pipeline.

For this command to work, the following environment variables must be set:

- `CONTENTFUL_ORG_ID` - The ID of your organization
- `CONTENTFUL_APP_DEF_ID` - The ID of the app to which to add the bundle
- `CONTENTFUL_ACCESS_TOKEN` - A personal [access token](https://www.contentful.com/developers/docs/references/content-management-api/#/reference/personal-access-tokens)

## Libraries to use

To make your app look and feel like Contentful use the following libraries:

- [Forma 36](https://f36.contentful.com/) – Contentful's design system
- [Contentful Field Editors](https://www.contentful.com/developers/docs/extensibility/field-editors/) – Contentful's field editor React components

## Using the `contentful-management` SDK

In the default create contentful app output, a contentful management client is
passed into each location. This can be used to interact with Contentful's
management API. For example

```js
// Use the client
cma.locale.getMany({}).then((locales) => console.log(locales));
```

Visit the [`contentful-management` documentation](https://www.contentful.com/developers/docs/extensibility/app-framework/sdk/#using-the-contentful-management-library)
to find out more.

## Learn More

[Read more](https://www.contentful.com/developers/docs/extensibility/app-framework/create-contentful-app/) and check out the video on how to use the CLI.

Create Contentful App uses [Create React App](https://create-react-app.dev/). You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started) and how to further customize your app.
