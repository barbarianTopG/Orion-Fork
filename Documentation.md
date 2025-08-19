# Orion Library Documentation

This document provides a comprehensive guide to using the Orion UI Library, including setup, window creation, the new key system, and all available UI elements.

## Table of Contents
1.  [Getting Started](#getting-started)
2.  [Key System (Optional)](#key-system-optional)
3.  [Window Creation (`:MakeWindow`)](#window-creation-makewindow)
4.  [Tabs (`:MakeTab`)](#tabs-maketab)
5.  [Sections (`:AddSection`)](#sections-addsection)
6.  [Elements](#elements)
    *   [Label (`:AddLabel`)](#label-addlabel)
    *   [Paragraph (`:AddParagraph`)](#paragraph-addparagraph)
    *   [Button (`:AddButton`)](#button-addbutton)
    *   [Toggle (`:AddToggle`)](#toggle-addtoggle)
    *   [Slider (`:AddSlider`)](#slider-addslider)
    *   [Dropdown (`:AddDropdown`)](#dropdown-adddropdown)
    *   [Keybind (`:AddBind`)](#keybind-addbind)
    *   [Textbox (`:AddTextbox`)](#textbox-addtextbox)
    *   [Colorpicker (`:AddColorpicker`)](#colorpicker-addcolorpicker)
7.  [Other Functions](#other-functions)
    *   [Notification (`:MakeNotification`)](#notification-makenotification)
    *   [Initialize Config (`:Init`)](#initialize-config-init)
    *   [Destroy (`:Destroy`)](#destroy-destroy)

---

## Getting Started

To begin using Orion Library, you first need to load it into your script.

    local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/barbarianTopG/Orion-Fork/refs/heads/main/source"))()

Once loaded, the first step is always to create a window. All other elements will be placed inside this window.

    local Window = OrionLib:MakeWindow({
        Name = "My Awesome Script",
        HidePremium = false,
        SaveConfig = true,
        ConfigFolder = "MyScriptConfig"
    })

---

## Key System (Optional)

The library includes an optional, fully customizable key system to authenticate users before they can access the UI.

### 1. How it Works
When enabled, a key input window will appear before the main UI. The user must enter a key. This key is then passed to a validation function that *you* define. If your function returns `true`, the UI loads. If it returns `false`, an error is shown.

### 2. Enabling and Configuring
The key system is enabled within the `:MakeWindow` configuration table.

| Option    | Type    | Default        | Description                               |
|-----------|---------|----------------|-------------------------------------------|
| `Enabled` | boolean | `false`        | Set to `true` to activate the key system. |
| `Title`   | string  | `"Key System"` | The main title of the key window.         |
| `Subtitle`| string  | `"Enter..."`   | The instructional text below the title.   |

**Example Configuration:**

    local Window = OrionLib:MakeWindow({
        Name = "My Awesome Script",
        -- ... other settings
        KeySystem = {
            Enabled = true,
            Title = "My Script Authentication",
            Subtitle = "Please enter your license key to continue."
        }
    })

### 3. Creating the Validation Logic (Crucial Step)

The library does not know what a "valid" key is. You must provide the validation logic by setting the `OnInvoke` property of `OrionLib.KeyValidationFunction`. This function receives the user's input and must return `true` for success or `false` for failure.

**This is where you would typically make an HTTP request to your own server/API to check the key.**

    -- IMPORTANT: Define this BEFORE you call MakeWindow!
    OrionLib.KeyValidationFunction.OnInvoke = function(key)
        -- This is where your custom validation logic goes.
        -- For example, you could check against a hardcoded key:
        if key == "123-ABC-SECRET" then
            return true -- The key is valid
        end
        
        -- Or, more commonly, you would send the key to your server:
        -- local response = game:HttpGet("https://your-api.com/validate?key=" .. key)
        -- if response == "VALID" then
        --     return true
        -- end

        return false -- The key is invalid
    end

### Full Key System Example

Here is a complete example of setting up the library with the key system.

    local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

    -- Step 1: Define your validation function.
    -- The user's input from the key box will be passed as the 'key' argument.
    OrionLib.KeyValidationFunction.OnInvoke = function(key)
        print("Checking key:", key)
        
        -- Replace this with your actual validation (e.g., an HttpGet request)
        if key == "mysecretkey" then
            print("Key was correct!")
            return true -- Allows the user to proceed
        else
            print("Key was incorrect.")
            return false -- Shows an "Invalid Key" error
        end
    end

    -- Step 2: Create the window with the KeySystem enabled.
    local Window = OrionLib:MakeWindow({
        Name = "My Key-Protected Script",
        HidePremium = false,
        SaveConfig = true,
        ConfigFolder = "KeyTestConfig",
        KeySystem = {
            Enabled = true,
            Title = "Script Verification",
            Subtitle = "Enter your key below to gain access."
        }
    })

    -- The code below this line will only run AFTER a valid key has been entered.
    local Tab = Window:MakeTab({
        Name = "Main",
        Icon = "rbxassetid://4483345998"
    })

    Tab:AddLabel("Welcome! Your key was accepted.")

---

## Window Creation (`:MakeWindow`)

This is the main function to initialize the UI. It returns a `Window` object that you use to create tabs.

**Usage:** `OrionLib:MakeWindow(configTable)`

**Configuration Options:**

| Option         | Type            | Default            | Description                                                                  |
|----------------|-----------------|--------------------|------------------------------------------------------------------------------|
| `Name`         | string          | `"Orion Library"`  | The title displayed at the top of the window.                                |
| `SaveConfig`   | boolean         | `false`            | If `true`, the state of toggles, sliders, etc., will be saved automatically. |
| `ConfigFolder` | string          | `Name`             | The name of the folder where the configuration file will be saved.           |
| `HidePremium`  | boolean         | `false`            | Hides the "Premium" text/features in the user info section.                  |
| `IntroEnabled` | boolean         | `true`             | Enables or disables the startup loading animation.                           |
| `IntroText`    | string          | `"Orion Library"`  | The text displayed during the intro animation.                               |
| `IntroIcon`    | string (asset)  | (default icon)     | The icon displayed during the intro animation.                               |
| `ShowIcon`     | boolean         | `false`            | If `true`, shows an icon next to the window name.                            |
| `Icon`         | string (asset)  | (default icon)     | The icon to display if `ShowIcon` is true.                                   |
| `CloseCallback`| function        | `function() end`   | A function that is called when the user closes the UI.                       |
| `KeySystem`    | table           | `{Enabled=false}`  | The configuration table for the Key System. See the section above for details. |

---

## Tabs (`:MakeTab`)

Tabs are used to organize your UI elements into different pages. You create tabs from the `Window` object.

**Usage:** `Window:MakeTab(configTable)`

**Configuration Options:**

| Option | Type   | Default | Description                                                        |
|--------|--------|---------|--------------------------------------------------------------------|
| `Name` | string | `"Tab"` | The name of the tab displayed on the left panel.                   |
| `Icon` | string | `""`    | An asset ID or Feather Icon name for the tab icon.                 |

**Example:**

    local HomeTab = Window:MakeTab({
        Name = "Home",
        Icon = "home" -- Using a Feather Icon name
    })

    local CombatTab = Window:MakeTab({
        Name = "Combat",
        Icon = "rbxassetid://4483345998" -- Using an asset ID
    })

---

## Sections (`:AddSection`)

Sections are containers within a tab used to group related elements under a title.

**Usage:** `Tab:AddSection(configTable)`

**Configuration Options:**

| Option | Type   | Default     | Description                              |
|--------|--------|-------------|------------------------------------------|
| `Name` | string | `"Section"` | The title of the section.                |

**Example:**

    local AimbotSection = CombatTab:AddSection({
        Name = "Aimbot Settings"
    })
    -- You would now add toggles, sliders, etc. to AimbotSection

---

## Elements

Elements are the interactive components of your UI. They are added to `Tab` or `Section` objects.

### Label (`:AddLabel`)
Displays a simple line of text.

**Usage:** `TabOrSection:AddLabel(text)`

**Returns:** A `Label` object with a `:Set(newText)` method.

**Example:**

    local MyLabel = HomeTab:AddLabel("This is a status label.")

    task.wait(5)
    MyLabel:Set("Status has been updated!")

### Paragraph (`:AddParagraph`)
Displays a title and a block of wrapped content text.

**Usage:** `TabOrSection:AddParagraph(title, content)`

**Returns:** A `Paragraph` object with a `:Set(newContent)` method.

**Example:**

    local Paragraph = HomeTab:AddParagraph("Information", "This script was made by YourName. Enjoy!")

### Button (`:AddButton`)
A clickable button that executes a function.

**Usage:** `TabOrSection:AddButton(configTable)`

**Configuration Options:**

| Option     | Type     | Default          | Description                              |
|------------|----------|------------------|------------------------------------------|
| `Name`     | string   | `"Button"`       | The text displayed on the button.        |
| `Callback` | function | `function() end` | The function to run when clicked.        |

**Returns:** A `Button` object with a `:Set(newText)` method.

**Example:**

    AimbotSection:AddButton({
        Name = "Execute Action",
        Callback = function()
            OrionLib:MakeNotification({Name = "Action", Content = "Button was clicked!"})
        end
    })

### Toggle (`:AddToggle`)
An on/off switch.

**Usage:** `TabOrSection:AddToggle(configTable)`

**Configuration Options:**

| Option     | Type     | Default          | Description                                                                     |
|------------|----------|------------------|---------------------------------------------------------------------------------|
| `Name`     | string   | `"Toggle"`       | The name of the toggle.                                                         |
| `Default`  | boolean  | `false`          | The initial state of the toggle (on/off).                                       |
| `Callback` | function | `function() end` | A function that runs when the state changes. It receives the new state as a boolean. |
| `Flag`     | string   | `nil`            | A unique identifier used for saving/loading configs.                            |
| `Save`     | boolean  | `false`          | Alias for `Flag`. If set, you should provide a `Flag` name.                     |


**Returns:** A `Toggle` object with a `:Set(boolean)` method and a `.Value` property.

**Example:**

    local AimbotToggle = AimbotSection:AddToggle({
        Name = "Enable Aimbot",
        Default = false,
        Callback = function(newValue)
            print("Aimbot enabled state:", newValue)
            if newValue == true then
                -- Start aimbot logic
            else
                -- Stop aimbot logic
            end
        end,
        Flag = "AimbotEnabled" -- This will be saved in the config file
    })

    -- You can also change it programmatically
    -- AimbotToggle:Set(true)

### Slider (`:AddSlider`)
A draggable slider to select a number within a range.

**Usage:** `TabOrSection:AddSlider(configTable)`

**Configuration Options:**

| Option      | Type     | Default          | Description                                                                     |
|-------------|----------|------------------|---------------------------------------------------------------------------------|
| `Name`      | string   | `"Slider"`       | The name of the slider.                                                         |
| `Min`       | number   | `0`              | The minimum value of the slider.                                                |
| `Max`       | number   | `100`            | The maximum value of the slider.                                                |
| `Default`   | number   | `50`             | The initial value of the slider.                                                |
| `Increment` | number   | `1`              | The step value. E.g., an increment of 5 will snap to 0, 5, 10...                |
| `ValueName` | string   | `""`             | A unit suffix to display after the value (e.g., "FOV", "%").                    |
| `Callback`  | function | `function() end` | A function that runs when the value changes. It receives the new number.        |
| `Flag`      | string   | `nil`            | A unique identifier used for saving/loading configs.                            |
| `Save`      | boolean  | `false`          | Alias for `Flag`.                                                               |

**Returns:** A `Slider` object with a `:Set(number)` method and a `.Value` property.

**Example:**

    AimbotSection:AddSlider({
        Name = "Aimbot FOV",
        Min = 10,
        Max = 200,
        Default = 50,
        Increment = 5,
        ValueName = "FOV",
        Callback = function(newValue)
            print("Aimbot FOV set to:", newValue)
        end,
        Flag = "AimbotFOV"
    })

### Dropdown (`:AddDropdown`)
A dropdown menu to select one option from a list.

**Usage:** `TabOrSection:AddDropdown(configTable)`

**Configuration Options:**

| Option     | Type         | Default          | Description                                                                  |
|------------|--------------|------------------|------------------------------------------------------------------------------|
| `Name`     | string       | `"Dropdown"`     | The name of the dropdown.                                                    |
| `Options`  | array        | `{}`             | A table of strings representing the choices. `{ "Head", "Chest", "Legs" }`.   |
| `Default`  | string       | `""`             | The initially selected option. Must exist in the `Options` table.            |
| `Callback` | function     | `function() end` | A function that runs when an option is selected. It receives the selected string.|
| `Flag`     | string       | `nil`            | A unique identifier used for saving/loading configs.                         |
| `Save`     | boolean      | `false`          | Alias for `Flag`.                                                            |

**Returns:** A `Dropdown` object with `:Set(optionString)` and `:Refresh(newOptions, shouldDeleteOld)` methods and a `.Value` property.

**Example:**

    local HitboxDropdown = AimbotSection:AddDropdown({
        Name = "Target Hitbox",
        Options = {"Head", "Torso", "HumanoidRootPart"},
        Default = "Torso",
        Callback = function(selectedOption)
            print("Target hitbox changed to:", selectedOption)
        end,
        Flag = "AimbotHitbox"
    })

    -- To update the options later:
    -- HitboxDropdown:Refresh({"Left Arm", "Right Arm"}, true) -- The 'true' deletes old options

### Keybind (`:AddBind`)
Allows the user to set a keybind to trigger an action.

**Usage:** `TabOrSection:AddBind(configTable)`

**Configuration Options:**

| Option     | Type          | Default                 | Description                                                                   |
|------------|---------------|-------------------------|-------------------------------------------------------------------------------|
| `Name`     | string        | `"Bind"`                | The name of the keybind.                                                      |
| `Default`  | `Enum.KeyCode`| `Enum.KeyCode.Unknown`  | The default key.                                                              |
| `Hold`     | boolean       | `false`                 | If `true`, the callback runs once on press and again on release (toggle mode).|
| `Callback` | function      | `function() end`        | The function to run. If `Hold` is true, it receives `true` on press and `false` on release.|
| `Flag`     | string        | `nil`                   | A unique identifier used for saving/loading configs.                          |
| `Save`     | boolean       | `false`                 | Alias for `Flag`.                                                             |

**Returns:** A `Bind` object with a `:Set(Enum.KeyCode)` method and a `.Value` property.

**Example:**

    -- Normal press-to-fire callback
    AimbotSection:AddBind({
        Name = "Aimbot Key",
        Default = Enum.KeyCode.E,
        Callback = function()
            print("Aimbot key pressed!")
        end,
        Flag = "AimbotKey"
    })

    -- Hold-to-enable callback
    AimbotSection:AddBind({
        Name = "Toggle Aimbot (Hold)",
        Default = Enum.KeyCode.Q,
        Hold = true,
        Callback = function(isHeld)
            print("Aimbot key is held:", isHeld)
        end,
        Flag = "AimbotToggleKey"
    })

### Textbox (`:AddTextbox`)
An input field for the user to type text.

**Usage:** `TabOrSection:AddTextbox(configTable)`

**Configuration Options:**

| Option         | Type     | Default          | Description                                                              |
|----------------|----------|------------------|--------------------------------------------------------------------------|
| `Name`         | string   | `"Textbox"`      | The name of the textbox.                                                 |
| `Default`      | string   | `""`             | The default text in the box.                                             |
| `TextDisappear`| boolean  | `false`          | If `true`, the text clears after the user presses Enter.                 |
| `Callback`     | function | `function() end` | A function that runs when the user presses Enter. It receives the input text.|

**Example:**

    HomeTab:AddTextbox({
        Name = "Set Player Speed",
        Default = "16",
        Callback = function(text)
            local speed = tonumber(text)
            if speed then
                game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = speed
            end
        end
    })

### Colorpicker (`:AddColorpicker`)
A full HSV color picker.

**Usage:** `TabOrSection:AddColorpicker(configTable)`

**Configuration Options:**

| Option     | Type       | Default                  | Description                                                              |
|------------|------------|--------------------------|--------------------------------------------------------------------------|
| `Name`     | string     | `"Colorpicker"`          | The name of the color picker.                                            |
| `Default`  | `Color3`   | `Color3.new(1, 1, 1)`    | The default color.                                                       |
| `Callback` | function   | `function() end`         | A function that runs when the color changes. It receives the new `Color3` value.|
| `Flag`     | string     | `nil`                    | A unique identifier used for saving/loading configs.                     |
| `Save`     | boolean    | `false`                  | Alias for `Flag`.                                                        |

**Returns:** A `Colorpicker` object with a `:Set(Color3)` method and a `.Value` property.

**Example:**

    local ESPTab = Window:MakeTab({Name = "Visuals"})
    ESPTab:AddColorpicker({
        Name = "ESP Box Color",
        Default = Color3.fromRGB(255, 0, 255),
        Callback = function(newColor)
            print("ESP color changed to:", newColor)
        end,
        Flag = "ESPColor"
    })

---

## Other Functions

### Notification (`:MakeNotification`)
Shows a notification pop-up at the bottom right of the screen.

**Usage:** `OrionLib:MakeNotification(configTable)`

**Configuration Options:**

| Option    | Type           | Default            | Description                              |
|-----------|----------------|--------------------|------------------------------------------|
| `Name`    | string         | `"Notification"`   | The title of the notification.           |
| `Content` | string         | `"Test"`           | The body text of the notification.       |
| `Image`   | string (asset) | (default icon)     | The icon for the notification.           |
| `Time`    | number         | `15`               | How long the notification stays on screen (seconds).|

**Example:**

    OrionLib:MakeNotification({
        Name = "Script Loaded",
        Content = "Welcome to My Awesome Script!",
        Time = 7
    })

### Initialize Config (`:Init`)
Loads the saved configuration file if `SaveConfig` was set to true in the window and a config file exists. This should be called *after* all elements have been created.

**Usage:** `OrionLib:Init()`

**Example:**

    local Window = OrionLib:MakeWindow({Name = "My Script", SaveConfig = true, ConfigFolder = "MyScript"})
    local Tab = Window:MakeTab({Name = "Main"})
    Tab:AddToggle({Name = "My Toggle", Flag = "Toggle1"})
    Tab:AddSlider({Name = "My Slider", Flag = "Slider1"})

    -- Load the saved values for Toggle1 and Slider1
    OrionLib:Init()

### Destroy (`:Destroy`)
Completely removes the UI and disconnects all of its events.

**Usage:** `OrionLib:Destroy()`

**Example:**

    -- This would be used in a script's self-destruct or unload function
    OrionLib:Destroy()
