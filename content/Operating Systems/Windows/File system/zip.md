unzip files in windows:
```powershell
Expand-Archive -Path DSI.zip -DestinationPath DSI -Force
Expand-Archive DSI.zip  DSI -Force # Fast
powershell -command "Expand-Archive -Path DSI.zip -DestinationPath DSI -Force" # cmd

# old systems
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory("DSI.zip", "DSI")
```