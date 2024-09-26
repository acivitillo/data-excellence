# Infrastructure as Code (IaC) <a name="iac"></a>


Infrastructure as Code (IaC) is a practice that involves managing and provisioning infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools. This approach enables teams to automate infrastructure deployment and manage it with the same tools and processes used for software development.

## Terraform for Cloud Environments <a name="iac_terraform"></a>
### What is Terraform?

Terraform is an open-source IaC tool that allows you to define cloud and on-premises resources in human-readable configuration files that you can version, reuse, and share. It supports multiple providers like AWS, Azure, Google Cloud Platform, and many others.

### Benefits of Using Terraform <a name="iac_terraform_benefits"></a>

**Consistency**: Ensures that all environments are provisioned identically.
**Scalability**: Simplifies the management of large-scale infrastructures.
**Version Control**: Infrastructure configurations can be stored in version control systems.
**Modularity**: Encourages the use of modules for reusable infrastructure components.
**Collaboration**: Teams can work together on infrastructure code just like application code with state of the cluster stored on a shared place like AWS S3 or Azure Blob Storage.

### Setting Up a CI/CD Pipeline with Terraform <a name="iac_cicd_setup"></a>
Setting up CI/CD pipeline with Terraform is fairly simple process, as long as we are starting new project. It is possible to import existing resources and use them further within Terraform, but it's not covered in below example.

1. **Writing Infrastructure Code**
    - Define resources using HashiCorp Configuration Language (HCL).
    - Organize code into modules for reusability.
    - Store code in a version control system like Git.
2. **Version Control Integration**
    - Use branching strategies (e.g., GitFlow) to manage code changes.
    - Implement pull requests for code reviews.
3. **Continuous Integration**
    - **Validation**: Use terraform validate to check the syntax.
    - **Linting**: Use tflint to ensure code quality and adherence to best practices.
    - **Security Scanning**: Use tools like tfsec to identify potential security issues.
4. **Continuous Deployment**
    - **Terraform Plan**: Generate an execution plan to preview changes.
    - **Approval Process**: Implement manual approvals for changes to critical environments.
    - **Terraform Apply**: Apply the changes to the infrastructure.
    - **State Management**: Use remote backends (e.g., AWS S3, Azure Blob Storage) for storing the state file securely.
5. **Monitoring and Logging**
    - Integrate logging and monitoring tools to track infrastructure changes and performance.
    - Use services like AWS CloudWatch or Azure Monitor.

### Example CI/CD Workflow with Terraform and GitHub Actions <a name="iac_cicd_example"></a>
```
name: Terraform CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  terraform:
    name: Terraform Plan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Init
        run: terraform version

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan

  approval:
    runs-on: ubuntu-latest
    environment: plan_approve
    steps:
      - run: echo "Terraform plan approved, terraform apply will be executed"

  terraform_apply:
    name: Terraform apply
    needs: [ approval ]
    runs-on: ubuntu-latest
    steps:
      - name: Terraform Apply
        run: terraform apply
```
The setup described above assumes the use of a single environment and a GitHub Enterprise license. However, this process can be easily adapted for multiple environments.

For example, in a development environment, you may choose to omit the pull request approval process or assign an internal team for approvals, as the changes are not yet applied to production. Once changes are merged into the main or default branch, the full approval and deployment process will apply accordingly. Environment `plan_approve` has Enterprise feature enabled with deployment protection rule that is forcing manual review.

### Terraform best practices <a name="iac_best_practices"></a>
1. **Use Remote State Storage**: Keep the Terraform state file in a remote backend with locking to prevent conflicts. AWS S3 or Azure Blob Storage are the most common examples.
2. **Sensitive Data Management**: Do not hard-code secrets; use environment variables or secret management tools. [Sops](https://github.com/getsops/sops) is also good idea which enables keeping encoded secrets in repository. 
3. **Modular Code**: Break infrastructure code into modules for better organization and reuse.
4. **Automated Testing**: Implement automated tests for infrastructure code using tools like Terratest.

## Ansible for On-Premises Environments <a name="ansible"></a>
### What is Ansible? <a name="ansible_overview"></a>

Ansible is an open-source automation tool used for configuration management, application deployment, and task automation. It uses a simple, agentless architecture and relies on standard SSH for communication. While it's not used that frequently for cloud environments, it's still a powerful tool for On-Premise environments.

### Benefits of Using Ansible <a name="ansible_benefits"></a>

**Agentless Architecture:** No need to install agents on managed nodes.
**Easy to Learn:** Uses YAML for playbooks, which is human-readable.
**Idempotent:** Ensures that applying configurations multiple times does not produce unintended changes.
**Extensible:** Supports custom modules and plugins.

### Setting Up a CI/CD Pipeline with Ansible <a name="ansible_cicd_setup"></a>

Setting up a CI/CD pipeline with Ansible involves writing playbooks and roles using YAML syntax to define tasks, organizing these tasks into modular roles for reusability, and defining inventories to specify groups of hosts. The playbooks and related files are stored in a version control system like Git, allowing for effective code management through branching strategies and pull requests.

Continuous integration practices are applied by performing syntax checks with `ansible-playbook --syntax-check`, enforcing code quality and best practices with ansible-lint, and utilizing testing tools like Molecule to validate roles and playbooks locally.

For continuous deployment, the pipeline automates the execution of playbooks against the inventory to configure systems, manages sensitive data using variables and Ansible Vault, and enhances deployment efficiency through parallel execution of tasks. This unified approach streamlines configuration management and ensures that on-premises environments are consistently and reliably configured through automated processes.

### Example CI/CD Workflow with Ansible and Gitlab CI/CD <a name="ansible_cicd_example"></a>
```
stages:
  - test
  - deploy

variables:
  ANSIBLE_HOST_KEY_CHECKING: 'False'

before_script:
  - apt-get update && apt-get install -y ansible

test:
  stage: test
  script:
    - ansible-playbook playbook.yml --syntax-check
    - ansible-lint playbook.yml

deploy:
  stage: deploy
  script:
    - ansible-playbook -i inventory playbook.yml --vault-password-file vault_pass.txt
  environment: production
  when: manual
```
In Gitlab CI/CD, managing manual approval is handled slightly different. Basic manual step is possible with usage of `when: manual`, but simplest way to limit group of people that can approve it can be done by making protection environment and assigning only people capable of doing any changes there.
