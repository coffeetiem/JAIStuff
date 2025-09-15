**JanitorAI "Scripts" Feature (Lorebooks) – Summary**

- **Definition:** *Scripts* in JanitorAI are a dynamic lorebook system. They let bot creators store extra information (character lore, world details, NSFW content, etc.) outside the main bot definition and inject it into the conversation **only when relevant**. This keeps bots lean but allows rich details to surface on demand.

- **How It Works:** Each script is made of **entries** with associated **trigger keywords**. When a **user's message** contains a trigger word, the corresponding entry’s content is **activated** and added to the AI’s prompt for its next reply. (Bot messages alone do not trigger scripts – only the user can trigger them.) By default, a triggered entry influences the bot’s **next response only** and then is removed from context (unless triggered again or marked constant).

- **Purpose/Benefits:** Scripts prevent overloading the AI with info that might not be needed. They **reduce permanent token usage** by only providing details when prompted. This allows:
  - **More detailed bots** (lots of lore, many characters, extensive backstory) without exceeding context limits.
  - **Better memory** for details: the bot can “remember” specific facts when asked, via triggers, instead of forgetting them in long chats.
  - **Controlled content flow:** e.g. NSFW or spoiler content stays hidden until the user explicitly triggers it, ensuring appropriate pacing and avoiding unwanted behavior upfront.
  - **Dynamic scenarios:** creators can include optional events or random encounters via scripts (with probability settings or specific trigger combos), making chats more interactive and varied.

- **Two Modes:** 
  1. **Lorebook Mode (Basic):** No coding required. Each entry has one set of keywords and one block of content. Ideal for straightforward use-cases (one trigger -> one lore text). Easy to import or create via UI.
  2. **Advanced Mode:** Allows multiple content sections per entry and scripting logic (in JavaScript-like format). Creators can define complex rules: e.g., multiple entries activating together, conditional triggers (requiring multiple keywords), mutually exclusive entries, time-delayed triggers, etc. Advanced mode enables **daisy-chaining** (one entry’s activation can lead to others) and setting priorities when many triggers fire at once. It’s more powerful but requires understanding the scripting syntax and logic.

- **Key Features/Settings for Entries:**
  - **Primary Keywords:** The main words/phrases that will trigger an entry. (Choose unique but intuitive keywords relevant to the entry.)
  - **Secondary Keywords:** Additional terms that must also appear for the entry to trigger (optional). Use these for more specific *AND* conditions if needed.
  - **Constant Entry:** If true, that entry is **always** included in the prompt (trigger not required). Use for essential core info that should persist every reply (e.g. protagonist’s fundamental traits).
  - **Case Sensitive:** If enabled, triggers consider letter case and punctuation (exact match). By default, "keyword" would match any case; with this on, it must match exactly.
  - **Min. Messages:** An entry will not activate until at least this many messages have occurred in the conversation. (A way to delay or stage certain lore reveals/events.)
  - **Probability:** A percentage (0–100%) chance that the entry activates when its trigger conditions are met. <100% introduces randomness (e.g. a 50% chance means sometimes the entry will fire when triggered, sometimes not).
  - **Group Label:** A label to group entries. If multiple entries share the same group and their triggers happen at once, only **one** entry from that group can activate. This prevents too many similar entries from piling on simultaneously.
  - **Group Priority/Weight:** Within a group, determines which entry wins if multiple are eligible. You can assign a weight value (higher weight = higher priority chance) or use selection modes (like “most keywords matched” wins, or a fixed priority order).
  - *(These settings can be combined. For instance, you might have a constant entry for core character info, a group of “random event” entries with probabilities, or an entry that triggers only after 30 messages and only if two specific keywords appear together, etc.)*

- **Limitations:**
  - Only user messages trigger entries. The bot cannot self-trigger its lorebook content.
  - Triggered entries persist only for the immediate next response unless the user triggers again. (No multi-turn memory for a single trigger, aside from what the bot naturally includes in conversation.)
  - Users cannot see the script/lorebook content or know the triggers by default. It’s hidden from the UI (at time of writing), so creators should design triggers that users will naturally use or hint at them in the bot’s dialogue.
  - Only the bot’s creator can add or edit its scripts. The script applies to all users of that bot once set up (i.e., it’s a shared behavior of the bot, not user-specific).

- **Best Practices:**
  - Offload non-essential but useful details from the character definition into script entries to save tokens.
  - Use clear, uncommon trigger words to avoid accidental activations. Test triggers to ensure they work in practice.
  - Keep entry content focused. Provide the info the bot needs in a concise way (the AI will integrate it into its reply). Long entries can be broken into smaller ones if needed.
  - Use **Min. Messages** to delay heavy lore drops or character reveals for better pacing. Use **Probability** and **Group labels** to add variety and avoid info overload.
  - Plan for trigger usage: if certain lore is important, consider how the conversation might naturally lead the user to mention it. You can have the bot or scenario hint at topics (“I have a strange tattoo...”) to encourage trigger words.
  - Continuously test your bot with different scenarios. Check that the right entries trigger at the right times and that the bot’s responses are improved by the injected info. Adjust triggers/content as necessary.
  - Stay updated with JanitorAI’s documentation and community guides for Scripts. This feature is evolving, and new functionalities or best practices may emerge over time.

---

# JanitorAI Scripts Overview

JanitorAI offers **Scripts** as a way to dynamically inject guidance, world details, or rules into the AI’s context without permanently bloating the character card. Scripts make character interactions richer and more flexible by responding to the **user’s messages** and/or **timing within the conversation**.

---

## Script Types

### 1. Lorebook-Style Scripts (Basic)
These are managed through the JanitorAI UI and resemble lorebook entries.

**Fields:**
- **Primary Keywords**: Words or phrases in the user’s message that activate the entry.
- **Secondary Keywords**: Optional additional triggers (both must match if filled).
- **Constant**: If checked, always injects every turn.
- **Min. Messages**: Minimum number of user turns before the entry can trigger.
- **Probability**: Chance (0–100%) for the entry to activate when triggered.
- **Group Label**: Ensures only one entry from the same group fires in a single turn.
- **Content**: The text injected into the model context when triggered.

**Usage:**
- Constants: Anchor a character’s format or personality rules.
- Triggers: Introduce setting details or NPCs only when relevant.
- Groups: Randomize hazards, scenery, or emotional beats by ensuring only one per group.

---

### 2. Advanced Scripts (JavaScript)
These use JavaScript for maximum flexibility and logic. They can:
- Gate events by **turn count**.
- Inject entries probabilistically.
- Control **one-per-group** injections.
- Create **timed “auto beats”** (recaps, ambience).
- Stage **Acts/Arcs** by using `minMessages` thresholds.

**Structure:**
- Define entries (constants, triggers, groups).
- Implement `onUserMessage(userText, api)` which runs on every user turn.
- Use `api.inject(text)` to add hidden context guidance.

---

## Common Patterns

- **Constants**: Compact reminders for voice, formatting, reply style.
- **Groups**: Prevents multiple similar entries firing at once (e.g., only one hazard).
- **MinMessages**: Stages reveals later in the arc (e.g., rival shows up after turn 10).
- **Probability**: Adds variation so world events don’t feel scripted every time.
- **Weighted Choice**: Pick one entry among many by weight.
- **Auto Beats**: Fire content every N turns (ambience, recap).
- **Act Gating**: Unlock deeper mysteries or hazards only after X turns.

---

## Best Practices
- Keep **constants** concise. Save descriptive prose for triggered entries.
- Use **groups** for variety (e.g., only one NPC beat or one hazard per turn).
- Gate **Act II/III content** with minMessages to avoid rushing.
- Test trigger keywords against how you naturally type in roleplay.
- Limit injections per turn (usually 3–4) to prevent overwhelming the model.
- Always end injections with actionable cues, questions, or environment details.

---

# JavaScript Example (Advanced Script)

```javascript
const ENTRIES = [
  // Constant: always injects
  { id:"anchor", constant:true, content:'Elara core: hyper-competent, dry wit; format "Dialogue", *Italics*, `Thoughts`; replies 6–10 sentences, end with a hook.' },

  // Place entries (group: place)
  { id:"place_terrace", group:"place", minMessages:0, probability:100,
    primary:["terrace","ruins","temple"], content:'*Salt spray needles the basalt steps; gulls wheel in the wind.*' },
  { id:"place_stairwell", group:"place", minMessages:5, probability:80,
    primary:["stairwell","submerged","green glow"], content:'*Green light trembles below the flooded stairwell; glyphs scallop the risers.*' },

  // Emotion entry
  { id:"praise", group:"emotion", minMessages:1, probability:100,
    primary:["good job","proud of you","well done"], content:'Praise response: Elara flushes, hides a smile, competence sharpens, vulnerability peeks in `Thoughts`.' }
];

const low = s => (s||"").toLowerCase();
const hasAny = (t, arr) => (arr||[]).some(k => low(t).includes(low(k)));

function onUserMessage(userText, api){
  const turn = api.getMessageCount?.() ?? 0;
  const t = userText.toLowerCase();
  const pool = [];

  // constants
  for (const e of ENTRIES) if (e.constant) pool.push(e);

  // triggers
  for (const e of ENTRIES) {
    if (e.constant) continue;
    if (turn < (e.minMessages||0)) continue;
    if (e.primary && hasAny(t, e.primary)) {
      if (Math.random()*100 < (e.probability||100)) pool.push(e);
    }
  }

  // group resolution: one per group
  const singles = []; const groups = new Map();
  for (const e of pool){
    if (!e.group) { singles.push(e); continue; }
    if (!groups.has(e.group)) groups.set(e.group, []);
    groups.get(e.group).push(e);
  }
  const resolved = [...singles];
  for (const [g, arr] of groups.entries()) resolved.push(arr[0]);

  // inject
  for (const e of resolved) api.inject(e.content);
}

registerScript({ onUserMessage });

