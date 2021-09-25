# Process Manager <!-- {docsify-ignore} -->

## Best Practices <!-- {docsify-ignore} -->

### Working With Responses

It's common to fork a path in a process by response of a service call.
While you're doing that, you should consider what you will put into *state* to use in your conditional flows.

```typescript
class GetOrderNotRecommendedWay {
    id: 'GET_ORDER'
    action: 'rbs.order.request.DATA'
    service: 'rbs.oms'
    parameters: { orderId: '$.payload.orderId' }
    outputPath: '$.0'
    resultPath: 'order'
    errorPath: '$.0.errors.0'
    onSuccess: { 'order.status': { EQ: 200 } }
}
```

Step definition above, tries to get details of an order from OMS via a synchronous service call.
As always, RBS Core middleware responds with an array and each item includes the original response from service as well as some metadata about HTTP communication between Process Manager and service such as headers, status, etc.

If you save *$.0* into state that means you will have more data than you actually need for your implementation.
Instead of that, you should consider saving *$.0.response* directly.

```typescript
class GetOrder {
    id: 'GET_ORDER'
    action: 'rbs.order.request.DATA'
    service: 'rbs.oms'
    parameters: { orderId: '$.payload.orderId' }
    outputPath: '$.0.response'
    resultPath: 'order'
    errorPath: '$.0.errors.0'
    onSuccess: { 'order.orderId': { EX: true } }
}
```

### Designing Loops

You can design a loop just like *for* in general programming. That kind of loops require a variable to track steps.

```typescript
class GetOrder {
    id: 'GET_ORDER'
    action: 'rbs.order.request.DATA'
    service: 'rbs.oms'
    parameters: { orderId: '$.payload.orderId' }
    outputPath: '$.0.response'
    resultPath: 'order'
    errorPath: '$.0.errors.0'
    onSuccess: { 'order.orderId': { EX: true } }
}

class Loop {
    id: 'LOOP'
    st: '{{ retry + 1 }}'
    resultPath: 'retry'
    isCompleted: { retry: { GT: 3 } }
}
```

> If <span style="color: #0074d9">GetOrder</span> fails, process should continue with <span style="color: #0074d9">Loop</span> step until the current value of <span style="color: #0074d9">retry</span> is greater than *3*. In initial state or one of the earlier steps of process, we should set <span style="color: #0074d9">retry</span> as *0*.

> It's also possible to implement a loop by consuming an array or object. But it's much more complicated than *for*-like loops, requires more steps and more open to end up with an infinite loop accidentally. You should consider risk of malfunction or higher cost if you decide to implement *while*-like loops. You can see how to implement that kind of loop in Typescript below.

```typescript
const items = [ 1, 2, 3, 4, 5 ];
let sum = 0;
while (items.length) {
    const n = items.pop();
    if (typeof n === 'number') sum += n;
}
console.log('sum: ', sum)
```
