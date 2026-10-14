# Configuration

The current configuration flow is primarily web-based.

## Normal startup flow

On boot, the Pico tries to connect to the saved Wi-Fi network using the stored SSID and password.

If the Wi-Fi connection succeeds, the adapter will continue normally and the configuration page is available only while the device is still in setup mode.

If the Wi-Fi connection is not available or the saved network is not configured yet, the Pico starts its own access point:

- SSID: `PicoAdapterGB`
- password: `magb!123`

Then the setup page is available at:

```text
http://192.168.4.1/
```

Important: the fallback hotspot does not currently provide DHCP service. When connecting to it, the client must be configured manually with a compatible IP in the same subnet, typically something like `192.168.4.2`, and then the browser should open `http://192.168.4.1/`.

## Web setup page

The built-in web interface lets you configure:

- Wi-Fi SSID
- Wi-Fi password
- primary and secondary DNS
- DNS port
- relay server
- relay token
- P2P port
- adapter device type
- unmetered mode
- redirect mail behavior

It also shows, read-only, the relay phone number the adapter was assigned by the relay server. The value is read once when the page loads:

- if no relay server is configured, it shows "No relay server configured"
- if the number has already been obtained, it shows the assigned number
- if the relay server is offline or the number has not been obtained yet, it shows "Not available"

The page is designed to be used from a laptop, phone, or tablet connected to the adapter's Wi-Fi network.

It also shows the board's pairing code, read-only. This identifies the board
in your account's device list; copying a configuration file to another board
does not copy its identity.

If the board has a pairing code but no mail authentication key, the page
shows a warning. Download a new `mobile_config.bin` from your account, upload
it with "Overwrite custom server config too? (like DNS and Relay Server)"
checked, then press `Save & Reboot` to provision the key.

## Web shutdown behavior

The configuration server is intentionally stopped when the Game Boy starts
a session. This prevents the setup page from staying active during normal
adapter operation.

The main firmware loop stops the web server about one second after libmobile
has processed the valid Start Session command. This lets the initial protocol
exchange finish before closing the HTTP connections. The web server stays
off until reboot.

## Save and reboot

The configuration form includes a single action called `Save & Reboot`.

When you save the configuration, the firmware persists the settings and reboots to apply them.

The form and file uploads edit a configuration snapshot, separate from the
running adapter. `Save & Reboot` queues persistence for the firmware loop;
the adapter loads the new settings after reboot.

## Format EEPROM

The `Format EEPROM` button asks for confirmation and resets the configuration
snapshot, including Wi-Fi and custom server settings. This explicit action
writes the reset snapshot to flash immediately in both implementations. It
is the authorized exception to deferred web configuration writes. The
running adapter loads the reset configuration on its next boot; formatting
does not reboot it automatically.

## EEPROM backup and restore

The web page also lets you download and upload the raw 512-byte Mobile Adapter GB EEPROM image (`mobile_config.bin`):

- `Download mobile_config.bin` saves the full 512 bytes from the configuration snapshot.
- `Upload mobile_config.bin` only accepts a file that is exactly 512 bytes and starts with the `MA` signature (the original adapter's config block); otherwise it's rejected.
- By default, uploading only restores that original adapter config block (offsets `0x00`-`0xBF`), leaving the current Wi-Fi/DNS/relay settings untouched.
- Checking "Overwrite custom server config too? (like DNS and Relay Server)" restores the full image and applies the DNS/relay/device/etc. block (offsets `0x100`-`0x15F`), which must start with the `LM` signature. It also applies the mail authentication key when the device-auth block has a valid `DA` header at offset `0x160`. The page updates the server settings immediately, decoded from the uploaded file in the browser.
- An upload only changes what's held in memory. Nothing is written to flash until you press `Save & Reboot`.

## LED status

The adapter has a single LED, used only as a simple boot/error indicator (not a general-purpose status light). On `picow` it's wired to the CYW43 Wi-Fi chip rather than the RP2040/RP2350 itself; on `esp` it's a plain RP2040/RP2350 GPIO (see each implementation's `led_hal_*` backend).

- **Solid on** from the moment the device powers on, for the whole boot sequence.
- **Solid off** once boot has fully finished: either the adapter reached normal operation, or the fallback setup hotspot is up and serving its page.
- **Blink code, then solid on again**: signals a real error partway through boot, before boot continues. Count the blinks (repeated 3 times, with a pause between each repetition):
  - **1 blink** — couldn't join the saved Wi-Fi network (generic failure: out of range, router unreachable, etc.); falling back to the setup hotspot.
  - **2 blinks** — the saved Wi-Fi network rejected the password; falling back to the setup hotspot.
  - **3 blinks** — the configuration failed to save to flash.
- **Blink code, repeating forever (boot halted)**:
  - **4 blinks** — the network hardware/module itself failed to initialize (e.g. on `esp`, the ESP-AT module never responded on UART - check wiring, power, and that it's running ESP-AT firmware at the expected baud rate; see [src/implementations/esp/README.md](../src/implementations/esp/README.md)). Unlike the other codes, boot does not continue: there's no working network hardware to fall back to a setup hotspot with, so the device halts here until power-cycled after the problem is fixed. On `picow`, if this is caused by the CYW43 chip itself failing to initialize, the LED (which is wired through that same chip) may not be able to blink this code - if boot seems stuck with no LED activity at all and the serial log shows "Network hardware failed to initialize", that's this same condition.

Blink codes only ever represent errors. Normal/expected states (e.g. no Wi-Fi configured yet on first boot, or waiting for the Game Boy) are not signaled through the LED.

Separately, while the device is running normally, the LED turning on again means there's a configuration change pending to be written to flash (see [doc/wiki/7-IMPORTANT NOTES.md](wiki/7-IMPORTANT%20NOTES.md)); wait for it to turn off before doing anything else on the Game Boy.

## Serial fallback

The serial console is no longer the normal configuration path for the device.

It is kept only as a troubleshooting and debugging fallback for cases such as:

- startup issues
- Wi-Fi recovery
- checking boot logs
- verifying that the adapter is initializing correctly

This is not the recommended way to configure the device in normal operation.

## Troubleshooting

### Web page does not appear

- Confirm the Pico is in access-point mode after boot.
- Check the Wi-Fi connection status on the serial console.
- Connect to `PicoAdapterGB` using the default password `magb!123`.
- Open the page at `http://192.168.4.1/`.

### Game Boy communication stops the web server too late

- Ensure the Game Boy link cable is wired correctly.
- Verify the level shifter and the serial pin mapping.
- Check the serial console for connection/log messages.

### Wi-Fi is not connecting

- Verify the SSID and password were saved correctly.
- Confirm the network is reachable and not blocked by AP isolation or captive portal rules.
- Reset the adapter and open the setup page again.

## Notes

This project is a compatibility-focused firmware. Small hardware and timing details matter, especially on the Game Boy link-cable side. If setup is failing, the first place to check is the physical wiring and the timing-sensitive Game Boy connection.
