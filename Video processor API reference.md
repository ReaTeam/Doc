_The document is accessible via F1 shortcut within the Video processor IDE_

`===============================================`  
### Video Processor Structure  
`===============================================`  

Video processors are written in the EEL2 language, and are primarily code. If the first line is a comment beginning with `//`, it will be used as the name of the processor.  

The processor can define up to 40 parameters using special comment lines:  
`//@param [<idx>[:varname]|varname] 'name' [defval minval maxval centval step]`   
For more information on the code language, please see the appendix near the bottom of this text.  

`===============================================`  
### Special Variables Used by Processors  
`===============================================` 

`project_time`: project time in seconds  
`project_timeoffs`: project setting time offset in seconds  
`project_tempo`: current tempo in BPM  
`project_ts_num`: current time signature numerator  
`project_ts_denom`: current time signature denominator  
`project_time_qn`: current project position in QN  
`time`: item time in seconds (if in item)  
`framerate`: project FPS (30.0, 29.97, etc)  
`project_w`: project preferred video width (code can override this before drawing)  
`project_h`: project preferred video height (code can override this before drawing)  
`project_wh_valid`: set nonzero if `project_w/project_h` reflect actual project setting (otherwise could be media-defined)  
`colorspace`: current rendering colorspace, e.g. `RGBA`, `YV12`, or `YUY2`. You can override this before drawing (or between drawing). This may be set to 0 initially if the user has the Auto project colorspace set. It will be automatically changed if 0 and a drawing operation occurs or an input is successfully queried via `input_info()`.  
`param_wet`: if in FX form, wet/dry mix of effect.  
`param1..param40`: parameters  
`gfx_r`: current drawing color (red 0..1)  
`gfx_g`: current drawing color (green 0..1)  
`gfx_b`: current drawing color (blue 0..1)  
`gfx_a`: current drawing alpha (0..1)  
`gfx_a2`: current drawing color alpha channel value (RGB-only, 0..1, defaults to 1)  
`gfx_mode`: drawing mode  
    `0` = normal  
    `1` = additive  
    `3` = multiply (very different in `YUV` vs `RGBA`)  
    `17` = `(dest + src*gfx_a)*.5 + .5` (only valid when using `YUV` colorspaces)  
    `18` = `dest + (src-0.5)*gfx_a*2.0` (only valid when using `YUV` colorspaces)  
    `19` = absolute difference: `abs(dest-src)*gfx_a` (only valid when using `YUV` colorspaces)  
    `0x100` (flag ORed to above mode) for `blit()` to enable filtering (if possible)  
    `0x10000` (flag ORed to above mode) to use source alpha (only valid when using `RGBA` colorspace)  
    `0x40000` (flag ORed to above mode) to use extra clamping in normal mode (for out of range alpha/gradient values)  
    `0x80000` (flag ORed to above mode) to interpret `gfx_r/gfx_g/gfx_b` as `YUV` values (in `YUV` colorspaces)  
`gfx_dest`: destination image handle, or -1 for main framebuffer  

`===============================================`  
### Video Processor Specific Functions  
`===============================================`  

`input_count()`  
Returns number of inputs available (total), range `[0..n]`  

`input_track_count()`  
Returns the number of available inputs on discrete tracks  

`input_track(x)`  
Returns input for bottommost item or FX on discrete-track `x` (0 is first track with video item above current, etc)  

`input_track_exact_count()`  
Returns the number of tracks above the current track that could possibly contain video items.  

`input_track_exact(x)`  
Returns input for bottommost item or FX on track relative to current track. Returns -1000 if track does not contain any video items at the current time, or -10000 if no further tracks contain video.

`input_next_item(x)`  
Returns the next input after x which is on a different item or track

`input_next_track(x)`  
Returns the next input after x which is on a different track

`input_ismaster()`  
Returns 1.0 if current FX is on master chain, 2.0 if on monitoring FX chain

`input_info(input, w, h[,srctime, wet, parm1, ...])`  
Returns 1 if input is available, sets `w/h` to dimensions. If srctime specified, it will be set with the source-local time of the underlying media. if input is a video processor in effect form, automated parameters can be queried via `wet/parm1/...`

`input_get_name(input, #str)`  
Gets the input take name or track name. returns >0 on success

`input_match(startidx,#pattern[,...])`  
Searches inputs for input starting at startidx whose track or item matches #pattern (see EEL2 `match()` for syntax), returns -10000 if not found.

`input_matchi(startidx,#pattern[,...])`  
Searches inputs for input starting at startidx whose track or item case-insensitively matches #pattern (see EEL2 `matchi()` for syntax), returns -10000 if not found.

`gfx_img_alloc([w,h,clear])`  
Returns an image index for drawing (can create up to 32 images). contents of image undefined unless clear set.

`gfx_img_resize(handle,w,h[,clear])`  
Sets an image size (handle can be -1 for main framebuffer). contents of image undefined after resize, unless clear set. clear=-1 will only clear if resize occurred. Returns the image handle (if handle is invalid, returns a newly-allocated image handle)

`gfx_img_hold(handle)`  
Retains (cheaply) a read-only copy of an image in handle. This copy should be released using `gfx_img_free()` when finished. Up to 32 images can be held.

`gfx_img_getptr(handle)`  
Gets a unique identifier for an image, valid for while the image is retained. can be used (along with `gfx_img_hold`) to detect when frames change in a low frame rate video

`gfx_img_free(handle)`  
Releases an earlier allocated image index.

`gfx_img_info(handle,w,h)`  
Gets dimensions of image, returns 1 if valid (resize if inexplicably invalidated)

`gfx_set(r,[g=r,b=r,a=1,mode=0,dest,a2=1])`  
Updates `r/g/b/a/mode` to values specified, dest is only updated if parameter specified.

`gfx_blit(input[,preserve_aspect=0,x,y,w,h,srcx,srcy,srcw,srch])`  
Draws input to framebuffer. preserve_aspect=-1 for no fill in pad areas

`gfx_fillrect(x,y,w,h)`  
Fills a rectangle with the current `color/mode/alpha`

`gfx_procrect(x,y,w,h,channel_tab[,mode])`  
`Processes a rectangle with 768-entry channel table [256 items of 0..1 per channel]. specify mode=1 to use `Y` value for `U/V` source channels (colorization mode)

`gfx_evalrect(x,y,w,h,code_string[,flags,src,init_code_string,src2])`  
Processes a rectangle with code_string being executed for every pixel/pixel-group. Returns -1 if code_string failed to compile. Code should reference per pixel values (0-255, unclamped), depending on colorspace:  
    `RGBA`: `r/g/b/a` (0-255, unclamped)  
    `YUY2`: `y1,y2, u, v` (0-255, unclamped; `u/v` are centered at 128)  
    `YV12`: `y1-y4, u, v` (0-255, unclamped; `u/v` are centered at 128)  
Additional options:  
    `flags|=1` in order to prevent multiprocessing (if your routine needs  to process pixels in-order)  
    `flags|=2` to ignore output (analysis-only). This is only valid when not using `src2` and not using one of the 4/8 modes.  
    `flags|=4,8` only valid in `RGBA/YV12`, and only if `src/src2` not specified. `flags&8` means process in vertical slices (top to bottom unless `flags&4`). `flags&4` but not `flags&8` means right-to-left. In each case `y1-y4` are reordered for convenience (the same filter code can typically be used in various orientations).  
    If init_code_string specified, it will be executed in each thread context before processing  
    If `src` specified (and >= -1), `sr/sg/sb/sa`, `sy1/su/sv` etc will be available to read. In this case only the intersection of valid rectangles between `src` and the destination buffer will be processed.  
    If `src` and `src2` specified (and >= -1), `s2r/s2g/s2b/s2a`, `s2y1/s2u/s2v` etc will also be available to read.  
    Note: variables `_1-_99` are thread-local variables which will always be initialized to 0, and _0 will be initialized to the thread index (usually 0 or 1). 6.70+: `_slice` is the position of the slice (which may differ from _0 in 6.71+). `_slices` is a count of the multiprocessing slices, `_span` is the number of calls per line, and _slice_size is the size of each slice in lines (the last slice may vary in size).  

`gfx_gradrect(x,y,w,h, r,g,b,a [,drdx,dgdx,dbdx,dadx, drdy,dgdy,dbdy,dady])`  
Fills rectangle. `r/g/b/a` supply color at top left corner, `drdx` (if specified) is amount red changes per X-pixel, etc.

`gfx_rotoblit(srcidx, angle [,x, y, w, h, srcx, srcy, w, h, cliptosrcrect=0, centxoffs=0, centyoffs=0])`  
Blits with rotate. This function behaves a bit odd when the source and destination sizes/aspect ratios differ, so `gfx_deltablit()` is generally more useful.

`gfx_deltablit(srcidx, x,y,w,h, srcx,srcy, dsdx, dtdx, dsdy, dtdy, dsdxdy, dtdxdy[, dadx, dady, dadxdy])`  
Blits with source pixel transformation control. `S` and `T` refer to source coordinates: `dsdx` is  how much the source `X` position changes with each `X` destination pixel, `dtdx` is how much the source `Y` position changes with each `X` destination pixel, etc.

`gfx_xformblit(srcidx, x,y,w,h,  wdiv, hdiv, tab[, wantalpha=0])`  
Blits with a transformation table. tab is `wdiv*hdiv*2` table of source point coordinates. If `wantalpha=1`, tab is `wdiv*hdiv*3` table of src points including alpha for each point.

`gfx_keyedblit(input[,x,y,w,h,srcx,srcy,kv1,kv2,kv3,kv4])`  
Chroma-key blits, using the source color as key. kv1-kv4 meaning depends on colorspace:  
    `YV12/YUY2`:  
        `kv1` is `U` target (-0.5 default)  
        `kv2` is `V` target (-0.5 default)  
        `kv3` is closeness-factor (0.4 default)  
        `kv4` is the gain (2.0 default)  
    `RGBA`:  
        `kv1` is green-factor (1.0 default)  
        `kv2` is blue-factor (-1.0 default)  
        `kv3` is offset (-1.0 default)  
        `kv4` enables spill removal (1.0 default)  

`gfx_destkeyedblit(input[,x,y,w,h,srcx,srcy,kv1,kv2,kv3,kv4])`  
Chroma-key blits, using destination color as key. ignores `gfx_a` and `gfx_mode`. See `gfx_keyedblit()` for `kv1-kv4` explanation.

`gfx_setfont(pxsize[,#fontname, flags)`  
Sets a font. flags are specified as a multibyte integer, using a combination of the following flags (specify multiple as `BI` or `OI` or `OBI` etc):  
    `B` - Bold  
    `I` - Italics  
    `R` - Blur  
    `V` - Invert  
    `M` - Mono  
    `S` - Shadow  
    `O` - Outline  

`gfx_str_measure(#string[,w,h])`  
Measures the size of #string, returns width

`gfx_str_draw(#string[,x,y,fxc_r,fxc_g,fxc_b])`  
Draw string, `fxc_r/g/b` are the FX color if Shadow/Outline are set in the font

`gfx_getpixel(input,x,y,v1,v2,v3[,v4])`  
Gets the value of a pixel from input at `x,y`. `v1/v2/v3` will be `YUV` or `RGB` (`v4` can be used to get `A`), returns 1 on success

`rgb2yuv(r,g,b)`  
Converts `r,g,b` to `YUV`, does not clamp `[0..1]`

`yuv2rgb(r,g,b)`  
Converts `YUV` to `r,g,b`, not clamping `[0..1]`  

`===============================================`  
### Advanced Functions  
`===============================================`  

`ui_get_state(ctx[,mouse_x, mouse_y,force_frame_in,mouse_wheel_state,mouse_hwheel_state])`  
Gets UI state and context, only usable from Monitoring FX (returns 0 if used from track). Returns state (`1/2/4` are `l/r/m` mouse buttons, `8/16/32` are `ctrl/shift/alt`, `1024` is whether configuration for this processor is visible). If `ctx` set to -1, context is video window and any returned mouse coordinates are `[0..1]` (where `0,0` is upper left corner, `1,1` is lower right corner of the video area). If `ctx` is set to `[1..40]`, it means the user is editing that knob. If force_frame_in is specified and is positive, then the processor will be re-executed in this amount of time (even if no new video source is available)

`time_precise([val])`  
Sets the parameter (or a temporary buffer if omitted) to a system-local timestamp in seconds, and returns a reference to that value. The granularity of the value returned is system defined (but generally significantly smaller than one second).

`on_parameter_change(pvar[, isdone])`  
Notifies that the parameter pointed to by pvar (must be `param1..param40` or a user-defined parameter) has changed. Specify `isdone=1` when done modifying parameter (e.g. releasing touch)

`get_host_placement([chain_pos,flags])`  
Returns track index, or -1 for master track, or -2 for hardware output FX. `chain_pos` will be position in chain. flags will have 1 set if takeFX, 4 if inactive project, 8 if in container (in this case, `chain_pos` will be set to the address, see `TrackFX_GetParamEx` etc).

`fft(buffer,size)`  
Performs a FFT on the data in the local memory buffer at the offset specified by the first parameter. The size of the FFT is specified by the second parameter, which must be a power of two 16-32768. The outputs are permuted, so if you plan to use them in-order, call    `fft_permute(buffer, size)` before and `fft_ipermute(buffer,size)` after in-order use. Inputs or outputs will need to be scaled down by `1/size`. Notes:  
    `fft()/ifft()` require real / imaginary input pairs, so a 256 point FFT actually works with 512 items.
    `fft()/ifft()` must NOT cross a 65,536 item boundary, so be sure to specify the offset accordingly.

`ifft(buffer,size)`  
Performs an inverse FFT. For more information see `fft()`.

`fft_real(buffer,size)`  
Performs a real FFT, taking size input samples and producing size/2 complex output pairs. Usually used along with fft_permute(size/2). Inputs/outputs will need to be scaled by 0.5/size. The first output complex pair will be (DC, nyquist).

`ifft_real(buffer,size)`  
Performs an inverse FFT, taking `size/2` complex input pairs (the first being DC, nyquist) and producing size real output values. Usually used along with `fft_ipermute(size/2)`.

`fft_permute(buffer,size)`  
Permutes the output of `fft()` to have bands in-order.

fft_ipermute(buffer,size)
Permutes the input for ifft(), taking bands from in-order to the order ifft() requires. See fft() for more information.

`convolve_c(dest,src,size)`  
Multiplies each of `size` complex pairs in `dest` by the complex pairs in `src`. Often used for convolution.

`===============================================`  
### Global Shared Memory
`===============================================`  

`gmem[]` can be used for a shared memory buffer (similar to JSFX), you can specify a named buffer which can be shared with ReaScripts and JSFX, by using:
    `//@gmem=sharedBufferName`  
on a line by itself. Note that when synchronizing with ReaScripts or JSFX, all processing is asynchronous, so your code will have to deal with synchronization issues (including vast differences between `project_time` and `playback_position`, and including race conditions). For Lua ReaScripts, see the function `gmem_attach()`.

`===============================================`  
### String Functions
`===============================================`  

`sprintf(#dest,"format"[, ...])`  
Formats a string and stores it in `#dest`. Format specifiers begin with `%`, and may include:  
    `%%` = %  
    `%s` = string from parameter  
    `%d` = parameter as integer  
    `%i` = parameter as integer  
    `%u` = parameter as unsigned integer  
    `%x` = parameter as hex (lowercase) integer  
    `%X` = parameter as hex (uppercase) integer  
    `%c` = parameter as character  
    `%f` = parameter as floating point  
    `%e` = parameter as floating point (scientific notation, lowercase)  
    `%E` = parameter as floating point (scientific notation, uppercase)  
    `%g` = parameter as floating point (shortest representation, lowercase)  
    `%G` = parameter as floating point (shortest representation, uppercase)  

Many standard C `printf()` modifiers can be used, including:  
    `%.10s` = string, but only print up to 10 characters  
    `%-10s` = string, left justified to 10 characters  
    `%10s` = string, right justified to 10 characters  
    `%+f` = floating point, always show sign  
    `%.4f` = floating point, minimum of 4 digits after decimal point  
    `%10d` = integer, minimum of 10 digits (space padded)  
    `%010f` = integer, minimum of 10 digits (zero padded)  

Values for format specifiers can be specified as additional parameters to `sprintf`, or within `{}` in the format specifier (such as `%{varname}d`, in that case a global variable is always used).

`matchi("needle","haystack"[, ...])`  
Case-insensitive version of `match()`.

`match("needle","haystack"[, ...])`  
Searches for the first parameter in the second parameter, using a simplified regular expression syntax.  
   `*` = match 0 or more characters  
   `*?` = match 0 or more characters, lazy  
   `+` = match 1 or more characters  
   `+?` = match 1 or more characters, lazy  
   `?` = match one character  

You can also use format specifiers to match certain types of data, and optionally put that into a variable:  
   `%s` means 1 or more chars  
   `%0s` means 0 or more chars  
   `%5s` means exactly 5 chars  
   `%5-s` means 5 or more chars  
   `%-10s` means 1-10 chars  
   `%3-5s` means 3-5 chars  
   `%0-5s` means 0-5 chars  
   `%x`, `%d`, `%u`, and `%f` are available for use similarly  
   `%c` can be used, but can't take any length modifiers  
   Use uppercase (`%S`, `%D`, etc) for lazy matching  

See also `sprintf()` for other notes, including specifying direct variable references via `{}`.

`strlen("str")`  
Returns the length of the string passed as a parameter

`strcpy(#str,"srcstr")`  
Copies the contents of `srcstr` to `#str`, and returns `#str`

`strcat(#str,"srcstr")`  
Appends `srcstr` to `#str`, and returns `#str`

`strcmp("str","str2")`  
Compares strings, returning 0 if equal

`stricmp("str","str2")`  
Compares strings ignoring case, returning 0 if equal

`strncmp("str","str2",maxlen)`  
Compares strings giving up after `maxlen` characters, returning 0 if equal

`strnicmp("str","str2",maxlen)`  
Compares strings giving up after `maxlen` characters, ignoring case, returning 0 if equal

`strncpy(#str,"srcstr",maxlen)`  
Copies srcstr to `#str`, stopping after `maxlen` characters. Returns `#str`.

`strncat(#str,"srcstr",maxlen)`  
Appends `srcstr` to `#str`, stopping after `maxlen` characters of `srcstr`. Returns `#str`.

`strcpy_from(#str,"srcstr",offset)`  
Copies `srcstr` to `#str`, but starts reading `srcstr` at `offset`

`strcpy_substr(#str,"srcstr",offs,ml))`  
PHP-style (start at `offs`, offs<0 means from end, `ml` for maxlen, ml<0 = reduce length by this amt)

`str_getchar("str",offset[,type])`  
Returns the data at byte-offset `offset` of `str`. If `offset` is negative, position is relative to end of string. `type` defaults to signed char, but can be specified to read raw binary data in other formats (note the single quotes, these are single/multi-byte characters):  
   `'c'` - signed char  
   `'cu'` - unsigned char  
   `'s'` - signed short  
   `'S'` - signed short, big endian  
   `'su'` - unsigned short  
   `'Su'` - unsigned short, big endian  
   `'i'` - signed int  
   `'I'` - signed int, big endian  
   `'iu'` - unsigned int  
   `'Iu'` - unsigned int, big endian  
   `'f'` - float  
   `'F'` - float, big endian  
   `'d'` - double  
   `'D'` - double, big endian


`str_setchar(#str,offset,val[,type]))`  
Sets value at `offset` offset, `type` optional. `offset` may be negative to refer to offset relative to end of string, or between 0 and length, inclusive, and if set to length it will lengthen string. See `str_getchar()` for more information on types.

`str_setlen(#str,len)`  
Sets length of `#str` (if increasing, will be space-padded), and returns `#str`.

`str_delsub(#str,pos,len)`  
Deletes `len` characters at offset `pos` from `#str`, and returns `#str`.

`str_insert(#str,"srcstr",pos)`  
Inserts `srcstr` into `#str` at offset `pos`. Returns `#str`
