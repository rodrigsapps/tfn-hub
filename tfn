--[[
	████████╗███████╗███╗   ██╗    ██╗  ██╗██╗   ██╗██████╗
	╚══██╔══╝██╔════╝████╗  ██║    ██║  ██║██║   ██║██╔══██╗
	   ██║   █████╗  ██╔██╗ ██║    ███████║██║   ██║██████╔╝
	   ██║   ██╔══╝  ██║╚██╗██║    ██╔══██║██║   ██║██╔══██╗
	   ██║   ██║     ██║ ╚████║    ██║  ██║╚██████╔╝██████╔╝
	   ╚═╝   ╚═╝     ╚═╝  ╚═══╝    ╚═╝  ╚═╝ ╚═════╝ ╚═════╝

	TFN HUB — Steal An Egg / "Roube um Ovo"
	PlaceId 107778070777162  |  UI: WindUI (Footagesus)

	IMPORTANTE — LEIA:
	Este jogo TEM anti-cheat ativo (ObbyAntiTPClient, WalkSpeedGovernor,
	CharacterIntegrity, RF/RigSync/Reconcile). Teleporte por CFrame e noclip
	causam kick com o codigo BAC-10518 ("removed for cheating").
	Por isso o TFN HUB usa MOVIMENTO LEGITIMO por padrao:
	PathfindingService + Humanoid:MoveTo, respeitando a WalkSpeed que o
	proprio jogo concede. O modo "Turbo (RISCO)" existe, mas vem DESLIGADO.
]]

----------------------------------------------------------------------
-- BOOT / SINGLETON
----------------------------------------------------------------------
if _G.TFN_HUB_LOADED and _G.TFN_HUB_DESTROY then
	pcall(_G.TFN_HUB_DESTROY)
end
_G.TFN_HUB_LOADED = true

local function try(f, ...)
	local ok, r = pcall(f, ...)
	if ok then return r end
	return nil
end

----------------------------------------------------------------------
-- SERVICOS
----------------------------------------------------------------------
local cloneref = (cloneref or clonereference or function(i) return i end)

local Players           = cloneref(game:GetService("Players"))
local RunService        = cloneref(game:GetService("RunService"))
local ReplicatedStorage = cloneref(game:GetService("ReplicatedStorage"))
local Workspace         = cloneref(game:GetService("Workspace"))
local CoreGui           = cloneref(game:GetService("CoreGui"))
local Pathfinding       = cloneref(game:GetService("PathfindingService"))
local StarterGui        = cloneref(game:GetService("StarterGui"))

local LP = Players.LocalPlayer

----------------------------------------------------------------------
-- CARREGAR WINDUI
----------------------------------------------------------------------
local WindUI
do
	local sources = {
		"https://github.com/Footagesus/WindUI/releases/latest/download/main.lua",
		"https://raw.githubusercontent.com/Footagesus/WindUI/main/dist/main.lua",
	}
	for _, url in ipairs(sources) do
		local src = try(function() return game:HttpGet(url) end)
		if type(src) == "string" and #src > 5000 then
			local fn = (loadstring or load)(src)
			if fn then
				local ok, lib = pcall(fn)
				if ok and type(lib) == "table" and lib.CreateWindow then
					WindUI = lib
					break
				end
			end
		end
	end
end

if not WindUI then
	try(function()
		StarterGui:SetCore("SendNotification", {
			Title = "TFN HUB",
			Text = "Falha ao baixar a WindUI. Verifique sua internet e tente de novo.",
			Duration = 8,
		})
	end)
	warn("[TFN HUB] WindUI nao carregou.")
	return
end

----------------------------------------------------------------------
-- CONFIG
----------------------------------------------------------------------
local CFG = {
	-- movimento
	SafeMode        = true,   -- Pathfinding/MoveTo (nao dispara anti-cheat)
	TurboMode       = false,  -- CFrame step (RISCO DE BAN) — off por padrao
	TurboSpeed      = 180,
	ReachRadius     = 8,      -- distancia considerada "chegou"
	TravelTimeout   = 45,     -- s por trajeto

	-- roubo
	AutoFarm        = false,
	AutoDeliver     = true,
	StealHoldTries  = 6,
	Delay           = 0.6,

	-- guards
	AvoidGuards     = true,
	GuardRadius     = 30,
	AntiRagdoll     = true,

	-- filtros
	MinMoney        = 0,
	RarityFilter    = "Todas",
	Search          = "",

	-- visual
	ESP             = false,
	ESPDistance     = false,

	-- player
	NoclipRisky     = false,
	InfJump         = false,
}

local RARITY_COLORS = {
	common    = Color3.fromRGB(170, 178, 190),
	uncommon  = Color3.fromRGB(110, 220, 140),
	rare      = Color3.fromRGB( 90, 165, 255),
	epic      = Color3.fromRGB(190, 110, 255),
	legendary = Color3.fromRGB(255, 190,  70),
	mythic    = Color3.fromRGB(255,  95, 120),
	divine    = Color3.fromRGB(120, 255, 235),
	secret    = Color3.fromRGB(255, 255, 255),
	godly     = Color3.fromRGB(255, 140,  40),
	limited   = Color3.fromRGB(255,  80, 200),
	event     = Color3.fromRGB(255, 215,   0),
}
local RARITY_ORDER = {
	"Todas", "Common", "Uncommon", "Rare", "Epic", "Legendary",
	"Mythic", "Divine", "Secret", "Godly", "Limited", "Event",
}

local ACCENT = Color3.fromHex("#8B5CF6")
local GREEN  = Color3.fromHex("#22C55E")
local RED    = Color3.fromHex("#EF4444")
local YELLOW = Color3.fromHex("#F59E0B")
local GREY   = Color3.fromHex("#83889E")

----------------------------------------------------------------------
-- HELPERS DE PERSONAGEM
----------------------------------------------------------------------
local function char() return LP.Character end
local function hrp()
	local c = char()
	return c and (c:FindFirstChild("HumanoidRootPart") or c.PrimaryPart)
end
local function hum()
	local c = char()
	return c and c:FindFirstChildOfClass("Humanoid")
end
local function alive()
	local h = hum()
	return h and h.Health > 0
end

local function comma(n)
	n = tonumber(n) or 0
	local s = string.format("%d", math.floor(n))
	local out = s:reverse():gsub("(%d%d%d)", "%1."):reverse()
	out = out:gsub("^%.", "")
	return out
end

local function short(n)
	n = tonumber(n) or 0
	local units = { {1e12,"T"}, {1e9,"B"}, {1e6,"M"}, {1e3,"K"} }
	for _, u in ipairs(units) do
		if n >= u[1] then return string.format("%.2f%s", n / u[1], u[2]) end
	end
	return comma(n)
end

local function parseNumber(txt)
	if type(txt) ~= "string" then return nil end
	local clean = txt:gsub("%s", "")
	local num, suf = clean:match("([%d%.,]+)%s*([KkMmBbTt]?)")
	if not num then return nil end
	num = num:gsub("%.(%d%d%d)", "%1"):gsub(",", ".")
	local v = tonumber(num)
	if not v then return nil end
	local mult = { k = 1e3, m = 1e6, b = 1e9, t = 1e12 }
	if suf and suf ~= "" then v = v * (mult[suf:lower()] or 1) end
	return v
end

local function notify(title, content, dur, icon)
	try(function()
		WindUI:Notify({
			Title = title or "TFN HUB",
			Content = content or "",
			Icon = icon or "egg",
			Duration = dur or 4,
		})
	end)
end

----------------------------------------------------------------------
-- REMOTES DO JOGO (ReplicatedStorage.Packages.Networking)
----------------------------------------------------------------------
local Net = ReplicatedStorage:FindFirstChild("Packages")
Net = Net and Net:FindFirstChild("Networking")

local R = {
	Carry    = "RF/EggWorld/AskFieldEggCarry",
	Drop     = "RF/EggWorld/AskFieldEggDrop",
	Snapshot = "RF/EggWorld/AskFieldEggSnapshot",
	PlaceEgg = "RF/EggWorld/AskPlaceEgg",
	WearTool = "RF/EggWorld/AskWearTool",
	DoffTool = "RF/EggWorld/AskDoffTool",
	Hatch    = "RF/EggWorld/AskHatch",
	Finish   = "RF/EggWorld/AskFinishHatch",
	SellAll  = "RE/PetSatchel/SellEveryPet",
	BaseUp   = "RE/Homestead/AskBaseTierRaise",
	Nearby   = "RE/Homestead/AskNearbyPurchase",
}

local function remote(name)
	if not Net then return nil end
	return Net:FindFirstChild(name)
end

local function invoke(name, ...)
	local rf = remote(name)
	if not rf then return nil, "remote nao encontrado" end
	if rf:IsA("RemoteFunction") then
		local ok, res = pcall(function(...) return rf:InvokeServer(...) end, ...)
		return ok and res or nil, ok and nil or tostring(res)
	elseif rf:IsA("RemoteEvent") then
		local ok, err = pcall(function(...) rf:FireServer(...) end, ...)
		return ok, ok and nil or tostring(err)
	end
	return nil, "tipo desconhecido"
end

----------------------------------------------------------------------
-- SCAN DE EGGS
----------------------------------------------------------------------
local MONEY_KEYS = {
	"income","money","cash","earnings","profit","persecond",
	"moneypersecond","generation","rate","value","price","worth","payout",
}
local RARITY_KEYS = { "rarity","tier","grade","quality" }

local function keyMatches(key, list)
	local k = tostring(key):lower():gsub("[^%a]", "")
	for _, w in ipairs(list) do
		if k:find(w, 1, true) then return true end
	end
	return false
end

local function readFromAttributes(inst)
	local money, rarity
	local attrs = try(function() return inst:GetAttributes() end)
	if attrs then
		for k, v in pairs(attrs) do
			if not money and type(v) == "number" and keyMatches(k, MONEY_KEYS) then money = v end
			if not rarity and type(v) == "string" and keyMatches(k, RARITY_KEYS) then rarity = v end
			if not money and type(v) == "string" and keyMatches(k, MONEY_KEYS) then money = parseNumber(v) end
		end
	end
	for _, v in ipairs(inst:GetChildren()) do
		if v:IsA("ValueBase") then
			if not money and keyMatches(v.Name, MONEY_KEYS) then
				money = (type(v.Value) == "number") and v.Value or parseNumber(tostring(v.Value))
			end
			if not rarity and keyMatches(v.Name, RARITY_KEYS) then rarity = tostring(v.Value) end
		end
	end
	return money, rarity
end

local function readFromBillboards(root)
	local money, rarity, name
	local descendants = try(function() return root:GetDescendants() end) or {}
	for _, d in ipairs(descendants) do
		if d:IsA("TextLabel") or d:IsA("TextBox") then
			local t = d.Text or ""
			if t ~= "" then
				local low = t:lower()
				if not money and (t:find("%$") or low:find("/s")) then
					money = parseNumber(t)
				end
				if not rarity then
					for key in pairs(RARITY_COLORS) do
						if low:find(key, 1, true) then
							rarity = key:sub(1, 1):upper() .. key:sub(2)
							break
						end
					end
				end
				if not name and #t > 2 and not t:find("%$") and not low:find("/s") and not tonumber(t) then
					local ln = d.Name:lower()
					if ln:find("name") or ln:find("title") or ln:find("egg") then name = t end
				end
			end
		end
	end
	return money, rarity, name
end

local function isCarryPrompt(pr)
	if not pr:IsA("ProximityPrompt") then return false end
	local n = (pr.Name or ""):lower()
	if n:find("carry") or n:find("steal") then return true end
	local a = ((pr.ActionText or "") .. " " .. (pr.ObjectText or "")):lower()
	return a:find("steal") ~= nil or a:find("carry") ~= nil or a:find("pegar") ~= nil or a:find("roub") ~= nil
end

local function eggModelFromPrompt(prompt)
	local p = prompt.Parent
	local node = p
	for _ = 1, 5 do
		if not node or node == Workspace then break end
		if node:IsA("Model") then
			local hasPart = node:FindFirstChildWhichIsA("BasePart", true)
			if hasPart then return node end
		end
		node = node.Parent
	end
	local pos = (p and p:IsA("BasePart")) and p.Position or nil
	if not pos then return nil end
	local eggsFolder = Workspace:FindFirstChild("Eggs")
	if not eggsFolder then return nil end
	local best, bestD = nil, 35
	for _, m in ipairs(eggsFolder:GetChildren()) do
		local pv = try(function() return m:GetPivot().Position end)
		if pv then
			local d = (pv - pos).Magnitude
			if d < bestD then best, bestD = m, d end
		end
	end
	return best
end

local Eggs = {}

local function scanEggs()
	local list, seen = {}, {}

	local function push(model, prompt, part)
		if not model or seen[model] then return end
		seen[model] = true
		local pos = try(function()
			return (part and part.Position) or model:GetPivot().Position
		end)
		if not pos then return end

		local money, rarity = readFromAttributes(model)
		local bMoney, bRarity, bName = readFromBillboards(model)
		money  = money  or bMoney
		rarity = rarity or bRarity

		if (not money or not rarity) and prompt and prompt.Parent then
			local m2, r2 = readFromAttributes(prompt.Parent)
			money  = money  or m2
			rarity = rarity or r2
			local m3, r3 = readFromBillboards(prompt.Parent)
			money  = money  or m3
			rarity = rarity or r3
		end

		local name = model:GetAttribute("EggName") or model:GetAttribute("AssetName")
			or model:GetAttribute("DisplayName") or bName or model.Name

		table.insert(list, {
			model  = model,
			prompt = prompt,
			part   = part,
			pos    = pos,
			name   = tostring(name):gsub("^%s+", ""):gsub("%s+$", ""),
			money  = money or 0,
			rarity = rarity or "Common",
		})
	end

	-- fonte principal: prompts de carry (SmartPromptPart > CarryAreaEgg)
	for _, d in ipairs(Workspace:GetDescendants()) do
		if d:IsA("ProximityPrompt") and isCarryPrompt(d) then
			local part = d.Parent
			local model = eggModelFromPrompt(d)
			push(model or part, d, (part and part:IsA("BasePart")) and part or nil)
		end
	end

	-- fallback: Workspace.Eggs
	local ef = Workspace:FindFirstChild("Eggs")
	if ef then
		for _, m in ipairs(ef:GetChildren()) do
			if m:IsA("Model") or m:IsA("BasePart") then
				local pr
				for _, d in ipairs(m:GetDescendants()) do
					if d:IsA("ProximityPrompt") then pr = d break end
				end
				push(m, pr, m:IsA("BasePart") and m or nil)
			end
		end
	end

	table.sort(list, function(a, b)
		if a.money == b.money then return a.name < b.name end
		return a.money > b.money
	end)

	for i, e in ipairs(list) do e.id = i end
	Eggs = list
	return list
end

local function passesFilter(e)
	if e.money < (CFG.MinMoney or 0) then return false end
	if CFG.RarityFilter ~= "Todas" then
		if (e.rarity or ""):lower() ~= CFG.RarityFilter:lower() then return false end
	end
	if CFG.Search ~= "" then
		if not e.name:lower():find(CFG.Search:lower(), 1, true) then return false end
	end
	return true
end

local function filteredEggs()
	local out = {}
	for _, e in ipairs(Eggs) do
		if passesFilter(e) then table.insert(out, e) end
	end
	return out
end

----------------------------------------------------------------------
-- BASE / PLOT
----------------------------------------------------------------------
local function myPlot()
	local plots = Workspace:FindFirstChild("Plots")
	if not plots then return nil end
	for _, p in ipairs(plots:GetChildren()) do
		local owner = p:GetAttribute("Owner") or p:GetAttribute("OwnerUserId")
			or p:GetAttribute("OwnerId") or p:GetAttribute("Player")
		if owner and (tostring(owner) == tostring(LP.UserId) or tostring(owner) == LP.Name) then
			return p
		end
		local sign = p:FindFirstChild("PlotSign", true)
		if sign then
			for _, d in ipairs(sign:GetDescendants()) do
				if d:IsA("TextLabel") and d.Text
					and (d.Text:find(LP.Name, 1, true) or d.Text:find(LP.DisplayName, 1, true)) then
					return p
				end
			end
		end
	end
	return nil
end

local function basePosition()
	local p = myPlot()
	if p then
		local pt = p:FindFirstChild("CenterPoint") or p:FindFirstChild("SpawnPoint")
		if pt and pt:IsA("BasePart") then return pt.Position end
		local pv = try(function() return p:GetPivot().Position end)
		if pv then return pv end
	end
	local objs = Workspace:FindFirstChild("__OBJECTS")
	local dh = objs and objs:FindFirstChild("DeliveryHitbox")
	if dh and dh:IsA("BasePart") then return dh.Position end
	local areas = objs and objs:FindFirstChild("Areas")
	local sp = areas and areas:FindFirstChild("StartArea")
	if sp and sp:IsA("BasePart") then return sp.Position end
	local spawn = Workspace:FindFirstChild("SpawnLocation", true)
	if spawn and spawn:IsA("BasePart") then return spawn.Position end
	return nil
end

----------------------------------------------------------------------
-- GUARDS
----------------------------------------------------------------------
local function guardModels()
	local out = {}
	local function add(folder)
		if not folder then return end
		local ds = try(function() return folder:GetDescendants() end) or {}
		for _, m in ipairs(ds) do
			if m:IsA("Model") and m ~= char() then
				local root = m:FindFirstChild("HumanoidRootPart") or m.PrimaryPart
				if root then table.insert(out, root) end
			end
		end
	end
	add(Workspace:FindFirstChild("_Guards"))
	local objs = Workspace:FindFirstChild("__OBJECTS")
	add(objs and objs:FindFirstChild("Sentries"))
	add(Workspace:FindFirstChild("Npcs"))
	return out
end

local GuardCache, lastGuardScan = {}, 0
local function guardsNear(pos, radius)
	if tick() - lastGuardScan > 2 then
		lastGuardScan = tick()
		GuardCache = guardModels()
	end
	local near = {}
	for _, root in ipairs(GuardCache) do
		if root.Parent then
			local d = (root.Position - pos).Magnitude
			if d < radius then table.insert(near, { root = root, dist = d }) end
		end
	end
	return near
end

----------------------------------------------------------------------
-- MOVIMENTO
----------------------------------------------------------------------
local Travelling, CancelTravel = false, false
local StatusFn = function() end

-- desvio lateral quando tem guard no caminho
local function dodgeOffset(target)
	local r = hrp()
	if not r then return target end
	local near = guardsNear(r.Position, CFG.GuardRadius)
	if #near == 0 then return target end
	local away = Vector3.new()
	for _, g in ipairs(near) do
		local dir = (r.Position - g.root.Position)
		if dir.Magnitude > 0.1 then
			away = away + dir.Unit * (CFG.GuardRadius - g.dist)
		end
	end
	if away.Magnitude < 0.1 then return target end
	away = Vector3.new(away.X, 0, away.Z)
	if away.Magnitude < 0.1 then return target end
	return target + away.Unit * 14
end

-- MOVIMENTO SEGURO: PathfindingService + Humanoid:MoveTo
local function walkTo(targetPos)
	local h, r = hum(), hrp()
	if not (h and r) then return false end

	local path = Pathfinding:CreatePath({
		AgentRadius        = 3,
		AgentHeight        = 6,
		AgentCanJump       = true,
		AgentCanClimb      = false,
		WaypointSpacing    = 6,
		Costs              = {},
	})

	local ok = pcall(function() path:ComputeAsync(r.Position, targetPos) end)
	local waypoints = (ok and path.Status == Enum.PathStatus.Success) and path:GetWaypoints() or nil

	local t0 = tick()

	if waypoints and #waypoints > 1 then
		for i = 2, #waypoints do
			if CancelTravel or not alive() then return false end
			if tick() - t0 > CFG.TravelTimeout then return false end

			local wp = waypoints[i]
			local goal = wp.Position
			if CFG.AvoidGuards then goal = dodgeOffset(goal) end

			if wp.Action == Enum.PathWaypointAction.Jump then
				h:ChangeState(Enum.HumanoidStateType.Jumping)
			end

			h:MoveTo(goal)
			local reached = h.MoveToFinished:Wait()
			if not reached then
				-- travou: recalcula do ponto atual
				break
			end
		end
	end

	-- aproximacao final direta (cobre falha do pathfinding e o ultimo trecho)
	while true do
		if CancelTravel or not alive() then return false end
		local rr = hrp()
		if not rr then return false end
		local dist = (Vector3.new(rr.Position.X, 0, rr.Position.Z)
			- Vector3.new(targetPos.X, 0, targetPos.Z)).Magnitude
		if dist <= CFG.ReachRadius then return true end
		if tick() - t0 > CFG.TravelTimeout then return false end

		local goal = targetPos
		if CFG.AvoidGuards then goal = dodgeOffset(goal) end

		local hh = hum()
		if not hh then return false end
		hh:MoveTo(goal)
		task.wait(0.35)
	end
end

-- MOVIMENTO TURBO (RISCO DE BAN): passos curtos de CFrame
local function turboTo(targetPos)
	local t0 = tick()
	while true do
		if CancelTravel or not alive() then return false end
		local r = hrp()
		if not r then return false end
		local delta = (targetPos + Vector3.new(0, 3, 0)) - r.Position
		local dist = delta.Magnitude
		if dist <= CFG.ReachRadius then return true end
		if tick() - t0 > CFG.TravelTimeout then return false end
		local step = math.min(CFG.TurboSpeed / 60, dist)
		r.CFrame = CFrame.new(r.Position + delta.Unit * step)
		r.AssemblyLinearVelocity = Vector3.new()
		RunService.Heartbeat:Wait()
	end
end

local function travelTo(pos)
	Travelling = true
	CancelTravel = false
	local ok
	if CFG.TurboMode then
		ok = turboTo(pos)
	else
		ok = walkTo(pos)
	end
	Travelling = false
	return ok
end

----------------------------------------------------------------------
-- ROUBAR / ENTREGAR
----------------------------------------------------------------------
local Busy = false

local function isCarrying()
	local c = char()
	if not c then return false end
	for _, t in ipairs(c:GetChildren()) do
		if t:IsA("Tool") then return true end
	end
	return false
end

local function firePrompt(pr)
	if not pr or not pr.Parent then return false end
	local oldHold = pr.HoldDuration
	local oldDist = pr.MaxActivationDistance
	try(function() pr.HoldDuration = 0 end)
	try(function() pr.MaxActivationDistance = math.max(oldDist, 20) end)
	local fired = false
	if fireproximityprompt then
		try(function() fireproximityprompt(pr, 1) end)
		fired = true
	end
	try(function() pr.HoldDuration = oldHold end)
	try(function() pr.MaxActivationDistance = oldDist end)
	return fired
end

local function grabEgg(egg)
	-- 1) prompt (caminho oficial do jogo — nao levanta flag)
	for _ = 1, CFG.StealHoldTries do
		if isCarrying() then return true end
		if egg.prompt and egg.prompt.Parent then
			firePrompt(egg.prompt)
		else
			-- reprocura o prompt no model
			if egg.model and egg.model.Parent then
				for _, d in ipairs(egg.model:GetDescendants()) do
					if d:IsA("ProximityPrompt") then egg.prompt = d break end
				end
			end
		end
		task.wait(0.3)
		if egg.model and not egg.model.Parent then return true end
	end

	-- 2) fallback: remote (assinatura desconhecida, tenta variacoes)
	if not isCarrying() and egg.model then
		local id = egg.model:GetAttribute("Id") or egg.model:GetAttribute("EggId")
			or egg.model:GetAttribute("Guid") or egg.model:GetAttribute("Uid")
			or egg.model.Name
		invoke(R.Carry, id)
		task.wait(0.2)
		if not isCarrying() then invoke(R.Carry, egg.model) end
		task.wait(0.2)
		if not isCarrying() then invoke(R.Carry) end
	end

	return isCarrying() or (egg.model and not egg.model.Parent) or false
end

local function deliverEgg()
	local base = basePosition()
	if not base then return false, "base nao encontrada" end

	StatusFn("Voltando pra base...", ACCENT)
	local ok = travelTo(base)
	if not ok then return false, "nao chegou na base" end
	task.wait(0.4)

	-- prompts do plot (delivery / place)
	local plot = myPlot()
	if plot then
		for _, d in ipairs(plot:GetDescendants()) do
			if d:IsA("ProximityPrompt") then
				firePrompt(d)
				task.wait(0.15)
				if not isCarrying() then break end
			end
		end
	end

	-- prompts soltos perto da base
	if isCarrying() then
		local r = hrp()
		if r then
			for _, d in ipairs(Workspace:GetDescendants()) do
				if d:IsA("ProximityPrompt") and d.Parent and d.Parent:IsA("BasePart") then
					if (d.Parent.Position - r.Position).Magnitude < 25 then
						firePrompt(d)
						task.wait(0.12)
						if not isCarrying() then break end
					end
				end
			end
		end
	end

	-- ultimo recurso: remote de place
	if isCarrying() then
		local cf = CFrame.new(base + Vector3.new(0, 3, 0))
		invoke(R.PlaceEgg, cf)
		task.wait(0.2)
		if isCarrying() then invoke(R.PlaceEgg, cf, 1) end
	end

	return not isCarrying()
end

local function stealEgg(egg)
	if Busy then
		notify("TFN HUB", "Ja tem uma acao em andamento.", 3, "clock")
		return
	end
	if not egg or not egg.model or not egg.model.Parent then
		notify("TFN HUB", "Esse egg nao existe mais. Atualize a lista.", 4, "triangle-alert")
		return
	end
	Busy = true

	StatusFn(("Indo ate %s..."):format(egg.name), ACCENT)
	local reached = travelTo(egg.pos)
	if not reached then
		StatusFn("Nao consegui chegar no egg.", RED)
		Busy = false
		return
	end

	StatusFn("Roubando...", GREEN)
	local got = grabEgg(egg)
	if not got then
		StatusFn("Nao consegui pegar o egg.", RED)
		Busy = false
		return
	end

	if CFG.AutoDeliver then
		local delivered = deliverEgg()
		StatusFn(delivered and ("Entregue: " .. egg.name) or "Peguei, mas falhou a entrega.",
			delivered and GREEN or YELLOW)
	else
		StatusFn("Peguei: " .. egg.name, GREEN)
	end

	task.wait(CFG.Delay)
	Busy = false
end

----------------------------------------------------------------------
-- ESP
----------------------------------------------------------------------
local espFolder
local function clearESP()
	if espFolder then
		espFolder:Destroy()
		espFolder = nil
	end
end

local function refreshESP()
	clearESP()
	if not CFG.ESP then return end
	espFolder = Instance.new("Folder")
	espFolder.Name = "TFN_ESP"
	espFolder.Parent = (gethui and gethui()) or CoreGui

	for _, e in ipairs(filteredEggs()) do
		local adornee = e.part or (e.model and e.model:FindFirstChildWhichIsA("BasePart", true))
		if adornee then
			local bb = Instance.new("BillboardGui")
			bb.Name = "TFN_" .. e.name
			bb.Adornee = adornee
			bb.Size = UDim2.new(0, 210, 0, 44)
			bb.StudsOffset = Vector3.new(0, 4, 0)
			bb.AlwaysOnTop = true
			bb.MaxDistance = 800
			bb.Parent = espFolder

			local lb = Instance.new("TextLabel")
			lb.BackgroundTransparency = 1
			lb.Size = UDim2.fromScale(1, 1)
			lb.Font = Enum.Font.GothamBold
			lb.TextSize = 13
			lb.TextStrokeTransparency = 0.35
			lb.RichText = true
			lb.TextColor3 = RARITY_COLORS[(e.rarity or ""):lower()] or Color3.new(1, 1, 1)
			lb.Text = ("%s\n$%s/s"):format(e.name, short(e.money))
			lb.Parent = bb
			e.espLabel = lb
		end
	end
end

----------------------------------------------------------------------
-- FOTO DO EGG (modelo 3D pra Viewport)
----------------------------------------------------------------------
local function eggPreviewModel(egg)
	local src
	-- 1) model do proprio egg no Workspace
	if egg and egg.model and egg.model:IsA("Model") then
		src = egg.model
	end
	-- 2) asset em ReplicatedStorage.Assets.Models.Eggs
	if egg then
		local assets = ReplicatedStorage:FindFirstChild("Assets")
		local models = assets and assets:FindFirstChild("Models")
		local eggsF  = models and models:FindFirstChild("Eggs")
		if eggsF then
			local match = eggsF:FindFirstChild(egg.name) or eggsF:FindFirstChild(egg.model and egg.model.Name or "")
			if match then src = match end
		end
	end

	local clone
	if src then
		clone = try(function()
			local c = src:Clone()
			for _, d in ipairs(c:GetDescendants()) do
				if d:IsA("ProximityPrompt") or d:IsA("Script") or d:IsA("LocalScript")
					or d:IsA("BillboardGui") or d:IsA("Sound") or d:IsA("ParticleEmitter") then
					d:Destroy()
				elseif d:IsA("BasePart") then
					d.Anchored = true
					d.CanCollide = false
				end
			end
			if c:IsA("BasePart") then
				local m = Instance.new("Model")
				c.Parent = m
				m.PrimaryPart = c
				return m
			end
			return c
		end)
	end

	if not clone then
		clone = Instance.new("Model")
		local p = Instance.new("Part")
		p.Shape = Enum.PartType.Ball
		p.Size = Vector3.new(4, 5, 4)
		p.Color = ACCENT
		p.Material = Enum.Material.SmoothPlastic
		p.Anchored = true
		p.CanCollide = false
		p.Parent = clone
		clone.PrimaryPart = p
	end

	return clone
end

----------------------------------------------------------------------
-- UI (WindUI)
----------------------------------------------------------------------
local Window = WindUI:CreateWindow({
	Title = "TFN HUB",
	Icon = "egg",
	Author = "Steal An Egg  •  Roube um Ovo",
	Folder = "TFNHub",
	Size = UDim2.fromOffset(600, 420),
	Theme = "Dark",
	Transparent = true,
	NewElements = true,
	HideSearchBar = false,
	Resizable = true,
	SideBarWidth = 190,
	Background = "",
	OpenButton = {
		Title = "TFN HUB",
		Enabled = true,
		Draggable = true,
		OnlyMobile = false,
		CornerRadius = UDim.new(1, 0),
		StrokeThickness = 2,
		Color = ColorSequence.new(Color3.fromHex("#8B5CF6"), Color3.fromHex("#22D3EE")),
	},
	Topbar = {
		Height = 44,
		ButtonsType = "Mac",
	},
})

try(function()
	Window:Tag({ Title = "v2.0", Icon = "github", Color = Color3.fromHex("#1c1c1c"), Border = true })
	Window:Tag({ Title = "SAFE MODE", Icon = "shield-check", Color = GREEN, Border = true })
end)

local SecMain   = Window:Section({ Title = "Principal" })
local SecPlayer = Window:Section({ Title = "Player" })
local SecInfo   = Window:Section({ Title = "Sobre" })

----------------------------------------------------------------------
-- TAB: EGGS
----------------------------------------------------------------------
local TabEggs = SecMain:Tab({
	Title = "Eggs",
	Desc = "Escolha o egg e o bot busca",
	Icon = "egg",
	IconColor = ACCENT,
	IconShape = "Square",
	Border = true,
})

local Selected = nil
local EggDropdown, EggViewport, EggInfo, StatusPara

TabEggs:Section({ Title = "Ovo selecionado" })

EggViewport = TabEggs:Viewport({
	Object = eggPreviewModel(nil),
	Height = 170,
	Interactive = true,
	Focused = true,
})

EggInfo = TabEggs:Paragraph({
	Title = "Nenhum ovo selecionado",
	Desc = "Aperte em Escanear e escolha um ovo na lista abaixo.",
})

TabEggs:Space({ Columns = 2 })

local function setSelected(egg)
	Selected = egg
	if not egg then
		try(function() EggInfo:SetTitle("Nenhum ovo selecionado") end)
		try(function() EggInfo:SetDesc("Aperte em Escanear e escolha um ovo na lista abaixo.") end)
		return
	end
	local r = hrp()
	local dist = r and math.floor((egg.pos - r.Position).Magnitude) or 0
	try(function() EggInfo:SetTitle(egg.name) end)
	try(function()
		EggInfo:SetDesc(("Renda: $%s/s   •   Raridade: %s   •   Distancia: %d studs")
			:format(short(egg.money), tostring(egg.rarity), dist))
	end)
	try(function()
		EggViewport:SetObject(eggPreviewModel(egg), false)
		EggViewport:Focus()
	end)
end

local function dropdownValues()
	local list = filteredEggs()
	local vals = {}
	for _, e in ipairs(list) do
		local r = hrp()
		local dist = r and math.floor((e.pos - r.Position).Magnitude) or 0
		table.insert(vals, {
			Title = ("%s  —  $%s/s"):format(e.name, short(e.money)),
			Desc = ("%s  •  %d studs"):format(tostring(e.rarity), dist),
			Icon = "egg",
			Callback = function() setSelected(e) end,
		})
	end
	if #vals == 0 then
		table.insert(vals, {
			Title = "Nenhum ovo encontrado",
			Desc = "Aperte em Escanear ou solte os filtros",
			Icon = "search-x",
			Callback = function() end,
		})
	end
	return vals
end

local function rebuildDropdown()
	if not EggDropdown then return end
	local vals = dropdownValues()
	local applied = try(function() EggDropdown:Refresh(vals) end)
	if applied == nil then
		try(function() EggDropdown:SetValues(vals) end)
	end
end

TabEggs:Section({ Title = "Lista de ovos" })

EggDropdown = TabEggs:Dropdown({
	Title = "Selecionar ovo",
	Desc = "Ordenado por maior renda",
	Icon = "list",
	Values = dropdownValues(),
	Value = nil,
	AllowNone = true,
})

TabEggs:Button({
	Title = "Escanear ovos",
	Desc = "Atualiza a lista, as fotos e o ESP",
	Icon = "radar",
	Callback = function()
		scanEggs()
		rebuildDropdown()
		if CFG.ESP then refreshESP() end
		notify("TFN HUB", ("%d ovos encontrados."):format(#Eggs), 3, "radar")
	end,
})

TabEggs:Space()

TabEggs:Button({
	Title = "ROUBAR O OVO SELECIONADO",
	Desc = "Vai ate o ovo, pega e entrega na base",
	Icon = "hand-coins",
	Color = ACCENT,
	Justify = "Center",
	IconAlign = "Left",
	Callback = function()
		if not Selected then
			notify("TFN HUB", "Selecione um ovo primeiro.", 3, "triangle-alert")
			return
		end
		task.spawn(stealEgg, Selected)
	end,
})

TabEggs:Space()

TabEggs:Button({
	Title = "Roubar o mais valioso",
	Desc = "Pega o de maior $/s que passar nos filtros",
	Icon = "trophy",
	Callback = function()
		scanEggs()
		rebuildDropdown()
		local list = filteredEggs()
		if #list == 0 then
			notify("TFN HUB", "Nenhum ovo passou nos filtros.", 3, "search-x")
			return
		end
		setSelected(list[1])
		task.spawn(stealEgg, list[1])
	end,
})

TabEggs:Space()

TabEggs:Button({
	Title = "PARAR",
	Desc = "Cancela o trajeto e o auto farm",
	Icon = "octagon-x",
	Color = RED,
	Justify = "Center",
	Callback = function()
		CancelTravel = true
		CFG.AutoFarm = false
		Busy = false
		local h = hum()
		if h then
			local r = hrp()
			if r then h:MoveTo(r.Position) end
		end
		StatusFn("Parado pelo usuario.", YELLOW)
	end,
})

TabEggs:Space({ Columns = 2 })
TabEggs:Section({ Title = "Status" })

StatusPara = TabEggs:Paragraph({
	Title = "Aguardando",
	Desc = "Nenhuma acao em andamento.",
})

StatusFn = function(text, _color)
	try(function() StatusPara:SetTitle(text) end)
	try(function()
		StatusPara:SetDesc(("Modo: %s   •   Ovos: %d   •   Carregando ovo: %s")
			:format(CFG.TurboMode and "TURBO (risco)" or "Seguro", #Eggs, isCarrying() and "sim" or "nao"))
	end)
end

----------------------------------------------------------------------
-- TAB: AUTO FARM
----------------------------------------------------------------------
local TabFarm = SecMain:Tab({
	Title = "Auto Farm",
	Desc = "Loop automatico de roubo",
	Icon = "repeat",
	IconColor = GREEN,
	IconShape = "Square",
	Border = true,
})

TabFarm:Section({ Title = "Loop" })

TabFarm:Toggle({
	Title = "Auto Farm",
	Desc = "Rouba os ovos filtrados em sequencia, do mais caro pro mais barato",
	Value = false,
	Callback = function(v)
		CFG.AutoFarm = v
		if v then
			notify("TFN HUB", "Auto Farm ligado.", 3, "repeat")
		end
	end,
})

TabFarm:Space()

TabFarm:Toggle({
	Title = "Entregar na base",
	Desc = "Depois de pegar, volta e deposita no seu plot",
	Value = CFG.AutoDeliver,
	Callback = function(v) CFG.AutoDeliver = v end,
})

TabFarm:Space()

TabFarm:Slider({
	Title = "Delay entre ovos",
	Desc = "Segundos de pausa entre um roubo e o proximo",
	Step = 0.1,
	Value = { Min = 0, Max = 5, Default = CFG.Delay },
	Callback = function(v) CFG.Delay = v end,
})

TabFarm:Space({ Columns = 2 })
TabFarm:Section({ Title = "Filtros" })

TabFarm:Input({
	Title = "Buscar por nome",
	Desc = "Deixe vazio pra listar todos",
	Placeholder = "ex: Golden",
	Callback = function(v)
		CFG.Search = tostring(v or "")
		rebuildDropdown()
	end,
})

TabFarm:Space()

TabFarm:Dropdown({
	Title = "Raridade",
	Desc = "Filtra a lista e o auto farm",
	Values = RARITY_ORDER,
	Value = "Todas",
	Callback = function(v)
		CFG.RarityFilter = tostring(v)
		rebuildDropdown()
		if CFG.ESP then refreshESP() end
	end,
})

TabFarm:Space()

TabFarm:Input({
	Title = "Renda minima ($/s)",
	Desc = "Ignora ovos abaixo desse valor. Aceita 1k, 2.5m",
	Placeholder = "0",
	Callback = function(v)
		CFG.MinMoney = parseNumber(tostring(v)) or 0
		rebuildDropdown()
		if CFG.ESP then refreshESP() end
	end,
})

TabFarm:Space({ Columns = 2 })
TabFarm:Section({ Title = "Extras" })

TabFarm:Button({
	Title = "Vender todos os pets",
	Desc = "RE/PetSatchel/SellEveryPet",
	Icon = "dollar-sign",
	Callback = function()
		local ok = invoke(R.SellAll)
		notify("TFN HUB", ok ~= nil and "Pedido de venda enviado." or "Remote nao respondeu.", 3, "dollar-sign")
	end,
})

TabFarm:Space()

TabFarm:Button({
	Title = "Chocar ovos do plot",
	Desc = "AskHatch + AskFinishHatch",
	Icon = "sparkles",
	Callback = function()
		invoke(R.Hatch)
		task.wait(0.3)
		invoke(R.Finish)
		notify("TFN HUB", "Pedido de choco enviado.", 3, "sparkles")
	end,
})

----------------------------------------------------------------------
-- TAB: MOVIMENTO
----------------------------------------------------------------------
local TabMove = SecPlayer:Tab({
	Title = "Movimento",
	Desc = "Como o bot se desloca",
	Icon = "footprints",
	IconColor = Color3.fromHex("#257AF7"),
	IconShape = "Square",
	Border = true,
})

TabMove:Section({
	Title = "O print de erro que voce mandou (BAC-10518) era BAN do anti-cheat, nao bug de UI.\nEste jogo detecta teleporte por CFrame e noclip. Por isso o modo padrao anda de verdade, com Pathfinding.",
	TextSize = 15,
	TextTransparency = 0.3,
	FontWeight = Enum.FontWeight.Medium,
})

TabMove:Space({ Columns = 2 })

TabMove:Toggle({
	Title = "Turbo (RISCO DE BAN)",
	Desc = "Desloca por CFrame. Muito mais rapido, mas foi isso que te kickou antes. Use por sua conta e risco.",
	Value = false,
	Callback = function(v)
		CFG.TurboMode = v
		if v then
			notify("TFN HUB", "Turbo LIGADO. Risco real de kick BAC-10518.", 7, "triangle-alert")
		else
			notify("TFN HUB", "Turbo desligado. Movimento seguro ativo.", 3, "shield-check")
		end
	end,
})

TabMove:Space()

TabMove:Slider({
	Title = "Velocidade do Turbo",
	Desc = "Studs por segundo (so vale com Turbo ligado)",
	Step = 10,
	Value = { Min = 60, Max = 400, Default = CFG.TurboSpeed },
	Callback = function(v) CFG.TurboSpeed = v end,
})

TabMove:Space()

TabMove:Slider({
	Title = "Timeout do trajeto",
	Desc = "Desiste do caminho depois de X segundos",
	Step = 5,
	Value = { Min = 15, Max = 120, Default = CFG.TravelTimeout },
	Callback = function(v) CFG.TravelTimeout = v end,
})

TabMove:Space({ Columns = 2 })
TabMove:Section({ Title = "Guards / NPCs" })

TabMove:Toggle({
	Title = "Desviar dos guards",
	Desc = "Recalcula a rota pra dar a volta nos NPCs em vez de passar por cima",
	Value = CFG.AvoidGuards,
	Callback = function(v) CFG.AvoidGuards = v end,
})

TabMove:Space()

TabMove:Slider({
	Title = "Raio de deteccao",
	Desc = "A que distancia comeca a desviar do guard",
	Step = 2,
	Value = { Min = 10, Max = 80, Default = CFG.GuardRadius },
	Callback = function(v) CFG.GuardRadius = v end,
})

TabMove:Space()

TabMove:Toggle({
	Title = "Anti-ragdoll",
	Desc = "Levanta o personagem quando o guard te derruba (seguro)",
	Value = CFG.AntiRagdoll,
	Callback = function(v) CFG.AntiRagdoll = v end,
})

TabMove:Space({ Columns = 2 })
TabMove:Section({ Title = "Atalhos" })

TabMove:Button({
	Title = "Ir para minha base",
	Icon = "house",
	Callback = function()
		local b = basePosition()
		if not b then
			notify("TFN HUB", "Nao achei sua base/plot.", 3, "triangle-alert")
			return
		end
		task.spawn(function()
			StatusFn("Indo pra base...", ACCENT)
			local ok = travelTo(b)
			StatusFn(ok and "Cheguei na base." or "Nao cheguei na base.", ok and GREEN or RED)
		end)
	end,
})

TabMove:Space()

TabMove:Button({
	Title = "Largar o ovo que estou carregando",
	Icon = "package-open",
	Callback = function()
		invoke(R.Drop)
		notify("TFN HUB", "Pedido de drop enviado.", 3, "package-open")
	end,
})

----------------------------------------------------------------------
-- TAB: VISUAL
----------------------------------------------------------------------
local TabVisual = SecPlayer:Tab({
	Title = "Visual",
	Desc = "ESP e informacoes na tela",
	Icon = "eye",
	IconColor = YELLOW,
	IconShape = "Square",
	Border = true,
})

TabVisual:Section({ Title = "ESP" })

TabVisual:Toggle({
	Title = "ESP dos ovos",
	Desc = "Mostra nome e $/s em cima de cada ovo (client-side, nao detectavel)",
	Value = false,
	Callback = function(v)
		CFG.ESP = v
		if v then
			if #Eggs == 0 then scanEggs() end
			refreshESP()
		else
			clearESP()
		end
	end,
})

TabVisual:Space()

TabVisual:Toggle({
	Title = "Mostrar distancia no ESP",
	Desc = "Atualiza a distancia em tempo real",
	Value = false,
	Callback = function(v) CFG.ESPDistance = v end,
})

TabVisual:Space({ Columns = 2 })
TabVisual:Section({ Title = "Player (usar com cuidado)" })

TabVisual:Toggle({
	Title = "Noclip (RISCO)",
	Desc = "Atravessa paredes. CharacterIntegrity pode pegar. Off por padrao.",
	Value = false,
	Callback = function(v)
		CFG.NoclipRisky = v
		if v then notify("TFN HUB", "Noclip ligado. Risco de kick.", 6, "triangle-alert") end
	end,
})

TabVisual:Space()

TabVisual:Toggle({
	Title = "Pulo infinito (RISCO)",
	Desc = "Pular no ar. Tambem pode levantar flag.",
	Value = false,
	Callback = function(v) CFG.InfJump = v end,
})

----------------------------------------------------------------------
-- TAB: SOBRE
----------------------------------------------------------------------
local TabAbout = SecInfo:Tab({
	Title = "Sobre",
	Desc = "Informacoes e diagnostico",
	Icon = "info",
	IconColor = GREY,
	IconShape = "Square",
	Border = true,
})

TabAbout:Section({ Title = "TFN HUB" })

TabAbout:Section({
	Title = "Feito para Steal An Egg / Roube um Ovo (PlaceId 107778070777162).\nUI: WindUI. Testado para Arceus X e executores mobile.\n\nO seu print anterior com \"You have been removed for cheating | CODE BAC-10518\" NAO era erro de interface: era o anti-cheat do jogo respondendo ao teleporte por CFrame. Este build anda com Pathfinding por padrao, entao o risco cai muito.",
	TextSize = 15,
	TextTransparency = 0.35,
	FontWeight = Enum.FontWeight.Medium,
})

TabAbout:Space({ Columns = 2 })

local DiagPara = TabAbout:Paragraph({
	Title = "Diagnostico",
	Desc = "Aperte em Rodar diagnostico.",
})

TabAbout:Button({
	Title = "Rodar diagnostico",
	Desc = "Confere remotes, plot e prompts",
	Icon = "stethoscope",
	Callback = function()
		local lines = {}
		table.insert(lines, ("Networking: %s"):format(Net and "OK" or "NAO ENCONTRADO"))
		for k, v in pairs(R) do
			table.insert(lines, ("%s: %s"):format(k, remote(v) and "OK" or "faltando"))
		end
		table.insert(lines, ("Plot: %s"):format(myPlot() and "encontrado" or "nao encontrado"))
		table.insert(lines, ("Base: %s"):format(basePosition() and "OK" or "nao achei"))
		table.insert(lines, ("Ovos no scan: %d"):format(#Eggs))
		table.insert(lines, ("fireproximityprompt: %s"):format(fireproximityprompt and "disponivel" or "AUSENTE"))
		try(function() DiagPara:SetDesc(table.concat(lines, "\n")) end)
		notify("TFN HUB", "Diagnostico atualizado.", 3, "stethoscope")
	end,
})

TabAbout:Space({ Columns = 2 })

TabAbout:Button({
	Title = "Fechar TFN HUB",
	Desc = "Remove a interface e para todos os loops",
	Icon = "shredder",
	Color = RED,
	Justify = "Center",
	Callback = function()
		if _G.TFN_HUB_DESTROY then _G.TFN_HUB_DESTROY() end
	end,
})

----------------------------------------------------------------------
-- LOOPS
----------------------------------------------------------------------
local Running = true
local Conns = {}

-- anti-ragdoll + noclip + inf jump
table.insert(Conns, RunService.Stepped:Connect(function()
	if not Running then return end

	if CFG.AntiRagdoll then
		local h = hum()
		if h then
			if h.PlatformStand then h.PlatformStand = false end
			if h.Sit then h.Sit = false end
		end
	end

	if CFG.NoclipRisky then
		local c = char()
		if c then
			for _, p in ipairs(c:GetDescendants()) do
				if p:IsA("BasePart") and p.CanCollide and p.Name ~= "HumanoidRootPart" then
					p.CanCollide = false
				end
			end
		end
	end
end))

table.insert(Conns, cloneref(game:GetService("UserInputService")).JumpRequest:Connect(function()
	if Running and CFG.InfJump then
		local h = hum()
		if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
	end
end))

-- status + ESP distancia
task.spawn(function()
	while Running do
		task.wait(1)
		if CFG.ESPDistance and CFG.ESP then
			local r = hrp()
			if r then
				for _, e in ipairs(Eggs) do
					if e.espLabel and e.espLabel.Parent then
						local d = math.floor((e.pos - r.Position).Magnitude)
						e.espLabel.Text = ("%s\n$%s/s  •  %dm"):format(e.name, short(e.money), d)
					end
				end
			end
		end
		if not Busy and not Travelling then
			StatusFn(CFG.AutoFarm and "Auto Farm rodando..." or "Aguardando", ACCENT)
		end
	end
end)

-- auto farm
task.spawn(function()
	while Running do
		task.wait(0.5)
		if CFG.AutoFarm and not Busy and alive() then
			scanEggs()
			local list = filteredEggs()
			if #list == 0 then
				StatusFn("Auto Farm: nenhum ovo passou nos filtros.", YELLOW)
				task.wait(3)
			else
				rebuildDropdown()
				setSelected(list[1])
				stealEgg(list[1])
			end
		end
	end
end)

-- rescan periodico
task.spawn(function()
	while Running do
		task.wait(12)
		if not Busy then
			scanEggs()
			rebuildDropdown()
			if CFG.ESP then refreshESP() end
		end
	end
end)

-- respawn
table.insert(Conns, LP.CharacterAdded:Connect(function()
	task.wait(2)
	CancelTravel = false
	Busy = false
end))

----------------------------------------------------------------------
-- DESTROY
----------------------------------------------------------------------
_G.TFN_HUB_DESTROY = function()
	Running = false
	CFG.AutoFarm = false
	CancelTravel = true
	for _, c in ipairs(Conns) do try(function() c:Disconnect() end) end
	Conns = {}
	clearESP()
	try(function() Window:Destroy() end)
	_G.TFN_HUB_LOADED = false
end

----------------------------------------------------------------------
-- START
----------------------------------------------------------------------
task.spawn(function()
	scanEggs()
	rebuildDropdown()
	local list = filteredEggs()
	if #list > 0 then setSelected(list[1]) end
	StatusFn("Pronto", ACCENT)
	notify("TFN HUB", ("Carregado. %d ovos encontrados. Modo seguro ativo."):format(#Eggs), 6, "egg")
end)

try(function() Window:SelectTab(1) end)
