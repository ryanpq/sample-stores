# OpenFGA Advanced Entitlements: Creating a Subscription Plan Model Based on Feature Usage

## Introduction

In this guide, we'll walk through how to implement an entitlements model inspired by Notion's subscription plans. This approach is useful for managing access to features based on usage limits, such as the number of collaborators, synced rows, or page history days available under a subscription. 

The main goal is to show you how to define an entitlements model using OpenFGA, a flexible authorization solution, and then test it using sample data. If you're working with software that has different subscription tiers, this model will be particularly relevant.

We'll start by setting up the model, explain the code snippets in detail, and show how to run tests to verify your setup.

The full model, tuples, and unit tests are available in the file [store.fga.yaml](./store.fga.yaml).

## Understanding the Use Case

The model we're creating is based on how platforms like Notion define feature limits for different subscription tiers (e.g., free vs. pro plans). For example, a free plan might allow up to 10 guest collaborators, while a pro plan allows 100. Similarly, the number of days for which page history is available or the number of rows you can sync from external databases can vary by plan.

By the end of this tutorial, you'll have a working model that can help you manage such feature entitlements in your own applications.

## Step-by-Step: Try It Out

Before we dive into the model, let's make sure your environment is set up correctly.

### Step 1: Install the OpenFGA CLI

First, make sure you have the OpenFGA CLI installed. You can follow the instructions [here](https://github.com/openfga/cli/?tab=readme-ov-file#installation) if you haven't already.

### Step 2: Run the Model Test

Once you have the CLI installed, navigate to the `advanced-entitlements` directory in your project. This directory contains the model and test data for this example.

Run the following command to test the model:

```bash
fga model test --tests store.fga.yaml
```

This command will use the model and test cases defined in `store.fga.yaml`. You can find the full file below, but let’s walk through some key parts first.

### The Model (store.fga.yaml)

```yaml
model: |
  model
    schema 1.1

  type user
  type organization
    relations
      define member: [user]

  type plan
    relations
      define subscriber: [organization#member]

  type feature
    relations
      define has_feature: [plan#subscriber, plan#subscriber with is_below_collaborator_limit, plan#subscriber with is_below_row_sync_limit, plan#subscriber with is_below_page_history_days_limit]

  condition is_below_collaborator_limit(collaborator_count: int, collaborator_limit: int) {
    collaborator_count <= collaborator_limit
  }

  condition is_below_row_sync_limit(row_sync_count: int, row_sync_limit: int) {
    row_sync_count <= row_sync_limit
  }

  condition is_below_page_history_days_limit(page_history_days_count: int, page_history_days_limit: int) {
    page_history_days_count <= page_history_days_limit
  }
```

Here, we’re defining a simple model with `user`, `organization`, `plan`, and `feature` types. The relationships (`relations`) help define the entitlements each plan has for different features.

For example, the `is_below_collaborator_limit` condition ensures that the number of collaborators does not exceed the plan’s limit. You can see similar logic applied for row syncing and page history days.

### Testing the Model

The `store.fga.yaml` file also defines some sample users, plans, and features, so let’s look at how we test the model using the OpenFGA CLI.

```yaml
tests:
  - name: Tests for Pro subscriber
    check:
    - user: user:beth
      object: feature:can-view-page-history
      context: 
        page_history_days_count: 10
      assertions:
        has_feature: true
```

In this example, we’re testing whether Beth (a pro plan subscriber) can view a page’s history when it’s set to 10 days. Since the pro plan has a page history limit of 30 days, this assertion will pass.

Each test block checks different features and plan entitlements, helping you verify that your model works as expected.

### Running Tests and Interpreting Results

When you run the tests with the command we provided earlier, the output will show whether each test passed or failed. If everything is working correctly, you'll see that users with the pro plan have more feature access compared to those on the free plan.

You can also modify the test data (e.g., change the number of collaborators or rows) to see how the entitlements change based on different limits.

## Conclusion

Congratulations! You've just built and tested an advanced entitlements model using OpenFGA. This model is a great starting point if you're managing subscription plans with feature limits. 

Next steps? Consider applying this model in your own projects. You could expand it by adding more feature conditions or integrate it with your existing authorization system. If you’re new to OpenFGA, check out the [official documentation](https://openfga.dev) for more advanced use cases and examples.
