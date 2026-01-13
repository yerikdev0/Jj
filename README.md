-- ══════════════════════════════════════
--               Core
-- ══════════════════════════════════════

local Find = function(Table)
	for _, Item in pairs(Table or {}) do
		if typeof(Item) == "table" then
			return Item
		end
	end
end

local Options = Find(({...})) or {
	Keybind = "Home",
	Tempo = 0.8889,
	Rainbow = false,
	Language = {
		UI = "pt-br",
		Words = "pt-br"
	},
	Experiments = {}
}

local Version = "2.1"
local Parent = gethui() or game:GetService("CoreGui")

local require = function(Name)
	return loadstring(game:HttpGet(
		"https://raw.githubusercontent.com/Zv-yz/AutoJJs/main/" .. Name .. ".lua"
	))()
end

-- ══════════════════════════════════════
--              Services
-- ══════════════════════════════════════

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LP = Players.LocalPlayer

-- ══════════════════════════════════════
--              Modules
-- ══════════════════════════════════════

local UI = require("UI")
local Notification = require("Notification")
local Extenso = require("Modules/Extenso")
local Character = require("Modules/Character")
local RemoteChat = require("Modules/RemoteChat")

-- ══════════════════════════════════════
--              Constants
-- ══════════════════════════════════════

local Char = Character.new(LP)
local UIElements = UI.UIElements
local Connections = {}

local Threading = nil
local FinishedThread = false
local Toggled = false

local Settings = {
	Keybind = Options.Keybind,
	Started = false,
	Jump = false,
	Config = {
		Start = nil,
		End = nil,
		Prefix = " !" -- PREFIXO CORRETO
	}
}

-- ══════════════════════════════════════
--           Funções Seguras
-- ══════════════════════════════════════

local function SafeJump()
	if not Settings.Jump then return end
	if not Char or not Char.Character then return end
	Char:Jump()
end

-- ══════════════════════════════════════
--              Methods
-- ══════════════════════════════════════

local Methods = {

	Normal = function(Message, Prefix)
		SafeJump()
		RemoteChat:Send(Message .. Prefix)
	end,

	Lowercase = function(Message, Prefix)
		SafeJump()
		RemoteChat:Send(string.lower(Message) .. Prefix)
	end,

	HJ = function(Message, Prefix)
		for i = 1, #Message do
			SafeJump()
			RemoteChat:Send(string.sub(Message, i, i) .. Prefix)
			task.wait(Options.Tempo)
		end
	end
}

-- ══════════════════════════════════════
--              Thread
-- ══════════════════════════════════════

local function EndThread(success)
	if Threading then
		task.cancel(Threading)
		Threading = nil
	end

	FinishedThread = false
	Settings.Started = false
	Notification:Notify(success and 6 or 12)
end

local function StartThread()
	local cfg = Settings.Config
	if not cfg.Start or not cfg.End then return end
	if Threading then EndThread(false) return end

	Notification:Notify(5)

	Threading = task.spawn(function()
		for i = tonumber(cfg.Start), tonumber(cfg.End) do
			local ok, text = Extenso:Convert(i)
			if ok then
				Methods.Normal(text, cfg.Prefix)
			end

			if i ~= tonumber(cfg.End) then
				task.wait(Options.Tempo)
			end
		end
		EndThread(true)
	end)
end

-- ══════════════════════════════════════
--              UI
-- ══════════════════════════════════════

UI:SetVersion(Version)
UI:SetLanguage({})
UI:SetRainbow(Options.Rainbow)
UI:SetParent(Parent)

Notification:SetParent(UI.getUI())

-- botão pular (100% corrigido)
table.insert(Connections,
	UIElements.Circle.MouseButton1Click:Connect(function()
		Toggled = not Toggled
		Settings.Jump = Toggled

		if Toggled then
			TweenService:Create(UIElements.Circle, TweenInfo.new(0.3), {
				Position = UDim2.new(0.772, 0, 0.5, 0)
			}):Play()

			TweenService:Create(UIElements.Slide, TweenInfo.new(0.3), {
				BackgroundColor3 = Color3.fromRGB(37, 150, 255)
			}):Play()
		else
			TweenService:Create(UIElements.Circle, TweenInfo.new(0.3), {
				Position = UDim2.new(0.22, 0, 0.5, 0)
			}):Play()

			TweenService:Create(UIElements.Slide, TweenInfo.new(0.3), {
				BackgroundColor3 = Color3.fromRGB(20, 20, 20)
			}):Play()
		end
	end)
)

-- botão play
table.insert(Connections,
	UIElements.Play.MouseButton1Up:Connect(function()
		if not Settings.Started then
			Settings.Started = true
			StartThread()
		else
			EndThread(false)
		end
	end)
)
