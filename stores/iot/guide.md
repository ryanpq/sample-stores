# Modeling IoT Permissions in OpenFGA: A Step-by-Step Guide

In this tutorial, we’ll walk through how to model access control for an IoT system using **OpenFGA**. Specifically, we’ll look at how to set up roles for different types of users (like IT Admins and Security Guards) who need various levels of access to IoT devices. By the end of this guide, you’ll understand how to model permissions, run tests, and implement role-based access control using OpenFGA.

For more background on OpenFGA, you can check out the [official documentation](https://openfga.dev/docs/modeling/advanced/iot).

## Use Case Overview

Imagine you’re building an IoT management system. In this system, **Security Guards** and **IT Admins** have different levels of access to IoT devices such as security cameras. Security Guards should be able to view live and recorded video feeds, while IT Admins should also have the ability to rename devices. Additionally, devices can be grouped, and access control rules should apply to all devices within a group.

This guide will show you how to model this scenario in OpenFGA, ensuring that permissions are correctly assigned and enforced across both individual devices and device groups.

### Requirements

- Security Guards can view live and recorded video from devices.
- IT Admins can view live and recorded video and rename devices.
- Devices can be grouped. If a Security Guard or IT Admin has access to a device group, they automatically have the same role for every device in that group.

### Scenario

We have four users in our system:

- **Anne** is a Security Guard with access to Device 1.
- **Beth** is an IT Admin with access to Device 1.
- **Charles** is a Security Guard with access to Device 1 and everything in Device Group 1 (which contains Device 2 and Device 3).
- **Dianne** is an IT Admin with access to Device 1 and everything in Device Group 1.

This scenario will help us demonstrate how OpenFGA handles role-based access control (RBAC) across users and devices.

### Expected Outcomes

Here’s what we expect from our access control system:

- Anne **is not** an IT Admin on Device 1, but she can view the recorded video.
- Dianne, as an IT Admin, can rename Device 2.
- Charles, as a Security Guard, **cannot** rename Device 2.

## Modeling in OpenFGA

Now, let’s look at how we can model this use case in OpenFGA. We’ll define users, devices, device groups, and the different roles for each.

### The Model

```python
model
  # We are using schema version 1.1 with type restrictions
  schema 1.1

# Define the types of entities in our system
type user

# Device groups can have IT Admins and Security Guards
type device_group
  relations
    define it_admin: [user]
    define security_guard: [user]

# Devices can have IT Admins and Security Guards, and IT Admins can rename devices
type device
  relations
    define it_admin: [user, device_group#it_admin]
    define security_guard: [user, device_group#security_guard]
    define can_rename_device: it_admin
    define can_view_live_video: it_admin or security_guard
    define can_view_recorded_video: it_admin or security_guard
```

In this model:

- A **device group** can have both IT Admins and Security Guards.
- Devices inherit IT Admins and Security Guards from their device groups.
- IT Admins can rename devices, while both IT Admins and Security Guards can view live and recorded video feeds.

For a full breakdown of the tuples and tests, check out the [store.yaml](./store.fga.yaml) file.

---

## Try It Out

Ready to see this in action? Here’s how you can test it out using the FGA CLI.

### Prerequisites
Make sure you’ve installed the [FGA CLI](https://github.com/openfga/cli/?tab=readme-ov-file#installation).

### Steps

1. Navigate to the `iot` directory where your files are located.
2. Run the following command to test the model and the defined tuples:

    ```bash
    fga model test --tests store.yaml
    ```

This will run a series of tests to ensure that our model behaves as expected. For example, it will check whether Anne can view recorded videos on Device 1, whether Dianne can rename Device 2, and so on.

### What to Expect

When you run this command, the FGA CLI will verify the permissions for each user. If all tests pass, you’ll see confirmation that your model is set up correctly. If any tests fail, double-check your `store.yaml` file to ensure that your user-to-device relationships are correctly defined.

--- 

### Conclusion

By following this guide, you’ve set up a basic IoT permission model using OpenFGA. You’ve also tested the model using FGA’s CLI to ensure that users have the correct access levels. This approach can be adapted to a variety of scenarios where fine-grained access control is required. 

For more complex use cases or advanced features, check out the official [OpenFGA documentation](https://openfga.dev/docs/modeling/advanced/iot).

