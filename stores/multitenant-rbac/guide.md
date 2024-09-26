# OpenFGA Multi-Tenant Role-Based Access Example

**Welcome to this tutorial on how to set up Role-Based Access Control (RBAC) for a B2B application using OpenFGA.**  
In this guide, we’ll walk through an example that demonstrates advanced RBAC with multi-tenancy, predefined and custom roles, group nesting, and role assignments. By the end, you’ll be able to manage access to resources across multiple tenants and test the setup with OpenFGA.

## What You’ll Learn
This example is ideal if you're building an application with multiple tenants where each tenant (organization) can manage its own users, roles, and permissions. Here’s what you’ll achieve:
- Define **multi-tenant organizations** where each has its own users, groups, roles, and resources.
- Set up **predefined roles** like 'admin' and allow organizations to define **custom roles**.
- Manage **nested groups** to easily share permissions across teams.
- Assign roles to both **individual users** and **groups**.
- Set up **coarse-grained permissions** for organizational resources (we'll skip fine-grained permissions to keep this example focused on role-based access).

Now, let's jump into the setup!

## Setting Up Multi-Tenant RBAC

Before running the example, ensure that you’ve installed the [FGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

### Defining the Model
In this example, we'll define a basic model for users, groups, roles, organizations, and documents. Here's a quick rundown of the important parts:

1. **Users**: Represent individuals.
2. **Groups**: Users can belong to groups, and groups can be nested (i.e., a group can be part of another group).
3. **Roles**: Roles can be assigned to users or groups, allowing members to inherit permissions.
4. **Organizations**: Each organization has predefined roles (like 'admin') and can define custom roles.
5. **Documents**: Each document belongs to an organization, and permissions (view/edit) are inherited from roles.

### Running the Example
To see it in action, follow these steps:

1. Clone the repository and navigate to the `multi-tenant-rbac` directory.
2. Run the following command to test the model with predefined tests:
   ```bash
   fga model test --tests store.fga.yaml
   ```

This command will execute the tests defined in `store.fga.yaml` to validate our access control logic.

### The Model Definition (store.fga.yaml)

Here’s the full model for our multi-tenant setup:

```yaml
model: |
  model  
    schema 1.1

  type user

  # Groups can be nested
  type group
    relations
        define member: [user, group#member]

  # Roles can be nested, and can be assigned to users and group members
  type role
    relations
        define assignee: [user, role#assignee, group#member]

  type organization
    relations  
        define admin: [user, role#assignee]
        define user_manager: [role#assignee] or admin
        define billing_manager: [role#assignee] or admin
        define document_manager: [role#assignee] or admin
        define document_viewer: [role#assignee] or admin

        define can_invite_user: user_manager
        define can_delete_user: user_manager
        define can_edit_billing: billing_manager
        define can_create_document: document_manager

  type document
    relations
        define organization: [organization]
        define editor: document_manager from organization
        define viewer: document_viewer from organization

        define can_view: viewer or editor
        define can_edit: editor
        define can_delete: editor
```

### Testing Access Control

In the `store.fga.yaml` file, we’ve defined several tests to verify access control for different users based on their roles within the organization.

Here’s an example of how user Emily, who belongs to the engineering group, has permissions to edit and view the readme document:

```yaml
tests:
  - name: Test document permissions for each user
    check:
      - user: user:emily
        object: document:readme
        assertions:
          can_edit: true
          can_view: true
```

In this case, Emily is part of the Engineering group, and since this group is assigned the 'document-manager' role, she inherits the permissions to both edit and view the document.

The tests also include other users like Anne, Ian, and Francis to check their specific permissions within the organization.

### Extending the Example

Once you’ve run the tests, you can modify the roles, groups, and permissions to suit your application’s needs. This is just a starting point for handling multi-tenant access in a B2B app using OpenFGA. Feel free to experiment by adding new roles, nesting groups, or even trying fine-grained permissions.

## Next Steps

Now that you’ve seen how to set up multi-tenant RBAC with OpenFGA, try integrating this model into your own project. You can adjust the model to fit different access scenarios, such as fine-grained document permissions or more complex group structures.

If you run into any issues, check out the [OpenFGA documentation](https://openfga.dev) for more in-depth guidance.

