# Process Manager <!-- {docsify-ignore} -->

A micro service architecture relies on services to do stuff.
But most of the time there is an orchestration layer.
You can use RBS Process Manager to orchestrate other micro services.

However RBS Process Manager can also be used to create web services, or action handlers for clients.
Without the need of any microservice RBS Process Manager can be used to create a standalone process.
It can use Codb as its data store.

So you can use RBS processes in following ways:

- Orchestrate other micro services
- Create a micro service using processes
- Combine both

## Anatomy of A Process

- A process must contain at least one step and each step must have a unique ID.
- A process can start from a certain step. You can set it via boolean **first** parameter. Otherwise it starts from first-defined step.
- A process has an internal storage unit called **state** which you can write to and read from it through steps.
- A process has also a user-based storage called **userState**.
While *state* is globally available in a single execution, users access their data in *userState* by a valid token.
- In a process, a step cannot call itself.

## API

For more information about Process Manager API please visit documentation section on developer console.

### Actions

You can interact with Process Manager API by calling one of the following actions.

#### rbs.process.request.LIST

Returns list of available processes in your project.
There is no parameter to filter processes yet!

#### rbs.process.request.EXECUTIONS

Returns list of recent executions of a process.

```typescript
interface {
    processId: string
}
```

#### rbs.process.request.START

Starts a new execution in normal mode.
If you don't send executionId, a sortable unique value will be assigned automatically.

```typescript
interface {
    processId: string
    executionId?: string
    payload: { [key: string]: any }
}
```

#### rbs.process.request.START_EXPRESS

Starts a new execution in express mode.
If you want to start an execution and retrieve the final output, please use express mode.
If you want it to be cached for awhile you should call *rbs.process.get.START_EXPRESS* instead.

```typescript
interface {
    processId: string
    executionId?: string
    payload: { [key: string]: any }
}
```

#### rbs.process.request.UPSERT

Updates an existing process or creates a new one if *processId* is not in use.

```typescript
interface  {
    processId: string
    data: any
    roles?: string[]
    state?: object
}
```

#### rbs.process.request.ADD_EVENT

Adds a new trigger event to a process.
When a trigger event fired, a new execution starts automatically.
To accomplish that properly, you should add the event also into receives part of Process Manager's role.

```typescript
interface  {
    processId: string
    event: string
}
```

#### rbs.process.request.REMOVE_EVENT

Removes a trigger event from a process.

```typescript
interface  {
    processId: string
    event: string
}
```

#### rbs.process.request.EVENTS

Returns a list of available trigger events of a process.

```typescript
interface  {
    processId: string
}
```

#### rbs.process.request.GET_EXECUTION

Returns details of an execution and steps the execution passed through.

```typescript
interface {
    processId: string
    executionId: string,
}
```

#### rbs.process.request.STOP

Returns inputs and outputs of a single step in execution of a process.

```typescript
interface {
    processId: string
    executionId: string,
}
```

## FAQ

> What is a process?

A process is a collection of steps each aiming to accomplish a small part of a business logic.

> What is the advantage of a process?

A process is the customization unit of a project. We recommend writing a process for each business function. For example creating an order or adding items to cart can be handled by processes. By using processes for these tasks you can easily customize workflows in the future. Instead of writing native code and implementing these tasks in microservices we recommend using processes.

> Is it possible to run multiple executions of a process?

You can start as many executions as you need, unless you reach project-based rate limits on the middleware.
When you hit the rate limits, you should wait for awhile before starting more processes.

There is also a soft limit on maximum step count which is 999 by default.
This can be updated to 9999.

> Is it possible to run multiple steps at the same time in a process?

Short answer is, no! It is not possible to run multiple steps at the same time in a single execution.

It is also not possible multiple interactions with a single execution of a process.
A process can respond to a single incoming request and proceeds to the next step accordingly.
That makes multiple interactions impossible on a single step.

> How can I start a process?

There are 2 ways of starting a process: normal (*rbs.process.request.START*) and express (*rbs.process.request.START_EXPRESS*) mode.
In normal mode, Process Manager returns details of the newly started execution.
If you want to track the progress of that execution, you should call *rbs.process.request.GET_EXECUTION* action.
In express mode, Process Manager executes all possible steps and responds with the result.
Express mode doesn't support asynchronous operations.
It's not recommended to start a process in express mode if it takes longer than 30 seconds to complete.

Express mode also has a cache (*rbs.process.get.START_EXPRESS*) feature.

> Where is the documentation of a process?

Processes are by their nature documented because they are composed of UI elements. They are created by a drag and drop editor. Even non developers can understand how a process works.

> Is there any way to analyze processes?

You can mine processes in the process mining screen. Process mining gives you insight on what routes are being executed most. It also gives you time based stats on steps.
