# replicix/fuser

[cberner/fuser](https://github.com/cberner/fuser) 0.18.0 with the changes
replicix projects (constellation, xcheckfs) need, kept as one branch per
change so each can be offered upstream on its own.

| branch | base | what |
|---|---|---|
| `handover` | `v0.18.0` | resume an initialised connection (`Session::from_fd_resumed`, `NegotiatedInit`), detach without unmounting (`Session::detacher`, `run_detachable`) |
| `io-uring` | `handover` | FUSE-over-io_uring transport behind the `io-uring` feature (off by default), from the [Skory/fuser](https://github.com/Skory/fuser) fork |
| `ring-idle-wait` | `io-uring` | bounded ring idle wait, stranded-reply and held-entry reporting (`RingHealth`) |
| `interrupt` | `v0.18.0` | `Filesystem::interrupt` instead of answering `FUSE_INTERRUPT` with `ENOSYS` |
| `no-fork` | `v0.18.0` | spawn `fusermount3` without `fork` (its socket as stdin) for a mount without `auto_unmount` |
| `replicix` | — | `ring-idle-wait` + `interrupt` + `no-fork` merged: what the projects build |
| `master` | — | mirrors upstream `master` |

`io-uring` depends on `handover` (its session changes build on the
detachable loop); it is offered upstream after `handover`, or rebased off
it.

## Using it

```toml
[patch.crates-io]
fuser = { git = "https://github.com/replicix/fuser", rev = "<replicix commit>" }
```

Pin a `rev`, never the branch. The `io-uring` feature has to be enabled
by the user crate (`fuser/io-uring`).

## Changing it

Change the branch the change belongs to, then rebuild `replicix`
(merge the changed branch into it, or recreate it: `ring-idle-wait`, then
merge `interrupt` and `no-fork`), run `cargo test --lib` and
`cargo test --lib --features io-uring` (the real-kernel ring tests run
on a host with `fuse.enable_uring=Y`), push, and bump the `rev` in the
projects.

## A new upstream release

Rebase each branch onto the new tag (`handover` and `interrupt` first,
then `io-uring` onto `handover`, `ring-idle-wait` onto `io-uring`),
recreate `replicix`, and run both test suites and the projects' handover
and ring tests.

## Why each change is there

[`replicix/PATCHES.md`](replicix/PATCHES.md) is the full rationale and
verification record (written when these were patch files vendored in
constellation; `patches/000N-*.patch` there are the branches above, in
order). [`replicix/UPSTREAM-ISSUES.md`](replicix/UPSTREAM-ISSUES.md) has
the upstream issues ready to file, with reproducers.
