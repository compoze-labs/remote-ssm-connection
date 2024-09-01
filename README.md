# remote-ssm-connection

remote-ssm-connection is a reusable [composite github action](https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-composite-action) to enable pipelines to perform [port forwarding](https://aws.amazon.com/blogs/aws/new-port-forwarding-using-aws-system-manager-sessions-manager/) to remote instances within AWS. The initial design is to facilitate automated database migrations via CI, in Github Actions pipelines. However, it can be used to facilitate other secure remote connection use cases.

## Usage
``` yaml
uses: compoze-labs/remote-ssm-connection@main
with:
    instance-id: ${{ secrets.INSTANCE_ID }}
    host: ${{ secrets.HOST }}
    local-port: 5431
    remote-port: 5432
    aws-region: ${{ secrets.AWS_REGION }}
    role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

| Parameter | Default Value | Description |
| --------- | --------- | ----------- |
| instance-id | **Required** | Id of the Ec2 instance that has the ssm agent installed. |
| host | **Required** | Url of the remote instance to forward traffic to. For example, an RDS instance |
| local-port | **Required** | Port to forward traffic to on the host machine. |
| remote-port | **Required** | Port of the remote instance to forward traffic to. |
| aws-region | **Required** | Region of the remote instance. |
| role-to-assume | **Required** | Role ARN to assume for interactign with AWS SSM agent. |

## Example
Below is an example Github Actions configuration that starts the remote connection, installs the database migration tool [dbmate](https://github.com/amacneil/dbmate), and runs the migration command.

``` yaml
name: Test

on:
  workflow_dispatch:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:

  migration:
    runs-on: ubuntu-latest
    permissions:
        id-token: write
        contents: read    
    steps:
      - uses: actions/checkout@v4
      - name: Run action
        uses: compoze-labs/remote-ssm-connection@main
        with:
          instance-id: ${{ secrets.INSTANCE_ID }}
          host: ${{ secrets.HOST }}
          local-port: 5431
          remote-port: 5432
          aws-region: ${{ secrets.AWS_REGION }}
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
      
      - name: Install dbmate
        run: |
          sudo curl -fsSL -o /usr/local/bin/dbmate https://github.com/amacneil/dbmate/releases/latest/download/dbmate-linux-amd64
          sudo chmod +x /usr/local/bin/dbmate     
      - name: Migate Database
        run: |
          make migrate
        shell: bash
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

## License

[MIT](./LICENSE).