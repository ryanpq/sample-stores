# Using OpenFGA to Model Attribute-Based Access Control with ReBAC in Practical Scenarios

## Why Model ABAC with ReBAC?

When building authorization systems, a common approach is Attribute-Based Access Control (ABAC). ABAC allows you to make decisions based on the attributes of users, resources, and the environment. However, in some cases, modeling these attributes can become cumbersome or inefficient. That's where Relationship-Based Access Control (ReBAC) comes in.

In this guide, we'll explore how you can use ReBAC constructs to model ABAC scenarios with OpenFGA. By rethinking attributes as relationships, you'll often find that your authorization model becomes more scalable and easier to maintain. Plus, you'll learn when and why you might want to opt for ReBAC over more traditional ABAC approaches.

### Key Questions:
- How can we represent a user's department or email verification as a relationship?
- What kinds of attributes are easy to model in ReBAC, and when do we need additional tools like conditions?
- What advantages does this approach offer in terms of performance and simplicity?

Let’s jump into some real-world scenarios to see how it works.

## Modeling ABAC with ReBAC: Common Patterns

There are a few different types of attributes you might want to model in an authorization system. Let’s go through some of the most common ones and see how they can be handled in a ReBAC model using OpenFGA.

### 1. Attributes That Reference Another Entity

When an attribute is essentially a reference to another entity (think of it like a foreign key in a database), it's easy to model it as a relationship.

**Example**: A user belongs to a department, and that department controls access to certain resources, such as folders. Here's how that can be modeled:

```
type user

type department
    relations
        define member : [user]

type folder
    relations
        define department : [department]
        define viewer : member from department
```

In this example, the relationship between users and their departments is modeled explicitly, and folders are accessed based on a user’s membership in a department. Instead of managing department as a static attribute, it's treated dynamically through relationships.

### 2. Modeling Boolean or Small Category Attributes

Sometimes, an attribute has just a few possible values. For example, a user may have an email verification status that is either "true" or "false." Rather than modeling this as a true/false flag, we can model it as a relationship.

**Example**: Users can view documents only if their email has been verified:

```
type user
    relations
        define email_verified : [user]
        
type document
    relations
        define viewer : [user]
        define can_view : email_verified from viewer
```

By creating an `email_verified` relationship, we can enforce that only users with a verified email can view certain documents. This approach simplifies logic and keeps it within the scope of the ReBAC model.

### 3. Handling Status or State of Resources

When a resource has a status (such as "published" or "draft"), you can create a relationship for each possible state.

**Example**: Only published documents are viewable by users.

```
type user
        
type document
    relations
        define published : [document]
        define draft : [document]
        define viewer : [user]
        define can_view : viewer from published
```

This model ensures that only documents marked as `published` are viewable by users. The relationship between a document and its status is reflected in the model rather than being treated as a standalone attribute.

### 4. Attributes That Are Hard to Model

Certain attributes, like birthdates, IP addresses, or currency amounts, are hard to represent purely with ReBAC relationships. These tend to be discrete variables with many possible values.

For these cases, you’ll need to leverage OpenFGA conditions, which allow you to introduce additional logic outside of basic relationship modeling. For more information on using conditions, check out [OpenFGA conditions](https://openfga.dev/docs/modeling/conditions).

However, whenever possible, try to model your attribute as a relationship—it often leads to a simpler, more performant system.

### Example: Store Model

The full example is available in the `store.fga.yaml` file, showcasing the patterns discussed above.

## Try It Out

Ready to try this out for yourself? Follow these steps:

1. Make sure you have the [FGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation) installed.
2. In the `abac-with-rebac` directory, run `fga model test --tests store.yaml` to test the models.

## Wrapping Up

As you’ve seen, modeling ABAC scenarios with ReBAC in OpenFGA is a powerful way to simplify your authorization logic. By thinking of attributes as relationships, you can build systems that are both easier to maintain and more flexible in the long run.

Have more attributes or complex rules to model? Consider exploring OpenFGA’s condition support for those trickier cases.