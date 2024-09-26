# Managing Expenses Approval with OpenFGA: A Step-by-Step Guide

In this guide, we’ll walk through a simple yet powerful use case for OpenFGA, where employees submit reports, and their managers approve them. We'll explore how you can use OpenFGA to model this scenario with a chain of command, ensuring only the right people have approval rights.

**Explore the model live here**: [FGA Playground](https://play.fga.dev/sandbox/?store=expenses)

## Table of Contents
- [Introduction](#introduction)
- [The Use Case](#the-use-case)
  - [System Requirements](#system-requirements)
  - [Scenario Overview](#scenario-overview)
  - [Expected Results](#expected-results)
- [Building the OpenFGA Model](#building-the-openfga-model)
  - [The Model](#the-model)
- [Testing the Model](#testing-the-model)

## The Use Case

### System Requirements

Imagine a reporting system for employees that ensures only managers can approve reports. Here’s how it works:

- Employees can submit reports, but only their managers can approve them.
- Managers include both direct supervisors and anyone up the management chain.
- Employees can't approve their own reports.

### Scenario Overview

Let’s take a look at an example:

- **Daniel, Matt, Sam, and Emily** work for the same company.
- Matt is Daniel’s direct manager.
- Sam manages Matt, and Emily is Sam’s manager.
- Daniel submits a report for a new office chair, while Sam submits a report for his own chair.

### Expected Results

Based on the structure:
- **Matt** is responsible for approving **Daniel’s** report.
- **Emily**, as Sam’s manager, can approve reports for both **Daniel** and **Sam**.
- **Daniel** cannot approve his own report.

## Building the OpenFGA Model

We can model this scenario using OpenFGA's relationship-based access control (ReBAC) system. If you’re new to OpenFGA modeling, check out the [OpenFGA Modeling Guides](https://openfga.dev/docs/modeling).

### The Model

```python
model
  # We're using schema 1.1 for this model with type restrictions
  schema 1.1

# Employees in the system
type employee
  relations
    # Each employee has a direct manager
    define manager: [employee]
    # An employee can be managed by their direct manager and all managers up the chain
    define can_manage as manager or can_manage from manager

# Reports submitted by employees
type report
  relations
    # The submitter of a report is the employee who created it
    define submitter: [employee]
    # The approver is any employee who can manage the submitter
    define approver: can_manage from submitter
```

This simple model defines relationships between employees and reports. Employees can manage other employees, and reports can only be approved by someone who manages the submitter.

For a deeper dive into how this works, feel free to check out the [OpenFGA Configuration Language](https://openfga.dev/docs/configuration-language).

## Testing the Model

Let’s put the model to the test! We’ll use the FGA CLI to simulate different approval scenarios and check if the model behaves as expected.

### Try It Out

Follow these steps to see the model in action:

1. **Install the FGA CLI** if you haven’t already by following the [installation instructions here](https://github.com/openfga/cli/?tab=readme-ov-file#installation).
   
2. In your project directory (in this case, the `entitlements` folder), run the following command to test the model:

   ```bash
   fga model test --tests store.yaml
   ```

This will execute a series of tests, including checking if the right people can approve reports.

### Testing Scenarios

We’ve already set up a few tests to see if the model works correctly. Here's what we’re testing:

- **Manager Tests**: Can Matt manage Daniel, and can Emily approve Daniel’s report?
- **Approval Tests**: Which reports can Emily approve, and can Daniel approve his own report?

Here’s a summary of what’s inside the `store.yaml` file:

```yaml
# FGA Playground: https://play.fga.dev/sandbox/?store=expenses
name: Expenses
model_file: ./model.fga
tuples:
  # Matt is Daniel's manager
  - user: employee:matt
    relation: manager
    object: employee:daniel
  # Sam is Matt's manager
  - user: employee:sam
    relation: manager
    object: employee:matt
  # Emily is Sam's manager
  - user: employee:emily
    relation: manager
    object: employee:sam
  # Daniel has submitted the "Daniel Chair" report
  - user: employee:daniel
    relation: submitter
    object: report:daniel-chair1
  # Sam has submitted the "Sam Chair" report
  - user: employee:sam
    relation: submitter
    object: report:sam-chair1
tests:
  - name: Test for managers and approvers
    check:
      - user: employee:matt
        object: employee:daniel
        assertions:
          can_manage: true
      - user: employee:emily
        object: report:daniel-chair1
        assertions:
          approver: true
      - user: employee:daniel
        object: report:daniel-chair1
        assertions:
          approver: false
```

You can expand on these tests by adding your own scenarios or tweaking the current ones.

## Conclusion

In this guide, we showed how OpenFGA can help model hierarchical approval processes, where employees can only approve reports for the people they manage. We’ve also tested this with a set of rules to ensure the model behaves as expected.

Want to explore more? Check out the [FGA Playground](https://play.fga.dev/sandbox/?store=expenses) and experiment with different setups and roles.
