 -- 0: filename
-- 1: filesize
-- 2: date
-- 3:format
-- 9: comment
-- 13: custom tag
-- 16: sample rate
-- 17: channels
-- 19: duration
-- 20: bitrate
-- 24: custom column 1

```lua
for i = 0, 30 do -- arbitrary limit
  local val = reaper.JS_ListView_GetItemText(file_LV, 0, i)
  Msg(i .." = " ..val)
end
```
