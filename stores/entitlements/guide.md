# Implementing an Entitlement System with OpenFGA: A Sample GitHub-Style Model

* **Title**: **Entitlements Example for Service Plans**
* **Documentation**: [OpenFGA Docs](https://openfga.dev/docs/modeling/advanced/entitlements)
* **Playground**: [OpenFGA Playground](https://play.fga.dev/sandbox/?store=entitlements)

## Overview

In this guide, we'll demonstrate how to model a simple entitlement system using OpenFGA, inspired by the structure of GitHub's service plans. By the end of this tutorial, you’ll understand how to create a system where organizations can subscribe to different plans, and members of those organizations can access features based on their plan. This model can be applied to any SaaS-style service offering tiered features based on subscription levels.

## Table of Contents
- [Use Case](#use-case)
  - [Requirements](#requirements)
  - [Scenario](#scenario)
  - [Expected Outcomes](#expected-outcomes)
- [Modeling in OpenFGA](#modeling-in-openfga)
  - [Model](#model)
- [Try It Out](#try-it-out)

## Use Case

### Requirements

In this example, we’ll build an entitlement model for a service that offers different subscription plans, similar to how GitHub offers different levels of access depending on your plan. We'll be focusing on modeling users, organizations, plans, and features. Here's how the relationships break down:

- **Organizations** can have **members**.
- **Organizations** subscribe to **plans**.
- **Plans** grant access to certain **features**.
- If an organization subscribes to a plan, the members of that organization inherit access to the features of that plan.

### Scenario

Let’s walk through a practical scenario:

There are three subscription plans:
- **Free** Plan: Access to Issues.
- **Team** Plan: Access to all Free plan features, plus Draft Pull Requests.
- **Enterprise** Plan: Access to all Team plan features, plus SAML Single Sign-On (SSO).

Three organizations subscribe to these plans:
- **Alpha Beta Gamma** is subscribed to the Free plan.
- **Bayer Water Supplies** is subscribed to the Team plan.
- **Cups and Dishes** is subscribed to the Enterprise plan.

Each organization has one member:
- **Anne** is a member of Alpha Beta Gamma.
- **Beth** is a member of Bayer Water Supplies.
- **Charles** is a member of Cups and Dishes.

The relationships between organizations, plans, and features look like this:

![Entitlements Scenario](https://openfga.dev/assets/images/entitlements-requirements-fdd4048edc4d4b3b78785f4c0671e0b1.svg)

### Expected Outcomes

Here's what we expect based on the scenario:

- **Anne** (Free plan):
  - Should have access to Issues.
  - Should **not** have access to Draft Pull Requests or Single Sign-On.

- **Beth** (Team plan):
  - Should have access to Issues and Draft Pull Requests.
  - Should **not** have access to Single Sign-On.

- **Charles** (Enterprise plan):
  - Should have access to Issues, Draft Pull Requests, and Single Sign-On.

By the end of this guide, you'll be able to model and verify these entitlements using OpenFGA.

## Modeling in OpenFGA

### Model

Here's how we can represent this structure using OpenFGA’s relationship-based access control (ReBAC) model. Each component, such as users, organizations, plans, and features, is mapped to a "type" with specific relationships:

```python
model
  # We are using the 1.1 schema with type restrictions
  schema 1.1

# Define users
type user

# Define organizations
type organization
  relations
    # Organizations have members (restricted to users)
    define member: [user]

# Define subscription plans
type plan
  relations
    # Plans have subscriber organizations
    define subscriber: [organization]
    # Any member of a subscriber organization becomes a "subscriber member"
    define subscriber_member: member from subscriber

# Define features
type feature
  relations
    # Features are associated with plans
    define associated_plan: [plan]
    # Users who have access to a feature are "subscriber members" of the associated plan
    define can_access: subscriber_member from associated_plan
```

In this model:
- **Users** are members of **organizations**.
- **Organizations** subscribe to **plans**, granting them access to specific **features**.
- **Members** of the organization inherit access to the features associated with their organization’s plan.

For example:
- If an organization subscribes to the Team plan, all members of that organization can access Issues and Draft Pull Requests.

### Try It Out

Ready to test your entitlements model? Follow these steps:

1. Install the [FGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation).
2. Navigate to the `entitlements` directory.
3. Run the following command to verify the model and test scenarios:
   ```bash
   fga model test --tests store.yaml
   ```

The `store.fga.yaml` file contains the necessary tuples and tests to validate the model. Here's a look at what the tests check:

- **Anne** can access Issues but not Draft Pull Requests or SSO.
- **Beth** can access Issues and Draft Pull Requests, but not SSO.
- **Charles** can access all features: Issues, Draft Pull Requests, and SSO.

## Wrapping Up

You've now created a simple entitlement system using OpenFGA. This model can easily be extended to more complex use cases, such as additional subscription plans or more granular feature controls. Feel free to modify the model and test out different scenarios in the [OpenFGA Playground](https://play.fga.dev/sandbox/?store=entitlements).

