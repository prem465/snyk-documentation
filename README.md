
trigger:
  branches:
    - develop

parameters:
  artifactName: 'pythonApp'

jobs:
- job: Build
  displayName: 'Package Python Code'
  pool:
    vmImage: 'ubuntu-latest'

  steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '3.x'

  - script: |
      mkdir -p build
      pip install -r requirements.txt -t build/
      cp *.py requirements.txt build/
      cp -r app/ build/
      zip -r app.zip build
    displayName: 'Package Python App'

  - publish: app.zip
    artifact: ${{ parameters.artifactName }}