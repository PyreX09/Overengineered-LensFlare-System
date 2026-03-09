# 🌟 Overengineered LensFlare System

A **highly customizable and unnecessarily optimized LensFlare System** for Roblox, built upon the original work by [@gluGPU](https://devforum.roblox.com/u/glugpu/summary).  
This version started as a **small modification…** and slowly turned into a **full-blown lens flare rendering system.**  
Because apparently, **a simple flare wasn't complicated enough.**

![Roblox](https://img.shields.io/badge/Roblox-Module-blue?logo=roblox)
![Status](https://img.shields.io/badge/Status-Active-success)
![Language](https://img.shields.io/badge/Lua-5.1%2B-yellow)
[![License](https://img.shields.io/badge/license-Educational%20%26%20Non--Commercial-blue)]([LICENSE](https://github.com/PyreX09/LensFlare?tab=License-1-ov-file))

---

## ⚠️ Warning

This project may contain:
- Excessive optimization
- Questionable engineering decisions
- Systems that absolutely did not need to exist
All for the purpose of **making a light flare look slightly cooler.**

If you came here for a **simple flare script,**  
you are in the **wrong place.**

---


## 📥 Installation

To download LensFlare, go to the [**Releases**](https://github.com/PyreX09/LensFlare/releases) page.
Download the module and drop it into **ReplicatedStorage.**

Then prepare yourself mentally for the amount of features it contains.

---

## ⚙️ Features

✅ Full **OOP System** for lens flare control  
✅ **Dynamic color blending** based on object color and attributes  
✅ Built-in **LOD (Level of Detail)** support for optimization  
✅ **Raycast-based visibility detection** for realistic light occlusion  
✅ Simple API — create and destroy flares with just a few lines  
✅ Designed for **Cameras, Attachments, or Parts**  
✅ Incremental Raycasting System  
✅ Adaptive Ray Budget (FPS-aware)  
✅ Per-frame Raycast Cost Limit  
✅ Smart Ray Offset Caching  
✅ Stable Occlusion & Alpha Blending  
✅ Sun Flare Priority Handling  

Additional things that probably didn't need to exist:  
🚀 Dynamic ray scaling based on distance  
🚀 Persistent ray state across frames  
🚀 FPS sampling system  
🚀 Fast occlusion fallback mode  
🚀 Attribute caching to reduce garbage collection  
  
Yes.  
  
All of this.  
  
For a **lensflare.**  

---

## 🚀 Usage Example

Surprisingly, using it is still simple for tool.

**Server:**
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local remote = ReplicatedStorage.LensFlare.RemoteEvent:WaitForChild("ToggleLight")

local on = false

script.Parent.Activated:Connect(function()
	on = not on

	script.Parent.Light.Shadow.Enabled = on
	script.Parent.Light.Light.Enabled = on
	script.Parent.Front.SurfaceLight.Enabled = on

	if on then
		script.Parent.Light.Sound:Play()
	else
		script.Parent.Light.Sound2:Play()
	end

	remote:FireAllClients(script.Parent, on)
end)
```


**Client:**
```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LensFlare = require(ReplicatedStorage.LensFlare.Modules.LensFlare)
local camera = workspace.CurrentCamera

local remote = ReplicatedStorage.LensFlare.RemoteEvent:WaitForChild("ToggleLight")

local flares = {}

remote.OnClientEvent:Connect(function(tool, on)
	if not tool then return end

	local handle = tool:FindFirstChild("Handle")
	if not handle then return end

	local att = handle:FindFirstChild("LensFlareAttachment")
	if not att then return end

	if not flares[att] then
		flares[att] = LensFlare.new(
			camera,
			att:GetAttribute("LensFlareStyle") or "Default",
			att
		)

		tool.AncestryChanged:Connect(function(_, parent)
			if not parent then
				if flares[att] then
					flares[att]:Destroy()
					flares[att] = nil
				end
			end
		end)
	end
	att:SetAttribute("LensFlareEnabled", on)
	flares[att].Enabled = on
end)
```

---

## 🧠 Developer Notes

Each flare runs on its own **RenderStep bind**, carefully optimized to avoid redundant updates.  

Raycasting is processed **incrementally per frame**, instead of firing everything at once.  
This prevents large ray bursts from causing sudden frame drops.  

Each flare also maintains its own **persistent ray state**, allowing smooth transitions when:

- ray counts change
- FPS fluctuates
- distance-based LOD kicks in

Ray offsets and filters are **aggressively cached** to minimize table allocation and garbage collection.

Fast occlusion mode automatically activates when FPS drops and **deactivates smoothly** to avoid visual flicker.

In other words:  
> the system does a lot of work  
> so your flare can look **slightly better**

---

## 👑 Special Thanks

[@gluGPU](https://devforum.roblox.com/u/glugpu/summary) : Creator of the original [LensFlare](https://create.roblox.com/store/asset/89532403908041/Lens-Flare-System)  
Without that system, this project would not exist.

Or maybe it would.  

But it would probably be **even more chaotic.**

---

## ❓ Q&A (Extended)

### Q: Why does this system exist?
**A:** Good question.

Originally, it started as a **small tweak** to the original LensFlare system.  

Then one thing led to another.  

And now you're reading documentation for a **lens flare framework.**  

---

### Q: Is this overkill?
**A:** Absolutely.  

---

### Q: Can I still use it normally?
**A:** Yes.  

Despite the absurd amount of internal systems, the API remains intentionally simple.  

Create flare → enable flare → destroy flare.

---

### Q: Why so many raycasts?
**A:** Because **realistic light occlusion looks cool.**  

Also because **I wanted to see how far I could push it.**   

---

### Q: Does incremental raycasting reduce accuracy?
**A:** No. 

Unless your definition of accuracy is  

"everything must happen in one frame".

---

### Q: What is `NUMBER_OF_RAYCASTS`?
**A:** This determines how many rays are shot to detect flare obstruction.  
- Lower values → faster performance, less accurate occlusion.  
- Higher values → slower performance, more precise flare visibility.

---

### Q: What does `RAYCAST_RADIUS` do?
**A:** It spreads out the rays around the flare source. Doesn’t affect performance, just changes flare obstruction detection area.

---

### Q: What is `RAY_PER_FRAME`?
**A:** It limits how many rays can be processed in a single frame.
- Prevents heavy raycasts from stalling the renderer
- Rays are continued in the next frame until completion
- Lower value → smoother FPS, slower occlusion update

---

### Q: How does `TRANSPARENCY_THRESHOLD` work?
**A:** Parts more transparent than this value won’t block lens flares.  
- 1 → even fully transparent parts block flares.  
- 0 → only opaque parts block flares.

---

### Q: How is sun flare strength determined?
**A:** Sun flare is based on:
- `SUN_ANGLE_THRESHOLD` → angle above horizon to start fading  
- `SUN_BRIGHTNESS_THRESHOLD` → Lighting.Brightness value for full strength  
- `SUN_EXPOSURE_ADJUSTMENT` → exposure change when looking directly at sun  
- `SUN_EXPOSURE_TIME` → duration of smooth exposure adjustment  

---

### Q: What is `LensFlareDistance`?
**A:** This is the **default maximum distance** at which a flare is visible.  
- Used when `LOD_ENABLED = false`.  
- Flare disappears if the camera is farther than this value.  
- Think of it as the **“always use this distance”** setting.

---

### Q: What does `LensFlareLOD` affect?
**A:** `LensFlareLOD` is a **multiplier for `LOD_SCALE`** used **when `LOD_ENABLED = true`**.  
- Maximum visible distance = `LOD_SCALE × LensFlareLOD`  
- Example:  
  - `LOD_SCALE = 200`  
  - `LensFlareLOD = 3`  
  - Maximum visible distance = 200 × 3 = 600 studs  
- Also affects **dynamic raycount & flare fading** based on distance.  

---

### Q: What is `MIN_RAY_RATIO`?
**A:** The minimum ratio of rays to keep during LOD adaptive scaling.
- Ensures that even at low FPS or far distances, at least this portion of rays are cast.
- Value between 0 and 1 (e.g., 0.2 → 20% of NUMBER_OF_RAYCASTS).

---

### Q: What is `TARGET_FPS`?
**A:** Desired framerate for adaptive LOD calculation.
- System adjusts ray count to keep performance near this FPS.
- Lower FPS than target → fewer rays → better performance.
- Higher FPS than target → more rays → better quality.

---

### Q: What is `FPS_SAMPLE_COUNT`?
**A:** Number of frames to average for FPS calculation.  
- Higher value → smoother FPS estimation, slower reaction to sudden frame drops.  
- Lower value → reacts quickly, but FPS value fluctuates more.  

---

### ⚡ TL;DR Comparison

| Setting                    | Used when    | Determines                       | Notes                                                          |
| -------------------------- | ------------ | -------------------------------- | -------------------------------------------------------------- |
| `LensFlareDistance`        | LOD_DISABLED | Max flare distance               | Default distance, flare always visible within this range       |
| `LensFlareLOD × LOD_SCALE` | LOD_ENABLED  | Max flare distance & dynamic LOD | Adjusts rays & fade, overrides LensFlareDistance for LOD logic |

---

### Q: What is `LensFlareStrength`?
**A:** Determines how visible/bright a flare is.  
- 0 → invisible  
- 1 → full brightness  

---

### Q: How does the system handle dynamic raycasts?
**A:** If LOD is enabled, the number of rays is recalculated based on distance.  
- Closer → more rays → accurate obstruction  
- Farther → fewer rays → performance optimized  

---

### Q: Can I make a sun lens flare?
**A:** The system already **creates the sun flare automatically**.  
It handles position, strength, and exposure based on Lighting and camera for you.  
You only need to mess with it if you wanna tweak the sun flare manually.

---

### Q: How does debug mode work?
**A:**  
If `DEBUG_MODE = true`, the console prints:
- Part name  
- Flare alpha  
- Number of rays hitting  
- Distance to camera  
- LOD recalculations  
- Re-emission events   
- FPS  

It’s useful for tweaking flare behavior during development.  

---

### Q: Does incremental raycasting reduce accuracy?

**A:** No it only reduces when rays are processed, not how many.  
Final occlusion result is the same, just achieved more efficiently.  

---

### Q: How does the system avoid flickering when ray count changes?

**A:** Ray state is preserved across frames, and alpha blending is smoothed.  
This prevents sudden jumps when FPS or distance changes rapidly.  

---

## 👑 Final Note

If you're wondering why this project exists:	

The answer is simple.

> **I got bored.**

And apparently that was enough to accidentally build  
a **lens flare rendering system.**

---


