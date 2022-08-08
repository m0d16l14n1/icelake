# icelake

[![Gitter](https://badges.gitter.im/ICE-LAKE-HACKINTOSH-DEVELOPMENT/community.svg)](https://gitter.im/ICE-LAKE-HACKINTOSH-DEVELOPMENT/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## Graphics related issues

| Bug | Description/Details | Status |
| ------ | ------ | ------ |
| 1. 10-15 seconds black screen glitch after boot | [LINK](https://github.com/acidanthera/bugtracker/issues/1329) | [FIXED](https://github.com/acidanthera/WhateverGreen/pull/92) - [details](https://github.com/acidanthera/bugtracker/issues/1805)|
| 2. **No HDMI support** | [LINK](https://github.com/Ardentwheel/OpenCore-Hasee-X57S1/issues/3#issuecomment-711080776) | **NOT FIXED** |
| 3. Issues with “default” ig-platform-id value | With default value (injected by WEG) in most cases laptop can't properly wake from sleep (result is a black screen and no panic). Log is [here](https://github.com/m0d16l14n1/icelake-hackintosh/blob/main/Logs/%22Default%22%20WEG%20inject%20ig-platform-id%20wake-up%20issue%20(Link%20training%20problem%3F)/defaultwegframeFAILtoWAKE.txt) | WA > Check 0x8A510002 (it's default value from MacBookAir9,1 which is working properly) |
| 4. HiDPi issues | [LINK](https://github.com/Ardentwheel/OpenCore-Hasee-X57S1/issues/3#issuecomment-790013456) | FIXED/WA > add AAPL,GfxYTile to iGPU device properties |
| 5. Black screen after wake up with kernel panic after | [LINK](https://github.com/acidanthera/bugtracker/issues/1207) | FIXED/WA > add "-noDC9" boot-arg |
| 6. **Screen backlight dimming** | Some of Lenovo/Razer/HP laptops have that issue. Brightness is much lower than on Windows before a sleep-wake cycle. | **FIXED** Just use brightness fix from WEG |
| 7. Black screen bug with 4k internal screens | Some Dells have that issue, still unclear why it's happening, no logs found | Currently no WA available / No fix or patch |
| 8. Panic after waking from sleep with external screen connected (any panic related to DC6 screen state) | Seems that some Ice Lake laptops have issues with those states (DC6/DC9) | You can simply "disable" DC6 by using boot-arg "dc6config=0"** |

**That's the explanation from [0xFireWolf]: 
1) dc6config=0 will disable DC6. In essence, enableHWDC6() and disableHWDC6() become NOPs. i.e. They do nothing and simply return. 
2) dc6config=1 will enable DC6. The effect is the same as when the boot argument is not present. 
3) dc6config=5 will enable "DC6" but up to DC5. i.e. enableHWDC6() will write 0x1 to the low 2 bits of the register DC_STATE_EN.



## USB-TB (and video output related) issues and WA

| Bug | Description/Details | Status |
| ------ | ------ | ------ |
| 1. Thunderbolt 3 > no drivers or patches available | "No drivers" in System report, hotplug is not supported | Still working, if you plug device before boot |
| 2. USB mapping on macOS may / may not work properly due to TXHC | Randomly it will fail to map it properly (TXHC) | Better to map in Windows using [THAT](https://github.com/USBToolBox/tool) tool |
| 3. **Type-C to HDMI is not working on some of Ice Lake laptops (for example some of Dell laptops)** | Seems the issue appears on Ice Lake laptops with no Thunderbolt support. Log is [here](https://github.com/m0d16l14n1/icelake-hackintosh/blob/main/Logs/Type-C%20to%20HDMI%20issue%20(dongle%20video%20ouput)/igfbType-CToHDMI.txt) | **Currently no WA available / No fix or patch** | 

## Sleep/power issues / WA

| Bug | Description/Details | Status |
| ------ | ------ | ------ |
| 1. **Hibernation is not working at all** | Stuck on booting or garbled screen right after selecting "macOS" drive in Boot Picker (OC) | Behavior is seems to be the same as it's mentioned [here](https://dortania.github.io/hackintosh/updates/2021/04/24/rocket-lake.html) in **"Power Cycle" part** |
| 2. Sleep issues (wake-up problem) | Some Ice Lake machines have **AOAC enabled** (can’t be disabled in most part of laptops because of “locked” BIOS) | * |

***
1) Use daliansky patches/SSDTs [LINK](https://github.com/alkindivv/OC-Little-English/tree/main/OC-Little-English/01-About%20AOAC%20) > Isn't so stable: battery life is low, some machines can't wake even with that patches
2) Unlock BIOS settings > disable AOAC (Low power S0 idle or any S0ix stuff) > It's the most hard way, but the most "stable"
3) !!!Patch FACP table!!! (I have 1 positive result of that method) > It seems it's possible to do patching (disabling AOAC) with the help of that efi driver > [LINK](https://github.com/m0d16l14n1/Hasee-KingBook-X57S1/blob/master/Tools/AcpiPatcher.efi.zip). Patcher was found on some reddit [topic](https://www.reddit.com/r/Dell/comments/h0r56s/getting_back_s3_sleep_and_disabling_modern/). Credit for tool goes to [datasone](https://github.com/datasone)

It's still might be a *little* problem there: your OEM vendor (for example [Dell](https://www.dell.com/community/XPS/XPS-15-9570-BIOS-1-3-0-sleep-mode-gone/td-p/6131926))) might disable/remove S3 state/event from DSDT at all. So better to check it and add/patch it back. 
***

### Collection of kernel panics / wa

| Bug | Description/Details | Status |
| ------ | ------ | ------ |
| 1. EL[0] was invalidated!! | [LINK](https://github.com/acidanthera/bugtracker/issues/1343) | Need to enable GuC firmware loading > use device property for iGPU or boot-arg "igfxfw=2" |
| 2. AppleHDA panic if you connect external screen with speakers, which means no audio support for external screens | [1](https://github.com/acidanthera/bugtracker/issues/1616), [2](https://github.com/acidanthera/bugtracker/issues/1551), [3](https://github.com/acidanthera/bugtracker/issues/1283) | Add "No-gfx-hda" to HDEF (sound) device properties or [patch AppleALC yourself](https://github.com/acidanthera/bugtracker/issues/1283#issuecomment-824802110) or if you really want to use external sound, just disable AppleALC, enable patch for renaming HDAS to HDEF, after that you will be able to use DP/HDMI sound with AppleGFXHDA, but no analog sound will be available (speakers/headphones)|

### Thanks

* To all of [acidanthera] and [dortania] team members for all of their kexts/guides and etc 
* For maintaining this repo - [m0d16l14n1]
* [OC-little] for AOAC patches and guides
* [kingo132] for Ice Lake DBuf workaround
* [0xFireWolf] for multiple Ice Lake graphics fixes (CDCLK/DVMT and DBuf), backlight smooth transition and many more explanations about ICL framebuffer stuff
* Apple
* To all members of Ice Lake gitter chat for providing panics/info

[m0d16l14n1]: <https://github.com/m0d16l14n1>
[OC-little]: <https://github.com/daliansky/OC-little>
[acidanthera]: <https://github.com/acidanthera>
[dortania]: <https://github.com/dortania>
[0xFireWolf]: <https://github.com/0xFireWolf>
[kingo132]: <https://github.com/kingo132>
