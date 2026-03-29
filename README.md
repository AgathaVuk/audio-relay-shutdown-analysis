# Shutdown on Pop!_OS with Audiorelay: diagnosis & solution

## Problem 
Computer shuts down abruptly 1–2 minutes after starting AudioRelay streaming.

## Environment
- Pop!_OS 24.04 LTS
- Intel 5 Series + GTX 1650
- AudioRelay

-----

## Investigation

### CPU temperature
- Checked CPU temperature to rule out overheating.
1. `sensors` - verify the CPU heat.
- It was regular. I discarded the possibility.

### CPU stress
- Analysis of the CPU under maximmun stress to verify if the problem was the Multi-task.
1. `sudo apt update && sudo apt install stress -y` - Install a CPU stressing tool.
2. `stress --cpu 4 --timeout 120s` -  Run all 4 CPU cores at 100% stress for 2 minutes. If the power supply was the problem, it would shut down here.
- It didn't shut down, I discarted the hyphotesis.

### Pattern observation
- The computer stayed on all night without issues, but shut down minutes after I started using it.
- The shutdown only happened when I opened AudioRelay and played a video.
- I realized that the problem was with theAudiorelay.

### Logs collect
- Collected real-time logs to capture what happened before shutdown.
1. `journalctl -f -o short-full > ~/logs_audiorelay.txt` - Move logs to /logs_audiorelay.txt on short-full mode.
2. `flatpak run net.audiorelay.AudioRelay 2>&1 | tee -a ~/logs_audiorelay_app.txt` - Execute AudioRelay and save logs while show on screen.
- Both logs stopped exactly at "Audio pipeline is emitting values" with no error messages.
- The shutdown was so abrupt that even the kernel couldn't write the last log entries.

### Isolated the PipeWire
- Stop momentarily the PipeWire services to test if the problem is it.
1. `systemctl --user stop pipewire pipewire-pulse` - Desable PipeWire.
- Tried to watch a video with audilrelay open and the computer still shut down after 1-2 minutes.
2. `systemctl --user start pipewire pipewire-pulse` - Enable PipeWire after test.

### Analysis of the audio hardware
- Identify which audio controllers is on the system.
1. `lspci | grep -E "VGA|Audio"` - Show audio controllers.
   - lspci output:
     ```
     00:1b.0 Audio device: Intel Corporation 5 Series/3400 Series Chipset High Definition Audio (rev 06)
     01:00.0 VGA compatible controller: NVIDIA Corporation TU117 [GeForce GTX 1650] (rev a1)
     01:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)
     ```
- That showed me that my pc had two audio chips: I5(very old, 2010), NVIDIA (gtx 1650, modern).
- I realized the problem can be the old chip failing when used intensively.

### Audio redrect (solution)
1. `pactl list short sinks` - List all audio outputs and its names.
   - pactl output:
     ```
     149 alsa_output.pci-0000_00_1b.0.iec958-stereo PipeWire s32le 2ch 48000Hz SUSPENDED
     168 alsa_output.pci-0000_01_00.1.hdmi-stereo PipeWire s32le 2ch 48000Hz SUSPENDED
     ```
- NVIDIA audio output was available. 
3. `pactl set-default-sink "alsa_output.pci-0000_01_00.1.hdmi-stereo"` - Define HDMI output as NVIDIA as default.
4. `pactl get-default-sink` - verify what output was the default. 
   - Pactl output:
     ```
     alsa_output.pci-0000_01_00.1.hdmi-stereo
     ```
     
### Turn solution permanent
- Created the PulseAudio folder.
1. `mkdir -p ~/.config/pulse` - Created the foulder.
2. `echo "set-default-sink alsa_output.pci-0000_01_00.1.hdmi-stereo" >> ~/.config/pulse/default.pa` - Add the command to inicialization arquive.
