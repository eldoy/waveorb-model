# Waveorb model

The proposed way of using database models in Waveorb.

```js
var project = await app.models.project.create(values)

// Throws, like before, only in APIs
await $.validate({})

// Add this to run validations, supports multiple
await $.validations(['createProject'])
await $.validations('createProject')

// Calling the validation directly doesn't throw, like from a script
var validation = await app.validations.createProject(values)
if (!validation.errors) {
  var project = await app.models.project.create(values)
} else {
  // Handle errors
  validation.errors
}
```
