# OpenFGA Super Admin Example: Setting Up a Multi-Tenant System

## Overview

In B2B SaaS applications, it's common for each customer to have one or more admin users with full permissions over their account. Additionally, employees of the SaaS company might need to perform specific actions across multiple customer accounts for tasks such as configuration or support.

In this guide, we'll walk through setting up a **multi-tenant Project Management system** using OpenFGA, defining various roles and permissions for both customers and internal employees. We'll create a model that supports:

- **System Administrators**: Users who can perform any action in the system.
- **Help-Desk Users**: Users who can view resources for a specific customer, but only for a limited time (1-hour access for security purposes).
- **Customer Administrators**: Users who have full access to resources linked to a specific customer.
- **System Management Service App**: A service application with full system access for maintenance tasks.

The resources we'll be managing include:
- **Projects**
- **Tasks**

By the end of this tutorial, you'll have an OpenFGA model with tests for verifying that the permissions and access controls work as intended.

## Prerequisites

To follow along with this example, you will need the [OpenFGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation) installed.

Once ready, navigate to the `superadmin` directory and run the following command to execute the model tests:

```bash
fga model test --tests store.yaml
```

## Defining the OpenFGA Model

We'll break the model down into its key components and explain the logic behind each definition.

### **System-Level Roles**

At the top level, we define the `system`, where **System Administrators** are granted full access. This allows certain internal employees or applications to manage the entire system, not just individual organizations.

```yaml
type system
  relations
    define admin : [employee, application] 
```

### **Customer Organizations**

Each customer is represented by an `organization`. Admins and help-desk users are associated with specific organizations. Notice that system admins automatically gain the same permissions as customer admins within any organization.

```yaml
type organization
  relations
    define system : [system]
    define admin : [user] or admin from system
    define member : [user]
    define helpdesk_member : [employee with non_expired_time_grant]
    define can_create_project : admin or member
```

Here, **helpdesk_member** is defined with a temporal condition that restricts access to a specific time period, ensuring best security practices.

### **Projects and Tasks**

For each `project` within an organization, we define who can edit and view tasks. Projects inherit permissions from the organization they belong to.

```yaml
type project
  relations
    define organization : [organization]
    define owner : [user]
    define editor : [user] or owner or admin from organization
    define viewer : [user] or editor or helpdesk_member from organization
```

Tasks similarly inherit permissions from their parent project:

```yaml
type task
  relations
    define project: [project]
    define owner : [user]
    define editor : [user] or editor from project
    define viewer : [user] or editor or viewer from project
```

### **Temporal Access Control**

We use a condition to allow help desk users to access resources for a limited time:

```yaml
condition non_expired_time_grant(current_time: timestamp, grant_time: timestamp, grant_duration: duration) {
    current_time < grant_time + grant_duration
}
```

This ensures that access is automatically revoked after the specified time period.

## Assigning Roles with Tuples

Now, let's define the relationships between users and resources using tuples.

### **System Admins**

Here we define `anne` and a system management app as super admins. These users have full access across the system:

```yaml
tuples:
  - user: employee:anne
    relation : admin
    object: system:global
  - user: application:system-management-app
    relation : admin
    object: system:global
```

### **Help-Desk User with Time-Limited Access**

We grant a help desk user (`john`) access to an organization (`acme`) for just one hour:

```yaml
  - user: employee:john
    relation: helpdesk_member
    object: organization:acme
    condition: 
      name: non_expired_time_grant
      context: 
        grant_time : "2024-01-01T00:00:00Z"
        grant_duration : 1h
```

### **Customer Admins and Projects**

Customer admins like `peter` have full control over resources in their organization:

```yaml
  - user: user:peter
    relation: admin
    object: organization:acme
```

We also define relationships between organizations, projects, and tasks:

```yaml
  - user: organization:acme
    relation: organization
    object: project:openfga
  - user: project:openfga
    relation: project
    object: task:create-example
```

## Testing the Permissions

We can now test our model to ensure that each user has the correct access based on their role.

### **Full Access for System Admins**

Since `anne` is a super admin, she should have full access:

```yaml
  - user: employee:anne
    object: task:create-example
    assertions: 
      viewer: true
      editor: true
```

### **Customer Admin Access**

Similarly, customer admin `peter` should have full access to tasks in their organization:

```yaml
  - user: user:peter
    object: task:create-example
    assertions: 
      viewer: true
      editor: true
```

### **Time-Limited Help Desk Access**

`john` only has access to view tasks during his 1-hour window:

```yaml
  - user: employee:john
    object: task:create-example
    context: 
      current_time: "2024-01-01T00:10:00Z"
    assertions: 
      viewer: true
      editor: false
```

## Conclusion

In this guide, we've set up a multi-tenant project management system with various roles and permissions using OpenFGA. This setup ensures secure, time-bound access for help desk users while giving admins full control over their respective organizations.

If you want to explore more about OpenFGA, check out [OpenFGA.dev](https://openfga.dev) or explore the CLI documentation for more advanced features.