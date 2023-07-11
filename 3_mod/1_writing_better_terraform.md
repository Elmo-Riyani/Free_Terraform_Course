# Writing Better Terraform
Up until this point we've been writing our code into a single file, `main.tf`. While this totally works, it's not best practice. Best practice would be to take advantage of the other file types that Terraform offers which will ultimately make our code easier to understand and more manageable. Let's continue configuring our S3 bucket code and focus on understanding these other file types.

## Terraform File Types
There's a standard naming convention with Terraform files and while you can deviate from this standard, I'd recommend not unless it makes sense to do so. Here's a quick overview before we create these:

| File | Description |
|----|----|
| `provider.tf` | Here is where you can configure things like state (covered later), the providers you'll use (e.g., AWS), provider versions and more.
| `main.tf` | Here is where you define your resources to be created (e.g., S3 bucket).
| `variables.tf` | Here is where we can declare variables which allows us to update the code in one place instead of multiple (e.g., the Tag scenario discussed above).
| `output.tf` | Here is where you can output resource attributes after they've been created (e.g., S3 bucket ARN)

## Updating our Terraform Code
So, let's now update our s3 bucket code (use below) by moving the Provider code into a new `provider.tf` file and replacing some key values with variables instead, using the `variables.tf` file.

**Original `main.tf`**
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
    region = "<AWS-REGION->"
}

resource "aws_s3_bucket" "bucket-1" {
  bucket = "<UNIQUE-BUCKET-NAME>"

  tags = {
    Team        = "<TEAM-THIS-BELONGS-TO>"
    Environment = "<ENV-DEPLOYED-IN>"
  }
}
```

**New `provider.tf` file**
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
    region = "<AWS-REGION->"
}
```

**New `variables.tf` file**
```
variable "aws_region" {
  description = "The aws region to deploy in."
  type = "string"
  default = "us-east-1"
}

variable "aws_bucket_name" {
  description = "The unique name of the AWS bucket."
  type = "string"
  default = "tylers-s3-bucket-dev-1ba3"
}

variable "aws_tagging" {
  description = "Resource tags."
  type        = map(string)
  default = {
    "Team"        = "security",
    "Environment" = "dev"
  }
}
```

**Updated `main.tf` file**
***Note:*** *you can also update your `provider.tf` file with variables as well. The same `variables.tf` file can apply to all your Terraform files!*

```
resource "aws_s3_bucket" "bucket-1" {
  bucket = var.aws_s3_bucket_name

  tags = var.aws_tagging
}
```
-----

Finally, let's create the `output.tf` file to output the bucket `id`, `ARN` and `bucket_domain_name` of the S3 bucket post creation. Refer to the documentation section [Attributes Reference](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket#attributes-reference) to learn of other exportable attributes for this resource. 

Here are two ways this can be done. 

```
# output.tf

output "s3_bucket_id" {
  description = "Outputs S3 bucket id"
  value = aws_s3_bucket.bucket-1.id
}

output "s3_bucket_arn" {
  description = "Outputs S3 bucket arn"
  value = aws_s3_bucket.bucket-1.arn
}

output "s3_bucket_domain" {
  description = "Outputs S3 bucket domain"
  value = aws_s3_bucket.bucket-1.bucket_domain_name
}

```

And now a second way. While it looks a bit complicated, all we're doing is referrencing the resource type (`aws_s3_bucket`), resource name (`bucket-1`), and exportable attributes (`id`, `arn`, `bucket_domain_name`) in a single resource block.

```
# output.tf

output "s3_bucket_details" {
  description = "Outputs attributes of our S3 bucket"
  value = [
    "${aws_s3_bucket.bucket-1.id}",
    "${aws_s3_bucket.bucket-1.arn}",
    "${aws_s3_bucket.bucket-1.bucket_domain_name}"
  ]
}
```

## Deploying the Updated Code

Now, let's deploy our bucket! Remember which commands to run?
- `terraform fmt`
- `terraform validate`
- `terraform init`
- `terraform plan`
- `terraform apply`

If all goes well, you should see an output similar to mine. 

```
s3_bucket_details = [
  "tylers-unique-bucket-name-15lkj5",
  "arn:aws:s3:::tylers-unique-bucket-name-15lkj5",
  "tylers-unique-bucket-name-15lkj5.s3.amazonaws.com",
]
```

## Wrap Up
Great job making it through this far! In this module we learned about the different types of Terraform files, their uses, and how we can utilize them to write better code.