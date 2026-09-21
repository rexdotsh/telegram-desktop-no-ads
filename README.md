# telegram-desktop-no-ads

Telegram Desktop for Windows, built from source with the ads stripped out. Nothing else is changed.

GitHub Actions does the building. You download a zip with `Telegram.exe` in it.

This is the Windows counterpart to [telegram-desktop-no-ads-pkgbuild](https://github.com/rexdotsh/telegram-desktop-no-ads-pkgbuild), which does the same thing as an Arch package.

## What gets patched

`remove-ads.patch` is a four commit series against the official tdesktop source:

| # | Change | Effect |
|---|---|---|
| 1 | `SponsoredMessages::canHaveFor(History*)` returns `false` | No sponsored messages in channels or bot chats |
| 2 | `SponsoredMessages::canHaveFor(HistoryItem*)` returns `false` | No sponsored ads over videos |
| 3 | Drop the `setupTopBarSuggestions()` call | No premium upsell or birthday bar above the chat list |
| 4 | `PeerSearch::Type::WithSponsored` becomes `JustPeers` | No sponsored channels injected into search results |

Commits 1 to 3 come from [vehlwn/tdesktop `feature/remove-ads`](https://github.com/vehlwn/tdesktop/tree/feature/remove-ads). Commit 4 is new here.

**This is an ads-only subset, deliberately.** The upstream vehlwn branch also disables reactions, voice and video calls, voice recording, stories, link previews, pinned bars, and more. Those are opinionated UX changes rather than ad removal, so they are left out. If you want the full set, use the branch directly.

Because the patch only ever flips behaviour to "off", it stays small and tends to survive version bumps.

## Building

1. Fork or clone this repo.
2. Actions tab, **Build Windows**, **Run workflow**. Set the tdesktop version if you want something other than the default.
3. Download the artifact when it finishes.

Tagging a commit `v7.1.3` runs the same build and publishes a GitHub Release.

### The first build takes several runs

Telegram Desktop vendors its entire dependency stack, Qt included. A cold build of those libraries takes well over the six hour job limit on a GitHub runner.

The workflow handles this by design. The libraries step has a five hour cap and is allowed to fail, and every run saves its progress to the cache under a run-unique key. When a run ends without finishing, it fails with a message telling you to run it again. Each subsequent run picks up where the last one stopped.

Expect roughly **two to four runs** before you get an `.exe`. After that, cache hits make a rebuild take around an hour.

To warm the cache without attempting the Telegram build, tick **libraries_only**.

> Caches count against the 10 GB per-repository limit. If builds start going backwards and re-doing work, clear old caches under Actions, Caches.

## Notes

**Auto-update is disabled.** It has to be. Otherwise Telegram would quietly replace your patched binary with the official one and the ads would come back. You update by running the workflow again on a newer version.

**API credentials.** The build defaults to the public credentials that tdesktop's own nightly builds use. Telegram rate-limits these and may refuse logins on them. To use your own, get them from [my.telegram.org](https://my.telegram.org) and add repository secrets `TDESKTOP_API_ID` and `TDESKTOP_API_HASH`.

**Not signed.** SmartScreen will warn on first launch.

**x64 only.** No 32-bit or ARM64 target.

## Bumping the tdesktop version

Run the workflow with a newer version. If the patch no longer applies, the "Apply the ad removal patch" step fails with a rejected hunk and you fix the context in `remove-ads.patch`. Changing `prepare.py` upstream invalidates the library cache, so a version bump may mean warming the cache again.

## Licence

Telegram Desktop is GPL-3.0 with an OpenSSL exception, and the patch is a derivative of GPL-3.0 code, so the same terms apply. This repository is not affiliated with Telegram.
