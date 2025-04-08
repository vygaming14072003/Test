--getgenv().team = "Marines" -- Change to "Pirates" if preferred
repeat wait() until game:IsLoaded() and game.Players.LocalPlayer:FindFirstChild("DataLoaded")

-- Reliable team selection method
if game:GetService("Players").LocalPlayer.PlayerGui:FindFirstChild("Main (minimal)") then
    repeat
        wait()
        local l_Remotes_0 = game.ReplicatedStorage:WaitForChild("Remotes")
        l_Remotes_0.CommF_:InvokeServer("SetTeam", getgenv().team)
        task.wait(3)
    until not game:GetService("Players").LocalPlayer.PlayerGui:FindFirstChild("Main (minimal)")
end


-- Global Variables
_G.AutoCollectChest = true
_G.CancelTween2 = false
_G.StopTween = false
_G.StopTween2 = false
_G.AutoRejoin = true
_G.starthop = true
_G.AutoHopEnabled = true
_G.LastPosition = nil
_G.LastTimeChecked = tick()
_G.LastChestCollectedTime = tick()
_G.AutoJump = true -- Flag to control auto jump
_G.Antikick = true
_G.AutoFightDarkbeard = nil
_G.FightDarkbeardOnlyWithFist = nil
_G.IsFightingBoss = false
_G.AutoCyborg = nil
_G.IsFightingCyborgBoss = false
_G.NeedCoreBrain = true -- Added to track Core Brain requirement
_G.HasCoreBrain = false -- Added to track if player has Core Brain
_G.HasFistOfDarkness = false -- Added to track if player has Fist of Darkness
_G.IsChestFarming = false
_G.IsCheckingForCoreBrain = false
_G.MicrochipPurchased = false -- Track if microchip has been purchased
_G.KeyDetected = false -- Added to track if key is detected
_G.FistDetected = false -- Added to track if Fist of Darkness is detected
_G.ClickAttempts = 0
_G.LastClickTime = 0
_G.ClickCooldown = 2 -- Cooldown 2 giây giữa các lần click
_G.MicrochipNotFound = false -- Flag để theo dõi thông báo "Microchip not found"
TweenSpeed = 350


-- Load external scripts
spawn(function()
    pcall(function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/mizuharasup/autobonuty/refs/heads/main/all.txt"))()
    end)
end)

spawn(function()
    pcall(function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/mizuharasup/autobonuty/refs/heads/main/zder.lua"))()
    end)
end)

-- Apply FPS Boost Function
local function ApplyFPSBoost()
    -- Use pcall for all operations to avoid errors
    
    -- Remove graphic effects in Lighting
    pcall(function()
        for i, v in pairs(game:GetService("Lighting"):GetChildren()) do
            if v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect") 
                or v:IsA("BloomEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Atmosphere") then
                v:Destroy()
            end
        end
    end)
    
    -- Remove FantasySky if present
    pcall(function()
        if game:GetService("Lighting"):FindFirstChild("FantasySky") then
            game:GetService("Lighting").FantasySky:Destroy()
        end
    end)
    
    -- Adjust lighting
    pcall(function()
        local l = game:GetService("Lighting")
        l.GlobalShadows = false
        l.FogEnd = 9e9
        l.Brightness = 0
    end)
    
    -- Change graphic settings
    pcall(function()
        settings().Rendering.QualityLevel = "Level01"
    end)
    
    pcall(function()
        UserSettings():GetService("UserGameSettings").SavedQualityLevel = Enum.SavedQualitySetting.Automatic
    end)
    
    -- Reduce volume
    pcall(function()
        UserSettings():GetService("UserGameSettings").MasterVolume = 0
    end)
    
    -- Safely adjust Terrain
    pcall(function()
        if workspace:FindFirstChild("Terrain") then
            local t = workspace.Terrain
            t.WaterWaveSize = 0
            t.WaterWaveSpeed = 0
            t.WaterReflectance = 0
            t.WaterTransparency = 0
        end
    end)
    
    -- Disable graphic effects (Process only 3000 elements each time to avoid freezing)
    local descendantCount = 0
    local maxDescendants = 3000
    pcall(function()
        for _, v in pairs(game:GetDescendants()) do
            descendantCount = descendantCount + 1
            if descendantCount > maxDescendants then break end
            
            if v:IsA("Part") or v:IsA("UnionOperation") or v:IsA("CornerWedgePart") or v:IsA("TrussPart") then
                v.Material = "Plastic"
                v.Reflectance = 0
            elseif v:IsA("Decal") or v:IsA("Texture") then
                v.Transparency = 1
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v.Lifetime = NumberRange.new(0)
                v.Enabled = false
            elseif v:IsA("Explosion") then
                v.BlastPressure = 1
                v.BlastRadius = 1
            elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") then
                v.Enabled = false
            elseif v:IsA("MeshPart") then
                v.Material = "Plastic"
                v.Reflectance = 0
                v.TextureID = 10385902758728957
            end
        end
    end)
    
    -- Notification of completion
    pcall(function()
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "FPS Boost",
            Text = "Graphics reduced successfully!",
            Duration = 5
        })
    end)
end
-- WhiteScreen function - Tắt/bật render 3D để tăng FPS
function ToggleWhiteScreen(enable)
    getgenv().Setting.WhiteScreen = enable
    
    if enable then
        -- Tắt render 3D khi bật WhiteScreen
        game:GetService("RunService"):Set3dRenderingEnabled(false)
        
        -- Thông báo cho người chơi
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "WhiteScreen",
            Text = "Đã bật chế độ WhiteScreen để tăng FPS",
            Duration = 3
        })
    else
        -- Bật lại render 3D khi tắt WhiteScreen
        game:GetService("RunService"):Set3dRenderingEnabled(true)
        
        -- Thông báo cho người chơi
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "WhiteScreen",
            Text = "Đã tắt chế độ WhiteScreen",
            Duration = 3
        })
    end
end

-- Thêm phần theo dõi thay đổi cài đặt WhiteScreen
spawn(function()
    local lastWhiteScreenSetting = getgenv().Setting.WhiteScreen
    
    while wait(1) do
        -- Kiểm tra nếu cài đặt đã thay đổi
        if getgenv().Setting.WhiteScreen ~= lastWhiteScreenSetting then
            lastWhiteScreenSetting = getgenv().Setting.WhiteScreen
        end
    end
end)

-- Khởi tạo cài đặt WhiteScreen ban đầu
-- Implement FPS Boost
ApplyFPSBoost()

-- Continue to run every minute (to reduce new graphics)
spawn(function()
    while wait(60) do
        pcall(ApplyFPSBoost)
    end
end)

-- No Collision
spawn(function()
    pcall(function()
        game:GetService("RunService").Stepped:Connect(function()
            if _G.AutoCollectChest or _G.IsFightingBoss or _G.IsFightingCyborgBoss then
                pcall(function()
                    local character = game:GetService("Players").LocalPlayer.Character
                    for _, descendant in pairs(character:GetDescendants()) do
                        if descendant:IsA("BasePart") then
                            descendant.CanCollide = false
                        end
                    end
                end)
            end
        end)
    end)
end)

-- Safe Tween function with error handling
function SafeTween(targetCF, speed)
    local success, result = pcall(function()
        if not game.Players.LocalPlayer.Character or not game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            return nil, "Character or HumanoidRootPart not found"
        end
        
        local Distance = (targetCF.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
        local actualSpeed = speed or TweenSpeed
        local tweenInfo = TweenInfo.new(Distance / actualSpeed, Enum.EasingStyle.Linear)
        local tween = game:GetService("TweenService"):Create(game.Players.LocalPlayer.Character.HumanoidRootPart, tweenInfo, {
            CFrame = targetCF
        })
        tween:Play()
        return tween, Distance / actualSpeed
    end)
    
    if success then
        return result
    else
        return nil, 0
    end
end
-- Hàm chống rơi và NoClip
function EnableNoClipAndAntiGravity()
    pcall(function()
        local character = game.Players.LocalPlayer.Character
        if not character then return end
        
        -- NoClip cho tất cả các part
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        
        -- Anti-gravity
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if hrp then
            -- Xóa BodyVelocity cũ nếu có
            for _, child in pairs(hrp:GetChildren()) do
                if child:IsA("BodyVelocity") and child.Name == "ChestFarmAntiGravity" then
                    child:Destroy()
                end
            end
            
            -- Tạo BodyVelocity mới
            local bodyVel = Instance.new("BodyVelocity")
            bodyVel.Name = "ChestFarmAntiGravity"
            bodyVel.MaxForce = Vector3.new(0, 9999, 0)
            bodyVel.Velocity = Vector3.new(0, 0.5, 0) -- Đẩy nhẹ lên trên
            bodyVel.P = 1500 -- Lực mạnh để chống rơi tốt hơn
            bodyVel.Parent = hrp
            
            -- Chống trạng thái Stun
            if character:FindFirstChild("Stun") then
                character.Stun.Value = 0
            end
            
            -- Chống rơi
            if character:FindFirstChild("Humanoid") then
                character.Humanoid:ChangeState(11)
                character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            end
        end
    end)
end
-- Hàm Tween2 giữ nguyên từ code cũ
function Tween2(targetCFrame)
    -- Đảm bảo NoClip và anti-gravity được kích hoạt
    EnableNoClipAndAntiGravity()
    
    -- Tạo tween mới
    pcall(function()
        local character = game.Players.LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return end
        
        local distance = (targetCFrame.Position - character.HumanoidRootPart.Position).Magnitude
        local speed = 350 -- Tốc độ bay
        
        -- Tạo tween info với easing style smooth
        local tweenInfo = TweenInfo.new(
            distance / speed,
            Enum.EasingStyle.Linear,
            Enum.EasingDirection.InOut,
            0, -- Số lần lặp lại (0 = không lặp)
            false, -- Đảo ngược
            0 -- Delay trước khi bắt đầu
        )
        
        -- Tạo và chạy tween
        local tween = game:GetService("TweenService"):Create(
            character.HumanoidRootPart,
            tweenInfo,
            {CFrame = targetCFrame}
        )
        
        -- Đăng ký sự kiện khi tween hoàn thành
        tween.Completed:Connect(function()
            -- Kích hoạt lại NoClip và anti-gravity sau khi tween hoàn thành
            EnableNoClipAndAntiGravity()
        end)
        
        -- Chạy tween
        tween:Play()
        
        -- Chờ tween hoàn thành (+ thêm 0.1s để đảm bảo)
        wait(distance / speed + 0.1)
    end)
end
-- BKP function
function BKP(Point)
    pcall(function()
        if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = Point
            task.wait()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = Point
        end
    end)
end

-- Tween function
function Tween(KG)
    pcall(function()
        if not game.Players.LocalPlayer.Character or not game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            return
        end
        
        local Distance = (KG.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
        local Speed = TweenSpeed  
        local tweenInfo = TweenInfo.new(Distance / Speed, Enum.EasingStyle.Linear)
        local tween = game:GetService("TweenService"):Create(game.Players.LocalPlayer.Character.HumanoidRootPart, tweenInfo, {
            CFrame = KG
        })
        tween:Play()
        if _G.StopTween then
            tween:Cancel()
        end
    end)
end

-- EquipTool function
function EquipTool(ToolSe)
    pcall(function()
        if game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe) then
            local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe)
            wait()
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
        end
    end)
end

-- Cancel Tween function
function CancelTween()
    _G.StopTween = true
    wait()
    pcall(function()
        Tween(game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame)
    end)
    wait()
    _G.StopTween = false
end

-- Equip function
function equip(tooltip)
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    
    if not humanoid then return false end
    
    -- Check Backpack
    for _, item in pairs(player.Backpack:GetChildren()) do
        if item:IsA("Tool") and item.ToolTip == tooltip then
            humanoid:EquipTool(item)
            return true
        end
    end
    
    -- Check if already equipped
    for _, item in pairs(character:GetChildren()) do
        if item:IsA("Tool") and item.ToolTip == tooltip then
            return true -- Already equipped
        end
    end
    
    return false
end

-- AutoHaki function
function AutoHaki()
    pcall(function()
        local player = game:GetService("Players").LocalPlayer
        if player.Character and not player.Character:FindFirstChild("HasBuso") then
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Buso")
        end
    end)
end

-- Cải tiến hàm kiểm tra tộc Cyborg để đáng tin cậy hơn
function isCyborg()
    local success, result = pcall(function()
        local player = game:GetService("Players").LocalPlayer
        local playerRace = player.Data.Race.Value
        return playerRace == "Cyborg"
    end)
    
    if success and result then
        -- Nếu đã có tộc Cyborg, tắt hết mọi hoạt động
        _G.AutoCyborg = false
        _G.AutoCollectChest = false
        _G.IsChestFarming = false
        _G.IsFightingBoss = false
        _G.AutoJump = false
        
        -- Thông báo cho người chơi
        pcall(function()
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "Auto Cyborg",
                Text = "Bạn đã có tộc Cyborg! Script đã dừng lại.",
                Duration = 10
            })
        end)
        
        return true
    else
        return false
    end
end

-- Check for Core Brain in inventory
function hasCoreBrain()
    local player = game:GetService("Players").LocalPlayer
    
    -- Check backpack
    for _, item in pairs(player.Backpack:GetChildren()) do
        if item.Name == "Core Brain" then
            -- Immediately disable auto chest when Core Brain is found
            _G.AutoCollectChest = false
            _G.IsChestFarming = false
            _G.HasCoreBrain = true
            _G.NeedCoreBrain = false
            _G.AutoJump = false -- Stop auto jump when Core Brain is found
            
            return true
        end
    end
    
    -- Check equipped items
    if player.Character then
        for _, item in pairs(player.Character:GetChildren()) do
            if item.Name == "Core Brain" then
                -- Immediately disable auto chest when Core Brain is found
                _G.AutoCollectChest = false
                _G.IsChestFarming = false
                _G.HasCoreBrain = true
                _G.NeedCoreBrain = false
                _G.AutoJump = false -- Stop auto jump when Core Brain is found
                
                
                return true
            end
        end
    end
    
    return false
end

-- Equip Core Brain if in inventory
function equipCoreBrain()
    local player = game:GetService("Players").LocalPlayer
    
    -- Check backpack
    for _, item in pairs(player.Backpack:GetChildren()) do
        if item.Name == "Core Brain" then
            local character = player.Character or player.CharacterAdded:Wait()
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:EquipTool(item)
                return true
            end
        end
    end
    
    -- Check if already equipped
    if player.Character then
        for _, item in pairs(player.Character:GetChildren()) do
            if item.Name == "Core Brain" then
                return true
            end
        end
    end
    
    return false
end
function hasFistOfDarkness()
    local player = game:GetService("Players").LocalPlayer
    
    -- Check backpack
    for _, item in pairs(player.Backpack:GetChildren()) do
        if item.Name == "Fist of Darkness" then
            if not _G.FistDetected then
                _G.FistDetected = true
                _G.HasFistOfDarkness = true
                _G.AutoJump = false
                _G.AutoCollectChest = false
                _G.IsChestFarming = false
                
                -- Notification and click detector
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Fist of Darkness Found",
                    Text = "Auto chest collection stopped. Processing...",
                    Duration = 10
                })
                
                -- Process Fist of Darkness
                processFistOfDarkness()
            end
            return true
        end
    end
    
    -- Check character
    if player.Character then
        for _, item in pairs(player.Character:GetChildren()) do
            if item.Name == "Fist of Darkness" then
                if not _G.FistDetected then
                    _G.FistDetected = true
                    _G.HasFistOfDarkness = true
                    _G.AutoJump = false
                    _G.AutoCollectChest = false
                    _G.IsChestFarming = false
                    
                    -- Process Fist of Darkness
                    processFistOfDarkness()
                end
                return true
            end
        end
    end
    
    return false
end

-- Check for Microchip in inventory
function hasMicrochip()
    local player = game:GetService("Players").LocalPlayer
    local hasMicrochipInInventory = false
    
    -- Check backpack
    for _, item in pairs(player.Backpack:GetChildren()) do
        if item.Name == "Microchip" then
            hasMicrochipInInventory = true
            break
        end
    end
    
    -- Check equipped items
    if not hasMicrochipInInventory and player.Character then
        for _, item in pairs(player.Character:GetChildren()) do
            if item.Name == "Microchip" then
                hasMicrochipInInventory = true
                break
            end
        end
    end
    
    return hasMicrochipInInventory
end

-- Buy Microchip function
function buyMicrochip()
    -- Check if player already has a microchip or already purchased one
    if hasMicrochip() then
        return true
    end
    
    if _G.MicrochipPurchased then
        if hasMicrochip() then
            return true
        else
            -- Reset the flag if we still don't have the microchip
            _G.MicrochipPurchased = false
        end
    end
    
    -- Buy microchip
    local args = { [1] = "BlackbeardReward", [2] = "Microchip", [3] = "2" }
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    
    -- Set flag to indicate purchase attempt
    _G.MicrochipPurchased = true
    
    -- Verify purchase was successful
    wait(1)
    if hasMicrochip() then
        return true
    else
        return false
    end
end

-- Buy Cyborg race function
function buyCyborgRace()
    local args = { [1] = "CyborgTrainer", [2] = "Buy" }
    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(unpack(args))
    
    -- Verify purchase was successful
    wait(2)
    if isCyborg() then
        _G.AutoCyborg = false
        _G.AutoCollectChest = false
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Auto Cyborg",
            Text = "Successfully obtained Cyborg race!",
            Duration = 10
        })
        return true
    else
        return false
    end
end

-- Core Brain position
local coreBrainPosition = CFrame.new(-6059.92236, 15.9929152, -5088.71289, -0.726370811, 3.41200179e-09, 0.687303007, 5.61764901e-09, 1, 9.72634195e-10, -0.687303007, 4.56752014e-09, -0.726370811)

-- Check if boss exists
function bossExists()
    return workspace.Enemies:FindFirstChild("Order") ~= nil
end

-- Find boss function
function findBoss()
    for _, v in pairs(workspace.Enemies:GetChildren()) do
        if v.Name == "Order" and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            return v
        end
    end
    
    return nil
end

-- Find the button's ClickDetector (Improved)
function findClickDetector()
    local success, result = pcall(function()
        -- Try to get the exact path from workspace
        local button = workspace:FindFirstChild("Map")
        if button then
            button = button:FindFirstChild("CircleIsland")
            if button then
                button = button:FindFirstChild("RaidSummon")
                if button then
                    button = button:FindFirstChild("Button")
                    if button then
                        button = button:FindFirstChild("Main")
                        if button then
                            local detector = button:FindFirstChild("ClickDetector")
                            if detector then
                                return detector, button.Position
                            end
                        end
                    end
                end
            end
        end
        
        -- Alternative search method if direct path fails
        for _, v in pairs(workspace:GetDescendants()) do
            if v.Name == "ClickDetector" and v.Parent and v.Parent.Name == "Main" and 
               v.Parent.Parent and v.Parent.Parent.Name == "Button" and
               v.Parent.Parent.Parent and v.Parent.Parent.Parent.Name == "RaidSummon" then
                return v, v.Parent.Position
            end
        end
        
        return nil, nil
    end)
    
    if success then
        return result
    else
        return nil, nil
    end
end

-- Check if player is close to button
function isPlayerCloseToButton(buttonPosition)
    local player = game.Players.LocalPlayer
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        return false
    end

    if not buttonPosition then
        return false
    end
    
    local playerPosition = character.HumanoidRootPart.Position
    local distance = (buttonPosition - playerPosition).Magnitude
    
    -- Most ClickDetectors have a range between 10-32 studs
    return distance <= 32
end

-- Teleport to button
function teleportToButton(buttonPosition)
    if not buttonPosition then
        return false
    end
    
    local player = game.Players.LocalPlayer
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        return false
    end
    
    -- Teleport near the button
    pcall(function()
        character.HumanoidRootPart.CFrame = CFrame.new(buttonPosition) + Vector3.new(0, 2, 0)
    end)
    wait(0.5) -- Wait for teleport to complete
    return true
end

-- Start NoClip function
function startNoClip()
    pcall(function()
        for _, part in pairs(game.Players.LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        
        -- Anti-gravity to prevent falling
        if game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            if not game.Players.LocalPlayer.Character.HumanoidRootPart:FindFirstChild("AntiGravity") then
                local ag = Instance.new("BodyVelocity")
                ag.Name = "AntiGravity"
                ag.MaxForce = Vector3.new(0, 9999, 0)
                ag.Velocity = Vector3.new(0, 0.1, 0)
                ag.Parent = game.Players.LocalPlayer.Character.HumanoidRootPart
            end
        end
    end)
end
-- Thêm biến để theo dõi thời gian click gần nhất
_G.LastClickTime = 0
_G.ClickCooldown = 2 -- Cooldown 2 giây giữa các lần click
_G.MicrochipNotFound = false -- Flag để theo dõi thông báo "Microchip not found"

function clickDetectorForNotification()
    -- Kiểm tra cooldown
    local currentTime = tick()
    if currentTime - _G.LastClickTime < _G.ClickCooldown then
        return false
    end
    
    -- Cập nhật thời gian click gần nhất
    _G.LastClickTime = currentTime
    
    local detector, buttonPosition = findClickDetector()
    
    if not detector then
        return false
    end
    
    -- Make sure player is close enough to click
    if not isPlayerCloseToButton(buttonPosition) and buttonPosition then
        teleportToButton(buttonPosition)
        wait(1) -- Wait after teleport
    end
    
    -- Chỉ click 2 lần thôi để tránh spam
    pcall(function() fireclickdetector(detector) end)
    wait(0.3)
    pcall(function() fireclickdetector(detector) end)
    
    return true
end

-- Khởi tạo môi trường khi script bắt đầu
spawn(function()
    -- Đợi 5 giây để đảm bảo game đã tải xong
    wait(5)
    
    -- Khởi tạo chức năng theo dõi chat và GUI
    setupChatMonitoring()
    setupGUIMonitoring()
    
    -- Khởi động chức năng AutoChestCollect nếu được bật
    if _G.AutoCollectChest then
        AutoChestCollect()
    end
    
    -- Debug info khi khởi động
end)

function handleNotifications(message)
    if string.find(message, "Microchip not found") then
        
        -- Đặt lại các cờ
        _G.MicrochipNotFound = true
        _G.AutoCollectChest = true
        _G.IsChestFarming = true
        _G.IsFightingBoss = false
        
        -- Đảm bảo không có cờ nào đang chặn AutoCollectChest
        _G.StopTween = false
        _G.StopTween2 = false
        _G.CancelTween2 = false
        
        -- Khởi động lại auto chest collect nếu nó đang không chạy
        if not _G.IsChestFarming then
            farmChestsForFistOfDarkness()
        end
        
        -- Khởi động lại chest collection ngay lập tức
        spawn(function()
            wait(1) -- Đợi 1 giây để các thông báo xử lý xong
            AutoChestCollect()
        end)
        
        return true
    elseif string.find(message, "Core Brain") then
        -- Tắt AutoCollectChest khi phát hiện Core Brain
        _G.AutoCollectChest = false
        _G.IsChestFarming = false
        _G.HasCoreBrain = true
        _G.NeedCoreBrain = false
        _G.AutoJump = false
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Core Brain Detected",
            Text = "Tiến hành mua tộc Cyborg",
            Duration = 5
        })
        
        -- Cầm Core Brain lên
        equipCoreBrain()
        
        -- Click detector
        clickDetectorForNotification()
        wait(5)  -- Chờ 5 giây
        
        -- Mua tộc Cyborg
        buyCyborgRace()
        
        -- Kiểm tra xem đã mua được chưa
        wait(2)
        if isCyborg() then
            _G.AutoCyborg = false
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "Success",
                Text = "Đã mua thành công tộc Cyborg!",
                Duration = 10
            })
        else

            clickDetectorForNotification()
            wait(1)
            buyCyborgRace()
        end
        
        return true
    end
    
    return false
end

-- Cập nhật hàm monitor GUI text
local function checkGUI(gui)
    pcall(function()
        if (gui:IsA("TextLabel") or gui:IsA("TextButton")) and gui.Visible and _G.AutoCyborg then
            if gui.Text then
                handleNotifications(gui.Text)
            end
        end
    end)
end

-- Cập nhật hàm monitor chat messages
function setupChatMonitoring()
    pcall(function()
        if game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents") then
            game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.OnMessageDoneFiltering.OnClientEvent:Connect(function(data)
                if data.Message then
                    if string.find(data.Message, "Microchip not found") then
                        handleNotifications("Microchip not found")
                    elseif string.find(data.Message, "Core Brain") then
                        handleNotifications("Core Brain")
                    end
                end
            end)
        end
    end)
end
function setupGUIMonitoring()
    pcall(function()
        local function checkGUI(gui)
            if (gui:IsA("TextLabel") or gui:IsA("TextButton")) and gui.Visible and _G.AutoCyborg then
                if gui.Text then
                    if string.find(gui.Text, "Microchip not found") then
                        handleNotifications("Microchip not found")
                    elseif string.find(gui.Text, "Core Brain") then
                        handleNotifications("Core Brain")
                    end
                end
            end
        end
        
        -- Kiểm tra GUI hiện tại
        for _, gui in pairs(game.Players.LocalPlayer.PlayerGui:GetDescendants()) do
            checkGUI(gui)
        end
        
        -- Theo dõi GUI mới
        game.Players.LocalPlayer.PlayerGui.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
                checkGUI(descendant)
                
                pcall(function()
                    descendant:GetPropertyChangedSignal("Text"):Connect(function()
                        checkGUI(descendant)
                    end)
                    
                    descendant:GetPropertyChangedSignal("Visible"):Connect(function()
                        checkGUI(descendant)
                    end)
                end)
            end
        end)
    end)
end
-- Cập nhật hàm fightBoss để bay tới boss thay vì teleport
function fightBoss()
    if _G.IsFightingBoss then
        return
    end
    
    _G.IsFightingBoss = true
    
    -- Start NoClip
    startNoClip()
    
    -- Enable Haki and equip weapon
    AutoHaki()
    equip("Melee")
    
    spawn(function()
        local attackCooldown = 0
        
        while _G.IsFightingBoss do
            -- Re-enable Haki periodically
            AutoHaki()
            
            if tick() - attackCooldown > 2 then
                equip("Melee")
                attackCooldown = tick()
            end
            
            -- Find boss
            local boss = findBoss()
            
            if boss and boss:FindFirstChild("HumanoidRootPart") and boss:FindFirstChild("Humanoid") and boss.Humanoid.Health > 0 then
                -- Check player health - if below 2000, fly 100 units above boss
                local player = game:GetService("Players").LocalPlayer
                local character = player.Character
                local humanoid = character and character:FindFirstChild("Humanoid")
                
                if humanoid and humanoid.Health < 2000 then
                    -- Turn off chest collection
                    _G.AutoCollectChest = false
                    
                    -- Get boss position
                    local bossPosition = boss.HumanoidRootPart.Position
                    -- Fly higher position (100 units above boss)
                    local higherPos = CFrame.new(
                        bossPosition.X, 
                        bossPosition.Y + 100, 
                        bossPosition.Z
                    )
                    
                    -- Fly to higher position (không teleport)
                    pcall(function()
                        Tween(higherPos)
                    end)
                    
                    -- Wait until health recovers to 5000
                    while humanoid.Health < 5000 do
                        -- Keep updating position to stay above boss
                        local currentBoss = findBoss()
                        if currentBoss and currentBoss:FindFirstChild("HumanoidRootPart") then
                            local currentBossPos = currentBoss.HumanoidRootPart.Position
                            local newHigherPos = CFrame.new(
                                currentBossPos.X,
                                currentBossPos.Y + 100,
                                currentBossPos.Z
                            )
                            pcall(function()
                                Tween(newHigherPos)
                            end)
                        end
                        wait(0.5)
                    end
                else
                    -- Move to boss position (slightly above the boss)
                    local bossPosition = boss.HumanoidRootPart.Position
                    local targetPos = CFrame.new(
                        bossPosition.X, 
                        bossPosition.Y + 25, 
                        bossPosition.Z
                    )
                    
                    -- Fly to boss position (không teleport)
                    pcall(function()
                        Tween(targetPos)
                    end)
                    
                    -- Face the boss
                    pcall(function()
                        if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(
                                game.Players.LocalPlayer.Character.HumanoidRootPart.Position,
                                Vector3.new(boss.HumanoidRootPart.Position.X, game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y, boss.HumanoidRootPart.Position.Z)
                            )
                        end
                    end)
                    
                    -- Attack
                    game:GetService("VirtualUser"):CaptureController()
                    game:GetService("VirtualUser"):ClickButton1(Vector2.new(0, 0))
                end
            elseif not boss then
                -- Check if boss is defeated
                _G.IsFightingBoss = false
                
                -- Wait a bit
                wait(1)
                
                -- Check if player has Core Brain after boss fight
                if hasCoreBrain() then
                    _G.HasCoreBrain = true
                    _G.NeedCoreBrain = false
                    _G.AutoCollectChest = false -- Tắt AutoCollectChest khi có Core Brain
                    _G.IsChestFarming = false   -- Đảm bảo tắt hoàn toàn farm rương
                    _G.starthop = false         -- Tắt auto hop
                    _G.AutoHopEnabled = false   -- Tắt auto hop
                    
                    -- Thông báo tìm thấy Core Brain
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Core Brain",
                        Text = "Đã tìm thấy Core Brain! Đang mua tộc Cyborg...",
                        Duration = 5
                    })
                    
                    -- Equip Core Brain
                    equipCoreBrain()
                    
                    -- Click detector for notification
                    clickDetectorForNotification()
                    wait(5)  -- Wait 5 seconds
                    
                    -- Buy Cyborg race
                    buyCyborgRace()
                    
                    -- Kiểm tra xem đã mua được chưa
                    wait(2)
                    if isCyborg() then
                        _G.AutoCyborg = false
                        game:GetService("StarterGui"):SetCore("SendNotification", {
                            Title = "Success",
                            Text = "Đã mua thành công tộc Cyborg!",
                            Duration = 10
                        })
                    else
                        -- Thử lại nhiều lần nếu chưa mua được
                        for i = 1, 3 do
                            clickDetectorForNotification()
                            wait(1)
                            buyCyborgRace()
                            wait(2)
                            
                            if isCyborg() then
                                _G.AutoCyborg = false
                                game:GetService("StarterGui"):SetCore("SendNotification", {
                                    Title = "Success",
                                    Text = "Đã mua thành công tộc Cyborg! (lần thử: " .. i .. ")",
                                    Duration = 10
                                })
                                break
                            end
                        end
                    end
                else
                    -- Reset microchip purchased flag
                    _G.MicrochipPurchased = false
                    
                    -- Kiểm tra Fist of Darkness
                    if hasFistOfDarkness() then
                        -- Thông báo cho người chơi
                        game:GetService("StarterGui"):SetCore("SendNotification", {
                            Title = "Fist of Darkness",
                            Text = "Đã có Fist of Darkness, đang mua Microchip...",
                            Duration = 5
                        })
                        
                        -- Tắt tạm thời farm rương trong khi mua microchip
                        _G.AutoCollectChest = false
                        _G.IsChestFarming = false
                        
                        -- Click detector để thử lại
                        clickDetectorForNotification()
                        wait(1)
                        
                        -- Kiểm tra và mua microchip (thử 3 lần)
                        if not hasMicrochip() then
                            for i = 1, 3 do
                                buyMicrochip()
                                wait(1)
                                if hasMicrochip() then
                                    game:GetService("StarterGui"):SetCore("SendNotification", {
                                        Title = "Microchip",
                                        Text = "Đã mua thành công Microchip!",
                                        Duration = 3
                                    })
                                    break
                                end
                            end
                        end
                        
                        -- Spawn boss nếu có microchip
                        if hasMicrochip() then
                            clickToSpawnBoss()
                        else
                            -- Nếu không thể mua microchip, bật lại farm rương
                            game:GetService("StarterGui"):SetCore("SendNotification", {
                                Title = "Lỗi",
                                Text = "Không thể mua Microchip! Tiếp tục farm rương...",
                                Duration = 5
                            })
                            _G.AutoCollectChest = true
                            _G.IsChestFarming = true
                        end
                    else
                        -- Nếu không có Fist of Darkness, bật lại AutoCollectChest
                        _G.AutoCollectChest = true
                        _G.IsChestFarming = true
                    end
                end
                
                -- Sửa phần cuối của hàm fightBoss()
-- Thêm phần còn thiếu sau dòng "break;"
                break
            end
            
            wait(0.1)
        end
    end)
end
-- Cập nhật clickToSpawnBoss để bay đến boss
function clickToSpawnBoss()
    if _G.IsFightingBoss then return end
    
    -- Kiểm tra Fist of Darkness
    if hasFistOfDarkness() and not _G.FistDetected then
        _G.FistDetected = true
        _G.HasFistOfDarkness = true
        _G.AutoJump = false
    end
    
    -- Click detector trước khi mua microchip
    clickDetectorForNotification()
    wait(1)
    
    -- Kiểm tra microchip trong inventory
    if not hasMicrochip() then
        buyMicrochip()
        wait(1)
    end
    
    -- Nếu có microchip, spawn boss
    if hasMicrochip() then
        local detector, buttonPosition = findClickDetector()
        if detector then
            -- Đảm bảo người chơi đủ gần để click
            if not isPlayerCloseToButton(buttonPosition) and buttonPosition then
                teleportToButton(buttonPosition)
                wait(1)
            end
            
            -- Click để spawn boss
            pcall(function() fireclickdetector(detector) end)
            wait(0.5)
            pcall(function() fireclickdetector(detector) end)
            
            wait(3)
            
            -- Kiểm tra xem boss đã spawn chưa và đánh
            if bossExists() then
                fightBoss()
            else
            end
        else
        end
    else
        -- Bật lại AutoCollectChest nếu không có microchip
        _G.AutoCollectChest = true
        _G.IsChestFarming = true
    end
end

-- Core Brain detection and handling
function checkForCoreBrain()
    if _G.IsCheckingForCoreBrain then return end
    
    _G.IsCheckingForCoreBrain = true
    
    -- Check if player already has Core Brain
    if hasCoreBrain() then
        _G.HasCoreBrain = true
        _G.NeedCoreBrain = false
        _G.AutoCollectChest = false
        _G.IsChestFarming = false
        _G.AutoJump = false

        
        -- Equip Core Brain
        equipCoreBrain()
        
        -- Click detector for notification
        clickDetectorForNotification()
        wait(5)  -- Wait 5 seconds
        
        -- Buy Cyborg race
        buyCyborgRace()
        
        _G.IsCheckingForCoreBrain = false
        return true
    end
    
    -- Check if race is already Cyborg
    if isCyborg() then
        _G.AutoCyborg = false
        _G.AutoCollectChest = false
        _G.IsCheckingForCoreBrain = false
        return true
    end
    
    -- First check for Fist of Darkness
    if hasFistOfDarkness() and not _G.FistDetected then
        _G.FistDetected = true
        _G.HasFistOfDarkness = true
        _G.AutoJump = false
        
        -- Click detector when Fist is found
        clickDetectorForNotification()
    end
    
    -- Find and click detector to check for messages
    clickDetectorForNotification()
    wait(3)
    
    -- Check if boss exists (which means we need a microchip, not Core Brain)
    if bossExists() then
        fightBoss()
        _G.IsCheckingForCoreBrain = false
        return false
    else
        -- If no boss, we need to continue chest farming
        _G.AutoCollectChest = true
        _G.IsChestFarming = true
        _G.IsCheckingForCoreBrain = false
        return false
    end
    
    _G.IsCheckingForCoreBrain = false
    return false
end

-- Farm chests until finding Fist of Darkness
function farmChestsForFistOfDarkness()
    if not _G.IsChestFarming then
        _G.IsChestFarming = true
        
        spawn(function()
            while _G.IsChestFarming and _G.NeedCoreBrain and not _G.HasCoreBrain do
                -- First priority: Check if we got Core Brain directly
                if hasCoreBrain() then
                    -- hasCoreBrain function will already disable chest farming
                    
                    -- Equip Core Brain
                    equipCoreBrain()
                    
                    -- Click detector for notification
                    clickDetectorForNotification()
                    wait(5)  -- Wait 5 seconds
                    
                    -- Buy Cyborg race
                    buyCyborgRace()
                    
                    break
                end
                
                -- Second priority: Check if we got Fist of Darkness
                if hasFistOfDarkness() and not _G.FistDetected then
                    _G.HasFistOfDarkness = true
                    _G.FistDetected = true
                    _G.AutoJump = false

                    -- Stop chest farming
                    _G.IsChestFarming = false
                    _G.AutoCollectChest = false
                    
                    -- Click detector multiple times
                    clickDetectorForNotification()
                    wait(1)
                    clickDetectorForNotification() 
                    wait(3)
                    
                    -- Check if boss exists now
                    if bossExists() then
                        fightBoss()
                    else
                        -- If still no boss, resume chest farming
                        if not hasCoreBrain() then  -- Double-check we don't have Core Brain
                            _G.AutoCollectChest = true
                            _G.IsChestFarming = true
                        end
                    end
                    
                    break
                end
                
                wait(3) -- Check more frequently (every 3 seconds)
            end
        end)
    end
end

-- Hàm dừng hoàn toàn quá trình thu thập rương
function ForceStopChestCollection()
    -- Đặt nhiều cờ để đảm bảo dừng hẳn việc thu thập rương
    _G.AutoCollectChest = false
    _G.IsChestFarming = false
    _G.starthop = false
    _G.AutoHopEnabled = false
    
    -- Hủy tween hiện tại
    _G.StopTween = true
    _G.StopTween2 = true
    _G.CancelTween2 = false
    
    -- Thông báo
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Auto Cyborg",
        Text = "Đã dừng thu thập rương để xử lý vật phẩm",
        Duration = 3
    })
    
    -- Đặt lại nhiều lần để đảm bảo
    spawn(function()
        for i = 1, 5 do
            wait(i * 0.2)
            _G.AutoCollectChest = false
            _G.IsChestFarming = false
        end
    end)
end

-- Main cycle function
function mainCycle()
    -- Kiểm tra ngay nếu đã có tộc Cyborg
    if isCyborg() then
        ForceStopChestCollection()
        _G.AutoCyborg = false
        _G.AutoJump = false
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Auto Cyborg",
            Text = "Bạn đã có tộc Cyborg! Script đã dừng lại.",
            Duration = 10
        })
        return
    end
    
    -- Kiểm tra nếu có Core Brain
    if hasCoreBrain() then
        ForceStopChestCollection()
        _G.HasCoreBrain = true
        _G.NeedCoreBrain = false
        _G.AutoJump = false
        
        -- Thông báo 
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Core Brain",
            Text = "Đã tìm thấy Core Brain! Đang mua tộc Cyborg...",
            Duration = 5
        })
        
        -- Equip Core Brain
        equipCoreBrain()
        
        -- Click detector for notification
        clickDetectorForNotification()
        wait(5)
        
        -- Buy Cyborg race
        buyCyborgRace()
        
        -- Kiểm tra lại xem đã mua được chưa
        wait(2)
        if not isCyborg() then
            -- Thử lại nếu chưa thành công
            for i = 1, 3 do
                clickDetectorForNotification()
                wait(1)
                buyCyborgRace()
                wait(2)
                
                if isCyborg() then break end
            end
        end
        
        return
    end
    
    -- Kiểm tra nếu cần Core Brain và không có Core Brain
    if _G.NeedCoreBrain and not _G.HasCoreBrain then
        farmChestsForFistOfDarkness()
    end
    
    -- Theo dõi liên tục trạng thái
    spawn(function()
        while _G.AutoCyborg do
            pcall(function()
                -- Kiểm tra Fist of Darkness trước
                if hasFistOfDarkness() and not _G.FistDetected then
                    ForceStopChestCollection()
                    _G.FistDetected = true
                    _G.HasFistOfDarkness = true
                    _G.AutoJump = false
                    
                    -- Thông báo
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Fist of Darkness",
                        Text = "Đã tìm thấy Fist of Darkness! Đang xử lý...",
                        Duration = 5
                    })
                    
                    -- Click detector nhiều lần
                    clickDetectorForNotification() 
                    wait(1)
                    clickDetectorForNotification()
                    
                    -- Kiểm tra microchip
                    if not hasMicrochip() then
                        -- Thông báo
                        game:GetService("StarterGui"):SetCore("SendNotification", {
                            Title = "Microchip",
                            Text = "Đang mua Microchip...",
                            Duration = 3
                        })
                        
                        for i = 1, 3 do
                            buyMicrochip()
                            wait(1)
                            if hasMicrochip() then break end
                        end
                    end
                    
                    -- Nếu có microchip, spawn boss
                    if hasMicrochip() then
                        clickToSpawnBoss()
                    end
                end
                
                -- Nếu có Core Brain, sử dụng nó
                if hasCoreBrain() and not isCyborg() then
                    ForceStopChestCollection()
                    _G.HasCoreBrain = true
                    _G.NeedCoreBrain = false
                    _G.AutoJump = false
                    
                    -- Thông báo
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Core Brain",
                        Text = "Đã tìm thấy Core Brain! Đang mua tộc Cyborg...",
                        Duration = 5
                    })
                    
                    -- Equip Core Brain
                    equipCoreBrain()
                    
                    -- Click detector for notification
                    clickDetectorForNotification()
                    wait(5)  -- Wait 5 seconds
                    
                    -- Buy Cyborg race
                    buyCyborgRace()
                    
                    -- Kiểm tra lại
                    wait(2)
                    if not isCyborg() then
                        for i = 1, 3 do
                            clickDetectorForNotification()
                            wait(1)
                            buyCyborgRace()
                            wait(2)
                            if isCyborg() then break end
                        end
                    end
                end
                
                -- Nếu phát hiện chìa khóa
                local Key = workspace:FindFirstChild("Key")
                if Key and not _G.KeyDetected then
                    -- Đặt cờ phát hiện chìa khóa
                    _G.KeyDetected = true
                    _G.AutoJump = false
                    
                    -- Thông báo
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Chìa khóa",
                        Text = "Đã phát hiện chìa khóa! Đang xử lý...",
                        Duration = 5
                    })
                    
                    -- Click detector nhiều lần
                    clickDetectorForNotification()
                    wait(1)
                    clickDetectorForNotification()
                    
                    -- Nếu có Fist of Darkness, sử dụng nó
                    if hasFistOfDarkness() and not bossExists() and not _G.IsFightingBoss then
                        ForceStopChestCollection()
                        _G.HasFistOfDarkness = true
                        _G.FistDetected = true
                        
                        -- Click detector lại
                        clickDetectorForNotification()
                        wait(3)
                        
                        -- Kiểm tra boss đã xuất hiện chưa
                        if bossExists() then
                            fightBoss()
                        end
                    else
                        -- Không có Fist of Darkness, nhưng có chìa khóa. Mua microchip và spawn boss
                        if not hasMicrochip() then
                            -- Thông báo
                            game:GetService("StarterGui"):SetCore("SendNotification", {
                                Title = "Microchip",
                                Text = "Đang mua Microchip...",
                                Duration = 3
                            })
                            
                            for i = 1, 3 do
                                buyMicrochip()
                                wait(1)
                                if hasMicrochip() then break end
                            end
                        end
                        
                        -- Nếu đã có microchip, click để spawn boss
                        if hasMicrochip() then
                            local detector, buttonPosition = findClickDetector()
                            if detector then
                                -- Đảm bảo người chơi đủ gần button
                                if buttonPosition and not isPlayerCloseToButton(buttonPosition) then
                                    teleportToButton(buttonPosition)
                                    wait(1)
                                end
                                
                                -- Thử các phương thức click
                                pcall(function() fireclickdetector(detector) end)
                                wait(0.5)
                                pcall(function() fireclickdetector(detector) end)
                                
                                wait(3)
                                
                                -- Kiểm tra xem boss đã xuất hiện chưa và đánh
                                if bossExists() then
                                    fightBoss()
                                end
                            end
                        end
                    end
                end
                
                -- Nếu boss tồn tại và chưa đánh, bắt đầu đánh
                if bossExists() and not _G.IsFightingBoss then
                    fightBoss()
                end
                
                -- Kiểm tra nếu đã có tộc Cyborg
                if isCyborg() then
                    ForceStopChestCollection()
                    _G.AutoCyborg = false
                    _G.AutoJump = false
                    
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Auto Cyborg",
                        Text = "Đã nhận thành công tộc Cyborg!",
                        Duration = 10
                    })
                    return
                end
                
                -- Kiểm tra xem người dùng có các công cụ mới không - đặc biệt là Fist of Darkness
                local player = game.Players.LocalPlayer
                if player and player.Backpack then
                    local checkBackpack = player.Backpack:GetChildren()
                    for _, item in pairs(checkBackpack) do
                        if item.Name == "Fist of Darkness" and not _G.FistDetected then
                            ForceStopChestCollection()
                            _G.FistDetected = true
                            _G.HasFistOfDarkness = true
                            _G.AutoJump = false
                            
                            -- Thông báo
                            game:GetService("StarterGui"):SetCore("SendNotification", {
                                Title = "Fist of Darkness",
                                Text = "Đã tìm thấy Fist of Darkness! Đang xử lý...",
                                Duration = 5
                            })
                            
                            -- Click detector nhiều lần
                            clickDetectorForNotification()
                            wait(1)
                            clickDetectorForNotification()
                            break
                        end
                    end
                end
                
                if player and player.Character then
                    local checkCharacter = player.Character:GetChildren()
                    for _, item in pairs(checkCharacter) do
                        if item.Name == "Fist of Darkness" and not _G.FistDetected then
                            ForceStopChestCollection()
                            _G.FistDetected = true
                            _G.HasFistOfDarkness = true
                            _G.AutoJump = false
                            
                            -- Thông báo
                            game:GetService("StarterGui"):SetCore("SendNotification", {
                                Title = "Fist of Darkness",
                                Text = "Đã tìm thấy Fist of Darkness! Đang xử lý...",
                                Duration = 5
                            })
                            
                            -- Click detector nhiều lần
                            clickDetectorForNotification()
                            wait(1)
                            clickDetectorForNotification()
                            break
                        end
                    end
                end
            end)
            
            wait(2) -- Kiểm tra mỗi 2 giây
        end
    end)
end
-- Add processFistOfDarkness function
function processFistOfDarkness()
    -- First check if player already has Cyborg race
    if isCyborg() then
        _G.AutoCyborg = false
        _G.AutoCollectChest = false
        _G.IsChestFarming = false
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Auto Cyborg",
            Text = "You already have Cyborg race! Script stopped.",
            Duration = 10
        })
        return
    end
    
    -- Stop chest collection temporarily
    ForceStopChestCollection()
    
    -- Notification
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Cyborg Process",
        Text = "Checking for Core Brain...",
        Duration = 5
    })
    
    -- Click detector to check for game messages about Core Brain
    local detector, buttonPosition = findClickDetector()
    if detector then
        -- Make sure player is close enough to button
        if buttonPosition and not isPlayerCloseToButton(buttonPosition) then
            teleportToButton(buttonPosition)
            wait(1)
        end
        
        -- Click to check for messages
        pcall(function() fireclickdetector(detector) end)
        wait(0.5)
        pcall(function() fireclickdetector(detector) end)
        
        -- Wait to ensure messages are processed
        wait(2)
    end
    
    -- Check if player already has Core Brain
    if hasCoreBrain() then
        -- Player has Core Brain, proceed to buy Cyborg race
        _G.HasCoreBrain = true
        _G.NeedCoreBrain = false
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Core Brain",
            Text = "Core Brain found! Buying Cyborg race...",
            Duration = 5
        })
        
        -- Equip Core Brain
        equipCoreBrain()
        
        -- Click detector to use Core Brain
        clickDetectorForNotification()
        wait(3)
        
        -- Buy Cyborg race
        buyCyborgRace()
        
        -- Check if successfully purchased
        wait(2)
        if isCyborg() then
            _G.AutoCyborg = false
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "Success",
                Text = "Successfully obtained Cyborg race!",
                Duration = 10
            })
        else
            -- Try again if failed
            for i = 1, 3 do
                clickDetectorForNotification()
                wait(1)
                buyCyborgRace()
                wait(2)
                
                if isCyborg() then
                    _G.AutoCyborg = false
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Success",
                        Text = "Successfully obtained Cyborg race! (attempt: " .. i .. ")",
                        Duration = 10
                    })
                    break
                end
            end
        end
        return
    end
    
    -- Check if boss already exists (this means we have used Fist of Darkness)
    if bossExists() then
        -- Boss already spawned, fight it
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Boss Found",
            Text = "Boss already spawned, fighting now...",
            Duration = 3
        })
        fightBoss()
        
        -- Wait for boss fight to complete
        local waitTime = 0
        while _G.IsFightingBoss and waitTime < 300 do
            wait(1)
            waitTime = waitTime + 1
        end
        
        -- Additional wait to ensure items are collected
        wait(3)
        
        -- Check for Core Brain after boss is defeated
        if hasCoreBrain() then
            _G.HasCoreBrain = true
            _G.NeedCoreBrain = false
            
            -- Equip Core Brain
            equipCoreBrain()
            
            -- Click detector to use Core Brain
            clickDetectorForNotification()
            wait(3)
            
            -- Buy Cyborg race
            buyCyborgRace()
            
            -- Check if successfully purchased
            wait(2)
            if isCyborg() then
                _G.AutoCyborg = false
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Success",
                    Text = "Successfully obtained Cyborg race!",
                    Duration = 10
                })
            else
                -- Try again if failed
                for i = 1, 3 do
                    clickDetectorForNotification()
                    wait(1)
                    buyCyborgRace()
                    wait(2)
                    
                    if isCyborg() then
                        _G.AutoCyborg = false
                        game:GetService("StarterGui"):SetCore("SendNotification", {
                            Title = "Success",
                            Text = "Successfully obtained Cyborg race! (attempt: " .. i .. ")",
                            Duration = 10
                        })
                        break
                    end
                end
            end
        else
            -- If no Core Brain after boss fight, buy Microchip and try again
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "Core Brain",
                Text = "Core Brain not found. Buying Microchip again...",
                Duration = 5
            })
            
            -- Reset Microchip purchased flag
            _G.MicrochipPurchased = false
            
            -- Buy Microchip if needed
            if not hasMicrochip() then
                buyMicrochip()
                wait(1)
            end
            
            -- Click detector to spawn boss
            clickDetectorForNotification()
            wait(3)
            
            -- Fight boss if it exists
            if bossExists() then
                fightBoss()
            else
                -- If boss doesn't spawn, resume chest farming
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Error",
                    Text = "Couldn't spawn boss! Resuming chest farming...",
                    Duration = 5
                })
                _G.AutoCollectChest = true
                _G.IsChestFarming = true
            end
        end
        return
    end
    
    -- If we have Fist of Darkness but no boss, use it
    if hasFistOfDarkness() then
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Fist of Darkness",
            Text = "Using Fist of Darkness...",
            Duration = 5
        })
        
        -- Click detector to use Fist of Darkness
        local detector, buttonPosition = findClickDetector()
        if detector then
            -- Make sure player is close enough to button
            if buttonPosition and not isPlayerCloseToButton(buttonPosition) then
                teleportToButton(buttonPosition)
                wait(1)
            end
            
            -- Click to use Fist of Darkness
            pcall(function() fireclickdetector(detector) end)
            wait(0.5)
            pcall(function() fireclickdetector(detector) end)
            
            -- Wait to ensure the click is processed
            wait(2)
        end
        
        -- Check if boss spawned after using Fist of Darkness
        if bossExists() then
            -- Boss spawned, fight it
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "Boss Spawned",
                Text = "Fighting boss now...",
                Duration = 3
            })
            fightBoss()
            
            -- Wait for boss fight to complete
            local waitTime = 0
            while _G.IsFightingBoss and waitTime < 300 do
                wait(1)
                waitTime = waitTime + 1
            end
            
            -- Additional wait to ensure items are collected
            wait(3)
            
            -- Check for Core Brain after boss is defeated
            if hasCoreBrain() then
                _G.HasCoreBrain = true
                _G.NeedCoreBrain = false
                
                -- Equip Core Brain
                equipCoreBrain()
                
                -- Click detector to use Core Brain
                clickDetectorForNotification()
                wait(3)
                
                -- Buy Cyborg race
                buyCyborgRace()
                
                -- Check if successfully purchased
                wait(2)
                if isCyborg() then
                    _G.AutoCyborg = false
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Success",
                        Text = "Successfully obtained Cyborg race!",
                        Duration = 10
                    })
                else
                    -- Try again if failed
                    for i = 1, 3 do
                        clickDetectorForNotification()
                        wait(1)
                        buyCyborgRace()
                        wait(2)
                        
                        if isCyborg() then
                            _G.AutoCyborg = false
                            game:GetService("StarterGui"):SetCore("SendNotification", {
                                Title = "Success",
                                Text = "Successfully obtained Cyborg race! (attempt: " .. i .. ")",
                                Duration = 10
                            })
                            break
                        end
                    end
                end
            else
                -- If no Core Brain after boss fight, buy Microchip and try again
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Core Brain",
                    Text = "Core Brain not found. Buying Microchip again...",
                    Duration = 5
                })
                
                -- Reset Microchip purchased flag
                _G.MicrochipPurchased = false
                
                -- Buy Microchip if needed
                if not hasMicrochip() then
                    buyMicrochip()
                    wait(1)
                end
                
                -- Click detector to spawn boss
                clickDetectorForNotification()
                wait(3)
                
                -- Fight boss if it exists
                if bossExists() then
                    fightBoss()
                else
                    -- If boss doesn't spawn, resume chest farming
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Error",
                        Text = "Couldn't spawn boss! Resuming chest farming...",
                        Duration = 5
                    })
                    _G.AutoCollectChest = true
                    _G.IsChestFarming = true
                end
            end
            return
        else
            -- If boss didn't spawn after using Fist of Darkness, buy Microchip
            if not hasMicrochip() then
                game:GetService("StarterGui"):SetCore("SendNotification", {
                    Title = "Microchip",
                    Text = "Buying Microchip...",
                    Duration = 3
                })
                
                -- Buy Microchip
                buyMicrochip()
                wait(1)
                
                if hasMicrochip() then
                    _G.MicrochipPurchased = true
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Microchip",
                        Text = "Successfully purchased Microchip!",
                        Duration = 3
                    })
                else
                    -- If couldn't buy Microchip, resume chest farming
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Error",
                        Text = "Couldn't buy Microchip! Resuming chest farming...",
                        Duration = 5
                    })
                    _G.AutoCollectChest = true
                    _G.IsChestFarming = true
                    return
                end
            end
            
            -- Click detector to spawn boss with Microchip
            local detector, buttonPosition = findClickDetector()
            if detector then
                -- Make sure player is close enough to button
                if buttonPosition and not isPlayerCloseToButton(buttonPosition) then
                    teleportToButton(buttonPosition)
                    wait(1)
                end
                
                -- Click to spawn boss
                pcall(function() fireclickdetector(detector) end)
                wait(0.5)
                pcall(function() fireclickdetector(detector) end)
                
                -- Wait to ensure the click is processed
                wait(3)
                
                -- Fight boss if it exists
                if bossExists() then
                    fightBoss()
                    
                    -- Wait for boss fight to complete
                    local waitTime = 0
                    while _G.IsFightingBoss and waitTime < 300 do
                        wait(1)
                        waitTime = waitTime + 1
                    end
                    
                    -- Additional wait to ensure items are collected
                    wait(3)
                    
                    -- Check for Core Brain after boss is defeated
                    if hasCoreBrain() then
                        _G.HasCoreBrain = true
                        _G.NeedCoreBrain = false
                        
                        -- Equip Core Brain
                        equipCoreBrain()
                        
                        -- Click detector to use Core Brain
                        clickDetectorForNotification()
                        wait(3)
                        
                        -- Buy Cyborg race
                        buyCyborgRace()
                        
                        -- Check if successfully purchased
                        wait(2)
                        if isCyborg() then
                            _G.AutoCyborg = false
                            game:GetService("StarterGui"):SetCore("SendNotification", {
                                Title = "Success",
                                Text = "Successfully obtained Cyborg race!",
                                Duration = 10
                            })
                        else
                            -- Try again if failed
                            for i = 1, 3 do
                                clickDetectorForNotification()
                                wait(1)
                                buyCyborgRace()
                                wait(2)
                                
                                if isCyborg() then
                                    _G.AutoCyborg = false
                                    game:GetService("StarterGui"):SetCore("SendNotification", {
                                        Title = "Success",
                                        Text = "Successfully obtained Cyborg race! (attempt: " .. i .. ")",
                                        Duration = 10
                                    })
                                    break
                                end
                            end
                        end
                    else
                        -- If no Core Brain after boss fight, resume chest farming
                        game:GetService("StarterGui"):SetCore("SendNotification", {
                            Title = "Core Brain",
                            Text = "Core Brain not found. Resuming chest farming...",
                            Duration = 5
                        })
                        _G.AutoCollectChest = true
                        _G.IsChestFarming = true
                    end
                else
                    -- If boss doesn't exist after clicking, resume chest farming
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Error",
                        Text = "Couldn't spawn boss! Resuming chest farming...",
                        Duration = 5
                    })
                    _G.AutoCollectChest = true
                    _G.IsChestFarming = true
                end
            end
        end
        return
    end
    
    -- If we don't have Fist of Darkness or Core Brain, enable auto chest farming
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Auto Chest",
        Text = "Enabling auto chest farming to find Fist of Darkness...",
        Duration = 5
    })
    _G.AutoCollectChest = true
    _G.IsChestFarming = true
end

-- Improved AutoChestCollect function
    GetChest = function()
        local distance = math.huge
        local a
        for r, v in pairs(workspace.Map:GetDescendants()) do
            if string.find(v.Name:lower(), "chest") then
                if v:FindFirstChild("TouchInterest") then
                    if (v.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude < distance then
                        distance = (v.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                        a = v
                    end
                end
            end
        end
        return a
    end
function AutoChestCollect()
    if not _G.ChestFarmingRunning then
        _G.ChestFarmingRunning = true
        
        -- Create variable to track collected chests
        local visitedChests = {}
        
        spawn(function()
            while wait(0.1) do
                -- Check for Core Brain, Cyborg race, and Fist of Darkness before each chest search
                if hasCoreBrain() then
                    _G.AutoCollectChest = false
                    _G.IsChestFarming = false
                    _G.starthop = false
                    _G.AutoHopEnabled = false
                    
                    -- Notification
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Auto Cyborg",
                        Text = "Core Brain found! Stopping chest farm...",
                        Duration = 5
                    })
                    
                    -- Process Core Brain
                    equipCoreBrain()
                    clickDetectorForNotification()
                    wait(5)
                    buyCyborgRace()
                    
                    continue
                elseif isCyborg() then
                    _G.AutoCollectChest = false
                    _G.IsChestFarming = false
                    _G.AutoCyborg = false
                    _G.starthop = false
                    _G.AutoHopEnabled = false
                    
                    -- Notification
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Auto Cyborg",
                        Text = "You already have Cyborg race! Script stopped.",
                        Duration = 10
                    })
                    
                    continue
                elseif isCyborg() then
                    _G.AutoCollectChest = false
                    _G.IsChestFarming = false
                    _G.AutoCyborg = false
                    _G.starthop = false
                    _G.AutoHopEnabled = false
                    
                    -- Notification
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Auto Cyborg",
                        Text = "You already have Cyborg race! Script stopped.",
                        Duration = 10
                    })
                    
                    continue
                elseif hasFistOfDarkness() then
                    _G.AutoCollectChest = false
                    _G.IsChestFarming = false
                    _G.starthop = false
                    _G.AutoHopEnabled = false
                    
                    -- Notification
                    game:GetService("StarterGui"):SetCore("SendNotification", {
                        Title = "Auto Cyborg",
                        Text = "Fist of Darkness found! Processing...",
                        Duration = 5
                    })
                    
                    -- Process Fist of Darkness
                    processFistOfDarkness()
                    
                    continue
                end
                
                -- Kiểm tra nếu chế độ đã bị tắt
                if not _G.AutoCollectChest then 
                    wait(1)
                    continue 
                end
                
                -- Kiểm tra nếu đang đánh boss
                if _G.IsFightingBoss or _G.IsFightingCyborgBoss then
                    continue
                end
                
                -- Rest of the chest collection code...
                -- (giữ nguyên phần code thu thập rương)
                
                -- Kích hoạt NoClip và anti-gravity
                EnableNoClipAndAntiGravity()
                
                pcall(function()
                    local character = game.Players.LocalPlayer.Character
                    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
                    
                    local hrpPosition = character.HumanoidRootPart.Position
                    
                    -- ===== CÂN BẰNG GIỮA GIÁ TRỊ RƯƠNG VÀ KHOẢNG CÁCH =====
                    -- Thu thập rương tốt nhất
                    if GetChest() then
                        Tween2(GetChest().CFrame)
                        pcall(function()
                            if workspace:FindFirstChild("Key") and not _G.KeyDetected then
                                _G.KeyDetected = true
                                _G.AutoJump = false
                            end
                            
                            -- Kiểm tra Fist of Darkness và Core Brain
                            hasFistOfDarkness()
                            hasCoreBrain()
                        end)
                    elseif tick() - _G.LastChestCollectedTime > 60 then
                        HopServer()
                    end
                end)
            end
        end)
    end
end

-- Kiểm tra liên tục việc chống rơi
spawn(function()
    while wait(0.5) do
        pcall(function()
            if _G.AutoCollectChest and not _G.IsFightingBoss and not _G.IsFightingCyborgBoss then
                EnableNoClipAndAntiGravity()
            end
        end)
    end
end)

-- Bắt đầu quá trình thu thập chest
AutoChestCollect()
-- Continuously check player position
spawn(function()
    while wait(1) do -- Check every second
        if _G.AutoHopEnabled and not _G.IsFightingBoss and not _G.IsFightingCyborgBoss then
            pcall(function()
                CheckIfStuckAndHop()
            end)
        end
    end
end)
-- Thêm hàm này gần đầu script, sau phần khai báo biến toàn cục
function ForceStopChestCollection()
    -- Đặt nhiều cờ để đảm bảo dừng hẳn việc thu thập rương
    _G.AutoCollectChest = false
    _G.IsChestFarming = false
    _G.starthop = false
    _G.AutoHopEnabled = false
    
    -- Hủy tween hiện tại
    _G.StopTween = true
    _G.StopTween2 = true
    _G.CancelTween2 = false
    
    -- Thông báo
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Auto Cyborg",
        Text = "Đã dừng thu thập rương để xử lý vật phẩm",
        Duration = 3
    })
    
    -- Đặt lại nhiều lần để đảm bảo
    spawn(function()
        for i = 1, 5 do
            wait(i * 0.2)
            _G.AutoCollectChest = false
            _G.IsChestFarming = false
        end
    end)
end
-- Server hopping function
function HopServer()
    local maxServerSize = 9 -- Limit on players in server to hop to
    local serverFound = false -- Variable to check if server change was successful

    -- Function to find and join new server
    local function findAndJoinNewServer()
        local serverBrowserService = game:GetService("ReplicatedStorage").__ServerBrowser
        for i = 1, math.huge do
            local availableServers = serverBrowserService:InvokeServer(i)
            for jobId, serverInfo in pairs(availableServers) do
                if jobId ~= game.JobId and serverInfo["Count"] < maxServerSize then
                    serverBrowserService:InvokeServer("teleport", jobId)
                    serverFound = true
                    return true
                end
            end
        end
        return false
    end

    -- Try to switch to new server
    pcall(function()
        while not serverFound do
            findAndJoinNewServer()
            wait(0.4) -- Wait a short period before trying again
        end
    end)
end

-- Auto-jump function (press E to save status)
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.E then
        -- Write data to file
        local data = {
            AutoCyborg = _G.AutoCyborg,
            HasMicrochip = hasMicrochip(),
            HasCoreBrain = hasCoreBrain(),
            HasCyborgRace = isCyborg(),
            AutoJump = _G.AutoJump,
            HasFistOfDarkness = hasFistOfDarkness()
        }
        
        -- Convert to string format
        local dataString = "AutoCyborg: " .. tostring(data.AutoCyborg) .. "\n" ..
                          "HasMicrochip: " .. tostring(data.HasMicrochip) .. "\n" ..
                          "HasCoreBrain: " .. tostring(data.HasCoreBrain) .. "\n" ..
                          "HasCyborgRace: " .. tostring(data.HasCyborgRace) .. "\n" ..
                          "AutoJump: " .. tostring(data.AutoJump) .. "\n" ..
                          "HasFistOfDarkness: " .. tostring(data.HasFistOfDarkness)
        
        -- Save to new file
        writefile("CyborgStatus.txt", dataString)
    end
end)

-- Anti-AFK function to make character jump
function AutoJump()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")

    while wait(math.random(15, 20)) do  -- Wait random time
        pcall(function()
            if _G.AutoJump and humanoid and humanoid.Health > 0 then  -- Check if auto jump is enabled and character is alive
                -- Double-check for Fist of Darkness before jumping
                if hasFistOfDarkness() and not _G.FistDetected then
                    _G.FistDetected = true
                    _G.HasFistOfDarkness = true
                    _G.AutoJump = false 

                    -- Click detector multiple times
                    clickDetectorForNotification()
                    wait(1)
                    clickDetectorForNotification()
                elseif _G.AutoJump then
                    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)  -- Activate jump state
                end
            end
        end)
    end
end

-- Start auto jump in separate coroutine
spawn(AutoJump)

-- Anti-kick function to prevent being kicked after 20 minutes idle
local function AntiKick()
    while true do
        wait(1)
        pcall(function()
            if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local v1518 = Instance.new("BillboardGui", game.Players.LocalPlayer.Character.HumanoidRootPart);
                v1518.Name = "Esp";
                v1518.ExtentsOffset = Vector3.new(0, 1, 0);
                v1518.Size = UDim2.new(1, 300, 1, 50);
                v1518.Adornee = game.Players.LocalPlayer.Character.HumanoidRootPart;
                v1518.AlwaysOnTop = true;
                local v1524 = Instance.new("TextLabel", v1518);
                v1524.Font = "Code";
                v1524.FontSize = "Size14";
                v1524.TextWrapped = true;
                v1524.Size = UDim2.new(1, 0, 1, 0);
                v1524.TextYAlignment = "Top";
                v1524.BackgroundTransparency = 1;
                v1524.TextStrokeTransparency = 0.5;
                v1524.TextColor3 = Color3.fromRGB(80, 245, 245);
                v1524.Text = "taphoamizu";
            end
            if game.Players.LocalPlayer.Character.HumanoidRootPart.Velocity.Magnitude < 0.1 then
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(0, 0, 0.01)
            end
        end)
    end
end

-- Call AntiKick function
spawn(AntiKick)

-- Automatic rejoin on kick
spawn(function()
    while wait() do
        if _G.AutoRejoin == true then
            getgenv().rejoin = game:GetService("CoreGui").RobloxPromptGui.promptOverlay.ChildAdded:Connect(function(child)
                if child.Name == 'ErrorPrompt' and child:FindFirstChild('MessageArea') and child.MessageArea:FindFirstChild("ErrorFrame") then
                    game:GetService("TeleportService"):Teleport(game.PlaceId)
                end
            end)
        end
    end
end)

-- Check for special items (Fist of Darkness, Core Brain)
local function CheckForSpecialItems()
    local startTime = tick()  -- Save start check time
    local timeLimit = 15 * 60 -- Time limit is 15 minutes

    spawn(function()
        while true do
            wait(1)  -- Check every second
            local currentTime = tick()
            local player = game.Players.LocalPlayer
            
            -- Check specifically for Core Brain first
            if hasCoreBrain() then
                -- Immediately disable auto chest when Core Brain is found
                _G.AutoCollectChest = false
                _G.IsChestFarming = false
                _G.HasCoreBrain = true
                _G.NeedCoreBrain = false
                _G.AutoJump = false
                
               
                
                -- Equip Core Brain
                equipCoreBrain()
                
                -- Click detector for notification
                clickDetectorForNotification()
                wait(5)  -- Wait 5 seconds
                
                -- Buy Cyborg race
                buyCyborgRace()
                
                startTime = tick()
            -- Then check for Fist of Darkness
            elseif hasFistOfDarkness() and not _G.FistDetected then
                -- Disable auto chest when Fist of Darkness is found
                _G.FistDetected = true
                _G.HasFistOfDarkness = true
                _G.AutoJump = false

                -- Click detector multiple times
                clickDetectorForNotification()
                wait(1)
                clickDetectorForNotification()
                
                startTime = tick()
            elseif (currentTime - startTime) > timeLimit then
                -- If over 15 minutes without finding item, change server
                HopServer()
                startTime = tick()  -- Reset time after server change
            end
            
            -- Check for key and stop auto jump if found
            if workspace:FindFirstChild("Key") and not _G.KeyDetected then
                _G.KeyDetected = true
                _G.AutoJump = false
                -- Click detector for notification multiple times
                clickDetectorForNotification()
                wait(1)
                clickDetectorForNotification()
            end
            
            -- Deep check player inventory for Fist of Darkness
            pcall(function()
                local backpack = player.Backpack:GetChildren()
                for _, item in pairs(backpack) do
                    if item.Name == "Fist of Darkness" and not _G.FistDetected then
                        _G.FistDetected = true
                        _G.HasFistOfDarkness = true
                        _G.AutoJump = false
                        -- Click detector multiple times
                        clickDetectorForNotification()
                        wait(1)
                        clickDetectorForNotification()
                    end
                end
                
                if player.Character then
                    local character = player.Character:GetChildren()
                    for _, item in pairs(character) do
                        if item.Name == "Fist of Darkness" and not _G.FistDetected then
                            _G.FistDetected = true
                            _G.HasFistOfDarkness = true
                            _G.AutoJump = false
                            -- Click detector multiple times
                            clickDetectorForNotification()
                            wait(1)
                            clickDetectorForNotification()
                        end
                    end
                end
            end)
        end
    end)
end

-- Call special item check function
CheckForSpecialItems()

-- Enhanced Core Brain detection
function setupCoreBrainDetection()
    -- Setup chat monitoring
    setupChatMonitoring()
    
    -- Function to teleport to Core Brain location
    local function teleportToCoreBrain()
        pcall(function()
            _G.NeedCoreBrain = true
            
            -- Stop chest farming and other activities
            _G.AutoCollectChest = false
            _G.IsChestFarming = false
            _G.AutoJump = false
            
            -- Teleport to Core Brain position
            if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = coreBrainPosition
            end
            
            -- Wait a bit after teleporting
            wait(2)
            
            -- Check if we now have Core Brain
            if hasCoreBrain() then
                _G.HasCoreBrain = true
                
                -- Equip Core Brain
                equipCoreBrain()
                
                -- Click detector for notification
                clickDetectorForNotification()
                wait(5)  -- Wait 5 seconds
                
                -- Buy Cyborg race
                buyCyborgRace()
            else
                -- If still no Core Brain, continue process
                if bossExists() then
                    fightBoss()
                else
                    clickToSpawnBoss()
                end
            end
        end)
    end
    
    -- Monitor GUI
    pcall(function()
        if game.Players.LocalPlayer.PlayerGui then
            -- Check current GUI
            for _, gui in pairs(game.Players.LocalPlayer.PlayerGui:GetDescendants()) do
                checkGUI(gui)
            end
            
            -- Monitor new GUI
            game.Players.LocalPlayer.PlayerGui.DescendantAdded:Connect(function(descendant)
                if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
                    checkGUI(descendant)
                    
                    pcall(function()
                        descendant:GetPropertyChangedSignal("Text"):Connect(function()
                            checkGUI(descendant)
                        end)
                    
                        descendant:GetPropertyChangedSignal("Visible"):Connect(function()
                            checkGUI(descendant)
                        end)
                    end)
                end
            end)
        end
    end)
    
    -- Monitor workspace for key
    pcall(function()
        workspace.ChildAdded:Connect(function(child)
            if child.Name == "Key" then
                -- Key was added to workspace, disable auto jump
                _G.KeyDetected = true
                _G.AutoJump = false

                clickDetectorForNotification()
                wait(1)
                clickDetectorForNotification()
            end
        end)
    end)
    
    -- Monitor inventory for Fist of Darkness
    local player = game:GetService("Players").LocalPlayer
    
    pcall(function()
        player.Backpack.ChildAdded:Connect(function(item)
            if item.Name == "Fist of Darkness" and not _G.FistDetected then
                _G.FistDetected = true
                _G.HasFistOfDarkness = true
                _G.AutoJump = false

                -- Click detector multiple times
                clickDetectorForNotification()
                wait(1)
                clickDetectorForNotification()
            end
        end)
    end)
    
    pcall(function()
        if player.Character then
            player.Character.ChildAdded:Connect(function(item)
                if item.Name == "Fist of Darkness" and not _G.FistDetected then
                    _G.FistDetected = true
                    _G.HasFistOfDarkness = true
                    _G.AutoJump = false

                    -- Click detector multiple times
                    clickDetectorForNotification()
                    wait(1)
                    clickDetectorForNotification()
                end
            end)
        end
    end)
    
    player.CharacterAdded:Connect(function(char)
        pcall(function()
            char.ChildAdded:Connect(function(item)
                if item.Name == "Fist of Darkness" and not _G.FistDetected then
                    _G.FistDetected = true
                    _G.HasFistOfDarkness = true
                    _G.AutoJump = false
                    -- Click detector multiple times
                    clickDetectorForNotification()
                    wait(1)
                    clickDetectorForNotification()
                end
            end)
        end)
    end)
end

-- Character death and respawn handling
pcall(function()
    game.Players.LocalPlayer.CharacterAdded:Connect(function(newCharacter)
        -- If we were fighting boss, this is likely a death
        if _G.IsFightingBoss then
            -- Wait a bit for character to fully load
            wait(3)
            
            -- Resume boss fight if boss still exists
            if bossExists() then
                fightBoss()
            else
                _G.IsFightingBoss = false
                
                -- Check if we need to continue with process
                if not isCyborg() and _G.AutoCyborg then
                    if hasCoreBrain() then
                        _G.HasCoreBrain = true
                        _G.NeedCoreBrain = false
                        _G.AutoJump = false
                        
                        -- Equip Core Brain
                        equipCoreBrain()
                        
                        -- Click detector for notification
                        clickDetectorForNotification()
                        wait(5)  -- Wait 5 seconds
                        
                        -- Buy Cyborg race
                        buyCyborgRace()
                    else
                        -- Continue process
                        checkForCoreBrain()
                    end
                end
            end
        end
        
        -- Also check for Fist of Darkness on respawn
        wait(1) -- Wait for items to load
        if hasFistOfDarkness() and not _G.FistDetected then
            _G.FistDetected = true
            _G.HasFistOfDarkness = true
            _G.AutoJump = false
            -- Click detector multiple times
            clickDetectorForNotification()
            wait(1)
            clickDetectorForNotification()
        end
    end)
end)

-- Race change detection
pcall(function()
    game.Players.LocalPlayer.Data.Race.Changed:Connect(function()
        if game.Players.LocalPlayer.Data.Race.Value == "Cyborg" then
            _G.AutoCyborg = false
            _G.AutoCollectChest = false
            _G.IsChestFarming = false
            _G.IsFightingBoss = false
            _G.AutoJump = false
            
            game:GetService("StarterGui"):SetCore("SendNotification", {
                Title = "Auto Cyborg",
                Text = "Successfully obtained Cyborg race!",
                Duration = 10
            })
        end
    end)
end)

-- Add ability activation for auto T, Y, and Ken
spawn(function()
    while wait() do
        if _G.IsFightingBoss or _G.IsFightingCyborgBoss then
            pcall(function()
                -- Auto Ken Haki
                game:GetService("ReplicatedStorage").Remotes.CommE:FireServer("Ken", true)
                
                -- Auto T Ability
                game:GetService("ReplicatedStorage").Remotes.CommE:FireServer("ActivateAbility")
                
                -- Auto Y Ability
                game:GetService("VirtualInputManager"):SendKeyEvent(true, "Y", false, game)
                wait()
                game:GetService("VirtualInputManager"):SendKeyEvent(false, "Y", false, game)
                
                -- V3/V4 Activation
                -- Check for V4
                if game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AwakeningChanger", "Check") == true then
                    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("AwakeningChanger", "Awaken")
                end
                
                -- Check for V3
                for i, v in pairs(game.Players.LocalPlayer.Backpack:GetChildren()) do
                    if v:IsA("Tool") and v.ToolTip == "Melee" then
                        if game.Players.LocalPlayer.Backpack:FindFirstChild(v.Name) then
                            local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(v.Name)
                            game.Players.LocalPlayer.Character.Humanoid:EquipTool(tool)
                        end
                    end
                end
                
                if not game:GetService("Players").LocalPlayer.Character:FindFirstChild("RaceTransformed") then
                    local RightClick = game:GetService("VirtualInputManager")
                    RightClick:SendKeyEvent(true, "E", false, game)
                    wait(0.1)
                    RightClick:SendKeyEvent(false, "E", false, game)
                end
            end)
        end
    end
end)

-- Stun prevention
task.spawn(function()
    while wait() do
        pcall(function()
            if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Stun") then
                game.Players.LocalPlayer.Character.Stun.Value = 0
            end
        end)
    end
end)

-- Additional NoClip for boss fights
spawn(function()
    while true do
        wait()
        if _G.IsFightingBoss or _G.IsFightingCyborgBoss then
            pcall(function()
                if game.Players.LocalPlayer.Character then
                    for _, part in pairs(game.Players.LocalPlayer.Character:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.CanCollide = false
                        end
                    end
                    
                    if game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
                        game.Players.LocalPlayer.Character.Humanoid:ChangeState(11) -- GettingUp state (more stable than Flying)
                        game.Players.LocalPlayer.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                        game.Players.LocalPlayer.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                    end
                end
            end)
        end
    end
end)

-- Smart Server Hopper (finds less crowded servers for better chest farming)
function SmartServerHop()
    if not _G.AutoHopEnabled then return end
    
    pcall(function()
        local servers = {}
        local req = game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100")
        local data = game:GetService("HttpService"):JSONDecode(req)
        
        for i,v in pairs(data.data) do
            if v.playing < v.maxPlayers and v.id ~= game.JobId then
                table.insert(servers, v.id)
            end
        end
        
        if #servers > 0 then
            game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)])
        else
            wait(30)
            SmartServerHop()
        end
    end)
end

-- Replace the original HopServer function with the smarter version if available
if pcall(function() game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100") end) then
    HopServer = SmartServerHop
end

-- Start the auto chest collect function
AutoChestCollect()

-- Setup Core Brain detection
setupCoreBrainDetection()

-- Initialize by checking for features needed
spawn(function()
    -- Start the main cycle after 10 seconds (allow game to fully load)
    wait(10)
    
    -- Check if already Cyborg
    if isCyborg() then
        _G.AutoCyborg = false
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "Auto Cyborg",
            Text = "You already have Cyborg race!",
            Duration = 10
        })
        return
    end
    
    -- Start the main process
    mainCycle()
    
    -- Print initial status
    if _G.AutoCollectChest then
        pcall(function()
            local collectionService = game:GetService("CollectionService")
            local chests = collectionService:GetTagged("_ChestTagged")
            if #chests > 0 then
            else
            end
        end)
    end
    
    -- Check for Fist of Darkness on startup
    if hasFistOfDarkness() then
        _G.FistDetected = true
        _G.HasFistOfDarkness = true
        _G.AutoJump = false

        -- Click detector multiple times
        clickDetectorForNotification()
        wait(1)
        clickDetectorForNotification()
    end
end)

-- Show notification that script is running
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "Auto Cyborg Script",
    Text = "Script started successfully!",
    Duration = 10
})

-- Hàm kiểm tra Boss Darkbeard (cải tiến)
function CheckBossAttack()
    -- Kiểm tra trong ReplicatedStorage
    local replicated = game:GetService("ReplicatedStorage"):FindFirstChild("Darkbeard")
    if replicated then return replicated end
    
    -- Kiểm tra trong workspace.Enemies
    local enemies = workspace:FindFirstChild("Enemies")
    if enemies then
        local boss = enemies:FindFirstChild("Darkbeard")
        if boss then return boss end
    end
    
    -- Kiểm tra trong workspace
    local workspaceBoss = workspace:FindFirstChild("Darkbeard")
    if workspaceBoss then return workspaceBoss end
    
    return nil
end

-- Hàm kiểm tra phần HumanoidRootPart của boss (cải tiến)
function DetectingPart(boss)
    if not boss then return false end
    
    -- Kiểm tra nếu boss có HumanoidRootPart
    if not boss:FindFirstChild("HumanoidRootPart") then return false end
    
    -- Kiểm tra Humanoid
    local humanoid = boss:FindFirstChild("Humanoid")
    if not humanoid then 
        -- Cố gắng tìm humanoid với tên khác
        for _, child in pairs(boss:GetChildren()) do
            if child:IsA("Humanoid") then
                humanoid = child
                break
            end
        end
        
        if not humanoid then return false end
    end
    
    -- Kiểm tra nếu boss còn sống
    if humanoid.Health <= 0 then return false end
    
    return true
end

-- Hàm kiểm tra Darkbeard và trả về nếu tìm thấy (cải tiến)
function checkDarkbeard()
    local boss = CheckBossAttack()
    if boss and DetectingPart(boss) then
        return boss
    end
    return nil
end

-- Kiểm tra xem người chơi có Fist of Darkness không
function HasFistOfDarkness()
    local player = game.Players.LocalPlayer
    return player.Backpack:FindFirstChild("Fist of Darkness") or player.Character:FindFirstChild("Fist of Darkness")
end

-- Hàm kiểm tra nếu có chìa khóa (God's Chalice)
function HasGodsChalice()
    local player = game.Players.LocalPlayer
    return player.Backpack:FindFirstChild("God's Chalice") or player.Character:FindFirstChild("God's Chalice")
end

-- Hàm bay đến vị trí của boss và đánh (đã chỉnh sửa để tương thích với tệp đầu tiên)
function FightDarkbeard()
    local boss = checkDarkbeard()
    
    if boss then
        _G.IsFightingBoss = true
        _G.AutoCollectChest = false
        _G.AutoHopEnabled = false
        
        -- Bắt đầu NoClip
        startNoClip()
        
        -- Kích hoạt Haki và trang bị vũ khí
        AutoHaki()
        equip("Melee")
        
        spawn(function()
            local attackCooldown = 0
            
            while _G.IsFightingBoss and boss do
                -- Kích hoạt lại Haki định kỳ
                AutoHaki()
                
                if tick() - attackCooldown > 2 then
                    equip("Melee")
                    attackCooldown = tick()
                end
                
                -- Tìm boss
                local bossUpdated = checkDarkbeard()
                
                if bossUpdated and bossUpdated:FindFirstChild("HumanoidRootPart") and 
                   bossUpdated:FindFirstChild("Humanoid") and bossUpdated.Humanoid.Health > 0 then
                    -- Kiểm tra sức khỏe người chơi - nếu dưới 2000, bay cao hơn boss
                    local player = game:GetService("Players").LocalPlayer
                    local character = player.Character
                    local humanoid = character and character:FindFirstChild("Humanoid")
                    
                    if humanoid and humanoid.Health < 2000 then
                        -- Vị trí boss
                        local bossPosition = bossUpdated.HumanoidRootPart.Position
                        -- Bay cao hơn (100 đơn vị trên boss)
                        local higherPos = CFrame.new(
                            bossPosition.X, 
                            bossPosition.Y + 100, 
                            bossPosition.Z
                        )
                        
                        -- Bay đến vị trí cao hơn
                        Tween(higherPos)
                        
                        -- Đợi đến khi hồi phục sức khỏe lên 5000
                        while humanoid.Health < 5000 do
                            -- Liên tục cập nhật vị trí để ở trên boss
                            local currentBoss = checkDarkbeard()
                            if currentBoss and currentBoss:FindFirstChild("HumanoidRootPart") then
                                local currentBossPos = currentBoss.HumanoidRootPart.Position
                                local newHigherPos = CFrame.new(
                                    currentBossPos.X,
                                    currentBossPos.Y + 100,
                                    currentBossPos.Z
                                )
                                Tween(newHigherPos)
                            end
                            wait(0.5)
                        end
                    else
                        -- Di chuyển đến vị trí boss (hơi cao hơn boss)
                        local bossPosition = bossUpdated.HumanoidRootPart.Position
                        local targetPos = CFrame.new(
                            bossPosition.X, 
                            bossPosition.Y + 25, 
                            bossPosition.Z
                        )
                        
                        -- Bay đến vị trí boss
                        Tween(targetPos)
                        
                        -- Quay mặt về phía boss
                        pcall(function()
                            if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(
                                    game.Players.LocalPlayer.Character.HumanoidRootPart.Position,
                                    Vector3.new(bossUpdated.HumanoidRootPart.Position.X, game.Players.LocalPlayer.Character.HumanoidRootPart.Position.Y, bossUpdated.HumanoidRootPart.Position.Z)
                                )
                            end
                        end)
                        
                        -- Tấn công
                        game:GetService("VirtualUser"):CaptureController()
                        game:GetService("VirtualUser"):ClickButton1(Vector2.new(0, 0))
                    end
                else
                    -- Kiểm tra xem boss đã bị đánh bại chưa
                    _G.IsFightingBoss = false
                    
                    -- Kiểm tra các vật phẩm đặc biệt sau khi đánh boss
                    if HasFistOfDarkness() then
                        -- Thông báo đã có Fist of Darkness
                        game:GetService("StarterGui"):SetCore("SendNotification", {
                            Title = "Đã nhận Fist of Darkness",
                            Text = "Tiếp tục săn boss hoặc tìm kiếm vật phẩm khác",
                            Duration = 5
                        })
                    else
                        -- Bật lại AutoCollectChest nếu không có vật phẩm đặc biệt
                        _G.AutoCollectChest = true
                        _G.AutoHopEnabled = true
                    end
                    
                    break
                end
                
                wait(0.1)
            end
        end)
    end
end

-- Hàm đi đến cổng Darkbeard khi có Fist of Darkness
function GoToDarkbeardGate()
    local darkbeardGate = CFrame.new(3779.50708, 15.0840397, -3500.45386, -0.998627782, 7.57007683e-08, 0.0523698553, 7.95809925e-08, 1, 7.20074809e-08, -0.0523698553, 7.60763115e-08, -0.998627782)
    
    -- Tắt tạm thời auto chest
    _G.AutoCollectChest = false
    
    -- Bay đến cổng
    SafeTween(darkbeardGate, 350)
    wait(1)  -- Đợi một chút ở cổng
    
    -- Kiểm tra xem boss đã xuất hiện chưa
    if checkDarkbeard() then
        FightDarkbeard()
    else
        -- Nếu không có boss, click detector nếu có
        local detector, buttonPosition = findClickDetector()
        if detector then
            if buttonPosition and not isPlayerCloseToButton(buttonPosition) then
                teleportToButton(buttonPosition)
                wait(1)
            end
            
            pcall(function() fireclickdetector(detector) end)
            wait(0.5)
            pcall(function() fireclickdetector(detector) end)
            
            wait(3)
            
            -- Kiểm tra lại xem boss đã xuất hiện chưa
            if checkDarkbeard() then
                FightDarkbeard()
            else
                -- Bật lại auto chest nếu không có boss
                _G.AutoCollectChest = true
            end
        else
            -- Bật lại auto chest nếu không tìm thấy detector
            _G.AutoCollectChest = true
        end
    end
end

-- Function Auto NoClip (bổ sung cho tập tin đầu tiên)
function startNoClip()
    pcall(function()
        for _, part in pairs(game.Players.LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        
        -- Anti-gravity để tránh rơi
        if game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            if not game.Players.LocalPlayer.Character.HumanoidRootPart:FindFirstChild("AntiGravity") then
                local ag = Instance.new("BodyVelocity")
                ag.Name = "AntiGravity"
                ag.MaxForce = Vector3.new(0, 9999, 0)
                ag.Velocity = Vector3.new(0, 0.1, 0)
                ag.Parent = game.Players.LocalPlayer.Character.HumanoidRootPart
            end
        end
    end)
end

-- Coroutine kiểm tra Darkbeard và đánh nếu cần
spawn(function()
    while wait(1) do
        if _G.AutoFightDarkbeard then  -- Luôn đánh Darkbeard (Chế độ 2)
            local boss = checkDarkbeard()
            if boss and not _G.IsFightingBoss then
                FightDarkbeard()
            end
        elseif _G.FightDarkbeardOnlyWithFist and HasFistOfDarkness() then  -- Chỉ đánh với Fist (Chế độ 1)
            local boss = checkDarkbeard()
            if boss and not _G.IsFightingBoss then
                FightDarkbeard()
            elseif not _G.IsFightingBoss then
                -- Nếu có Fist of Darkness nhưng không có boss, bay đến cổng
                GoToDarkbeardGate()
                wait(5)  -- Đợi một lúc ở cổng trước khi kiểm tra lại
            end
        end
    end
end)