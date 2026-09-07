# Fork notes: LuismaSP89/xenia-edge_Luisma

This fork tracks [has207/xenia-edge](https://github.com/has207/xenia-edge) and
differs from it by a single feature commit:

**[D3D12] Restore the memexport readback path behind `readback_memexport`**

Upstream edge removed the memexport readback path in `a84f4d1ff` (2026-07-15)
and replaced it with two-buffer memexport routing. Some titles (LEGO The Lord
of the Rings, `5752081D`) render garbage with the new scheme. This fork adds the
old path back as an opt-in cvar, the same mechanism Xenia Canary uses:

```toml
[GPU]
readback_memexport = true
```

With it enabled, memexport draws write the device buffer, the exported ranges
are copied to a readback buffer, the GPU is awaited and the data is written into
guest RAM right after the draw. Two-buffer routing is disabled for the title.
Off by default. Direct3D 12 only.

The official `xenia.exe` deletes unknown cvars when it saves the config, so
the line has to be re-added after running an official build.

## Layout

- `edge` (default): upstream `edge` + the feature commit + this file +
  `sync-upstream.cmd` + the CI tweak that lets this repository publish
  releases. Always rebased, never merged, so `git log origin/edge..edge` shows
  exactly the fork's changes.
- Releases: one per build, tagged with the short commit hash, created by the
  `Create Release` job of `.github/workflows/CI.yml`.

## Updating to the latest upstream edge

Run `sync-upstream.cmd` from the repository root. It:

1. fetches `origin/edge` (has207),
2. rebases `edge` onto it,
3. force-pushes `edge` to `fork` (this repository) with `--force-with-lease`,
4. starts the CI workflow with `gh workflow run` (pushes alone do not trigger
   it on this fork).

When the run finishes, the Windows build is in the new release under
https://github.com/LuismaSP89/xenia-edge_Luisma/releases, and also as the
`xenia_edge_windows` artifact of the run.

If the rebase stops with conflicts, upstream changed the code the feature
commit touches (`src/xenia/gpu/d3d12/d3d12_command_processor.{cc,h}` and
`src/xenia/gpu/command_processor.cc`). Resolve them, `git add`, then
`git rebase --continue` and re-run the script.

## Remotes expected by the script

```
origin  https://github.com/has207/xenia-edge.git
fork    https://github.com/LuismaSP89/xenia-edge.git   (redirects to xenia-edge_Luisma)
```
