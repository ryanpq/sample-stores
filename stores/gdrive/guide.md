# Modeling Google Drive Permissions in OpenFGA: A Hands-On Guide

In this guide, we’ll walk you through how to model Google Drive-style permissions in OpenFGA. You’ll learn how to set up users, groups, folders, and documents, define ownership, and establish permissions like viewing, sharing, and editing files. By the end of this guide, you'll be able to implement a simplified permissions system for a Google Drive-like application.

OpenFGA allows you to model complex authorization scenarios in a clear and manageable way using **relationship-based access control** (ReBAC). Let’s dive in and explore how we can model permissions for Google Drive using OpenFGA!

> **Note**: This example provides a simplified subset of Google Drive’s permissions model. It showcases how OpenFGA handles relationships between users, groups, folders, and documents.

---

## Table of Contents

1. [Use-Case](#use-case)
    - [Requirements](#requirements)
    - [Scenario](#scenario)
    - [Expected Outcomes](#expected-outcomes)
2. [Modeling in OpenFGA](#modeling-in-openfga)
    - [Model](#model)
3. [Try It Out](#try-it-out)

---

## Use-Case

### Requirements

For this example, we are modeling a simplified version of Google Drive. Here are the key concepts and relationships:

- **Users**: Individuals who interact with files and folders.
- **Groups**: Collections of users with shared permissions.
- **Folders**: Contain documents and can have their own permissions and parent folders.
- **Documents**: Stored in folders and can inherit permissions from parent folders.

### Permissions:
- Users can belong to groups.
- **Folders**: Can have owners, viewers, and parent folders.
- **Documents**: Can have owners, viewers, and parent folders.
- **Ownership**: Only owners can share or modify permissions.
- **Viewers**: A user can view a document or folder if they’ve been granted access, if they own it, or if their group has been given access.
- **File creation**: Only folder owners can create new files in that folder.

### Scenario

Let’s look at an example scenario:

1. **Users & Groups**:
    - Anne and Beth are members of the "Contoso" group.
    - Charles belongs to the "Fabrikam" group.

2. **Folders & Documents**:
    - The "Product 2021" folder contains two documents: "Public Roadmap" and "2021 Roadmap."
    - The **Fabrikam group** has been granted viewer access to the "Product 2021" folder.
    - **Anne** is an owner of the "Product 2021" folder, and **Beth** is a viewer of the "2021 Roadmap" document.
    - The "Public Roadmap" is accessible to everyone.

### Expected Outcomes

| User      | Action                                   | Can They?            |
|-----------|------------------------------------------|----------------------|
| Anne      | Write to "2021 Roadmap"                  | ✅ Yes               |
| Beth      | Change ownership of "2021 Roadmap"       | ❌ No                |
| Charles   | Read "2021 Roadmap"                      | ✅ Yes               |
| Charles   | Write to "2021 Roadmap"                  | ❌ No                |
| Daniel    | Read "2021 Roadmap"                      | ❌ No                |
| Daniel    | Read "Public Roadmap"                    | ✅ Yes               |
| Anne      | Write to "Public Roadmap"                | ✅ Yes               |
| Charles   | Write to "Public Roadmap"                | ❌ No                |

---

## Modeling in OpenFGA

To model this use case in OpenFGA, we define the relationships between users, groups, folders, and documents using the following model:

```python
model
  schema 1.1
  
# There are users
type user

# Groups contain users
type group
  relations
    # Members are users
    define member: [user]

# Folders have owners, viewers, and parent folders
type folder
  relations
    define owner: [user]
    define parent: [folder]
    # Viewers include users, groups, and anyone who is a viewer of the parent folder or an owner
    define viewer: [user, user:*, group#member] or owner or viewer from parent
    # Only owners can create files
    define can_create_file: owner

# Documents also have owners, viewers, and parent folders
type doc
  relations
    define owner: [user]
    define parent: [folder]
    define viewer: [user, user:*, group#member]
    # Only owners can change ownership or share the document
    define can_change_owner: owner
    define can_share: owner or owner from parent
    define can_write: owner or owner from parent
    # Viewers, owners, or viewers of the parent folder can read
    define can_read: viewer or owner or viewer from parent
```

This model shows how OpenFGA can help manage complex relationships and permissions in a system like Google Drive.

---

## Try It Out

Now, let’s test the model in the OpenFGA CLI.

### Step 1: Install OpenFGA CLI

Make sure you have the [OpenFGA CLI installed](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

### Step 2: Test the Model

Run the following command in your terminal from the `gdrive` directory:

```bash
fga model test --tests store.yaml
```

You should see results showing which users have the expected permissions for each document and folder.

> **Pro Tip**: Experiment with modifying the model or test cases in `store.yaml` to explore how different relationships impact permissions.

---

By following this guide, you’ve successfully modeled and tested a simplified version of Google Drive’s permission system in OpenFGA! Explore further by building on this model and adding more layers of complexity as needed.