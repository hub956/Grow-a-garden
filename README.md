# Grow-a-garden
Nothing
-- AdminPetTransfer.lua (put in ServerScriptService)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Replace with your admin userids (numbers). Example: {12345678, 87654321}
local AdminUserIds = {
    -- put admin numeric ids here
    -- you can get your numeric id by: Players:GetUserIdFromNameAsync("lynxfriox")
}

local PetsFolder = workspace:FindFirstChild("Pets") -- where pet Instances are kept
if not PetsFolder then
    warn("AdminPetTransfer: workspace.Pets not found. Create a folder named 'Pets' and put pet instances inside.")
end

local function isAdmin(userId)
    for _,id in ipairs(AdminUserIds) do
        if id == userId then return true end
    end
    return false
end

-- Find pet instance by PetId attribute
local function findPetInstanceById(petId)
    if not PetsFolder then return nil end
    for _, pet in pairs(PetsFolder:GetChildren()) do
        local pId = pet:GetAttribute("PetId")
        if pId and tostring(pId) == tostring(petId) then
            return pet
        end
    end
    return nil
end

-- Move pet to target user (server-only). Returns success, message
local function transferPetById(petId, targetUserId, adminUserId)
    -- validate admin
    if not isAdmin(adminUserId) then
        return false, "Not authorized (not an admin)."
    end

    local pet = findPetInstanceById(petId)
    if not pet then
        return false, "Pet not found."
    end

    -- optional: check current owner (for logging)
    local oldOwnerId = pet:GetAttribute("OwnerUserId")

    -- update attributes / parent to new owner area
    pet:SetAttribute("OwnerUserId", tonumber(targetUserId))

    -- Example: move instance to a folder named after the owner under workspace.Inventory
    local inventoryRoot = workspace:FindFirstChild("Inventory")
    if inventoryRoot then
        local targetFolder = inventoryRoot:FindFirstChild(tostring(targetUserId))
        if not targetFolder then
            targetFolder = Instance.new("Folder")
            targetFolder.Name = tostring(targetUserId)
            targetFolder.Parent = inventoryRoot
        end
        pet.Parent = targetFolder
    end

    -- If you use DataStore or server tables for persistence, update them here.
    -- e.g., update your Pets table or DataStore to reflect new owner.

    -- Log action (print + server log)
    print(string.format("Admin %d transferred pet %s from %s to %s", adminUserId, tostring(petId), tostring(oldOwnerId), tostring(targetUserId)))

    return true, "Transfer complete."
end

-- Example: run from server console
-- In Studio's Command Bar (View → Command Bar), run:
--     game:GetService("ReplicatedStorage").AdminPetTransfer:Invoke("transfer",  "PET_ID", TARGET_USERID, ADMIN_USERID)
-- We'll expose a BindableFunction in ReplicatedStorage to help call from Command Bar if you want.

-- Optional: create a BindableFunction for local server calls (only accessible server-side)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local bindName = "AdminPetTransfer"
local existing = ReplicatedStorage:FindFirstChild(bindName)
if existing then existing:Destroy() end

local bindable = Instance.new("BindableFunction")
bindable.Name = bindName
bindable.Parent = ReplicatedStorage

bindable.OnInvoke = function(command, petId, targetUserId, adminUserId)
    if command == "transfer" then
        return transferPetById(petId, targetUserId, adminUserId)
    else
        return false, "Unknown command"
    end
end

print("AdminPetTransfer initialized. Use ReplicatedStorage.AdminPetTransfer:Invoke('transfer', petId, targetUserId, adminUserId)")
