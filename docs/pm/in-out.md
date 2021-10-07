# Process Manager <!-- {docsify-ignore} -->

## Inputs & Outputs <!-- {docsify-ignore} -->

A process starts with an optional payload, then executes step by step sequentially.
You can interact with the client (caller) on any step you want again.
A process might also have multiple outputs depend on conditional flows.
For example; some paths end up with error messages, while others end up with some result data.

Each executed path will be reported in process mining separately with possible errors and extra information (*info1*, *info2* and *info3*).

### Dynamic Paths

State is available for all steps.
At first, it contains only client's initial payload with some metadata such as processId, executionId, details of token, etc.
Each step can manipulate (add, update, remove) any key on the state.
To access a key in the state, you should declare a path starting with a dollar ($) sign.

> <span style="color:#0074d9">$.payload.phoneNumber</span> returns the phone number that has been sent by client as initial payload.

If you want to save value of a dynamic path into state or send them to another service, you should declare the key has a dynamic value by ending the name with a dollar ($) sign.

```json
{
    "dynamic_key.$": "$.payload.originalValue",
    "static_key": "Some Static Value"
}
```

To access any element of an array in a dynamic path, you should add index of the element into your path declaration.

```json
{
    "keyFromFirstItem.$": "$.payload.itemsArray.0.someKey"
}
```

### Directives

Besides dynamic paths, a process can also use directives to work with dynamic values.
Process Manager has two kinds of directives: variables like *$$.TIMESTAMP*, *$$.USER* and functions like *$$.CONCAT()*, *$$.OR()*

#### $$

Returns all of the current state.

#### $$.USER_ID or $$.USER

Both variables returns caller user's ID.

#### $$.TIMESTAMP

Returns current timestamp in milliseconds.

#### $$.ULID

Returns a sortable ULID.

#### $$.UUID

Returns a UUID.

#### $$.APPEND(item: any, target?: any[])

Appends a new item into target array. If you don't provide a target, it creates a new array and appends the item into it.

```typescript
{
    "key.$" : "$$.APPEND($.anItemSomewhereInState, $.anArraySomewhereInState)"
}
```

#### $$.BASE64(input: string)

Converts an input to base64 string.

```typescript
{
    "converted.$" : "$$.BASE64($.aKeySomewhereInState)"
}
```

#### $$.BCRYPT(input: string, salt: number)

Returns hash of an input in bcrypt format.

```typescript
{
    "hash.$" : "$$.BCRYPT($.payload.password, 10)"
}
```

#### $$.BCRYPT_COMPARE(input: string, hash: string)

Checks if input matches with provided bcrypt hash.

```typescript
{
    "isOK.$" : "$$.BCRYPT_COMPARE($.payload.password, $.hash)"
}
```

#### $$.CONCAT(separator: string, ...items: any[])

Returns parameters in a form concatenated by separator.
If you want to concatenate arrays, you should pass *ARRAY!* as separator, then provide more than one arrays to combine them into a single array.

```typescript
{
    "strValue.$" : "$$.CONCAT(' ', 'Name:', $.payload.name)"
}
```

```typescript
{
    "arrValue.$" : "$$.CONCAT('ARRAY!', $.anArraySomewhereInState, $.anotherArraySomewhereInState)"
}
```

#### $$.DATE(method: string, d: string | number | Date, ...params: any[])

Date directive is simplified version of date component under third party libraries. It uses [date-fns](https://date-fns.org) under the hood.
While you can chain multiple methods in date component, you can only call a single method here at once.

```typescript
{
    "year.$": "$$.DATE('format', $.payload.createdAt, 'yyyy')"
}
```

```typescript
{
    "nextDay.$": "$$.DATE('addDay', $.payload.createdAt, 1)"
}
```

#### $$.LENGTH(input: string | any[])

Returns the length of input. If input is string, returns character length. Otherwise it returns number of elements in the array.

#### $$.MERGE(...items: object[]) or $$.SPREAD(...items: object[])

Combines all objects in provided order and returns a single object.

```typescript
{
    "merged.$": "$$.MERGE('addDay', $.anObjectSomewhereInState, $.anotherObjectSomewhereInState)"
}
```

#### $$.MD5()

Returns hash of an input in md5 format.

```typescript
{
    "hash.$": "$$.MD5($.aStringSomewhereInState)"
}
```

#### $$.MODE(input: string, shardCount?: number)

Calculates which shard the input should go into. Default *shardCount* is 100.

```typescript
{
    "shardNo.$": "$$.MODE($.aStringSomewhereInState, 5)"
}
```

#### $$.OR(...items: any[])

Returns first *defined* parameter.

```typescript
{
    "name.$": "$$.OR($.aPathToUndefinedValue, $.anotherPathToUndefinedValue, $.payload.name)"
}
```

```typescript
{
    "name.$": "$$.OR($.aPathToUndefinedValue, $.anotherPathToUndefinedValue, 'Name')"
}
```

#### $$.POP()

Returns last element of an array.

#### $$.RANDOM(len?: number, numbersOnly?: boolean)

Returns a random string in length of *len* parameter. Default value for len parameter is 8.
It uses only capital letters and numbers to generate the random string.

#### $$.REPLACE(input: string, pathToContext: string) or $$.VTL(input: string, pathToContext: string)

Returns context-applied form of a string template.

> Assume that we have an initial state like this: *{ template: 'Hello {name}!', context: { name: 'John', surname: 'Doe' } }*

```typescript
{
    "replacedText.$": "$$.REPLACE($.template, $.context)"
}
```

> At the end, *replacedText* ends up being "Hello John!".

#### $$.SPLIT(input: string, separator?: string)

Parses a string by separator and returns the array back. Default separator is an empty string.

```typescript
{
    "parsed.$": "$$.SPLIT($.aStringSomewhereInState, ',')"
}
```

### Conditional Flows

Moving one step to another is possible in two ways: *conditional* and *default*. You can consider these like *if*, *else if* and *else* statements.
Process Manager will try conditions one by one. If there is a match, its target step will be executed. Otherwise, default flow will be executed.

All conditional expressions use [Rule Engine](https://github.com/rettersoft/rbs-rule-engine) under the hood.
By default, rule engine can resolve attributes from state. So, you don't have to provide dollar ($) sign on left part of your condition.
On right part which is the value part, you can use dynamic paths as well as you can use constant values.

```typescript
{
    "payload.username": { "EX": true }
}
```

```typescript
{
    "serviceResult.status": { "EQ": 200 }
}
```

```typescript
{
    "payload.password": { "EQ": "$.passwordSomewhereInState" }
}
```
