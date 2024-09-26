# OpenFGA Conditional Data Types in Relationship Tuples

## Introduction

When building authorization models, being able to define complex conditions for access control is crucial. OpenFGA allows you to use [Google's CEL expressions](https://github.com/google/cel-spec/blob/master/doc/langdef.md) to create conditional relationship tuples, enabling dynamic and context-sensitive rules for granting permissions.

In this guide, we'll explore the different data types and expressions supported in OpenFGA conditions. By the end, you'll understand how to apply conditions based on strings, integers, timestamps, and more—giving you fine-grained control over authorization logic.

We'll walk through a basic model definition, complete with conditions and tests, so you can try it out for yourself.

## Use Case: Dynamic Authorization Rules

Imagine you're building an e-commerce application where different types of users (e.g., customers, admins, managers) can access different resources based on several factors. You might want to allow a user to view an order only if the order is recent or if the user’s account meets certain conditions.

With OpenFGA and CEL, you can write these kinds of rules easily. For example:
- Allowing access only if a timestamp is within a valid range.
- Validating that a string matches a specific format or pattern.

This tutorial demonstrates how to set up a basic model with conditions and how to test it using the OpenFGA CLI.

## Prerequisites

Before getting started, ensure you have the following:
1. Installed the [OpenFGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation) on your machine.
2. Have a basic understanding of authorization models and conditional logic.

## Step-by-Step Guide

### Step 1: Defining the Model and Conditions

To get started, we'll define a model that uses several data types in its conditions, including strings, integers, doubles, timestamps, and more. This model is written in YAML and includes conditions that check if these data types meet certain criteria.

Here’s the content of `store.fga.yaml`:

```yaml
# Expressions are defined using Google CEL https://github.com/google/cel-spec/blob/master/doc/langdef.md

model: |
  model
    schema 1.1

  type user

  type datatype_test
    relations
      define is_valid: [user with is_valid_string, user with is_valid_int, user with is_valid_uint, user with is_valid_double, user with is_valid_duration, user with is_valid_timestamp, user with is_valid_map_string, user with is_valid_list_string, user with is_valid_ipaddress]

  condition is_valid_string(_string:string) {
    _string != "" && _string.startsWith("1") && _string.endsWith("1") && _string.contains("1") && _string.matches("[0-9]")
  } 

  condition is_valid_int(_int:int) {
    _int != 0 && _int > 0
  }

  condition is_valid_uint(_uint: uint) {
    _uint != 0u && _uint > 0u
  }

  condition is_valid_double(_double: double) {
    _double != 0.0 && _double > 0.0
  }

  condition is_valid_duration(_duration :duration) {
    _duration != null && _duration != duration("0s") && _duration > duration("0s") 
  }

  condition is_valid_timestamp(_timestamp: timestamp) {
    _timestamp != null && _timestamp != timestamp("2019-01-01T00:00:00Z") && _timestamp > timestamp("2019-01-01T00:00:00Z") 
  }

  condition is_valid_map_string(_mapstring: map<string>) {
    "key" in _mapstring && _mapstring["key"] != ""  && _mapstring["key"] > "" 
  }

  condition is_valid_list_string(_liststring : list<string>) {
    "1" in _liststring && _liststring[0] != "" && _liststring[0] > "" && _liststring.exists(x, x > "") && _liststring.exists_one(x, x > "") && _liststring.all(x, x > "")
  }

  condition is_valid_ipaddress(_ipaddress: ipaddress) {
    _ipaddress != null &&  _ipaddress != ipaddress("192.0.0.1")
  }
```

This YAML defines a `datatype_test` object that checks whether various types of data (such as strings, ints, etc.) meet the specified conditions. For instance:
- **String Condition**: Checks if a string starts and ends with the number "1" and contains only digits.
- **Int Condition**: Verifies that the integer is greater than 0.

### Step 2: Writing and Running Tests

We’ve defined several unit tests to check if the conditions are working as expected. Each test passes in different values for the conditions to verify that they behave correctly.

To run the tests, use the following command in the `condition-data-types` directory:

```bash
fga model test --tests store.fga.yaml
```

The tests defined in the `store.fga.yaml` file will evaluate the conditions using a variety of data types and values.

Here’s an example test:

```yaml
tests: 
  - name: "Test with context in tuples"
    tuples:
      - user: user:int
        object: datatype_test:one
        relation: is_valid
        condition:  
          name: is_valid_uint
          context:
            _uint: 1     
```

This test checks whether the `is_valid_uint` condition passes when a `user:int` tuple is supplied with the value of 1.

### Step 3: Interpreting Results

If your tests pass, you’ll see success messages for each tuple and condition. If they fail, the output will provide details on which condition failed and why. For example, if a string doesn’t match the required pattern, you’ll know exactly what went wrong.

### Summary and Next Steps

By following this guide, you've now seen how to use OpenFGA's support for Google's CEL to create complex conditional relationship tuples. Whether you're working with strings, numbers, or timestamps, CEL gives you the flexibility to define rules that fit your authorization model's needs.

Next, you might want to explore:
- Adding more complex conditions.
- Integrating OpenFGA into a real-world application.
- Learning more about how OpenFGA handles authorization logic.

For more information, check out [OpenFGA's official documentation](https://openfga.dev).
