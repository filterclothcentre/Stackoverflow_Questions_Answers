## Question 1 ##

### Copy File's Parent directory path from context menu ###

I am learning batch scripting since it's very useful for setting some quick custom context menu options of Windows user's choice to get file and it's parent directory's path.

Now I know following command to get the passed argument as file path and copy it to clipboard:

```bat
cmd /c (echo.|set /p=""%1"") | clip
```

But then why isn't following syntax working as expected when set in Context menu ?

```cmd
cmd /c (echo.|set /p=""%~dp1"") | clip
```

Is it a variable expansion issue ? Please guide as to why it isn't working and how to resolve it so that it expands properly.

Sample of registry entry wherein I am permanently setting the command to be run, with switches, variable expansions etc.

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\*\shell\Copy File's Parent Path]
@="Copy File's Parent Path"

[HKEY_CURRENT_USER\Software\Classes\*\shell\Copy File's Parent Path\Command]
@="cmd.exe /c (echo.|set /p=\"\"%~dp1\"\") | clip"
```

## Answers ##

### Accepted Answer ###

Variables like %1, %*, %V in registry, are placeholders which will be resolve and replaced by Windows Shell component (Win32 Shell). They just happened to be similar to the variables that is used by Windows Command Processor(CMD.EXE) at Command Prompt or in batch files, But they are in no way related to each other.

For instance with CMD.EXE %1 can only be used within batch files it has no meaning at Command Prompt or when passed as a command by /C or /K switch. In your registry sample %1 will be resolved by Windows Shell not by CMD.EXE so you can not use things like %~dp0 or %~dp1 in registry, They have no meaning to Windows Shell.

So you have to take %1 by CMD.EXE (which have been automatically replaced by its actual value) and do processing with it in a FOR loop to obtain the directory path of the file.

This is the actual command that will extract path information from file name:

```cmd
cmd.exe /e:on /d /s /c "for %%a in ("%1") do @(set /p "=%%~dpa")<nul | clip"
```

It can be entered as is, in Registry Editor to the (default) value of the appropriate Key e.g: HKEY_CURRENT_USER\Software\Classes\*\shell\Copy Path to clipboard\Command for current user or HKEY_CLASSES_ROOT\*\shell\Copy Path to clipboard\Command for all users.

And the string for registry script:

```cmd
@="cmd.exe /e:on /d /s /c \"for %%a in (\"%1\") do @(set /p \"=%%~dpa\")<nul | clip\""
```

A sample .REG script will look like this:

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\*\shell\Copy Path to clipboard\Command]
@="cmd.exe /e:on /d /s /c \"for %%a in (\"%1\") do @(set /p \"=%%~dpa\")<nul | clip\""
```

When you want to pass a string that contains percents(%) you have to escape the percents by doubling them, pretty much the same as you do in batch files.

### Answer 2 ###

%W, which generally holds the working directory, when you right click on a file, would be the files' parent.

You could therefore try this:

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\*\shell\HoldParent]
@="Parent to Clipboard"

[HKEY_CURRENT_USER\Software\Classes\*\shell\HoldParent\command]
@="Cmd /C \"Echo %W|Clip\""
```

You could extend this a little for potential unicode/character issues, but in general it should perfom as required without.

### Answer 3 ###

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\*\shell\Copy Files Path\command]
@="cmd /c (echo. | set /p =\"\"%1\"\") | clip"

[HKEY_CURRENT_USER\Software\Classes\*\shell\Copy Files Parent Path\command]
@="cmd /c (echo. | set /p =\"\"%w\"\") | clip"
```

Replace %1 with %w. 2 registry entries to demonstrate %1 as Copy Files Path and %w as Copy Files Parent Path.

Reference from [Extending Shortcut Menus](https://web.archive.org/web/20111002101214/http://msdn.microsoft.com/en-us/library/windows/desktop/cc144101(v=vs.85).aspx target="_blank") at the bottom of the page.

```
%0 or %1  the first file parameter. For example “C:\Users\Eric\Desktop\New Text Document.txt”. Generally this should be in quotes and the applications command line parsing should accept quotes to disambiguate files with spaces in the name and different command line parameters (this is a security best practice and I believe mentioned in MSDN).
%N  (where N is 2 - 9), replace with the nth parameter
%*  replace with all parameters
%~  replace with all parameters starting with and following the second parameter
%d  desktop absolute parsing name of the first parameter (for items that don’t have file system paths)
%h  hotkey value
%i  IDList stored in a shared memory handle is passed here.
%l  long file name form of the first parameter. Note win32 applications will be passed the long file name, win16 applications get the short file name. Specifying %L is preferred as it avoids the need to probe for the application type.
%s  show command
%v  for verbs that are none implies all, if there is no parameter passed this is the working directory
%w  the working directory
```
[Microsoft Reference Link](http://msdn.microsoft.com/en-us/library/windows/desktop/cc144101(v=vs.85).aspx target="_blank")


## Question 2 ##

### Custom Context Menu option for duplicating selected file not working as expected ###

I am trying to create a custom Context Menu option for duplicating the selected file and appending date and time string to the copied file's name.

Below is the command I have set in registry, in the HKCU > Softwares > Classes > * > Shell > Duplicate This File > Command:

```cmd
cmd /s /d /c @echo off & setlocal EnableExtensions EnableDelayedExpansion & set TIME=%TIME: =0% & set DateTimeFn=%DATE:~10,4%-%DATE:~4,2%-%DATE:~7,2%_!TIME:~0,2!-!TIME:~3,2!-!TIME:~6,2! & copy /y %1 %1_!DateTimeFn! & pause > nul
```

But somehow the enabledelayedexpansion doesn't work correcty, cause when I try to use this on file test.js it duplicates it as test.js_!DateTimeFn!.

Also it doesn't work well with spaces in filenames. Can anyone guide and help fix this ?

I would prefer one-liner over creating separate batch script as far as it's possible.

Sample of registry file in which I am trying to run the command with switches and variable expansions:

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\*\shell\Duplicate This File II]

[HKEY_CURRENT_USER\Software\Classes\*\shell\Duplicate This File II\command]
@="cmd /v:on /c @echo off & set TIME=%TIME: =0% & set DateTimeFn=%DATE:~10,4%-%DATE:~4,2%-%DATE:~7,2%_!TIME:~0,2!-!TIME:~3,2!-!TIME:~6,2! & copy /y %1 %1_!DateTimeFn! & pause > nul"
```

## Answers ##

### Accepted Answer ###

You will need to use the argument in a FOR command and then you can use the command modifiers to get the file name separated from the extension.

```cmd
cmd /Q /V:ON /E:ON /C "set TIME=%%TIME: =0%% & set DateTimeFn=%%DATE:~10,4%%-%%DATE:~4,2%%-%%DATE:~7,2%%_!TIME:~0,2!-!TIME:~3,2!-!TIME:~6,2! &FOR %%G IN ("%1") do copy "%1" "%%~nG_!DateTimeFn!%%~xG" & pause>nul"
```

Here is the actual registry export.

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\*\shell\Time_Stamp_FileName]

[HKEY_CURRENT_USER\Software\Classes\*\shell\Time_Stamp_FileName\command]
@="cmd /Q /V:ON /E:ON /C \"set TIME=%%TIME: =0%% & set DateTimeFn=%%DATE:~10,4%%-%%DATE:~4,2%%-%%DATE:~7,2%%_!TIME:~0,2!-!TIME:~3,2!-!TIME:~6,2! &FOR %%G IN (\"%1\") do copy \"%1\" \"%%~nG_!DateTimeFn!%%~xG\" & pause>nul\""
```
