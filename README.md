local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/SixZensED/Scripts/refs/heads/main/UI%20Libraries/dis.lua"))()

local main = library:Window("Budget : 400")
local serv = main:Server("Farm", "")
local btns = serv:Channel("Main")

local function checkLine(s)
    local a = 0
    for i in string.gmatch(s, "[^\n]+") do
        a = a + 1
    end
    return a
end

local function addspace(a)
    local st = ""
    for i = 0, a * 2 do
        st = st .. " "
    end
    return st
end

local function usd(v, i)
    local turn
    if typeof(v) == "Axes" then
        turn = "Axes.new(" .. tostring(v) .. ")"
    elseif typeof(v) == "BrickColor" then
        turn = "BrickColor.new(" .. tostring(v) .. ")"
    elseif typeof(v) == "CFrame" then
        turn = "CFrame.new(" .. tostring(v) .. ")"
    elseif typeof(v) == "Vector3" then
        turn = "Vector3.new(" .. tostring(v) .. ")"
    elseif typeof(v) == "Color3" then
        if v.r <= 1 and v.g <= 1 and v.b <= 1 and v.r >= 0 and v.g >= 0 and v.b >= 0 then
            local R = v.r * 255
            local G = v.g * 255
            local B = v.b * 255
            turn = "Color3.fromRGB(" .. R .. ", " .. G .. ", " .. B .. ")"
        else
            turn = "Color Error"
        end
    elseif typeof(v) == "Instance" then
        turn = "game." .. tostring(v:GetFullName())
    elseif typeof(v) == "Enum" then
        turn = tostring(v)
    else
        turn = tostring(typeof(v)) .. ".new(" .. tostring(v) .. ")"
    end
    if i then
        if type(i) == "number" then
            return "[" .. tostring(i) .. "] = " .. tostring(turn)
        end
        return '["' .. tostring(i) .. '"] = ' .. tostring(turn)
    end
    return tostring(turn)
end

local convert
local startSpace = 1
function convert(v, i, a)
    if not a then
        a = startSpace
    end
    if type(v) == "string" then
        if checkLine(v) >= 2 then
            if type(i) == "number" then
                return "[" .. tostring(i) .. "] = [[" .. tostring(v) .. "]]"
            end
            return '["' .. tostring(i) .. '"] = [[' .. tostring(v) .. "]]"
        else
            if type(i) == "number" then
                return "[" .. tostring(i) .. '] = "' .. tostring(v) .. '"'
            end
            return '["' .. tostring(i) .. '"] = "' .. tostring(v) .. '"'
        end
    elseif type(v) == "number" then
        if type(i) == "number" then
            return "[" .. tostring(i) .. "] = " .. tostring(v)
        end
        return '["' .. tostring(i) .. '"] = ' .. tostring(v)
    elseif type(v) == "boolean" then
        if type(i) == "number" then
            return "[" .. tostring(i) .. "] = " .. tostring(v)
        end
        return '["' .. tostring(i) .. '"] = ' .. tostring(v)
    elseif type(v) == "nil" then
        if type(i) == "number" then
            return "[" .. tostring(i) .. "] = " .. tostring(v)
        end
        return '["' .. tostring(i) .. '"] = nil'
    elseif type(v) == "function" then
        local Name = ""
        if debug.getinfo(v).name == "" then
            Name = tostring(v)
        else
            Name = debug.getinfo(v).name
        end
        if type(i) == "number" then
            return "[" .. tostring(i) .. "] = function()end [" .. tostring(Name) .. "]"
        end
        return '["' .. tostring(i) .. '"] = function()end [' .. tostring(Name) .. "]"
    elseif type(v) == "userdata" or type(v) == "vector" then
        return usd(v, i)
    elseif type(v) == "table" then
        local stt = ""
        if i ~= nil then
            if type(i) == "number" then
                stt = "[" .. tostring(i) .. "] = {"
            else
                stt = '["' .. tostring(i) .. '"] = {'
            end
        else
            stt = "{"
        end
        local count_table = 0
        for i, v in pairs(v) do
            count_table = count_table + 1
            if count_table == 1 then
                stt = stt .. "\n" .. tostring(addspace(a)) .. tostring(convert(v, i, a + 1))
            else
                stt = stt .. ",\n" .. tostring(addspace(a)) .. tostring(convert(v, i, a + 1))
            end
        end
        return stt .. "\n}"
    elseif type(v) == "thread" then
        if type(i) == "number" then
            return "[" .. tostring(i) .. "] = " .. tostring(v)
        end
        return '["' .. tostring(i) .. '"] = ' .. tostring(v)
    end
end

local RecordMacroTable = {}
local step = 1

btns:Toggle("Start Record", false, function(bool)
    startrecord = bool
    if startrecord then
        RecordMacroTable = {}
        step = 1
    else
        writefile("macroO.txt", convert(RecordMacroTable))
    end
end)

local remote_record = {
    "PlaceTower",
    "Upgrade",
    "Sell"
}
local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)
mt.__namecall = function(self, ...)
    if not checkcaller() then
        local args = { ... }
        if startrecord then
            if self.Name == "PlaceTower" then
                table.insert(RecordMacroTable, {
                    ["step"] = step,
                    ["type"] = self.Name,
                    ["data"] = {
                        ["name"] = args[1],
                        ["position"] = args[2]
                    }
                })
                step = step + 1
            elseif table.find(remote_record, self.Name) then
                table.insert(RecordMacroTable, {
                    ["step"] = step,
                    ["type"] = self.Name,
                    ["data"] = {
                        ["position"] = args[1].HumanoidRootPart.CFrame
                    }
                })
                step = step + 1
            end
        end
    end
    return old(self, ...)
end