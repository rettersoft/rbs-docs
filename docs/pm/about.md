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

> Where is the documentation of a process?

Processes are by their nature documented because they are composed of UI elements. They are created by a drag and drop editor. Even non developers can understand how a process works.

> Is there any way to analyze processes?

You can mine processes in the process mining screen. Process mining gives you insight on what routes are being executed most. It also gives you time based stats on steps.
