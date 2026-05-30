# Community

Vianium runs a small community space focused on the six end-user
products in the ecosystem:

- [`vianium-browser`](https://github.com/vianium/vianium-browser)
- [`vianigram`](https://github.com/vianium/vianigram)
- [`vianium-music`](https://github.com/vianium/vianium-music)
- [`vianium-localsend`](https://github.com/vianium/vianium-localsend)
- [`vianium-chat`](https://github.com/vianium/vianium-chat)
- [`vianium-explorer`](https://github.com/vianium/vianium-explorer)

The internal libraries (Tier 1 and Tier 2) are discussed on GitHub
issues, pull requests, and discussions. The community space exists for
real-time conversation, user support, beta coordination, and
international reach.

## Where to find us

| Need | Channel |
|---|---|
| Real-time chat, user support, beta testing | Discord — invite link in each product README |
| Bug reports | GitHub issues in the respective repository |
| Design conversations | GitHub Discussions in [`vianium-docs`](https://github.com/vianium/vianium-docs/discussions) |
| Security disclosures | See [SECURITY.md](SECURITY.md) — do **not** post in Discord |
| Commercial licensing inquiries (Tier 3) | `hello@angelcareaga.com` |

## Operating language

The primary language is **English** across all channels except the
dedicated international channels listed below. Maintainers will reply
in English by default.

## Server structure

### Roles

| Role | Assignment | Purpose |
|---|---|---|
| Founder | manual | Angel Careaga |
| Maintainer | manual | Co-maintainers per product (when applicable) |
| Beta Tester | manual | Access to pre-release builds via the beta channel |
| Bug Hunter | manual | Awarded to members who file confirmed bugs |
| Member | default | All other members |
| Bots | manual | Bot accounts |

The following roles are self-assigned via reaction roles in the welcome
channel:

| Group | Roles |
|---|---|
| Product interest | Browser, Vianigram, Music, LocalSend |
| Internals interest | Internals (Tier 1 + Tier 2 libraries) |
| Device ownership | WP8.1 owner, W10M owner |
| Language | Italian, Russian, Polish, Spanish |

### Channel categories

**Info**

- `#welcome` — rules, links, reaction roles, permanent invite.
- `#announcements` — announcement channel for releases and news; read-only for members.
- `#roadmap` — current phase status across the six products.
- `#governance` — discussion preceding new ADRs in `vianium-docs`.

**Community**

- `#general` — general discussion.
- `#showcase` — screenshots and videos of the products running on real hardware.
- `#wp81-w10m-zone` — Windows Phone 8.1 and Windows 10 Mobile owners.
- `#off-topic` — anything else, within the Code of Conduct.

**Vianium Browser**

- `#browser-general` — usage, questions, feedback.
- `#browser-bugs` — Forum channel. Tags: `tabs`, `history`, `bookmarks`, `downloads`, `settings`, `credentials`.
- `#browser-dev` — contributors and design discussion.

**Vianigram**

- `#vianigram-general`
- `#vianigram-bugs` — Forum channel. Tags: `mtproto`, `voip`, `ui`, `sync`, `notifications`, `media`.
- `#vianigram-dev`

**Vianium Music**

- `#music-general`
- `#music-bugs` — Forum channel. Tags: `playback`, `yt-music`, `library`, `equalizer`, `palette`, `tags`.
- `#music-dev`

**Vianium LocalSend**

- `#localsend-general`
- `#localsend-bugs` — Forum channel. Tags: `discovery`, `transfer`, `pairing`, `firewall`.
- `#localsend-dev`

**International**

The four international channels reflect the historically strongest
Windows Phone communities and the languages where active retro-WP
audiences still exist:

- `#italiano` — Italy was one of the top markets for Lumia/Windows Phone.
- `#русский` — strong Russian WP community and high Telegram penetration (relevant for Vianigram).
- `#polski` — persistent Eastern European WP community.
- `#español` — combined LATAM and Spain audience.

Additional language channels (e.g. Romanian, Portuguese, German) may
be added later based on demand.

**Shared**

- `#internals-libs` — single channel for discussion of the 18 internal libraries.
- `#design-feedback` — UX and UI mockup feedback.

**Private** (Founder + Maintainers only)

- `#maintainers` — coordination between maintainers.
- `#security-disclosures` — coordination for incoming security reports per [SECURITY.md](SECURITY.md).
- `#beta-channel` — distribution of pre-release builds to Beta Testers.

**Voice**

- `Pair Programming`
- `Bug Bash` — live testing sessions on real WP8.1 / W10M hardware.

## Channel guidelines

- Bugs belong in `#<product>-bugs`, not in `#<product>-general`.
- `#<product>-dev` is for contributors discussing implementation. End-user questions belong in `#<product>-general`.
- The four international channels do not duplicate every category. They are general-purpose spaces in the local language; for bug reports, members are asked to use the product-specific `-bugs` channel (English) so reports are searchable by maintainers.
- The `#beta-channel` is the only authorised place to share pre-release builds. Re-distributing beta builds outside the server is not permitted.

## Licensing reminder

The six products are licensed under
**PolyForm Noncommercial 1.0.0**, unlike the Tier 1 and Tier 2
libraries which are Apache 2.0. Contributions to the products are
accepted under the terms of each repository's `CONTRIBUTING.md` and
follow the [DCO process](contribution-guide.md#developer-certificate-of-origin-dco).
Commercial use of the products requires a separate licence —
contact `hello@angelcareaga.com`.

See [licensing-policy.md](licensing-policy.md) for the full policy.

## Conduct

The community space is governed by the project
[Code of Conduct](CODE_OF_CONDUCT.md). Enforcement is the
responsibility of the Founder and Maintainers. Reports may be sent to
`hello@angelcareaga.com`.

---

## Appendix A — Welcome channel message

Pinned in `#welcome`.

```text
Welcome to the Vianium community.

Vianium is a clean-room native ecosystem for Windows Phone 8.1 and
Windows 10 Mobile, built around six products:

  • Vianium Browser   — github.com/vianium/vianium-browser
  • Vianigram         — github.com/vianium/vianigram
  • Vianium Music     — github.com/vianium/vianium-music
  • Vianium LocalSend — github.com/vianium/vianium-localsend
  • Vianium Chat      — github.com/vianium/vianium-chat
  • Vianium Explorer  — github.com/vianium/vianium-explorer

Before posting:

  1. Read the rules below.
  2. Pick the product roles you care about from the reactions at the
     end of this channel. Channels you do not select will stay hidden.
  3. Pick a device role if you own a WP8.1 or W10M phone.
  4. Pick a language role if you want access to the international
     channels.

Primary language across the server is English. The international
category has dedicated channels in Italian, Russian, Polish, and
Spanish.

Bug reports go in #<product>-bugs (forum channel with tags). General
questions and feedback go in #<product>-general.

Maintained by Angel Careaga — hello@angelcareaga.com
```

## Appendix B — Rules

Pinned in `#welcome` directly after the welcome message.

```text
Server rules

1. Follow the Vianium Code of Conduct.
   github.com/vianium/vianium-docs/blob/main/CODE_OF_CONDUCT.md

2. English is the primary language. Use the dedicated international
   channels for Italian, Russian, Polish, or Spanish conversation.

3. Bug reports go in #<product>-bugs as a new forum post with the
   correct tags. Do not file bugs in #<product>-general or in DMs.

4. Do not share beta builds or pre-release binaries outside the server.
   Builds are distributed only in #beta-channel to members with the
   Beta Tester role.

5. Do not advertise other servers, services, or commercial offerings.

6. Do not post security disclosures publicly. Follow SECURITY.md:
   github.com/vianium/vianium-docs/blob/main/SECURITY.md

7. The six products are licensed under PolyForm Noncommercial 1.0.0.
   Forks or builds for commercial use require a separate licence.
   Contact hello@angelcareaga.com.

8. Maintainers may remove messages, mute, or ban accounts that breach
   these rules or the Code of Conduct.
```

## Appendix C — Product pinned messages

Pinned in each `#<product>-general`.

### Vianium Browser

```text
Vianium Browser — clean-room native web browser for WP8.1 / W10M.

Repository:    github.com/vianium/vianium-browser
Licence:       PolyForm Noncommercial 1.0.0
Minimum spec:  WP8.1, ARMv7, 512 MB RAM

Channels for this product:
  • #browser-general — usage, questions, feedback (you are here)
  • #browser-bugs    — report bugs as a forum post with tags
  • #browser-dev     — contributors and implementation discussion

Before reporting a bug, please search existing posts in #browser-bugs
and check the README for known limitations.
```

### Vianigram

```text
Vianigram — clean-room native Telegram client for WP8.1 / W10M.

Repository:    github.com/vianium/vianigram
Licence:       PolyForm Noncommercial 1.0.0
Minimum spec:  WP8.1, ARMv7, 512 MB RAM

Channels for this product:
  • #vianigram-general — usage, questions, feedback (you are here)
  • #vianigram-bugs    — report bugs as a forum post with tags
  • #vianigram-dev     — contributors and implementation discussion

Vianigram is not affiliated with Telegram Messenger LLP. Login uses
your existing Telegram account. Do not share session strings or
2FA codes in public channels.
```

### Vianium Music

```text
Vianium Music — native music player with YouTube Music streaming.

Repository:    github.com/vianium/vianium-music
Licence:       PolyForm Noncommercial 1.0.0
Minimum spec:  WP8.1, ARMv7, 512 MB RAM

Channels for this product:
  • #music-general — usage, questions, feedback (you are here)
  • #music-bugs    — report bugs as a forum post with tags
  • #music-dev     — contributors and implementation discussion

Streaming functionality depends on third-party services. Availability
of remote sources is not guaranteed and may change without notice.
```

### Vianium LocalSend

```text
Vianium LocalSend — P2P file sharing on the local network for WP8.1 / W10M.

Repository:    github.com/vianium/vianium-localsend
Licence:       PolyForm Noncommercial 1.0.0
Minimum spec:  WP8.1, ARMv7, 512 MB RAM

Channels for this product:
  • #localsend-general — usage, questions, feedback (you are here)
  • #localsend-bugs    — report bugs as a forum post with tags
  • #localsend-dev     — contributors and implementation discussion

LocalSend transfers happen directly between devices on the same
network. The protocol is compatible with the upstream LocalSend
specification.
```

---

## Author

**Angel Careaga**
`hello@angelcareaga.com` · [angelcareaga.com](https://angelcareaga.com) · [@AngelCareaga](https://github.com/AngelCareaga)
