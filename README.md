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

	Language = {
		UI = "pt-br",
		Words = "pt-br"
	},

	Experiments = {},
	Rainbow = false,
}

local Version = "2.2"
local Parent = gethui() or game:GetService("CoreGui")

local require = function(Name)
	return loadstring(game:HttpGet(
		string.format("https://raw.githubusercontent.com/Zv-yz/AutoJJs/main/%s.lua", Name)
	))()
end

-- ══════════════════════════════════════
--              Services
-- ══════════════════════════════════════
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
--        Chat Queue Controller
-- ══════════════════════════════════════
local ChatQueue = {}
local Sending = false

-- ⏱️ TEMPO FIXO ENTRE MENSAGENS
local CHAT_INTERVAL = 0.8889

local function ProcessQueue()
	if Sending then return end
	Sending = true

	task.spawn(function()
		while #ChatQueue > 0 do
			local msg = table.remove(ChatQueue, 1)

			pcall(function()
				RemoteChat:Send(msg)
			end)

			local start = os.clock()
			while os.clock() - start < CHAT_INTERVAL do
				task.wait()
			end
		end
		Sending = false
	end)
end

local function SafeSend(message)
	table.insert(ChatQueue, tostring(message))
	ProcessQueue()
end

-- ══════════════════════════════════════
--              Constants
-- ══════════════════════════════════════
local Char = Character.new(LP)
local UIElements = UI.UIElements
local Connections = {}

local Threading
local FinishedThread = false

local Settings = {
	Started = false,
	Jump = false,

	Config = {
		Start = nil,
		End = nil,
	}
}

-- ══════════════════════════════════════
--              Methods
-- ══════════════════════════════════════
local Methods = {

	["Normal"] = function(Message)
		if Settings.Jump then Char:Jump() end
		SafeSend(string.upper(Message) .. " !")
	end,

	["Lowercase"] = function(Message)
		if Settings.Jump then Char:Jump() end
		SafeSend(string.upper(Message) .. " !")
	end,

	["HJ"] = function(Message)
		if Settings.Jump then Char:Jump() end
		SafeSend(string.upper(Message) .. " !")
	end,
}
-- ══════════════════════════════════════
--              Functions (Corrigidas)
-- ══════════════════════════════════════

local function EndThread()
    if Threading then
        -- Se o thread ainda estiver rodando, cancelamos
        task.cancel(Threading)
        Threading = nil
    end
    -- Resetamos o estado para que o botão "Play" saiba que pode iniciar de novo
    Settings.Started = false
    FinishedThread = false
end

local function StartThread()
    local Config = Settings.Config
    if not Config.Start or not Config.End then 
        Settings.Started = false -- Resetar caso falte config
        return 
    end

    Threading = task.spawn(function()
        for i = Config.Start, Config.End do
            DoJJ("Normal", i)
        end
        
        -- IMPORTANTE: Aguarda as mensagens terminarem de ser enviadas 
        -- antes de permitir um novo início automático
        repeat task.wait() until #ChatQueue == 0
        
        FinishedThread = true
        EndThread() -- Finaliza o estado ao chegar no fim do loop
    end)
end

-- ══════════════════════════════════════
--                Main
-- ══════════════════════════════════════

-- ... (resto do código anterior)

table.insert(Connections, UIElements.Play.MouseButton1Up:Connect(function()
    if not Settings.Config.Start or not Settings.Config.End then 
        return 
    end

    if not Settings.Started then
        -- Inicia a execução
        Settings.Started = true
        StartThread()
    else
        -- Interrompe a execução manual
        EndThread()
    end
end))


-- ══════════════════════════════════════
--                Main
-- ══════════════════════════════════════
UI:SetVersion(Version)
UI:SetRainbow(Options.Rainbow)
UI:SetParent(Parent)

Notification:SetParent(UI.getUI())

for Name, Element in pairs(UIElements.Box) do
	task.spawn(Listen, Name, Element)
end

table.insert(Connections, UIElements.Play.MouseButton1Up:Connect(function()
	if not Settings.Config.Start or not Settings.Config.End then return end

	if not Settings.Started then
		Settings.Started = true
		StartThread()
	else
		EndThread()
	end
end))

if Notification then
	Notification:SetupJJs()
end
