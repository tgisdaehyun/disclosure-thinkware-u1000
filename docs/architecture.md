# Management architecture and trust boundaries

The supplied U1000 image contains an ARM RTOS payload and an AArch64 Linux environment. The management/file-transfer behavior crosses both sides.

```text
Local network peer
    |
    +-- TCP 23 ------------------------------> Linux Telnet / root shell
    |
    +-- TCP 7878 --> command daemon --> RPC --> RTOS JSON/session dispatcher
    |                                             |
    |                                             +-- status handler --> reply
    |                                             |
    |                                             +-- GET_FILE / PUT_FILE
    |                                                      |
    +-- TCP 8787 <--> Linux data daemon <-------- RPC data job
                           |
                           +-- file open/read/write
```

This is a functional summary, not a complete process or RPC ABI reconstruction. The live process listing included the command/data/system-service daemons and the RTSP server, running as root on the tested device.

## Session handling

The RTOS receiver at `0x000A6A40` parses the request's token and message ID. A new `START_SESSION` request is handled separately from ordinary token matching. Token allocator `0x000A47C0` returns and increments a positive counter. Normal commands require a matching current session token.

An optional admission callback is installed into `0x03810B9C` by initializer `0x000A75EC`. The application initializer at `0x0004D674` supplies `0x00049EC8`. That function queries internal state through `0x00048734` and compares the result with registered internal values. The inspected callback does not parse a request credential, signature, or pairing proof. The concrete names of all permitted states were not established.

These checks regulate session and application state. The traced path does not establish an owner-authentication boundary. During the hardware tests, a new local session, model response, and basic status responses were accepted without a separate owner password. This does not prove every command is available in every state.

## File transfer boundary

`GET_FILE` and `PUT_FILE` accept a filename through the RTOS request path. The shared converter preserves a leading-slash absolute Linux path. The Linux data job then opens that path directly. An operating-system access check can still deny a file, but that is different from authorizing a particular remote user to access a recording directory.

A session token, an MD5 checksum, and the fact that the official app normally requests legitimate filenames are not substitutes for filesystem confinement and client authorization.

## Service startup caveat

The Linux `S60postservice` file has commented-out command/data/RTSP startup examples. This does **not** prove those services are disabled. RTOS function `0x00401AD4` contains a launch path for the remote API command, data, and system-service executables. Their running processes were subsequently observed on the test device.

## Network boundary

The camera was reachable as a station on an owner-controlled hotspot. Obtaining local reachability still matters: this work did not bypass WPA association, a firewall, or remote carrier NAT. The relay used during testing does not establish that the camera is publicly exposed.

Root Telnet, protocol file access, status responses, RTSP streaming, and Cloud application behavior are separate mechanisms. A successful test of one is not a reproduction of the others.
