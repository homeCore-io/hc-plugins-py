# hc-plugins-py

homeCore plugins written in Python. One directory per plugin, one CI, one
release workflow — mirroring what `core/plugins/` does for the Rust ones.

A plugin here is not a binary homeCore unpacks and runs. It ships as a
**hermetic artifact** — the plugin, the SDK and every dependency as wheels —
that a **plugin runtime** installs offline and supervises. See
`docs/pluginRuntimesPlan.md` in core for the whole design.

## Why one repository

Thirteen near-identical Rust plugin repos each carried a copy of the same
workflow and drifted, which is why they were folded into core. Standing up a
fresh repo per Python plugin would repeat that one language later.

## Layout

```
plugins/
  hc-<name>/
    pyproject.toml      # [project] + [tool.homecore]
    hc_<name>/          # the package
    tests/
```

## A plugin's manifest

`pyproject.toml` carries what a Cargo.toml carries, in the places Python
already puts them:

```toml
[project]
name        = "hc-foo"
version     = "0.2.1"
description = "Foo lights, over the Foo cloud API"   # → the registry card

[tool.homecore]
id         = "plugin.foo"
runtime    = "python"
entrypoint = "hc_foo.main"
```

`entrypoint` is what the runtime's adapter launches as `python -m <entrypoint>
<config>`. Every homeCore plugin, in every language, receives its config path
as `sys.argv[1]` — `PluginBase.from_config()` reads it.

## Releasing

Tag `hc-<name>-v<version>`. `python-release.yml` in hc-scripts builds one
artifact per architecture, **installs it offline into a clean venv and waits
for the plugin to register** before publishing, then dispatches to the
registry and reads the served index back.

That install-and-run step is the gate. "It compiled" does not exist as a signal
here, and a well-formed tarball proves nothing about whether the plugin starts.

## The interpreter is pinned centrally

The Python version and manylinux baseline are defaults in **hc-scripts**, not
per plugin, so bumping the interpreter moves the whole fleet in one edit and no
plugin drifts onto its own.
