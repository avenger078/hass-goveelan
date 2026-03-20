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

## Installation

Recommended first step: Obtain an HTTP API key from the Govee app:
* Open the Account Page of the Govee mobile app (the person icon in the bottom right)
* Click the settings "cog" icon in the top right
* Click Apply for API Key and fill out the form
* Your key will be emailed to you

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
