# Terraform Providers
In the previous section, [(What is Terraform?)](../1_mod/2_terraform_overview.md) we briefly discussed what a Terraform Provider was, but as a reminder, Providers abstract API calls and enable Terraform to interact with services' APIs on our behalf. Providers exist for many solutions including AWS, Salesforce, Azure, and others. When viewing the [providers registry](https://registry.terraform.io/browse/providers) you'll come across providers that are created and maintained by different parties:
- **Official** providers that Hashicorp owns and maintains
- **Partner** providers that third-party companies create for their own APIs
- **Community** providers that are created and published by the community 

Hashicorp offers a [tutorial](https://developer.hashicorp.com/terraform/tutorials/providers-plugin-framework?utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS) for creating your own Provider if interested.

## Working with Providers
Now that you have a general understanding of the purpose of Providers, let's dive deeper into the AWS Provider. Check out the top snippet of our code from the previous example below. 

```
# Open your IDE then copy this code into a file e.g., `main.tf` and change the elements within the brackets < >
provider "aws" {
    region = "<AWS-REGION->"
}
```

This is what tells Terraform to download the AWS Provider once you run the `terraform init` command. Let's run that command again and notice the output starting on line 5. Terraform is searching for the AWS Provider and in our case downloading the latest version.

```
% terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.5.0...
- Installed hashicorp/aws v5.5.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!
```

Now, search this directory for hidden files `ls -alh` and you should see two new files starting with `.terraform`


```
total 32
drwxr-xr-x@ 7 tyler  staff   224 Jun 28 18:44 .
drwxr-xr-x@ 7 tyler  staff   224 Jun 28 18:44 ..
drwxr-xr-x@ 3 tyler  staff    96 Jun 28 18:44 .terraform
-rw-r--r--@ 1 tyler  staff  1376 Jun 28 18:44 .terraform.lock.hcl
-rw-r--r--@ 1 tyler  staff   432 Sep  6  2022 main.tf
```

If we continue to list `ls` the contents, we can see how Terraform downloaded the provider details here.

```
% ls .terraform/providers/registry.terraform.io/hashicorp/aws 
5.5.0
```

## Provider Versioning
While it's always best to continue using the latest supported Provider versions, there is an option to specify specific versions. The latest method to specify the version can be seen below. 

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
    region = "<AWS-REGION->"
}
```

We have four kinds of operators to specify a Provider's version. 

- `=` Allows specifying an exact version number e.g., `4.0` or `5.5.0`
- `!=` Allows excluding an exact version number
- `>` `>=` `<` `<=` Allows for versions where the comparison is true e.g., `>=5.0` allows for any version `5.0` or newer
- `~>` Allows the *rightmost* version component to increment e.g., `~> 5.0` allows for `5.1` or `5.2` but not `5.1.1`

If Provider version constraints are used, you can view them as shown below once running the `terraform init` command.

```
% cat .terraform.lock.hcl 
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.1.0"
  constraints = "5.1.0"

[snip]
```

The [AWS Provider documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#argument-reference) will be a regular resource to reference for any changes to new Provider versions as well as additional configuration options. 