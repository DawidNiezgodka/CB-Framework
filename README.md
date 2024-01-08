# CB-Framework
Continuous Benchmarking Framework that facilitates and provides end-to-end automation of (application) benchmarks

# Description

A reusable workflow for integrating
the concept of continuous benchmarking into your
CI/CD pipeline.
Currently, the workflow is configured to run on GCP.
Future versions will include support for AWS and Azure.

Features:
- [x] Performs necessary authentication
- [x] Performs input validation
- [x] Provisions the infrastructure that represents the benchmarking system using Terraform (automates authentication to GCP)
- [x] Automatically provides the benchmarking infrastructure with the necessary software using Ansible
- [x] Enables you to structure the benchmark as a process consisting of an arbitrary number of phases
- [x] ...

## How to use it

To successfully run the workflow, you need to have the following:

### GCP

1. You need a GCP account
2. You need a GCP project
3. You must enable the following APIs for your project: ...
4. You need a GCP service account with the following permissions...

Below, you will find a reference to a repository that allows you to programmatically
create a service account and enable the necessary APIs using Terraform.
Furthermore, using the logic in the repository, you will be able to extract
the private key of the service account and store it as a GitHub secret.

5. You need the private key of the service account.

### GitHub

1. You need to have a GitHub account
2. You need to have a GitHub repository
3. You need to provide the following variables via GitHub secrets:  
   **For GCP**:
    - `GCP_PROJECT_ID`: The ID of your GCP project
    - `GCP_SERVICE_ACCOUNT_EMAIL`: The name of your GCP service account
    - `GCP_SERVICE_ACCOUNT_KEY`: The private key of your GCP service account. Please provide the complete JSON structure that was generated. Apart from the key itself, this structure includes fields such as "type" and "project id"
    - `GCP_USER`: The user that will be used to authenticate to GCP. For now, only a single user is supported.
    - `GCP_BUCKET_NAME`: The name of the bucket where the Terraform state is stored.
      The bucket must exist before the workflow is run.
      The workflow will create a corresponding Terraform resource so that the state can be read from the bucket.  
      **For Ansible**:
    - `ANSIBLE_KEY`: The private key that will be used to provision the benchmarking infrastructure with Ansible
    - `ANSIBLE_USER`: The user that will be used to authenticate to the benchmarking infrastructure.
      For now, only a single user is supported.
      Currently, it must be the same as GCP_USER
      **Misc**
    - `TOKEN_FOR_PRIVATE_REPOS`: The token that will be used to check out the private repositories 
    - (for example, with your own Terraform configuration or with a benchmark) or to push data to repositories.
    -

# Troubleshooting

This section contains remarks that might help you to solve your problem.

## General remarks

1. Check if provided secrets are correct.
2. Check if tokens are valid and not expired.
3. Check if tokens have the necessary permissions. At least read/write content.
4. Furthermore, in your repository settings, go to actions and change the workflow permissions to "Workflows have read and write permissions in the repository for all scopes."
5. Check if you have a matching pair of a private key and the corresponding public key.
6. When adding private keys to GitHub secrets, make sure that you do not add any newlines.
   Also make sure to add `-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----` to the key.
7. When adding GCP service account key, you must provide the complete JSON.

## Terraform (infrastructure provisioning workflow)

If you get the following error message:
```
Error: Error acquiring the state lock

Error message: writing "gs://***/default.tflock" failed:
googleapi: Error 412: At least one of the pre-conditions you specified did
not hold., conditionNotMet
Lock Info:
  ID:        12345
  Path:      gs://***/default.tflock
  Operation: OperationTypeApply
  Who:       root@abc
  Version:   1.2.3
  Created:   2023-12-05 16:58:04.478471711 +0000 UTC
  Info:    
```
then you must unlock the state. To do so, add the following input to your workflow:
```
infra_lock_id: "12345"
```
where `"12345"` is the ID of the lock that you want to unlock (replace with the one
you see in your error message and provide as a string not as a number).

