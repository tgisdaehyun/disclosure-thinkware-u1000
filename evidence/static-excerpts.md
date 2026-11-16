# Compact analytical reconstructions

These examples are **analyst pseudocode**, not vendor source and not verbatim decompiler output. They intentionally omit unrelated error handling and implementation details. Exact component hashes, addresses, and evidence IDs are in the adjacent JSON manifests.

## Absolute-path conversion

RTOS `0x000A41D0`, as called by `GET_FILE` and `PUT_FILE`:

```text
convert_filename(input, output, capacity):
    reject invalid arguments
    if input begins with '/':
        copy input into output, bounded by capacity
        return success
    obtain current working directory
    combine current directory and input
```

The absolute-path branch does not enforce a recording-directory root.

## Read-open operation

Linux data library `0x00104D74`:

```text
file = fopen64(job.filename, "rb")
if file opens:
    seek to requested offset
    read requested file bytes
    send bytes through the associated data connection
```

Transfer-state checks exist elsewhere. No per-client path boundary was found in the inspected filename-to-open path. This is static evidence, not a captured file download.

## Write-open wrapper

Linux data library `0x00103EF8`, reached from a type-5 data job:

```text
if offset == 0:
    fd = open64(requested_path, 0x101241, 0777)
else:
    fd = open64(requested_path, 0x101002)
```

The zero-offset flags include write/create/truncate behavior. The supplied path is not confined in this wrapper. This does not prove that read-only mounts are writable or that a network upload completed.

## Credential-bearing response

RTOS `0x0054E708`:

```text
ssid = read configured SSID
password = read configured Wi-Fi password
optionally update values from current Wi-Fi status helper
log SSID and password
build GET_STATUS response:
    type = "wifi_info"
    param.ssid = ssid
    param.password = password
send response
```

No live credential response is included or claimed.

## Session token and admission callback

RTOS allocator `0x000A47C0`:

```text
return current positive counter as the session token
increment counter, maintaining a positive value
```

The receiver checks the current token on ordinary commands. The separately installed callback at `0x00049EC8` checks an internal application-state value. Neither observation establishes an owner secret. Exact receiver code was checked with disassembly because two attempted decompilations resolved to mismatched functions and were excluded.

## Telnet startup and account state

The image's init configuration runs `rcS`; `rcS` executes startup entries including `S50telnet`; that entry starts `telnetd -F`. The extracted root account has UID 0, shell `/bin/sh`, and an empty shadow password field. Those static observations support, but do not replace, the separately recorded hardware login result and its configuration caveat.
