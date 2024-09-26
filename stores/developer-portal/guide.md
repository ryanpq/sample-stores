# Building a Developer Portal with OpenFGA: A Step-by-Step Guide

In this guide, we're going to walk through how to build a developer portal using OpenFGA, a powerful tool for managing fine-grained access control. We’ll focus on a common use case: a B2B SaaS platform where your customers can define applications that interact with your APIs. The goal? To make sure every member of your customer organizations has the right level of access—no more, no less.

## Why This Matters

If you're building a SaaS platform, managing access control at a granular level can be tricky. Different customers have different needs, and often they need to control who can manage their applications, view member lists, or configure application components. With OpenFGA, you can set this all up efficiently while ensuring that access is controlled according to your customers' needs.

In this example, we’ll cover how to:

- Allow customers to define members and administrators within their organization.
- Enable customers to manage applications and the components those applications can access.
- Define specific permissions for different roles—what each user can view, edit, or delete.

This will all be done through the model we create, where we define relationships (like members, applications, and components) and permissions for each entity.

## Setting Up OpenFGA

To start, you’ll need the OpenFGA CLI installed. If you haven’t done that yet, follow the [installation instructions here](https://github.com/openfga/cli/?tab=readme-ov-file#installation). Once you're set up, you'll be able to define and test models quickly.

### Step 1: Creating the Access Control Model

In our use case, each customer organization will have members and administrators. Administrators will be able to manage users and applications, while regular members will have more limited access. Let’s take a look at the model:

```yaml
model: |
  model
    schema 1.1
  ...
```

In this model:
- **Organizations** represent the customers using your SaaS platform.
- **Users** can either be regular members or administrators within an organization.
- **Applications** belong to an organization and have their own credentials.
- **Components** represent features (like "Purchases" or "Payments") that applications can access.

#### How Access Works

- Administrators can invite and remove members, manage applications, and configure application access to specific components.
- Regular members can view application details but can’t make changes.

### Step 2: Defining Access Rules in YAML

Here’s how we define these relationships and permissions in our `store.fga.yaml` file:

```yaml
type organization
  relations
      define member: [user]
      define admin: [user]
  ...
```

We start by defining the **organization** and its relation to the **users**. Each organization can have both members and admins. Admins get more permissions, such as the ability to invite or remove members and manage applications.

Similarly, we define applications and components. Each application has certain permissions (like viewing, editing, or deleting), and we can configure which components an application has access to, based on what the customer has paid for.

### Step 3: Running the Model Tests

To ensure your model works as expected, you can run tests. These tests check that the relationships and permissions we’ve defined are being enforced correctly. 

Make sure you're in the `developer-portal` directory and run:

```bash
fga model test --tests store.yaml
```

This will validate the permissions for different users and applications.

### Step 4: Understanding the Tests

Let’s break down what the tests are checking:

```yaml
- name: Test permissions for users and applications
  check:
    - user: user:anne
      object: application:1
      assertions:
        can_edit : true
        can_delete : true
```

In this test, we're checking if **Anne**, an administrator, can edit and delete an application. Since she's an admin, the test should pass. 

We also check for regular users like **Marie**:

```yaml
- user: user:marie
  object: application:1
  assertions:
    can_edit : false
    can_view : true
```

Marie, a regular member, should be able to view the application but not edit or delete it. These tests help ensure that permissions are correctly applied for each user role.

### Step 5: Expanding Your Model

This model is just a starting point. As your SaaS platform grows, you can expand the model to cover more specific use cases. For example, you can create different types of components, each with their own permissions, or set up more granular access controls for specific data within your applications.

---

By following these steps, you can set up fine-grained access control in your developer portal using OpenFGA. For more details, be sure to check out the [OpenFGA documentation](https://openfga.dev) and experiment with different models that fit your platform's needs.