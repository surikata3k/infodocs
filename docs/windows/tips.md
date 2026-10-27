# Windows tips

## Check disk
```powershell
winsat disk -drive F
```

## Edit network
```powershell
ncpa.cpl
```

## View folder of current wallpaper
```text
%AppData%\Microsoft\Windows\Themes\CachedFiles
```

## Right menu with all options
```powershell
reg.exe add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
```

## Install skipping Microsoft user
Press ++shift+f10++ and type one of these:

```text
start ms-cxh:localonly
```
```text
OOBE\BYPASSNRO
```

## Mouse: disable wake up from sleep
1. Find the exact name of your mouse by running this command:

    ```bat
    powercfg -devicequery wake_armed
    ```

2. Disable the wake capability (replace `DeviceName` with the name you copied):

    ```bat
    powercfg -devicedisablewake "DeviceName"
    ```

!!! example
    `powercfg -devicedisablewake "HID-compliant mouse"`
