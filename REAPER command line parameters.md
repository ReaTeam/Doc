# REAPER Command Line Parameters

Windows: ```C:\REAPER\reaper.exe -h```
Mac: ```/Applications/REAPER64.app/Contents/MacOS/REAPER -h```

## Usage

```
reaper [options] [filename.rpp] [filename.wav]
```

## Options

- `audiocfg`: show audio configuration at startup
- `cfgfile file.ini`: use full path for alternate resource directory, otherwise uses default path
- `new`: start with new project
- `template filename.rpp`: start with template project
- `saveas newfilename.rpp`: save project (after creating/loading) as file
- `renderproject filename.rpp`: render project and exit
- `ignoreerrors`: do not show errors on load
- `batchconvert filelist.txt`: batch converter mode, filelist.txt includes:
   - list of files to convert:
     ```filename.wav```
       or
     ```filename.wav(TAB CHARACTER)outputfile.wav```
   - `<CONFIG` block:
     - `<FXCHAIN` sub block with contents of FxChain file
     - `FXCHAIN` 'fxchainfilename' (use full path if specified, otherwise FxChains directory)
     - `<OUTFMT` block (base64 output format, copy from project file)
     - `SRATE` 44100 (omit to use source samplerate)
     - `NCH` 2 (omit to use source channel count)
     - `RSMODE` modeidx (resample mode, copy from project file)
     - `DITHER` 3 (1=dither, 2=noise shaping, 3=both)
     - `PAD_START` 1.0 (leading silence in sec, can be negative)
     - `PAD_END` 1.0 (trailing silence in sec, can be negative)
     - `OUTPATH` 'path'

## Windows-only Options

(Run the `-h` command line on windows)
