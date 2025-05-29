- script: |
    echo "Pipeline.Workspace: $(Pipeline.Workspace)"
    echo "Checking artifact directory..."
    ls -al $(Pipeline.Workspace)
    echo "Checking inside pythonApp..."
    ls -al $(Pipeline.Workspace)/pythonApp
  displayName: "Debug: List pipeline workspace and artifact contents"
