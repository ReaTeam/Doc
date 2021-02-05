# ReaScript API - Stretch Markers

Managing Stretch Markers (SM) from ReaScript API is complex. Here are some guidelines, following the discussion on [FR/Q: Stretch Markers API and Slopes related actions/API functions - Cockos Incorporated Forums](https://forum.cockos.com/showthread.php?t=248801)

Basically, you have access to have count function, two get functions (one for positions, one for slope), associated with two set functions, and one delete.

All the other thing you might need, like setting rates, moving later SMs (just like with mouse slope editing) etc, need to be calculated using these functions.

## Core Ideas

- SM source positions is related to the positions in source media; it can never be below 0 or above source media length.
- SM positions is related to the positions in the item. It can be negative if the item is trimmed and the SM is hidden outside item edge. You have to take care about take start offset.
- A SM alone isn't helpful, you have to consider SM by pair/section. When you add a first SM to a item and move it, you will create 3 SM (two segments), one new SM being added at start of item source, and one at end. Taking an SM and its next one make more sense in most cases due to how rate is calculated. SM without a pair can't have a slope.

- SM positions are pre Take rate so you don't have to care about this.

## Calculations

### Rates

A SM section is characterized by its left (start) and right (end) rate. If they are the same, only left one is displayed on REAPER arrange view, and the slope is equal to 0. We can calculate SM sections rates using the following math:

```lua
count =  reaper.GetTakeNumStretchMarkers( take )
for i = 0, count - 1 do
  slope = reaper.GetTakeStretchMarkerSlope( take, i )
  retval, pos_a, srcpos_a = reaper.GetTakeStretchMarker( take, i )
  retval, pos_b, srcpos_b = reaper.GetTakeStretchMarker( take, i+1 )
  -- Calculation
  len_init = srcpos_b - srcpos_a -- length between two SM source positions
  len_after = pos_b - pos_a -- Length between two SM actual item positions
  rate_left = (len_init / len_after) * (1-slope)
  rate_right = (len_init / len_after) * (1+slope)
  rate_ratio = rate_right / rate_left
end
```

Rate value can't be equal or below 0.

### Equations

Slope parameter is a value between -1 (for rate acceleration) and 1 (for rate deceleration); 0 is for `left_rate = right_rate`.

Slope value can be extrapolated from this main equation:

```lua
(1+slope) / (1-slope) = rate_right / rate_left
```

As you can only modify SM pos and slope via API, all your calculation should end by modifying this variables. For eg, if you want to modify `rate_right` only, adjusting `slope`, you will have to express `pos_b` and `slope` with a known value of `rate_right`, starting from `rate_right` and `slope` equation. 

So, from the core equation, you can extrapolate the following:

```lua
slope = (rate_ratio - 1) / (rate_ratio + 1)
```

And then, calculate the SM position based your desired `rate_right`.

```lua
pos_b = (1+slope) / rate_right * (len_init) + pos_a
```

Processing right SM for position change is often the desired way to alter SM positions (like with mouse editing).

## Implementation

### Get SM Data

Getting all SMs in a table will make calculation faster, debugging easier, and allow to bypass some ReaScript API limitations.

```lua
function GetSMData( take )
  local SMs = {}
  local count =  reaper.GetTakeNumStretchMarkers( take )
  for i = 0, count - 1 do
    local slope = reaper.GetTakeStretchMarkerSlope( take, i )
    local retval, pos_a, srcpos_a = reaper.GetTakeStretchMarker( take, i )
    local retval, pos_b, srcpos_b = reaper.GetTakeStretchMarker( take, i+1 )
    local len_init = srcpos_b - srcpos_a
    local len_after = pos_b - pos_a
    local rate_right = (len_init / len_after) * (1+slope)
    local rate_left = (len_init / len_after) * (1-slope)
    if i == count - 1 then -- consider the following for the last SM
      rate_right = 1
      rate_left = 1
      slope = 0
    end
    table.insert( sm, {slope = slope, pos = pos_a, srcpos = srcpos_a, len_init = len_init, len_after = len_after, rate_right = rate_right, rate_left = rate_left})
  end
  return SMs
end
```

### Modify SM Data

One of this strong API limitation is that the Set Slope function doesn't behaves as it does with mouse SM editing. Indeed, with mouse, changing an SM slope ripple all the next SM. With API, nothing is ripple, and so, `rate_left` isn't preserved, which might not be expected: you have to deal with offsetting the right SM pos as well.

To make this more efficient, you can make your calculation from first to last SM, and have a `cumulated_offset` variable to store how much `new_pos_b` has been offset from `pos_b`, and adding this value to any every `new_pos_b` while looping.

```lua
cumulated_offset = 0
for i, sm in ipairs( SMs) do
  -- Apply cumulated offset to current marker
  local pos_a = sm.pos + cumulated_offset -- current SM pos with cumulated offset
  local pos_b = SMs[i+1].pos + cumulated_offset -- next SM pos with cumulated offset
  --[[ 
     You new_pos_b calculation here and then, based on what you want to achieve
  --]]
  local offset = new_pos_b - pos_b
  cumulated_offset = cumulated_offset + offset
  -- Setting new value
  sm.new_pos = pos_a -- pos with cumulated offset
  sm.new_slope = new_slope
end
```

### Set SM Data

SM Set API functions has a strong limitation: a SM can only be inserted between two existing SM. For this reason, we delete any existing SM first before insertion.

```lua
function ApplySMData(take, SMs)
  local count =  reaper.GetTakeNumStretchMarkers( take )
  reaper.DeleteTakeStretchMarkers( take, 0,  reaper.GetTakeNumStretchMarkers( take ) ) -- Remove existing SM
  for i, sm in ipairs(SMs) do
    local id = reaper.SetTakeStretchMarker( take, -1, sm.new_pos, sm.srcpos ) -- .new_pos value
    if id >= 1 then -- If there is a previous SM, then we can add a slope to the previous marker
      reaper.SetTakeStretchMarkerSlope( take, id-1, SMs[i-1].new_slope ) -- .new_slope value.
    end
  end
  reaper.Undo_OnStateChange("Apply SM")
end
```

## Examples

An implementation of all this has been made in the following script: **X-Raym_Smooth selected items stretch markers transitions by adjusting slope and rate.lua**.

## Edge Cases

In some cases, adding stretch marker by hand could lead to last marker src_pos equal to previous from last marker src_pos, so len_init is 0 and all the next math is wrong. Experimentation involved using + 0.0000001.

Also, if adding an SM at `pos = 0` is needed, you might try `0.0000001` instead.

## Credits

Doc written by X-Raym. Thanks mpl for having proved it was possible to edit SM without new API functions, and juliansader who cracked the `slope / rate` equation which allowed to get the other equations.

