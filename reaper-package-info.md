# REAPER Packages

Source: https://www.reaper.fm/sdk/reaper-package-info.txt

## Concept

reaper packages are a .zip file with the extension `.ReaperPkg`

the zip file must contain a file named `reaper-package-info.txt` (this file). other files in the zip file will be installed, but in most cases they MUST be in one of the supported subdirectories (see below).

---

## NAME is required

```javascript
NAME "Example Package with a Script"
```

**these are optional**

```javascript
AUTHOR "Justin Frankel"
VERSION "0.1"
URL "https://cockos.com/fictionalpackage/"
```

---

## INFO

if uncommented, contents of the referenced text file (change log, notes) will be displayed. (limited to about 16kb)

```javascript
INFO "textfile.txt"
```

---

## SCRIPT_NO_REG

if uncommented, the referenced script will not be installed. can be used to include support scripts, dependencies, optional content, etc that requires special handling by the user. (can end in `*` to match all files beginning with that string)

```javascript
SCRIPT_NO_REG Scripts/whatever.lua
```

---

## NO_OVERWRITE

if uncommented, prevent automatically overwriting an existing file. (user can still choose to overwrite) (can end in `*` to match all files beginning with that string)

```javascript
NO_OVERWRITE Scripts/whatever.lua
```

---

**other files contained in the zip file must be in one of the supported subdirectories:**

- AutomationItems
- ColorThemes
- Data
- Docs
- Effects
- FXChains
- KeyMaps
- MouseMaps
- MenuSets
- OSC
- presets
- ProjectTemplates
- Scripts
- TrackTemplates
- UserPlugins
- reaper_www_root
- Cursors
- LangPack

the filenames do not need to be specified in this file, they are pulled from the .zip directly.

---

**optional OS/arch-specific prefixes for paths:** (os_xyz will be quietly ignored)

```javascript
    os_win/          // any windows
    os_win_32/
    os_win_64/       // matches both x64 and arm64ec
    os_win_x64/      // matches only x64
    os_win_arm64ec/ 

    os_mac/          // any macOS
    os_mac_32/
    os_mac_64/       // matches both arm64 and x86_64
    os_mac_arm64/
    os_mac_x86_64/

    os_linux/        // any linux
    os_linux_i686/
    os_linux_x86_64/
    os_linux_armv7l/
    os_linux_aarch64/
```

if using these, you can specify `OS_NO_WARN 1` in this file to suppress warning if no matching OS path was found, if these OS-specific components are optional.

---

example: to install an extension to UserPlugins, you might have this directory structure:

```javascript
reaper-package-info.txt
os_win_64/UserPlugins/reaper_extension_win64.dll
os_win_32/UserPlugins/reaper_extension_win32.dll
os_mac/UserPlugins/reaper_extension_macos_universal.dylib
os_linux_x86_64/UserPlugins/reaper_extension_x86_64.so
```

---

**also optional (somewhat experimental, since profile support is already a bit undocumented):**

```javascript
FORCE_PROFILE "<profilename>"
```

- forces package to install in `Profiles/<profilename>`
- if a package specifies a profile name, it is permitted to create files outside of the permitted directories (e.g. reaper.ini, etc), for full customization of that profile.
- note that if a profile name is specified, then scripts will not be auto-registered.
- (it is assumed that the package manages such itself)
