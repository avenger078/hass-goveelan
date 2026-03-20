# Govee LAN Control for Home Assistant

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg?style=for-the-badge)](https://github.com/hacs/integration)

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=avenger078&repository=hass-goveelan&category=integration)

## Fork Notice

This is a community-maintained fork of [wez/govee-lan-hass](https://github.com/wez/govee-lan-hass), originally created by [@wez](https://github.com/wez). Full credit for the original integration and the underlying [govee-led-wez](https://github.com/wez/govee-py) Python library goes to him.

The upstream repository appears to be unmaintained. This fork was created to restore compatibility with **Home Assistant 2026.3+**, which removed several deprecated light platform APIs that the original integration relied on.

### Changes from upstream

The following breaking changes introduced in HA 2026.3 were addressed:

- Removed deleted light constants from imports: `ATTR_COLOR_TEMP`, `SUPPORT_BRIGHTNESS`, `SUPPORT_COLOR`, `SUPPORT_COLOR_TEMP`
- Removed `ColorMode.BRIGHTNESS` from `supported_color_modes` — combining it with `COLOR_TEMP`/`RGB` is now a hard error
- Removed deleted `_attr_color_temp` (mireds) entity attribute assignments
- Removed the mired-based `ATTR_COLOR_TEMP` handler from `async_turn_on` (HA now only sends `ATTR_COLOR_TEMP_KELVIN`)
- Replaced `async_forward_entry_setup` (singular, removed) with `async_forward_entry_setups` (plural)
- Replaced deprecated `HANDLERS.register` decorator and `CONNECTION_CLASS` in config flow
- Fixed `OptionsFlow.__init__` conflict with HA's read-only `config_entry` base class property
- Replaced `hass.loop.create_task` with `hass.async_create_task`
- Replaced `controller.stop` with `controller.async_stop` for proper BLE cleanup on unload

## ⚠️ Consider using the [Govee2MQTT AddOn](https://github.com/wez/govee2mqtt) Instead ⚠️

> Before you get too invested in setting up `govee-lan-hass`, I wanted to note
that I've shifted my development focus to my [Govee2MQTT AddOn](https://github.com/wez/govee2mqtt).
That has support for new APIs and devices, and is more flexible than the `govee-lan-hass` integration.
> 
> Note that due to how Govee's LAN API works, you cannot run both the addon and the integration at the same time.

## Govee LAN Control for Home Assistant

This works in conjunction with my
[govee-led-wez](https://github.com/wez/govee-py) python library to provide
control over Govee-manufactured lights, preferentially using the LAN protocol
for local control.

## Installation

**Note: you need to [enable the LAN API for each individual device](#tips-on-enabling-the-lan-api)!**

Recommended first step: Obtain an HTTP API key from the Govee API:
* Open the Account Page of the Govee mobile app (the person icon in the bottom right)
* Click the settings "cog" icon in the top right
* Click Apply for API Key and fill out the form
* Your key will be emailed to you.

It is recommended to wait until you have the key before configuring the
integration, as the HTTP API is used to retrieve the names of the devices from
your account, and those names influence the entity ids that are set up for the
devices.

You don't require an HTTP API key to use this integration if all of the devices
that you want to control are supported by the LAN API, but having your names
set up from the app is nice, so I recommend getting that set up anyway.

* Install [HACS - the Home Assistant Community Store](https://hacs.xyz/docs/setup/download/)
* Add this fork as a custom repository in HACS:
  1. In Home Assistant, go to **HACS**
  2. Click the three-dot menu in the top right and select **Custom repositories**
  3. Enter the repository URL: `https://github.com/avenger078/hass-goveelan`
  4. Set the category to **Integration** and click **Add**
  5. Search for "Govee LAN Control" in HACS and click **Download**

* Once installed, restart Home Assistant
* Then go to **Settings → Devices & Services** and click **Add Integration**
* Search for "Govee LAN Control" and add the integration
* Enter your HTTP API key where prompted

## Notes

* The `govee-led-wez` library doesn't perform immediate *read-after-write* of
  the device state after controlling a device. When using the HTTP API, doing
  so would double the number of calls made to the web service and increase the
  chances of hitting a rate limit. For the LAN API, while the devices generally
  respond immediate to a control request, they don't reliably return the
  updated device state for several seconds.  As such, this integration
  assumes that successful control requests result in the state reflecting
  the request.  If you are using other software to also control the lights,
  then you may experience incorrect information being reported in Home Assistant
  until the devices are polled.
* LAN devices have their individual state polled once per minute
* HTTP devices have their individual state polled once every 10 minutes
* New LAN devices are discovered every 10 seconds
* New HTTP devices are discovered every 10 minutes
* You can force re-discovery/updating of HTTP device and their names by
  reloading the integration

## Tips on Enabling the LAN API

**Note: you need to enable the LAN API for each individual device!
Repeat these steps for each of your devices!**

The [LAN API](https://app-h5.govee.com/user-manual/wlan-guide) docs have a list
of supported models.  The Govee app sometimes needs coaxing to show the LAN
Control option for supported devices.  Here's what works for me:

* Open the app and ensure that the device(s) are fully up to date and have WiFi configured
* Close/kill the Govee app
* Turn off wifi on your mobile device; this seems to help encourage the app to show the LAN Control option.
* Open the app and go to the settings for the device
* The LAN Control option should appear for supported devices
* Turn it on
* Once done enabling LAN Control for your Govee devices, re-enable wifi on your mobile device

## Requirements of the LAN API

* Home Assistant must be running on the same network as your Govee devices.
  If you are running it in docker, you will need to use `network_mode: host`
  or use a macvlan network.
* UDP port 4001 much be reachable from the integration. The LAN discovery
  protocol sends a multicast packet to 239.255.255.250 port 4001.
* UDP port 4002 must be available for the integration to receive UDP packets
  from the discovery protocol ping. 
* UDP port 4003 must be reachable from the integration. Govee devices will
  listen for commands on this port.
* These fix port requirements are unfortunately part of the LAN API protocol.
  That means that you cannot run two different implementations of the
  Govee LAN API from the same IP address (eg: homebridge's govee plugin cannot
  run on the same IP as this HASS integration).  If you need to do that for
  some reason, you will need to configure each of them to run on separate IP
  addresses.
* Your network needs to support *multicast UDP* over wifi. Your wifi router may
  require some specific configuration to allow this to work reliably. Note that
  this is NOT the same thing as *multicast DNS*, although there is some relation
  between them.

## Troubleshooting

If you add this to your `configuration.yaml` and restart home assistant, you'll get verbose logging that might reveal more about what's happening:

```yaml
logger:
  logs:
    custom_components.govee_lan: debug
    govee_led_wez: debug
```

In addition, some diagnostics are recorded as extended attribute data associated
with each entity. In HASS, go to "Developer Tools" -> "State", then type in the name
of the light you were trying to control; it should show something like this screenshot:

![image](https://user-images.githubusercontent.com/117777/212545829-e0d2dc54-20f3-44bf-ac25-6bc679c76583.png)

