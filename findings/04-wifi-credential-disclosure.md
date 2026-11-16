# U1000-04: Plaintext Wi-Fi credentials included in status responses

- **Evidence level:** static response-construction confirmation; no live credential query.
- **Component:** RTOS `GET_STATUS` / `wifi_info` handler and Wi-Fi helper scripts.
- **Interface:** command TCP 7878; `GET_STATUS` ID 2050.
- **Candidate weakness:** CWE-200, Exposure of Sensitive Information to an Unauthorized Actor.
- **CVE:** none assigned in this report.

## Description

In the supplied U1000 1.00.18 firmware, the `wifi_info` status handler reads Wi-Fi configuration, constructs a JSON response containing an `ssid` and a plaintext `password`, and passes it to the normal response sender. The inspected handler does not apply a separate credential-disclosure permission check.

This is a status-response path, distinct from reading a configuration file through `GET_FILE` or Telnet.

## Static evidence

The `GET_STATUS` dispatcher at `0x00552F40` uses a table of 24-byte entries. The `wifi_info` entry selects handler `0x0054E708`.

That handler:

1. Reads RTOS paths `A:\pref\wifi_ssid` and `A:\pref\wifi_passwd_new`.
2. Depending on mode and helper output, may replace those values with current Wi-Fi status information.
3. Emits a diagnostic log containing the SSID and password values.
4. Creates a successful response with `msg_id: 2050`, `type: "wifi_info"`, and a parameter object holding the plaintext values.

The structural example below is an **analyst reconstruction with placeholders, not a captured network response**:

```json
{
  "rval": 0,
  "msg_id": 2050,
  "type": "wifi_info",
  "param": {
    "ssid": "<configured SSID>",
    "password": "<configured plaintext password>"
  }
}
```

This object structure is inferred from the U1000 handler. It is not copied from a U3000 runtime capture, and should not be assumed interchangeable with every U3000 client's expected response shape.

## Related storage and logging

The extracted AP startup script writes a passphrase to `/tmp/hostapd.conf`. The station startup script writes a PSK to `/tmp/wpa_supplicant.conf` and has a conditional SD configuration-cache path. `wifi_status.sh` reads Wi-Fi password material and includes it in its output.

Such local configuration files can be required by Wi-Fi software. **Plaintext storage alone is not proof of unauthenticated network disclosure.** The concern here is the combination of a password-bearing response, its admission path, diagnostic logging, and the independently observed root management exposure.

## Preconditions and limits

A local management session and basic status queries were observed without a separate owner password. The static session/admission analysis is described in [architecture](../docs/architecture.md). **The live camera was not sent a `wifi_info` request, and no live network password was collected for this report.**

Which configured credential is returned depends on operating mode and helper results. This report does not establish that every stored hotspot password is disclosed or that any Cloud account password is involved.

## Potential impact and correction

If the status path is reachable under the observed session model, configured Wi-Fi credentials could be exposed without filesystem access. Reused credentials could increase the impact beyond the camera, but no such reuse or secondary access was tested.

Remove passwords from status responses and diagnostic logs. Return non-sensitive connection metadata only, enforce owner authorization, restrict diagnostic channels, and audit any configuration cache for appropriate access and lifecycle handling.

[Back to overview](../README.md)
