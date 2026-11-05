# Gea

**A web OS with Spaces — one place, several rooms.**

A browser shell for your work, plus a small local service that gives it real files, scoped per
space, with policy actually enforced. Local-first, offline-capable, and free.

Geometry for complexity. Aero for anyone. A matrix of spaces.

---

## Why this exists

Most "web desktops" are one of two things: a demo that stores nothing real, or a remote desktop
streamed from someone else's server. The first can't hold your work. The second isn't yours, needs
a network, and bills by the hour.

Gea is the third option. The shell is a single HTML file that runs in any browser. The service is
a few hundred lines of Python with no dependencies. Your files are on your disk, in your spaces,
and nothing leaves the machine.

## Run it

```sh
python3 spacesd.py --ui index.html
```

Then open <http://127.0.0.1:8471>. No pip install, no build step, no container.

On a phone or tablet, `index.html` also opens standalone and runs in **local mode**: the same
Spaces mechanic with state in `localStorage` and no real files. The badge in the header says which
mode you are in, so the two are never confused.

## What the service does

| endpoint | what it does |
|---|---|
| `GET /` | serves the shell |
| `GET /api/state` | spaces and current policy |
| `GET /api/fs?space=&path=` | lists a directory inside one space |
| `POST /api/write` | writes a file inside one space |
| `POST /api/policy` | grants or revokes a capability for one space |
| `GET /api/audit?limit=` | recent audit events, newest first |

Each space is a directory under `--root` (default `~/gea-spaces`), with `policy.json` and an
append-only `audit.log` beside them. The service binds loopback only.

## The two rules the code enforces

**Off unless granted.** A write in a space without the `files` grant is refused with 403, the file
is not created, and the refusal is written to the audit log. A refusal that leaves no trace is
indistinguishable from a bug.

**A path cannot leave its space.** Absolute paths and any parent-directory segment are refused
outright; a resolved-path containment check backs that up. An earlier version silently re-rooted
them inside the space, which is contained but hides caller bugs — an API boundary should refuse a
nonsense path, not quietly reinterpret it.

## Not here yet, on purpose

No AI, no shell execution, no authentication, no network egress. Each should arrive with its policy
check already written rather than bolted on afterwards.

## Tests

```sh
python3 test_spacesd.py
```

28 assertions: permitted writes, refusals, per-space isolation, path escape, unknown inputs, and
persistence across a restart. Run by `.github/workflows/ci.yml` on two Python versions, alongside
checks that the shell is genuinely self-contained, that its JS parses, and that its tags balance.

## Credit

The idea of a browser-native OS with separated workspaces was inspired by
**[WarmWind OS](https://warmwind.com/)**, which does something related and quite different: a
custom Linux distribution server-side, streaming a desktop to the browser over Wayland and VNC,
sold per work-hour.

Gea takes the opposite trade-off on purpose. No streaming, no server, no account — a native web
shell talking to a local service. That's what makes free, offline and private possible here, and
it's also what it can't do: it runs web-shaped things, not arbitrary desktop applications.

No code was taken from WarmWind. The inspiration is the idea, and it's credited because that's
cheap and honest.

## Licence

**GNU Affero General Public License v3.0** — see [LICENSE](LICENSE).

AGPL means: use it, change it, self-host it, ship it — but if you run a modified version as a
network service, you have to publish your changes. That's deliberate. The whole point of this
project is that it stays yours and stays open, including against the business model of streaming
other people's computers back to them.

Contributions are accepted under AGPL-3.0. By opening a pull request you agree your contribution
is licensed under the same terms.
