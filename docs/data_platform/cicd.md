CI/CD process for deploying and managing a data platform is visible in multiple places. Starting from infrastructure, there should be no place for manually created server, network or any other resources, with tools like Terraform or Ansible managing all of it. This part is explained further in [Infrastructure as Code page](iac.md). Second CI/CD layer relates to application part, where all changes within containers running on a cluster should be tested and prepared in an automated way. Final place for CI/CD is within data flows that are going to be executed on orchestrator like Prefect, Apache Airflow or Dagster. This part shouldn't allow manual changes with everything being hosted on repository like GitHub or Gitlab. Here we will focus on typical CI/CD for application part, and also on Data Flow Automation

# Application Deployment Automation <a name="cicd"></a>
In this chapter, we’ll dive into the technical details of setting up a robust CI/CD pipeline using GitHub workflows, Dockerfiles, and Helm charts for deployments on a Kubernetes cluster. By combining these tools, we can streamline the process of building, testing, and deploying applications efficiently. We’ll walk through how GitHub workflows automate tasks, Dockerfiles enable consistent application environments, and Helm charts facilitate scalable Kubernetes deployments—ultimately creating a seamless pipeline from code commit to production-ready deployments. This setup is essential for maintaining agility, reliability, and scalability in modern software development practices.

1. Set up GitHub repository
We should start with creating or configuring a GitHub repository for a project. In our scenario we will use `main` branch that will be protected. Good start for `Branch protection rule` for `main` branch should look like this:
- `Require a pull request before merging` with `Require approvals: 1` should be turned on. With such configuration we are sure that review process is established for repository
- `Require status checks to pass before merging` with `Require branches to be up to date before merging` and CI job we will create in next steps should be added in `Status checks that are required`. This way we will be sure that our automation is passing on latest main code and there is no way to merge any pull request without it. By default GitHub is not forcing workflows to pass before merging.

There are more options worth considering, but they should be applied according to project's needs.

2. Create GitHub Workflows
All GitHub Actions workflows should be stored in `.github/workflows` folder, where we can define multiple workflows. For our CI/CD pipeline we can start with such `on` clause:
```
on:
  pull_request:
    branches: [ main ]
    paths:
      - "path_with_code_changes/**"
  workflow_dispatch:
```
This way workflow will be executed every time any change appears within pull request. There is also possibility to run workflow manually with `workflow_dispatch`, where we can also add additional properties like `debug` in case we need more information about failing workflow.

Inside, we can put basic CI checks, depending on technology we are using, below is simple example for parsing dbt:
```
  validate_dbt_project:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4.2.0
      - uses: actions/setup-python@v5.2.0
        with:
          python-version: "3.12"

      - name: Install dependencies
        env:
          DBT_ENV_SECRET_GITHUB_TOKEN: ${{ secrets.SECRET_GH_TOKEN }}
          PIP_ROOT_USER_ACTION: ignore
        run: pip install -qr path_with_code_changes/requirements.txt --no-build-isolation

      - name: Install dbt package depenedencies
        working-directory: path_with_code_changes
        env:
          DBT_ENV_SECRET_GITHUB_TOKEN: ${{ secrets.SECRET_GH_TOKEN }}
        run: dbt -q deps

      - name: Mount profiles.yml
        # Required by dbt parse.
        # The var is the contents of the profiles.yml, encoded in base64:
        # echo .github/workflows/act/profiles.yml | base64
        env:
          PROFILES_YML_BASE64: ${{ vars.PROFILES_YML_BASE64 }}
        run: echo $PROFILES_YML_BASE64 | base64 --decode > path_with_code_changes/profiles.yml

      - name: Validate the dbt project
        working-directory: path_with_code_changes
        env:
          DBT_ENV_SECRET_GITHUB_TOKEN: ${{ secrets.SECRET_GH_TOKEN }}
        run: dbt -q parse
```

Such workflow can be save within `ci.yml` file, and validate_dbt_project can be added to `Status checks that are required` explained in previous step. The thing that need to be watched out here is the fact, that `ci.yml` might not be executed for every change. To enable mandatory check, it's better to run such CI for every pull request.

In separate workflow file CD part can be prepared, where basic setup is focused on building and pushing docker image. `on` clause can be set to be executed only on main branch, or we can have separate job that will build docker image on pull request, with `push` being enabled only on main. Better approach is to check build process already on pull_request, that's why below pipeline will be used both for `pull_request` and `push`:
```
name: Build and Push Docker Image

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout the code
        uses: actions/checkout@v4.2.0

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3.6.1

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3.3.0
        with:
          registry: ghcr.io
          username: ${{ secrets.GITHUB_ACTOR }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build Docker Image
        id: docker_build
        uses: docker/build-push-action@v6.7.0
        with:
          context: .
          file: ./Dockerfile
          platforms: linux/amd64,linux/arm64
          tags: ghcr.io/${{ github.repository }}/my-image:tag
          push: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
```
This way on every change in our pull request, docker image will be build only, and once we are comfortable, after merging changed to `main`, push will happen to GitHub Container Registry (ghcr). In order to use image already on non-prod environment before merging changes, there is an option to modify tagging of the images, where we can use branch name as a tag on pull request, while using proper tagging for pushed changes to main.

Similarly we can handle lint, package and push process for helm charts. In pull request we can use chart-testing (ct) library that will validate correctness of our yml files, and once we are sure that it's working as expected, workflow executed on main branch should package and push helm chart.

3. 

# Data Flow Automation <a name="flow"></a>
TBD