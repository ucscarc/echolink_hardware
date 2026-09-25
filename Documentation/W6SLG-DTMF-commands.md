# W6SLG-R DTMF commands

**Configuration checked:** 2026-09-24

**Node:** W6SLG-R, EchoLink 839170

**Software:** installed SvxLink 15.11 build (server 1.5.0), with `SimplexLogic`

This is the command inventory for the **installed W6SLG-R node**. It combines
the active configuration with the [SvxLink 15.11 command
implementation](https://github.com/sm0svx/svxlink/tree/15.11/src/svxlink).
Commands marked *tested* were exercised during the 2026-09 recovery session;
others are present in that version's code and configuration but have not been
tested over RF on this node. The other logic and modules present as example
sections in the configuration file are not active.

Send DTMF during a normal, identified transmission and release PTT after the
final `#`. This node does not enable command execution on squelch close, so
include `#` to terminate each command. Some handhelds lack an A-D keypad;
their users cannot enter the `D` macros. A first `D1#` trial was decoded as
`A1#`, so check for the expected announcement before assuming a macro worked.

## Core and configured shortcuts

| Keys | Action | Evidence |
| --- | --- | --- |
| `*#` | Manual station identification and configured status announcement | Implemented in the active logic; not RF-tested in this session |
| `0#` | Activate Help | Module ID 0; source-verified |
| `01#` | Play Parrot help without activating Parrot | Help module subcommand; source-verified |
| `02#` | Play EchoLink help without activating EchoLink | Help module subcommand; source-verified |
| `1#` | Activate Parrot | Tested; playback heard clearly over RF |
| `2#` | Activate EchoLink | Module ID 2; source-verified |
| `22#` | Announce this node's EchoLink number without opening the module | EchoLink idle subcommand; source-verified |
| `D1#` | Activate EchoLink and connect to *ECHOTEST* node 9999 | Configured macro; tested, with clear returned audio |
| `D9#` | Activate Parrot and have it speak digits `0123456789` | Configured macro; untested |
| `D03400#` | Another configured shortcut to *ECHOTEST* | Configured macro; untested |

The `D` shortcuts are macros, not EchoLink node numbers. A macro will refuse
to switch modules if a different module is already active. Send `#` to leave
the active module first.

## While Help is active

| Keys | Action |
| --- | --- |
| `<module-id>#` | Play help for that module: `0#` Help, `1#` Parrot, `2#` EchoLink |
| `#` | Leave Help |

## While Parrot is active

| Keys | Action |
| --- | --- |
| Speak, then release PTT | Record and play back received audio; no DTMF command needed |
| `0#` | Play Parrot help |
| `<digits>#` | Speak the received digits back |
| `#` | Leave Parrot |

Parrot now deactivates after **20 seconds of inactivity** (`TIMEOUT=20`,
changed from 60). This is an idle timer, not a cap on one recording. The
recording FIFO remains 60 seconds and the playback delay remains 1 second.
These behaviors follow the [15.11 Parrot
manual](https://github.com/sm0svx/svxlink/blob/15.11/src/doc/man/ModuleParrot.conf.5)
and the installed configuration.

## While EchoLink is active

| Keys | Action | Notes |
| --- | --- | --- |
| `<node-number>#` | Connect to that EchoLink node | A numeric node ID of at least four digits, such as `9999#` for *ECHOTEST* |
| `#` | Disconnect the last connected station; if none remains, leave EchoLink | Tested after *ECHOTEST*; with multiple connections, repeat as needed |
| `0#` | Play EchoLink help | Source-verified |
| `1#` | Announce connected stations | Source-verified |
| `2#` | Announce W6SLG-R's node number | Source-verified |
| `31#` | Connect to a random link or repeater | Source-verified; creates an external call |
| `32#` | Connect to a random conference | Source-verified; creates an external call |
| `4#` | Reconnect the most recently disconnected station | Available only when a prior station is remembered |
| `50#` / `51#` | Turn listen-only mode off / on | Source-verified |
| `6*<code>#` | Look up an exact callsign code and read matches | `<code>` is SvxLink's numeric callsign code, not typed letters |
| `6*<code>*#` | Look up callsign codes with that prefix | Results must be narrowed to at most nine matches |
| `7#` | List connected stations for selective disconnect | Requires at least one connection |

For `6*...#` or `7#`, the node reads a numbered list and then waits up to
60 seconds. Send `1#` through `9#` to choose a station, `0#` to repeat the
list, or `#` to cancel. These forms come from the
[15.11 EchoLink module source](https://github.com/sm0svx/svxlink/blob/15.11/src/svxlink/modules/echolink/ModuleEchoLink.cpp);
they have **not** been exercised on W6SLG-R. A normal four-or-more-digit
numeric command is interpreted as a node number when EchoLink is active.
Callsign codes use telephone-keypad letter groups (ABC=2 through WXYZ=9),
retain digits, map punctuation such as `-` to `1`, and omit `*` in conference
names, as implemented by
[`callToCode`](https://github.com/sm0svx/svxlink/blob/15.11/src/echolib/EchoLinkStationData.cpp).

## Commands present in source but not operational here

- `00#` and `00<country-code>#` are recognized as language commands, but the
  installed Tcl handlers explicitly say `NOT IMPLEMENTED`. The installed
  sound set is English only. Do not rely on these for an on-air language change.
- The online/offline DTMF command is disabled (`ONLINE_CMD` is commented out).
- The QSO recording command is disabled (`QSO_RECORDER` is commented out).
- Logic-link commands are disabled because `[GLOBAL] LINKS` is commented out.
- Automatic long-command EchoLink activation is disabled; activate EchoLink
  with `2#` before entering an arbitrary node number.

`*` prefixes a command to send it to the core while another module is active.
This is an advanced parser feature, not a substitute for leaving the current
module cleanly with `#`. Unknown or malformed commands produce a failure or
unknown-command response; they should not be used as a way to probe the node
repeatedly on air.

## Maintenance note

The source of truth is the Pi's active `/etc/svxlink/svxlink.conf` and
`/etc/svxlink/svxlink.d/Module*.conf` files. This document intentionally
contains no EchoLink password, proxy credential, SSH information, or raw
configuration dump. Recheck the inventory whenever modules, macros, IDs, Tcl
handlers, or SvxLink versions change. The 20-second Parrot timer was verified
after a single-process SvxLink restart: the public proxy reconnected,
EchoLink directory status returned to ON, and GPIO PTT was inactive at idle.
