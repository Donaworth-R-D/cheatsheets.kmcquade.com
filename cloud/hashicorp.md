---
description: Various HashiCorp tooling
---

# HashiCorp Terraform, Packer, Consul

## Terraform <a id="terraform"></a>

### Recursively remove .terraform folders

It may happen that Terraform working directory \(`.terraform`\) already exists but not in the best condition \(eg, not initialized modules, wrong version of Terraform, etc\). To solve this problem you can find and delete all `.terraform` directories in your repository using this command:

```text
find . -type d -name ".terraform" -print0 | xargs -0 rm -r
```

### Using AWS SSM Parameter Store to stash terraform.tfvars variables

* Store a parameter at:
  * `/some-parameter-path/stage/1`
  * `/some-parameter-path/stage/2`

```text
some_text="my_list = $(aws ssm get-parameters-by-path --output=json --with-decryption --path=/some-parameter-path/stage | jq -r '.Parameters | map(.Value)')"
echo "$some_text" > terraform.tfvars 
```

Then the contents of Terraform.tfvars will be:

```text
my_list = [
  "stage1value",
  "stage2value"
]
```

#### Terraform Output: Conditional creation of resources in a module

```text
output "ecs_service_name" {
  value = "${element(concat(aws_ecs_service.app.*.name, aws_ecs_service.app_no_load_balancer.*.name, list("")), 0)}"
  #value = "${aws_ecs_service.app.name}"
}
```

Or:

```text
output "ssh_security_group_name" {
  description = "The name of the security group managing SSH traffic from on-premises to the VPC"
  value = "${element(concat(aws_security_group.ssh.*.name, list("")), 0)}"
  //value = "${aws_security_group.ssh.name}"
}

```

## Packer <a id="packer"></a>

#### Login shells

```text
    {
      "type": "shell",
      "execute_command": "sudo chmod +x {{ .Path }}; sudo -S -E -i env {{ .Vars }} {{ .Path }}",
      "scripts": [
        "scripts/install/install-base-aws.sh",
        "scripts/install/install-rbenv-systemwide.sh"
      ],
      "environment_vars": [
        "SSH_USERNAME={{ user `ssh_username` }}"
      ]
    },
```

#### Skipping post processors

```text
jq 'del(."post-processors")' packer-local-virtualbox-troubleshooting.json | packer build -
```

## Consul

### Snapshots: List the most recent consul snapshot file in an S3 bucket folder

```text
aws s3 ls s3://bucket-name/consul-snapshot-uat/ --recursive | sort | tail -n 1
```

## Vault

### Local Server

```text
vault server -dev
export VAULT_ADDR='http://127.0.0.1:8200

```

* Launch new session

```text
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root-token'
```

