trigger: none          # CD runs only when you manually create a release

resources:
  pipelines:
  - pipeline: build
    source: ccdc-batch-jobs            # <-- name (not ID) of your CI pipeline
    trigger: true                      # every successful CI run starts CD

variables:
- group: Batch-CD-Config               # variable group created in step 1

stages:
- stage: Deploy
  displayName: 'Deploy to Azure Batch'
  jobs:
  - job: BatchDeploy
    displayName: 'Upload, register, run task'
    pool:
      vmImage: 'ubuntu-latest'

    steps:
    # 1. download the artifact that CI produced
    - download: build
      artifact: pythonApp

    # 2. run all Batch + Storage CLI in one AzureCLI@2 task
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'MyServiceConnection'
        scriptType: bash
        scriptLocation: inlineScript
        inlineScript: |
          set -e

          # ---------- variables ----------
          ZIP_PATH="$(System.DefaultWorkingDirectory)/pythonApp/app.zip"
          VERSION="$(Build.BuildId)"          # use CI build‑ID as package version
          POOL_ID="pool-$(date +%s)"
          JOB_ID="job-$(date +%s)"

          echo "Uploading $ZIP_PATH to blob storage ..."
          az storage blob upload \
              --account-name  $(storageAccount) \
              --container-name $(container) \
              --name ${VERSION}.zip \
              --file "$ZIP_PATH" \
              --overwrite

          echo "Creating / updating Batch application package ..."
          az batch application create \
              --application-id $(appId) \
              --account-name   $(batchAccount) || true

          az batch application package create \
              --application-id $(appId) \
              --version        ${VERSION} \
              --format         zip \
              --file           "$ZIP_PATH" \
              --account-name   $(batchAccount)

          az batch application set \
              --application-id $(appId) \
              --default-version ${VERSION} \
              --account-name    $(batchAccount)

          echo "Creating pool $POOL_ID ..."
          az batch pool create \
              --id                   $POOL_ID \
              --vm-size              $(vmSize) \
              --target-dedicated-nodes 1 \
              --image                $(nodeImage) \
              --node-agent-sku-id    "$(nodeSku)" \
              --account-name         $(batchAccount)

          echo "Creating job $JOB_ID ..."
          az batch job create \
              --id          $JOB_ID \
              --pool-id     $POOL_ID \
              --account-name $(batchAccount)

          echo "Submitting task ..."
          az batch task create \
              --job-id      $JOB_ID \
              --task-id     runtask \
              --application-package-references $(appId)#${VERSION} \
              --command-line "/bin/bash -c 'python3 main.py'" \
              --account-name $(batchAccount)

          echo "🎉 Deployment finished — task submitted."
