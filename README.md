# RBS Documentation.

Check [wiki](https://github.com/rettersoft/rbs-docs/wiki) for documentation.

### Rest Calls

Rest folder contains all necessary request files to make rbs calls to RBS services. To execute calls [VS Code Rest Client extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) should be installed.

### Example VS Code Environment File

```
"rest-client.environmentVariables": {
    "beta": {
        "host": "core-test.rettermobile.com",
        "projectId": "933a51e1c87a9ccc181d21fca91c2aad",
        "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6InJicy5idXNpbmVzc3VzZXJhdXRoLmFkbWluIiwiYW5vbnltb3VzIjpmYWxzZSwicHJvamVjdElkIjoiOTMzYTUxZTFjODdhOWNjYzE4MWQyMWZjYTkxYzJhYWQiLCJ1c2VySWQiOiJlbWFpbEB0ZXN0LmNvbSIsInRpbWVzdGFtcCI6MTYwODEyOTI2NzI5OSwiaWF0IjoxNjA4MTI5MjY3LCJleHAiOjE2MTgxMjkyNjd9.KfXN9CINUMG8vGNeVEAKBINwQ9FQLyLCcL7aKhdTSwY"
    }
}
```

This json should go into your VS Code settings.json file. 

[VS Code Settings](https://code.visualstudio.com/docs/getstarted/settings) 