getgenv().Settings = {
    -- ESP Settings
    ESP = {
        Enabled = false,
        BoxColor = Color3.fromRGB(255, 255, 255),
        BoxThickness = 1,
        BoxTransparency = 1,
        BoxSize = {             -- Added back: user-defined box size.
            Width = 2900,
            Height = 4800
        },
        HealthBar = {           -- Fixed health bar settings; width is fixed at 2.
            Enabled = false,
            Color = Color3.fromRGB(0, 255, 0)
        }
    },
    
    -- Name ESP Settings
    NameESP = {
        Enabled = false,
        Color = Color3.fromRGB(255, 255, 255),
        Size = 13,
        Offset = Vector3.new(0, 2.5, 0)
    },
    
    -- Outline Settings
    Outline = {
        Enabled = false,
        Color = Color3.fromRGB(255, 255, 255),
        FillColor = Color3.fromRGB(0, 0, 0),
        FillTransparency = 1
    },
    
    -- Aimbot Settings
    Aimbot = {
        Enabled = false,
        Smoothness = 0.5,
        TargetPart = "Head",
        BindType = "Mouse", -- "Mouse" or "Key"
        MouseBind = Enum.UserInputType.MouseButton2,
        KeyBind = Enum.KeyCode.X,
        Method = "Mouse",
        StickyAim = false,
        Toggle = false,
        FOV = {                      -- New FOV settings
            Enabled = false,
            Shape = "Circle",        -- "Circle" or "Box"
            Size = 200,              -- Diameter for circle or side length for box
            Method = "FollowMouse"   -- "FollowMouse" or "Center"
        }
    },
    
    -- Team Check Settings
    TeamCheck = {
        Enabled = false,
        TeamColor = Color3.fromRGB(0, 255, 0),
        EnemyColor = Color3.fromRGB(255, 0, 0)
    },

    -- Tracer Settings
    Tracer = {
        Enabled = false,
        Color = Color3.fromRGB(255, 255, 255),
        Thickness = 1,
        Origin = "Bottom",
        Transparency = 1
    },

    -- CFrame Speed Settings
    CFrameSpeed = {
        Enabled = false,
        Speed = 2,
        Key = Enum.KeyCode.Z,
        HoldToUse = false
    },

    -- Orbit Settings
    Orbit = {
        Enabled = false,
        Key = Enum.KeyCode.X,
        Height = 0,
        Speed = 1,
        Distance = 10,
        HoldToUse = false,
        FaceTarget = true
    },

    -- Fly Settings
    Fly = {
        Enabled = false,
        Speed = 2,
        Key = Enum.KeyCode.F,
        HoldToUse = false,
        Multiplier = 5
    }
}

loadstring(game:HttpGet("https://raw.githubusercontent.com/z2de/universal-test/refs/heads/main/README.md"))()
