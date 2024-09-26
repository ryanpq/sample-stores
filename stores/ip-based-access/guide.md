# OpenFGA IP-Based Access Control for Documents

In this tutorial, we’ll show you how to use OpenFGA to control access to documents based on a user's IP address. The goal is to restrict access so that only users in a specific IP range (like a corporate network) can view certain documents.

This can be useful for organizations that need to ensure sensitive documents are only accessed from within a secure network.

---

## Why IP-Based Access?

Imagine a company wants its employees to access documents only when they are within the corporate network. This guide walks you through how to define an access control model where users can only view documents if:
1. They are part of the organization.
2. Their IP address falls within the organization’s allowed range.

The OpenFGA model, tuples, and unit tests are detailed in the provided `store.fga.yaml` file.

---

## Setup

Before we dive into the code, make sure you have the necessary tools installed:

### Prerequisites
- **OpenFGA CLI**: If you don’t have it yet, install it by following [these instructions](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

### Running the Model
Once the CLI is installed, navigate to the `ip-based-access` directory and run the following command to test the model:

```bash
fga model test --tests store.fga.yaml
```

---

## Understanding the Model

Let’s break down the core elements of our access control model, defined in the `store.fga.yaml` file.

### The Model Definition

The access control model defines the relationships and permissions we want to enforce. Here's how it works:

```yaml
model: |
  model
    schema 1.1

  type user

  type organization
    relations
      define member : [user]

      # An IP range can be assigned to the members of every organization
      define ip_based_access_policy : [organization#member with in_company_network]

  type document
    relations
      define organization : [organization]      
      define viewer: [user]

      # A user can view a document if they are assigned as a viewer, and the Organization's IP range 
      # matches the provided IP address 
      define can_view: viewer and ip_based_access_policy from organization
```

- **User Type**: Represents people who will be accessing documents.
- **Organization Type**: Links users to organizations and defines the allowed IP range for access.
- **Document Type**: Represents the documents users want to view.

The key rule here is that a user can view a document only if:
- They are a viewer of the document.
- Their IP falls within the allowed range of the organization.

---

### Defining IP-Based Conditions

We use a custom condition to check if the user’s IP address falls within the allowed range for the organization:

```yaml
condition in_company_network(user_ip: ipaddress, cidr: string) {
  user_ip.in_cidr(cidr)
}
```

This condition ensures that only users connecting from a specific IP range (defined by CIDR) can access documents.

---

### Sample Data (Tuples)

The tuples define actual relationships between users, organizations, and documents in the system:

```yaml
tuples:
    - user: "organization:acme#member"
      relation: "ip_based_access_policy"
      object: "organization:acme"
      condition: 
        name: in_company_network
        context: 
          cidr: "192.168.0.0/24"
```

- **Organization Acme** has an IP-based access policy.
- The allowed IP range is `192.168.0.0/24`.

Next, we define who can view documents and from where:

```yaml
    - user: "organization:acme"
      relation: "organization"
      object: "document:1"
      
    - user: "user:anne"
      relation: "viewer"
      object: "document:1"
      
    - user: "user:anne"
      relation: "member"
      object: "organization:acme"
```

In this case, Anne is part of the Acme organization and is assigned as a viewer for `document:1`.

---

### Writing Tests

To ensure our model works correctly, we’ll run some tests. These tests simulate different scenarios and verify that users can or cannot view documents based on their IP address.

```yaml
tests:
  - name: Users in the corporate network can view documents
    check:
    - user: user:anne
      object: document:1
      context: 
        user_ip: 192.168.0.1
      assertions:
        can_view: true
```

This test checks that Anne, connected from the IP `192.168.0.1` (inside the allowed range), **can** view the document.

We also check that users outside the network are denied access:

```yaml
  - name: Users outside the corporate network cannot view documents
    check:
    - user: user:anne
      object: document:1
      context: 
        user_ip: 192.168.1.1
      assertions:
        can_view: false
```

Here, Anne, connecting from the IP `192.168.1.1` (outside the range), **cannot** view the document.

---

## Next Steps

Now that you've successfully set up IP-based access control, here are some ways to expand on this example:
- Try adding more granular permissions, like restricting document access based on user roles within the organization.
- Explore integrating this model with your authentication system to dynamically fetch user IPs.
- Check out the [OpenFGA documentation](https://openfga.dev) to learn more about defining custom access models.
