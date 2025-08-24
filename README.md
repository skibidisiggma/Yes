--[[ ⚔️ Guild: Nyxara Exploits Hub ]]
-- Full Pet Sniper + ESP + Webhook A + Webhook B + Auto Server Hop + Auto Restart + 👥 Players
-- Steal a Brainrot — Script Completo con Traits y Mutaciones actualizados (Agosto 2025)

--// 🎯 Targeted Pet Configuration
getgenv().WebhookATargets = {
    "Chicleteira Bicicleteira",
    "Dragon Cannelloni",
    "La Grande Combinasion",
    "Garama and Madundung",
    "Nuclearo Dinossauro",
    "Los Combinasionas",
    "Esok Sekolah",
    "Los Hotspotitos",
    "Pot Hostpot"
}

getgenv().WebhookBTargets = {
    "La Vacca Saturno Saturnita",
    "Chimpanzini Spiderini",
    "Los Tralaleritos",
    "Tortuginni Dragonfrutini",
    "Las Vaquitas Saturnitas",
    "Graipuss Medussi",
    "Las Tralaleritas"
}

--// 🌐 Webhooks
local webhookA_Temprana = "https://discord.com/api/webhooks/1406038878879748229/o0w6vCqGVaWt-ay64pLieGdnXk9Kwwfu4c2m9eTdlcpUTeMkY0Qaoj1btieclAL3vZri"
local webhookA = "https://discord.com/api/webhooks/1404942415525183599/41L2bmvHY8XUiFzeJA96gBIxc4wnlLGdY8jxulVcOzpvW-ZyhZRn3PTGApQvCTnu_BtQ"
local webhookB = "https://discord.com/api/webhooks/1404225086940250203/VJzWDW_vkq6iid9CyrPfZeUYOHt2u9d12YaOuE_cJ6VieoHMd7kb5N4PdZqvxgz85VuU"

--// 🔧 Services
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local LocalPlayer = Players.LocalPlayer

--// 📦 State
local visitedJobIds = {[game.JobId] = true}
local hops = 0
local maxHopsBeforeReset = 1000
local maxTeleportRetries = 10

--// 🧠 Brainrot Data (base values)
local brainrotData = {
    ["Dragon Cannelloni"] = {baseValue = 100000000},
    ["Chicleteira Bicicleteira"] = {baseValue = 3500000},
    ["Los Combinasionas"] = {baseValue = 15000000},
    ["La Grande Combinasion"] = {baseValue = 10000000},
    ["Garama and Madundung"] = {baseValue = 50000000},
    ["Nuclearo Dinossauro"] = {baseValue = 15000000},
    ["Esok Sekolah"] = {baseValue = 30000000},
    ["Los Hotspotitos"] = {baseValue = 20000000},
    ["Pot Hostpot"] = {baseValue = 2500000},

    ["La Vacca Saturno Saturnita"] = {baseValue = 250000},
    ["Chimpanzini Spiderini"] = {baseValue = 325000},
    ["Los Tralaleritos"] = {baseValue = 500000},
    ["Tortuginni Dragonfrutini"] = {baseValue = 350000},
    ["Las Vaquitas Saturnitas"] = {baseValue = 750000},
    ["Graipuss Medussi"] = {baseValue = 1000000},
    ["Las Tralaleritas"] = {baseValue = 650000}
}

--// Trait y Mutación multipliers (Steal a Brainrot)
local traitMultipliers = {
    -- Traits
    Taco = 3,
    ["Nyan"] = 6,
    ["Claws"] = 5,
    ["Tung Tung Attack"] = 4,
    Bubblegum = 4,
    Cometstruck = 4,
    Glitched = 5,
    ["Fire (Solar Flare)"] = 5,
    Concert = 5,
    Shark = 4,
    ["10B Visits"] = 4,
    Rain = 4,
    Snowy = 3,
    Starfall = 6,
    ["Brazil"] = 6,
    ["Matteo's Hat"] = 4,
    Galactic = 4,
    Asteroid = 4,
    Firework = 6, 

    -- Mutaciones
    Rainbow = 10,
    Gold = 1.25,
    Diamond = 1.5,
    ["Blood Root"] = 2,
    Candy = 5, -- ahora también es mutación
}

--// Función para obtener traits
local function getTraits(petModel)
    local traits = {}
    for _, child in ipairs(petModel:GetDescendants()) do
        if (child:IsA("StringValue") or child:IsA("IntValue") or child:IsA("NumberValue")) then
            local nameLower = string.lower(child.Name)
            if string.find(nameLower,"trait") or string.find(nameLower,"mutation") or string.find(nameLower,"effect") then
                table.insert(traits,tostring(child.Value))
            end
        end
    end
    for attrName, attrValue in pairs(petModel:GetAttributes()) do
        local nameLower = string.lower(attrName)
        if string.find(nameLower,"trait") or string.find(nameLower,"mutation") or string.find(nameLower,"effect") then
            table.insert(traits,tostring(attrValue))
        end
    end
    if #traits==0 then return "None" else return table.concat(traits,", ") end
end

--// Formatear dinero corto
local function formatMoney(money)
    if money>=1000000 then
        return string.format("%.1fM/s",money/1000000)
    elseif money>=1000 then
        return string.format("%.0fk/s",money/1000)
    else
        return money.."/s"
    end
end

--// Calcular dinero con traits y mutaciones
local function calculateMoneyPerSecond(petModel)
    local data=brainrotData[petModel.Name]
    if not data then return 0 end
    local total=data.baseValue
    local detectedTraits=getTraits(petModel)
    if detectedTraits~="None" then
        for trait in string.gmatch(detectedTraits,"[^,]+") do
            trait=trait:match("^%s*(.-)%s*$")
            local mult=traitMultipliers[trait]
            if mult then total=total*mult end
        end
    end
    return total
end

--// ESP
local function addESP(targetModel)
    if targetModel:FindFirstChild("PetESP") then return end
    local Billboard=Instance.new("BillboardGui")
    Billboard.Name="PetESP"
    Billboard.Adornee=targetModel
    Billboard.Size=UDim2.new(0,100,0,30)
    Billboard.StudsOffset=Vector3.new(0,3,0)
    Billboard.AlwaysOnTop=true
    Billboard.Parent=targetModel

    local Label=Instance.new("TextLabel")
    Label.Size=UDim2.new(1,0,1,0)
    Label.BackgroundTransparency=1
    Label.Text="🎯 Target Pet"
    Label.TextColor3=Color3.fromRGB(255,0,0)
    Label.TextStrokeTransparency=0.5
    Label.Font=Enum.Font.SourceSansBold
    Label.TextScaled=true
    Label.Parent=Billboard
end

--// Webhook Sender
local function sendWebhook(foundPets,jobId,url)
    local formattedPets={}
    for _,pet in ipairs(foundPets) do
        local traits=getTraits(pet)
        local moneyPerSec=formatMoney(calculateMoneyPerSecond(pet))
        table.insert(formattedPets,pet.Name.." \n⭐ Traits/Mutations: "..traits.." \n💰 Money/sec: "..moneyPerSec)
    end

    local playersCount=#Players:GetPlayers()

    local joinerUrl="https://testing5312.github.io/joiner/?placeId="..tostring(game.PlaceId).."&gameInstanceId="..jobId
    local jsonData=HttpService:JSONEncode({
        ["content"]=(url==webhookA or url==webhookA_Temprana) and "@everyone" or "",
        ["embeds"]={{["title"]="Shadow Notifier⭐️",
            ["description"]="Sniped Brainrot in server",
            ["fields"]={
                {["name"]="User",["value"]=LocalPlayer.Name},
                {["name"]="Found Pet(s)",["value"]=table.concat(formattedPets,"\n\n")},
                {["name"]="👥 Players",["value"]=tostring(playersCount)},
                {["name"]="Server JobId",["value"]=jobId},
                {["name"]="🌐 Join Server",["value"]="[Click here]("..joinerUrl..")"},
                {["name"]="Time",["value"]=os.date("%Y-%m-%d %H:%M:%S")}
            },
            ["color"]=0x800080}}
    })

    local req=http_request or request or (syn and syn.request)
    if req then
        pcall(function() req({Url=url,Method="POST",Headers={["Content-Type"]="application/json"},Body=jsonData}) end)
    end
end

--// Detectar Pets
local function checkForPets()
    local foundA,foundB={},{}
    for _,obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and not obj:FindFirstChild("PetESP") then
            local nameLower=string.lower(obj.Name)
            for _,target in pairs(getgenv().WebhookATargets) do
                if string.find(nameLower,string.lower(target)) then
                    addESP(obj)
                    table.insert(foundA,obj)
                end
            end
            for _,target in pairs(getgenv().WebhookBTargets) do
                if string.find(nameLower,string.lower(target)) then
                    addESP(obj)
                    table.insert(foundB,obj)
                end
            end
        end
    end
    return foundA,foundB
end

--// Server Hop
local function serverHop(delayTime)
    hops+=1
    if hops>=maxHopsBeforeReset then
        visitedJobIds={[game.JobId]=true}
        hops=0
    end

    task.wait(delayTime or 0)
    local tries,cursor=0,nil
    while tries<maxTeleportRetries do
        local url="https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100"
        if cursor then url=url.."&cursor="..cursor end
        local success,response=pcall(function() return HttpService:JSONDecode(game:HttpGet(url)) end)
        if success and response and response.data then
            local servers={}
            for _,server in ipairs(response.data) do
                if server.playing<2 and not visitedJobIds[server.id] and server.id~=game.JobId then
                    table.insert(servers,server.id)
                end
            end
            if #servers>0 then
                local picked=servers[math.random(1,#servers)]
                visitedJobIds[picked]=true
                TeleportService:TeleportToPlaceInstance(game.PlaceId,picked)
                return
            end
            cursor=response.nextPageCursor
            if not cursor then tries+=1;task.wait(0.05) end
        else tries+=1;task.wait(0.05) end
    end
    TeleportService:Teleport(game.PlaceId)
end

--// Sniping Loop
local function startSniper()
    local foundA,foundB = checkForPets()
    if #foundA > 0 then
        -- Webhook temprana inmediata con @everyone
        sendWebhook(foundA, game.JobId, webhookA_Temprana)
        -- Webhook principal 20s delay con @everyone
        task.delay(5, function()
            sendWebhook(foundA, game.JobId, webhookA)
        end)
        -- Server Hop qps delay
        task.delay(5, function()
            serverHop(5)
        end)
    elseif #foundB > 0 then
        sendWebhook(foundB, game.JobId, webhookB)
        serverHop(5)
    else
        serverHop(0.1)
    end
    task.delay(5, startSniper)
end

--// Start
task.delay(0.1, startSniper)
