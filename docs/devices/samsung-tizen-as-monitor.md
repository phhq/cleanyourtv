# Samsung Tizen as a dumb monitor: the settings sweep, since nothing can be disabled

A Samsung TV runs Tizen, and retail Tizen has no shell: no `adb`, no `pm disable-user`, no
third-party launcher, nothing in `tools/` applies. See [samsung-tizen.md](samsung-tizen.md) for the
ports a 2021 set answers on. So the Smart Hub cannot be removed. What can be done is to make the TV
boot straight into one HDMI input and never show the Hub, the ads or the "set as default" nags
unless someone presses Home on purpose. For a set used as a PC or Mac monitor that is the whole job.

Menu paths below are for 2021-2022 sets (Tizen 6, `Settings → General → ...`). On 2023 and later
sets the same items sit under `Settings → All Settings → General & Privacy → ...`, and on 2018-2020
sets a few of them under `Settings → Support`. The names of the toggles are the same across years;
if a path here does not match, use the search box at the top of the Settings menu.

## 1. Boot into HDMI, never into the Hub

| Setting | Path | Set to |
|---|---|---|
| Autorun Smart Hub | General → Smart Features (2023+: General & Privacy → Start Screen Options) | Off |
| Autorun Last App | same menu | Off |
| Auto Update apps | Home → Apps → Settings (gear) → Auto Update | Off |
| Software auto update | Support → Software Update → Auto Update | Off, so it stops rebooting into the Hub to announce itself |

With the first two off, power-on lands on whatever source was active at power-off. Pick the HDMI
input once and the TV stays there.

## 2. Kill the ads and the data collection

All of these are under `General → Terms & Privacy` (2023+: `General & Privacy → Terms & Privacy`;
2018-2020: `Support → Terms & Policy`):

- **Interest-Based Advertisements Service**: off. This is the toggle behind the sponsored tiles.
- **Viewing Information Services**: off. This is the content-recognition (ACR) that fingerprints
  whatever is on screen, including your desktop.
- **Voice Recognition Services**: off.
- Any **Bixby** / voice assistant item under `General → Voice`: off, and skip the Bixby setup if the
  TV offers it.
- `General → System Manager → Notifications` (where present): off, so nothing pops over your work.
- `General → Smart Features → Autorun Samsung TV Plus` (where present): off.

The ad toggle also governs the small Hub pop-up that appears on top of HDMI on some firmware; with
it off the pop-up does not appear.

## 3. Make the input behave like a monitor

| Setting | Path | Set to | Why |
|---|---|---|---|
| Device type of the HDMI input | Source → highlight the input → Edit (up arrow) → PC | PC | Enables 4:4:4 chroma, drops the TV's motion and sharpening processing, and lets the TV use its PC picture handling, so text renders cleanly |
| Input Signal Plus | General → External Device Manager → Input Signal Plus | On for that HDMI | Full-bandwidth HDMI; needed for 4K at 60 Hz with full chroma and for 120 Hz where the set has it |
| Game Mode | General → External Device Manager → Game Mode Settings | Auto or On | Lowest input lag. Some sets choose this for you once the input is labelled PC |
| Picture Size | Picture → Picture Size Settings | 16:9 Standard, Fit to Screen: On | 1:1 pixel mapping, no overscan |
| Anynet+ (HDMI-CEC) | General → External Device Manager → Anynet+ (HDMI-CEC) | Off | CEC is what switches sources behind your back and wakes or sleeps the TV with another device. A PC or Mac does not send CEC without an adapter, so there is nothing to lose |
| Auto Power Off | General → Power and Energy Saving (older: Eco Solution) | Off, or the longest value | Stops the TV switching itself off after hours without a remote press, which on a monitor is every day |
| Universal Remote Setup | General → External Device Manager | Skip or Cancel any prompt | The "new device detected, set it up?" prompt is this feature. Decline it once per input |
| Ambient / Art mode | Ambient button | Leave off | It only starts from its own button |

## 4. The remote problem

Samsung remotes talk to the TV two ways:

- **The Smart Remote (the small Bluetooth one, solar or AAA).** Once paired it sends every key over
  Bluetooth to its own TV only, except **Power**, which it also sends over infrared so it can wake a
  TV whose radio is asleep. If the remote has lost its pairing, it falls back to infrared for a few
  basic keys too.
- **The plain black remote**, where supplied, is infrared only, and every Samsung TV within line of
  sight answers it. There is no code set to change, unlike some other brands.

So the crosstalk you see is infrared: Power certainly, and more if a remote is unpaired. Three ways
to stop a monitor TV answering the living-room remote, in order of effort:

1. **Line of sight.** Infrared needs it. If the monitor TVs cannot see the living-room sofa, their
   receivers never hear that remote.
2. **Tape the infrared window.** It is the small dark window on the bottom bezel, usually next to
   the standby LED and the TV Controller button; a strip of black electrical tape over it blinds the
   TV to every remote while leaving Bluetooth working for your own paired Smart Remote. Test with the
   remote at arm's length first, then move the tape until Power stops working. This is reversible and
   does nothing to the firmware. There is no supported setting that turns the receiver off; the
   hotel-mode and factory service menus have one, but a wrong move in those menus can leave the set
   in a state that needs a service call, so this guide does not use them.
3. **Put the monitor TVs' remotes in a drawer** and drive them from the computer, below.

### The button on the TV

Every recent Samsung TV has one physical control, the **TV Controller**: a single button or small
jog stick under the Samsung logo on the bottom bezel, or on the back lower right. Press it once when
the set is off to turn it on. Press it once when the set is on and a small menu appears; repeated
short presses move between Power Off, Source and Return (some sets also show Settings), and a long
press selects. Source then steps through the inputs. It is slow, but it is the manual source switch
you were looking for, and it does not need any remote.

## 5. Control from the computer instead of a remote

The network remote the [recon page](samsung-tizen.md) found on port 8002 is enough to power the TV
on and off and to change input from a terminal on the PC or Mac it is attached to. The TV asks
"Allow?" once per computer; after that the token is saved.

Prerequisites on the TV: `General → Network → Expert Settings → Power On with Mobile` (also called
IP Remote): On. Wired Ethernet makes wake-on-LAN reliable; over Wi-Fi it works on most sets but not
all, since the Wi-Fi radio powers down in standby.

```sh
pip install samsungtvws wakeonlan
```

```python
# tv.py  <ip>  <mac>  on|off|source
import sys
from samsungtvws import SamsungTVWS
from wakeonlan import send_magic_packet

ip, mac, action = sys.argv[1:4]
tv = SamsungTVWS(host=ip, port=8002, token_file="tizen-token.txt")

if action == "on":
    send_magic_packet(mac)            # wakes the set; the token keeps the Allow prompt away
elif action == "off":
    tv.send_key("KEY_POWER")          # toggles, so only send it when the TV is on
elif action == "source":
    tv.send_key("KEY_SOURCE")         # opens the source bar; follow with KEY_RIGHT / KEY_ENTER
    tv.send_key("KEY_RIGHT")
    tv.send_key("KEY_ENTER")
```

Direct keys `KEY_HDMI1` to `KEY_HDMI4` exist and work on some years and are ignored on others,
which is why the example steps through the source bar instead. The TV's IP and MAC are under
`General → Network → Network Status`; give the TV a DHCP reservation so the IP does not move.
Bind the script to a hotkey, or to the computer's sleep and wake hooks, and the TV behaves like a
monitor that follows its computer.

## 6. Two network choices

Both work; pick one per TV.

- **No network at all.** `General → Network → Open Network Settings → forget the Wi-Fi network`, or
  just never join one. No ads, no telemetry, no update reboots, no Hub content, and nothing to block.
  The only cost is that section 5 does not work and the Hub shows a "not connected" card if anyone
  presses Home. For a monitor this is the simplest and most thorough option.
- **Network, with blocking at the router.** Keep Wi-Fi for section 5 and have the router or a
  Pi-hole refuse the TV's ad and reporting hosts. The usual Samsung lists cover `samsungads.com`,
  `samsungacr.com`, `samsungotn.net` (firmware) and `samsungcloudsolution.com` among others; take
  a maintained Samsung blocklist rather than typing hosts by hand, since they change by year and
  region. Watch that the block does not also stop the one HTTP API on 8001/8002, which is local and
  needs nothing from the internet.

## 7. Order of work, per TV

1. Settings sweep: sections 1 and 2, then 3. Ten minutes with the remote, once.
2. Decide on network (section 6). If you keep it, set up the script (section 5) and confirm on, off
   and source each work from the computer.
3. Only then deal with the remote (section 4): tape the window, or leave it if the line-of-sight test
   already passes.
4. Power the TV off and on from the wall. It should come back on the HDMI input with the computer's
   desktop and nothing else. If the Hub appears, Autorun Smart Hub did not take; check it again.

## What this does not do

- Nothing is uninstalled or disabled. Pressing Home still opens the Hub, with its apps and its
  "not connected" or sponsored row, until you press Return. There is no way around that on Tizen
  without developer firmware.
- A firmware update can reset Terms & Privacy toggles or re-enable Autorun. Repeat section 2 after
  any update, which is one more reason to turn automatic updates off or to keep the set offline.
- Paths and names were checked against 2021-2023 US sets. Other years and regions move items
  between `General`, `Support` and `General & Privacy`, so treat the paths as a guide and the toggle
  names as the thing to find.
