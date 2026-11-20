# Rödd starts from its own home, not from the pin's apps/rodd

## What was wrong

`start.sh` refused every start when it ran from the apps home (`~/.local/share/ymir-apps/rodd/current`):

> Rödd refuses to start — the pin <runtime> carries no apps/rodd — run bin/engine/runtime-deploy.sh

Since the split (Ymir PR #823) the pinned Ymir runtime no longer carries `apps/rodd`, by design. The window
lives in its own home, but step 0 of `start.sh` still demanded the pin's copy and handed over to it. The unit
`ymir-rodd.service` had failed this way on every start since 2026-11-13.

## What changed

- Step 0 (the pin check and hand-over) now runs only when the window still sits inside a Ymir tree (the old
  road, `IN_YMIR_TREE=1`).
- In its own home the window runs where it stands: its code is the release folder, and `YMIR_RUNTIME_DIR` is set
  to the Ymir that `scripts/ymir-root.sh` found. That Ymir serves only Ymir's doors: the voice door, the head,
  the toolbox and the desk check.
- `start.sh --install-unit` in its own home runs `activate.sh`, which renders the same unit from
  `electron/ymir-rodd.service.in`. Before, it fell through and started the window.

## How it was proved

A release of this change was built and flipped by Ymir's `bin/apps/apps.sh update rodd` (against a scratch apps
home whose `origin/main` held this commit). `systemctl --user start ymir-rodd.service` then stayed active, and its
journal ended with the window and toolbox lines.
