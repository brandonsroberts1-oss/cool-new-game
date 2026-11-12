# Cool New Game

A sidewalk shooter you can open in a browser. One soldier — then a squad, then a tank — strafes the near edge of a road and fires into an endless march of enemy soldiers. Waves get faster, tougher, and more crowded. Points buy arithmetic upgrades during the run. Gems, unlocks, and scores stay in this browser.

No build step, no bundler, and no network. The whole game is `index.html`.

## Run locally

Open `index.html` in any current browser (Chrome, Firefox, Safari, or Edge). Double-click the file, or drag it into a browser window.

A tiny static server works too, if you would rather use `http://localhost`:

```bash
python3 -m http.server 8080
```

Then visit `http://localhost:8080`. Nothing else needs to be installed.

The game plays offline. It does not load scripts, images, or fonts from the network.

## Controls

Keyboard only for this version.

| Action | Keys |
| --- | --- |
| Move left / right | `A` / `D`, or Left / Right arrow |
| Shoot | Hold `Space` |
| Upgrades | `U`, or the Upgrades button |
| Gem store | `B`, or the Store button |
| Pause | `P` or `Esc` |
| Mute | `M` |
| Start, resume, or play again | `Enter` |

`Space` does not scroll the page. Arrow keys do not scroll the page while a run is being played.

With the upgrade panel open:

| Key | Upgrade |
| --- | --- |
| `1` | Divide |
| `2` | Multiply damage |
| `3` | Multiply fire rate |
| `4` | Subtract (next weapon) |
| `5` | Tank |
| `Esc` | Close the panel |

Line your troops up with an enemy's lane. Anything that reaches the bottom of the road costs a life. You start with 3 lives. At 0, the run is over.

## Waves and enemies

Each wave spawns more soldiers, then clears when the march is gone. Later waves are faster and hit harder. New types join the march over time. Every tenth wave sends a commander.

| Enemy | Shows up | Feel |
| --- | --- | --- |
| Grunt | Wave 1 | Standard green soldier |
| Runner | Wave 3 | Thin, yellow, fast |
| Brute | Wave 5 | Large red soldier with a lot of health |
| Elite | Wave 8 | Purple, with a cyan shield that breaks first |
| Commander | Every 10th wave | Gold, caped, slow, and worth a pile of points |

Kills add to both **score** and **points**. Score is the number on the leaderboard and never goes down. Points are the pile you spend on upgrades. Clearing a wave adds `50 × wave` to both, and some gems.

## Upgrades

Open Upgrades during a fight (the fight pauses) and spend points. The buttons are the operations.

- **Divide** splits the squad. You go from 1 trooper to 2, then onward. Everyone strafes together and fires. Costs 40, then 90, 160, 280, and 480 points. Cap is 5, or 6 with the troop multiplier.
- **Multiply** is two labeled choices. Damage is multiplied by 1.25 each rank. Fire rate is multiplied by 1.18 each rank. Each rank costs more than the last, starting at 50 points. Seven ranks each.
- **Subtract** spends points to put away the smaller gun. Pistol, then Rapid Rifle (80), Spread Shotgun (200), Heavy Cannon (420). Buying one also unlocks it for later runs.

The tank is a different body: slower, a splash cannon, and 3 armor pips that soak hits before you lose a life. A pip returns after 12 seconds if you are missing one. Deploy it from the same panel. It costs 900 points once you have reached wave 4. After it is unlocked, deploying it is free, and you can start the next run in it from the menu. Dismount returns you to the squad.

## Gems

Gems are the persistent currency. They are saved in this browser.

You earn them by playing:

- Grunt or runner: 1 gem
- Brute: 3
- Elite: 4
- Commander: 12
- Wave clear: 4 plus the wave number
- A daily reward of 20 gems, once per calendar day

The store sells shortcuts. None of them are required to finish the arsenal.

| Item | Gems | Also earned by playing |
| --- | --- | --- |
| Extra life | 25 | — (cap: 2 bought per run, and 5 lives held) |
| Rapid Rifle | 100 | Clear wave 2, or Subtract 80 points |
| Spread Shotgun | 175 | Clear wave 4, or Subtract 200 points |
| Heavy Cannon | 250 | Clear wave 7, or Subtract 420 points |
| Troop multiplier | 150 | Clear wave 6 (new runs start with 2 soldiers) |
| Tank | 500 | Clear wave 10, or spend 900 points from wave 4 |
| Desert Tan skin | 50 | Gem store only |
| Night Ops skin | 120 | Gem store only |
| Jungle skin | 180 | Gem store only |

Gem packs (100, 550, and 1200) are placeholder real-money products. In the browser they report `iap_unavailable`. See [MONETIZATION.md](MONETIZATION.md).

Skins recolor the squad and the tank: Field Olive is free, plus Desert Tan, Night Ops, and Jungle.

## Scores

Every run with a score above zero is saved in `localStorage` under the key `cool-new-game-v1`. The game-over screen shows your best score and this browser's top 5 runs (score, wave, and date). There is no online leaderboard.

**Share** uses the Web Share API with the message `I scored X in Cool New Game — can you beat it?`. If sharing is missing or fails, the game copies that text to the clipboard and says it was copied. If the clipboard is blocked too, the message is shown so you can copy it yourself.

Gems, unlocked weapons, the troop multiplier, the tank, the selected skin, and the mute setting are in the same save.

## For developers

`window.CoolNewGame` exposes the product catalog, `Commerce`, and `setIapAdapter()`. Combat and the store UI read unlock flags and the gem balance. They do not call a store SDK. Every grant goes through `Commerce.grantReward()`. The included adapter logs the product id and returns `iap_unavailable`, so a Capacitor, StoreKit, or Play Billing adapter can be swapped in later without rewriting the store. Details are in [MONETIZATION.md](MONETIZATION.md).
