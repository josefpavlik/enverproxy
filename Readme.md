# enverproxy

This is heavily based on the work of [@MEitelwein](https://github.com/MEitelwein) and forked from [Enverbridge-Proxy](https://gitlab.eitelwein.net/MEitelwein/Enverbridge-Proxy) (thanks for that).

Using this python script, you can decode the traffic between your envertech EVB202 and EVB300 and envertecportal.com. All data will be send to an MQTT broker.

## How to use

### Receiving side

Clone this repo on a linux machine at `/opt/enverproxy` and configure it as a [systemd unit](enverproxy.service). Copy and update the config at [/etc/enverproxy.conf](enverproxy.conf) to your needs.
Requires `paho-mqtt`. You can install it via `pip3 install paho-mqtt`

### EVB202

Restart your EVB202 and immediately press `OK` to enter the boot menu. Under `Set DHCP` set `USE DHCP` to `NO`. Under `Set Client IP` set the IP of your EVB202 to a static IP in your subnet. Under `Set Server IP` configure the IP address of your linux machine running `enverproxy.py`. Under `Set Server Mode` select `Local` as Server - yes the manual say's it's not supported for EVB202, but it works.

### EVB300

Configure your EVB300 to use WiFi in STA mode or AP+STA mode under `WiFi Setting`. STA Mode WiFi needs to be able to connect your enverproxy host.
Under `Trans Setting` select `SocketA Connect Setting` to `TCP-Client`. Enter the host IP or DNS name of your host running enverproxy. Port needs to match with your enverprox settings - port 10013 is successfully tested.
Do not connect an ethernet cable, that will stop the device from sending out data as configured in `Trans Settings`!

### Total power calculation

If activated in the configuration, enverproxy can calculate the total power and the number of microinverters online.
The format follows the format used in [https://github.com/mr-manuel/venus-os_dbus-mqtt-grid](venus-os_dbus-mqtt-grid), so you can use the data from enverproxy to emulate a power meter in your Venus OS device, e.g. Victron Cerbo GX.
If you specify the `total_phase_map` in the configuration, the returned data will also contain the total power per phase for a 3-phase environment.
The topic published for total power calculation is `enverbridge/total`.

## Nasty details

The EVB202 will connect to the server every second - even if there is no data to transmit. This will blow up your log file if the log level is set to 3 or higher. Every 20 seconds there is a transmission of some unknown data. If the microinverters are online there will be data approximately once every minute.

If the EVB202 is configured to Server Mode `Local`, it will connect to the configured Server IP via TCP on port 1898. If Server Mode is set to `Net` (the default value) it will try to connect to www.envertecportal.com via DNS and fallback to 47.91.242.120 (which is an outdated but hardcoded IP of envertecportal.com) via TCP on port 10013.

Instead of changing the Server Mode, you may also intercept the DNS query (and reply with a local IP) or redirect the TCP connection to your local machine.

There is a proxy mode, but I'm not using it, so this is totally untested.

## I've found a bug

Just open an [issue](https://github.com/zivillian/enverproxy/issues/new) with as many details as possible.

## Helpful links

- [initial FHEM discussion 🇩🇪](https://forum.fhem.de/index.php?topic=61867.0)
- [alternative to SetID 🇬🇧](https://sven.stormbind.net/blog/posts/iot_envertech_enverbridge_evb202/)
- [lengthy discussion about EVB202 🇩🇪](https://www.photovoltaikforum.com/thread/125652-envertech-bridge-evb202-oder-evb201/)
