# Pilaz bundles

Data files the Pilaz Android app downloads at runtime (roadmap 3.1), so that a bank changing how it
words its notifications — or rearranging the menus where its push alerts are switched on — can be
fixed without shipping an app release.

Public because the app fetches these over plain HTTPS with no credentials. They contain **no user
data of any kind**: `bank_templates.json` is a set of regular expressions, `bank_push_guides.json` a
set of menu paths, `bank_directory.json` a list of app package names and SMS sender addresses — and
all three already ship inside every copy of the app.

| File | What it is |
| --- | --- |
| `bank_templates.json` | Per-bank patterns that turn a bank's notification into an amount, a merchant and a card's last four digits, plus the rules that set a non-movement aside. |
| `bank_push_guides.json` | Per-bank steps for switching transaction alerts on inside the bank's own app, per channel. |
| `bank_directory.json` | **Which apps and which SMS addresses count as a bank at all.** |

They are fetched as three separate requests, deliberately: `formatVersion` refuses a file *whole*,
so a mistake in one cannot take the other two down with it.

## Publishing a fix

1. Edit the file here.
2. **Raise `bundleVersion`.** It is the only thing that makes an installed app adopt the new file: a
   download whose version is not strictly higher than the bundle already in use is discarded. It
   only ever goes up.
3. Leave `formatVersion` alone unless the file's *shape* changed. An app that does not recognise a
   format refuses the whole file and stays on the copy it shipped with — deliberately, so an older
   app cannot half-read a newer template and misread an amount.
4. Push to `main`. `raw.githubusercontent.com` caches for a few minutes; phones check once a day.

Then copy the file back into the app's `app/src/main/assets/` before the next release. A fresh
install runs on the copy inside the APK until its first successful download, and a phone with no
signal runs on it indefinitely. (The reverse is handled automatically: a shipped bundle with a
higher version than the last download wins, so an app update is never held back by a stale file.)

## `bank_directory.json` — the one that decides whether a bank is heard at all

The other two files make a bank's notifications *readable*. This one decides whether they are
**stored in the first place**: an app package name not listed here is dropped before its text is
ever looked at, and so is an SMS from an address not listed here. That is what makes "we only keep
notifications from your bank" a fact about the code rather than a promise — and it is why a wrong
entry is the worst failure this app has. It produces **no symptom**: no crash, no error, just a bank
that never captures, which is indistinguishable from a bank that has not sent anything yet. All six
package names were guessed once and all six were wrong; it took three phases to notice.

- **`bundleVersion` must be at least 2 to have any effect.** The app ships version 1.
- **`institution` must be a key the app already knows**: `banco_industrial`, `bac`, `banrural`,
  `bam`, `gyt_continental`, `promerica`, `google_wallet`, `other`. An entry naming anything else is
  dropped and the rest are kept — admitting a bank the app has never heard of needs a *name* the
  bundle can carry, which is not built yet.
- **An address may be written any way a person would write it.** `"5400 1718"`, `"+502 5400 1718"`
  and `"54001718"` are stored as the same entry, reachable by whichever form Android hands over.
- **Both lists must be non-empty**, or the file is refused whole: accepting a half-empty one would
  switch off a whole channel for every bank at once, silently.
- **Withdrawing an entry works**, and is why this is safe to publish: delete it, raise
  `bundleVersion`, and that app or that sender stops being captured on every phone at its next check.

Unlike the other two there is **no copy to put back into the app's assets** — its fallback is
compiled into the app, because an unreadable template file costs unparsed notifications while an
unreadable whitelist would stop capture for every bank at once.

## What must never be published here

A template written from invented notification text, a guide whose menu path came from a bank's
website rather than from reading it off a phone, or a package name or SMS address written from
anywhere but a real device. A pattern written against mistyped text passes its
own test and matches nothing a bank ever sends; steps that name a menu which does not exist send
people hunting inside an app they already half-distrust. Neither rule gets easier to break because
publishing no longer needs a release — it gets easier to *do*, which is why they are repeated here.

A template that is too eager is worse than one that fails: an unparsed notification keeps its text
and can be retried, while a movement invented from a declined purchase has to be noticed and deleted
by the user. Every template must declare a `reject` pattern.
