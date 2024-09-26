# **OpenFGA GitHub Permissions Model: A Step-by-Step Guide**

In this tutorial, we’ll walk through a sample permissions model for GitHub, using OpenFGA, a modern authorization system for fine-grained access control. By the end, you’ll have a clear understanding of how to model complex relationships like GitHub's user roles, teams, and repositories using OpenFGA’s schema. This model mirrors GitHub's permission structure, making it a great real-world example.

* **Title**: **GitHub Sample**  
* **Documentation**: [OpenFGA GitHub Model](https://openfga.dev/docs/modeling/advanced/github)  
* **Playground**: [FGA Sandbox](https://play.fga.dev/sandbox/?store=github)

---

## **Understanding the GitHub Permission Model**

Before diving into the code, let’s look at the core requirements of GitHub’s permission system. 

### **What Are We Building?**

GitHub’s permission model allows for different levels of access to repositories, such as **admins**, **maintainers**, **writers**, **triagers**, and **readers**. Each level inherits access from the one below it. For instance, an admin can perform all the actions of a maintainer, writer, triager, and reader.

We’ll model this structure in OpenFGA, allowing us to simulate and test how users and teams interact with GitHub repositories.

---

## **The Scenario**

Here’s the scenario we’ll be working with:

- **Users**: Anne, Beth, Charles, Diane, Erik
- **Teams**: openfga/core, openfga/backend
- **Organization**: OpenFGA owns the repository openfga/openfga.
  
In this setup:
- Members of the openfga/backend team are also members of the openfga/core team.
- Core team members have **admin** access to the openfga/openfga repository.
- The organization, OpenFGA, has a base permission of **repository admin**, granting admin access to all repositories it owns.
  
We’ll now define how OpenFGA handles these relationships.

---

## **Expected Outcomes**

Here’s what we expect after modeling:

- **Anne** is a **reader** but not a **triager** on the openfga/openfga repository.
- **Diane** is an **admin** on the repository.
- **Erik** is a **reader** on the repository.
- **Charles** is a **writer** on the repository.
- **Beth** is a **writer**, but **not** an **admin** on the repository.

---

## **Modeling the GitHub Permission Structure in OpenFGA**

Now, let's translate the GitHub model into OpenFGA’s schema. Below is the code to define the relationships.

### **OpenFGA Model Schema**

```python
model
  schema 1.1

# Define users
type user

# Define organizations
type organization
  relations
    define owner: [user]
    define member: [user] or owner
    define repo_admin: [user, organization#member]
    define repo_reader: [user, organization#member]
    define repo_writer: [user, organization#member]

# Define teams
type team
  relations
    define member: [user, team#member]

# Define repositories
type repo
  relations
    define owner: [organization]
    define admin: [user, team#member] or repo_admin from owner
    define maintainer: [user, team#member] or admin
    define writer: [user, team#member] or maintainer or repo_writer from owner
    define triager: [user, team#member] or writer
    define reader: [user, team#member] or triager or repo_reader from owner
```

### **Key Points in the Model**:
- **Organizations** own repositories and have members.
- **Teams** can have members who inherit access based on their roles.
- **Repositories** inherit permissions from both teams and organizations.

---

## **Try It Out**

Now that we’ve modeled the scenario, it’s time to test it out with the OpenFGA CLI. Here’s how you can verify the model:

### **Step 1**: Install the FGA CLI
If you haven’t installed the FGA CLI yet, follow the [installation instructions](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

### **Step 2**: Run Tests
Navigate to the `github` directory and run the following command:

```bash
fga model test --tests store.yaml
```

This command will run the tests defined in `store.yaml`, checking the relationships and permissions you’ve just set up.

---

### **Conclusion**

Congratulations! You’ve successfully modeled GitHub’s permission system using OpenFGA and tested it with real-world scenarios. Now, you can apply this knowledge to other projects where complex relationships between users, teams, and organizations are involved.

For more advanced use cases, check out the [OpenFGA Documentation](https://openfga.dev/docs/modeling/advanced/github) or explore the [FGA Sandbox](https://play.fga.dev/sandbox/?store=github) to experiment further.

---

### **Next Steps**
- Experiment with additional roles or teams.
- Apply these concepts to your own organization’s repositories.
- Visit the OpenFGA GitHub repo for more models and real-world use cases.

