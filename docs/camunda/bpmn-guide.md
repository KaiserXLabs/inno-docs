# @inno/camunda-bpmn

This repository collects all the BPMN processes and their dependencies created for the Inno Platform and deployed to Camunda. It also contains a simple `camunda.dt.ts` that enables simple intellisense when creating script task scripts.

# Inno Platform Camunda Process Deployer

This Node.js application helps you to deploy Camunda BPMN processes that consist of more than one file, usually a BPMN file and several JavaScript files referenced within the BPMN process. In this case the built in deployment feature of the Camunda Modeler cannot be used.

## Prerequesites

1. Node.js installed
2. To create Camunda specific BPMN files, you need to install the [Open Source Camunda Modeler](https://camunda.com/download/modeler).
3. Run `npm install`

## Creating a complex BPMN process

1. In your file system - create a folder `<process id>`, for example `~/camunda/processes/or/whatever/cyberschutz` where _cyberschutz_ is the process id

Within the Camunda Modeler do:

2. `File / New File / BPMN Diagram` to create a new diagram
3. Model your process and don't forget to set the id of the process to your `<process id`>, for example `cyberschuz`
4. `File / Save as` to save the new diagram, choose the folder you just created and give it a name, usually `<process id>.bpmn`, for example `cyberschutz.bpmn`

If you stop here and you do not use any external reference in your process, your process consists of only one file can be deployed by using the Camunda Modeler built in deployment feature. However, if your process is getting more complex, inline scripting within script tasks is getting more and more incomprehensible. Not mentioning, that you cannot reuse inline code. And this is when external resource references come into play.

5. Inside your process folder you created in step 1, create a JavaScript file, for example `settings.js`
6. Inside your process - create a script task and instead of choosing `Inline Script` as _Script Type_, choose `External Resource`.
7. Reference the JavaScript file, you just created, by entering `deployment://<script name>.js`, for example `deployment://settings.js` into the _Resource_ field
8. Don't forget to enter `javascript` into the _Script Format_ field

## Deploy a complex BPMN process

Note: camunda basic auth credentials can be found in 1password in the InnoPlatform vault at "Camunda Basic Auth".

```bash
npm start <path to your process folder> https://<your camunda endpoint>/engine-rest <camunda basic auth username> <camunda basic auth password>
```

For example:

```bash
npm start journeys/cyberschutz https://camunda.staging.inno.kxlabs.allianz.de/engine-rest <camunda basic auth username> <camunda basic auth password>
```

## Organizing script files inside the process folder

To organize script files inside the process folder, you can create a subfolder structure and place your JavaScript files to that structure, for example:

```
tree ~/camunda/processes/or/whatever/cyberschutz
~/camunda/processes/or/whatever/cyberschutz
├── calculatePrices
│   └── index.js
├── cyberschutz.bpmn
└── settings.js
```

Since you cannot have such a structure inside Camunda, the camunda deployer is flattening the structure for you.

So, for example a path like `subFolder/subSubFolder/scriptName.js` becomes `subFolder.subSubFolder.scriptName.js`.

Going with the example above, if you want to reference the `index.js` file located inside the `calculatePrices` subfolder, you have to enter `deployment://calculatePrices.index.js` into the _Resource_ field of the script task.

## Verifying the deployment with the Camunda Cockpit

1. Open Camunda inside your browser - for example: https://camunda.innox.kxlabs.io
2. Enter the admin credentials - the credentials for the example in step 1 can be found at [here](https://start.1password.com/open/i?a=33HQLPHGMRATNIRXY35QOC2VTU&v=6cj633pwmncbplpxfoqo7fqhsa&i=3k2gbcipn5aqxciunjsh75cs6q&h=kaiserxlab.1password.com).

The cockpit should be visible now.

3. Go to `Deployed / Deploymens` and click on the number

Your latest deployment should be visible at the top of the list of deployments and it's name should be `<process id>`.

4. In the column right to the list, your flattened process folder structure should appear.
5. Click on a file in that column and you will see it's text content or in case of a BPMN file it's diagram.

## Cypress Smoke Tests

To start the cypress test runner perform one of the following commands - in order to run pferdelv tests you have to provide an agent number + password:

### Headless with video recording

```bash
npm run cypress:run -- --env "PFERDELV_AGENT_NUMBER=<an agent number>,PFERDELV_PASSWORD=<an agent password>"
```

### Browser

Note: Google Chrome stops running after the first sucessful run. Firefox is working stable.

```bash
npm run cypress:open -- --env "PFERDELV_AGENT_NUMBER=<an agent number>,PFERDELV_PASSWORD=<an agent password>"
```

## Delete all deployments of complex BPMN process

```bash
npm run delete <path to your process folder> https://<your camunda endpoint>/engine-rest <camunda basic auth username> <camunda basic auth password>
```
