# OpenFGA Groups and Resource Attributes: A Sample Store Tutorial

## Overview

In this tutorial, we will explore how to create and use group-based access controls in OpenFGA by incorporating resource attributes. The example shows how members of specific groups, like 'marketing' or 'content,' can access documents based on their status (e.g., 'draft' or 'published'). This enables you to fine-tune access control rules, ensuring different teams can only interact with documents that match their roles.

By the end of this tutorial, you will have an OpenFGA model that controls document access based on group membership and document attributes, and you will be able to test your setup using the FGA CLI.

## Use Case Scenario

Imagine a scenario where members of the 'marketing' team are only allowed to view documents that are 'published,' while the 'content' team members can also view 'draft' documents. This ensures that sensitive drafts are only visible to those responsible for content creation, while the marketing team sees only final, ready-to-go material.

We’ll demonstrate this scenario using the OpenFGA model and condition logic to define who can access what.

## Setting Up the Model

To implement this, we need to define the following:

- **Users**: Individual members of groups (e.g., Anne and Bob).
- **Groups**: Such as 'marketing' and 'content.'
- **Documents**: Which can have attributes like 'status.'
- **Conditions**: Rules to control document access based on attributes like `status`.

### The OpenFGA Model

```yaml
model: |
  model
    schema 1.1

  type user

  type organization
    relations
      define member: [user]
      define can_access_docs: [group#member with doc_viewer_condition]

  type group
    relations
      define organization: [organization]
      define member: [user]

  type document
    relations
      define organization: [organization]
      define can_access: can_access_docs from organization

  condition doc_viewer_condition(document_attributes: map<string>, allowed_statuses: list<string>) {
    document_attributes["status"] in allowed_statuses 
  }
```

Here’s a breakdown of the model:
- **User Type**: Represents the users in the system.
- **Group Type**: Defines different groups like 'marketing' and 'content.' Each group is linked to an organization.
- **Document Type**: Represents a document, which can have attributes such as `status`. Access to documents is controlled by the `can_access` relation and the `doc_viewer_condition` function, which checks whether the document's `status` is in the allowed list.

### Defining Tuples

Tuples define relationships between users, groups, and documents. Below are the tuples for this example:

```yaml
tuples:
  - user: "organization:acme"
    relation: "organization"
    object: "document:1"

  - user: "user:anne"
    relation: "member"
    object: "group:content"
  - user: "user:bob"
    relation: "member"
    object: "group:marketing"
    
  - user: "group:content#member"
    relation: "can_access_docs"
    object: "organization:acme"
    condition: 
      name: doc_viewer_condition
      context: 
        allowed_statuses : ["draft", "published"]

  - user: "group:marketing#member"
    relation: "can_access_docs"
    object: "organization:acme"
    condition: 
      name: doc_viewer_condition
      context: 
        allowed_statuses : ["published"]
```

In this setup:
- Anne is a member of the 'content' group, giving her access to both 'draft' and 'published' documents.
- Bob is a member of the 'marketing' group, allowing him to only view 'published' documents.

### Testing the Model

Now, let’s test the model to confirm that the access rules work as expected.

1. First, ensure that you have the [OpenFGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation) installed on your system.

2. In your terminal, navigate to the `group-resource-attributes` directory and run the following command:

   ```bash
   fga model test --tests store.fga.yaml
   ```

   This command will test the rules defined in `store.fga.yaml` and check if users have the correct access based on the document attributes.

### Expected Test Results

Below is an example of the expected test results:

```yaml
tests:
  - check:
    - user: "user:anne"
      object: "document:1"
      context: 
        document_attributes :
          status: "draft"
      assertions:
        can_access: true

    - user: "user:anne"
      object: "document:1"
      context: 
        document_attributes :
          status: "published"
      assertions:
        can_access: true

    - user: "user:bob"
      object: "document:1"
      context: 
        document_attributes :
          status: "draft"
      assertions:
        can_access: false
        
    - user: "user:bob"
      object: "document:1"
      context: 
        document_attributes :
          status: "published"
      assertions:
        can_access: true
```

In this test:
- Anne has access to both 'draft' and 'published' documents.
- Bob only has access to 'published' documents, and attempting to view a 'draft' document will result in a failed access check.

## Wrapping Up

By following this tutorial, you’ve successfully implemented a group-based access control system with resource attributes in OpenFGA. You've also tested the model to ensure that your access rules behave as expected.

This is just one way to define access control policies using OpenFGA, and the flexibility of resource attributes allows you to extend this to other use cases like document versions, roles, and more. You can continue experimenting with different conditions or explore more advanced features of OpenFGA by referring to the [OpenFGA documentation](https://openfga.dev/docs/).

