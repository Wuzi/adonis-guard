# Adonis Guard 🔰

This package is an **authorization provider** built on top of [@slynova/fence](https://github.com/Slynova-Org/fence).

## Getting Started

Install the package using the `adonis` CLI.

```bash
> adonis install adonis-guard
```

Follow instruction that are displayed ([or read them here](https://github.com/RomainLanz/adonis-guard/blob/master/instructions.md)).

## Defining your authorization

### Gate
Gates must be defined inside the `start/acl.js` file. This file will be loaded only once when the server is launch.
To define a gate, use the `Gate` facade.

```js
// start/acl.js
const Gate = use('Gate')

Gate.define('gateName', (user, resource) => {
  // Payload
  // e.g. return user.id === resource.author_id
})
```

### Policy
You can generate a new policy by using the command `adonis make:policy {name}`.
This will generate a file in `app/Policies/{Name}Policy.js`.
To attach a policy to a resource, you need to call the `policy` method of the `Gate` facade.

```js
// start/acl.js
const Gate = use('Gate')

Gate.policy('App/Models/MyResource', 'App/Policies/MyPolicy')
```

## Usage

Adonis Guard automaticaly share an instance of the `guard` in the context of each request.
To validate the authorization of a user you simply need to extract it from the context and run the gate/policy.

```js
// Controller
async show ({ guard, params }) {
  const post = await Post.find(params.id)

  if (guard.denies('show', post)) {
    // abort 401
  }

  // ...
}
```

```js
// RouteValidator
async authorize () {
  const post = await Post.find(this.ctx.params.id)

  if (this.ctx.guard.denies('show', post)) {
    // abort 401
  }

  // ...
}
```

You can also use it in your view to choose to display or not an element.

```html
@if(guard.allows('edit', post))
  <a href="/posts/{{ post.id }}/edit">Edit</a>
@endif

@can('edit', post)
  <a href="/posts/{{ post.id }}/edit">Edit</a>
@endcan

@cannot('edit', post)
  <p>Not allowed!</p>
@endcannot
```

The `@can` and `@cannot` tags have the same signature as `guard.allows()` and `guard.denies()`.

You can also use the middleware `can` in your route.<br>
Notice that this middleware doesn't work with resource. It will execute a gate with the loggedIn user only.

```js
Route.get('/admin/posts', 'Admin/PostController.index')
  .middleware('can:viewAdminPosts')
```

A second argument can be supplied that will replace a resource in your gate. This is useful when you want to have dynamic route rules.

```js
Route.get('/admin/posts', 'Admin/PostController.index')
  .middleware('can:hasRole,admin,editor')
```

`admin,editor` will be extracted into an array that you can retrieve as the second parameter in your gate.

**Public API**

```js
guard.allows('gateName/Policy Method', resource) // It will use per default the authenticated user or return false if not authenticated
guard.denies('gateName/Policy Method', resource) // It will use per default the authenticated user or return true if not authenticated
guard.allows('gateName/Policy Method', resource, user)
guard.denies('gateName/Policy Method', resource, user)
guard.can(user).pass('gateName').for(resource)
guard.can(user).callPolicy('Policy Method', resource)
```
