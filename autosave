-- !
local httpService = game:GetService("HttpService")

local SaveManager = {} do
	SaveManager.Folder = "FluentSettings"
	SaveManager.Ignore = {}
	SaveManager.AutoLoadEnabled = false
	SaveManager.LastConfigFile = SaveManager.Folder .. "/settings/lastconfig.txt"
	SaveManager.AutoLoadStateFile = SaveManager.Folder .. "/settings/autoload_state.txt"

	SaveManager.Parser = {
		Toggle = {
			Save = function(idx, object)
				return { type = "Toggle", idx = idx, value = object.Value }
			end,
			Load = function(idx, data)
				if SaveManager.Options[idx] then 
					SaveManager.Options[idx]:SetValue(data.value)
				end
			end,
		},
		Slider = {
			Save = function(idx, object)
				return { type = "Slider", idx = idx, value = tostring(object.Value) }
			end,
			Load = function(idx, data)
				if SaveManager.Options[idx] then 
					SaveManager.Options[idx]:SetValue(data.value)
				end
			end,
		},
		Dropdown = {
			Save = function(idx, object)
				return { type = "Dropdown", idx = idx, value = object.Value, mutli = object.Multi }
			end,
			Load = function(idx, data)
				if SaveManager.Options[idx] then 
					SaveManager.Options[idx]:SetValue(data.value)
				end
			end,
		},
		Colorpicker = {
			Save = function(idx, object)
				return { type = "Colorpicker", idx = idx, value = object.Value:ToHex(), transparency = object.Transparency }
			end,
			Load = function(idx, data)
				if SaveManager.Options[idx] then 
					SaveManager.Options[idx]:SetValueRGB(Color3.fromHex(data.value), data.transparency)
				end
			end,
		},
		Keybind = {
			Save = function(idx, object)
				return { type = "Keybind", idx = idx, mode = object.Mode, key = object.Value }
			end,
			Load = function(idx, data)
				if SaveManager.Options[idx] then 
					SaveManager.Options[idx]:SetValue(data.key, data.mode)
				end
			end,
		},
		Input = {
			Save = function(idx, object)
				return { type = "Input", idx = idx, text = object.Value }
			end,
			Load = function(idx, data)
				if SaveManager.Options[idx] and type(data.text) == "string" then
					SaveManager.Options[idx]:SetValue(data.text)
				end
			end,
		},
	}

	function SaveManager:SetIgnoreIndexes(list)
		for _, key in next, list do
			self.Ignore[key] = true
		end
	end

	function SaveManager:SetFolder(folder)
		self.Folder = folder
		self:BuildFolderTree()
	end

	function SaveManager:Save(name)
		if (not name) then return false, "no config file is selected" end
		local fullPath = self.Folder .. "/settings/" .. name .. ".json"

		local data = { objects = {} }
		for idx, option in next, SaveManager.Options do
			if not self.Parser[option.Type] or self.Ignore[idx] then continue end
			table.insert(data.objects, self.Parser[option.Type].Save(idx, option))
		end	

		local success, encoded = pcall(httpService.JSONEncode, httpService, data)
		if not success then return false, "failed to encode data" end

		writefile(fullPath, encoded)
		writefile(self.LastConfigFile, name)
		return true
	end

	function SaveManager:Load(name)
		if (not name) then return false, "no config file is selected" end
		local file = self.Folder .. "/settings/" .. name .. ".json"
		if not isfile(file) then return false, "invalid file" end

		local success, decoded = pcall(httpService.JSONDecode, httpService, readfile(file))
		if not success then return false, "decode error" end

		for _, option in next, decoded.objects do
			if self.Parser[option.type] then
				task.spawn(function() self.Parser[option.type].Load(option.idx, option) end)
			end
		end
		return true
	end

	function SaveManager:IgnoreThemeSettings()
		self:SetIgnoreIndexes({ "InterfaceTheme", "AcrylicToggle", "TransparentToggle", "MenuKeybind" })
	end

	function SaveManager:BuildFolderTree()
		for _, path in next, { self.Folder, self.Folder .. "/settings" } do
			if not isfolder(path) then makefolder(path) end
		end
	end

	function SaveManager:RefreshConfigList()
		local list = listfiles(self.Folder .. "/settings")
		local out = {}
		for _, file in next, list do
			if file:sub(-5) == ".json" then
				local name = file:match("([^/\\]+)%.json$")
				if name and name ~= "options" then table.insert(out, name) end
			end
		end
		return out
	end

	function SaveManager:SetLibrary(library)
		self.Library = library
		self.Options = library.Options

		-- Đọc trạng thái AutoLoad sau khi Options đã có
		if isfile(self.AutoLoadStateFile) then
			local state = readfile(self.AutoLoadStateFile)
			self.AutoLoadEnabled = (state == "true")
		else
			self.AutoLoadEnabled = false
		end
	end

	function SaveManager:AutoLoadLastUsed()
		if self.AutoLoadEnabled and isfile(self.LastConfigFile) then
			local name = readfile(self.LastConfigFile)
			local success, err = self:Load(name)
			if success then
				warn("[SaveManager] Auto-loaded last config:", name)
			else
				warn("[SaveManager] Failed to auto-load last config:", err)
			end
		end
	end

	function SaveManager:BuildConfigSection(tab)
		assert(self.Library, "Must set SaveManager.Library")
		local section = tab:AddSection("Configuration")

		section:AddToggle("SaveManager_AutoLoad", {
			Title = "Auto Load Last Used Config",
			Default = self.AutoLoadEnabled,
			Callback = function(state)
				SaveManager.AutoLoadEnabled = state
				writefile(SaveManager.AutoLoadStateFile, state and "true" or "false")
			end
		})

		section:AddInput("SaveManager_ConfigName", { Title = "Config name" })
		section:AddDropdown("SaveManager_ConfigList", {
			Title = "Config list",
			Values = self:RefreshConfigList(),
			AllowNull = true
		})

		section:AddButton({ Title = "Create config", Callback = function()
			local name = SaveManager.Options.SaveManager_ConfigName.Value
			if name:gsub(" ","") == "" then
				return self.Library:Notify({Title = "Interface", Content = "Invalid config name", Duration = 5})
			end
			local success = self:Save(name)
			if success then
				SaveManager.Options.SaveManager_ConfigList:SetValues(self:RefreshConfigList())
				SaveManager.Options.SaveManager_ConfigList:SetValue(nil)
			end
		end})

		section:AddButton({ Title = "Load config", Callback = function()
			local name = SaveManager.Options.SaveManager_ConfigList.Value
			self:Load(name)
		end})

		section:AddButton({ Title = "Overwrite config", Callback = function()
			local name = SaveManager.Options.SaveManager_ConfigList.Value
			self:Save(name)
		end})

		section:AddButton({ Title = "Refresh list", Callback = function()
			SaveManager.Options.SaveManager_ConfigList:SetValues(self:RefreshConfigList())
		end})

		SaveManager:SetIgnoreIndexes({
			"SaveManager_ConfigList",
			"SaveManager_ConfigName"
		})
	end

	SaveManager:BuildFolderTree()
end

return SaveManager
