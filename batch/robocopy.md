Script to copy files from source folder to destination folder with status. Force overwrite if file already exists.

```batch
@ECHO OFF
color 07

set source="C:\DND_SW\TeamBuild2017\TFS2015_vNext"
set dest="C:\git\TFSBuild2017\tfsbuild2017"

ROBOCOPY %source% %dest% *.* /S

IF %ERRORLEVEL% LSS 8 goto end

:failed
echo.
color 47
ECHO Copy or update files... ERROR
REM exit /b 1
pause

:end
echo.
color 27
ECHO Copy or update files... OK
REM exit /b 0
pause
```