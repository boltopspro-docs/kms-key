<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/kms-key/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# KMS Key CloudFormation Blueprint

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions a KMS Key and allows to manage it with code.

* Provides example code showing how to create KMS keys.
* Optionally create a KMS Alias associated with the key.
* Configure the KMS Key admins and users with variables: `@key_admins` and `@key_users`
* Configure additional policy statements associated with the KMS keys with: `@key_policy_statements`

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/kms-key values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "kms-key", git: "git@github.com:boltopspro/kms-key.git"
```

## Configure

First you want to configure the `configs/kms-key` [config files](https://lono.cloud/docs/core/configs/).  You can use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed kms

The generated files in `config/kms-key` folder look something like this:

    configs/kms-key/
    ├── params
    │   └── development.txt
    └── variables
        └── development.rb

Here's an example config:

configs/kms-key/params/development.txt:

    # Parameter Group: AWS::KMS::Key
    AliasName=my-key

    # Parameter Group: AWS::KMS::Alias
    # Description=
    # EnableKeyRotation=true
    # Enabled=
    # KeyUsage=
    # PendingWindowInDays=7

## Deploy

Use the [lono cfn deploy](https://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    LONO_ENV=development lono cfn deploy kms-development --blueprint kms --sure

## Configure: More Details

### Stack Name Convention

By leveraging the lono Stack Name and [CLI conventions](https://lono.cloud/docs/conventions/cli/), we can organize the configs files in a way that matches the stack name. Example:

    lono cfn deploy my-key-1 --blueprint kms-key
    lono cfn deploy my-key-2 --blueprint kms-key

Will use the corresponding config files:

    configs/kms-key/development/my-key-1.txt
    configs/kms-key/development/my-key-2.txt

### Setting Key Admins and Policies

You can set the key admins and policies with the `@key_admins` and `@key_policy_statements` variables. Example:

configs/kms-key/variables/development.rb:

```ruby
# Add to Kms Key policy
@key_admins = ["arn:aws:iam::${AWS::AccountId}:user/tung"]

@key_policy_statements = [{
  Sid: "Additional policy statement",
  Effect: "Allow",
  Principal: {Service: "sns.amazonaws.com"},
  Action: [
    "kms:Decrypt",
    "kms:ReEncryptFrom"
  ],
  Resource: "*",
}]
```

In the example, the IAM user tung is an admin user of the key and can update it. Also, the key provides permission for the sns service to use it to Decrypt and ReEncryptFrom SNS Topics.

### Deleting KMS Keys

AWS does not allow immediately deleting KMS Keys. Instead, KMS keys get scheduled to be deleted. The scheduled time can be anywhere from 7-30 days. By default, this blueprint as `PendingWindowInDays=7`. So if you delete the stack, you'll see the Key in Pending Deletion 7 days out.