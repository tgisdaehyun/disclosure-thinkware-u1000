# U1000-02: Client-controlled absolute paths reach file-read operations

- **Evidence level:** static data-flow confirmation; no network file-read reproduction.
- **Component:** RTOS `GET_FILE` processing and the Linux remote API data service.
- **Interfaces:** TCP 7878 command / TCP 8787 data; `GET_FILE` ID 1285.
- **Candidate weakness:** CWE-73, External Control of File Name or Path.
- **CVE:** none assigned in this report.

## Description

The supplied U1000 1.00.18 firmware accepts a request-supplied filename, preserves absolute Linux paths, and ultimately opens that filename in the Linux data service. The inspected path does not confine reads to the camera's recording directories.

This is not a claim that a traversal string is required: an absolute path beginning with `/` is accepted directly by the common converter.

## Traced path

| Stage | Analyzed address | Observed behavior |
|---|---|---|
| RTOS command manager | `0x000A8254` | Dispatches `GET_FILE` |
| RTOS read handler | `0x000A51AC` | Extracts `param`, offset, and requested size; checks connection/transfer flags |
| Common path conversion | `0x000A41D0` | Copies an absolute path unchanged and reports success |
| RTOS/Linux request | `0x00402A94` call site | Sends a type-1 data request containing the converted filename |
| Linux request adapter | `0x00105CAC` | Copies the filename into the data job |
| Linux request dispatcher | `0x001033A4` | Queues the requested transfer |
| Linux file-read worker | `0x00104D74` | Calls `fopen64(job.filename, "rb")`, then reads and sends file bytes |

Addresses use the mappings documented in [methodology](../docs/methodology.md). The function evidence index and compact analytical reconstruction are in [static evidence](../evidence/static-evidence.json) and [excerpts](../evidence/static-excerpts.md).

## Authentication context

The firmware checks a camera-issued session token and connection state. The traced admission callback checks internal state rather than a client password. A basic session and model/status queries were accepted during the owner-authorized network test without a separate owner credential. **That observation is not an end-to-end `GET_FILE` test.**

An existing session, recording mode, transfer state, filesystem permissions, or another runtime condition may still affect a specific file request. Local WPA/network access is also a separate prerequisite.

## What is established, and what is not

Established: request-controlled absolute paths pass through the RTOS converter and reach a direct Linux read-open operation without an observed recording-directory allowlist.

Not performed: downloading a private configuration file, network-reading an out-of-directory test file, exercising every listing command, or demonstrating the maximum accessible path set. The earlier Telnet reads used a separate root shell and must not be mislabeled as `GET_FILE` evidence.

## Potential impact

If the operation is reachable in an allowed runtime state, file confidentiality depends on the data service's operating-system access rather than a per-client recording-file policy. Configuration and other readable local files could be exposed. Actual network access to a sensitive file has not been demonstrated here.

## Verification and correction

For independent verification, obtain the exact firmware identified by the manifest and trace the request filename through the listed stages. A controlled hardware test should use an owner-created, non-sensitive file outside the intended recording directory; such a test remains outstanding in this investigation.

Require owner authorization for file access, anchor operations to an explicitly permitted recording root, and enforce the boundary after path resolution. Include symlink and concurrent-path-change cases when implementing the fix. Reject arbitrary absolute paths rather than relying on the official app to send benign ones.

[Back to overview](../README.md)
