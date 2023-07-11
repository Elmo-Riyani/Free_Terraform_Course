# Who is this course for?
While I believe anyone can learn anything with the right level of perseverance and determination, this course assumes you are at least comfortable with the command line, basic scripting, and general AWS. 

## Prerequisites
Before staring this course you'll need a few things setup and installed. The below will walk you through this. 

1. [A computer or virtual machine running Windows, MacOS, or Linux](#computer--virtual-machine)
1. [An IDE](#ide)
1. [An AWS account](#aws-account)
1. [An AWS IAM user](#aws-iam-user)
1. [Terraform](#terraform-install)


### Computer / Virtual Machine
During this course we will be utilzing Terraform for deploying infrastructure-as-code to an AWS account. You will need a computer or virtual machine running an operating system e.g., Windows, MacOS, or Linux.


### IDE
An Integrated Development Environment (IDE) is a useful software for writing code. While it's not a *requirement* to use an IDE for writing code, plugins certainly makes it easier by offering syntax highlighting and intellisense for example! Consider the following options:
- [Visual Studio Code](https://code.visualstudio.com/) (my personal favorite!)
- [PyCharm](https://www.jetbrains.com/pycharm/)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/)


### AWS Account
You will need an AWS account during this course - create one [here](https://portal.aws.amazon.com/billing/signup#/start/email) if needed. I recommend to **use your own** AWS account for learning!

There should be minimal expense if any during this course as AWS provides [12 months free](https://docs.aws.amazon.com/whitepapers/latest/how-aws-pricing-works/get-started-with-the-aws-free-tier.html) of select services when creating a new account. **You are fully responsible for any cost!** Here's an [additonal reference on AWS billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/checklistforunwantedcharges.html) should you want to understand more.

Some tips to consider:
- Some email services (such as Gmail) support adding a **+** at the end of your email. This is useful if your email is already associated with an AWS account and you're not wanting to manage a second email account. AWS will see this as a new email. For example, if your email is ```yourname@gmail.com``` you can do this ```yourname+terraformtraining@gmail.com``` Any email sent to this address will be sent to your regular email address. 
- Add [MFA to your AWS account](https://aws.amazon.com/iam/features/mfa/) to further secure it. A hardware or software app (e.g., Google Authenticator) can be used. 
- After account creation you will have credentials to a root user - **do not use this account**. Instead create a new IAM user to use for daily operations and throughout this course. 


### AWS IAM User
It's best practice not to use the root user for AWS operations as this is a powerful account. Instead follow this AWS tutorial to [create a new IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) for use with this course. When you get to the **Set permissions** step (step #7 at time of this writing) I recommend attaching the **Administrators** policy to this user. This has less permissions than the root account and will allow you to fully complete the exercises in this course. Of course best practice is to follow the principle of least privilege (i.e., only have the permissions you need and no more) so do that if you understand what you're doing.


### Terraform Install
For up-to-date instructions to install Terraform, refer to [HashiCorp's website](https://developer.hashicorp.com/terraform/downloads?ajs_aid=ea57c591-7f6e-494c-8f41-5117fa1b6aed&product_intent=terraform) (makers of Terraform).