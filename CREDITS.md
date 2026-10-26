# Credits

## Upstream

This fork is based on [tsl0922/Tag_FW_CC26xx](https://github.com/tsl0922/Tag_FW_CC26xx).
Original firmware authorship, third-party notices and license terms are preserved.

## CC2630 short-wait bug

**Daniel (@daniel4999)** discovered the post-upgrade failures, carried out the
hardware reproduction and flashing work, and physically validated the fixed tag.
The source investigation, minimal patch and regression tools were developed in
that troubleshooting session with Codex assistance.

Suggested attribution when incorporating the fix:

```text
Reported-by: Daniel (@daniel4999)
Tested-by: Daniel (@daniel4999)
```

The technical report records the source-level defect, controlled radio comparisons,
and the successful one-tag test with the published official radio application.
