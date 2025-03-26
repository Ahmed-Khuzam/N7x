local WebhookManager = {}
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

local WEBHOOK_QUEUE = {}
local IS_PROCESSING = false
local RATE_LIMIT = 5
local MAX_RETRIES = 3

local function processQueue()
    if IS_PROCESSING then return end
    IS_PROCESSING = true
    
    while #WEBHOOK_QUEUE > 0 do
        local request = table.remove(WEBHOOK_QUEUE, 1)
        local retryCount = 0
        local success = false
        
        repeat
            success, response = pcall(function()
                return HttpService:PostAsync(
                    request.url,
                    HttpService:JSONEncode(request.payload),
                    Enum.HttpContentType.ApplicationJson
                )
            end)
            
            if not success then
                retryCount += 1
                task.wait(2 ^ retryCount)
            end
        until success or retryCount >= MAX_RETRIES
        
        task.wait(RATE_LIMIT)
    end
    
    IS_PROCESSING = false
end

function WebhookManager.send(url, data)
    if RunService:IsStudio() then return false end
    if not url or not data then return false end
    
    table.insert(WEBHOOK_QUEUE, {
        url = url,
        payload = data
    })
    
    if not IS_PROCESSING then
        task.spawn(processQueue)
    end
    
    return true
end

function WebhookManager.monitorAdminPanel(player, webhookUrl)
    local adminGui = player:WaitForChild("PlayerGui"):WaitForChild("MainGui"):WaitForChild("AdminApp")
    local editFrame = adminGui:WaitForChild("MainFrame"):WaitForChild("EaditVars")
    local targetButton = editFrame:WaitForChild("btnVar")
    local targetIdField = editFrame:WaitForChild("txtPlrID")
    local varNameField = editFrame:WaitForChild("txtVarName")
    local varValueField = editFrame:WaitForChild("txtVarValue")

    local oldValues = {}

    local function getAdminInfo()
        local info = {
            Name = player.Name,
            UserId = player.UserId,
            plrCardName = player.Name,
            plrCardId = "N/A"
        }
        
        pcall(function()
            info.plrCardName = player.plrVars.plrCardName.Value
            info.plrCardId = player.plrVars.plrCardId.Value
        end)
        
        return info
    end

    targetButton.MouseButton1Click:Connect(function()
        local targetId = targetIdField.Text or "N/A"
        local varName = varNameField.Text or "N/A"
        local newValue = varValueField.Text or "N/A"
        
        local adminInfo = getAdminInfo()
        local targetInfo = {
            plrCardName = "Unknown",
            UserId = targetId,
            plrCardId = "N/A",
            currentValue = "N/A"
        }

        for _, targetPlayer in ipairs(game:GetService("Players"):GetPlayers()) do
            local success, cardId = pcall(function() 
                return targetPlayer.plrVars.plrCardId.Value 
            end)
            
            if (success and tostring(cardId) == targetId) or tostring(targetPlayer.UserId) == targetId then
                pcall(function()
                    targetInfo.plrCardName = targetPlayer.plrVars.plrCardName.Value
                    targetInfo.plrCardId = targetPlayer.plrVars.plrCardId.Value
                    targetInfo.currentValue = tostring(targetPlayer.plrVars[varName].Value)
                end)
                break
            end
        end

        local oldValue = oldValues[varName] or targetInfo.currentValue
        
        local embed = {
            title = "ADMIN PANEL MODIFICATION",
            description = string.format([[
**👤 Admin:** %s (%s)
**🎯 Target:** %s (%s)
**🔄 Change:** %s: %s → %s]],
                adminInfo.Name, adminInfo.UserId,
                targetInfo.plrCardName, targetInfo.UserId,
                varName, oldValue, newValue),
            color = 0x3498DB,
            footer = {text = os.date("%Y-%m-%d %H:%M:%S")}
        }

        WebhookManager.send(webhookUrl, {embeds = {embed}})
        oldValues[varName] = newValue
    end)
end

task.spawn(processQueue)

return WebhookManager
