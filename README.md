# Waveorb model

We run into the problem that we need to validate database operations outside of API actions, for instance scripts. We also need defaults for database gets and finds sometimes, and bigger operations like cascading deletes. Having an ORM, or something similar, might help organize the code better and allow a cleaner interface.

In our actions, we have patterns like this:

```js
await $.validate({
  query: {
    id: {
      required: true,
      is: 'id'
    }
  },
  values: {
    name: {
      is: 'string'
    }
  }
})

var { query, values } = $.params

// Set up defaults
if (typeof values.name != 'undefined') {
  values.md5 = $.tools.md5(values.name)
}

// Get
var result = await db('project').get(query)
return result

// Find
var result = await db('project').find(query)
return result

// Count
var result = await db('project').count(query)
return result

// Create
var result = await db('project').create(values)
return result

// Update
var result = await db('project').update(query, values)
return { ok: 1 }

// Delete
var result = await db('project').delete(query)
return { ok: 1 }
```

We want a better interface for this. The idea is to use classes and move validations into the model. We want a version that throws so we can return validation errors from actions, and another version that doesn't throw so we can use it in scripts without try catch? Or is it reasonable to use try catch, or even promise-catch?

The models are defined in `app/models` and are pascal-cased classes which extends the waveorb-model class, which in turn speaks to the database. They can be mapped to a database collection, or operate on several collections.

We would want to use them like this in API actions:

```js
// Simple
var project = new app.models.project({ query, values })

// Global
var project = new Project({ query, values })

// Instance methods
var result = await project.find()
var result = await project.get()
var result = await project.count()
var result = await project.create()
var result = await project.update()

// Specialized
var result = await project.findSuperAdmins()

// Or static if we are using classes
var result = await Project.findSuperAdmins($)
```

What do we do with validations? Separate call? Or part of instance methods?

This is what we could do in API actions:
```js
// Maps to functions in app/validations
await $.validations(['project-create'])

// await $.validate(...)
```

This can be done in scripts, where `$` doesn't exist:

```js
var { validate } = require('waveorb')

// or...
var validate = require('waveorb/validate')

await app.validations.projectCreate({ query, values })

// or through validate functions on the model
var project = new Project({ query, values })

// Returns 'true' or 'false'
var validated = await project.update({ validate: true })
```

### Traditional ORM

We could also get rid of this `query` and `values` thing:

```js
// Find by id
var project = Project.get(query.id)
var project = Project.first(query.id)
var project = Project.last(query.id)
var project = Project.get(query)

await project.update(values)
await project.delete(query)

var project = new Project(values)
await project.validate()

await project.validateOnlySomeStuff()
```

### Without classes or only static methods

```js
var project = Project.get(query)

await Project.update(query, values)
await Project.delete(query)

// This throws
await Project.validateOnCreate(values)

await Project.validateOnUpdate(query, values)

// Save immediately
var project = await Project.create(values)
```

This latter one is almost the same as what we have now:

```js
var project = await db('project').create(values)

var project = await Project.create(values)

var project = await Project.create(values)
```


Alternative syntax:
```js
var project = await Project($).validate().create()

// Same as
var project = Project($)
await project.validate()
await project.create()
```

The difference is that we can bake in defaults like sorting
