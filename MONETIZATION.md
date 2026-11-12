# Monetization plan — Cool New Game

This is a plan for Brandon (owner), not a live checkout. The browser game has no real payment integration. Gem packs call an adapter that logs the product id and returns `iap_unavailable`.

Default recommendation, used by the current build: **real-money IAP sells gem packs and cosmetics. Weapons, troops, and the tank stay fully earnable by playing.** Gems are a shortcut, not a key.

## 1. Gem economy

### How gems are earned

Gems drop straight into the wallet saved in this browser.

| Source | Amount |
| --- | --- |
| Grunt or runner kill | 1 |
| Brute kill | 3 |
| Elite kill | 4 |
| Commander kill (every 10th wave) | 12 |
| Wave clear | 4 + wave number |
| Daily reward | 20, once per local calendar day |

A run that clears the first ten waves earns on the order of a couple hundred gems from kills and wave bonuses, before counting tougher enemy mixes. That is enough to buy a weapon or a skin. The tank at 500 gems is a longer save, and clearing wave 10 unlocks it with no gems at all.

### What the store sells

| Product id | Item | Price | Kind |
| --- | --- | --- | --- |
| `com.coolnewgame.extra_life` | Extra life | 25 gems | Consumable, this run only |
| `com.coolnewgame.weapon.machinegun` | Rapid Rifle | 100 gems | Non-consumable |
| `com.coolnewgame.weapon.shotgun` | Spread Shotgun | 175 gems | Non-consumable |
| `com.coolnewgame.weapon.cannon` | Heavy Cannon | 250 gems | Non-consumable |
| `com.coolnewgame.troops.multiplier` | Troop multiplier | 150 gems | Non-consumable |
| `com.coolnewgame.tank` | Tank | 500 gems | Non-consumable |
| `com.coolnewgame.skin.desert` | Desert Tan | 50 gems | Non-consumable cosmetic |
| `com.coolnewgame.skin.night` | Night Ops | 120 gems | Non-consumable cosmetic |
| `com.coolnewgame.skin.jungle` | Jungle | 180 gems | Non-consumable cosmetic |
| `com.coolnewgame.gems.100` | 100 gems | Real-money pack (placeholder) | Consumable |
| `com.coolnewgame.gems.550` | 550 gems | Real-money pack (placeholder) | Consumable |
| `com.coolnewgame.gems.1200` | 1200 gems | Real-money pack (placeholder) | Consumable |

Suggested store prices for the gem packs, once billing exists: 100 gems at $0.99, 550 at $4.99, 1200 at $9.99. Those are drafts, not live prices.

### Design rules

- Paying is not required to field the rifle, shotgun, cannon, troop multiplier, or tank. Wave clears unlock them, and the in-run Subtract / points paths can equip guns and the tank earlier.
- Divide, Multiply, and Subtract spend **points** earned in the current run. Points are not gems and are not sold.
- Extra lives are cheap on purpose and capped: 2 purchases per run, and never more than 5 lives held. A purchase cannot make a run immortal.
- Skins do not change damage, fire rate, or armor. They are the cosmetic sink.
- Gem drops and prices are constants in `index.html` (`ENEMY_TYPES`, `PRODUCT_CATALOG`, `MILESTONES`). Change them together and keep this file in step.
- The daily reward is a reason to reopen the game, not a day's wages. Twenty gems is short of an extra life (25) and far short of a weapon.

## 2. IAP abstraction

One reward function applies every grant: kills, wave bonuses, the daily reward, gem purchases, point unlocks, wave milestones, and (later) a verified store receipt.

```text
UI or combat
    -> Commerce.purchase(productId)     // gems, or the IAP adapter
    -> Commerce.grantReward(productId)  // the only writer of unlocks, gems, and lives
```

Combat code reads `Save.data.unlocks` and `Save.data.gems`. It never calls a store API. The store buttons call `Commerce` only.

`IapAdapter` is a stub named `BrowserIapAdapter`:

- `purchase(productId)` logs `[IapAdapter] purchase` plus the product id and returns `{ ok: false, status: 'iap_unavailable', productId }`.
- `restore()` logs `[IapAdapter] restore` and returns `{ ok: false, status: 'iap_unavailable' }`.

Swap it without rewriting the shop:

```js
CoolNewGame.setIapAdapter({
  purchase(productId) { /* StoreKit / Play Billing */ },
  restore() { /* owned non-consumable ids */ }
});
```

Expected statuses are `success`, `iap_unavailable`, `cancelled`, and `failed`. `Commerce.purchase` calls `grantReward` only after `status: 'success'`.

When a real adapter ships:

- **Receipt validation.** Do not treat a client-side success flag as proof of payment. Send the receipt to a server you control, verify it with Apple or Google, then call the same `grantReward(productId, { source: 'iap' })`.
- **Restore purchases.** `restore()` should return the player's owned **non-consumable** product ids (weapons, troop multiplier, tank, skins). `Commerce.restore` re-grants those idempotently. Do not restore consumable gem packs or extra lives; that would duplicate a spent purchase.
- Placeholder ids above are the catalog. Add new items as catalog rows, not as one-off balance edits.

## 3. Store listing requirements

Figures below are from the linked official pages. Confirm them in the consoles before paying anyone, because regional prices and fee programs move.

**Apple Developer Program.** Enrollment is required to distribute on the App Store. The annual fee is **99 USD** (local currency where Apple offers it). The Enterprise program is a different product at 299 USD and is not what this game needs.

- https://developer.apple.com/help/account/membership/program-enrollment/

**Apple commission.** The standard commission on paid apps and in-app purchases is **30%**. The [App Store Small Business Program](https://developer.apple.com/app-store/small-business-program/) reduces that to **15%** for developers who earned up to 1 million USD in proceeds in the prior calendar year, and for developers new to the store. If a member passes 1 million USD in the current year, the standard rate applies to future sales. Apple's announcement states both numbers directly: a reduced 15 percent commission, and "The App Store’s standard commission rate of 30 percent remains in place" above that threshold.

- https://developer.apple.com/app-store/small-business-program/
- https://www.apple.com/newsroom/2020/11/apple-announces-app-store-small-business-program/

**Google Play registration.** A Play Console developer account has a **US$25 one-time** registration fee. Personal accounts also have identity and testing requirements before an app can be distributed publicly.

- https://support.google.com/googleplay/android-developer/answer/6112435

**Google Play service fee.** There is not one global rate. The maintained article is Play Console's service-fee page:

- https://support.google.com/googleplay/android-developer/answer/112622

For markets that article still lists on the previous model, developers in the 15% tier pay **15% on the first $1M (USD) of earnings each year and 30% above that**. Subscriptions have their own 15% rate.

Google has been rolling out a 2026 model in some regions that splits a **service fee** from a **billing fee**. The same Play Console article and the Android developers blog describe it. In the regions already on that model, the first $1M of earnings is a 10% service fee, plus a separate billing fee when the purchase uses Google Play Billing (5% in the US, UK, and EEA as published there). Added together, Play Billing on that first $1M is often still around 15%. Other transactions, program enrollments, and "new install" versus "existing install" rates differ, and later regions have later dates. Use the official tables rather than a single percentage.

- https://support.google.com/googleplay/android-developer/answer/112622
- https://developer.android.com/blog/posts/expanded-billing-choice-and-lower-fees-on-google-play

## 4. Wrapping the HTML5 game for the stores

The App Store and Google Play do not accept a raw `index.html` upload. Ship the same file inside a native WebView shell. **Capacitor is the preferred wrapper.** Apache Cordova can do the same job if you already have a Cordova project; Capacitor is the one to start with.

The shape of the work, later, not in this repo:

1. Keep this directory as the web root (`index.html` plus the two docs). No bundler is required.
2. Create a Capacitor app, set `webDir` at this folder, and add the iOS and Android platforms.
3. Replace `BrowserIapAdapter` with a Capacitor plugin that speaks StoreKit on iOS and Play Billing on Android. `Commerce.grantReward` and the store UI stay as they are.
4. Add icons, splash screens, a privacy policy URL, and the age/content questionnaire. This game is stylized combat with no accounts and no tracking in the browser build.
5. Test purchases in App Store Sandbox and Play license-test accounts before review.

Cordova's equivalent is a `config.xml` project with the In-App Purchase plugin of your choice, loading the same `index.html`. The commerce contract above does not change between the two wrappers.

Official starting points:

- https://capacitorjs.com/docs/
- https://cordova.apache.org/docs/en/latest/

## 5. Recommended launch sequence

1. **Play the free browser build first.** Hand `index.html` to a few people. Tune wave pacing, point costs, and gem drops until a run is fun without spending. The browser game is the whole combat loop; the stores do not make it more fun.
2. **Add IAP only after that.** Turn on gem packs and the cosmetic skins for real money. Leave weapons, the troop multiplier, and the tank earnable. Validate receipts on a server before granting.
3. **Then wrap and list.** Capacitor (or Cordova), store listings, sandbox purchases, then App Store and Play review. Do not pay the annual Apple fee or start a store listing until the browser game is a game you would play again.

## 6. Open questions for Brandon

1. Are the draft gem-pack prices ($0.99 / $4.99 / $9.99) the ones you want on the stores, or do you want a single $1.99 pack until you see what people actually buy?
2. Should any combat upgrade be gem-only? The current default is no: IAP sells gem packs and cosmetics, and weapons, troops, and the tank remain fully earnable by playing. Say if you want the tank, or the cannon, to be purchase-only instead.
3. Is an extra life allowed as a real-money consumable later, or should lives stay gem-only so a purchase cannot be spammed past the per-run cap without a gem sink?
4. Do you want a global online leaderboard, or is "top 5 on this device" enough for launch? Online scores want accounts and cheat resistance; the local list does not.
5. Personal or organization store accounts? Apple's seller name and Google's testing rules differ. The $99/year and $25 fees above assume the accounts are yours.
6. Any region you will not ship in at launch? The 2026 Play fee split is regional, and Apple's small-business rate depends on worldwide proceeds, so the first listing's country list matters.
