# Terraform State Files

One of the best benefits of Terraform is it can keep track of resource state. This means if a resource's configuration changes (this is called **drift**) Terraform will revert its state back to that which is defined in the resource state. Let's use the S3 bucket to talk through a relevant example here. 

Currently, our S3 bucket has no versioning enabled which means our data can irreversibly be overwritten or deleted with no option to recover it. Let's modify our bucket to enable this.

#### Enabling S3 Bucket Versioning via Terraform

*Refer to the [Terraform documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_versioning) for understanding how to enable S3 bucket versioning.*

**Updated `main.tf`**
```
  # Create an S3 Bucket
  resource "aws_s3_bucket" "bucket-1" {
    bucket = var.aws_s3_bucket_name

    tags = var.aws_tagging
  }

  # Enable S3 Bucket Versioning
  resource "aws_s3_bucket_versioning" "bucket-1-set-versioning" {
    bucket = aws_s3_bucket.bucket-1.id

    versioning_configuration {
      status = var.aws_s3_bucket_versioning
    }

  }
```

**Updated `variables.tf`**
```
  variable "aws_s3_bucket_versioning" {
    description = "Configure bucket version"
    type        = string
    default     = "Enabled"
  }
```

Now, let's run `terraform plan` and we should see it wanting to enable versioning on the bucket. Yours should look similar to mine below.

```
  Terraform will perform the following actions:

    # aws_s3_bucket_versioning.bucket-1-set-versioning will be created
    + resource "aws_s3_bucket_versioning" "bucket-1-set-versioning" {
        + bucket = "tylers-unique-bucket-name-15lkj5"
        + id     = (known after apply)

        + versioning_configuration {
            + mfa_delete = (known after apply)
            + status     = "Enabled"
          }
      }

  Plan: 1 to add, 0 to change, 0 to destroy.
```

Go ahead and apply this change with `terraform apply`.

#### Creating Drift

Now, let's demonstrate how Terrafrom can detect drift. Run this AWS CLI command to suspend the bucket versioning we just enabled. 

`aws s3api put-bucket-versioning --bucket <your-bucket-name> --versioning-configuration Status=Suspended`

Once you've run that command, let's run `terraform plan` and see what happens. 

```
  Terraform will perform the following actions:

  # aws_s3_bucket_versioning.bucket-1-set-versioning will be updated in-place
  ~ resource "aws_s3_bucket_versioning" "bucket-1-set-versioning" {
        id     = "tylers-unique-bucket-name-15lkj5"
        # (1 unchanged attribute hidden)

      ~ versioning_configuration {
          ~ status = "Suspended" -> "Enabled"
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.
```

As you can see, Terraform detected drift from the original state of our S3 bucket. The last time Terraform applied, it Enabled bucket versioning but now it detects it's no longer in an Enabled state. How does it do this?

## Understanding Terraform State Files

When Terraform is run, it creates two new files in the Configuration Directory, `terraform.tfstate` and `terraform.tfstate.backup`. These files currently store the configuration state for our S3 bucket. If you want to view the configuration state for a resource, you could view the files in Visual Studio or your preferred code editor but Terraform also has a command to do this. 

If we run this command `terraform state show aws_s3_bucket_versioning.bucket-1-set-versioning` we can understand why Terraform wants to make the change above from Suspended to Enabled.

```
  % terraform state show aws_s3_bucket_versioning.bucket-1-set-versioning      
  # aws_s3_bucket_versioning.bucket-1-set-versioning:
  resource "aws_s3_bucket_versioning" "bucket-1-set-versioning" {
      bucket = "tylers-unique-bucket-name-15lkj5"
      id     = "tylers-unique-bucket-name-15lkj5"

      versioning_configuration {
          status = "Enabled"
      }
  }
```

After viewing the Terraform state, we can see that when Terraform last created this resource, it set the Versioning to Enabled. However, since we modified this to Suspended outside of Terraform, we've effectively created drift!

#### Fixing Drift
Let's re-apply our Terraform configuration for the S3 bucket and re-enable bucket versioning. 

`terraform apply`

```
  Terraform will perform the following actions:

    # aws_s3_bucket_versioning.bucket-1-set-versioning will be updated in-place
    ~ resource "aws_s3_bucket_versioning" "bucket-1-set-versioning" {
          id     = "tylers-unique-bucket-name-15lkj5"
          # (1 unchanged attribute hidden)

        ~ versioning_configuration {
            ~ status = "Suspended" -> "Enabled"
          }
      }

  Plan: 0 to add, 1 to change, 0 to destroy.

  Do you want to perform these actions?
    Terraform will perform the actions described above.
    Only 'yes' will be accepted to approve.

    Enter a value: yes

  aws_s3_bucket_versioning.bucket-1-set-versioning: Modifying... [id=tylers-unique-bucket-name-15lkj5]
  aws_s3_bucket_versioning.bucket-1-set-versioning: Modifications complete after 2s [id=tylers-unique-bucket-name-15lkj5]

  Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

Now, we can be assured that bucket versioning has been re-enabled. 


## Local vs Remote State Files

When working with Terraform we have two options for storing resource state, locally and remotely. The local option is what we saw previously, the `terraform.tfstate` file within our Configuration directory. HashiCorp also offers [several options](https://developer.hashicorp.com/terraform/language/state/remote-state-data) for storing this state remotely, however. 

Let's talk through the use cases here. Local Terraform state is fine but what happens when you're on a team and someone else needs to update the infrastructure? That team member would need a copy of the local `terraform.tfstate` file to modify that infrastructure. While you could share this file with that team member, it's not a great idea. Instead, you should use a Remote Terraform state file that all team members have access to. Now you don't need to worry about passing around a file, understanding which is the most up-to-date, worrying about file corruption, and so on and so forth.  

#### Setting up a Remote State 

We're going to modify the `provider.tf` file and set up the new remote state. We'll go ahead and use our existing S3 bucket to hold this file. 

*Refer to the [Terraform documentation](https://developer.hashicorp.com/terraform/language/settings/backends/s3) for understanding how to enable an S3 bucket backend.* 

**Original `provider.tf`**
```
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = ">= 5.5.0"
      }
    }
  }

  provider "aws" {
    region = "us-east-1"
  }
```

**Updated `provider.tf`**
```
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = ">= 5.5.0"
      }
    }
    backend "s3" {
      bucket            = "tylers-unique-bucket-name-15lkj5"
      key               = "terraform.tfstate"
      region            = "us-east-1"
      encrypt           = true
    }
  }

  provider "aws" {
    region = "us-east-1"
  }
```

#### Migrating from Local State to Remote State

Let's review what we did here - 
1. We defined the *backend block* i.e., `backend "s3"`
1. We defined the bucket our remote state file should be stored in (this is the bucket id) i.e., `tylers-unique-bucket-name-15lkj5`
1. We defined a unique key to store our current S3 bucket resource i.e., `terraform.tfstate`
1. We defined the region where this data should be stored i.e., `us-east-1`
1. We enabled server-side encryption of the state file i.e., `true`

***Note:*** before running the next command, make sure the local state files exist in your Configuration Directory. (They should if you've followed along). Otherwise the state won't automatically migrate and you'll have to either undo/redo these steps or import each resource into the new state. (We'll cover importing resources in a later module).

Now, run `terraform init` to initialize our new backend i.e., the S3 bucket. You should see a message to copy existing state (from the local file) to the new remote file. 

```
  Initializing the backend...
  Do you want to copy existing state to the new backend?
    Pre-existing state was found while migrating the previous "local" backend to the
    newly configured "s3" backend. No existing state was found in the newly
    configured "s3" backend. Do you want to copy this state to the new "s3"
    backend? Enter "yes" to copy and "no" to start with an empty state.

    Enter a value: yes


  Successfully configured the backend "s3"! Terraform will automatically
  use this backend unless the backend configuration changes.
```

Next, run `terraform plan` and you should see the state require no changes. This indicates that we've successfully migrated from a local state file to the new remote state file in our S3 bucket!

```
  aws_s3_bucket.bucket-1: Refreshing state... [id=tylers-unique-bucket-name-15lkj5]
  aws_s3_bucket_versioning.bucket-1-set-versioning: Refreshing state... [id=tylers-unique-bucket-name-15lkj5]

  No changes. Your infrastructure matches the configuration.

  Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
```

Next, we can check our S3 bucket and see the new remote state file. Run the command `aws s3 ls s3://tylers-unique-bucket-name-15lkj5`

```
  2023-07-01 19:36:02       3967 terraform.tfstate
```

Feel free to now remove the local state files. `rm terraform.tfstate terraform.tfstate.backup`

## Information Disclosure in Terraform State Files

While State files are a key feature of Terraform, we need to be cognizant of the data held in these files and take the necessary measures to protect them. We know that if we view these state files, we'll find all sorts of information related to our resources. Additionally, if you've hard-coded secrets or used particular Terraform resource types to upload or read secrets, these can all end up in the Terraform state file.  

We'll cover secrets management in a future module but do make sure to always protect the Terraform state file. In our case we've uploaded it to an S3 bucket and encrypted it with server-side encryptioin. We also need to ensure access to the file and the ability to read it is locked down. This can be done with AWS IAM policy or an S3 Bucket Policy.