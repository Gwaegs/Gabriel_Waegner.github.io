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
excerpt: "A gameplay rebalance for Long-type Gunlance that adds weapon-element scaling to shelling attacks, creating a distinct elemental build with matchup-dependent strengths and weaknesses."
---

## Overview

A gameplay rebalance for the Long-type Gunlance in *Monster Hunter Wilds* that adds weapon-element scaling to Gunlance's unique shelling attacks. The project uses elemental shelling to give Long a more distinct gameplay identity while opening build options that the weapon's standard shelling system does not normally support.

## Goal

Gunlance has three shelling types—Normal, Long, and Wide—that modify the behavior of its defining shelling attacks. In *Monster Hunter Wilds*, however, I found these types did not create three equally distinct playstyles. Normal and Wide supported recognizable builds, while Long lacked a comparably strong niche.

I wanted Long to offer a reason to build and play Gunlance differently rather than functioning primarily as another variation of the same shelling system.

Elemental shelling provided that avenue. Standard shelling does not inherit the equipped weapon's elemental damage, which limits how much Gunlance can engage with elemental weapons, skills, and matchup-specific builds. By making Long shelling scale with weapon element, I could address both problems at once: give Long a unique identity and introduce a new category of Gunlance builds.

## My Contribution

I designed the intended elemental-shelling behavior, damage model, Long-specific identity, and interactions with existing game systems.

I investigated the game's weapon data and runtime behavior using unpacked game files, diagnostic scripts, runtime logging, and testing. This involved identifying the parameters responsible for shelling damage, determining where weapon element could be introduced into the damage calculation, locating the data used to distinguish shelling types, and testing how those systems behaved across different Gunlances and equipment contexts.

I then oversaw the implementation through runtime hooks, Lua scripts, and modifications to load-time weapon data. I evaluated in-game behavior, diagnosed incorrect or inconsistent results, and iterated until the implementation aligned with the intended behavior and the weapon-performance targets I established through numerical analysis.

## Process

### Step 1: Enabling Elemental Shelling

Using tools created by other mod developers, I unpacked the game's weapon data and found the values tied to Gunlance shelling attacks. No simple flag enabled weapon-element damage. Instead, shells contained a fixed amount of fire damage that did not scale with the weapon's elemental value or element type.

That meant I needed to look beyond the weapon-data files and investigate the game's damage calculation itself. I used a diagnostic script to log player- and enemy-related subroutines that ran during the short window when a shell struck a monster. This identified a damage-processing subroutine whose argument contained the shell's attack parameters.

Among those parameters were two values that controlled whether weapon-element damage should be applied and how much of the weapon's elemental value should be used in the calculation. Both were set to zero for shelling attacks.

By hooking that subroutine at runtime, I could modify those parameters before the game completed its damage calculation, enabling elemental damage and supplying an elemental multiplier of my choosing.

**Step 1 was complete:** shelling could now inherit the equipped weapon's actual element.

### Step 2: Restricting Elemental Shelling to Long

The next requirement was making the mechanic exclusive to Long-type Gunlances.

I used a similar runtime-investigation process, this time logging subroutines that executed when a player equipped a weapon. The weapon objects passed through these routines contained a `ShellingType` enum, which let me identify the value corresponding to Long shelling.

I then added a condition to the elemental-shelling logic so that the damage modification occurred only when the equipped weapon was a Gunlance using the Long shelling type.

*Monster Hunter Wilds* introduced an additional complication through its weapon-crafting system. Element-focused crafted Gunlances normally received Wide shelling, while I wanted them to become a natural source of Long elemental weapons. A second modification therefore detects the appropriate crafted Gunlances and substitutes their Wide shelling identifier with Long before the elemental-shelling check occurs.

I tested the behavior across multiple Gunlances and equipment methods, making small adjustments to ensure both the shelling-type override and elemental-damage calculation remained consistent.

**Step 2 was complete:** elemental shelling was now restricted to Long Gunlances.

### Step 3: Supporting Long as a Unique Build Option

The final stage was balancing Long's weapon performance so it could distinguish itself from Normal and Wide without overshadowing them to the point of invalidating them.

This required applying my knowledge of Gunlance's skill interactions and damage output, testing how the new implementation behaved with those systems, analyzing Long's current performance, and calculating values that would establish a distinct but competitive niche.

Elemental shelling gave Long both a disadvantage and an advantage. Unlike Normal and Wide, its performance now depended on hitting element-weak body parts in favorable matchups; in exchange, its damage increased with elemental damage boosts. I leaned into these differences so Long could stand out as the higher-effort, higher-precision shelling type: it would require appropriate weapons, different skill investment, and accurate targeting of element-weak zones, but reward those conditions with greater damage potential.

To reinforce that identity, I skewed Long toward elemental damage using a Lua script similar to the one used in Step 1. The script intercepts damage calculations, identifies damage boosts that apply only to a shell's physical Attack portion, and neutralizes those boosts for Long. I then converted the lost physical Attack scaling into additional Element scaling at a rate I could tune independently.

I calculated the damage per second of Normal, Long, and Wide across several representative attacks and monster body parts that a player would reasonably target in-game. This let me identify the elemental values Long needed to deal below-average damage against element-resistant zones while outperforming the other shelling types against sufficiently element-weak targets.

I adjusted Long's base elemental multiplier and the conversion rate between physical Attack bonuses and elemental scaling until the calculated values reached those benchmarks. I then tested the implementation in-game to verify that the relevant skills were being converted correctly and that actual damage remained consistent with the target damage-per-second values.

**Step 3 was complete:** Long had become an elementally skewed shelling type that required more precision, more specialized investment, and a new build, while offering greater damage potential in favorable matchups.

## What I Gained From This Experience

This project gave me practical experience with Lua, runtime hooking, and the broader process of modifying an unfamiliar game system. It also improved my understanding of how large software systems organize and pass data between objects, and gave me experience using numerical analysis to translate a gameplay-design goal into measurable implementation and balance targets.
