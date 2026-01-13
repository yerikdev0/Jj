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
		Prefix = " !"
	}
}

-- ══════════════════════════════════════
--           Funções Seguras
-- ══════════════════════════════════════

local function SafeJump()
	if Settings.Jump and Char and Char.Character then
		Char:Jump()
	end
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
--            FUNÇÃO CHAVE (ESSENCIAL)
-- ══════════════════════════════════════

local function DoJJ(MethodName, Number, Prefix)
	local success, text = Extenso:Convert(Number)
	if not success then return end

	local Method = Methods[MethodName]
	if not Method then return end

	Method(text, Prefix)
end

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
			DoJJ("Normal", i, cfg.Prefix)

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

-- botão pular
table.insert(Connections,
	UIElements.Circle.MouseButton1Click:Connect(function()
		Toggled = not Toggled
		Settings.Jump = Toggled

		TweenService:Create(UIElements.Circle, TweenInfo.new(0.3), {
			Position = Toggled and UDim2.new(0.772,0,0.5,0) or UDim2.new(0.22,0,0.5,0)
		}):Play()

		TweenService:Create(UIElements.Slide, TweenInfo.new(0.3), {
			BackgroundColor3 = Toggled and Color3.fromRGB(37,150,255) or Color3.fromRGB(20,20,20)
		}):Play()
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
