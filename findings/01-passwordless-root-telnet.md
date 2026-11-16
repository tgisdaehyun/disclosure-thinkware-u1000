# U1000-01: Root Telnet access without a password

- **Evidence level:** hardware confirmed in the documented configuration; firmware-supported.
- **Component:** Linux Telnet service, TCP 23, and root account configuration.
- **Prerequisite:** network reachability to the camera.
- **Candidate weakness:** CWE-306, Missing Authentication for Critical Function.
- **CVE:** none assigned in this report.

## Observation

An owner-authorized Telnet connection to the tested U1000 displayed an `AmbaLink login:` prompt. Submitting the username `root` produced a shell without a password prompt. The `id` output identified UID 0. The session also returned kernel/system information and permitted selected reads of RTOS physical memory through `/dev/mem`.

The [sanitized transcript excerpt](../evidence/telnet-root-excerpt.txt) retains the login and UID evidence. Private addresses, device identifiers, unrelated command output, and recordings are excluded.

## Firmware explanation

In the supplied 1.00.18 image, the startup chain is:

```text
/etc/inittab
  -> /etc/init.d/rcS
     -> /etc/init.d/S50telnet start
        -> /usr/sbin/telnetd -F
```

`rcS` runs the `S??*` startup entries. `S50telnet` starts the daemon without a conditional activation check in that script. The root account has UID 0 and `/bin/sh`; its shadow password field is empty. An `/etc/securetty` file was not present in the extracted filesystem.

The empty field alone would not prove network login behavior, because login implementations can reject empty passwords. The observed root shell supplies that missing hardware evidence for this test configuration.

## Scope of the hardware result

The hardware result applies to the tested research configuration. No firmware patch or newly installed service was used to enable Telnet; factory-reset behavior was not separately verified.

The live unit identified itself as U1000. Its entire installed firmware was not hashed or independently verified as the exact supplied archive. The result must not be presented as a test of every unmodified or factory-reset U1000.

## Minimal authorized verification

On a camera and local network you control, observe whether TCP 23 is reachable. If a login prompt appears, submit the root username once and record whether a password is requested. If a shell is unexpectedly granted, use `id` to determine its privilege level, then exit. A rejected login or password challenge is a different result; do not substitute password guessing for this test.

The investigation did not use firmware replacement, password guessing, a memory-corruption exploit, or an installed persistence mechanism to obtain the shell.

## Impact

A reachable unauthenticated root shell grants much more authority than normal dashcam viewing. The observed session could inspect system state and selected RTOS memory. Potential access to recordings, configuration, and writable files follows from root authority, but this report does not claim that every protected partition is writable or that every possible privileged operation was executed.

## Recommended correction

Remove Telnet from production startup, lock passwordless root access, and separate diagnostics from the normal recording/network path. If servicing requires remote management, use an authenticated, explicitly enabled channel and verify its behavior on a clean production boot. Changing the Wi-Fi password reduces exposure but does not replace service-level authentication.

[Back to overview](../README.md)
