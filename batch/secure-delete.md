```batch
@echo off
setlocal

echo This script will permanently delete all files and folders in the current directory.
echo Ensure that you have a backup of important data before running this script.
echo.
pause

echo Deleting files and folders...
for /f "delims=" %%i in ('dir /b /a-d') do (
    echo Deleting "%%i"
    del "%%i" /q
)

echo Deleting folders...
for /f "delims=" %%i in ('dir /b /ad') do (
    echo Deleting "%%i"
    rd "%%i" /s /q
)

echo All files and folders in the current directory have been securely deleted.
pause
```