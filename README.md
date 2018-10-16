# Node.js App Deployment

Repo containing a basic 'Hello World' node.js web application for the purposes of demonstrating cloud deployment.

## Requirements

1. AWS CLI installed and configured with **default** profile, **including access key and secret** - See [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) for details.
2. Your IAM user account will need sufficient access to create the infrstructure. Granting `Administrator` access is easiest, but production environments should generally follow principle of least privilege.
3. [Terraform](https://terraform.io). *Note: The deployment script will assume the `terraform` binary is in your `$PATH`*.
4. Docker CE installed and running.

## Setup and Run

Clone the repo:

```bash
git clone git@github.com:jaymickey/node-hello-world.git
```

- Configure your AWS CLI default profile, along with `Access Key` and `Access Secret`: Instructions can be found [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).
- Ensure `terraform` is in your `$PATH`. [Windows](https://www.howtogeek.com/118594/how-to-edit-your-system-path-for-easy-command-line-access/) | [Linux](https://www.techrepublic.com/article/how-to-add-directories-to-your-path-in-linux/)
- Execute the deployment script:
  - `deploy.sh create` - Deploy or update the application. Terraform is idempotent, so it will only update resources as required. Docker also caches layers, so will only push a new image if there are changes. A new deployment of the container in ECS is forced as part of the script. **Note**: The service can take 1-2 minutes to start on the first deploy!
  - `deploy.sh destroy` - Kill the application and all related infrastructure. **WARNING:** This **WILL** destroy all the infrastructure managed by terraform with *extreme prejudice*. There is no confirmation and cancelling after the 5 second window will likely leave your infrastructure in an incomplete state.

## Resources

- **Terraform** - Terraform is used as the infrastucture as code solution for creating and destroying the cloud-based resources. Terraform offers a declarative syntax, and is idempotent.
- **AWS** - AWS was the chosen cloud provider. I chose this due to having great familiarity with the platform, and better support for Terraform than the alternatives. AWS also has a wide feature set and allows for global scalability.
- **Fargate** - AWS ECS & Fargate offer a simple, easily scalable solution to running applications in the cloud. The same image can be run on the developers machine and in production. Fargate also requires less overall configuration and management of infrastructure (e.g. EC2 VMs) for a slightly higher cost.

## Improvements

- The provided `terraform` will deploy the full AWS environment. Therefore each developer would most likely require their own AWS sandbox account. This is fine for testing as part of a development workflow, but production deployments will preferably be done via a CI/CD pipeline.
- The `deploy.sh` script provides little to no testing or guardrails. If something goes wrong it does not rollback gracefully, and could leave the AWS environment in an incomplete state. Again, these issues would best be dealt with via a CI/CD pipeline with Behaviour Driven Infrastructure (BDI) testing and automated rollbacks.
- The `deploy.sh` currently has little feedback, but this is preferrable in many cases to the spam of `terraform` and the `awc ecs` CLI. An improvement to this script would be to make the output level configurable.
- Currently the Fargate configuration is set to 2 containers, with no auto-scaling. This isn't difficult to add, but for the purposes of dev deployments it's fine.