Resolved Power BI Pipeline Permission Issues
As a DevOps Engineer, I investigated and identified that the service principal lacked necessary permissions, raised a request with the BAM team, and resolved the issue to ensure smooth pipeline execution.

Configured OneLake Endpoint for Dev and UAT Environments
As a DevOps Engineer, I successfully configured the OneLake endpoint for both development and UAT environments, enabling comprehensive pipeline testing and validation.

Updated Environment Variables for Successful File Processing
As a DevOps Engineer, I updated the environment variables in the pipeline, allowing the detection of changed files and ensuring successful file processing with accurate output generation.

Established File Transfer Mechanism Between Dev and UAT
As a DevOps Engineer, I collaborated with teams to test and implement efficient file transfer methods between development and UAT environments, streamlining the data transition process.

Collaborated with BAM Team for Service Principal Permissions
As a DevOps Engineer, I worked closely with the BAM team to approve and validate the necessary permissions for the service principal, ensuring error-free pipeline execution.

Integrated Snyk at the Repository Level for Vulnerability Detection
As a DevOps Engineer, I configured Snyk at the repository level, enabling early detection of vulnerabilities and promoting secure development practices within the codebase.

Implemented Snyk in the CI/CD Pipeline
As a DevOps Engineer, I successfully integrated Snyk into the CI/CD pipeline, scanned for vulnerabilities, fixed errors, and reran the pipeline to ensure compliance with security standards.

Remediated Vulnerabilities Detected by Snyk
As a DevOps Engineer, I identified and resolved vulnerabilities detected by Snyk in the CI/CD pipeline, leading to a secure and successful pipeline re-execution.

Conducted KT Session on Snyk Integration for Pipelines
As a DevOps Engineer, I attended a knowledge transfer session with Henry, gaining insights on implementing Snyk in pipelines and IDEs, and subsequently applied this knowledge effectively.

Configured Snyk in IDEs (e.g., VS Code)
As a DevOps Engineer, I configured Snyk in IDEs like VS Code, enabling local development vulnerability scanning and ensuring developers can proactively address security issues.









............................


trigger:
  - main  # Trigger on the main branch

pool:
  vmImage: 'windows-latest'

stages:
  # Stage 1: Continuous Integration
  - stage: CI
    displayName: "Continuous Integration"
    jobs:
      - job: ValidateAndPackage
        displayName: "Validate and Package Artifacts"
        steps:
          # Step 1: Validate Files
          - task: PowerShell@2
            displayName: "Validate Power BI Files"
            inputs:
              targetType: 'inline'
              script: |
                Write-Host "Validating Power BI resources..."
                if (-Not (Test-Path "$(System.DefaultWorkingDirectory)/static resources/definition.pbir")) {
                    Write-Error "Definition.pbir file is missing in static resources folder."
                } else {
                    Write-Host "Definition.pbir found."
                }
                if (-Not (Test-Path "$(System.DefaultWorkingDirectory)/custodial loads monitor.SemanticModel/definition.pbism")) {
                    Write-Error "Definition.pbism file is missing in SemanticModel folder."
                } else {
                    Write-Host "Definition.pbism found."

          # Step 2: Publish Artifacts
          - task: PublishBuildArtifacts@1
            displayName: "Publish Build Artifacts"
            inputs:
              pathToPublish: '$(System.DefaultWorkingDirectory)'
              artifactName: 'CustodialLoadArtifacts'

  # Stage 2: Continuous Deployment
  - stage: CD
    displayName: "Continuous Deployment"
    dependsOn: CI  # This stage depends on the successful completion of CI
    jobs:
      - job: Deploy
        displayName: "Deploy to Power BI Workspace"
        pool:
          vmImage: 'windows-latest'
        steps:
          # Step 1: Install Power BI Module
          - task: PowerShell@2
            displayName: "Install Power BI Module"
            inputs:
              targetType: 'inline'
              script: |
                Install-Module -Name MicrosoftPowerBIMgmt -Force -AllowClobber

          # Step 2: Connect to Power BI Service Account
          - task: PowerShell@2
            displayName: "Connect to Power BI Service Account"
            inputs:
              targetType: 'inline'
              script: |
                $credential = New-Object System.Management.Automation.PSCredential ($env:SP_APP_ID, (ConvertTo-SecureString $env:SP_SECRET -AsPlainText -Force))
                Connect-PowerBIServiceAccount -ServicePrincipal -Credential $credential -TenantId $env:TENANT_ID

          # Step 3: Deploy Reports
          - task: PowerShell@2
            displayName: "Deploy Reports to Workspace"
            inputs:
              targetType: 'inline'
              script: |
                $reportPath = "$(Pipeline.Workspace)/static resources/definition.pbir"
                Import-PowerBIReport -Path $reportPath -WorkspaceId $env:WORKSPACE_ID

          # Step 4: Refresh Datasets
          - task: PowerShell@2
            displayName: "Refresh Power BI Dataset"
            inputs:
              targetType: 'inline'
              script: |
                $datasetId = "YOUR_DATASET_ID" # Replace with your actual dataset ID
                Invoke-PowerBIRestMethod -Url "datasets/$datasetId/refreshes" -Method POST

