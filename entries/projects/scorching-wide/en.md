---
title: "Monster Hunter Wilds – Scorching Wide"
type: project
layout: case-study
lang: en
slug: scorching-wide
permalink: /entries/scorching-wide/
date: 2026-10-02
year: 2026
show_cover: false
label: "ROLE"
role: "Gameplay Systems Design, Runtime Investigation, Testing & Balance Analysis"
technologies: [Lua, REFramework]
code: "https://github.com/Gwaegs/Elemental-Long-and-Scorching-Wide"
demo: ""
paper: ""
excerpt: "A gameplay rebalance for the Wide-type Gunlance in Monster Hunter Wilds that reworks the Heat Gauge and adds Scorching Blade to create a distinct shelling-to-melee playstyle."
---

## Overview

A gameplay rebalance for the Wide-type Gunlance in *Monster Hunter Wilds* that adds a gauge that rewards use of the weapon’s unique shelling attacks. The project reworks the “Heat Gauge” introduced in *Monster Hunter Generations* to give Wide-type Gunlances a distinct gameplay identity while expanding build options, similar to my Elemental Long-type project.

## Goal

Gunlance has three shelling types—Normal, Long, and Wide—that modify the behavior of its defining shelling attacks. In *Monster Hunter Wilds*, however, I found these types didn't create three equally distinct playstyles. In my Elemental Long project, I successfully differentiated Long from Normal and Wide; this project aims to differentiate Wide from Normal beyond the minimal differences it currently has.

As with Long, I wanted Wide to offer a reason to build and play Gunlance differently. With the experience the Long project gave me, I wanted to take on a more challenging implementation.

Reworking the “Heat Gauge” from *Monster Hunter Generations*—a design players disliked because of its implementation—gave me the challenge and avenue for differentiation I sought. Normal was the type to use shelling attacks reliably and relied on the fewest external factors; Long had become the specialist with high reward for high precision and game knowledge; the “Heat Gauge” would make Wide the Gunlance of choice when shelling attacks were at their weakest. By making Wide shelling improve non-shelling attacks, I could add a third distinct identity to match the three distinct shelling types.

## My Contribution

I designed the behavior, damage model, and interactions with *Monster Hunter Wilds* systems that the reworked “Heat Gauge” would have. I also conceptualized the “Scorching Blade” mechanic—which builds on the “Heat Gauge” idea—balanced its damage model, and designed its unique implementation.

Drawing on the experience from the Elemental Long project, I edited in-game files, used diagnostic scripts, and tested throughout development.

I once again oversaw the runtime hooks and Lua scripts, and spent extensive time numerically analyzing the weapons' damage performance to align it with my goals.

## Relevant Files

- **Edited Asset:** `natives/STM/GameDesign/Player/ActionData/Wp07/GlobalParam/Wp07GlobalActionParam.user.3`
- **Heat Blade Wide Lua scripts:** `gl_override`, `gog_manager`, `v1_heat_blade`
- **Installation Guide:** `README.txt`

## Process

### Step 1: Implementing the Heat Gauge

My experience on the Elemental Long project helped greatly in replicating the “Heat Gauge.”

After confirming the scripts I had used to identify Long-type Gunlances and attacks could be repurposed to identify Wide-type, I had a method to assign Heat buildup and decay to shelling attacks exclusive to Wide.

With some research, I visualized the “Heat Gauge” meter on-screen and added ticks for current Heat, time remaining in locked Heat, and reserve Heat. These ticks would become relevant when it was time to implement “Scorching Blade.”

A variation of the script used to dump subroutines and identify shelling attacks could identify non-shelling attacks, and a hook injected base damage increases into qualifying attacks—an early deviation from the established “Heat Gauge,” which provided a weaker Attack stat boost.

### Step 2: Expanding Upon Heat Gauge with Scorching Blade

While the improvements to Wide’s non-shelling attacks provided a noticeable power boost, my numerical analysis found it wasn’t enough to distinguish itself from Normal-type gameplay, nor did it create unique builds.

I drafted several potential mechanics that could push Wide in a more distinct direction and chose the “Scorching Blade” draft. The “Scorching Blade” draft used the “Heat Gauge” level to decide on a secondary damage instance that would be applied to all non-shelling attacks: this would distinguish itself from Normal’s slow, shelling-heavy attacks by making Wide favor fast, non-shelling attacks that activated the “Scorching Blade” damage instance more often.

After dumping the subroutines that ran during attacks once more, I found that the code used to handle damage creation wasn't exposed by current mod tools, forcing me to take a different approach. This brought me to the titular “Scorcher” skill, which has a chance to apply fixed damage to attacks when the skill is activated.

I repeated my runtime logging with “Scorcher” equipped and identified subroutines with variables that controlled its activation chance and damage. The resulting Lua implementation intercepted these subroutines, manipulated the damage they dealt, and set the activation chance to 100%.

This made “Scorching Blade” reliant on “Scorcher” being equipped by the player, which I couldn't guarantee. To fix this, I exposed the player’s equipped armor data and matched skills to their internal ID until I had the ID for “Scorcher.” The final script checked player armor when they equipped a Wide Gunlance and either averaged “Scorcher” damage into “Scorching Blade” when the skill was equipped, or injected the “Scorcher” skill ID into an armor-piece array of granted skills. These injected “Scorcher” IDs were marked as synthetic so the implementation could better distinguish whether or not “Scorcher” was naturally equipped if the player changed builds later.

My testing ensured “Scorching Blade” damage matched injected values, synthetic “Scorcher” pieces weren't counted as natural ones, and “Scorcher” detection worked correctly across the many ways a player can equip weapons and armor.

### Step 3: Support for Wide as a Unique Build Option

The final stage was balancing “Scorching Blade” so its damage profile felt distinct from Normal and Long, and strengthening its connection to established Gunlance and “Heat Gauge” identity.

On identity, I adjusted the “Heat Gauge” values so the player felt sufficiently rewarded for using shelling attacks to empower non-shelling attacks, rather than feeling obliged to. “Scorching Blade” was tied to “Heat Gauge” by making the damage of “Scorching Blade” dependent on Heat when activated; hence the gauge informed players of how strong “Scorching Blade” would be when activated, as well as how much time the buff had remaining. “Scorching Blade” damage was given properties shared with the Gunlance’s defining shelling attacks: fixed fire damage and body-part-independent damage scaling.

For the damage profile, I ran a similar numerical analysis to the one used in my Elemental Long project to compare Wide's damage per second with the other shelling types, and found Wide still fell behind. To remedy this, I adjusted “Scorching Blade” damage using a mathematical formula that factored in the player’s critical-hit-chance stat and the player's critical-damage-boost skills, both of which were exposed in Step 2’s process. This distinguished itself from Normal and Long by providing a higher damage boost from critical-hit investment, which the former two received less damage-per-second improvement from because shelling attacks couldn’t crit and their non-shelling attacks lacked a “Heat Gauge” to boost damage further.

Wide now had a unique identity and build priority. Normal-type remained the consistent type that ignored crits and all body-part weaknesses/resistances; Long dealt elemental damage that couldn’t crit but rewarded elemental matchup knowledge and hitting element-weak body parts. Wide emphasized non-shelling attacks that rewarded fast attacks, Heat management, critical-hit investment, and hitting physical-weak body parts.

### What I Gained From This Experience

This project expanded on the runtime-investigation techniques I learned while developing Elemental Long, but required applying them to a more complex, interconnected system. I gained experience coordinating mechanics across different objects, working around inaccessible features, testing stateful systems, and balancing a more complex damage profile.
