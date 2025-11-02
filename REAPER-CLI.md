_If REAPER is run from the command line with an unrecognized parameter (like 'reaper -?' for example) the command line help will be displayed, including documentation for the -batchconvert file list ([source](https://forum.cockos.com/showthread.php?p=2690395#5))._  
_E.g. on Windows_  
_(reaper.exe installation path is on the system drive)_  
`C:\Users\%USER NAME%>cd ..`  
`C:\Users>cd ..`  
`C:\>Prorgam Files\REAPER\reaper.exe -?`  
_(reaper.exe installation path is on drive D)_  
`C:\Users\%USER NAME%>D:`  
`D:\>Prorgam Files\REAPER\reaper.exe -?`

## Usage:

```reaper.exe [options] [projectfile.rpp | mediafile.wav | scriptfile.lua [...]] | fxchainpreset.RfxChain | vstbank.fxb | vstpatch.fxp | vstpatch.vstpreset```

Multiple media files and/or scripts may be specified, and will be added or run in order.
**-nonewinst** can be used to add media files and/or run scripts in an already-running instance of REAPER.

FX chain, VST plugin bank and preset will be applied to a new automatically added track. The bank/preset will trigger instantiation of the compatible plugin on the newly added track. The feature is supported since build 6.43.

Passing both project/template and a media file isn't supported as of Oct 10, 2021 (see Limitations below).  
Since build 6.80 passing either project/template or a media file AND a script file is supported, e.g.  

`reaper.exe projectfile.rpp scriptfile.lua`  
`reaper.exe -nonewinst media.wav scriptfile.lua`

## Options:

- **-noactivate** : launch but do not activate _(until 7.29 was only supported on Windows, since then on MacOS/Linux as well)_
- **-audiocfg** : show audio configuration at startup
- **-cfgfile file.ini** : use full path for alternate resource directory, otherwise uses default path
- **-new** : start with new project
- **-template filename.rpp** : start with template project
- **-saveas newfilename.rpp** : save project (after creating/loading) as file
- **-renderproject filename.rpp** : render project and exit
- **-ignoreerrors** : do not show errors on load
- **-nosplash** : do not show slpash screen window
- **-splashlog /path/to/filename.log** : write splash screen message log to file; the first line of logfile.txt will include the version number, _([source](https://forum.cockos.com/showthread.php?t=258280#4))_
- **-newinst | -nonewinst** : override preference for new instance checking
- **-close[all][:save|:nosave][:exit]** : close project(s), optionally not prompting for save and exiting _(**:exit** argument since 7.29)_
- **-noactivate** : launch but do not activate
- **-batchconvert filelist.txt** : batch converter mode, filelist.txt includes:
  - list of files to convert:
    `filename.wav`
      or
    `filename.wav(TAB CHARACTER)outputfile.wav`
      plus any number of additional files
  - \<CONFIG ...> block (optional, can be generated via the Batch Converter presets button) which can contain:
    - SRATE 44100 (omit to use source samplerate)
    - NCH 2 (omit to use source channel count; *since 7.46:* -1=explode channels to separate files; -3=explode stereo pairs to separate files)
    - RSMODE modeidx (resample mode, copy from project file)
    - DITHER 3 (1=dither, 2=noise shaping, 3=both)
    - USESRCSTART 1 (1=write source media BWF start offset to output)
    - USESRCMETADATA 1 (1=attempt to preserve original media file metadata if possible)
    - TRIM_START 0.0 	(trim leading silence, 0.5=-6dB peak) -- _(since 7.29)_
    - TRIM_END 0.0 	(trim trailing silence, 0.5=-6dB peak) -- _(since 7.29)_
    - PAD_START 1.0 (leading silence in sec, can be negative) -- _(since 7.29)_
    - PAD_END 1.0 (trailing silence in sec, can be negative) -- _(since 7.29)_
    - PAD_OUTSIDE_TRIM 0 	(add padding outside trimmed content *(default: inside trimmed content)*
                          &1=add start padding before trimmed start, &2=add end padding after trimmed end) -- _(since 7.48)_
    - NORMALIZE 1 -6.0 0 (1=peak, 2=true peak, 3=lufs-i, 4=lufs-s, 5=lufs-m, *since 7.50:* 6=rms;  
                         2nd parameter is dB,  
                         3rd parameter: 1=normalize only if too loud)
    - BRICKWALL 1 -1.0 (1=peak, 2=true peak, 2nd parameter is dB ceiling) -- _(since 6.43)_
    - FADE 0.0 0.0 1 1 (fade-in length, fade-out length, fade-in shape, fade-out shape; length 0.001 = 1 ms)
    - OUTPATH 'path'
    - OUTPATTERN 'wildcardpattern'
    - FXCHAIN 'fxchainfilename' (use full path if specified, otherwise FxChains directory)
    - FX_NCH 4 (if not specified, FX will be configured to 4 channels)
    - CPULIMIT 0 (0 or omit=use all available CPU cores, 1=limit to 1 core, etc) -- _([since 6.81](https://forum.cockos.com/showthread.php?p=2690395))_
    - \<FXCHAIN  
          (contents of .RfxChain file)  
      \>
    - \<OUTFMT  
          (base64 data, e.g. contents of <RENDER_CFG or <RECORD_CFG block from project file)  
      \>
    - \<METADATA  
          (contents of RENDER_METADATA block from project file)  
      \>

## Limitations:

As of Oct 13, 2021 Passing two filenames (like a template and a media file) via the command line is not yet supported ([source](https://forum.cockos.com/showthread.php?t=258395#18)), e.g. 
`X:\mypath\reaper.exe X:\myotherpath\my_template.rpp X:\myotherpath\my_file.mp4`

## References:
[Using command line syntax with OS shortcuts](https://forum.cockos.com/showthread.php?t=258487)  
https://forum.cockos.com/showthread.php?t=258395  
[Using CONFIG block to batch convert files](https://forum.cockos.com/showthread.php?t=178689)

