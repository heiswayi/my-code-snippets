Example use case: Auto update `access_token` variable

For authentication API endpoint that returns `access_token` in the JSON data, **Scripts** > **Post-response**:
```js
const json_data = pm.response.json();
const context = pm.environment.name ? pm.environment : pm.collectionVariables;
context.set("access_token", json_data.access_token);
```

Then, the `access_token` variable can be used at any other API endpoint:
```
{{access_token}}
```
