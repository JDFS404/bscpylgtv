## Additional manifests
List of additional manifest files that can be used with `-m` switch:
- `manifest-comp.json` : compatibility manifest for webOS26 and later versions that provides limited functionality ([more info](https://github.com/home-assistant/core/issues/172703))

### Note on luna setters and `manifest-comp.json`

`manifest-comp.json` does not request `WRITE_NOTIFICATION_ALERT`. Every luna-based setter in this
library (`set_settings`, `set_picture_settings`, `set_device_info_luna`, `enable_tpc_or_gsr`, …) goes
through `luna_request()`, which calls `system.notifications/createAlert`. Without that permission the
call fails with `401 insufficient permissions`, so with this manifest no luna setter works at all —
including with the webOS 26 fallback in `luna_request()`.

Adding `WRITE_SETTINGS` does not help on its own. On webOS 26 the ssap endpoint
`settings/setSystemSettings` still returns `401 insufficient permissions` even when that permission is
present in the unsigned manifest and the on-screen prompt was accepted. That tier appears to still
require a valid signature, which is exactly why the `createAlert` route remains necessary.

Measured on an LG OLED65G36LA, firmware 43.21.71 (webOS 26), using an unsigned 37-permission manifest
(no `signed` block, all permissions in the outer array):

| call | result |
| --- | --- |
| `system.notifications/createAlert` | ok |
| `system.notifications/closeAlert` | no reply — see `luna_request()` |
| `audio/setVolume` | ok |
| `com.webos.service.networkinput/getPointerInputSocket` | ok |
| `settings/setSystemSettings` | `401 insufficient permissions` |
| `externalpq/setExternalPqData` | `401 insufficient permissions` |
