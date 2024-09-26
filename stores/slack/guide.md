# Implementing Slack Role-Based Access Control (RBAC) with OpenFGA

In this guide, we will walk through how to model Slack's roles and permissions system using OpenFGA. We’ll take a real-world use case from Slack’s role management and permissions, and show you how to model similar relationships in OpenFGA. 

By the end of this guide, you will have a clear understanding of how to implement role-based access control (RBAC) for users, workspaces, and channels, and how to test these models using the OpenFGA CLI.

### What You’ll Learn:
- Model user roles like **Member**, **Admin**, and **Guest**.
- Define relationships for channel access and permissions.
- Use the OpenFGA CLI to test and validate your model.

Before we dive in, make sure you're familiar with basic OpenFGA concepts. If not, you can check out the [OpenFGA documentation](https://openfga.dev/docs/modeling/advanced/slack) for an introduction.

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

This example is inspired by Slack's publicly documented roles and permissions. For reference, here are some of Slack's key resources:

- [Slack Role Management](https://slack.engineering/role-management-at-slack/)
- [Types of Roles in Slack](https://slack.com/help/articles/360018112273-Types-of-roles-in-Slack)
- [Permissions by Role](https://slack.com/help/articles/201314026-Permissions-by-role-in-Slack)

In our scenario, we will model the following roles in OpenFGA:

- **Guest**: Limited user who can only see specific channels.
- **Member**: Base user with standard access to workspaces and channels.
- **Legacy Admin**: Full administrative privileges across Slack.
- **Channels Admin**: Limited to managing channels (creating, renaming, and archiving channels).

### Scenario

Let’s consider the following:

- We have five users: Amy, Bob, Catherine, David, and Emily.
- A workspace called **Sandcastle** has three channels: 
  - **#general**: A public channel where all members can view content, but only select members can post.
  - **#proj_marketing_campaign**: A public channel open for all members to post in.
  - **#marketing_internal**: A private channel only visible to select users.

Roles in the **Sandcastle** workspace:
- Amy is a **Legacy Admin** with full administrative rights.
- Bob is a **Channels Admin**, managing channel settings.
- Catherine and Emily are **Members** of the workspace.
- David is a **Guest**, with limited access.

Channel permissions:
- All members can view **#general**, but only Amy and Emily can post.
- **#proj_marketing_campaign** is open for all members to post in.
- **#marketing_internal** is restricted to Bob and Emily.

### Expected Outcomes

We expect the following permission rules:

1. Amy should be able to manage channels in the Sandcastle workspace.
2. David should not have channel management access.
3. Emily should be allowed to post in **#marketing_internal**.
4. David should only have write access to **#proj_marketing_campaign**.
5. Bob should not be able to post in **#general**.

---

## Modeling in OpenFGA

### Model

Here’s how we can model this Slack-inspired scenario using OpenFGA:

```python
model
  schema 1.1

# Users
type user

# Workspaces
type workspace
  relations
    define legacy_admin: [user] 
    define channels_admin: [user] or legacy_admin
    define member: [user] or legacy_admin or channels_admin
    define guest: [user]

# Channels
type channel
  relations
    define parent_workspace: [workspace]
    define writer: [user, workspace#member]
    define commenter: [user, workspace#member] or writer
```

This model captures the essential roles and relationships for users, workspaces, and channels. Now, let's see how we can put it into practice.

---

## Try It Out

### Step 1: Install the OpenFGA CLI
Make sure you have the [OpenFGA CLI installed](https://github.com/openfga/cli/?tab=readme-ov-file#installation) on your machine.

```bash
npm install -g @openfga/cli
```

### Step 2: Run the Test

Navigate to the directory containing your Slack model and run the following command:

```bash
fga model test --tests store.yaml
```

This command will validate the relationships and roles you’ve defined, checking that permissions work as expected.

