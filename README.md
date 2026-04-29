CI/CD pipeline templates for Azure deployments with approval gates

# Main Azure DevOps Pipeline
# Orchestrates CI + CD using reusable templates


trigger:
  branches:
    include:
      - main
      - develop
  paths:
    exclude:
      - '**/*.md'
      - 'docs/*'

pr:
  branches:
    include:
      - main

variables:
  - group: azure-credentials
  - name: containerRegistry
    value: 'myregistry.azurecr.io'
  - name: imageTag
    value: $(Build.BuildId)

# CI Stage ──────────────────────────────────────────────
extends:
  template: templates/ci-template.yml
  parameters:
    nodeVersion: '18.x'
    runTests: true
    runSecurityScan: true

# CD: Staging ───────────────────────────────────────────
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/develop') }}:
  - template: templates/cd-template.yml
    parameters:
      environment: staging
      serviceConnection: $(AZURE_SERVICE_CONNECTION_STAGING)
      appServiceName: 'myapp-staging-app'
      resourceGroup: 'myapp-staging-rg'
      containerRegistry: $(containerRegistry)
      imageTag: $(imageTag)

# CD: Production ────────────────────────────────────────
- ${{ if eq(variables['Build.SourceBranch'], 'refs/heads/main') }}:
  - template: templates/cd-template.yml
    parameters:
      environment: production
      serviceConnection: $(AZURE_SERVICE_CONNECTION_PROD)
      appServiceName: 'myapp-prod-app'
      resourceGroup: 'myapp-prod-rg'
      containerRegistry: $(containerRegistry)
      imageTag: $(imageTag)
      runSmokeTests: true
