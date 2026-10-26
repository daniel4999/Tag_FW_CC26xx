# Changelog

## v0.1.0 — 26 October 2026

Community release of the CC2630 short-wait fix.

- Guard the wait-minus-20 subtraction so a nonzero wait below 20 ms cannot become
  a multi-day unsigned delay.
- Add a source-derived host C regression covering all 65,536 uint16 wait values.
- Validate a patched spare with distinct images and later check-ins, including
  the published official OpenEPaperLink 3.00 / C6 0024 application.
- Provide full and application-only wired-flash images, instructions and SHA256 checksums.
- Keep tag pushes as build-only workflows; reviewed releases are published
  manually so automation cannot overwrite published metadata or assets.

The tag still advertises software version 0. The release adds no OTA updater or
tag-config handler. No battery-current or long-term reliability claim is made.
Original and third-party licenses remain in force.
