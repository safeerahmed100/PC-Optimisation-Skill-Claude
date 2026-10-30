# System audit commands

Give the user these blocks and ask them to paste the full output back. Tell them which need an elevated prompt (right-click Start → Terminal (Admin)). All commands here are read-only; none change the system.

## Contents
- Block 1: Hardware and OS
- Block 2: Memory speed
- Block 3: Storage health
- Block 4: Security and boot state
- Block 5: Power, startup, and background load
- Block 6: GPU and drivers
- Optional tools
- How to read the results

## Block 1: Hardware and OS (normal prompt)

```powershell
Get-CimInstance Win32_Processor | Select Name,NumberOfCores,NumberOfLogicalProcessors,MaxClockSpeed
Get-CimInstance Win32_BaseBoard | Select Manufacturer,Product
Get-CimInstance Win32_BIOS | Select SMBIOSBIOSVersion,Manufacturer,ReleaseDate
Get-CimInstance Win32_OperatingSystem | Select Caption,Version,BuildNumber,OSArchitecture
```

## Block 2: Memory speed

```powershell
Get-CimInstance Win32_PhysicalMemory | Select BankLabel,Manufacturer,PartNumber,@{n='GB';e={$_.Capacity/1GB}},Speed,ConfiguredClockSpeed
```

`Speed` is the module's rated speed; `ConfiguredClockSpeed` is what it actually runs at. If configured is far below rated (e.g., 4800 vs 6000), XMP/EXPO is probably off. Also confirm two sticks are in the correct slots for dual channel.

## Block 3: Storage health

```powershell
Get-PhysicalDisk | Select FriendlyName,MediaType,HealthStatus,@{n='SizeGB';e={[math]::Round($_.Size/1GB)}}
Get-Volume | Where DriveLetter | Select DriveLetter,FileSystemLabel,@{n='FreeGB';e={[math]::Round($_.SizeRemaining/1GB)}},@{n='SizeGB';e={[math]::Round($_.Size/1GB)}}
```

Flag: HealthStatus not Healthy, system or game drive over ~85–90% full, or the game installed on an HDD.

## Block 4: Security and boot state (elevated)

```powershell
Confirm-SecureBootUEFI
Get-Tpm | Select TpmPresent,TpmReady
Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard | Select VirtualizationBasedSecurityStatus,SecurityServicesRunning
```

VBS status: 0 = off, 1 = enabled but not running, 2 = running. `SecurityServicesRunning` containing 2 means HVCI (Memory Integrity) is running. Record the state; do not recommend changing it yet. See `faceit-safety.md` and the VBS entry in `advanced-windows.md`.

## Block 5: Power, startup, background load

```powershell
powercfg /getactivescheme
Get-CimInstance Win32_StartupCommand | Select Name,Command,Location
Get-Process | Sort WorkingSet -Descending | Select -First 15 Name,@{n='MB';e={[math]::Round($_.WorkingSet/1MB)}},CPU
```

## Block 6: GPU and drivers

```powershell
Get-CimInstance Win32_VideoController | Select Name,DriverVersion,DriverDate,CurrentRefreshRate,CurrentHorizontalResolution,CurrentVerticalResolution
nvidia-smi
```

`nvidia-smi` only exists on NVIDIA systems; if it errors, ignore it. For AMD/Intel, the first line is enough.

## Optional tools (ask the user to screenshot or export)

- **HWiNFO64** (sensors-only mode): CPU/GPU temps, clocks, power draw, throttling flags. Most useful logged during a benchmark run.
- **CapFrameX** or **PresentMon**: frametime capture for the benchmark.
- **LatencyMon**: only if stutter/audio crackle suggests DPC/ISR problems.
- **CrystalDiskInfo**: SMART detail if storage health looks off.

## How to read the results

Build a ranked findings list. Typical high-impact finds, roughly in order:

1. RAM running below rated speed, or single-channel.
2. Thermal throttling or very high temps (CPU near 95 °C+, GPU hot spot throttling).
3. Outdated BIOS/chipset/GPU drivers, or a generic Microsoft display driver in use.
4. Wrong monitor refresh rate selected in Windows.
5. Game on a nearly-full or unhealthy drive.
6. Heavy startup/background load (launchers, RGB suites, overlays, browser tabs).
7. Power plan limiting the CPU (laptops especially).

Only after these should anything in Tier 2/3 be considered.
