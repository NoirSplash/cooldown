# Cooldown by NoirSplash
**Simple non-yielding Roblox module for managing time between actions.**

Similar to a [Maid](https://medium.com/roblox-development/how-to-use-a-maid-class-on-roblox-to-manage-state-651bf74de98b), Cooldown intends to streamline development by keeping your debounce and cooldown management in one place.

<details>
<summary>How do I use this module?</summary>

## Installation

### Import the Module
**Option A: From Roblox**
- Get the module [here](https://www.roblox.com/library/14555653947/Cooldown).
- Insert the module from your toolbox into somewhere your script can see it.

**Option B: From Github**
- Find the Lua file [here](https://github.com/NoirSplash/cooldown/releases/tag/major-release).
- Import the file into roblox studio using one of the following methods;
  - Right click the object you want to be the parent of your module and `Insert from File...`. Change the file type to "Script Files" (from "All Roblox Model Files") and select the Lua file you downloaded. You must **transfer the contents of the script to a ModuleScript**.
    
    **or**
  - Open the RAW script (either from the downloaded file or github's raw text viewer) and paste its contents into a ModuleScript in your experience.

### Require the Module
- After you've imported the module into your experience, require it from any Script or LocalScript you intend to use it in. For this example, we've placed our module from the previous step into `ReplicatedStorage`.
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Cooldown = require(ReplicatedStorage.Cooldown()
```
Now that you've setup the module, take a look below to see how to use it!

---
</details>

> [!IMPORTANT]
> Cooldown is not a timer module. There are no methods or signals provided to listen for when a cooldown expires and their status is evaluated only when called.

---
[Roblox Model](https://www.roblox.com/library/14555653947/Cooldown) | [Latest Release](https://github.com/NoirSplash/cooldown/releases/tag/major-release)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/A0A8OKGQH)

## Properties

### GARBAGE_COLLECT_INTERVAL : number
`private, constant`

Describes how long (in seconds) the garbage collector should wait between cleanings. Expired cooldowns are automatically cleaned up when they are queried, garbage collection only affects cooldowns that expire and are never called on again. Does nothing if `Cooldown.doCleaning` is set to _false_. `default 120`

### Cooldown.doCleaning : boolean
`public, variable`

Determines if expired cooldowns are automatically cleaned up by the garbage collector. If set to `false`, the cache must be cleaned manually by iterating through and calling `Cooldown.get()` on each entry to remove expired cooldowns or by other means to avoid memory leaks. The loop cannot be restarted once disabled. `default true`

## Methods
### Cooldown.set(string, number, boolean?) -> ()
This function sets (or resets) a cooldown based on the given identifier and duration.
#### Parameters
| Name  | Type | Description |
| --- | --- | --- |
| cooldownId | string | The unique identifying string you will use to keep track of the cooldown. |
| duration | number | How many seconds (or milliseconds) from the current time that the cooldown will expire |
| isMillis | boolean? | Whether the duration provided is in seconds _(false)_ or milliseconds _(true)_. `default false`

### Cooldown.get(string) -> (number?)
Returns the remaining duration of the cooldown matching the identifier given or nil if it is expired/does not exist.
#### Parameters
| Name  | Type | Description |
| --- | --- | --- |
| cooldownId | string | The unique identifying string you gave to `Cooldown.set()`. |

## Code Examples
### Generic Debounce Pattern
The following code block shows off the intended usage for simple cooldowns (or debounce.) Because `Cooldown.get()` returns _nil_ if a cooldown is expired, the conditional statement will only evaluate true if the duration of the cooldown has elapsed or does not exist.
```lua
if Cooldown.get("Debounce") then
    return
end
Cooldown.set("Debounce", DEBOUNCE_LENGTH)
```

### Refreshable Cooldown/Combo Timer
Instead of blocking a function this code will reset the timer on a cooldown if it is active, else it will reset a variable. Keep in mind the value will not reset until called again, even if the timer expires.
```lua
local combo = 0
if Cooldown.get("ComboTimer") then
    combo += 1
    Cooldown.set("ComboTimer", COMBO_DURATION) -- Reset the timer so you can continue the combo!
else
    combo = 0
    Cooldown.set("ComboTimer", COMBO_DURATION) -- Start the combo chain!
end
```
**Simplified**
```lua
local combo = 0
local function combo()
    combo = if Cooldown.get("ComboTimer") then combo + 1 else 0
    Cooldown.set("ComboTimer", COMBO_DURATION)
end
```

### Jump Reducer
This script combines the above two techniques into a system that reduces the player's jump height the more consecutive jumps they make, without restricting jumping entirely.
```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local Cooldown = require(ReplicatedStorage.lib.Cooldown)

local DEFAULT_JUMP_HEIGHT = 7.2
local JUMP_COOLDOWN = 0.7
local REQUEST_DEBOUNCE = 75 -- milliseconds

local jumpCount = 0

local function setCooldownTimer()
    if Cooldown.get("LocalJump") then
        Cooldown.set("LocalJump", JUMP_COOLDOWN)
    else
        jumpCount = 0
        Cooldown.set("LocalJump", JUMP_COOLDOWN)
    end
end

UserInputService.JumpRequest:Connect(function()
    if Cooldown.get("JumpRequestDebounce") then
        return
    end
    Cooldown.set("JumpRequestDebounce", REQUEST_DEBOUNCE, true)
    setCooldownTimer()
    jumpCount += 1

    local character = Players.LocalPlayer.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    if not character or not humanoid then
        return
    end

    local jumpHeight = DEFAULT_JUMP_HEIGHT * math.clamp(1 - jumpCount * 0.1, 0, 1)
    humanoid.JumpHeight = jumpHeight
end)
```
