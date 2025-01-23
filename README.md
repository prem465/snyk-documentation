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



------------------------------------------------------------------------------------


Detailed Talking Script for the Presentation
Slide 1: Agenda
"Good [morning/afternoon], everyone. Today, I’m excited to present an in-depth view of our Power BI deployment pipeline. The agenda for this presentation will ensure that we cover every aspect of the pipeline:

Introduction: Understanding why this pipeline was implemented and its significance.
Purpose of the Pipeline: The objectives and goals it fulfills.
Key Features: The technical highlights and capabilities of the pipeline.
Flowchart: A visual representation of how the pipeline functions step-by-step.
Key Functions in the Code: We’ll dive into the technical implementation to understand the mechanics behind the automation.
Advantages: Why this pipeline stands out and its value to the organization.
Conclusion: A summary tying everything together.
This session aims to provide a comprehensive understanding of the deployment pipeline, from its conceptual framework to its technical execution."

Slide 2: Purpose of the Deployment Pipeline
"The Power BI deployment pipeline was designed to address specific challenges we faced during manual deployments:

Automation: By automating the deployment of reports and semantic models, we eliminate repetitive manual steps, allowing the team to focus on value-added tasks.
Linkage: The pipeline ensures that every report is correctly linked to its semantic model. This is crucial because mislinked or missing connections between reports and models can lead to inaccuracies in reporting.
Flexibility: It handles all scenarios: adding, modifying, and deleting reports or semantic models seamlessly.
Robustness: The fallback mechanism guarantees that even when dependencies are missing, the pipeline can resolve them by searching locally or in other environments.
In essence, this pipeline not only simplifies the deployment process but also makes it more reliable and efficient."

Slide 3: Key Features of the Pipeline
"Let’s break down the key technical features that make this pipeline stand out:

Automated Detection of Changes: Using Git’s commit comparison logic, the pipeline can detect changes such as additions, modifications, and deletions. This ensures that only the necessary updates are deployed.
Scenario Handling:
Add/Modify Logic: When files are added or modified, the pipeline retrieves IDs for semantic models and links the reports appropriately.
Delete Logic: The pipeline uses a systematic multi-step approach to remove deleted files from UAT.
Fallback Mechanism: In cases where a report doesn’t have an associated semantic model, the pipeline searches:
Locally.
In the UAT workspace using exact, base name, and wildcard matches.
Secure Authentication: The pipeline leverages service principal credentials to ensure secure access to Azure, Power BI, and OneLake environments.
Comprehensive Logging: Every step of the process is logged to ensure transparency and traceability. Errors and successes are clearly documented for debugging and review.
Together, these features ensure that the pipeline is adaptable, secure, and reliable."

Slide 4: Flowchart of the Pipeline
"This flowchart provides a step-by-step view of the pipeline’s workflow:

Step 1: Authentication: The pipeline begins by authenticating with Fabric, Azure, and OneLake using service principal credentials. This step ensures secure access to all resources required during deployment.
Step 2: Detect Changes: Git is used to compare the current commit (HEAD) with the previous commit (HEAD~1). The pipeline identifies the changes in semantic models or reports.
Step 3: Scenario Handling:
If both a report and its semantic model are added or modified, the pipeline imports both, retrieves the semantic model ID, and links the report.
If only a report is modified, it triggers the fallback mechanism to find a semantic model.
If deletions are detected, it removes the files using Fabric API calls.
Step 4: Finalization: Once all changes are handled, the pipeline validates the deployment and refreshes the UAT workspace items.
This structured flow ensures that all scenarios—add, modify, and delete—are managed effectively."

Slide 5: Key Functions in the Pipeline Code
"Let’s take a closer look at the core functions in the pipeline’s code:

Git Operations:
By comparing HEAD~1 and HEAD, the pipeline identifies the exact changes between the previous and current commits.
This enables the pipeline to deploy only the modified or added files, making it efficient.
Import Logic:
Reports and semantic models are imported first.
Semantic models are retrieved, and reports are linked using their semantic model IDs.
Fallback Mechanism:
This mechanism ensures that reports without associated semantic models are resolved using a three-step process:
Exact match.
Base name match.
Wildcard search (optional, if enabled).
Deletion Logic:
Deleted files are handled using Fabric API calls to ensure the workspace remains clean and consistent.
Each of these functions works together to create a seamless deployment experience."

Slide 6: Advantages of the Deployment Pipeline
"Why is this pipeline so valuable? Let me highlight its advantages:

Time-Saving: Automation eliminates the manual effort required during deployments.
Consistency: Deployments are uniform across environments, reducing errors caused by manual inconsistencies.
Reliability: By ensuring reports are always linked to the correct semantic models, the pipeline guarantees accuracy in reporting.
Error Handling: The fallback mechanism and comprehensive logging ensure that even edge cases are handled gracefully.
Security: The use of service principals ensures that all operations comply with Azure, Power BI, and OneLake security standards.
Overall, this pipeline not only simplifies our work but also ensures the integrity and security of our deployments."

Slide 7: Conclusion
"In conclusion, the Power BI deployment pipeline has:

Streamlined the deployment process, making it faster and more efficient.
Enhanced reliability by ensuring reports are linked to semantic models accurately.
Minimized errors with robust error handling and fallback mechanisms.
Fostered collaboration across teams by providing a consistent and reliable deployment process.
This pipeline exemplifies how leveraging automation and secure practices can transform complex workflows into efficient and reliable processes. It sets a strong foundation for future enhancements and scalability within our organization."

Additional Notes for Emphasis:
Interactivity: Encourage questions after each major section to keep the audience engaged.
Clarity: Use examples where necessary, such as explaining how fallback mechanisms work or providing a scenario for Git commit comparisons.
Confidence: Speak with assurance about the technical depth of the pipeline, emphasizing its impact on the organization’s processes.
This detailed script will ensure that the audience leaves with a comprehensive understanding of the pipeline.


---------------------------------------------------------


Technical Script: Detailed Workflow of the Pipeline
Introduction:
This pipeline is designed to automate the deployment of Power BI reports and their corresponding semantic models to the UAT environment. It handles four key scenarios:

Addition of both a report and its semantic model.
Modification or addition of only a report.
Reverting to a previous version of a report.
Deleting a report from the UAT workspace.
Each scenario is carefully orchestrated to ensure consistency, accuracy, and traceability across deployments. Let’s walk through each of these scenarios in detail using an example of sample.report.

1. If Both Report and Semantic Model Are Added
Scenario:
sample.report and its corresponding sample.model are added in the Git repository. This might happen when a new report is created alongside a new semantic model.

Workflow:

Change Detection:

The pipeline compares the current commit (HEAD) with the previous commit (HEAD~1).
It identifies both sample.report and sample.model as added or modified.
Folder Mapping:

The pipeline maps the folders for both files to locate their paths.
This ensures that the semantic model and report are correctly categorized.
Importing Semantic Model:

The semantic model (sample.model) is imported first because reports depend on its ID for linking.
Once imported, the pipeline retrieves the model’s unique ID.
Attaching Report to the Model:

The report (sample.report) is imported and linked to the retrieved semantic model ID.
This step ensures that the report is functionally connected to its data model.
Deployment:

Both items are deployed to the UAT workspace, completing the deployment for this scenario.
2. If Only a Report Is Added or Modified
Scenario:
Only sample.report is updated or added in the Git repository. This often happens when a report requires changes but the semantic model remains untouched.

Workflow:

Change Detection:

The pipeline identifies that only sample.report is added or modified.
Semantic Model Retrieval:

The pipeline searches for the corresponding semantic model in a systematic fallback process:
Local Search: It checks if the model is available in the local repository.
UAT Search (Exact Match): If not found locally, it searches the UAT workspace for an exact match by name.
Base Name Match: As a last resort, it attempts to match the base name of the report with models in UAT.
Linking and Deployment:

Once the semantic model ID is retrieved, the report is linked to the model and deployed to the UAT workspace.
Error Handling:

If no corresponding semantic model is found, the pipeline halts and logs the issue for resolution.
3. Reverting to a Previous Version of a Report
Scenario:
A previous version of sample.report needs to be redeployed due to errors or rollbacks.

Workflow:

Reverting Changes in Git:

The pipeline retrieves the specific commit containing the desired version using git checkout or git revert.
Removing Current Version:

The current version of sample.report is removed from the UAT workspace to maintain consistency.
Deploying Reverted Version:

The reverted version of the report is redeployed and linked to the corresponding semantic model.
Verification:

The pipeline verifies that the reverted version is deployed correctly and logs the results.
4. Deleting a Report
Scenario:
sample.report is marked for deletion in the Git repository. This might happen when a report is no longer needed.

Workflow:

Deletion Detection:

The pipeline identifies the deletion of sample.report during the Git comparison.
Processed Folder Check:

The pipeline checks if the report’s folder is already marked as processed to avoid duplicate deletions.
Removing the Report:

It locates the report in the UAT workspace using its metadata and removes it using a Fabric API call.
Final Cleanup:

After deletion, the pipeline validates the workspace to ensure the report is no longer present.
Key Technical Highlights of the Pipeline:
Fallback Mechanism:
Ensures that reports can be linked to their semantic models even if the models are not immediately available locally. It systematically checks local folders and the UAT workspace using exact and base name matches.

Error Handling and Logging:
Comprehensive logging ensures that every action is traceable, and errors are promptly reported for resolution.

Automation:
The entire process is automated, reducing manual intervention and minimizing errors during deployment, reverts, or deletions.

Integration with Git:
The use of Git ensures that all changes are version-controlled, and the pipeline dynamically adapts to detected changes.

Example Scenario Summary
Case 1: Adding Both Report and Model
sample.report and sample.model are imported sequentially.
The report is linked to the model and deployed to UAT.
Case 2: Adding/Modifying Only the Report
sample.report is added/modified.
The pipeline finds the corresponding model using fallback mechanisms and links the report.
Case 3: Reverting
The current version of sample.report is removed from UAT.
The reverted version is deployed and linked.
Case 4: Deleting
sample.report is detected as deleted.
The pipeline removes the report from the UAT workspace.
This detailed explanation covers the complete workflow of the pipeline for all possible scenarios, ensuring a clear and comprehensive understanding for all stakeholders.

-------------------------------------------------------------------------

Title Slide
Good [morning/afternoon], everyone. Today, I’ll walk you through the Power BI Deployment Pipeline, an automated framework designed to simplify and streamline the deployment of Power BI reports and semantic models. By the end of this presentation, you’ll have a clear understanding of its purpose, functionality, and technical details.

Agenda
We’ll start by discussing the purpose of the pipeline, then move on to its key features and a detailed flowchart of how it operates. After that, I’ll explain the key functions in the pipeline code, followed by its advantages. Finally, we’ll wrap up with a conclusion and take any questions you might have.

Purpose of the Deployment Pipeline
The deployment pipeline was created to address the manual and error-prone processes involved in deploying Power BI assets. It automates the deployment of Power BI reports and their linked semantic models to environments like UAT. This ensures all files are properly linked and handled, eliminating the risk of manual errors. The pipeline seamlessly supports scenarios where files are added, modified, or deleted, and it includes a fallback mechanism to handle edge cases where dependencies like semantic models may be missing. Ultimately, it enables faster and more reliable deployments.

Key Features of the Pipeline
The pipeline includes several features that ensure it operates efficiently and securely:

Automated detection of changes: It uses Git to compare base and head commits to identify files that have been added, modified, or deleted.
Add/modify logic: When reports or models are added, it links the reports to their corresponding semantic models, ensuring a functional deployment.
Delete logic: It identifies deleted files and removes them from the UAT workspace via REST API calls.
Fallback mechanism: For scenarios where a report lacks its associated model, the pipeline searches for the model locally or in the UAT workspace and links it automatically.
Secure authentication: It uses service principal credentials to authenticate with Azure, Power BI, and OneLake securely.
Comprehensive error handling: Every step is logged for traceability and debugging.
Flowchart of the Pipeline
Let’s look at how the pipeline functions through its flowchart:

Authentication: The pipeline starts by authenticating with Azure, Fabric, and OneLake using service principal credentials.
Change Detection: It identifies files added, modified, or deleted by comparing base and head commits in the Git repository.
Processing Changes:
If both the report and model are added, they are imported and linked.
If only the report is added, the fallback mechanism searches for the model locally or in UAT and links it.
If files are marked for deletion, the pipeline removes them from UAT using REST API calls.
Finalization: Once all changes are processed, the pipeline validates and refreshes the UAT workspace.
This logical structure ensures that every scenario—whether it’s adding, modifying, or deleting files—is handled seamlessly.

lide 1: Key Functions in the Pipeline Code (Part 1)
Talking Script:

The pipeline starts with Authentication, which establishes secure connections to Azure, Power BI, and OneLake through an Azure Service Principal. This is achieved by passing credentials like client ID, client secret, and tenant ID to authenticate the pipeline securely. Without this, no operations like data import or deletion can proceed. It ensures compliance with enterprise-grade security standards.

Next, we have Git Operations, a crucial step where the pipeline detects changes in the repository by comparing the base and head commits. This operation identifies which files have been added, modified, or deleted—specifically focusing on .pbix and .pbism files. This selective targeting minimizes unnecessary processing and ensures accurate handling of deployment changes.

Finally, Import Logic deals with the addition or modification of semantic models and reports. It first imports the semantic model, retrieves its ID, and associates it with the corresponding report. This linking is crucial to maintaining the integrity of dependencies between models and reports, ensuring that deployed reports always function correctly in the target environment.

Slide 2: Key Functions in the Pipeline Code (Part 2)
Talking Script:

The Fallback Mechanism is a robust feature designed to handle scenarios where a report is added or modified without its corresponding semantic model. The pipeline first checks for the model locally; if it finds it, it imports it and retrieves its ID. If not, it searches in the UAT workspace by matching file names. This ensures that all reports are linked to appropriate models, even if the model is missing from the current commit.

Deletion Logic is another critical function, ensuring that deleted reports and models are cleaned up from the environment. This logic follows a systematic two-step approach: first, it tries an exact match for the file name. If no exact match is found, it falls back to a base name match, removing associated files cleanly while maintaining data integrity.

Lastly, Error Handling is incorporated throughout the pipeline, providing detailed logs for every action. These logs not only offer transparency but also enable quick troubleshooting during failures, making the pipeline more resilient and efficient.
Advantages
The pipeline brings several advantages:

It automates tedious processes, saving time and reducing manual effort.
It ensures consistency in deployments across environments, minimizing errors.
It provides robust handling of edge cases, such as missing models, through the fallback mechanism.
It is secure and compliant with Azure, Power BI, and OneLake standards, ensuring data integrity and safety.
The comprehensive error handling and logging improve visibility and traceability.
Conclusion
In summary, the Power BI Deployment Pipeline is an essential tool for our operations. It automates the deployment process, ensures reliability, and provides robust mechanisms to handle edge cases. Whether adding, modifying, or deleting files, the pipeline guarantees accuracy and consistency across environments. By eliminating manual errors and streamlining workflows, it saves time and enhances collaboration across teams.

Thank you for your time. I’m happy to address any questions or dive deeper into any specific parts of the pipeline.

