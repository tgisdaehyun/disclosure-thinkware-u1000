# U1000-03: Client-controlled absolute paths reach file-write operations

- **Evidence level:** static data-flow confirmation; no network write reproduction.
- **Component:** RTOS `PUT_FILE` processing and the Linux remote API data service.
- **Interfaces:** TCP 7878 command / TCP 8787 data; `PUT_FILE` ID 1286.
- **Candidate weakness:** CWE-73, External Control of File Name or Path.
- **CVE:** none assigned in this report.

## Description

The supplied U1000 1.00.18 firmware passes a client-supplied absolute filename from `PUT_FILE` into a Linux file-open operation with write/create/truncate behavior. The inspected stages do not impose an intended upload-directory boundary.

The write path shares a filename converter with [U1000-02](02-unrestricted-file-read.md), but reaches a different file operation and has a distinct integrity impact.

## Traced path

1. RTOS handler `0x000A7C88` reads the request's filename, offset, size, and MD5 field. It checks connection/transfer flags and requires a nonempty MD5 string.
2. Converter `0x000A41D0` copies a path beginning with `/` unchanged.
3. The handler sends transfer type **5** and the filename through the Linux RPC path.
4. Linux adapter `0x00105CAC` and dispatcher `0x001033A4` move the requested filename into a queued data job.
5. Worker `0x00103760` dispatches the type-5 job through `0x00103EF8`.
6. At offset zero, the open wrapper calls `open64(path, 0x101241, 0777)`. At nonzero offsets, it calls `open64(path, 0x101002)`. The zero-offset flags include write access, creation, and truncation. The requested mode remains subject to the process umask and filesystem behavior.

The inspected wrapper contains no filename allowlist or confinement to a recording/update staging directory. See [the analytical excerpt](../evidence/static-excerpts.md#write-open-wrapper).

## Checks that do not establish authorization

An MD5 field can check transfer integrity; it does not prove that the client owns the camera or may modify the requested path. Likewise, a session token issued to a new local connection is not itself an owner credential. Details and remaining runtime constraints are covered in [architecture](../docs/architecture.md).

## Limits

**No file was created, truncated, overwritten, or uploaded over this protocol during the investigation.** The receive worker contains indirect dispatch that was not completely reconstructed. The published evidence establishes the request-to-write-open path, not a completed network transfer.

The firmware includes a read-only SquashFS root filesystem. Therefore, unrestricted path selection does not mean that every selected system path can be modified. Process credentials, writable mounts, offsets, transfer state, and runtime mode constrain the actual effect. No execution of an uploaded file, reboot persistence, or code-execution chain was tested.

## Potential impact

If reachable in an allowed state and directed at a writable target, the operation could modify files outside the intended upload area, compromising recordings or configuration integrity. The report does not claim a universal writable-filesystem or remote-code-execution primitive.

## Verification and correction

Independent static verification can follow the addresses and component hashes in this repository. Any hardware confirmation should use a unique disposable file in an owner-controlled writable directory, with known contents and cleanup; it must not target boot files or real credentials. That confirmation is not claimed here.

Require owner authorization, restrict uploads to a dedicated staging directory, and resolve paths against that boundary securely. Keep update verification separate from general file transfer and use authenticated integrity checks where the operation changes trusted software.

[Back to overview](../README.md)
