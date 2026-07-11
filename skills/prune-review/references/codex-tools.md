# Codex Platform Adaptation

Special instructions for running `prune-review` on Codex.

## Subagent Dispatch Requires Multi-Agent Support

This skill dispatches an independent reviewer subagent in Step 3. On Codex, the subagent toolset (`spawn_agent`, `wait_agent`, `close_agent`) is gated behind the `multi_agent` feature flag. If it is not enabled, the skill cannot proceed past Step 3.

Add the following to your Codex configuration (`~/.codex/config.toml`):

```toml
[features]
multi_agent = true
```

This enables `spawn_agent`, `wait_agent`, and `close_agent` for dispatching subagents.
