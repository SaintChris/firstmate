Mode: Codex foreground checkpoint.

When this session owns supervision and away mode is not active:
1. Drain first with `bin/fm-wake-drain.sh`.
   After handling all emitted wakes and reconciling open decisions and unread status lines, run the exact `--ack-through` command printed as `WAKE_ACK_REQUIRED`; until then the work remains durable for idempotent re-handling after interruption.
2. Source `__FM_X_MODE_ENV__` first when Relay is active.
3. The tracked Codex Stop hook owns one bounded foreground watcher checkpoint whenever supervision is needed and no healthy watcher is live.
   Its timeout is 900 seconds and it rejects a configured checkpoint over 840 seconds, leaving room for shutdown grace.
4. An actionable checkpoint resumes Codex with the watcher result; drain, handle, and run the exact acknowledgement command before continuing.
5. A quiet checkpoint resumes one continuation without permitting a blind stop; the next Stop hook owns the successor checkpoint.
6. Never use shell `&` or Codex background tasks for firstmate watcher supervision.
7. Do not run `bin/fm-watch-arm.sh` as Codex's normal supervision command.
   If it is ever shelled anyway, a backgrounded, piped, or bundled anti-pattern is denied automatically by the PreToolUse seatbelt (`bin/fm-arm-pretool-check.sh`) registered in `.codex/hooks.json`.
8. A failed or unavailable hook-owned checkpoint refuses the Stop and reports the failure; correct the local failure before ending the turn.

Codex cannot reason while its hook-owned foreground checkpoint is running.
The bounded checkpoint returns control regularly without relying on background-task wake semantics.
