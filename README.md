# Pilaz bundles

Data files the Pilaz Android app downloads at runtime (roadmap 3.1), so that a bank changing how it
words its notifications — or rearranging the menus where its push alerts are switched on — can be
fixed without shipping an app release.

Public because the app fetches these over plain HTTPS with no credentials. They contain **no user
data of any kind**: `bank_templates.json` is a set of regular expressions, `bank_push_guides.json` a
set of menu paths, and both already ship inside every copy of the app.

| File | What it is |
| --- | --- |
| `bank_templates.json` | Per-bank patterns that turn a bank's notification into an amount, a merchant and a card's last four digits. |
| `bank_push_guides.json` | Per-bank steps for switching transaction alerts on inside the bank's own app. |

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

## What must never be published here

A template written from invented notification text, or a guide whose menu path came from a bank's
website rather than from reading it off a phone. A pattern written against mistyped text passes its
own test and matches nothing a bank ever sends; steps that name a menu which does not exist send
people hunting inside an app they already half-distrust. Neither rule gets easier to break because
publishing no longer needs a release — it gets easier to *do*, which is why they are repeated here.

A template that is too eager is worse than one that fails: an unparsed notification keeps its text
and can be retried, while a movement invented from a declined purchase has to be noticed and deleted
by the user. Every template must declare a `reject` pattern.
