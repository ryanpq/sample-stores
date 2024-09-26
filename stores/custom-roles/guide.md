# Custom Roles Sample Store with OpenFGA

**Documentation**: [OpenFGA Custom Roles Documentation](https://openfga.dev/docs/modeling/custom-roles)  
**Playground**: [OpenFGA Playground](https://play.fga.dev/sandbox/?store=custom-roles)

## Table of Contents
- [Use Case](#use-case)
  - [Requirements](#requirements)
  - [Scenario](#scenario)
  - [Expected Outcomes](#expected-outcomes)
- [Modeling in OpenFGA](#modeling-in-openfga)
  - [Model](#model)
- [Try It Out](#try-it-out)

## Use Case

In this tutorial, we’ll walk through a scenario involving custom roles within an organization. The goal is to model the permissions different users have when accessing certain assets, all within the OpenFGA framework. You’ll see how organizations can create custom roles and categories for assets, assign users to these roles, and define specific permissions. 

### Requirements

For this example, there are several types of entities involved:

- **Users**: Individuals within or associated with an organization.
- **Organizations**: Groups where users can be members.
- **Teams**: Subgroups within an organization (e.g., Design, QA).
- **Assets**: Resources that need to be accessed or managed.
- **Custom Roles**: These can vary per organization and are used to define what users can do with assets.

### Scenario

Let’s break down a more detailed scenario. Here's a snapshot of the relationships we’re modeling:

- **Carlos** is the owner of the Contoso organization, and several other users belong to Contoso:
  - **Anne**, **Beth**, **Daniel** are members of Contoso.
  - Each member belongs to specific teams:
    - **Anne** is on the Design team.
    - **Beth** is on the Marketing team.
    - **Daniel** is on the QA team.
  
- The organization also has different asset categories:
  - **Website Content**: Includes assets like the Homepage.
  - **Website Media**: Includes assets like the Website Hero Image.
  
- Users in these teams are assigned various roles, giving them different permissions on these assets:
  - **Marketing team** members are assigned the "Content Manager" role, which allows them to manage content assets.
  - **QA team** members are assigned the "Content QA" role, giving them the ability to QA assets.
  - **Design team** members are assigned the "Media Asset Manager" role, giving them control over media assets.

- Additionally, **Edith** is part of the Branding Contractor 1 organization, with a special role as a Media Asset Creator.

### Expected Outcomes

After modeling the above relationships, the following outcomes should hold true:
- **Carlos** can create and edit assets in the Contoso organization.
- **Anne** can view the Website Hero Image but cannot edit it.
- **Beth** can edit the Homepage asset but not the Website Hero Image.
- **Daniel** can view, but not edit, the Homepage asset.
- **Edith** can create assets in the Website Media category but cannot view or edit assets in the Website Content category.

These expected outcomes demonstrate how custom roles can be used to manage access to assets effectively.

---

## Modeling in OpenFGA

Next, let's model this scenario using OpenFGA. Below is the schema that defines how the different entities, roles, and permissions fit together.

### Model

```python
model
  # We are using the 1.1 schema with type restrictions
  schema 1.1
type user
type team
  relations
    define member: [user]
type role
  relations
    define assignee: [user,team#member,org#member]
type org
  relations
    define asset_category_creator: [role#assignee] or owner
    define asset_commenter: [role#assignee] or asset_editor
    define asset_creator: [role#assignee] or owner
    define asset_editor: [role#assignee] or owner
    define asset_viewer: [role#assignee] or asset_commenter
    define member: [user] or owner
    define owner: [user]
    define role_assigner: [role#assignee] or owner
    define role_creator: [role#assignee] or owner
    define team_assigner: [role#assignee] or owner
    define team_creator: [role#assignee] or owner
type asset-category
  relations
    define asset_creator: [role#assignee] or asset_creator from org
    define commenter: [role#assignee] or editor or asset_commenter from org
    define editor: [role#assignee] or asset_editor from org
    define org: [org]
    define viewer: [role#assignee] or commenter or asset_viewer from org
type asset
  relations
    define category: [asset-category]
    define comment: [role#assignee] or edit or commenter from category
    define edit: [role#assignee] or editor from category
    define view: [role#assignee] or comment or viewer from category
```

---

## Try It Out

Now that you’ve seen the model, it’s time to try it out! Follow the steps below to set up and test this in your environment:

1. **Install the FGA CLI**: Make sure you have the [FGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation).
   
2. **Run the Model Tests**: In your `custom-roles` directory, use the following command to test the model and see if everything works as expected:
   ```bash
   fga model test --tests store.yaml
   ```

---

### Conclusion

By using OpenFGA’s custom roles, you can manage complex access control scenarios effortlessly. This sample store demonstrates how different users, teams, and roles can work together to control access to specific assets in a highly granular and customizable way.

For more examples and documentation, be sure to check out the official [OpenFGA site](https://openfga.dev/docs/modeling/custom-roles).