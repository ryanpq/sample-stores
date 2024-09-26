# OpenFGA: Implementing Access Control in a Banking System

## Use-Case: Role-Based Access Control for Bank Transactions

In this post, we’ll walk through how to model a simple banking system using OpenFGA to manage account owners, account managers, and transaction limits. By leveraging OpenFGA’s flexible policy framework, we can set custom limits on bank transactions and override them when necessary.

This model allows bank customers to transfer money within their assigned limits, while account managers have elevated permissions to handle larger transactions.

By the end of this guide, you’ll know how to:
- Define users, roles, and relationships in OpenFGA.
- Set up transaction limits for different user roles.
- Write and run tests to validate these permissions.

Let's dive in!

## Overview of the Model

The core components of this system include:
- **Customers** who own bank accounts and have transaction limits.
- **Account Managers** who oversee accounts and have higher transaction limits.
- **Bank Transfer Policies** that govern who can make transactions and for how much.

In this model, we define relationships and conditions to ensure that transactions follow the appropriate policies. These conditions can be overridden for individual transactions if needed.

## Setting Up the Model

To start, ensure you have the [OpenFGA CLI installed](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

Now, let’s look at the actual model and how we define it in OpenFGA’s DSL (Domain-Specific Language).

```yaml
model: |
  model
    schema 1.1

  type employee
  type customer

  # Global bank type to define customers and transaction limit policies
  type bank
    relations
      define customer : [customer]
      define account_manager : [employee]

      # Different transfer limits for customers and account managers
      define transfer_limit_policy : [bank#customer with transfer_limit_policy, bank#account_manager with transfer_limit_policy]

  type account
    relations
      define bank : [bank]
      define owner : [customer] 
      define account_manager : [employee]

      # Permissions for making bank transfers based on transfer limit policies
      define can_make_bank_transfer : (owner or account_manager) and transfer_limit_policy from bank

  # Condition to override policy limits for specific transactions
  condition transfer_limit_policy(transaction_amount: double, transaction_limit: double, new_transaction_limit_approved: double) {
     transaction_amount <= transaction_limit || transaction_amount <= new_transaction_limit_approved
  }
```

### Explanation of the Model

1. **Bank, Customers, and Account Managers**: The `bank` type defines the relationships between the bank, its customers, and its account managers.
2. **Transfer Limit Policies**: Each bank has a `transfer_limit_policy` that assigns different transaction limits to customers and account managers.
3. **Accounts and Transactions**: The `account` type defines the relationship between accounts, their owners, and account managers. It also defines the conditions under which a bank transfer can be made, enforcing the transfer limits defined earlier.

## Defining Tuples

Tuples in OpenFGA represent specific relationships between users and objects (in our case, accounts and banks). Let’s define the tuples for our banking system:

```yaml
tuples:
  # Customers can transfer up to $100 
  - user: bank:acme#customer
    relation: transfer_limit_policy
    object: bank:acme
    condition:
      name: transfer_limit_policy
      context: 
        transaction_limit: 100

  # Account managers can transfer up to $1000 
  - user: bank:acme#account_manager
    relation: transfer_limit_policy
    object: bank:acme
    condition:
      name: transfer_limit_policy
      context: 
        transaction_limit: 1000

  # Anne is a customer of Acme Bank
  - user: customer:anne
    relation: customer
    object: bank:acme

  # Bob is an account manager at Acme Bank
  - user: employee:bob
    relation: account_manager
    object: bank:acme

  # The `123` account belongs to Acme Bank
  - user: bank:acme
    relation: bank
    object: account:123

  # Anne is the owner of account `123`
  - user: customer:anne
    relation: owner
    object: account:123

  # Bob manages account `123`
  - user: employee:bob
    relation: account_manager
    object: account:123
```

### Summary of the Tuples

- **Transaction Limits**: We’ve defined that customers can transfer up to $100, while account managers have a limit of $1000.
- **Account Ownership**: Anne owns account `123`, and Bob is the account manager.

## Testing the Model

Next, we’ll validate the model by running a series of tests. These tests simulate various transactions to ensure that the permissions are enforced correctly.

Run the tests using the CLI:

```bash
fga model test --tests store.fga.yaml
```

Here are the test cases:

```yaml
tests:
  - name: Test bank transfers from customers
    check:
    - user: customer:anne
      object: account:123
      context: 
        transaction_amount: 10
        new_transaction_limit_approved: 0
      assertions:
        can_make_bank_transfer: true
        
    - user: customer:anne
      object: account:123
      context: 
        transaction_amount: 1000
        new_transaction_limit_approved: 1000
      assertions:
        can_make_bank_transfer: true

    - user: customer:anne
      object: account:123
      context: 
        transaction_amount: 1000
        new_transaction_limit_approved: 0
      assertions:
        can_make_bank_transfer: false

  - name: Test bank transfers from account managers
    check:
    - user: employee:bob
      object: account:123
      context: 
        transaction_amount: 1000
        new_transaction_limit_approved: 0
      assertions:
        can_make_bank_transfer: true
```

### Test Breakdown:

- **Customer Transfers**: We test if Anne, a customer, can make a bank transfer with different amounts and approved limits.
- **Account Manager Transfers**: We verify that Bob, the account manager, can make larger transfers based on his role and the policies in place.

## Next Steps

Congratulations! You’ve successfully modeled a banking system with OpenFGA, including user roles, transaction limits, and test validation.

From here, you might explore:
- Adding more granular policies for specific transaction types.
- Extending this model to include other roles, such as auditors or regulators.
- Reading more about OpenFGA’s advanced features in the [official documentation](https://openfga.dev).

