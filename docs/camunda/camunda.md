# @inno/camunda

This is [GraalVM](https://www.graalvm.org) enabled version of the standard [Camunda](https://camunda.com) BPMN Platform [docker container](https://github.com/camunda/docker-camunda-bpm-platform) in order to run modern ES6 JavaScript within BPMN script tasks.

## Run locally

### Prerequisites

1. Install OpenJDK 11 with homebrew and make sure it's used when you do `java --version` on the commandline:

```sh
brew install openjdk@11
# optional:
echo 'export PATH="/usr/local/opt/openjdk@11/bin:$PATH"' >> ~/.zshrc # or whatever the brew output is suggesting here
```

To start Camunda locally without injecting any explicit micro service base URL:

```sh
./start-restapi.sh
```

## Open local camunda cockpit

1. Open the local camunda cockpit in the browser at http://localhost:8080
2. Login with:

   Username: demo

   Password: demo

## Custom inno platform tasks

Custom inno platform tasks are Java classes inside the Camunda JAR that can be referenced in a BPMN Service task, whose implementation is _Java Class_ (deprecated) and/or _Delegate expression_ (recommended).

### PDFGeneratorTask

Calls a PDF Generator service (set via `PDF_GENERATOR_BASE_URL` environment variable) and returns it's PDF response that can be stored as bas64 encoded string or actual PDF file to the process.

#### BPMN service task settings

##### Implementation

Recommended:

- Type: `Delegate expression`
- Delegation expression: `${pdfGeneratorTask}`

_Deprecated_:

- Type: `Java class`
- Java class: `com.kaiserx.camunda.task.PDFGeneratorTask`

##### Input params

| Input              | Required | Description                                                                                                                  |
| ------------------ | -------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `payload`          | yes      | a JSON string or serialized object with a signature according to the `endpoint` and the actual PDF template                  |
| `fileVariableName` | no       | This is the name of the process variable where the actual PDF file should be stored. If it's not set, no file will be stored |
| `endpoint`         | no       | the endpoint of the cluster's PDF service to be used. Default value is `/generate-merged-pdf` if not set                     |

##### Output params

| Output               | Scope   | Description                                                                                                |
| -------------------- | ------- | ---------------------------------------------------------------------------------------------------------- |
| `error`              | task    | Contains the error message if service request was not successful. Otherwise it's null                      |
| `statusCode`         | task    | Contains the HTTP status code of the service request or 500 if an error occured                            |
| `response`           | task    | Contains the base64 encoded PDF file string or null if an error occured                                    |
| `downloadTimestamp`  | task    | Contains the creation timestamp of the PDF document or null if an error occured                            |
| `<fileVariableName>` | process | Contains the PDF file object or not set at all if an error occured or `fileVariableName` input was not set |

#### Example process

See the included [\_pdfGeneratorTest.bpmn](src/main/resources/default-deployment/_pdfGeneratorTest.bpmn) for how to set it up correctly.

#### Run the example process

1. Have the [PDF generator service](https://github.com/KaiserXLabs/pdf-generator-api) running locally at http://localhost:3000
2. Start camunda locally and inject the `PDF_GENERATOR_BASE_URL` like this:

```sh
PDF_GENERATOR_BASE_URL=http://localhost:3000 ./start-dev.sh
```

3. Start the process via Camunda Rest API:

```sh
curl --location --request POST 'http://localhost:8080/engine-rest/process-definition/key/_pdfGeneratorTest_/start' \
--header 'Content-Type: application/json' \
--data-raw '{}'
```

### EmailServiceTask

Takes a variable `payload` whose signature should follow the DTO signature of the concrete email-service endpoint.

#### BPMN service task settings

##### Implementation

Recommended:

- Type: `Delegate expression`
- Delegation expression: `${emailServiceTask}`

_Deprecated_:

- Type: `Java class`
- Java class: `com.kaiserx.camunda.task.EmailServiceTask`

##### Input params

| Input       | Required | Description                                                                                                   |
| ----------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| `_emailUri` | yes      | Uri to the email service that must be initialized before this task                                            |
| `payload`   | yes      | A JSON string or serialized object with a signature according to the `endpoint` and the actual email template |

#### Run the example process

1. Have the order repository service running locally at http://localhost:3000
2. Start camunda locally by injecting the `EMAIL_SERVICE_BASE_URL` like this (injecting is optional and only required if the order repository service is not running at http://localhost:3000):

```sh
EMAIL_SERVICE_BASE_URL=http://localhost:3000 ./start-dev.sh
```

3. Start the process via Camunda Rest API:

```sh
curl --location --request POST 'http://localhost:8080/engine-rest/process-definition/key/_emailServiceTest/start' \
--header 'Content-Type: application/json' \
--data-raw ''
```

### PricingServiceTask

Takes a variable `payload` whose signature should follow the DTO signature of the concrete pricing-service endpoint.

Note: In order to point this task to the journey specific pricing service endpoint, a private variable called `_pricingUri` needs to live in the BPMN process and also needs to be initialized before the first call of the this task.

The Java Class that has to be referenced in the Service Task: `com.kaiserx.camunda.task.PricingServiceTask`.

See the included [pricingServiceTest.bpmn](src/main/resources/pricingServiceTest.bpmn) for how to set it up correctly.

#### Run example process locally

1. Have the pricing service running locally at http://localhost:3000
2. Start camunda locally by injecting the `PRICING_SERVICE_BASE_URL` like this (injecting is optional and only required if the pricing service is not running at http://localhost:3000):

```sh
PRICING_SERVICE_BASE_URL=http://localhost:3000 ./start-dev.sh
```

3. Start the process via Camunda Rest API:

```sh
curl --location --request POST 'http://localhost:8080/engine-rest/process-definition/key/_pricingServiceTest/start' \
--header 'Content-Type: application/json' \
--data-raw '{}'
```

### OrderTasks

The following tasks are order related. See the included [orderTestProcess.bpmn](src/main/resources/orderTestProcess.bpmn) for how to set them up correctly.

### UpsertOrderTask

Checks if there is an `_order` variable already in the process. If yes it updates otherwise it creates the order with the customer information available in the process. When creating a new order, the current `processInstanceId` is added as a new order item.

It expects the following variables to be in the process:

- `firstName`
- `lastName`
- `email`
- `phone`
- `street`
- `houseNumber`
- `zipCode`
- `city`
- `birthDate`

If they are not set yet, it will default to `NOT_SET`.

If your process for what ever reason has to use different variable names - you can map them to the required names above via task input parameters in the camunda modeler:

```xml
<camunda:inputOutput>
  <camunda:inputParameter name="firstName">${myCustomFirstName}<camunda:inputParameter>
</camunda:inputOutput>
```

The Java Class that has to be referenced in the Service Task: `com.kaiserx.camunda.task.order.UpsertOrderTask`.

### AddOrderItemTask

This task adds a new order item to an existing `_order`. If there is no `_order` in the process it does nothing. It expects a variable `itemProperties` that can be set in the camunda moderler via task input parameter:

```xml
<camunda:inputOutput>
  <camunda:inputParameter name="itemProperties">
    <camunda:script scriptFormat="javascript">
      const payload = {key1:"value1",...};
      S(JSON.stringify(payload));
    </camunda:script>
  </camunda:inputParameter>
</camunda:inputOutput>
```

The Java Class that has to be referenced in the Service Task: `com.kaiserx.camunda.task.order.AddOrderItemTask`.

### CompleteOrderTask

This task takes an existing `_order` and sets it's state to `"completed"`.

The Java Class that has to be referenced in the Service Task: `com.kaiserx.camunda.task.order.CompleteOrderTask`.

#### Run example process locally

1. Have the order repository service running locally at http://localhost:3000
2. Start camunda locally by injecting the `ORDER_REPOSITORY_BASE_URL` like this (injecting is optional and only required if the order repository service is not running at http://localhost:3000):

```sh
ORDER_REPOSITORY_BASE_URL=http://localhost:3000 ./start-dev.sh
```

3. Start the process via Camunda Rest API:

```sh
curl --location --request POST 'http://localhost:8080/engine-rest/process-definition/key/_pricingServiceTest/start' \
--header 'Content-Type: application/json' \
--data-raw '{}'
```

### AgencySearchTask

This searches for a agent information at the inno platform bff agency search and returns it as `response`.

See the included [agencySearchTest.bpmn](src/main/resources/agencySearchTest.bpmn) for how to set it up correctly.

#### BPMN service task settings

##### Implementation

Recommended:

- Type: `Delegate expression`
- Delegation expression: `${agencySearchTask}`

_Deprecated_:

- Type: `Java class`
- Java class: `com.kaiserx.camunda.task.agencysearch.AgencySearchTask`

##### Input params

| Input      | Required | Description                   |
| ---------- | -------- | ----------------------------- |
| `agencyId` | yes      | A number represetings a agent |

##### Output params

| Output     | Scope | Description                                                                           |
| ---------- | ----- | ------------------------------------------------------------------------------------- |
| `error`    | task  | Contains the error message if service request was not successful. Otherwise it's null |
| `response` | task  | Contains the data if service request was successful. Otherwise it's null              |

#### Run the example process

1. Have the order repository service running locally at http://localhost:3000
2. Start camunda locally by injecting the `BACKEND_BASE_URL` like this (injecting is optional and only required if the order repository service is not running at http://localhost:3000):

```sh
BACKEND_BASE_URL=http://localhost:3000 ./start-dev.sh
```

Or just start with:

```sh
./start-dev.sh
```

3. Start the process via Camunda Rest API:

```sh
curl --location --request POST 'http://localhost:8080/engine-rest/process-definition/key/_agencySearchTest/start' \
--header 'Content-Type: application/json' \
--data-raw ''
```

### HashJsonTask

This task creates a hash of from a payload payload and return it.

See the included [\_jsonHashTest.bpmn](src/main/resources/_jsonHashTest.bpmn) for how to set it up correctly.

#### BPMN service task settings

##### Implementation

Recommended:

- Type: `Delegate expression`
- Delegation expression: `${hashJsonTask}`

_Deprecated_:

- Type: `Java class`
- Java class: `com.kaiserx.camunda.task.agencysearch.HashJsonTask`

##### Input params

| Input        | Required | Description                                                                                                 |
| ------------ | -------- | ----------------------------------------------------------------------------------------------------------- |
| `payload`    | yes      | a JSON string or serialized object with a signature according to the `endpoint` and the actual PDF template |
| `digestName` | no       | Defaults to SHA-1                                                                                           |

##### Output params

| Output     | Scope | Description              |
| ---------- | ----- | ------------------------ |
| `response` | task  | Returns the hash created |

#### Run the example process

1. Start camunda locally:

```sh
./start-dev.sh
```

2. Start the process via Camunda Rest API:

```sh
curl --location --request POST 'http://localhost:8080/engine-rest/process-definition/key/_jsonHashTest/start' \
--header 'Content-Type: application/json' \
--data-raw ''
```

### CleanupStaleProcessInstancesTask

This task takes a `processDefinitionKeyList` and an optional `sessionTimeoutInMinutes` and queries all process instances that started `sessionTimeoutInMinutes` ago and terminates them, no matter if the user was active ten seconds ago. If `sessionTimeoutInMinutes` is not set it defaults to 60 minutes.

#### BPMN service task settings

##### Implementation

Recommended:

- Type: `Delegate expression`
- Delegation expression: `${cleanupStaleProcessInstancesTask}`

_Deprecated_:

- Type: `Java class`
- Java class: `com.kaiserx.camunda.task.agencysearch.CleanupStaleProcessInstancesTask`

##### Input params

| Input                      | Required | Description                                                                                                 |
| -------------------------- | -------- | ----------------------------------------------------------------------------------------------------------- |
| `processDefinitionKeyList` | yes      | a JSON string or serialized object with a signature according to the `endpoint` and the actual PDF template |
| `sessionTimeoutInMinutes`  | no       | If `sessionTimeoutInMinutes` is not set it defaults to 60 minutes.                                          |

### CleanupExpiredSessionsTask

This task takes a `processDefinitionKeyList` and an optional `sessionTimeoutInMinutes` and queries all variable instances of `__lastModifiedTimestamp` that where updated `sessionTimeoutInMinutes` ago and terminates their associated process instances. It depends on [UpdateLastModifiedTimestampTask](#UpdateLastModifiedTimestampTask).

#### BPMN service task settings

##### Implementation

Recommended:

- Type: `Delegate expression`
- Delegation expression: `${cleanupExpiredSessionsTask}`

_Deprecated_:

- Type: `Java class`
- Java class: `com.kaiserx.camunda.task.agencysearch.CleanupExpiredSessionsTask`

##### Input params

| Input                      | Required | Description                                                                                                 |
| -------------------------- | -------- | ----------------------------------------------------------------------------------------------------------- |
| `processDefinitionKeyList` | yes      | a JSON string or serialized object with a signature according to the `endpoint` and the actual PDF template |
| `sessionTimeoutInMinutes`  | no       | If `sessionTimeoutInMinutes` is not set it defaults to 60 minutes.                                          |

### UpdateLastModifiedTimestampTask

This task updates `__lastModifiedTimestamp` or creates it if not it does not exist in the process. So put it after user tasks or inside message branches in order to update user activity.

#### BPMN service task settings

##### Implementation

Recommended:

- Type: `Delegate expression`
- Delegation expression: `${updateLastModifiedTimestampTask}`

_Deprecated_:

- Type: `Java class`
- Java class: `com.kaiserx.camunda.task.agencysearch.UpdateLastModifiedTimestampTask`

##### Input params

| Input                     | Required | Description                                                                                                 |
| ------------------------- | -------- | ----------------------------------------------------------------------------------------------------------- |
| `__lastModifiedTimestamp` | yes      | a JSON string or serialized object with a signature according to the `endpoint` and the actual PDF template |

##### Output params

| Input                     | Required | Description                                                                                                 |
| ------------------------- | -------- | ----------------------------------------------------------------------------------------------------------- |
| `__lastModifiedTimestamp` | yes      | a JSON string or serialized object with a signature according to the `endpoint` and the actual PDF template |

## Task Helpers

Task Helpers, as the name implies, provide generalized functionality throughout the execution scripts. The idea was to get rid of things which have been implemented several times within the `javascript` files and simply provide them with these new helper classes. They are written and `Java` and hence you have to import them first in order to be able to use them. In order to do so, you would do the following at the top of your `javascript` file:

```javascript
const variableHelper = new com.kaiserx.camunda.task.helper.VariableHelper(
  execution
);
```

Doing so, you have access to all the Class declarations. For now, we have two helper classes in place, which are:

- VariableHelper
- DateHelper
