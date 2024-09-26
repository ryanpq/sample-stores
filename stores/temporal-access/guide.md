# OpenFGA Temporal Access Sample Store

## Introduction

In many applications, you might need to grant users access to a document or resource, but only for a limited period of time. Whether it's time-limited access to sensitive information or temporary collaboration rights, ensuring that access is revoked when it's no longer valid is critical. 

In this guide, we'll demonstrate how you can use OpenFGA to implement **temporal access control**—restricting access to a document based on a specific time window—using **conditional relationship tuples**. OpenFGA allows you to define and enforce dynamic access policies, making it a great fit for this kind of scenario.

We'll walk through setting up a simple example that grants users access to a document based on time and how you can test it using OpenFGA’s CLI.

## Prerequisites

Before we dive in, make sure you have the OpenFGA CLI installed. You can install it by following the instructions in the [OpenFGA CLI installation guide](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

You’ll also need a basic understanding of OpenFGA models and relationship tuples. If you're new to OpenFGA, check out the [official docs](https://openfga.dev/docs/).

## Use Case: Temporal Access to Documents

In this example, we need to grant users access to a document, but only within a certain timeframe. To achieve this, we’ll use **conditional relationship tuples**—a feature in OpenFGA that allows you to define relationships with conditions such as time limits.

We’ll model a simple document access control where:
- Some users have indefinite access.
- Others have time-limited access, based on a specific start time and duration.

We’ll define the model and tuples in `store.fga.yaml`, along with some unit tests to verify everything works as expected.

## Setting Up the Model

In the `temporal-access` directory, create a file called `store.fga.yaml` with the following content:

```yaml
model: |
  model
    schema 1.1

  type user

  type document
    relations
      # Defining the 'viewer' relation
      define viewer: [user, user with non_expired_grant]

  # A condition that restricts access to a user based on time
  condition non_expired_grant(current_time: timestamp, grant_time: timestamp, grant_duration: duration) {
     current_time < grant_time + grant_duration
  }

# Define relationship tuples to specify who can access what
tuples:
    - user: user:bob
      relation: viewer
      object: document:1

    - user: user:anne
      relation: viewer
      object: document:1
      condition: 
        name: non_expired_grant
        context: 
          grant_time : "2023-01-01T00:00:00Z"
          grant_duration : 1h

    - user: user:anne
      relation: viewer
      object: document:2
      condition: 
        name: non_expired_grant
        context: 
          grant_time : "2023-01-01T00:00:00Z"
          grant_duration : 5s
```

Here’s a breakdown of the key components:
- **User and Document Types**: We define two types, `user` and `document`.
- **Viewer Relation**: The `viewer` relation defines who can view the document. It includes a condition for users with a non-expired grant.
- **Time Condition**: The `non_expired_grant` condition checks that the current time is within the valid time window for access.

## Testing the Temporal Access

To test our temporal access model, we'll use the FGA CLI to run unit tests against our `store.fga.yaml` file.

### Run the Tests

1. Open your terminal in the `temporal-access` directory.
2. Run the following command to execute the tests:
   ```bash
   fga model test --tests store.fga.yaml
   ```

This will run a series of tests to verify the temporal access logic. Let’s break down the tests we’re running.

### Example Tests

```yaml
tests:
  - name: Test temporal access
    check:
    - user: user:anne
      object: document:1
      context: 
        current_time: "2023-01-01T00:10:00Z"
      assertions:
        viewer: true

    - user: user:anne
      object: document:1
      context: 
        current_time: "2023-01-01T02:00:00Z"
      assertions:
        viewer: false

    - user: user:anne
      object: document:2
      context: 
        current_time: "2023-01-01T00:00:09Z"
      assertions:
        viewer: false

    - user: user:bob
      object: document:1
      assertions:
        viewer: true
```

- **First Test**: Anne has access to `document:1` at `00:10:00` but loses access at `02:00:00` since the grant duration is only one hour.
- **Second Test**: Anne’s access to `document:2` expires after 5 seconds, so the assertion is `false`.
- **Third Test**: Bob has indefinite access to `document:1`, so the assertion is always `true`.

### Additional Tests

You can also test which documents a user can view or which users can view a specific document:

```yaml
  - name: Test the documents that anne can view
    list_objects:
      - user: user:anne
        type: document
        context: 
          current_time: "2023-01-01T00:00:01Z"
        assertions:
          viewer: 
            - document:1
            - document:2

  - name: Test the users that can view document:1
    list_users:
      - object: document:1
        context: 
          current_time: "2023-01-01T00:00:01Z"
        user_filter:
          - type: user
        assertions:
          viewer:
            users: 
              - user:anne
              - user:bob
```

These tests help verify not only individual access but also who can access multiple documents at specific times.

## Next Steps

Congratulations! You’ve successfully implemented and tested temporal access control using OpenFGA. Now that you’ve seen how powerful conditional relationship tuples can be, try experimenting with other conditions or more complex use cases. 

For more examples and advanced scenarios, check out the [OpenFGA documentation](https://openfga.dev/docs/).
