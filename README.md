-- Version 1.2

if myHero.charName ~= "LeeSin" then return end

require 'VPrediction'

local qRange, qDelay, qSpeed, qWidth = 1050, 0.25, 1400, 90

local allyMinions = {}
local lastTime, lastTimeQ, bonusDmg = 0, 0, 0
local qDmgs = {50, 80, 110, 140, 170}
local useSight, lastWard, targetObj, friendlyObj = nil, nil, nil, nil
local VP = nil

function OnLoad()
	HConfig = scriptConfig("[ 盲僧很忙 - 晴依汉化]", "LeeSinCombo")
	HConfig:addParam("scriptActive", "连招", SCRIPT_PARAM_ONKEYDOWN, false, 32)
	HConfig:addParam("insecMake", "回踢", SCRIPT_PARAM_ONKEYDOWN, false, 84)
	HConfig:addParam("harass", "骚扰", SCRIPT_PARAM_ONKEYDOWN, false, 71)
	HConfig:addParam("wardJump", "跳眼", SCRIPT_PARAM_ONKEYDOWN, false, 67)
	HConfig:addParam("wardJumpmax", "鼠标距离太远，则最大距离跳向鼠标", SCRIPT_PARAM_ONOFF, false)
	HConfig:addParam("drawInsec", "显示回踢线", SCRIPT_PARAM_ONOFF, false)
	HConfig:addParam("drawQ", "显示Q范围", SCRIPT_PARAM_ONOFF, false)
	HConfig:addParam("following", "连招时跟随鼠标", SCRIPT_PARAM_ONOFF, true)
	HConfig:addParam("useEvade", "使用免费Freaking Good Evade躲避", SCRIPT_PARAM_ONOFF, false)
	
	for i=1, heroManager.iCount do
		local enemy = heroManager:GetHero(i)
		if enemy.team ~= myHero.team then
			HConfig:addParam("focus"..enemy.charName, "Ult on "..enemy.charName, SCRIPT_PARAM_ONOFF, true)
		end
	end
	
	allyMinions = minionManager(MINION_ALLY, 1050, myHero, MINION_SORT_HEALTH_DES)
	
	VP = VPrediction()
	
	HConfig:permaShow("scriptActive")
	HConfig:permaShow("insecMake")
	HConfig:permaShow("harass")
	HConfig:permaShow("wardJump")
	
	PrintChat(" >> Lee Sin - The Master of InSec by HunteR")
	PrintChat(" >> Lee Sin - GAD汉化组汉化，祝您五杀！")
end

function OnTick()
	if myHero.dead then
		return
	end
	
	local SIGHTlot = GetInventorySlotItem(2049)
	local SIGHTREADY = (SIGHTlot ~= nil and myHero:CanUseSpell(SIGHTlot) == READY)
	local SIGHTlot2 = GetInventorySlotItem(2045)
	local SIGHTREADY2 = (SIGHTlot2 ~= nil and myHero:CanUseSpell(SIGHTlot2) == READY)
	local SIGHTlot3 = GetInventorySlotItem(3340)
	local SIGHTREADY3 = (SIGHTlot3 ~= nil and myHero:CanUseSpell(SIGHTlot3) == READY)
	local SIGHTlot4 = GetInventorySlotItem(2044)
	local SIGHTREADY4 = (SIGHTlot4 ~= nil and myHero:CanUseSpell(SIGHTlot4) == READY)
	local SIGHTlot5 = GetInventorySlotItem(3361)
	local SIGHTREADY5 = (SIGHTlot5 ~= nil and myHero:CanUseSpell(SIGHTlot5) == READY)
	local SIGHTlot6 = GetInventorySlotItem(3362)
	local SIGHTREADY6 = (SIGHTlot6 ~= nil and myHero:CanUseSpell(SIGHTlot6) == READY)
	local SIGHTlot7 = GetInventorySlotItem(3154)
	local SIGHTREADY7 = (SIGHTlot7 ~= nil and myHero:CanUseSpell(SIGHTlot7) == READY)
	local SIGHTlot8 = GetInventorySlotItem(3160)
	local SIGHTREADY8 = (SIGHTlot8 ~= nil and myHero:CanUseSpell(SIGHTlot8) == READY)
	
	useSight = nil
	if SIGHTREADY then
		useSight = SIGHTlot
	elseif SIGHTREADY2 then
		useSight = SIGHTlot2
	elseif SIGHTREADY7 then
		useSight = SIGHTlot7
	elseif SIGHTREADY8 then
		useSight = SIGHTlot8
	elseif SIGHTREADY3 then
		useSight = SIGHTlot3
	elseif SIGHTREADY5 then
		useSight = SIGHTlot5
	elseif SIGHTREADY6 then
		useSight = SIGHTlot6
	elseif SIGHTREADY4 then
		useSight = SIGHTlot4
	end
	
	bonusDmg = myHero.addDamage * 0.90
	
	local target = GetTarget()
	if target ~= nil then
		if string.find(target.type, "Hero") and target.team ~= myHero.team then
			targetObj = target
		elseif target.team == myHero.team then
			friendlyObj = target
		end
	end
	
	if HConfig.insecMake then
		if insec() then return end
	end
	
	if HConfig.scriptActive or HConfig.insecMake then
		local inseca = nil
		if HConfig.insecMake then inseca = targetObj end
		combo(inseca)
		return
	end
	
	if HConfig.wardJump then
		wardJump()
		return
	end
	
	if HConfig.harass then
		harass()
	end
end

function harass()
	for i=1, heroManager.iCount do
		local target = heroManager:GetHero(i)
		if ValidTarget(target, 1050) then
			if myHero:CanUseSpell(_Q) == READY then
				if myHero:GetSpellData(_Q).name == "BlindMonkQOne" then
					local CastPosition,  HitChance,  Position = VP:GetLineCastPosition(target, qDelay, qWidth, qRange, qSpeed, myHero, true)
					if HitChance >= 2 then
						CastSpell(_Q, CastPosition.x, CastPosition.z)
						return
					end
				elseif targetHasQ(target) then
					if myHero:CanUseSpell(_W) == READY and myHero:GetSpellData(_W).name == "BlindMonkWOne" and enemiesAround(1500) == 1 and myHero.mana >= 80 and (myHero.health / myHero.maxHealth) > 0.5 then
						allyMinions:update()
						for i, minion in pairs(allyMinions.objects) do
							if minion ~= nil and minion.valid then
								local distMinionTarget = GetDistance(target, minion)
								if distMinionTarget > 450 and distMinionTarget < 650 then
									CastSpell(_Q)
									return
								end
							end
						end
					end
				end
			end
			
			if myHero:CanUseSpell(_Q) ~= READY and myHero:CanUseSpell(_E) == READY and myHero:GetSpellData(_E).name == "BlindMonkEOne" and enemiesAround(375) >= 1 and myHero.mana >= 100 then
				CastSpell(_E)
				return
			end
			
			if myHero:CanUseSpell(_Q) ~= READY and myHero:CanUseSpell(_W) == READY and myHero:GetSpellData(_W).name == "BlindMonkWOne" and enemiesAround(300) >= 1 then
				allyMinions:update()
				local maxDist = 0
				local miniona = nil
				for i, minion in pairs(allyMinions.objects) do
					if minion ~= nil and minion.valid then
						local distMinionTarget = GetDistance(target, minion)
						if distMinionTarget > 450 and distMinionTarget < 650 and distMinionTarget > maxDist then
							miniona = minion
							maxDist = distMinionTarget
						end
					end
				end
				
				if miniona ~= nil then
					CastSpell(_W, miniona)
				end
			end
		end
	end
end

function enemiesAround(range)
	local playersCount = 0
	for i=1, heroManager.iCount do
		local target = heroManager:GetHero(i)
		if ValidTarget(target, range) then
			playersCount = playersCount + 1
		end
	end
	return playersCount
end

function wardJump()
	if myHero:CanUseSpell(_W) == READY and myHero:GetSpellData(_W).name == "BlindMonkWOne" then
		if lastTime > (GetTickCount() - 1000) then
			if (GetTickCount() - lastTime) >= 10 then
				CastSpell(_W, lastWard)
			end
		elseif useSight ~= nil then
			local wardX = mousePos.x
			local wardZ = mousePos.z
			if HConfig.wardJumpmax then
				local distanceMouse = GetDistance(myHero, mousePos)
				if distanceMouse > 600 then
					wardX = myHero.x + (600 / distanceMouse) * (mousePos.x - myHero.x)
					wardZ = myHero.z + (600 / distanceMouse) * (mousePos.z - myHero.z)
				end
			end
			
			CastSpell(useSight, wardX, wardZ)
		end
	end
end

function OnCreateObj(object)
	if myHero.dead then return end
	
	if HConfig.wardJump or HConfig.insecMake then
		if object ~= nil and object.valid and (object.name == "VisionWard" or object.name == "SightWard") then
			lastWard = object
			lastTime = GetTickCount()
		end
	end
end

function insec()
	if myHero:CanUseSpell(_R) == READY and friendlyObj ~= nil and targetObj ~= nil and friendlyObj.valid and targetObj.valid and ValidTarget(targetObj) then
		if myHero:GetDistance(targetObj) < 375 then
			local dPredict = GetDistance(targetObj, myHero)
			
			local xE = myHero.x + ((dPredict + 500) / dPredict) * (targetObj.x - myHero.x)
			local zE = myHero.z + ((dPredict + 500) / dPredict) * (targetObj.z - myHero.z)
			
			local positiona = {}
			positiona.x = xE
			positiona.z = zE
			
			local newDistance = GetDistance(friendlyObj, targetObj) - GetDistance(friendlyObj, positiona)
			if newDistance > 0 and (newDistance / 500) > 0.7 then
				CastSpell(_R, targetObj)
				return true
			end
		end
		
		if myHero:CanUseSpell(_W) == READY and myHero:GetSpellData(_W).name == "BlindMonkWOne" then
			if lastTime > (GetTickCount() - 1000) then
				if (GetTickCount() - lastTime) >= 10 then
					CastSpell(_W, lastWard)
					return true
				end
			elseif useSight ~= nil then
				targetObj2 = targetObj
				
				local wardDistance = 300
				local dPredict = GetDistance(targetObj2, friendlyObj)
				local xE = friendlyObj.x + ((dPredict + wardDistance) / dPredict) * (targetObj2.x - friendlyObj.x)
				local zE = friendlyObj.z + ((dPredict + wardDistance) / dPredict) * (targetObj2.z - friendlyObj.z)
				
				local positiona = {}
				positiona.x = xE
				positiona.z = zE
				if GetDistance(myHero, positiona) < 600 then
					CastSpell(useSight, xE, zE)
					return true
				end
			end
		end
	end
	
	return false
end

function OnProcessSpell(unit, spell)
	if unit.name == myHero.name then
		if spell.name == "BlindMonkQOne" then
			lastTimeQ = GetTickCount()
		end
	end
end

function combo(inseca)
	local QREADY = (myHero:CanUseSpell(_Q) == READY)
	local WREADY = (myHero:CanUseSpell(_W) == READY)
	local EREADY = (myHero:CanUseSpell(_E) == READY)
	local RREADY = (myHero:CanUseSpell(_R) == READY)
	
	local TIAMATSlot = GetInventorySlotItem(3077)
	local TIAMATREADY = (TIAMATSlot ~= nil and myHero:CanUseSpell(TIAMATSlot) == READY)
	local HYDRASlot = GetInventorySlotItem(3074)
	local HYDRAREADY = (HYDRASlot ~= nil and myHero:CanUseSpell(HYDRASlot) == READY)
	local BLADESLot = GetInventorySlotItem(3153)
	local BLADEREADY = (BLADESLot ~= nil and myHero:CanUseSpell(BLADESLot) == READY)
	local BILGESlot = GetInventorySlotItem(3144)
	local BILGEREADY = (BILGESlot ~= nil and myHero:CanUseSpell(BILGESlot) == READY)
	local RANDSlot = GetInventorySlotItem(3143)
	local RANDREADY = (RANDSlot ~= nil and myHero:CanUseSpell(RANDSlot) == READY)
	local bladeaSlot = GetInventorySlotItem(3142)
	local bladeaaREADY = (bladeaSlot ~= nil and myHero:CanUseSpell(bladeaSlot) == READY)

	local focusEnemy = nil
	local minimumHit = -1
	local lowPriority = false
	
	local rangeFocus = 400
	if QREADY then
		rangeFocus = 1000
	end
	
	local insecOk = false
	if inseca ~= nil and inseca.valid and ValidTarget(inseca) then
		focusEnemy = inseca
		insecOk = true
	else
		for i=1, heroManager.iCount do
			local target = heroManager:GetHero(i)
			if ValidTarget(target, rangeFocus) then
				local dmg = getDmg("Q", target, myHero)
				local hits = (target.health / dmg)
				if minimumHit == -1 or (hits < minimumHit and HConfig["focus"..target.charName]) or hits <= 1.05 or (not lowPriority and minimumHit > 1.05) then
					focusEnemy = target
					minimumHit = hits
					lowPriority = HConfig["focus"..target.charName]
				end
			end
		end
	end
	
	if focusEnemy ~= nil then
		if QREADY then
			if myHero:GetSpellData(_Q).name == "BlindMonkQOne" then
				local CastPosition,  HitChance,  Position = VP:GetLineCastPosition(focusEnemy, qDelay, qWidth, qRange, qSpeed, myHero, true)
				if HitChance >= 2 then
					CastSpell(_Q, CastPosition.x, CastPosition.z)
					return
				end
			elseif targetHasQ(focusEnemy) and (myHero:GetDistance(focusEnemy) > 500 or insecOk or (getQDmg(focusEnemy, 0) + getDmg("AD", focusEnemy, myHero)) > focusEnemy.health or (GetTickCount() - lastTimeQ) > 2500) then
				CastSpell(_Q)
				return
			end
		end
		
		if EREADY then
			if myHero:GetSpellData(_E).name == "BlindMonkEOne" and enemiesAround(375) >= 1 then
				CastSpell(_E)
				return
			elseif enemiesAround(500) >= 1 and myHero:GetSpellData(_E).name ~= "BlindMonkEOne" then
				CastSpell(_E)
				return
			end
		end
		
		if RREADY and HConfig["focus"..focusEnemy.charName] and myHero:GetDistance(focusEnemy) <= 375 then
			local prociR = getDmg("R", focusEnemy, myHero) / focusEnemy.health
			local healthLeft = focusEnemy.health - getDmg("R", focusEnemy, myHero)
			
			if (prociR > 1 and prociR < 2.5) or (getQDmg(focusEnemy, healthLeft) > healthLeft and targetHasQ(focusEnemy) and QREADY) then
				CastSpell(_R, focusEnemy)
				return
			end
		end
		
		if WREADY and not insecOk then
			local yourLifePercent = myHero.health / myHero.maxHealth
			if myHero:GetSpellData(_W).name == "BlindMonkWOne" then
				if yourLifePercent < 0.4 then
					CastSpell(_W, myHero)
					return
				end
			elseif yourLifePercent < 0.6 then
				CastSpell(_W)
				return
			end
		end
		
		if BILGEREADY and myHero:GetDistance(focusEnemy) < 450 then
			CastSpell(BILGESlot, focusEnemy)
			return
		end
		
		if BLADEREADY and myHero:GetDistance(focusEnemy) < 450 then
			CastSpell(BLADESLot, focusEnemy)
			return
		end
		
		if TIAMATREADY and enemiesAround(350) >= 1 then
			CastSpell(TIAMATSlot)
			return
		end
		
		if HYDRAREADY and (enemiesAround(350) >= 2 or (getDmg("AD", focusEnemy, myHero) < focusEnemy.health and enemiesAround(350) == 1)) then
			CastSpell(HYDRASlot)
			return
		end
		
		if RANDREADY and enemiesAround(450) >= 1 then
			CastSpell(RANDSlot)
			return
		end
		
		if canAutoMove() then
			myHero:Attack(focusEnemy)
			return
		end
	end
	
	if HConfig.following and canAutoMove() then
		myHero:MoveTo(mousePos.x, mousePos.z)
	end
end

function targetHasQ(target)
	local dd = false
	for b=1, target.buffCount do
		local buff = target:getBuff(b)
		if buff.valid and (buff.name == "BlindMonkQOne" or buff.name == "blindmonkqonechaos") and (buff.endT - GetGameTimer()) >= 0.3 then
			dd = true
			break
		end
	end
	
	return dd
end

function getQDmg(target, health)
	local dmg = 0
	local qDMG = 0
	if myHero:CanUseSpell(_Q) == READY then
		local spellQ = myHero:GetSpellData(_Q)
		if spellQ.name == "BlindMonkQOne" then
			qDMG = qDmgs[spellQ.level] + bonusDmg
		else
			local dmgHealth = (target.maxHealth - target.health) * 0.08
			if health > 0 then
				dmgHealth = (target.maxHealth - health) * 0.08
			end
			qDMG = qDmgs[spellQ.level] + bonusDmg + dmgHealth
		end
	end
	
	if qDMG > 0 then
		dmg = myHero:CalcDamage(target, qDMG)
	end
	
	return dmg
end

function canAutoMove()
	local linea = nil
	if HConfig.useEvade then
		local file = io.open(SCRIPT_PATH.."movementblock.txt", "r")
		if file ~= nil then
			for line in file:lines() do linea = line break end
			file:close()
		end
	end
	
	if linea == 1 then
		return false
	else
		return true
	end
end

function DrawLine3Dcustom(x1, y1, z1, x2, y2, z2, width, color)
    local p = WorldToScreen(D3DXVECTOR3(x1, y1, z1))
    local px, py = p.x, p.y
    local c = WorldToScreen(D3DXVECTOR3(x2, y2, z2))
    local cx, cy = c.x, c.y
    DrawLine(cx, cy, px, py, width or 1, color or 4294967295)
end

function OnDraw()
	if HConfig.drawQ then
		DrawCircle(myHero.x, myHero.y, myHero.z, 1050, 0x25de69)
	end
	
	local QREADY = (myHero:CanUseSpell(_Q) == READY)
	local WREADY = (myHero:CanUseSpell(_W) == READY)
	local EREADY = (myHero:CanUseSpell(_E) == READY)
	local RREADY = (myHero:CanUseSpell(_R) == READY)
	local spellQ = myHero:GetSpellData(_Q)
	
	if RREADY and WREADY then
		if useSight ~= nil then
			local validTargets = 0
			if targetObj ~= nil and targetObj.valid and ValidTarget(targetObj) then
				DrawCircle(targetObj.x, targetObj.y, targetObj.z, 70, 0x00CC00)
				validTargets = validTargets + 1
			end
			
			if friendlyObj ~= nil and friendlyObj.valid then
				DrawCircle(friendlyObj.x, friendlyObj.y, friendlyObj.z, 70, 0x00CC00)
				validTargets = validTargets + 1
			end
			
			if validTargets == 2 and HConfig.drawInsec then
				local dPredict = GetDistance(targetObj, friendlyObj)
				local rangeR = 300
				if myHero:GetDistance(targetObj) <= 1100 then
					rangeR = 800
				end
				local xQ = targetObj.x + (rangeR / dPredict) * (friendlyObj.x - targetObj.x)
				local zQ = targetObj.z + (rangeR / dPredict) * (friendlyObj.z - targetObj.z)
				
				local positiona = {}
				positiona.x = xQ
				positiona.z = zQ
				
				DrawLine3Dcustom(targetObj.x, targetObj.y, targetObj.z, positiona.x, targetObj.y, positiona.z, 2)
			end
		end
	end
	
	local TIAMATSlot = GetInventorySlotItem(3077)
	local TIAMATREADY = (TIAMATSlot ~= nil and myHero:CanUseSpell(TIAMATSlot) == READY)
	local HYDRASlot = GetInventorySlotItem(3074)
	local HYDRAREADY = (HYDRASlot ~= nil and myHero:CanUseSpell(HYDRASlot) == READY)
	
	for i=1, heroManager.iCount do
		local enemy = heroManager:GetHero(i)
		if ValidTarget(enemy) then
			local tempHealth = enemy.health - getDmg("AD", enemy, myHero)
			
			if QREADY then
				tempHealth = tempHealth - myHero:CalcDamage(enemy, (qDmgs[spellQ.level] + bonusDmg))
			end
			
			if RREADY and HConfig["focus"..enemy.charName] then
				tempHealth = tempHealth - getDmg("R", enemy, myHero)
			end
			
			if EREADY then
				tempHealth = tempHealth - getDmg("E", enemy, myHero)
			end
			
			if QREADY then
				tempHealth = tempHealth - myHero:CalcDamage(enemy, (qDmgs[spellQ.level] + bonusDmg + ((enemy.maxHealth - tempHealth) * 0.08)))
			end
			
			if tempHealth < 0 then
				DrawText3D(tostring("快点快点打死他！"), enemy.x, enemy.y, enemy.z, 20, RGB(222, 245, 15), true)
			end
		end
	end
end
