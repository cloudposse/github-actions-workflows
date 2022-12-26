<!-- markdownlint-disable -->
## Workflows

| Name | Description |
|------|-------------|
| [CD - Deploy to ECS with Spacelift](#cd--deploy-to-ecs-with-spacelift) | Deploy Docker image to ECS with Spacelift |
| [CD - Deploy to EKS with Helmfile](#cd--deploy-to-eks-with-helmfile) | Deploy Docker image to EKS with Helmfile |
| [CD - Deploy to ECS QA/Preview envs with Spacelift](#cd--deploy-to-ecs-qapreview-envs-with-spacelift) | Deploy Docker image to ECS QA/Preview envs with Spacelift |
| [CD - Deploy to EKS Preview envs with Helmfile](#cd--deploy-to-eks-preview-envs-with-helmfile) | Deploy Docker image to ECS Preview envs with Helmfile |
| [CI - Build Docker image](#ci--build-docker-image) | Build Docker image and push it to ECR |
| [CI - Promote or build Docker image](#ci--promote-or-build-docker-image) | Promote or build Docker image and push it to ECR |
| [CI - Promote Docker image ](#ci--promote-docker-image) | Promote Docker image to specific version tag and push it to ECR |
| [CI - Verify Docker image exists](#ci--verify-docker-image-exists) | Verify Docker image exists on ECR |
| [Controller - Draft release](#controller--draft-release) | Create or update draft release |
| [Controller - Reingtegrate hotfix branch](#controller--reingtegrate-hotfix-branch) | Create PR into `target\_branch` to reintegrate hotfix from current branch  |
| [Controller - Create Release branch](#controller--create-release-branch) | Create `release/{version}` branch for the release  |
| [Controller - Create hotfix release](#controller--create-hotfix-release) | Create next patch version release  |
| [Controller - Monorepo Controller](#controller--monorepo-controller) | Mocked monorepo controller that outputs list of applications, lists of apps with and without changes. |




## CD - Deploy to ECS with Spacelift

Deploy Docker image to ECS with Spacelift

### Usage 

```yaml
  name: Deploy
  on:
    push:
      branches: [ main ]

  jobs:
    cd:
      uses: cloudposse/github-actions-workflows/.github/workflows/cd-ecs.yml@main
      with:
        image: registry.hub.docker.com/library/nginx
        tag: latest
        repository: ${{ github.event.repository.name }}
        environment: dev
        spacelift-organization: ${{ inputs.spacelift-organization }}
      secrets:
        secret-outputs-passphrase: "${{ secrets.secret-outputs-passphrase }}"
        github-private-actions-pat: "${{ secrets.github-private-actions-pat }}"
        spacelift-api-key-id: "${{ secrets.spacelift-api-key-id }}"
        spacelift-api-key-secret: "${{ secrets.spacelift-api-key-secret }}"
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| app | Application name. Used with monorepo pattern when there are several applications in the repo | string | N/A | false |
| environment | Environment name deploy to | string | N/A | true |
| image | Docker Image to deploy | string | N/A | true |
| matrix-key | Matrix key - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| matrix-step-name | Matrix step name - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| spacelift-organization | Spacelift organization name | string | N/A | true |
| tag | Docker Image tag to deploy | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |
| spacelift-api-key-id | Spacelift API Key ID | true |
| spacelift-api-key-secret | Spacelift API Key Secret | true |






## CD - Deploy to EKS with Helmfile

Deploy Docker image to EKS with Helmfile

### Usage 

```yaml
  name: Deploy
  on:
    push:
      branches: [ main ]

  jobs:
    cd:
      uses: cloudposse/github-actions-workflows/.github/workflows/cd-helmfile.yml@main
      with:
        image: registry.hub.docker.com/library/nginx
        tag: latest
        repository: ${{ github.event.repository.name }}
        environment: dev
      secrets:
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}
        github-private-actions-pat: ${{ secrets.github-private-actions-pat }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| environment | Environment name deploy to | string | N/A | true |
| image | Docker Image to deploy | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| tag | Docker Image tag to deploy | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## CD - Deploy to ECS QA/Preview envs with Spacelift

Deploy Docker image to ECS QA/Preview envs with Spacelift

### Usage 

```yaml
  name: Feature Branch
  on:
    pull_request:
      branches: [ 'master' ]
      types: [opened, synchronize, reopened, closed, labeled, unlabeled]

  jobs:
    cd:
      uses: cloudposse/github-actions-workflows/.github/workflows/cd-preview-ecs.yml@main
      if: ${{ always() }}
      with:
        image: registry.hub.docker.com/library/nginx
        tag: latest
        repository: ${{ github.event.repository.name }}
        spacelift-organization: ${{ inputs.spacelift-organization }}
        open: ${{ github.event.pull_request.state == 'open' }}
        labels: ${{ toJSON(github.event.pull_request.labels.*.name) }}
        ref: ${{ github.event.pull_request.head.ref }}
        exclusive: true
        env-label: |
          qa1: deploy/qa1
          qa2: deploy/qa2
          qa3: deploy/qa3
          qa4: deploy/qa4
      secrets:
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}
        github-private-actions-pat: ${{ secrets.github-private-actions-pat }}
        spacelift-api-key-id: "${{ secrets.spacelift-api-key-id }}"
        spacelift-api-key-secret: "${{ secrets.spacelift-api-key-secret }}"  
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| app | Application name. Used with monorepo pattern when there are several applications in the repo | string | N/A | false |
| env-label | YAML formatted {environment}: {label} map | string | preview: deploy<br> | false |
| exclusive | Deactivate previous GitHub deployments | boolean | true | false |
| image | Docker Image to deploy | string | N/A | true |
| labels | Pull Request labels | string | {} | false |
| matrix-key | Matrix key - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| matrix-step-name | Matrix step name - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| open | Pull Request open/close state. Set true if opened | boolean | N/A | true |
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| spacelift-organization | Spacelift organization name | string | N/A | true |
| tag | Docker Image tag to deploy | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |
| spacelift-api-key-id | Spacelift API Key ID | true |
| spacelift-api-key-secret | Spacelift API Key Secret | true |






## CD - Deploy to EKS Preview envs with Helmfile

Deploy Docker image to ECS Preview envs with Helmfile

### Usage 

```yaml
  name: Feature Branch
  on:
    pull_request:
      branches: [ 'master' ]
      types: [opened, synchronize, reopened, closed, labeled, unlabeled]

  jobs:
    cd:
      uses: cloudposse/github-actions-workflows/.github/workflows/cd-preview-helmfile.yml@main
      if: ${{ always() }}
      with:
        image: registry.hub.docker.com/library/nginx
        tag: latest
        repository: ${{ github.event.repository.name }}
        open: ${{ github.event.pull_request.state == 'open' }}
        labels: ${{ toJSON(github.event.pull_request.labels.*.name) }}
        ref: ${{ github.event.pull_request.head.ref }}
        exclusive: false
        env-label: |
          preview: deploy
      secrets:
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}
        github-private-actions-pat: ${{ secrets.github-private-actions-pat }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| env-label | YAML formatted {environment}: {label} map | string | preview: deploy<br> | false |
| exclusive | Deactivate previous GitHub deployments | boolean | true | false |
| image | Docker Image to deploy | string | N/A | true |
| labels | Pull Request labels | string | {} | false |
| open | Pull Request open/close state. Set true if opened | boolean | N/A | true |
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| tag | Docker Image tag to deploy | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## CI - Build Docker image

Build Docker image and push it to ECR

### Usage 

```yaml
  name: Deploy
  on:
    push:
      branches: [ main ]

  jobs:
    ci:
      uses: cloudposse/github-actions-workflows/.github/workflows/ci-dockerized-app-build.yml@main
      with:
        organization: ${{ github.event.repository.owner.login }}
        repository: ${{ github.event.repository.name }}
      secrets:
        ecr-region: ${{ secrets.ecr-region }}
        ecr-iam-role: ${{ secrets.ecr-iam-role }}
        registry: ${{ secrets.registry }}
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| registry | ECR Docker registry | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |



### Outputs

| Name | Description |
|------|-------------|
| image | Docker Image |
| tag | Docker image tag |



## CI - Promote or build Docker image

Promote or build Docker image and push it to ECR

### Usage 

```yaml
  name: Deploy
  on:
    push:
      branches: [ main ]

  jobs:
    ci:
      uses: cloudposse/github-actions-workflows/.github/workflows/ci-dockerized-app-promote-or-build.yml@main
      with:
        organization: ${{ github.event.repository.owner.login }}
        repository: ${{ github.event.repository.name }}
        force-build: false
      secrets:
        ecr-region: ${{ secrets.ecr-region }}
        ecr-iam-role: ${{ secrets.ecr-iam-role }}
        registry: ${{ secrets.registry }}
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| app | Application name. Used with monorepo pattern when there are several applications in the repo | string | N/A | true |
| force-build | Force build (skip promotion) | boolean | false | true |
| matrix-key | Matrix key - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| matrix-step-name | Matrix step name - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| registry | ECR Docker registry | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |



### Outputs

| Name | Description |
|------|-------------|
| image | Docker Image |
| tag | Docker image tag |



## CI - Promote Docker image 

Promote Docker image to specific version tag and push it to ECR

### Usage 

```yaml
  name: Release
  on:
    release:
      types: [published]

  jobs:
    ci:
      uses: cloudposse/github-actions-workflows/.github/workflows/ci-dockerized-app-promote.yml@main
      with:
        organization: ${{ github.event.repository.owner.login }}
        repository: ${{ github.event.repository.name }}
        version: ${{ github.event.release.tag_name }}
      secrets:
        ecr-region: ${{ secrets.ecr-region }}
        ecr-iam-role: ${{ secrets.ecr-iam-role }}
        registry: ${{ secrets.registry }}
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}

```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| app | Application name. Used with monorepo pattern when there are several applications in the repo | string | N/A | false |
| matrix-key | Matrix key - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| matrix-step-name | Matrix step name - matrix output workaround. [Read more](https://github.com/cloudposse/github-action-matrix-outputs-write#introduction) | string | N/A | false |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| version | New version tag | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| registry | ECR Docker registry | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |



### Outputs

| Name | Description |
|------|-------------|
| image | Docker Image |
| tag | Docker image tag |



## CI - Verify Docker image exists

Verify Docker image exists on ECR

### Usage 

```yaml
  name: Release
  on:
    release:
      types: [published]

  jobs:
    ci:
      uses: cloudposse/github-actions-workflows/.github/workflows/ci-dockerized-app-verify.yml@main
      with:
        organization: ${{ github.event.repository.owner.login }}
        repository: ${{ github.event.repository.name }}
        version: ${{ github.event.release.tag_name }}
      secrets:
        ecr-region: ${{ secrets.ecr-region }}
        ecr-iam-role: ${{ secrets.ecr-iam-role }}
        registry: ${{ secrets.registry }}
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}

```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| app | Application name. Used with monorepo pattern when there are several applications in the repo | string | N/A | true |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| version | Release version tag | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| registry | ECR Docker registry | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |



### Outputs

| Name | Description |
|------|-------------|
| image | Docker Image |
| tag | Docker image tag |



## Controller - Draft release

Create or update draft release

### Usage 

```yaml
  name: Draft release
  on:
    push:
      branches: [ main ]

  jobs:
    do:
      uses: cloudposse/github-actions-workflows/.github/workflows/ci-dockerized-app-verify.yml@main
      with:
        organization: ${{ github.event.repository.owner.login }}
        repository: ${{ github.event.repository.name }}
        version: ${{ github.event.release.tag_name }}
      secrets:
        ecr-region: ${{ secrets.ecr-region }}
        ecr-iam-role: ${{ secrets.ecr-iam-role }}
        registry: ${{ secrets.registry }}
        secret-outputs-passphrase: ${{ secrets.secret-outputs-passphrase }}
```





### Secrets

| Name | Description | Required |
|------|-------------|----------|
| github-private-actions-pat | Github PAT allow to create release | true |






## Controller - Reingtegrate hotfix branch

Create PR into `target_branch` to reintegrate hotfix from current branch 

### Usage 

```yaml
  name: Release
  on:
    release:
      types: [published]

  jobs:
    do:
      uses: cloudposse/github-action-workflows/.github/workflows/controller-hotfix-reintegrate.yml@main
      with:
        ref: ${{ github.ref }}
        target_branch: main
      secrets:
        github-private-actions-pat: ${{ secrets.github-private-actions-pat }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | N/A | true |
| target\_branch | Target branch to reintegrate hotfix | string | main | false |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| github-private-actions-pat | Github PAT allow to create a pull request | true |






## Controller - Create Release branch

Create `release/{version}` branch for the release 

### Usage 

```yaml
  name: Main branch
  on:
    release:
      types: [published]

  jobs:
    do:
      uses: cloudposse/github-action-workflows/.github/workflows/controller-hotfix-release-branch.yml@main
      with:
        version: ${{ github.event.release.tag_name }}
```  



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| version | Release version | string | N/A | true |








## Controller - Create hotfix release

Create next patch version release 

### Usage 

```yaml
  on:
    push:
      branches: [ 'release/**' ]

  jobs:
    do:
      uses: cloudposse/github-action-workflows/.github/workflows/controller-hotfix-release.yml@main
      with:
        ref: ${{ github.ref }}
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | N/A | true |





### Outputs

| Name | Description |
|------|-------------|
| version | Release version |



## Controller - Monorepo Controller

Mocked monorepo controller that outputs list of applications, lists of apps with and without changes.

### Usage 

```yaml
  name: Monorepo
  on:
    push:
      branches: [ main ]

  jobs:
    do:
      uses:  cloudposse/github-actions-workflows/.github/workflows/controller-monorepo.yml@main
      with:
        dir: ./apps
```  



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| dir | Directory with applications | string | N/A | true |





### Outputs

| Name | Description |
|------|-------------|
| apps | Applications |
| changes | Changed applications |
| no-changes | Unchanged applications |


<!-- markdownlint-restore -->
