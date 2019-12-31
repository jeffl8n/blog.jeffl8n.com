---
title: "Synchronize Subversion to CA AllFusion Harvest"
date: 2011-03-28T12:05:32-05:00
---

For a long time I’ve been searching how to synchronize code from a Subversion code repository to a CA AllFusion Harvest code repository.  Finally, I decided to just write a script to do it myself.  The script takes one argument, which is the Harvest package name.  There are other constants you’ll need to set within the script, such as the Subversion repository and Harvest username and password.  You can easily change those to be a script parameter, too.

We’re using this script right now with our [Jenkins](http://jenkins-ci.org/) (formerly [Hudson](http://hudson-ci.org/)) CI server so a new Harvest package is created (named by the %JOB_NAME% variable set by Jenkins) with each Production build is run.

```
@echo off
 
REM ###########################################################################
REM #
REM # Script file to move changes from Subversion to Harvest
REM #
REM ###########################################################################
 
if "%1" == "" goto usage
 
setlocal
 
set PROJECT_STAGE=-b "" -en "" -st ""
set VIEW=-vp ""
set CREDENTIALS=-usr "" -pw ""
set SUBVERSION_REPO=svn:////trunk/
 
REM Clean up any build artifacts if present
call ant clean
 
REM Create Harvest package
hcp %1 %PROJECT_STAGE% %CREDENTIALS%
 
REM Delete the checked-out Subversion code
REM Note: You will need to remove everything (except this file of course), so more rmdir or del statements may be required below
rmdir /S /Q project
 
REM Check out the files in Harvest to modify
hco * %PROJECT_STAGE% %CREDENTIALS% %VIEW% -p "%1" -pn "Check Out Files for Update" -up -r -s * -op pc -cp %CD%
 
REM Delete the checked-out Harvest code
REM Note: You will need to remove everything (except this file of course), so more rmdir or del statements may be required below
rmdir /S /Q project
 
REM Replace with the latest code from Subversion repository
svn co %SUBVERSION_REPO% .
 
REM Delete the .svn directories
for /f "tokens=* delims=" %%i in ('dir /s /b /a:d *svn') do (
  rd /s /q "%%i"
)
 
REM What are the updates for Harvest?  Check them into Harvest
hci * %PROJECT_STAGE% %CREDENTIALS% %VIEW% -p "%1" -pn "Check In Modified Files" -s * -ur -de "Added in Subversion" -if ne -op pc -cp %CD%
 
REM Remove the log files from Harvest
REM Note: You may need to change or remove this statement depending on whether you want the Harvest logs checked in
hdv *.log %PROJECT_STAGE% %CREDENTIALS% %VIEW% -pn "Delete Versions"
 
REM What removals from Harvest do we need to process? (What files were deleted in Subversion?)
hsv %PROJECT_STAGE% %CREDENTIALS% %VIEW% -p "%1" -it r -s "*"
 
REM This will not work if the file path has spaces in it.  You can use %%j %%k ... in the -vp switch for one space in your project name (For example, if you have 2 spaces, it should be -vp %%j %%k %%l)
for /f "tokens=1-5 skip=3" %%i in (hsv.log) do (
    if not "%%i"=="hsv" (
        hci "%%i" %PROJECT_STAGE% %CREDENTIALS% -vp "%%j" -p "%1" -pn "Check In Modified Files" -ro -cp %CD%
        hri "%%i" %PROJECT_STAGE% %CREDENTIALS% -vp "%%j" -p "%1"
    )
)
 
REM remove read-only attribute from all files
attrib -r -h * /s
 
REM delete all harvest.sig files
del /f /s /q harvest.sig
 
endlocal
 
goto end
 
:usage
 
echo USAGE:
echo -----------------------------------------------------------------------------
echo "svn2harvest {package_name}"
echo -----------------------------------------------------------------------------
 
:end
```

As you can see, I’m using the command line utilities for both Subversion and Harvest to make this happen so you’ll need both installed.

Let me know if you see anything that could be improved or have any other questions about it.