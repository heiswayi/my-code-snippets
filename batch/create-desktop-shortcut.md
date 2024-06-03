Script to create a desktop shortcut

```batch
path/to/create-shortcut.bat "<URL>" "<SHORTCUT_NAME>"
```

`create-shortcut.bat`

```batch
@echo off
setlocal

REM Check if both arguments are provided
if "%1"=="" (
    echo Please provide the web URL.
    goto :eof
)

if "%2"=="" (
    echo Please provide the shortcut name.
    goto :eof
)

REM Set variables
set "url=%1"
set "shortcutName=%2"
set "shortcutPath=%cd%\%shortcutName%.lnk"

REM Create the shortcut
echo Set objShell = CreateObject("WScript.Shell") > "%temp%\CreateShortcut.vbs"
echo Set objShortcut = objShell.CreateShortcut("%shortcutPath%") >> "%temp%\CreateShortcut.vbs"
echo objShortcut.TargetPath = "%url%" >> "%temp%\CreateShortcut.vbs"
echo objShortcut.Save >> "%temp%\CreateShortcut.vbs"

cscript //nologo "%temp%\CreateShortcut.vbs"
del "%temp%\CreateShortcut.vbs"

echo Shortcut created: %shortcutPath%
goto :eof
```