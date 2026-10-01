---
title: "Monster Hunter Wilds – Elemental Shelling"
type: project
layout: case-study
lang: en
slug: elemental-shelling
permalink: /entries/elemental-shelling/
date: 2026-09-30
year: 2026
show_cover: false
label: "ROLE"
role: "Gameplay Systems Design, Runtime Investigation, Testing & Balance Analysis"
technologies: [Lua, REFramework]
code: ""
demo: ""
paper: ""
excerpt: "A gameplay rebalance for the Long-type Gunlance in Monster Hunter Wilds that adds weapon-element scaling to Gunlance's unique shelling attacks."
---

## Overview

A gameplay rebalance for the Long-type Gunlance in *Monster Hunter Wilds* that adds weapon-element scaling to Gunlance's unique shelling attacks. The project uses elemental shelling to give Long a more distinct gameplay identity while opening build options that the weapon's standard shelling system does not normally support.

## Goal

Gunlance has three shelling types—Normal, Long, and Wide—that modify the behavior of its defining shelling attacks. In *Monster Hunter Wilds*, however, I found these types didn't create three equally distinct playstyles. Normal and Wide supported recognizable builds, while Long lacked a comparably strong niche.

I wanted Long to offer a reason to build and play Gunlance differently rather than functioning primarily as another variation of the same shelling system.

Elemental shelling provided that avenue. Standard shelling does not inherit the equipped weapon's elemental damage, which limits how much Gunlance can engage with elemental weapons, skills, and matchup-specific builds. By making Long shelling scale with weapon element, I could address both problems at once: give Long a unique identity and introduce a new genre of Gunlance builds.

## My Contribution

I was the sole contributor, designing the intended elemental-shelling behavior, damage model, Long-specific identity, and interactions with existing game systems.

I investigated the game's weapon data and runtime behavior using unpacked game files, diagnostic scripts, runtime logging, and testing. This involved identifying the parameters responsible for shelling damage, determining where weapon element could be introduced into the damage calculation, locating the data used to distinguish shelling types, and testing how those systems behaved over different Gunlances and equipment contexts.

I then oversaw the implementation through runtime hooks and Lua scripts, and modifications to load-time weapon data. Additionally, I evaluated in-game behavior, diagnosed results, and iterated until the implementation aligned with intended game behavior and the weapon-performance metrics I targeted through numerical analysis.

### Step 1: Enabling Elemental Shelling

Using tools created by other mod developers, I unpacked the game's weapon data and found the values tied to Gunlance shelling attacks. No simple flag enabled weapon-element damage. Instead, shells contained a fixed amount of fire damage that didn't scale with the weapon's elemental value or element type.

That meant I needed to look beyond the weapon-data files and investigate the game's damage calculation itself. I used a diagnostic script to log player- and enemy-related subroutines that ran during the short window when a shell struck a monster. This identified a damage-processing subroutine whose argument contained the shell's attack parameters.

Among those parameters were two values that controlled whether weapon-element damage should be applied and how much of the weapon's elemental value should be used in the calculation. Both were set to zero for shelling attacks.

By hooking that subroutine at runtime, I could modify those parameters before the game completed its damage calculation, enabling elemental damage and supplying an elemental multiplier of my choosing.

Step 1 was complete: shelling could now inherit the equipped weapon's actual element.

### Step 2: Restricting Elemental Shelling to Long

The next requirement was making the mechanic exclusive to Long-type Gunlances.

I used a similar runtime-investigation process, this time logging subroutines that executed when a player equipped a weapon. The weapon objects passed through these routines contained a ShellingType enum, which let me identify the value corresponding to Long shelling.

I then added a condition to the elemental-shelling logic so that the damage modification occurred only when the equipped weapon was a Gunlance using the Long shelling type.

There was a complication unique to *Monster Hunter Wilds* however: Element-focused crafted Gunlances normally received Wide shelling, while I wanted them to become a natural source of Long elemental weapons. A second modification therefore detects the appropriate crafted Gunlances and substitutes their Wide shelling identifier with Long before the elemental-shelling check occurs.

I tested the behavior across multiple Gunlance and equip methods, making small adjustments to ensure both the shelling-type override and elemental-damage calculation remained consistent.

Step 2 was complete: elemental shelling was now restricted to all Long Gunlances.

### Step 3: Support for Long as a Unique Build Option

The final stage was to balance Long’s weapon performance so it could distinguish itself from Normal and Wide, without overshadowing them to the point of invalidating them.

This required my knowledge of Gunlance’s behavior with skills and damage output, testing how the current implementation of Long behaved with them, analyzing Long’s current damage output, and calculating a distinct but competitive niche.

Elemental Shelling gave Long a disadvantage and an advantage. Unlike Normal and Wide, its performance now depended on hitting element-weak body parts in a favorable matchup; in exchange, its damage increased with elemental damage boosts. I leaned into these differences so Long could stand out as the higher-effort, higher-precision shelling type: it would require appropriate weapons, different skills, and correctly targeting element-weak zones, but rewarded those conditions with the highest damage.

To start, I skewed Long toward elemental damage using a Lua script—similar to the one I used in Step 1—to intercept damage calculations, identify damage boosts that applied only to a shell’s physical Attack portion, and neutralize those boosts. I then converted the lost physical Attack boost into an Element attack boost, with a rate I tuned later.

I then calculated the damage per second of Normal, Long, and Wide across several attacks and body parts a player would expect to attack in-game. This allowed me to identify the Elemental values Long needed to deal both below-average damage against body parts resistant to the element and above-average damage against parts weak to it.

I modified the base Element damage of Long’s shells and the damage boost it got from the Attack Boost-to-Element Boost conversion script to hit those benchmarks, and extensively tested to ensure the script correctly manipulated skills and let Long hit target damage-per-second.

The final Step was complete: Long was an Elemental skewed weapon that required more precision, more investment, and a new build, but offered higher potential damage in favorable environments.

### What I Gained From This Experience

This project gave me an understanding of Lua, runtime hooking, and the strategies behind modifying games. It also helped me understand how large systems organize and pass data between objects, and gave me experience using numerical analysis to translate a game design goal into measurable balance targets.
