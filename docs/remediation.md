# Defensive recommendations

These recommendations address the observed design boundaries. No modified firmware or vendor fix is distributed by this repository, and no proposed fix has been validated on production releases.

## For the vendor

1. **Remove unnecessary remote administration.** Disable Telnet in production startup, lock empty-password root access, and review serial/diagnostic entry points separately. Any required servicing path should have explicit authenticated activation and an appropriate lifetime.
2. **Authorize clients before privileged requests.** Establish an owner-authentication mechanism rather than treating local reachability or a newly issued token as permission to administer the camera. Tie authorization to the permitted operations.
3. **Confine file operations.** Accept recording identifiers or carefully resolved relative paths under a fixed root. Enforce the boundary after normalization and symlink handling. Keep uploads within a dedicated staging area and separate trusted update installation from general transfer.
4. **Minimize credential exposure.** Remove passwords from status responses and logs. Restrict configuration/cache access and remove obsolete copies when credentials are changed.
5. **Validate the actual product states.** Test production and diagnostic configurations, access-point and station modes, existing sessions, recording transitions, and recovery paths. Confirm behavior on hardware rather than relying only on source-level changes.

## For owners and integrators

Until behavior is verified for the installed release, avoid placing the camera on a shared or untrusted LAN. Use a unique hotspot/Wi-Fi password and network isolation that prevents unnecessary client-to-client access. Disable camera networking when it is not needed, where the product supports that operation without disrupting required functions.

Network isolation reduces exposure; it does not repair passwordless management or unrestricted file paths. Do not assume that an app pairing screen prevents direct access to every service.

No claim is made that a later release is affected or fixed. Obtain an applicable vendor advisory or perform a controlled verification before making that decision.
