# 5e Magic Item Rarity Score

**[Try the calculator](https://mugasofer.github.io/item-costs/)**

A formula reverse-engineered from ~480 official magic items (DMG, XGtE, TCoE) via linear regression. It produces a continuous score that maps to rarity tiers and can be interpolated into gold piece prices.

## The Formula

Start with a **base score of 0.51**, then add or subtract the following.

### Spell Level (the biggest factor)

If the item replicates or is based on a spell effect, add **0.396 per spell level**, minus **0.013 per spell level squared**.

In practice:

| Spell Level | Contribution |
|---|---|
| Cantrip (0) | +0.00 |
| 1st | +0.38 |
| 2nd | +0.74 |
| 3rd | +1.07 |
| 4th | +1.37 |
| 5th | +1.65 |
| 6th | +1.89 |
| 7th | +2.11 |
| 8th | +2.30 |
| 9th | +2.46 |

The curve flattens at higher levels -- the jump from 1st to 2nd matters more than 8th to 9th.

If the item doesn't replicate a spell at all, skip this step — its power will be captured by the other fields below.

**Healing items:** Assign a spell level based on the equivalent upcast Cure Wounds — match average healing output to the closest level. The designers priced healing items by raw throughput, not by spell identity.

| Average HP Restored | Cure Wounds Level |
|---|---|
| ~7-9 | 1st |
| ~11-14 | 2nd |
| ~16-18 | 3rd |
| ~20-23 | 4th |
| ~25-27 | 5th |
| ~29-32 | 6th |
| ~34-36 | 7th |
| ~38-41 | 8th |
| ~43-45 | 9th |

### Adjusting Spell Level for Modified Effects

If your item replicates a spell but changes how it works, adjust the spell level. These adjustments are calibrated against ~130 official items:

- **Removing concentration: +0.5 levels.** Less than the +1 some homebrew guides suggest. *Haste without concentration → spell level 3.5.*
- **Weaker/partial effect: -0.5 levels per limitation.** Self-only when the spell targets others, or only part of the spell's effect. *Only the resistance portion of Protection from Energy → spell level 2.5.*
- **Stronger effect: +1 level per upgrade.** More targets, longer duration, wider area, ignores resistance. *Fireball with larger area + ignores fire resistance → spell level 5.*
- **Bonus action casting: ~+1 level.** Small sample (6 items), but the trend is clear.

Use half-levels freely — the formula handles fractional spell levels fine.

### How the Item Is Used

The base score assumes the item replicates a spell via unlimited, at-will, or consumable usage. Adjust if the item differs from that baseline:

| Usage Type | Modifier | Examples |
|---|---|---|
| No spell-like usage | +0.52 | Plain +X weapons, ability score items, damage-only weapons |
| Unlimited / at-will / permanent spell | — | Cloak of Elvenkind, Broom of Flying |
| Consumable (destroyed after use) | — | Potions, scrolls, beads |
| Charges per day (recharges daily) | -0.16 | Wands, staves, most charge-based items |

For items with daily charges, add **+0.046 per charge per day**. For consumable items with multiple charges before breaking, add **+0.011 per charge**.

An item with 7 charges/day: -0.16 + 7 x 0.046 = +0.16 total (nearly cancelling out — 7/day is practically at-will for most adventuring days).

### Passive Features

Add modifiers for always-on features the item grants while worn or carried. "Passive" means no charges spent, no action required, no resources consumed. A Staff of Fire's fire resistance counts (always on, even though the staff also has charges for casting spells). A Gem of Seeing's truesight does not (requires spending charges).

**Damage resistance: +0.37 per damage type.** The item passively grants resistance to one or more damage types while worn/carried. Most items grant exactly one type; Armor of Invulnerability grants three (nonmagical B/P/S).

**Damage immunity: ~+0.44 per damage type.** The item passively grants immunity to a damage type. The immunity estimate is resistance (+0.37) plus a small additional premium (+0.07), but with only 5 immunity items in the dataset this figure is unreliable — treat it as "at least as valuable as resistance."

*Resistance examples: Armor of Resistance, Ring of Resistance, Dragon Scale Mail, Frost Brand, Staff of Fire.*
*Immunity examples: Efreeti Chain (fire), Periapt of Proof against Poison, Ring of Fire Elemental Command.*

**Saving throw advantage or bonus: +0.29.** The item grants advantage on certain saving throws, or a passive bonus to saves beyond what's captured by the +X bonus field.

*Examples: Mantle of Spell Resistance, Ring of Spell Turning, Robe of the Archmagi, Scarab of Protection, Spellguard Shield, Stone of Good Luck.*

**HP preservation: +0.33.** The item passively reduces incoming damage, regenerates hit points, or otherwise preserves HP without requiring charges or activation.

*Examples: Cloak of Displacement, Ring of Regeneration, Periapt of Wound Closure, Arrow-Catching Shield, Gloves of Missile Snaring.*

The defensive categories contribute in the +0.29–0.37 range per instance. With only 7–27 items each, the individual estimates are noisy; if you want a single number for "has any defensive passive", +0.33 is a safe approximation.

**Extra equipment slot: +1.19.** The item effectively gives you an extra equipment slot — it fights on its own, orbits your head, or otherwise acts independently without occupying your hands or an action. This is the single largest modifier in the formula.

*Examples: All Ioun Stones, Dancing Sword, Animated Shield, Scimitar of Speed, Tentacle Rod.*

**Special vision or light: +0.52.** The item grants enhanced vision (darkvision, truesight, see invisible) or sheds magical light as a passive property.

*Examples: Robe of Eyes, Goggles of Night, Crystal Ball, Eyes of the Eagle, Flame Tongue, Sun Blade, Sentinel Shield.*

**Utility presence: -0.63.** The item provides a passive convenience or utility effect — things like environmental immunity, stealth enhancement, or niche protections that are useful but not combat-defining. The *negative* coefficient means these items tend to be priced *lower* than you'd expect from their spell equivalents; designers seem to consider passive utility less impressive than active power.

*Examples: Weapon of Warning, Cloak of Elvenkind, Boots of the Winterlands, Periapt of Health, Ring of Mind Shielding, Necklace of Adaption.*

These categories can overlap with each other and with other modifiers. For example, Cloak of Displacement has both "HP preservation" (+0.33) and would score its main effect through spell level as normal.

**Note on attunement:** You can ignore whether an item requires attunement. Attunement is a consequence of what an item does, not an independent driver of rarity — when the formula's variables are present, attunement's coefficient drops to zero. (The passive features above, combined with spell level, charges, +X bonuses, and ability score effects, collectively account for the same item features that [trigger attunement](https://www.enworld.org/threads/reverse-engineering-the-real-rules-of-attunement.682660/).)

### +X Bonus (AC, Save DC, Spell Attack, or Weapon)

Add **+0.53 per plus**. If the item has several bonus types (e.g. a Rod of the Pact Keeper grants both spell save DC and spell attack), use whichever is highest -- they tend to come in matched sets.

| Bonus | Contribution |
|---|---|
| +1 | +0.53 |
| +2 | +1.05 |
| +3 | +1.58 |

### Ability Score Effects

- **Sets an ability score**: subtract 1.78, then add **+0.58 per point of modifier** the score grants.
- **Grants +2 to an ability score** (tomes/manuals): add **+0.65**.

| Set Score | Modifier | Net Contribution |
|---|---|---|
| 19 | +4 | +0.54 |
| 21 | +5 | +1.12 |
| 23 | +6 | +1.70 |
| 25 | +7 | +2.28 |
| 27 | +8 | +2.86 |
| 29 | +9 | +3.44 |

### Multiple Spell Effects

If the item has additional spell effects beyond the primary one, add **+0.131 per spell level of the highest secondary effect**. If there's both a second and third spell effect, use whichever has the higher spell level.

### Bonus Damage

If the item deals extra damage beyond what's captured by its spell level (bonus elemental damage, AoE bursts, triggered effects), add a modifier. Two options:

#### Option A: Headline Damage (better fit to official rarities)

Add **+0.032 per point of average headline damage**. This is the straightforward face-value average of the damage dice, summed across all of the item's damage effects, regardless of how often they trigger. This approach better predicts what WotC actually assigned (R² = 0.573), suggesting the designers priced items based on how impressive the damage *looks* rather than how it plays out in practice.

To calculate, add up the average damage from all the item's bonus damage sources:

- **Always-on damage** -- extra damage on every hit, no resource cost. Examples: Flame Tongue's 2d6 fire (avg 7), Dragon Slayer's 3d6 vs dragons (avg 10).
- **Charge/trigger-based damage** -- extra damage that costs a charge, triggers on a specific roll, or has a per-day limit. Examples: Staff of Striking's 3d6 per charge (avg 10), Ring of the Ram's 6d10 at 3 charges (avg 33), Sword of Sharpness's extra damage on a nat 20 (avg 14).
- **Consumable damage** -- one-time damage from an item that's destroyed. Examples: Bead of Force's 5d4 (avg 12), Arrow of Slaying's extra 6d10 (avg 33).

If an item has damage in multiple categories, add them together. For example, Sword of Sharpness has charge-based damage of 14 (the bonus slashing on a nat 20) and always-on damage of 6 (extra cutting damage), for a headline total of 20.

| Headline Damage | Contribution |
|---|---|
| 5 | +0.16 |
| 10 | +0.32 |
| 15 | +0.48 |
| 20 | +0.64 |
| 25 | +0.80 |
| 30 | +0.96 |

#### Option B: Estimated DPR (better for evaluating actual power)

Add **+0.033 per point of estimated damage per round**. This adjusts for how often the damage actually occurs in a typical combat round, making it a better measure of real combat impact -- but it fits the official rarities slightly less well (R² = 0.562), because WotC apparently didn't always do this math themselves.

To estimate DPR, adjust the headline damage by how often it triggers. Assume a typical martial character making 2 attacks/round with ~65% hit rate:

- **Always-on, every hit**: headline x 1.3 (= 2 attacks x 0.65 hit rate). *Flame Tongue 7 -> 9.1 DPR.*
- **Nat 20 only**: headline x 0.1 (= 2 attacks x 0.05 chance). *Vicious Weapon 7 -> 0.7 DPR.*
- **Situation-specific** (only vs certain creature types): always-on x 0.3. *Dragon Slayer 10 -> 3.9 DPR, Mace of Disruption 7 -> 2.7 DPR.*
- **1/day ability**: headline / 24 (amortized over ~24 combat rounds/day). *Dagger of Venom 11 -> ~0.15 DPR* (also needs a hit and failed save).
- **Charge-based, per hit**: estimate charges spent per day / 24 rounds. *Staff of Striking 10 avg per hit, ~6 uses/day -> ~2.5 DPR.*
- **Charge-based, uses action**: uses per day x headline / 24. *Ring of the Ram ~2 uses/day x 33 / 24 -> ~1.8 DPR.*
- **Consumable (one-time-ever)**: headline / 4 (amortized over one combat). *Arrow of Slaying 33 -> ~8.2 DPR.*
- **Stacking effects**: estimate average accumulated damage over a 4-round combat. *Sword of Wounding hits ~1.3x/round, bleeds stack: avg ~6.5 DPR.*

If an item has multiple damage sources, add their DPR estimates together.

| DPR | Contribution |
|---|---|
| 2 | +0.07 |
| 5 | +0.17 |
| 10 | +0.33 |
| 15 | +0.50 |
| 20 | +0.66 |
| 30 | +1.00 |

#### Which should I use?

- **Headline damage** to match WotC's conventions and the official "feel."
- **DPR** to price homebrew by actual combat power — penalizes flashy-but-rare triggers, rewards reliable damage.

The two mostly agree. They diverge for items like Ring of the Ram (headline 33, DPR 1.8) or Tentacle Rod (headline 10, DPR 19.5).

### Has Extra Features

Add **+0.40** if the item has notable additional capabilities beyond its main spell/bonus/damage effects. This includes:

- **Conditions**: the item inflicts a condition like Poisoned, Stunned, Frightened, Restrained, etc.
- **Miscellaneous extras**: other features that don't fit into the categories above -- for example, the Cloak of the Bat grants flight and polymorph on top of its main stealth effect, or the Helm of Brilliance has a huge stockpile of extra spell uses in its jewels.

Use your judgement: if you'd describe the item as "it does X *and also* Y", the "also Y" part probably qualifies. A Rod of the Pact Keeper has +X to spell attacks and save DC, recovers a spell slot, *and* acts as a spellcasting focus -- that last part is an extra. A simple +1 Longsword that just hits harder doesn't qualify.

## Interpreting the Score

### Rarity Tiers

Round the score to the nearest integer:

| Score | Rarity |
|---|---|
| 0 | Common |
| 1 | Uncommon |
| 2 | Rare |
| 3 | Very Rare |
| 4 | Legendary |

### Model Accuracy

- **49% exact match** to official rarity
- **95% within one tier**
- R-squared = 0.57 (the formula explains over half the variance; the rest is designer judgement, flavour, and "vibes")

### Known Weaknesses

The formula tends to underpredict rarity for:
- "Iconic" items given Legendary status partly for flavour (Deck of Many Things, Ring of Invisibility, Sphere of Annihilation)
- Items whose power is hard to quantify as a spell level (Apparatus of Kwalish)

It tends to overpredict rarity for:
- Items with high spell levels but severe limitations (Weapon of Warning has a Foresight-equivalent effect but is Uncommon)
- Low-level consumables with no charges that the designers priced as Common despite having 1st-2nd level effects (Bead of Nourishment, Veteran's Cane)

## Estimating Price

The rarity score can be interpolated into a gold piece price using the Xanathar's Guide base prices as anchors:

| Rarity | Score | Base Price |
|---|---|---|
| Common | 0 | 100 gp |
| Uncommon | 1 | 400 gp |
| Rare | 2 | 4,000 gp |
| Very Rare | 3 | 40,000 gp |
| Legendary | 4 | 200,000 gp |

For fractional scores, interpolate on a log scale between the two adjacent tiers. For example, a score of 2.35 is 35% of the way from Rare to Very Rare: price = 4,000 × (40,000/4,000)^0.35 ≈ 8,955 gp.

Or just use this table:

| Score | Price | Score | Price |
|---|---|---|---|
| 0.00 | 100 | 2.25 | 7,100 |
| 0.25 | 140 | 2.50 | 12,600 |
| 0.50 | 200 | 2.75 | 22,500 |
| 0.75 | 280 | 3.00 | 40,000 |
| 1.00 | 400 | 3.25 | 59,800 |
| 1.25 | 710 | 3.50 | 89,400 |
| 1.50 | 1,260 | 3.75 | 133,700 |
| 1.75 | 2,250 | 4.00 | 200,000 |
| 2.00 | 4,000 | | |

### Consumable Items

Consumable items (potions, single-use items) are priced at roughly **1/6 of the full price**. This is much steeper than Xanathar's suggested 1/2 discount, but fits the actual published potion prices far better — a one-shot item that's gone after a single use should cost a fraction of a permanent one, not half.

The 1/6 multiplier is calibrated against the Potion of Healing crafting costs in Xanathar's (p130), which it matches well for Greater through Supreme potions. The basic Potion of Healing (25 gp) is priced below even this curve — it's a commodity item, intentionally cheap enough that adventurers stock up like buying bandages.

### Spell Scrolls

Spell scroll prices follow their own progression (Xanathar's p133) that doesn't map cleanly onto the general rarity curve. Low-level scrolls (cantrip, 1st) are priced as trivial goods, while high-level scrolls (8th, 9th) cost more than permanent items of equivalent rarity. This likely reflects the unique value of scrolls to wizards, who can permanently learn spells by copying from them — a scroll of a 9th-level spell isn't just a one-shot casting, it's permanent access to that spell. Use the published scroll cost table directly rather than deriving from rarity score.

## Examples

**Wand of Fireballs** (3rd-level spell, 7 charges/day):
- Base: 0.51
- Spell level 3: +1.07
- Charges/day: -0.16
- 7 charges: +0.32
- **Total: 1.74 (Rare)** -- matches official rarity
- Price: ~2,200 gp (between Uncommon 400 and Rare 4,000)

**+2 Shield** (no spell, +2 AC):
- Base: 0.51
- No spell usage: +0.52
- +2 bonus: +1.05
- **Total: 2.08 (Rare)** -- matches official rarity
- Price: ~4,800 gp

**Flame Tongue** (cantrip-level spell for light, unlimited, 7 avg permanent bonus damage, special vision/light):
- Base: 0.51
- Spell level 0: +0.00
- Bonus damage (7): +0.22
- Special vision/light: +0.52
- **Total: 1.26 (Uncommon)** -- officially Rare (model slightly underpredicts)
- Price: ~720 gp

**Belt of Fire Giant Strength** (sets STR to 25):
- Base: 0.51
- No spell usage: +0.52
- Sets score (mod +7): -1.78 + 7 x 0.58 = +2.28
- **Total: 3.31 (Very Rare)** -- matches official rarity
- Price: ~66,000 gp

**Potion of Greater Healing** (2nd-level Cure Wounds equivalent, consumable):
- Base: 0.51
- Spell level 2: +0.74
- **Total: 1.25 (Uncommon)** -- matches official rarity
- Price: ~120 gp (consumable: 710 × 1/6; Xanathar's crafting cost: 100 gp)

**Holy Avenger** (+3 weapon, 4th-level spell, unlimited, 11 avg permanent bonus damage, misc extra features, saving throw aura):
- Base: 0.51
- Spell level 4: +1.37
- +3 bonus: +1.58
- Bonus damage (11): +0.35
- Misc features: +0.40
- Save advantage: +0.29
- **Total: 4.50 (Legendary)** -- matches official rarity
- Price: ~200,000+ gp (capped at Legendary)

**Ring of Invisibility** (3rd-level spell, unlimited):
- Base: 0.51
- Spell level 3: +1.07
- **Total: 1.58 (Rare)** -- officially Legendary (iconic prestige bump)
- Price: ~1,500 gp by formula; actual value much higher given Legendary status

**Mantle of Spell Resistance** (save advantage, no spell effect):
- Base: 0.51
- No spell usage: +0.52
- Save advantage: +0.29
- **Total: 1.33 (Uncommon)** -- officially Rare (the "advantage on saves vs spells" is hard to capture as a spell level)
- Price: ~850 gp

## Thanks

Thanks to varansl of Dump Stat for their systematic analysis of magic item power and value, the dataset of which formed the bedrock of this analysis: https://www.reddit.com/r/dndnext/comments/1j8t5fi/magic_items_pricing_guide_an_adjustable/

Thanks to Sword of Spirit on enworld for their analysis of Attunement: https://www.enworld.org/threads/reverse-engineering-the-real-rules-of-attunement.682660/