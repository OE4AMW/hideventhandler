# hideventhandler
React on HID events from USB-Devices

- claims configured USB-devices and listens for HID event reports
- for configured HID events, reactions can be configured:
    - respond USB-device immediately
    - run a binary/shellscript
        - with respons depending on returnvalue

The tool does not evaluate the Report-Descriptor - it matches complete USB-messages against the configured once (and ignores all others).

## Configuration

Find Vendor- and Product-ID (e.g. using lsusb) of your HID-Device

Create a file config.json, either in the program's working directory, or the home-directory of the running user.

For each device add a new object in the `devices`-array.

The object contains following  properties:
- `deviceDescription` with following properties:
    - `vendor`: vendor-ID as Hex-String
    - `product`: product-ID as Hex-String
    -  `messageLength`: Length of messages to be evaluated (as integer). Shorter HID-Reports are cascadeded before evaluation, longer HID-Reports are split.
- `commands`: an array of objects (one object per HID-Report to be evaluated) with following properties:
    - `command`: HID-Report as HEX-String
    - `exec`: Name of binary to be executed on receipt of `command`
    - `immediateResponse`: optional HEX-String to be responded before `exec`is called
    - `postExecPositiveResponse`: optional HEX-String to be responded after `exec`finished with a return-value of `0`
    - `postExecNegativeResponse`: optional HEX-String to be responded after `exec`finished with a return-value != `0`

the repo contains an example with two devices:
- a Logitech-Mouse (simple example)
    - Button-Clicks are just logged
    - movements are not handled
- a jabra-headset ()
    - pressing the "Phone"-Button:
        - turns the green LED on the headset on
        - runs a "voicekeyer.sh - script
        - turns the green LED on the headset off after the script finished
    - other buttons are not handled

## Run as non-root user
The USB-device needs to be detached from the HID driver and claimed by the tool. This requires full access to the device and is usually forbidden for non-root users.

1. add yourself to the "plugdev"-group:

Run

    sudo usermod -a -G plugdev $USER
(and re-login)

2. Create/edit a udev-rule

Run

    sudo nano /etc/udev/rules.d/99-myhid.rules
and add a line such as

    #example for an old logitech-mouse
    ATTRS{idProduct}=="c018", ATTRS{idVendor}=="046d", MODE="0660", GROUP="plugdev"
with the correct Vendor- and Product-ID

3. Reload udev-Rules:
 
Run

    sudo  udevadm control --reload-rules && sudo udevadm trigger

