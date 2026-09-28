---
name: windows-cmd-scripts
description: Structure, conventions, and best practices for writing, logging, and testing robust Windows .cmd/.bat scripts.
---

# Windows CMD/BAT Scripting Guidelines & Patterns

This document establishes the patterns, structure, and best practices for writing high-quality, concise, and robust Windows CMD/BAT scripts. These rules optimize scripts for readability and reliability under common execution edge cases.

---

## 1. Script Structure & Layout

Every script must follow a predictable, logical layout split into clearly demarcated sections:

1. **Self-Documenting Usage Headers**: Block comments at the top of the file prefixed with `::|` detailing description, requirements, defaults, and option syntax.
2. **Default Settings**: Declaration of default flags, paths, and patterns.
3. **Option Parsing**: A `:parse` loop reading argument inputs with validation.
4. **Initialization & Prerequisites**: Local path updates and required tool verification.
5. **Execution Entry**: Fallback for empty arguments, followed by the main argument loop.
6. **Subroutines**: Clean, reusable helper routines separated by `exit /b`.

### Structure Skeleton

```cmd
::|Usage: {{scriptName}} [options] {project-path-glob ...}
::|
::|Generates water/sewer concession tender documentation for specified projects.
::|
::|Required tools in system PATH:
::|  yq    - Install from https://github.com/mikefarah/yq releases
::|  tera  - Install from https://github.com/chevdor/tera-cli releases
::|
::|  Install tools with `{{scriptName}} --install` or manually:
::|  > winget install MikeFarah.yq
::|
::|Options:
::|  -d, --defaults <file>  Input defaults file [{{scriptName}}.defaults.yaml]
::|  -f, --force         Run even if output exists
::|  -c, --continue      Continue if any project or template generation fails
::|  -I, --install       Install required tools and quit
::|  -h, --help          Show this help message
@echo off
setlocal

:: 1. Default Settings
set self=%~n0
set debug=
set force=
set continue=
set "defaultsPath=%~dpn0.defaults.yaml"

:: 2. Option Parsing
:parse
if "%~1"=="-D" echo on && set "debug=true" && shift && goto :parse
if "%~1"=="--debug" echo on && set "debug=true" && shift && goto :parse
if "%~1"=="-h" call :usage "%~f0" && exit /b 0
if "%~1"=="--help" call :usage "%~f0" && exit /b 0
if "%~1"=="-I" call :installTools && exit /b || exit /b
if "%~1"=="--install" call :installTools && exit /b || exit /b
if "%~1"=="-f" set "force=true" && shift && goto :parse
if "%~1"=="--force" set "force=true" && shift && goto :parse
if "%~1"=="-c" set "continue=true" && shift && goto :parse
if "%~1"=="--continue" set "continue=true" && shift && goto :parse

if "%~1"=="--" (set "checkOption=" && shift) else set "checkOption=%~1"
if "%checkOption:~0,1%"=="-" call :error "invalid option: %checkOption%" || exit /b 2

:: 3. Initialization
call :useToolsPaths
call :requireTools yq.exe tera.exe || exit /b

:: 4. Execution Entry
if "%~1"=="" call :processProject . && exit /b || exit /b

set rc=0
:projectLoop
for /d %%d in (%1) do call :continuedLoop :processProject "%%~d" || exit /b
if not "%~1"=="" shift && goto :projectLoop
exit /b %rc%

:: 5. Subroutines
:processProject
:: ... logic ...
exit /b 0
```

---

## 2. Self-Documenting Usage Headers

Document all script dependencies, environment defaults, and options directly within the headers using `::|`. 

### Key Requirements to Document
* **Requirements list**: Explicitly list third-party binaries required in the system `PATH` (e.g. `yq`, `tera`).
* **Settings & Option Defaults**: List default values in brackets (e.g., `[{{scriptName}}.defaults.yaml]`, `[Podklady\.zadani.yaml]`).
* **Tool Installation Instructions**: Provide `winget` and `mise` commands in the usage block for quick copy-paste to empower users to install missing dependencies manually.

### Parsing and Rendering Usage
Render the help comments dynamically without printing diacritics issues or syntax-errors by processing lines starting with `::|`:

```cmd
:usage
setlocal enabledelayedexpansion
for /f "usebackq delims=| tokens=1*" %%l in ("%~1") do if "%%~l"=="::" (
  set "line=%%~m" 
  if defined line set "line=!line:{{scriptName}}=%~n1!"
  echo.!line!
)
exit /b 0
```

---

## 3. Variable Naming Conventions

Always use distinct variable suffixes to indicate structural types:
* `xxDir`: Represents a directory path (e.g., `templateDir`, `projectOutputDir`, `outputDir`).
* `xxFile`: Represents a file name/spec (e.g., `templateFile`, `projectInputFile`, `defaultsFile`).
* `xxPath`: Represents the final resolved file path (e.g., `templatePath`, `outputPath`, `defaultsPath`).

Using this naming standard maintains semantic clarity regarding whether a variable holds a folder path, a relative filename, or a fully-qualified file path.

---

## 4. Option-Driven Configuration

Avoid hardcoded global values. Instead, prioritize passing settings via command-line options so the script is versatile:
* Map directory structure parameters to flags (e.g., `-t/--templates`, `-i/--input`, `-o/--output`, `-r/--ref`).
* Standardize on standard options:
  * `--debug` (`-D` or `--debug`): Enables command echo tracing.
  * `--force` (`-f` or `--force`): Force overwrites existing files/outputs.
  * `--continue` (`-c` or `--continue`): Skip failure exit and process subsequent projects/files.

---

## 5. Control Flow & Loop Management

### Subroutine-Free Option Prefix Validation
Check for flag prefixes without subroutines or delayed expansion by evaluating the string character on the *next line* using batch string slicing:
```cmd
if "%~1"=="--" (set "checkOption=" && shift) else set "checkOption=%~1"
if "%checkOption:~0,1%"=="-" call :error "invalid option: %checkOption%" || exit /b 2
```

### Argument Looping & Shift Processing
* Loop over remaining unshifted arguments using `%1` (never use `%*` as it retains option flags).
* Keep loop initializations (`set rc=0`) **outside** the loop label to avoid resetting status codes on subsequent iterations.
* Use `for /d` to support glob expansion (e.g. `Projekty\*`) matching directory structures safely:
  ```cmd
  set rc=0
  :projectLoop
  for /d %%d in (%1) do call :continuedLoop :processProject "%%~d" || exit /b
  if not "%~1"=="" shift && goto :projectLoop
  exit /b %rc%
  ```

### Continued Loop Execution
Use the `:continuedLoop` helper to handle `--continue` flags dynamically. This captures execution failures without aborting the parent script context:
```cmd
:continuedLoop
call %* && exit /b 0
set rc=%errorlevel%
if defined continue (exit /b 0) else (exit /b %rc%)
```

---

## 6. Context-Aware Redirection Logging

Always use structured, standardized logging subroutines. Logs should print to stderr (`>&2`) to avoid polluting data pipelines redirection.

### Standard Logging Methods
Each logging level represents a precise execution state:
* `call :error "message"`: Prefixes with `###` (indicates error/mismatch). Returns code `1`.
* `call :ok "message"`: Prefixes with `===` (indicates successful completion). Returns code `0`.
* `call :na "message"`: Prefixes with `---` (indicates skipped/not applicable actions). Returns code `0`.
* `call :info "message"`: Prefixes with `...` (indicates status updates/path inclusion). Returns code `0`.
* `call :run <command>`: Prefixes with `>>>` (logs dry-run command invocation before executing it).
* `call :log <label-prefix> "message"`: The underlying formatting engine.

### Dynamic Scope Prefixing
Incorporate standard context loop variables (like `projectName` and `templateName`) into the logging output dynamically to supply rich context:
```cmd
:log
setlocal
set "label=%~1"
set "message=%~2"
if defined projectName set "label=%~1 %projectName%:"
if defined templateName set "label=%~1 %projectName%: %templateName%:"
setlocal enabledelayedexpansion
echo !label! !message! >&2
exit /b 0
```

> [!IMPORTANT]
> Toggling the delayed expansion state inside `:log` prevents:
> 1. Redirection syntax errors (handles `<>` and `&` dynamically inside `!message!`).
> 2. Exclamation mark stripping (keeps literal `!` characters inside filenames intact by mapping variables when delayed expansion is **disabled**).

---

## 7. Tool Prerequisites & Installation Routines

Scripts should actively check for tool dependencies and supply an automated path-resolution and installation framework.

### Checking Required & Optional Tools
* Use `:requireTools` during script initialization to fail fast if binaries are missing.
* For optional tools, check them inline within narrowed scopes using `where /q <binary>`.

```cmd
:: Init checks
call :requireTools yq.exe tera.exe || exit /b

:: Narrowed scope optional tool check
where /q diff.exe && call :diffRefOutput && exit /b 1
```

```cmd
:requireTools
setlocal
set rc=0
for %%x in (%*) do where /q "%%~x" || call :error "missing tool in system PATH: %%~x" || set rc=1
exit /b %rc%
```

### Shadowed Path Resolution
Dynamically check configuration paths (like Winget links or Mise shims) and add them to the session path. Propagate values safely past the `setlocal` boundary:
```cmd
:useToolsPaths
call :usePaths "%LocalAppData%\Microsoft\WinGet\Links" "%LocalAppData%\mise\shims"
exit /b

:usePaths
setlocal enabledelayedexpansion
for /d %%d in (%*) do if exist "%%~d" if "!PATH:%%~d=!"=="!PATH!" call :info "use path: '%%~d'" && set PATH=!PATH!;%%d
endlocal & set "PATH=%PATH%"
exit /b 0
```

### Tool Installation Routines
Bind `--install` to install required binaries using available system managers (`winget` and `mise`):
```cmd
:installTools
setlocal
set rc=0
call :useToolsPaths
call :installWinGetTool MikeFarah.yq yq.exe || set rc=1
call :installWinGetTool Wilfred.difftastic difft.exe || set rc=1
call :installWinGetTool GnuWin32.DiffUtils diff.exe || set rc=1
call :installMiseTool github:chevdor/tera-cli tera.exe || set rc=1
exit /b %rc%

:installWinGetTool
if not "%~2"=="" where /q "%~2" && call :info "already installed: %~2 / winget:%~1" && exit /b 0
call :run winget install --id "%~1"
exit /b

:installMiseTool
if not "%~2"=="" where /q "%~2" && call :info "already installed: %~2 / mise:%~1" && exit /b 0
call :installWinGetTool jdx.mise mise.exe || exit /b
call :run mise use -g "%~1"
exit /b
```

---

## 8. Handling Delayed Expansion & Exclamation Mark Safety

When writing or modifying Windows batch files, you must be extremely cautious about the `!` character, especially when invoking tools like `powershell`, regex matchers, or processing files whose names/contents include `!`.

### The Inheritance Pitfall
If `setlocal enabledelayedexpansion` is active (either enabled explicitly in the current script or **inherited from a parent caller process/script**), `cmd.exe` evaluates the `!` character as a variable boundary (similar to `%`).

When passing commands inline:
```cmd
:: If delayed expansion is active, the `!` will be stripped or evaluated!
powershell -NoProfile -Command "if (!$a) { exit 1 }"
```
In the above example, `cmd.exe` parses the `!` and heavily mangles the string before passing it into `powershell.exe`. This silently breaks the logic (e.g., stripping the NOT operator `!` entirely) or causes bizarre parsing syntax errors.

### Toggling/Disabling Delayed Expansion Safely
If your batch file includes any inline code or arguments that require literal `!` characters, or if you are writing a robust script meant to be called by arbitrary parent scripts, you must explicitly disable delayed expansion locally:

```cmd
:: Ensure inherited delayed expansion is safely disabled
setlocal disabledelayedexpansion

:: Now `!` is treated as a literal character
powershell -NoProfile -Command "if (!$a) { exit 1 }"
```

If you must use delayed expansion (e.g., modifying state inside a parenthesized `for` loop block), use localized toggling to parse values under a disabled context and print or evaluate them under an enabled context:

```cmd
:: Parse variable while delayed expansion is disabled to prevent stripping literal !
set "val=%~1"
setlocal enabledelayedexpansion
:: Echo/Evaluate using delayed expansion boundaries
echo !val!
endlocal
```

---

## 9. Testing & Developing CMD Scripts in Sandboxes

When testing or creating loops, pipes, and complex syntax for Windows `cmd.exe` or `powershell` directly from your agent execution environment (`run_command`), you **must** avoid directly evaluating inline commands using `cmd.exe /c "..."` if the script relies heavily on escaping, symbols like `$`, `` ` ``, `%`, or `&`. 

### The Problem
If the parent shell (e.g., PowerShell or Bash) evaluates special characters before `cmd` encounters them, it leads to double-evaluation errors and false negatives.

### Standard Operating Procedure for Testing
1. **Direct Script Creation**:
   Instead of using `run_command` with inline strings containing escaping nightmares, use the `write_to_file` tool to create a temporary test script inside the conversation scratch directory: `<appDataDir>\brain\<conversation-id>\scratch\test_*.cmd`.
2. **Execution**:
   Use `run_command` to execute the file directly via Windows script engines:
   ```powershell
   # Executing a test .cmd script in PowerShell
   & "C:\Users\iki\.gemini\antigravity-ide\brain\09b38457-559d-4848-b7e0-648ae1aafea8\scratch\test_case.cmd" arg1 arg2
   ```
3. **Isolation**:
   Keeping testing artifacts in the `/scratch/` directory prevents workspace pollution while ensuring that script engines parse the test files under native conditions.
