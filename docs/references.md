# References and attribution

The following public research informed the comparison. Their reported results belong to their stated products and test scope.

| Project | Reviewed revision | Relevance |
|---|---|---|
| [turretsec/disclosure-thinkware-u3000](https://github.com/turretsec/disclosure-thinkware-u3000) | [`2c8d076b1c81e0fe9d42a46b09d940f86a7166ec`](https://github.com/turretsec/disclosure-thinkware-u3000/tree/2c8d076b1c81e0fe9d42a46b09d940f86a7166ec) | U3000 file read/write and Wi-Fi response findings; related CVEs are not assigned here to U1000 |
| [turretsec/u3000py](https://github.com/turretsec/u3000py) | [`d4372401c93006712a9e8f017ff06bac63f978bb`](https://github.com/turretsec/u3000py/tree/d4372401c93006712a9e8f017ff06bac63f978bb) | Command IDs and client/protocol documentation; not asserted to be fully U1000-compatible |
| [geo-chen/Thinkware-Dashcam](https://github.com/geo-chen/Thinkware-Dashcam) | [`2ea5dcf3b927b1647addb228cffaafde778b1644`](https://github.com/geo-chen/Thinkware-Dashcam/tree/2ea5dcf3b927b1647addb228cffaafde778b1644) | F800 Pro local management issues and a separate Cloud Android APK issue |

The U3000 report lists CVE-2026-101053, CVE-2026-101054, and CVE-2026-101055. The F800 Pro/app report lists other CVEs. This repository does not extend their official affected-product lists or claim those identifiers for the U1000.

Credit for the related U3000 disclosure goes to Ryan Moore (**turretsec**), and credit for the F800 Pro/app research goes to **geo-chen**. The U1000 artifact analysis and hardware observations in this repository were performed independently using owner-authorized equipment, with those projects as comparison sources.

Tooling included REA, Ghidra 12.1.4, Capstone, SquashFS extraction utilities, and FFmpeg for the unsuccessful RTSP check. Tool use is not third-party endorsement or independent validation of this report.
