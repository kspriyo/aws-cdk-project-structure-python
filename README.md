# Recommended AWS CDK project structure for Python applications
The project implements a *user management backend* component that uses 
Amazon API Gateway, AWS Lambda and Amazon DynamoDB to provide basic CRUD operations 
for managing users. The project also includes a continuous deployment pipeline.

![diagram](https://user-images.githubusercontent.com/4362270/190571364-50987a50-1168-4ff2-95e5-61d9bd680d6e.png)
\* Diagram generated using https://github.com/pistazie/cdk-dia

## Create a new repository from aws-cdk-project-structure-python
This project is a template. Click “Use this template” (see the screenshot below) in 
the repository [main page](https://github.com/aws-samples/aws-cdk-project-structure-python)
to create your own repository based on aws-samples/aws-cdk-project-structure-python. 
This is optional for deploying the component to the development environment, but 
**required** for deploying the pipeline.

![template](https://user-images.githubusercontent.com/4362270/128629234-31cd275e-6a3f-4a6a-9010-028a0a279950.png)

The instructions below use the aws-cdk-project-structure-python repository.

## Create development environment
See [Getting Started With the AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html)
for additional details and prerequisites

### Clone the code
```bash
git clone https://github.com/aws-samples/aws-cdk-project-structure-python
cd aws-cdk-project-structure-python
```

### Create Python virtual environment and install the dependencies
```bash
python3.7 -m venv .venv
source .venv/bin/activate

# [Optional] Needed to upgrade dependencies and cleanup unused packages
# Pinning pip-tools to 6.4.0 and pip to 21.3.1 due to
# https://github.com/jazzband/pip-tools/issues/1576
pip install pip-tools==6.4.0
pip install pip==21.3.1

./scripts/install-deps.sh
./scripts/run-tests.sh
```

### [Optional] Upgrade AWS CDK Toolkit version
**Note:** If you are planning to upgrade dependencies, first push the upgraded AWS CDK Toolkit version.
See [(pipelines): Fail synth if pinned CDK CLI version is older than CDK library version](https://github.com/aws/aws-cdk/issues/15519) 
for more details.

```bash
vi package.json  # Update "aws-cdk" package version
./scripts/install-deps.sh
./scripts/run-tests.sh
```

### [Optional] Upgrade dependencies (ordered by constraints)
Consider [AWS CDK Toolkit (CLI)](https://docs.aws.amazon.com/cdk/latest/guide/reference.html#versioning) compatibility 
when upgrading AWS CDK packages version.

```bash
pip-compile --upgrade api/runtime/requirements.in
pip-compile --upgrade requirements.in
pip-compile --upgrade requirements-dev.in
./scripts/install-deps.sh
# [Optional] Cleanup unused packages
pip-sync api/runtime/requirements.txt requirements.txt requirements-dev.txt
./scripts/run-tests.sh
```

## Deploy the component to sandbox environment
The `UserManagementBackendSandbox` stack uses your default AWS account and region.

```bash
npx cdk deploy UserManagementBackendSandbox
```

Example outputs for `npx cdk deploy UserManagementBackendSandbox`:
```text
 ✅  UserManagementBackendSandbox

Outputs:
UserManagementBackendSandbox.APIHandlerArn = arn:aws:lambda:eu-west-1:123456789012:function:UserManagementBackendSandbox-APIHandler-C9PR5KQGMbpk
UserManagementBackendSandbox.APIHandlerName = UserManagementBackendSandbox-APIHandler-C9PR5KQGMbpk
UserManagementBackendSandbox.EndpointURL = https://gtvk7d9k8h.execute-api.eu-west-1.amazonaws.com/v1/
UserManagementBackendSandbox.RestAPIId = gtvk7d9k8h
```

## Deploy the toolchain
**Prerequisites**
- Create a new repository from aws-cdk-project-structure-python, if you haven't done 
  this already. See [Create a new repository from aws-cdk-project-structure-python](README.md#create-a-new-repository-from-aws-cdk-project-structure-python)
  for instructions
- Create AWS CodeStar Connections [connection](https://docs.aws.amazon.com/dtconsole/latest/userguide/welcome-connections.html)
  for the pipeline
- Update the toolchain account in [app.py](app.py) 
- Update the toolchain constants in [toolchain.py](toolchain.py)
- Commit and push the changes: `git commit -a -m 'Update toolchain account and constants' && git push`

```bash
npx cdk deploy UserManagementBackendToolchain
```

## Delete all stacks
**Do not forget to delete the stacks to avoid unexpected charges**
```bash
npx cdk destroy UserManagementBackendSandbox
npx cdk destroy UserManagementBackendToolchain
npx cdk destroy UserManagementBackendToolchain/Pipeline/Production/UserManagementBackend
```

Delete the AWS CodeStar Connections connection if it is no longer needed. Follow the instructions
in [Delete a connection](https://docs.aws.amazon.com/dtconsole/latest/userguide/connections-delete.html).

## Testing the web API
Below are examples that show the available resources and how to use them:

```bash
endpoint_url=$(aws cloudformation describe-stacks \
  --stack-name UserManagementBackendSandbox \
  --query 'Stacks[*].Outputs[?OutputKey==`EndpointURL`].OutputValue' \
  --output text)

curl \
    -H "Content-Type: application/json" \
    -X POST \
    -d '{"username":"john", "email":"john@example.com"}' \
    "${endpoint_url}/users"

curl \
    -H "Content-Type: application/json" \
    -X GET \
    "${endpoint_url}/users/john"

curl \
    -H "Content-Type: application/json" \
    -X PUT \
    -d '{"country":"US", "state":"WA"}' \
    "${endpoint_url}/users/john"

curl \
    -H "Content-Type: application/json" \
    -X DELETE \
    "${endpoint_url}/users/john"
```

# Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

# License

This code is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.
