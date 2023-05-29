## Usage:

```reaper [options] [filename.rpp] [filename.wav] [scriptfile.lua]```

Passing both project/template and a media file isn't supported as of Oct 10, 2021 (see Limitations below).  
Since build 6.80 passing either project/template or a media file AND a script file is supported, e.g.  
`reaper.exe projectfile.rpp scriptfile.lua`  
`reaper.exe -nonewinst media.wav scriptfile.lua`

## Options:

- **-audiocfg** : show audio configuration at startup
- **-cfgfile file.ini** : use full path for alternate resource directory, otherwise uses default path
- **-new** : start with new project
- **-template filename.rpp** : start with template project
- **-saveas newfilename.rpp** : save project (after creating/loading) as file
- **-renderproject filename.rpp** : render project and exit
- **-ignoreerrors** : do not show errors on load
- **-nosplash** : do not show slpash screen window
- **-splashlog /path/to/filename.log** : write splash screen message log to file; the first line of logfile.txt will include the version number, [source](https://forum.cockos.com/showthread.php?t=258280#4)
- **-newinst | -nonewinst** : override preference for new instance checking
- **-close[all][:save|:nosave]** : close project(s), optionally not prompting for save
- **-batchconvert filelist.txt** : batch converter mode, filelist.txt includes:
  - list of files to convert:
    `filename.wav`
      or
    `filename.wav(TAB CHARACTER)outputfile.wav`
  - \<CONFIG ...> block (optional) which can contain:
    - SRATE 44100 (omit to use source samplerate)
    - NCH 2 (omit to use source channel count)
    - RSMODE modeidx (resample mode, copy from project file)
    - DITHER 3 (1=dither, 2=noise shaping, 3=both)
    - USESRCSTART 1 (1=write source media BWF start offset to output)
    - USESRCMETADATA 1 (1=attempt to preserve original media file metadata if possible)
    - PAD_START 1.0 (leading silence in sec, can be negative)
    - PAD_END 1.0 (trailing silence in sec, can be negative)
    - OUTPATH 'path'
    - OUTPATTERN 'wildcardpattern'
    - FXCHAIN 'fxchainfilename' (use full path if specified, otherwise FxChains directory)
    - \<FXCHAIN
        (contents of .RfxChain file)
      \>
    - \<OUTFMT
      (base64 data, e.g. contents <RECORD_CFG block from project file)
      \>
    - \<METADATA
      (contents of RENDER_METADATA block from project file)
      \>

## Windows-only options:

- **-noactivate** : launch but do not activate

## Limitations:

As of Oct 13, 2021 Passing two filenames (like a template and a media file) via the command line is not yet supported ([source](https://forum.cockos.com/showthread.php?t=258395#18)), e.g. 
`X:\mypath\reaper.exe X:\myotherpath\my_template.rpp X:\myotherpath\my_file.mp4`

## Rerefernces:
https://forum.cockos.com/showthread.php?t=258487  
https://forum.cockos.com/showthread.php?t=258395


