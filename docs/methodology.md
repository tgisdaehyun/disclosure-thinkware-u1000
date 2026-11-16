# Methodology, scope, and limitations

## Inputs and tools

The static target was an owner-supplied vendor firmware archive labeled U1000 1.00.18. Its identity, selected extracted component hashes, and address conventions are published in [artifact-manifest.json](../evidence/artifact-manifest.json). Firmware binaries and the extracted filesystem are not redistributed.

Analysis used **REA with Ghidra 12.1.4**, targeted ARM little-endian disassembly with Capstone, ordinary filesystem inspection, and hash verification. Decompiled output is a tool interpretation, not the vendor's original source. Analyst pseudocode in this repository is a compact reconstruction of the cited paths.

No new firmware was installed for the security comparison. Original archive and payload hashes were unchanged at the end of the static work.

## Address conventions

- The main raw RTOS payload was mapped at `0x00020000` using ARM little-endian interpretation.
- For the analyzed package, RTOS package offset = `VA - 0x20000 + 0x330`.
- Linux library addresses shown in the analysis use Ghidra's `0x00100000` import bias. Subtract `0x00100000` to obtain the corresponding ELF virtual address; this is not automatically a file offset or a runtime ASLR address.

These mappings apply only to the hashed components, not to arbitrary firmware versions.

## Extraction coverage

SquashFS extraction recovered 576 regular files. The unprivileged extractor could not create the `dev/console` and `dev/null` special device nodes and therefore returned exit status 2. This does not invalidate the regular configuration/scripts/libraries used for these findings, but it is not a complete reconstruction of all device nodes.

## Hardware tests

The owner-controlled camera was found through its hotspot's associated-client list and expected services. `APP_CONNECT` returned model `U1000`. Basic status queries and RTSP requests were attempted, followed by a Telnet login and limited read-only shell inspection.

The Telnet session submitted `root` and reached a UID-0 shell without a password prompt. Subsequent commands queried system information and selected RTOS memory. A selected 32-byte RTOS code prefix matched the supplied firmware. The installed firmware's complete version and full digest were not independently obtained.

The hardware observations apply to an owner-controlled research configuration. No newly installed service or firmware patch was used to enable Telnet, and a separate factory-reset production baseline was not tested.

Earlier RTSP tests temporarily changed management application mode and requested streaming, then returned to main mode and disconnected. The final system status response reported `recording` and RTSP `off`. A separate liveview status field reported `stopped`; the discrepancy was not resolved, and no recording-file playback was used to prove recording health. No usable RTSP video was obtained.

Sensor-query attempts found a zero-valued candidate buffer and an inactive-looking sampling state. Because freshness and cache coherency were not established, these are not presented as sensor measurements or a hardware failure. They are not a security finding.

## What was not tested

- Network `GET_FILE` access outside recording directories.
- Any network `PUT_FILE` transfer or filesystem modification through it.
- A live `wifi_info` credential response.
- Code execution or persistence through an uploaded file.
- Factory-reset production-baseline Telnet behavior.
- Working RTSP video interception.
- Single-session denial of service.
- A Thinkware Cloud APK or Cloud account attack.
- Every firmware revision, hardware variant, or current vendor release.

The absence of a reproduced result is preserved as an open item, rather than replaced with a result from a related model.

## Evidence handling

The published [static evidence index](../evidence/static-evidence.json) provides component/function identities, analyst observations, and retained REA evidence IDs. Those IDs identify local retained records; they are not public evidence endpoints. Full tool bundles are withheld because they include unnecessary firmware material and private analysis paths.

The public hardware record contains selected observations only. The Telnet excerpt marks omissions and does not invent intervening output. Device serials, UUIDs, MAC addresses, real IP addresses, GPS positions/speeds, network credentials, and recordings are excluded.

## Analysis exclusions

Two attempted receiver decompilations addressed code not recognized as a function. The tool fell back to a different source-file-offset mapping. Those outputs were rejected; the receiver logic was checked using exact-address disassembly instead. Ambiguous-symbol query errors and an unrelated offset-fallback result were also excluded. No finding is based on those mismatched functions.

Indirect calls, file-receive dispatch, state transitions, and shared-memory coherency remain partially unresolved. A tool export containing no metadata-level unknowns would not remove these substantive limitations.
