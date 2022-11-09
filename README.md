print("HavenRBX Is a qt")

game:GetService("StarterGui"):SetCore("SendNotification",{     

Title = "เปิดuiไอสัส",     

Text = "CTRL ขวา",


Duration = 5

})

if not syn or not protectgui then
    getgenv().protectgui = function()end
end
local Library = loadstring(game:HttpGet('https://lindseyhost.com/UI/LinoriaLib.lua'))()
if not game:IsLoaded() then 
    game.Loaded:Wait()
end

local Camera = workspace.CurrentCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local GuiService = game:GetService("GuiService")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local GetChildren = game.GetChildren
local WorldToScreen = Camera.WorldToScreenPoint
local WorldToViewportPoint = Camera.WorldToViewportPoint
local GetPartsObscuringTarget = Camera.GetPartsObscuringTarget
local FindFirstChild = game.FindFirstChild
local RenderStepped = RunService.RenderStepped
local GuiInset = GuiService.GetGuiInset

local resume = coroutine.resume 
local create = coroutine.create

local ValidTargetParts = {"Head", "HumanoidRootPart"};

local function getPositionOnScreen(Vector)
    local Vec3, OnScreen = WorldToScreen(Camera, Vector)
    return Vector2.new(Vec3.X, Vec3.Y), OnScreen
end

local function ValidateArguments(Args, RayMethod)
    local Matches = 0
    if #Args < RayMethod.ArgCountRequired then
        return false
    end
    for Pos, Argument in next, Args do
        if typeof(Argument) == RayMethod.Args[Pos] then
            Matches = Matches + 1
        end
    end
    return Matches >= RayMethod.ArgCountRequired
end

local function getDirection(Origin, Position)
    return (Position - Origin).Unit * 1000
end

local function getMousePosition()
    return Vector2.new(Mouse.X, Mouse.Y)
end

local function IsPlayerVisible(Player)
    local PlayerCharacter = Player.Character
    local LocalPlayerCharacter = LocalPlayer.Character
    
    if not (PlayerCharacter or LocalPlayerCharacter) then return end 
    
    local PlayerRoot = FindFirstChild(PlayerCharacter, Options.TargetPart.Value) or FindFirstChild(PlayerCharacter, "HumanoidRootPart")
    
    if not PlayerRoot then return end 
    
    local CastPoints, IgnoreList = {PlayerRoot.Position, LocalPlayerCharacter, PlayerCharacter}, {LocalPlayerCharacter, PlayerCharacter}
    local ObscuringObjects = #GetPartsObscuringTarget(Camera, CastPoints, IgnoreList)
    
    return ((ObscuringObjects == 0 and true) or (ObscuringObjects > 0 and false))
end

local function getClosestPlayer()
    if not Options.TargetPart.Value then return end
    local Closest
    local DistanceToMouse
    for _, Player in next, GetChildren(Players) do
        if Player == LocalPlayer then continue end
        if Toggles.TeamCheck.Value and Player.Team == LocalPlayer.Team then continue end

        local Character = Player.Character
        if not Character then continue end
        
        if Toggles.VisibleCheck.Value and not IsPlayerVisible(Player) then continue end

        local HumanoidRootPart = FindFirstChild(Character, "HumanoidRootPart")
        local Humanoid = FindFirstChild(Character, "Humanoid")

        if not HumanoidRootPart or not Humanoid or Humanoid and Humanoid.Health <= 0 then continue end

        local ScreenPosition, OnScreen = getPositionOnScreen(HumanoidRootPart.Position)

        if not OnScreen then continue end

        local Distance = (getMousePosition() - ScreenPosition).Magnitude
        if Distance <= (DistanceToMouse or (Toggles.fov_Enabled.Value and Options.Radius.Value) or 2000) then
            Closest = ((Options.TargetPart.Value == "Random" and Character[ValidTargetParts[math.random(1, #ValidTargetParts)]]) or Character[Options.TargetPart.Value])
            DistanceToMouse = Distance
        end
    end
    return Closest
end

local Window = Library:CreateWindow("Diff meayed")

local GeneralTab = Window:AddTab("General")
local MainBOX = GeneralTab:AddLeftTabbox("Main")
do
    local Main = MainBOX:AddTab("Main")
    Main:AddToggle("aim_Enabled", {Text = "Enabled เปิด"})
    Main:AddToggle("TeamCheck", {Text = "Team Check อยู่มารีนด้วยกันไม่ล็อคกันเอง"})
    Main:AddToggle("VisibleCheck", {Text = "Visible Check"})
    Main:AddDropdown("TargetPart", {Text = "Target Part", Default = 1, Values = {
        "HumanoidRootPart"
    }})
    Main:AddDropdown("Method", {Text = "Silent Aim Method", Default = 1, Values = {
        "Mouse.Hit/Target"
    }})
end

local FieldOfViewBOX = GeneralTab:AddLeftTabbox("ล็อคโครตรันฟันโครตเหลือง")
local MiscellaneousBOX = GeneralTab:AddLeftTabbox("Miscellaneous")

local fov_circle = Drawing.new("Circle")
fov_circle.Thickness = 1
fov_circle.NumSides = 100
fov_circle.Radius = 180
fov_circle.Filled = false
fov_circle.Visible = false
fov_circle.ZIndex = 999
fov_circle.Transparency = 1
fov_circle.Color = Color3.fromRGB(32, 32, 32)
    
local mouse_box = Drawing.new("Square")
mouse_box.Visible = true 
mouse_box.ZIndex = 999 
mouse_box.Color = Color3.fromRGB(32, 32, 32)
mouse_box.Thickness = 20 
mouse_box.Size = Vector2.new(20, 20)
mouse_box.Filled = true 

local PredictionAmount = 0.165

do
    local Main = FieldOfViewBOX:AddTab("ล็อคโครตรันฟันโครตเหลือง")
    Main:AddToggle("fov_Enabled", {Text = "Enabled เปิด"})
    Main:AddSlider("Radius", {Text = "Radius", Min = 0, Max = 360, Default = 70, Rounding = 0}):OnChanged(function()
        fov_circle.Radius = Options.Radius.Value
    end)
    Main:AddToggle("Visible", {Text = "Show FOV ปิดได้ล็อคปกติ"}):AddColorPicker("Color", {Default = Color3.fromRGB(32, 32, 32)}):OnChanged(function()
        fov_circle.Visible = Toggles.Visible.Value
    end)
    Main:AddToggle("MousePosition", {Text = "ZSD"}):AddColorPicker("MouseVisualizeColor", {Default = Color3.fromRGB(32, 32, 32)}):OnChanged(function()
        mouse_box.Visible = Toggles.MousePosition.Value 
    end)
    
    local PredictionTab = MiscellaneousBOX:AddTab("กูบอกกูปวดขี้!")
    PredictionTab:AddToggle("Prediction", {Text = ""})
    PredictionTab:AddSlider("Amount", {Text = "อย่ายุ่ง", Min = 0.165, Max = 1, Default = 0.165, Rounding = 3}):OnChanged(function()
        PredictionAmount = Options.Amount.Value
    end)
end

resume(create(function()
    RenderStepped:Connect(function()
        if Toggles.MousePosition.Value then 
            if Toggles.aim_Enabled.Value == true and Options.Method.Value == "Mouse.Hit/Target" then
                mouse_box.Color = Options.MouseVisualizeColor.Value 
                
                mouse_box.Visible = ((getClosestPlayer() and true) or false)
                mouse_box.Position = ((getClosestPlayer() and Vector2.new(WorldToViewportPoint(Camera, getClosestPlayer().Parent.PrimaryPart.Position).X, WorldToViewportPoint(Camera, getClosestPlayer().Parent.PrimaryPart.Position).Y)) or Vector2.new(-9000, -9000)) -- I am too lazy to write this differently - xaxa
            end
        end
        
        if Toggles.Visible.Value then 
            fov_circle.Visible = Toggles.Visible.Value
            fov_circle.Color = Options.Color.Value
            fov_circle.Position = getMousePosition() + Vector2.new(0, 36)
        end
    end)
end))

local ExpectedArguments = {
    FindPartOnRayWithIgnoreList = {
        ArgCountRequired = 3,
        Args = {
            "Instance", "Ray", "table", "boolean", "boolean"
        }
    },
    FindPartOnRayWithWhitelist = {
        ArgCountRequired = 3,
        Args = {
            "Instance", "Ray", "table", "boolean"
        }
    },
    FindPartOnRay = {
        ArgCountRequired = 2,
        Args = {
            "Instance", "Ray", "Instance", "boolean", "boolean"
        }
    },
    Raycast = {
        ArgCountRequired = 3,
        Args = {
            "Instance", "Vector3", "Vector3", "RaycastParams"
        }
    }
}

local oldNamecall
oldNamecall = hookmetamethod(game, "__namecall", function(...)
    local Method = getnamecallmethod()
    local Arguments = {...}
    local self = Arguments[1]

    if Toggles.aim_Enabled.Value and self == workspace then
        if Method == "FindPartOnRayWithIgnoreList" and Options.Method.Value == Method then
            if ValidateArguments(Arguments, ExpectedArguments.FindPartOnRayWithIgnoreList) then
                local A_Ray = Arguments[2]

                local HitPart = getClosestPlayer()
                if HitPart then
                    local Origin = A_Ray.Origin
                    local Direction = getDirection(Origin, HitPart.Position)
                    Arguments[2] = Ray.new(Origin, Direction)

                    return oldNamecall(unpack(Arguments))
                end
            end
        elseif Method == "FindPartOnRayWithWhitelist" and Options.Method.Value == Method then
            if ValidateArguments(Arguments, ExpectedArguments.FindPartOnRayWithWhitelist) then
                local A_Ray = Arguments[2]

                local HitPart = getClosestPlayer()
                if HitPart then
                    local Origin = A_Ray.Origin
                    local Direction = getDirection(Origin, HitPart.Position)
                    Arguments[2] = Ray.new(Origin, Direction)

                    return oldNamecall(unpack(Arguments))
                end
            end
        elseif (Method == "FindPartOnRay" or Method == "findPartOnRay") and Options.Method.Value:lower() == Method:lower() then
            if ValidateArguments(Arguments, ExpectedArguments.FindPartOnRay) then
                local A_Ray = Arguments[2]

                local HitPart = getClosestPlayer()
                if HitPart then
                    local Origin = A_Ray.Origin
                    local Direction = getDirection(Origin, HitPart.Position)
                    Arguments[2] = Ray.new(Origin, Direction)

                    return oldNamecall(unpack(Arguments))
                end
            end
        elseif Method == "Raycast" and Options.Method.Value == Method then
            if ValidateArguments(Arguments, ExpectedArguments.Raycast) then
                local A_Origin = Arguments[2]

                local HitPart = getClosestPlayer()
                if HitPart then
                    Arguments[3] = getDirection(A_Origin, HitPart.Position)

                    return oldNamecall(unpack(Arguments))
                end
            end
        end
    end
    return oldNamecall(...)
end)

local oldIndex = nil 
oldIndex = hookmetamethod(game, "__index", function(self, Index)
    if self == Mouse and (Index == "Hit" or Index == "Target") then 
        if Toggles.aim_Enabled.Value == true and Options.Method.Value == "Mouse.Hit/Target" and getClosestPlayer() then
            local HitPart = getClosestPlayer()

            return ((Index == "Hit" and ((Toggles.Prediction.Value == false and HitPart.CFrame) or (Toggles.Prediction.Value == true and (HitPart.CFrame + (HitPart.Velocity * PredictionAmount))))) or (Index == "Target" and HitPart))
        end
    end

    return oldIndex(self, Index)
end)
local prefix = "."
local theplayer = game.Players.LocalPlayer
local usedweapon = "Revolver"


	local notif = loadstring(game:HttpGet("https://raw.githubusercontent.com/insanedude59/notiflib/main/main"))()

	function findplr(plr)
		local checkyes = string.len(plr)
		for i,v in pairs(game.Players:GetPlayers()) do
			local flitering = string.sub(string.lower(v.Name), 1, checkyes)
			if v.ClassName == "Player" and flitering == string.lower(plr) then
				return v
			end
		end
	end

	function plrfromdisplay(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		local foundyes = false
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		for i,v in pairs(game.Players:GetPlayers()) do
			if v.DisplayName == datas[2] then
				foundyes = true
				notif:Notification("Copied Name",v.Name,"GothamSemibold","Gotham", 5)
				setclipboard(v.Name)
			end
		end
		if foundyes == false then
			notif:Notification("Error","Inputted name is invalid!","GothamSemibold","Gotham", 5)	    
		end
	end

	function buything(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		for i,v in pairs(workspace.Ignored.Shop:GetChildren()) do
			if string.match(v.Name, datas[2]) and v:FindFirstChild("Head") then
				theplayer.Character.HumanoidRootPart.CFrame = v.Head.CFrame
				break
			end
		end
	end

	function changeprefix(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		local foundyes = false
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		prefix = datas[2]
		notif:Notification("Updated Prefix","New prefix: " .. prefix,"GothamSemibold","Gotham", 5)	    
	end

	function humchange(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if type(datas[2]) == "string" and type(tonumber(datas[4])) == "number" then
			if findplr(datas[3]) then
				local target = findplr(datas[3]).Name
				game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
				{
					Name = "[".. usedweapon .."]",
					Ammo = workspace.Players[target].Humanoid[datas[2]],
					MaxAmmo = {Value = tonumber(datas[4])}
				}
				)
			end
		end
	end

	function ohyeah(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if findplr(datas[2]) then
			local target = findplr(datas[2]).Name
			game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
			{
				Name = "[".. usedweapon .."]",
				Ammo = workspace.Players[target].Humanoid["HeadScale"],
				MaxAmmo = {Value = 10000000}
			}
			)
		end
	end

	function headless(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if findplr(datas[2]) then
			local target = findplr(datas[2]).Name
			game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
			{
				Name = "[".. usedweapon .."]",
				Ammo = workspace.Players[target].Humanoid["HeadScale"],
				MaxAmmo = {Value = -1}
			}
			)
		end
	end

	function changecash(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if type(tonumber(datas[3])) == "number" then
			if findplr(datas[2]) then
				local target = findplr(datas[2]).Name
				game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
				{
					Name = "[".. usedweapon .."]",
					Ammo = game.Players[target].DataFolder.Currency,
					MaxAmmo = {Value = tonumber(datas[3])}
				}
				)
			end
		end
	end

	function bankset(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if type(tonumber(datas[3])) == "number" then
			if findplr(datas[2]) then
				local target = findplr(datas[2]).Name
				game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
				{
					Name = "[".. usedweapon .."]",
					Ammo = game.Players[target].DataFolder.BankDeposit,
					MaxAmmo = {Value = tonumber(datas[3])}
				}
				)
			end
		end
	end

	function changeleaderwanted(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if type(tonumber(datas[3])) == "number" then
			if findplr(datas[2]) then
				local target = findplr(datas[2]).Name
				game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
				{
					Name = "[".. usedweapon .."]",
					Ammo = game.Players[target].leaderstats.Wanted,
					MaxAmmo = {Value = tonumber(datas[3])}
				}
				)
			end
		end
	end

	function armor(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if type(tonumber(datas[3])) == "number" then
			if findplr(datas[2]) then
				local target = findplr(datas[2]).Name
				game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
				{
					Name = "[".. usedweapon .."]",
					Ammo = workspace.Players[target].BodyEffects.Armor,
					MaxAmmo = {Value = tonumber(datas[3])}
				}
				)
			end
		end
	end

	function disableanticheat()
		local chr = theplayer.Character
		for i,v in pairs(chr:GetChildren()) do
			if #v.Name >= 15 and v.ClassName == "Script" then
				if v:FindFirstChild("LocalScript") then
					v.LocalScript:Destroy()
					notif:Notification("anti-cheat","is gone","GothamSemibold","Gotham", 5)
				end
			end
		end
	end

	function setup(msg)
		local separate = string.sub(msg, 2)
		local datas = {}	
		for data in string.gmatch(separate, "%S+") do
			table.insert(datas, data)
		end
		if type(datas[2]) == "string" then
			usedweapon = datas[2]
			game.ReplicatedStorage:FindFirstChild(".gg/untitledhood"):FireServer("Reload",
			{
				Name = "[".. usedweapon .. "]",
				Ammo = {Value = math.huge*9e9},
				MaxAmmo = {Value = 0}
			}
			)
			notif:Notification("Weapon set to:",usedweapon,"GothamSemibold","Gotham", 5)
		end
	end

	theplayer.Chatted:connect(function(msg) 
		if string.sub(msg, 1, 1) == prefix then
			if string.sub(msg, 2, 4) == "hum"  then
				humchange(msg) 
			elseif string.sub(msg, 2, 3) == "sp"  then
				setup(msg)
			elseif string.sub(msg, 2, 6) == "dname" then
				plrfromdisplay(msg)
			elseif string.sub(msg, 2, 5) == "cash" then
				changecash(msg)
			elseif string.sub(msg, 2, 3) == "ls" then
				changeleaderwanted(msg)
			elseif string.sub(msg, 2, 6) == "armor" then
				armor(msg)
			elseif string.sub(msg, 2, 7) == "prefix" then
				changeprefix(msg)
			elseif string.sub(msg, 2, 5) == "noac" then
				disableanticheat()
			elseif string.sub(msg, 2, 5) == "bank" then
				bankset(msg)
			elseif string.sub(msg, 2, 5) == "nuke" then
				ohyeah(msg)
			elseif string.sub(msg, 2, 9) == "headless" then
				headless(msg)
			elseif string.sub(msg, 2, 4) == "ito" then
				buything(msg)
			end
		end
	end)
