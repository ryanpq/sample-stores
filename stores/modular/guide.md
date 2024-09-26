# Organizing Permissions with Modular Models in OpenFGA

In this guide, we’ll explore how you can use [OpenFGA’s modular models](https://openfga.dev/modeling/modular-models) to organize your permissions across different parts of your application—making it easier to manage complex permissions in a scalable way. We'll look at an example that organizes a simple app into **three modules**: a core, a wiki, and an issue tracker.

If you’ve ever dealt with complex permissions in an app, you know how tricky it can be to manage roles, relationships, and access. Modular models make it easier by allowing you to split your permission logic across different files, keeping everything neat and organized.

## The Modular Model Example

For this example, we’ll use three modules:

- The **core module** handles basic types like users, groups, and organizations, which are shared across the app.
- The **wiki module** adds the entity types for managing wiki spaces and pages.
- The **issue tracker module** focuses on project and ticket management, broken down into two files—one for [projects](./issue-tracker/projects.fga) and another for [tickets](./issue-tracker/tickets.fga).

All these pieces are brought together by an [fga.mod](./fga.mod) file, which acts as a manifest, declaring all the modules. You can write this model to your OpenFGA store using the command:

```bash
fga model write --file fga.mod --store-id <store_id>
```

Once written, you can retrieve the model as a single combined file (with module annotations) using:

```bash
fga model get --format fga --store-id <store_id>
```

Here’s an example of how OpenFGA presents the combined model:

```dsl
model
  schema 1.2

type group # module: core, file: core.fga
  relations
    define member: [user]

type organization # module: core, file: core.fga
  relations
    define admin: [user]
    define member: [user] or admin
    define can_create_project: admin # extended by: module: issue-tracker, file: issue-tracker/projects.fga
    define can_create_space: admin # extended by: module: wiki, file: wiki.fga

type project # module: issue-tracker, file: issue-tracker/projects.fga
  relations
    define organization: [organization]
    define viewer: member from organization
```

As you can see, this model combines data from multiple files but still keeps track of where each piece came from. You get the benefit of a clean, modular structure, without losing the context of your entities. 

## How to Try It Out

Ready to give it a try? Here’s what you need to do:

1. **Install the FGA CLI**: If you haven’t already, download and install the [OpenFGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

2. **Run the tests**: Navigate to the `modular` directory and run the following command to test the model against predefined rules in `store.fga.yaml`:

```bash
fga model test --tests store.fga.yaml
```

## Breaking Down the Modules

Let’s break down the modular components a bit more.

### The Core Module

The core module is the foundation. It defines the basic entities like `user`, `organization`, and `group`. These are shared across all parts of the app.

Here’s the code for the `core.fga` file:

```dsl
module core

type user

type organization
  relations
    define member: [user] or admin
    define admin: [user]

type group
  relations
    define member: [user]
```

### The Wiki Module

In this module, we extend the organization to include `can_create_space`, which allows an admin to create wiki spaces. It also defines how spaces and pages relate to users and organizations.

Here’s the `wiki.fga` file:

```dsl
module wiki

extend type organization
  relations
    define can_create_space: admin

type space
  relations
    define organization: [organization]
    define can_view_pages: member from organization

type page
  relations
    define space: [space]
    define owner: [user]
```

### The Issue Tracker Module

The issue tracker module is split into two parts: projects and tickets. In this module, we define permissions around creating projects and viewing tickets.

Here’s the file for projects:

```dsl
module issue-tracker

extend type organization
  relations
    define can_create_project: admin

type project
  relations
    define organization: [organization]
    define viewer: member from organization
```

And here’s the file for tickets:

```dsl
module issue-tracker

type ticket
  relations
    define project: [project]
    define owner: [user]
```

## Unit Tests for the Model

The model, its tuples, and tests are all defined in the [store.fga.yaml](./store.fga.yaml) file. Here’s a quick look at how we test the model:

```yaml
tests:
  - name: Members can view projects
    check:
      - user: user:anne
        object: organization:openfga
        assertions:
          admin: true
          member: true
          can_create_space: true
      - user: user:anne
        object: space:openfga
        assertions:
          can_view_pages: true
      - user: user:anne
        object: project:openfga
        assertions:
          viewer: true
```

## Final Thoughts

This example shows how you can cleanly organize your OpenFGA model using modular files, which is especially useful when your app grows and your permissions become more complex. If you’re building a SaaS platform or managing roles across different app components, modular models are a great way to keep things organized.

Ready to get started? You can explore the full modular model example on GitHub [here](https://github.com/openfga/sample-stores/tree/main/stores/modular).

