### HOW TO DISPLACE AWS CONNECTED ACCOUNT INFORMATION

The terraform snippet below will diplace the details of the AWS account you are connected to.
The information includes:

aws_account_id = "654654474411"
current_user_account_id = "654654474411"
current_user_arn = "arn:aws:iam::654654474411:user/Terrform"

```
provider "aws" {
  region = "us-east-1"  # Replace with your desired region
}

# Data source to retrieve current caller identity (user)
data "aws_caller_identity" "current" {}

# Output current user's details
output "current_user_arn" {
  value = data.aws_caller_identity.current.arn
}

output "current_user_account_id" {
  value = data.aws_caller_identity.current.account_id
}

# Output AWS account ID
output "aws_account_id" {
  value = data.aws_caller_identity.current.account_id
}


```