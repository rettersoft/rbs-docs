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
interface CommonAttributes {
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

| Attribute Name |  Type  |  Default  | Must | Description |
| -------------- | ------ | --------- | ---- | ----------- |
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

### Put Document

```typescript
{
    path: string
    document: string
    documentTTL?: number
    outputPath?: string
    resultPath?: string
    errorPath?: string
}
```

| Attribute Name |  Type  |  Default  | Must | Description |
| -------------- | ------ | --------- | ---- | ----------- |
| path | string | undefined | ✅ | CoDB document path to save data. It should start with a slash "/". Even number of folders in paths represent documents. |
| document | string | undefined | ✅ | Path to data. |
| documentTTL | string | undefined | ❌ | The amount of time in seconds to keep the data. |
| outputPath | string | $ | ❌ | Path to resolve document response into output. |
| resultPath | string | undefined | ❌ | Variable name to store output into state. |
| errorPath | string | undefined | ❌ | Path to resolve document error. |

### Working With RBS Services

Service call

### Working With Other Processes

Service call

### Native Code Support

Service call

### Third Party Libraries

Service call
