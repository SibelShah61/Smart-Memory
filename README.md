# Smart Memory

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](https://github.com/senjinthedragon/Smart-Memory/blob/main/LICENSE)
[![Author: Senjin the Dragon](https://img.shields.io/badge/Author-Senjin_the_Dragon-gold.svg)](https://github.com/senjinthedragon)

Give your AI a memory that lasts. Smart Memory is a SillyTavern extension that quietly works in the background, keeping your AI oriented in long stories, aware of what happened this session, and grounded in facts it has learned across every previous chat with a character - even after weeks away.

It runs automatically. You don't have to do anything special. Just chat, and it takes care of the rest.

Both 1:1 chats and group chats are supported. In group chats, each character gets their own independent memory store and receives their own memories before they respond. A character selector in the settings panel lets you switch between group members to view and manage each character's memories, profiles, and entity registry independently.

_This is an independent extension for SillyTavern and is not affiliated with the SillyTavern development team._

<table style="border: none;">
  <tr>
    <td align="center" style="border: none;">
      <img src="https://raw.githubusercontent.com/senjinthedragon/Smart-Memory/dev/assets/smart-memory1.webp" alt="Smart Memory extension panel" /><br/>
      <em>The extension panel</em>
    </td>
    <td align="center" valign="top" style="border: none;">
      <img src="https://raw.githubusercontent.com/senjinthedragon/Smart-Memory/dev/assets/smart-memory2.webp" alt="Session memory list" /><br/>
      <em>Session memories</em>
    </td>
  </tr>
  <tr>
    <td colspan="2" align="center" style="border: none;">
      <img src="https://raw.githubusercontent.com/senjinthedragon/Smart-Memory/dev/assets/smart-memory3.webp" alt="Entity and memory force graph" /><br/>
      <em>Entity and memory graph - hover any node to inspect it</em>
    </td>
  </tr>
</table>

## ☕ Support the Developer

I'm a solo developer building tools to make AI roleplay better for everyone. Smart Memory is maintained in my free time, and as I'm currently navigating some financial challenges, any support means a lot.

If this extension adds something to your stories, please consider:

- **[Sponsoring me on GitHub](https://github.com/sponsors/senjinthedragon)**
- **[Buying me a coffee on Ko-fi](https://ko-fi.com/senjinthedragon)**
- **Bitcoin:** `bc1qjsaqw6rjcmhv6ywv2a97wfd4zxnae3ncrn8mf9`
- **Starring this repository** to help others find it.

## Installation

- In SillyTavern, open the **Extensions** menu (the stack of cubes icon)
- Click **Install extension**
- Paste: `https://github.com/senjinthedragon/Smart-Memory`
- Click **Install just for me** (or for all users if you're on a shared server)

Restart SillyTavern. **Smart Memory** will appear in your Extensions panel.

## Quick Start

After installing, two things to set up before your first session:

### 1. Pick a memory LLM

Smart Memory needs a language model to do its work - summarizing, extracting facts, generating recaps. It runs this separately from your main roleplay model so the two don't compete. Open the extension panel, scroll down to **Configuration**, and set the **Memory LLM** to whichever source fits your setup. See [Recommended Local Models](#recommended-local-models) for suggestions on what to run locally.

### 2. Set up semantic embeddings (optional but recommended)

Embeddings let Smart Memory compare memories by meaning rather than exact wording, which catches far more duplicates and keeps your memory lists clean. If you already use SillyTavern's Vector Storage with Ollama, you likely have `nomic-embed-text` installed and nothing extra is needed - just make sure **Use semantic embeddings** is enabled in the **Configuration** section. If not, see [Memory Deduplication](#memory-deduplication) for setup instructions.

### 3. Leave the memory context budget alone (for now)

The default **Memory context budget** in simple mode is 3750 tokens - the combined total of all memory tiers at their default sizes. This number was chosen to work well with the recommended local models. If you reduce it, one or more tiers will start getting trimmed or cut off entirely to fit the lower limit, which may noticeably affect quality. Leave it at the default until you have a feel for how Smart Memory works in your setup, then adjust carefully in small steps if needed.

That's it - Smart Memory will start building memories automatically from your next chat.

## What It Does

Smart Memory runs several memory systems in the background, each focused on a different slice of your story's history.

### Token Usage Display

A small bar in the settings panel shows how many tokens each memory tier is currently using in the AI's context. It updates after every response so you can see at a glance whether Smart Memory is taking up a sensible amount of your context.

When a tier is actively trimming content to fit its budget, its segment on the bar shows a red strip along the bottom edge. Hovering over the segment shows how many tokens were dropped. This is your signal to consider raising that tier's injection budget in the settings - or lowering another tier's budget to free up room.

**Auto-tune budgets** adjusts tier budgets automatically based on observed demand. After each injection pass it measures how many tokens each tier actually needed, then sets the budget to that amount plus 15% headroom - enough to stay comfortable without wasting context on unused space. Budgets are never reduced below their defaults, only grown. In group chats it tracks the highest demand seen across all characters in the session, so budgets are sized for the most memory-heavy character rather than whichever one injected last. Enabling auto-tune applies immediately using whatever data has already been collected; after that it re-tunes after every injection pass.

### Long-term Memory - Persistent Facts

Facts, relationship history, preferences, and significant events are extracted from your chats and saved for each character. These memories survive across all sessions - when you open a new chat with a character, everything they have learned is already there waiting.

Over time, memories are automatically consolidated so the same information does not pile up in slightly different forms. Smart Memory is good at recognising when two differently-worded entries are saying the same thing, so you end up with a clean, rich picture of the character rather than a growing cluttered list.

When a new memory describes a change - "Alex no longer distrusts Finn", "she moved to the capital", "the guild was disbanded" - the old fact is automatically retired and replaced rather than left alongside the newer truth as a contradiction.

Each memory is assigned a set of **activation triggers** - keywords derived automatically from its content. When a trigger appears in the current chat turn, that memory gets a scoring boost so it rises to the top of what gets injected into context. Memories that fire a trigger are also placed in a secondary slot closer to the prompt, so the AI sees them right before it responds. You do not need to configure anything for this - it runs automatically.

### Session Memory - Within-Chat Details

Granular details from the current session: scene descriptions, things that were revealed, how the relationship shifted, specific objects or places that came up. More detailed than long-term memory, and scoped to this chat only - it does not carry forward to future sessions, but it keeps the AI sharp on the specifics of what is happening right now.

Session memory is aware of what is already in long-term memory, so the two complement each other rather than duplicating information.

### Short-term Memory - Context Summary

When your conversation grows long enough that older messages start falling out of the AI's awareness, Smart Memory automatically writes a summary of everything so far and keeps it in context. The AI stays oriented in long stories even as older messages scroll away.

After the first summary is written, only new messages get folded in - the summary grows with your story rather than being rewritten from scratch every time. It also knows what is already captured in your long-term and session memories, so it focuses on narrative flow rather than repeating facts stored elsewhere.

The rolling summary sits alongside canon if you have it enabled - both can be active at the same time, covering different spans of your story's history.

### Scene Detection and History

Smart Memory watches for scene transitions - time skips, location changes, those little `---` dividers between scenes, and intimate partner changes. When one is detected, a short summary of the completed scene is saved. The last few scene summaries are kept in context so the AI always knows where the story has been, not just where it is.

### Story Arcs - Open Threads

Unresolved narrative threads - promises made, character goals, mysteries introduced, tensions left hanging - are tracked and kept in context. When the story resolves one, it gets marked closed and a short narrative summary is generated for the record. This keeps the AI oriented toward where the story is going, not just reacting to the last message.

Story arcs normally start fresh with each new chat. If you are running a continuing story across multiple chats (new chats as chapters rather than fresh starts), you can **pin an arc** with the thumbtack button next to it. Pinned arcs are stored at the character level and appear automatically in every new chat with that character. Unpinning returns it to chat-local scope.

Group chats support pinned arcs too. Pinned arcs in a group are stored against the group itself rather than any individual character, and appear automatically whenever that group starts a new chat.

When a pinned arc gets resolved, it moves to a separate **Resolved Threads** section below the active list rather than being deleted. Resolved arcs are not injected into context but stay visible as a record of where the story has been. A **re-open** button reactivates the thread if the story revisits it; a **remove** button discards it entirely. The resolved state carries into future chats so closed threads arrive already marked as resolved. The Resolved Threads section is collapsed by default and hidden when empty.

### Canon

Once you have at least one resolved arc summary, you can generate a **canon document** - a stable prose narrative synthesized from those arc summaries and high-importance long-term facts. Think of it as a "story bible" for the character: not a list of bullet points, but a composed history written by the model from everything it has learned.

Canon gets its own dedicated slot, separate from the rolling short-term summary. Both can be active at the same time: the summary covers recent events, canon covers the broader history. Canon is stored at the character level and carries forward to new chats with the same character. It is cleared by **Fresh Start** and by the **Clear** button in the Long-term Memory section.

### Character and World Profiles

After each extraction pass, Smart Memory generates compact state snapshots from stored memories and adds them to context alongside the other tiers:

- **Character state** - current emotional state, active goals, relationship posture
- **World state** - active threads in the setting, recent developments, current scene context
- **Relationship matrix** - one-line directional summary per named entity with a confidence score

Profiles are regenerated after each extraction pass and on chat load if stale. On Profile B, an optional message-count schedule can keep them fresh between extraction passes. A manual regenerate button is available in the settings panel.

In group chats, each character has their own independent profile. Switching the character selector updates the profiles panel to show that character's snapshot, and each character's profile is added to context when they are about to respond.

### Relationship History

Alongside individual memories, Smart Memory tracks the current emotional state of every named relationship as a set of descriptors - words like `warm`, `cautious`, `fragile`, `mistrustful`. Each descriptor carries its own magnitude (`low`, `medium`, `high`), displayed as `word(magnitude)` - e.g. `trusting(high), cautious(medium)`. After each extraction pass the model reviews the scene and emits any shifts; new descriptors are merged in and existing ones updated rather than replaced, so the state accumulates across sessions without losing earlier signals. A maximum of six descriptors per pair is kept; the lowest-magnitude entry is dropped when the cap is exceeded.

The **Relationship History** section in the settings panel shows all tracked pairs for the current character. You can add pairs manually (useful for seeding a relationship described in the character card), edit any pair to correct or refine the descriptors, and delete pairs that are no longer relevant. Relationship History is cleared alongside long-term memories when you use **Clear Memories** or **Fresh Start**.

Only pairs whose names appear in the recent message window are injected into context, so the block stays small even for characters with many relationships.

### Perspectives & Secrets

At each scene break, Smart Memory extracts a per-character knowledge map: what each named character knows, suspects, falsely believes, is unaware of, and is actively hiding from someone else. The responding character's knowledge block is then injected privately into their prompt, so the AI can maintain perspective-accurate behaviour - knowing what to reveal, what to deflect, and what they genuinely do not know.

The five tags map to distinct epistemic states:

- **Knows** - facts the character is confident about
- **Suspects** - things they believe but have not confirmed
- **Believes (false)** - things they are sure of that are actually wrong (dramatic irony)
- **Unaware** - things they do not know and have not guessed
- **Hiding** - things they know but are actively concealing from a specific other character

The **Perspectives & Secrets** section in the settings panel shows all extracted entries for the current character. You can add entries manually, edit existing ones, and delete entries that are no longer relevant. The section is collapsed by default on Profile A hardware (local/low-VRAM) where extraction requires a reasoning-capable model - an override toggle in the section enables it when the hardware can handle it.

**Believes** and **Hiding** entries are kept behind a spoiler block at the bottom of the list. It is collapsed by default and requires confirming a warning before it opens, so you do not accidentally reveal secrets that would spoil a mystery scenario. The block is always visible even when empty so you can tell at a glance whether any spoiler-type entries were extracted.

When the entry list grows beyond the injection budget, Smart Memory will ask whether to increase the budget for this roleplay. In normal play the budget grows in 100-token increments per confirmation; after **Memorize Chat** a single dialog sets the exact size needed plus 100 tokens of headroom. The increase applies only to the current chat and does not change the settings slider.

Long-term memories also benefit: memories extracted from a scene the responding character was not present for are labelled `[secondhand]` in their injection, so the AI knows the character learned this through hearsay rather than direct experience. This can be turned off to omit non-witnessed memories entirely.

### Entity Registry

As memories are extracted, Smart Memory tracks the named entities behind them - characters, places, objects, factions, and concepts. The AI classifies each entity by type from the context of the memory it appeared in, so invented names and made-up settings are handled just as well as real-world ones. The registry is built from both your long-term memories and your current session memories, and is updated after each extraction pass.

A collapsible **Entity Registry** panel in the settings shows all tracked entities with:

- Type badge (character / place / object / faction / concept) - click to edit if the classification is wrong
- Number of memories referencing the entity
- Last seen message index
- A **timeline button** that expands a vertical timeline of all memories involving that entity, ordered chronologically, with retired entries shown in muted style
- A **merge button** to combine two entities into one - useful when the model has created separate entries for the same person under different names, or when you want to collapse entries from different tiers together
- A **trash button** to remove an entity entirely, which also cleans up all references to it from stored memories

A **View Graph** button opens a full-screen, force-directed canvas of your character's entire memory network. Entity nodes (larger, coloured by type) are connected to the memories that reference them. Where one memory has replaced another, a directed arrow shows the supersession chain so you can see exactly what was retired and what replaced it. The graph supports pan, zoom, node dragging, click-to-highlight neighbours, and hover tooltips with full memory content. Filters for session memories and retired memories can be toggled on and off without closing the graph. The graph follows your active SillyTavern theme automatically.

### State Ledger

The State Ledger tracks the current observable physical and operational state of named entities as a compact snapshot injected near the active turn. Where long-term memory records permanent facts ("Kael was once a soldier") and session memory records developing details, the State Ledger answers right now questions: where is Kael standing, what is he wearing, what is he carrying, what is the room's current state.

State cards are maintained per entity type:

- **Character** - location, injuries, outfit/disguise, mood, active goal, carried items
- **Object** - owner, location, condition, status
- **Place** - occupants, hazards, political control, damage, accessibility
- **Faction** - leadership, objective, alliances, hostility level

Extraction runs once per session against the current message window. State cards are chat-scoped and do not carry over when a new chat begins. Cards can also be filled in or edited manually from the entity panel - click the edit button below any entity row to open the field editor.

When you merge two entities that both have state cards, a modal asks which card to keep. Deleting an entity with a state card shows a warning before proceeding. Changing an entity's type migrates its state card to the new type automatically.

The section is off by default on Profile A hardware (local/low-VRAM) because weaker models tend to pad unknown fields with placeholder values - an override toggle enables it when the local model is reasoning-capable.

**Tip:** Both Perspectives & Secrets and State Ledger inject close to the active turn by default, which gives stronger models better attention on the content. If your model starts including memory content verbatim in its responses, switch these tiers to **In-Prompt** injection in the Advanced settings - this moves them clearly outside the conversation flow and stops the leaking.

### Away Recap

Come back after a long break and not quite remember where you left off? Smart Memory generates a short "Previously on..." recap and shows it as a dismissible popup before the AI responds. It disappears after the first response - just a gentle reminder, not a permanent fixture.

### Continuity Checker

A manual tool for when something feels off. Click **Check Last Response** (or use `/sm-check`) and Smart Memory asks the AI whether the last response contradicts anything in your es