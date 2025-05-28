Everything else is already known and can be locked into your Dev variable group:

Key	Example value
BatchAccount	lvbatchdev
Region	eastus2
StorageAccount	lvbatchdev
ApplicationId	ccdc-batch-jobs
AppVersionFormat	1.$(Build.BuildId)
CmdLineWin	cmd /c %AZ_BATCH_APP_PACKAGE_ccdc-batch-jobs%\\startup.ps1 && python %AZ_BATCH_APP_PACKAGE_ccdc-batch-jobs%\\main.py
