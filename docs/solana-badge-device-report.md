# Connected Badge / Solana Scavenger Hunt Device Report

Date observed: 2026-09-19  
Host: macOS on Apple Silicon MacBook Pro  
Primary serial device: `/dev/cu.usbmodem101`

## Executive Summary

I found one notable connected USB device that looks like badge hardware: an Espressif USB Serial/JTAG interface exposed over USB. This is consistent with an ESP32-S3-based badge or development board.

After probing the serial console, the device identified itself as a badge with a `badge>` command prompt and Hack the North branding. Its installed application list includes a Solana-related app:

```text
blindbox          SOL  Solana Scavenger Hunt
```

The badge filesystem also contains a Solana data file:

```text
/littlefs/solana.json
```

That file contains a Solana public key, an empty item inventory, and a plaintext secret key. The private key is intentionally redacted from this report. The public key had no visible funds or token accounts on the public Solana RPC checks I ran.

Overall assessment: this appears to be a Hack the North event badge or badge firmware running on ESP32-S3 badge hardware, with a bundled "Solana Scavenger Hunt" application and an on-device Solana wallet/state file. It is related to Solana, but not currently holding any public-chain assets that I could see.

## USB Device Inventory

The notable external USB device found in the macOS USB registry was:

```text
USB Product Name: USB JTAG_serial debug unit
USB Product String: USB JTAG/serial debug unit
USB Vendor Name: Espressif
Vendor ID: 0x303a
Product ID: 0x1001
Serial Number: E8:3D:C1:23:26:24
Location ID: 0x00100000
USB Address: 1
Speed: 12 Mb/s
Power allocation: 500 mA
```

macOS exposed the device as:

```text
/dev/cu.usbmodem101
/dev/tty.usbmodem101
```

Other serial devices visible on the system included:

```text
/dev/cu.Bluetooth-Incoming-Port
/dev/cu.GrindFuel
/dev/cu.debug-console
/dev/cu.usbmodem101
```

The badge-like device is `/dev/cu.usbmodem101`, because it appeared at the time of inspection and matched the Espressif USB JTAG/serial interface.

## Why This Looked Like Solana Badge Hardware

The public Solana DEF CON badge documentation describes an ESP32-S3 badge. The hardware walkthrough lists:

- ESP32-S3-WROOM-1-N16R8 module
- 2.8 inch SPI LCD
- Six buttons via I2C expander
- Stereo PDM microphones
- Addressable RGB LEDs
- NXP SE050 secure element
- USB-C connectivity
- Li-Po battery charging
- Expansion header

The Solana badge documentation also describes native USB / USB-Serial-JTAG behavior. Public Espressif documentation identifies `303a:1001` as the ESP32 USB JTAG/serial debug unit, which is exactly what macOS reported.

Relevant public references:

- Solana Badge hardware walkthrough: https://solanadefcon.com/
- SkyRizz Solana DEF CON badge page: https://skyrizz.com/solana-defcon
- Espressif USB Serial/JTAG hardware check: https://docs.espressif.com/projects/rust/no_std-training/02_1_hardware.html
- Public Solana badge repository: https://github.com/spacemandev-git/solana-defcon-badge-26

Important nuance: the connected device is compatible with the hardware profile of a Solana/ESP32-S3 badge, but the firmware actually running on it appears to be Hack the North-specific rather than the public Solana OS image.

## Bluetooth and Card Reader Observations

Bluetooth inventory did not produce useful connected-device details:

```text
SPBluetoothReporter getBluetoothControllerDict controllerInfo == nil
Bluetooth:
```

The built-in SD card reader was present but inactive:

```text
Built in SD Card Reader:
  Vendor ID: 0x17a0
  Device ID: 0x9755
  Link Width: Off
  Link Speed: Off
```

No separate SD/NFC/card-reader evidence related to the Solana badge was found.

## Serial Console Access

The device was available over:

```text
/dev/cu.usbmodem101
```

Baud rate used:

```text
115200
```

An initial passive listen at 115200 produced no spontaneous output. Sending the single character `r`, based on the public Solana badge factory-test docs, produced app lifecycle logs rather than a factory-test status report:

```text
I (163509) app_reg: heap after exit My Badge: free=65116 largest=51200
I (163739) app_reg: heap after enter Launcher: free=65116 largest=51200
I (163739) app_reg: launched Launcher
r
```

This suggested the badge was not running the factory test firmware. It was running an application launcher/OS environment.

I then tried Solana OS push-protocol-style probes:

```text
PING
INFO
```

The device responded:

```text
PING
Unrecognized command
badge>  INFO
Unrecognized command
badge>
```

That established that the device had an interactive console prompt named:

```text
badge>
```

## Console Help Output

Running `help` listed these available commands:

```text
help
press
shot
uitree
ls
cat
snapshot
xfer
radio
heap
chess
rm
mkdir
put
blindbox_response
apps
reload
card
config
ripple
sponsormap
reboot
logo
debug
appmode
badge_token
badge_profile
prov
seedall
factory_reset
```

Notable conclusions from these commands:

- The device has a UI framework capable of dumping widget trees (`uitree`) and screen captures (`shot`).
- It has a filesystem exposed through `ls`, `cat`, `mkdir`, `put`, and `rm`.
- It has app management through `apps` and `reload`.
- It has radio diagnostics through `radio`.
- It has provisioning/admin commands such as `prov`, `debug`, `appmode`, `seedall`, and `factory_reset`.
- It has Solana-specific or challenge-specific hooks such as `blindbox_response`, `ripple`, and `sponsormap`.
- It includes a `badge_token` command, which I did not run.

I avoided destructive or credential-revealing commands such as `rm`, `put`, `prov erase`, `factory_reset`, `debug`, `badge_token get`, or filesystem-wide `snapshot`.

## Installed Applications

The `apps` command returned the following installed apps:

```text
badgeid           ME   My Badge
connect           CON  Connect
scanner           SCN  Scanner
sync              SYN  Sync
share             SHR  Share
blindbox          SOL  Solana Scavenger Hunt
sponsors          SPN  Sponsors
workshops         HEX  Workshop Quest
mood              RGB  Mood LEDs
socials           WEB  Socials
schedule          SCH  Schedule
rules             i    Info & Rules
museum            MUS  Museum
progress          %    Progress
snake             SNK  Snake
dice              D20  Dice
67                67   67
level             LVL  Level
eightball         8    EightBall
conversation      TLK  Conversation Starters
lightshow         FX   LightShow
tetris            TTR  Tetris
2048              MRG  Merge Mode
breakout          BRK  Breakout
tictactoe         T3   TicTacToe
chess             CHS  Chess
maze              MZE  TiltMaze
flappy            FLP  Flappy
bubblewrap        POP  Bubble Wrap
ktnh              HNK  Keep Talking and Nobody Honks
hardwarephysics   PHY  Hardware Physics
teardown          HW   Teardown
flashlight        FL   Flashlight
blackjack         21   Blackjack
accel             ACC  Accel
test              TST  Test
swag              SHT  Swag
directory         DIR  Directory
files             [folder icon]  Files
diagnostics       DBG  Diag
meminfo           MEM  Memory
reaction          RCT  React
whack             WHK  Whack
pong              PNG  Pong
slots             7    Slots
hangman           HNG  Hangman
mines             MN9  Mines9
fortune           FRT  Fortune Cookie
etch              ETC  Etch
kaleido           KLD  Kaleido
lava              LVA  Lava
dance             DNC  Dance
sokoban           SB   Sokoban
hopper            HP   Hopper
jetpack           JP   Jetpack
invaders          IV   Invaders
blocks            3D   Blocks
outrun            Race  Outrun
```

The relevant app for your original request is:

```text
blindbox          SOL  Solana Scavenger Hunt
```

The app id `blindbox` plus the command `blindbox_response` suggest this is a challenge, loot-box, scavenger-hunt, or item-collection feature rather than a normal wallet app.

## Hack the North Branding

Running the `logo` command printed large ASCII art spelling:

```text
HACK THE NORTH
```

This is the strongest firmware-level indicator that the connected badge is a Hack the North badge or Hack the North-customized badge firmware.

This also explains why the firmware did not behave like the public Solana DEF CON badge firmware despite using Solana-related storage and ESP32-S3 badge-like hardware.

## Heap and Runtime State

The `heap` command returned:

```text
sys_free=64756
sys_largest=51200
sys_min=55788
lv_total=61952
lv_free=41836
lv_max_used=22948
lv_used_pct=33
lv_frag_pct=2
```

Running tasks:

```text
task=console_repl prio= 2 stack_free=2160
task=accel_cache  prio= 1 stack_free=2416
task=IDLE         prio= 0 stack_free=1248
task=taskLVGL     prio= 4 stack_free=12416
task=btn_poll     prio= 5 stack_free=3812
task=esp_timer    prio=22 stack_free=3812
task=Tmr Svc      prio= 1 stack_free=1784
```

Interpretation:

- The badge is live and running a multitask embedded firmware.
- It uses LVGL or an LVGL-like UI stack.
- It has a button polling task and an accelerometer cache task.
- Free heap is modest but normal for an ESP32-class badge.

## Radio Diagnostics

The `radio` command returned:

```text
last_error_stage=none
last_error_code=0 (0x0)
controller_state=0
free_heap=64740
largest_heap_block=51200
```

The `xfer` command returned:

```text
state=Idle
fail=None
server_status=0
progress=0%
frames=0
recv=0
repairs=0
sent=0
elapsed_ms=0
station=none heard recently
```

Interpretation:

- No radio error was reported at the time of inspection.
- No active sync/transfer was in progress.
- The badge had not heard a station recently, at least according to this transfer/radio subsystem.

## Filesystem Findings

The top-level filesystem listing showed:

```text
d        0  config
-      386  identity.json
-      227  solana.json
```

Listing `/littlefs` returned the same:

```text
d        0  config
-      386  identity.json
-      227  solana.json
```

Attempts to find an app directory at `/littlefs/apps/blindbox` failed:

```text
opendir /littlefs/apps/blindbox: No such file or directory
```

That means the app registry is likely compiled into firmware, stored somewhere else, or generated from a different application bundle format. This is different from the public Solana OS layout, where apps live under `/apps/<id>/` on LittleFS.

I did not read `identity.json`, because identity files can contain user/device secrets or tokens and were not necessary to answer the Solana question.

## Solana Data File

The file `/littlefs/solana.json` contained the following fields:

```json
{
  "public_key": "279sMthuRBJB39EKVqLZaRdtjMXePS3EtKjBrH3ucMnk",
  "secret_key": "[REDACTED]",
  "all_items": [],
  "item_count": 0,
  "equipped_pal": 0,
  "last_link": ""
}
```

The actual device file contains a plaintext `secret_key` value. I am intentionally not preserving it in this report.

Interpretation:

- The badge has a Solana keypair stored locally.
- The secret key is stored in plaintext in JSON.
- The app inventory is empty:
  - `all_items` is `[]`
  - `item_count` is `0`
- No item/pal appears to be equipped beyond the default value:
  - `equipped_pal` is `0`
- No last Solana/link interaction is recorded:
  - `last_link` is empty

Security note: because the secret key is stored in plaintext and readable over the badge console, this wallet should be considered compromised. Do not deposit funds or valuable NFTs/tokens into it. If it was ever used for anything important, move those assets to a fresh wallet whose private key was never stored on this device.

## Public Solana Account Checks

I queried public Solana RPC endpoints using only the public key:

```text
279sMthuRBJB39EKVqLZaRdtjMXePS3EtKjBrH3ucMnk
```

### Mainnet SOL Balance

RPC endpoint:

```text
https://api.mainnet-beta.solana.com
```

Result:

```json
{
  "value": 0
}
```

The account had `0` lamports on mainnet at the time checked.

### Devnet SOL Balance

RPC endpoint:

```text
https://api.devnet.solana.com
```

Result:

```json
{
  "value": 0
}
```

The account had `0` lamports on devnet at the time checked.

### Mainnet SPL Token Accounts

Token Program:

```text
TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
```

Result:

```json
{
  "value": []
}
```

No standard SPL token accounts were found on mainnet.

### Mainnet Token-2022 Accounts

Token-2022 Program:

```text
TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
```

Result:

```json
{
  "value": []
}
```

No Token-2022 accounts were found on mainnet.

## Public Solana Badge Reference Details

The public Solana Badge / DEF CON 34 material describes a badge with the following properties:

- ESP32-S3-WROOM-1-N16R8
- 16 MB flash
- 8 MB octal PSRAM
- 2.8 inch ILI9341 SPI LCD
- Six physical buttons over a TCA9534 I2C expander
- Stereo PDM microphones
- Two WS2812B-compatible RGB LEDs
- NXP SE050C2 secure element
- USB-C with a switchable data path
- Battery charging and power path
- Expansion header

The public docs also describe two firmware modes:

- Factory hardware test kit
- Solana OS, a Lua runtime and app launcher

The factory test firmware reportedly accepts single-key serial commands such as:

```text
h help
s I2C scan
r status report
i re-init I2C
t touch detail
T GT911 dump
c/C config dump
j USB HID self-test
b battery
m microphones
e SE050 re-test
g RGB self-test
p pin map
v verbose log
R reboot
```

The connected badge did not behave like this factory test firmware. Sending `r` did not print a hardware status report; it appeared to interact with the current app/launcher instead.

The public Solana OS documentation also describes:

- A launcher
- Settings screens
- Lua apps
- App push over Wi-Fi/BLE/serial
- Badge identity derived from an Ed25519 keypair
- A public key shown as a Solana-style base58 address

Some concepts overlap with this connected device, but the actual console commands and Hack the North logo show this firmware is customized or unrelated to the public Solana OS tree.

## Risk and Privacy Notes

### Plaintext Private Key

The most important security finding is that `/littlefs/solana.json` contains a plaintext Solana secret key.

Practical implications:

- Anyone with serial console access can read it.
- Anyone with a filesystem dump can read it.
- If the badge is lost, handed around, or connected to an untrusted machine, that key should be treated as exposed.
- The public account currently appears empty, which reduces immediate financial risk.

Recommended handling:

- Do not fund this wallet.
- Do not treat it as a secure wallet.
- If this key was imported anywhere else, retire it.
- If any assets are later deposited to the public key, move them to a fresh wallet immediately.

### Badge Admin Commands

The console exposes commands such as:

```text
debug
appmode
badge_token
prov
seedall
factory_reset
```

I did not run admin or destructive commands. These may reveal credentials, change device state, unlock hidden content, or erase data.

### Identity File

The file `/littlefs/identity.json` exists but was not read. Based on name and context, it may contain identity, profile, or token-like data. It should be treated as sensitive until inspected carefully.

## What I Did Not Do

I did not:

- Dump the full filesystem.
- Read `/littlefs/identity.json`.
- Read or print `badge_token`.
- Run `debug` with a password.
- Run provisioning erase or factory reset commands.
- Modify files on the badge.
- Delete or install applications.
- Use the Solana secret key for any transaction or signature.
- Query private services with badge credentials.

All Solana blockchain checks used only the public key.

## Likely Interpretation

The most likely story is:

1. The connected USB device is an ESP32-S3 badge exposing its native USB Serial/JTAG interface.
2. The hardware identity is compatible with the public Solana DEF CON badge class of devices, or at least with ESP32-S3 badge hardware.
3. The firmware currently installed is Hack the North-branded and has its own command console and application registry.
4. The firmware includes a Solana-themed application named `Solana Scavenger Hunt`.
5. That app stores its state in `/littlefs/solana.json`.
6. The app has generated or provisioned a Solana keypair, but the corresponding public account appears empty.
7. The app's item/inventory state is currently empty.

## Useful Commands If You Continue Manually

Read-only or low-risk commands:

```text
help
apps
heap
radio
xfer
logo
ls
ls /littlefs
cat /littlefs/solana.json
```

Use caution with:

```text
snapshot
cat /littlefs/identity.json
badge_token get
debug <password>
appmode
prov
seedall
factory_reset confirm
rm
put
config
```

Potentially useful next investigations:

- Inspect the badge UI for the `Solana Scavenger Hunt` app.
- Check whether scanning another badge or QR code populates `all_items`.
- Inspect `/littlefs/identity.json` only if you are comfortable handling potentially sensitive identity data.
- Use the public key on a Solana explorer to monitor whether it ever receives assets.
- Ask the event organizers or firmware source whether `blindbox` is intended to be on-chain or just uses Solana-style keys for local identity.

## References

- Solana Badge hardware walkthrough: https://solanadefcon.com/
- SkyRizz Solana DEF CON badge overview: https://skyrizz.com/solana-defcon
- Solana badge repository: https://github.com/spacemandev-git/solana-defcon-badge-26
- Espressif USB Serial/JTAG device documentation: https://docs.espressif.com/projects/rust/no_std-training/02_1_hardware.html
- Solana Token-2022 program documentation: https://www.solana-program.com/docs/token-2022
- Solana SPL Token CLI reference: https://solana.com/docs/references/spl-token-cli
