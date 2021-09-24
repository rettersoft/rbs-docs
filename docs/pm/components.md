# Process Manager <!-- {docsify-ignore} -->

## Components <!-- {docsify-ignore} -->

As we mentioned earlier, a process is a collection of steps.
You can equip those steps up with certain attributes to make them components.

Process Manager supports various components to accomplish any kind of customization tasks in a project.
You can use these components to solve a problem within your business logic via a process.

### Simple Steps

Simple steps should have only common attributes such as "id", "skip", "state", etc.
You can use them for manipulating state data or declaring an error step.

#### Common Attributes

```typescript
interface {
    id: string
    state?: {}
    userState?: { uid: string, data: {} }
    response?: {}
    info1?: string
    info2?: string
    info3?: string
    error?: string
    skip?: boolean
    persist?: boolean
    flush?: boolean
    first?: boolean
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| id | string | undefined | ✅ | Unique Step ID. |
| state | object | undefined | ❌ | Manipulates execution state. Must be a valid JSON. |
| userState | object | undefined | ❌ | Manipulates user-based state. Stores data by user id. Users can access only their own data via their valid access token. |
| error | string | undefined | ❌ | Custom error string. If *error* is defined in earlier steps, you can't overwrite it with this parameter. You should use *state* instead. |
| response <sup style="color: #0074d9">*</sup> | object | undefined | ❌ | Path to final result. This will be the output. |
| info\[1-3\] <sup style="color: #0074d9">**</sup> | string | undefined | ❌ | Extra information to analyze in process mining. |
| flush | boolean | false | ❌ | Flag to ignore saving execution data into storage. It decreases storage operations. |
| persist | boolean | false | ❌ | Flag to re-enable saving execution data into storage. Use this flag to persist your data when *flush* is enabled. It increases storage operations. |
| first | boolean | false | ❌ | Flag to overwrite starting step. |
| skip | boolean | false | ❌ | Flag to ignore step. |

> In other component steps, you can use common attributes too.

> <span style="color: #0074d9">*</sup> *There are a few ways to return data back to the client that starts the process such as inputPath, parameters and response.*

> <span style="color: #0074d9">**</sup> *Process mining covers pre-defined attributes such as processId, executionId, stepId, etc. With these info parameters, you can pass 3 more data to process mining data pool: info1, info2 and info3.*

#### Simple Step Examples

```typescript
class EmptyStep {
    id: 'EMPTY'
    skip: true
}

class ErrorStep {
    id: 'ERROR'
    error: 'Something went wrong please try again later!'
}

class StateStep {
    id: 'UPDATE_STATE'
    state: {
        "static_data": "static_value",
        "dynamic_data.$": "$.payload.param1"
    }
}
```

### Database (CoDB)

RBS has a DynamoDB-based NoSQL database called CoDB which you can use within your processes.
It has a Collection/Document/Collection/Document/… structure.
Process Manager fully supports CoDB operations: get, put, list, delete.

#### Put Document

```typescript
interface {
    path: string
    document: string
    documentTTL?: number
    outputPath?: string
    resultPath?: string
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| path | string | undefined | ✅ | CoDB document path to save data. It should start with a slash "/". Even number of folders in paths represent documents. |
| document | string | undefined | ✅ | Path to data. |
| documentTTL | string | undefined | ❌ | The amount of time in seconds to keep the data. |
| outputPath | string | $.0.response | ❌ | Path to resolve document response into output. |
| resultPath | string | undefined | ❌ | Variable name to store output into state. |

```typescript
class PutDocumentExample {
    id: 'SAVE_INTO_CODB'
    path: '/collectionA/document1'
    document: '$.pathToDocument'
    documentTTL: 86400
    outputPath: '$.0.response'
    resultPath: 'putDocumentResult'
}
```

#### Get Document

```typescript
interface {
    path: string
    documentCache: boolean
    cacheTTL?: number
    outputPath?: string
    resultPath?: string
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| path | string | undefined | ✅ | CoDB document path to retrieve data. It should start with a slash "/". Even number of folders in paths represent documents. |
| documentCache | boolean | false | ❌ | Flag to retrieve document from cache. |
| cacheTTL | number | undefined | ❌ | The amount of time in seconds to keep the data in cache. It must be between 60 and 86400. |
| outputPath | string | $.0.response.data | ❌ | Path to resolve document response into output. |
| resultPath | string | undefined | ✅ | Variable name to store output into state. |

```typescript
class GetDocumentExample {
    id: 'GET_FROM_CODB'
    path: '/collectionA/document1'
    documentCache: true
    cacheTTL: 900
    resultPath: 'getDocumentResult'
}
```

#### List Documents

```typescript
interface {
    path: string
    limit?: number
    token?: string
    resultPath?: string
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| path | string | undefined | ✅ | CoDB collection path to retrieve list of its document IDs. It should start with a slash "/". Even number of folders in paths represent documents. |
| limit | number | 20 | ❌ | The amount of documents to retrieve at once. |
| token | string | undefined | ❌ | Pagination token. |
| resultPath | string | undefined | ✅ | Variable name to store output into state. |

```typescript
class ListDocumentsExample {
    id: 'LIST_CODB_DOCUMENTS'
    path: '/collectionA/document1'
    limit: 100
    token: '$.listDocumentsResult._paginationToken'
    resultPath: 'listDocumentsResult'
}
```

#### Delete Document

```typescript
interface {
    path: string
    limit?: number
    token?: string
    resultPath?: string
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| path | string | undefined | ✅ | CoDB collection path to retrieve list of its document IDs. It should start with a slash "/". Even number of folders in paths represent documents. |
| resultPath | string | undefined | ❌ | Variable name to store output into state. |

```typescript
class ListDocumentsExample {
    id: 'DELETE_CODB_DOCUMENT'
    path: '/collectionA/document1'
    resultPath: 'deleteDocumentResult'
}
```

### Working With RBS Services

RBS has various micro services which are available through middleware (RBS Core).
A process can interact with those services in two ways: synchronous and asynchronous.

In synchronous way, service responds with the answer directly. Process Manager evaluates the response according to your attributes.
In asynchronous way, service responds with a simple success, then sends the actual answer through callback actions to complete the request as successful or failure.
At first, Process Manager checks if the service got the request successfully, then awaits for asynchronous callback to evaluate the response according to your attributes.

#### Synchronous Service Call

```typescript
interface {
    inputPath?: string
    parameters?: object
    each?: string
    action: string
    service?: string
    timeout?: { seconds: number, jump: string }
    retry?: { backoffRate?: number, interval?: number, maxAttempts?: number, errors?: string[] }
    headers?: { processId: string, executionId: string }
    auth?: { userId?: string, token?: string, roles?: string[], claims: object }
    outputPath?: string
    resultPath?: string
    errorPath?: string
    actionRollback?: string
    rollbackOnFailure?: boolean
    plain?: boolean
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| inputPath | string | $ | ❌ | Path to input parameters. Make sure it starts with a dollar sign "$". Default value is "$". If you want to use a custom input, use "parameters" instead. |
| parameters | object | undefined | ❌ | Custom parameters. Must be a valid JSON. If you want to send data from state, use "inputPath" instead. |
| each | object | undefined | ❌ | Path to array of "parameters". All parameters will be called via *Promise.all*, then collect all the responses. |
| action | string | undefined | ✅ | RBS action name to call. |
| service | string | undefined | ❌ | Target service id. |
| timeout | object | undefined | ❌ | Timeout in seconds. You should provide an action to jump to when timeout triggered. It's disabled by default. |
| retry | object | undefined | ❌ | Retry policy with exponential backoff support. It's disabled by default. |
| headers | object | undefined | ❌ | Adds processId and executionId headers to request. |
| auth | object | undefined | ❌ | Generates or uses custom user token before calling the service. Be careful! User will have privileges according to these settings. |
| outputPath | string | $ | ❌ | Path to resolve service response into output. |
| resultPath | string | undefined | ❌ | Variable name to store output into state. |
| errorPath | string | undefined | ❌ | Path to resolve service error. |
| rollbackOnFailure | boolean | false | ❌ | Flag for applying rollback. |
| actionRollback | string | undefined | ❌ | RBS action name for rollback. |
| plain | boolean | false | ❌ | Flag for removing process related headers from service calls. |

> You should provide one of these attributes <span style="color:#0074d9">inputPath</span>, <span style="color:#0074d9">parameters</span> or <span style="color:#0074d9">each</span> with <span style="color:#0074d9">action</span> attribute to send a payload in your request.

> If the service return an error when <span style="color:#0074d9">retry</span> is enabled, Process Manager retries the request with an exponential backoff according to your configuration.
Do not forget that timeout and retry might end up a race condition.

> If step fails when <span style="color:#0074d9">rollbackOnFailure</span> is enabled, Process Manager calls every <span style="color:#0074d9">actionRollback</span> executed before failed step.

> If you want to call another process or you don't want to send metadata of current process in a service call, you should set <span style="color:#0074d9">plain</span> as true. Default value for *plain* is false.

> If you want to send metadata of a process in a service call, you should set processId and executionId in <span style="color:#0074d9">headers</span> attribute. 
Main purpose of this feature is that returning output of a subprocess to parent process.
To accomplish that, you should also collect parent processId and executionId as initial payload to be able to return them back.

> By default a service call contains a service token created by Process Manager.
To be able to use process' metadata on service side, there are some extra headers automatically passed to the service like *processId*, *processExecutionId*, *processExecutorId* and *processExecutorRole*.
On the other hand, you can make a service call on user's behalf by using <span style="color:#0074d9">auth</span> attribute.
This feature has two ways of usages. In one of them, you should generate a valid custom token for a specific user and use it in *auth* attribute with <span style="color:#0074d9">token</span> parameter. Otherwise you should provide <span style="color:#0074d9">userId</span>, <span style="color:#0074d9">roles</span> and <span style="color:#0074d9">claims</span> to generate a token before making the request.

```typescript
class SynchronousServiceCallExample {
    id: 'GET_ORDER'
    action: 'rbs.order.request.DATA'
    parameters: { orderId: '$.payload.orderId' }
    outputPath: '$.0.response'
    resultPath: 'data'
    errorPath: '$.0.errors.0'
}
```

#### Asynchronous Service Call

The only difference between synchronous and asynchronous service calls are callback attributes.

```typescript
interface {
    actionCallback?: string
    actionFailureCallback?: string
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| actionCallback | string | undefined | ✅ | Callback action for successful requests. |
| actionFailureCallback | string | undefined | ❌ | Callback action for failed requests. |

> If you need an extra information in a step other than the first step, you can use <span style="color:#0074d9">actionCallback</span> without providing an action. In that case, instead of making a service call, Process Manager awaits for an asynchronous response from a client instance.

```typescript
class AsynchronousServiceCallExample {
    id: 'PAYMENT'
    action: 'rbs.billing.request.PAY'
    inputPath: '$.payload.paymentInput'
    actionCallback: 'rbs.billing.event.SUCCESS'
    actionFailureCallback: 'rbs.billing.event.FAILURE'
    outputPath: '$.0.response'
    resultPath: 'payment'
    errorPath: '$.0.errors.0'
}
```

### Working With Other Processes

A process can start other processes as well as interact with them while they are still in progress.
In general programming, working with solution oriented, small and maintainable classes or functions is a best practice.
That works well in process development too.

> We recommend to develop processes to solve one problem at a time.

```typescript
interface {
    process: string
    invocationType: string
    inputPath?: string
    parameters?: object
    executionId?: string
    outputPath?: string
    resultPath?: string
    errorPath?: string
}
```

| Attribute Name | Type | Default | Must | Description |
| -------------- | ---- | ------- | ---- | ----------- |
| process | string | undefined | ✅ | Process ID to start. |
| invocationType | string | NORMAL | ✅ | Running mode for execution. Available options are *NORMAL*, *EXPRESS*, *EXPRESS_CACHED*. |
| inputPath | string | $ | ❌ | Path to input parameters. Make sure it starts with a dollar sign "$". Default value is "$". If you want to use a custom input, use "parameters" instead. |
| parameters | object | undefined | ❌ | Custom parameters. Must be a valid JSON. If you want to send data from state, use "inputPath" instead. |
| executionId | string | undefined | ❌ | Execution ID to prevent PM assigning an auto generated ID. |
| outputPath | string | $.0.response | ❌ | Path to resolve service response into output. |
| resultPath | string | undefined | ❌ | Variable name to store output into state. |
| errorPath | string | undefined | ❌ | Path to resolve service error. |

```typescript
class StartProcessExample {
    id: 'START_PROCESS'
    process: 'TEST_PROCESS'
    parameters: {}
    outputPath: '$.0.response'
    resultPath: 'startedProcess'
    errorPath: '$.0.errors.0'
}
```

```typescript
class StartAnotherProcessExample {
    id: 'START_PROCESS_IN_EXPRESS_MODE'
    process: 'ANOTHER_TEST_PROCESS'
    startMode: 'EXPRESS'
    parameters: {}
    outputPath: '$.0.response'
    resultPath: 'startedProcess'
    errorPath: '$.0.errors.0'
}
```

> You should provide one of these attributes <span style="color:#0074d9">inputPath</span> or <span style="color:#0074d9">parameters</span> to send an initial payload for the process.

### Native Code Support

Service call

### Third Party Libraries

Service call
