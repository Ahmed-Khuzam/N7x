local HttpService = game:GetService("HttpService")
local Webhook_URL = "https://discord.com/api/webhooks/1354208343874470010/TVEWeIpFVEI72sEvP0HCHwDz0oTzE_XB5ZphxjidqaT0wDA6hFjqHmID1JOmBmar7oYZ"

local player = game:GetService("Players").LocalPlayer
while not player:FindFirstChild("PlayerGui") do wait(0.1) end

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

local function getTargetInfo(targetId)
    targetId = tostring(targetId)
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        local success, cardId = pcall(function() return player.plrVars.plrCardId.Value end)
        if (success and tostring(cardId) == targetId) or tostring(player.UserId) == targetId then
            local targetInfo = {
                plrCardName = player.Name,
                UserId = player.UserId,
                plrCardId = "N/A",
                currentValue = "N/A"
            }
            
            pcall(function()
                targetInfo.plrCardName = player.plrVars.plrCardName.Value
                targetInfo.plrCardId = player.plrVars.plrCardId.Value
                targetInfo.currentValue = tostring(player.plrVars[varNameField.Text].Value)
            end)
            
            return targetInfo
        end
    end
    return {
        plrCardName = "Unknown", 
        UserId = targetId, 
        plrCardId = "N/A",
        currentValue = "N/A"
    }
end

local function sendAdminActionToWebhook()
    local targetId = pcall(function() return targetIdField.Text end) and targetIdField.Text or "N/A"
    local varName = pcall(function() return varNameField.Text end) and varNameField.Text or "N/A"
    local newValue = pcall(function() return varValueField.Text end) and varValueField.Text or "N/A"
    
    local adminInfo = getAdminInfo()
    local targetInfo = getTargetInfo(targetId)
    local oldValue = oldValues[varName] or targetInfo.currentValue
    
    local message = string.format([[
**📊 The Information - Admin Panel Activity**

**🛠️ Administrator:**
- Name: `%s`
- UserID: `%s`
- plrCardName: `%s`
- plrCardId: `%s`

**🎯 Target Player:**
- plrCardName: `%s`
- UserID: `%s`
- plrCardId: `%s`

**🔄 Value Change:**
- Variable: `%s`
- Old Value: `%s`
- New Value: `%s`
]],
    adminInfo.Name,
    adminInfo.UserId,
    adminInfo.plrCardName,
    adminInfo.plrCardId,
    targetInfo.plrCardName,
    targetInfo.UserId,
    targetInfo.plrCardId,
    varName,
    oldValue,
    newValue)

    local payload = {
        embeds = {{
            title = "ADMIN PANEL MODIFICATION",
            description = message,
            color = 0x3498DB,
            footer = {
                text = "Action Performed: "..os.date("%Y-%m-%d %H:%M:%S")
            }
        }}
    }

    local success, response = pcall(function()
        HttpService:PostAsync(Webhook_URL, HttpService:JSONEncode(payload))
    end)
    
    if not success then
        warn("Webhook failed:", response)
    else
        oldValues[varName] = newValue
    end
end

targetButton.MouseButton1Click:Connect(sendAdminActionToWebhook)
