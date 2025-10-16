# JAIscripts.md — v2
*A practical, copy-pasteable handbook for building JanitorAI Scripts fast. Integrated with field-tested patterns from the JAI KBs and community advanced examples.*

# JanitorAI Scripts Overview

JanitorAI offers **Scripts** as a way to dynamically inject guidance, world details, or rules into the AI’s context without permanently bloating the character card. Scripts make character interactions richer and more flexible by responding to the **user’s messages** and/or **timing within the conversation**.

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

## 0) What this gives you
- **Clear mental model** of Lorebook vs Advanced Scripts
- **Drop-in templates** for: Advanced Lorebook (v12-style), Dynamic Lorebook (simple), Emotion Engine (lite), Random Encounter (weighted), Likes/Dislikes/Fears (preference capture)
- **Recipe patterns** (probability, groups, gating, staging, daisy-chains)
- **Debug & QA checklist** to keep scripts predictable

> **Only user messages trigger entries.** Bot messages don’t. Triggered content affects the **next reply only** unless marked constant or retriggered.

---

## 1) Quick Start — choose your path

### A. Lorebook (no code)
Use for one-shot facts or ambience snippets.

**Fields to care about**: Primary keys · Secondary keys · Probability · Group label · Min Messages · Constant.

**Use cases**: setting details, NPC dossiers, format anchors, safety rails.

### B. Advanced Script (code)
Use when you need multiple pockets (personality + scenario), staged arcs, tag chaining, or cross-entry logic.

**Core concepts**: keywords, requires/not, triggers (tags), min/max messages, priority buckets, group resolution, apply limit.

---

## 2) Templates (copy/paste)

### 2.1 Advanced Lorebook — compact v12-style skeleton
```js
/* Advanced Lorebook — compact skeleton */
// Output guards (append-only)
context.character = context.character || {};
context.character.personality = typeof context.character.personality === "string" ? context.character.personality : "";
context.character.scenario  = typeof context.character.scenario  === "string" ? context.character.scenario  : "";

// Window controls (how many recent msgs to scan)
var WINDOW_DEPTH = 5;

// Canonicalize last N messages (whole-word match; hyphen/underscore → space)
function norm(s){ s = String(s||"").toLowerCase().replace(/[^a-z0-9_\s-]/g," ").replace(/[-_]+/g," ").replace(/\s+/g," ").trim(); return s; }
var msgs = (context.chat && context.chat.last_messages) || [];
var joined = msgs.slice(-Math.max(1,Math.min(20,WINDOW_DEPTH))).map(m=>String(m && (m.message||m))).join(" ");
var HAY = " " + norm(joined) + " ";

function has(term){
  var t = String(term||"").toLowerCase().trim();
  if(!t) return false;
  if(t.endsWith("*")){
    var stem = t.slice(0,-1).replace(/[.*+?^${}()|[\]\\]/g,"\\$&");
    return new RegExp("(^|\\s)"+stem+"[a-z]*?(?=\\s|$)").test(HAY);
  }
  var w = t.replace(/[.*+?^${}()|[\]\\]/g,"\\$&");
  return new RegExp("(^|\\s)"+w+"(?=\\s|$)").test(HAY);
}

// Entries (edit below)
var APPLY_LIMIT = 6; // cap per turn (highest priority wins first)
function msgCount(){
  var c = (context.chat || {});
  var n = (c.messageCount != null) ? c.messageCount : c.message_count;
  return (n|0);
}
var entries = [
  // Always-on tiny anchor (optional)
  {priority:2, personality:" Keep replies concrete; show don’t tell; end with a hook."},

  // Greeting example → emits a tag other entries can listen for
  {keywords:["hello","hi","hey"], priority:4, triggers:["base_greet"], personality:" {{char}} greets warmly and asks how they can help."},
  {tag:"base_greet", priority:5, personality:" {{char}} confirms names if given and restates the greeting succinctly."},

  // Staged reveal example
  {keywords:["backstory","origin"], min:12, priority:4, personality:" {{char}} shares a concise origin beat; keep it actionable."},

  // Grouped ambience (only one fires)
  {group:"ambience", probability:1, keywords:["cafe","coffee"], scenario:" Steam sighs; ceramic clinks under low jazz."},
  {group:"ambience", probability:1, keywords:["library","stacks"], scenario:" Dust motes drift in slant light between tall stacks."},
];

// Engine
function probOK(p){ if(p==null) return true; var n = (typeof p==="string"&&p.endsWith("%"))?+p.slice(0,-1)/100:+p; n = isFinite(n)?Math.max(0,Math.min(1,n)):1; return Math.random() < n; }
function kwHit(e){ return Array.isArray(e.keywords) && e.keywords.some(has); }
function pass(e){
  if (e.min != null && msgCount() < e.min) return false;
  if(e.keywords && !kwHit(e)) return false;
  return probOK(e.probability);
}
var picked = [], tags = Object.create(null);
for (var i=0;i<entries.length;i++){
  var e=entries[i];
  if((!e.keywords && e.tag==null) || kwHit(e)){
    if(pass(e)){ picked.push({i,p:e.priority||3}); (e.triggers||[]).forEach(t=>tags[t]=1); }
  }
}
for (var j=0;j<entries.length;j++){
  var e=entries[j]; if(tags[e && e.tag]) picked.push({i:j,p:e.priority||3});
}
// Priority sort (5→1) + apply limit
picked.sort((a,b)=> (b.p||3)-(a.p||3)); picked = picked.slice(0,APPLY_LIMIT);
var bufP = "", bufS = "";
picked.forEach(({i})=>{
  var e=entries[i]||{};
  if(e.personality) bufP += "\n\n"+e.personality;
  if(e.scenario)    bufS += "\n\n"+e.scenario;
});
if(bufP) context.character.personality += bufP;
if(bufS) context.character.scenario    += bufS;
```
**When to use:** You want tag chaining, staged reveals, multi-pocket output, and strict apply limits—without pulling in a huge engine.

### 2.2 Dynamic Lorebook — “no-frills” ES5
```js
// Dynamic Lorebook (simple) — keywords → personality/scenario
context.character = context.character || {};
if(typeof context.character.personality!=="string") context.character.personality="";
if(typeof context.character.scenario!=="string") context.character.scenario="";
var last = String((context.chat && (context.chat.last_message||context.chat.lastMessage))||"").toLowerCase();
var count = (context.chat && context.chat.message_count)||0;

var lore = [
  {keys:["mother","mom","family"], min:0, personality:" Quietly pragmatic; loves by provisioning.", scenario:" Mentions Sunday check-ins and care packages."},
  {keys:["father","dad","old photos"], personality:" Supportive, steady; values shared silence.", scenario:" Recalls evening walks and reading aloud."}
];

for (var i=0;i<lore.length;i++){
  var e=lore[i]; if(e.min && count<e.min) continue;
  var hit=false;
  for(var k=0;k<(e.keys||[]).length;k++){
    if(last.indexOf(String(e.keys[k]).toLowerCase())!==-1){ hit=true; break; }
  }
  if(hit){
    if(e.personality) context.character.personality += "\n\n"+e.personality;
    if(e.scenario)    context.character.scenario    += "\n\n"+e.scenario;
  }
}
```
**When to use:** You just need keyword → text without groups/priorities.

### Emotion Engine — lite (top-3 non-conflicting)
```js
// Emotion Engine (lite)
context.character=context.character||{};
context.character.personality=context.character.personality||"";
context.character.scenario=context.character.scenario||"";
function canon(s){s=String(s||"").toLowerCase().replace(/[^\x20-\x7e]/g," ").replace(/[^a-z0-9\s]/g," ").replace(/\s+/g," ").trim();return s;}
function pad(s){return " "+s+" ";}
var msg = pad(canon((context.chat && context.chat.last_message)||""));
var packs=[
  {cat:"Joy",  keys:["happy","excited","great","awesome"],         pers:" joyful, upbeat",  scen:" The air feels lighter."},
  {cat:"Sad",  keys:["sad","miserable","lonely","burnt out"],      pers:" somber, gentle",  scen:" A quiet heaviness lingers."},
  {cat:"Angry",keys:["angry","furious","irritated","can’t stand"], pers:" tense, clipped",  scen:" The air crackles with friction."},
  {cat:"Calm", keys:["calm","peaceful","relaxed","serene"],        pers:" calm, composed",  scen:" A tranquil steadiness settles."}
];
var hits=[]; for(var i=0;i<packs.length;i++){ for(var j=0;j<packs[i].keys.length;j++){ if(msg.indexOf(pad(canon(packs[i].keys[j])))!==-1){ hits.push(packs[i]); break; }}}
for(var h=0;h<Math.min(3,hits.length);h++){
  context.character.personality += "\n\nThe mood is"+hits[h].pers+".";
  context.character.scenario    += "\n\n"+hits[h].scen;
}
```
**When to use:** You want quick tonal scaffolding without full taxonomy.

### 2.4 Random Encounter — weighted, one-pick
```js
// Random Encounter (weighted one-pick)
context.character=context.character||{};
context.character.personality=context.character.personality||"";
context.character.scenario=context.character.scenario||"";
var RATE=0.10, LIMIT=1;
if(Math.random()>RATE) return;
var choices=[
  {w:3, personality:" She smirks over the laundry: ‘Fold mine and I’ll… supervise.’", scenario:" A soft heap of tees blooms on the couch."},
  {w:3, personality:" She leans on the counter: ‘We could do dishes… or not.’",       scenario:" Artful stacks glint in the sink."},
  {w:2, personality:" She taps your phone: ‘Rent’s due. Pay me in compliments first.’", scenario:" A ping; she lounges like a landlord of vibes."}
];
var sum=choices.reduce((a,c)=>a+(+c.w>0?+c.w:0),0); if(sum<=0) return;
var r=Math.random()*sum,acc=0,pick=-1;
for(var i=0;i<choices.length;i++){ var w=+choices[i].w||0; if(w<=0) continue; acc+=w; if(r<=acc){ pick=i; break; }}
if(pick<0) return; var c=choices[pick]; if(!c) return;
if(LIMIT>0){
  if(c.personality) context.character.personality += "\n\n"+c.personality;
  if(c.scenario)    context.character.scenario    += "\n\n"+c.scenario;
}
```
**When to use:** Ambient variety or event hooks without flooding.

### 2.5 Likes/Dislikes/Fears — preference capture
```js
// Preference Capture (first-hit per subpack)
context.character=context.character||{};
context.character.personality=context.character.personality||"";
context.character.scenario=context.character.scenario||"";
function canon(s){s=String(s||"").toLowerCase().replace(/[^\x20-\x7e]/g," ").replace(/[^a-z0-9\s]/g," ").replace(/\s+/g," ").trim();return s;}
function pad(s){return " "+s+" ";}
var msg = pad(canon((context.chat && context.chat.last_message)||""));
function has(tok){tok=pad(canon(tok)); return msg.indexOf(tok)!==-1; }
var PACKS=[
  {name:"LIKES",    limit:1, rules:[{keywords:["love","like","fond of"],           scenario:" Record a like linked to the cue.",     personality:" Mark tone warmer when cue is present."}]},
  {name:"DISLIKES", limit:1, rules:[{keywords:["hate","dislike","not a fan"],      scenario:" Record a dislike linked to the cue.",  personality:" Mark tone cautious when cue is present."}]},
  {name:"FEARS",    limit:1, rules:[{keywords:["afraid of","scared of","anxious about"], scenario:" Record a fear linked to the cue.", personality:" Mark tone gentle around this cue."}]}
];
for (var p=0;p<PACKS.length;p++){
  var used=0, rs=PACKS[p].rules||[];
  for(var r=0;r<rs.length;r++){
    if(used>=(PACKS[p].limit||1)) break;
    var rule=rs[r]; var tok=""; var keys=rule.keywords||[];
    for(var k=0;k<keys.length;k++){ if(has(keys[k])){ tok=keys[k]; break; }}
    if(tok){
      if(rule.personality) context.character.personality += "\n\n"+rule.personality;
      if(rule.scenario)    context.character.scenario    += "\n\n"+rule.scenario;
      used++;
    }
  }
}
```
When to use: Lightweight “record the user’s stated preferences” behavior.

### 2.6 Hybrid Entries (Basic + per-entry Advanced)

> Keep the **Lorebook** overall in **Basic** mode, but flip **specific entries** to **Advanced (Script)** so each one can decide—via code—whether it should fire. Standard UI fields (Group, Priority) still apply **after** your script returns `true`.

**When to flip an entry to Advanced Activation**
- Composite conditions (e.g., *message count* **and** *keyword* **and** *not during onboarding*).
- Regex/pattern checks beyond simple word match.
- Narrow windows (turn ranges), cool-downs, or one-shots.
- You want randomness in one place (do the roll in code; set UI Probability = **100%**).

### 2.7 Advanced Entry (Content Script) — multi-pocket injection

> Use this when a **single Lorebook entry** needs to inject **both** `personality` and `scenario` via code. This runs in the entry’s **Content Script** (not the Activation Script) and only fires if the entry wins normal selection (groups/priority/probability or your per-entry activation returns `true`).

**When to use**
- You want a one-off entry to add multi-pocket text without the global engine.
- You’re mixing Hybrid activation (2.6) + this content script for richer output.
- You want to keep prose localized to a specific entry.

**UI guidance**
- If you control randomness in the Activation Script, set the entry’s **Probability = 100%**.
- Groups still resolve as usual; only one entry in a group applies that turn.

**Minimal Content Script (drop-in)**
```js
// Runs in this entry's **Content Script** box
context.character = context.character || {};
var p = String(context.character.personality || "");
var s = String(context.character.scenario || "");

// Optional: tiny dedupe guard (don't re-append twice in the same turn)
context.state = context.state || {};
var c = (context.chat || {});
var turn = ((c.messageCount != null) ? c.messageCount : c.message_count) | 0;
if (context.state._lastThisEntryTurn === turn) return;
context.state._lastThisEntryTurn = turn;

// Inject multi-pocket content
context.character.personality = p + "\n\n {{char}} keeps replies concrete and ends with one actionable next step.";
context.character.scenario    = s + "\n\n Neon rain needles the window; a tram hums along the elevated line.";
```
**Variant: react to the last user message**
```js
context.character = context.character || {};
var p = String(context.character.personality || "");
var s = String(context.character.scenario || "");

// Access last user message (normalized)
var c = (context.chat || {});
var last = String(c.lastMessage || c.last_message || "");
var low  = last.toLowerCase();

// Light branching for tone
var tone = /\b(sad|tired|burnt out|exhausted)\b/.test(low)
  ? "gentle, unhurried; validates feelings before suggesting next step."
  : "engaged, confident; offers a concise plan then invites input.";

context.character.personality = p + "\n\n {{char}} is " + tone;
context.character.scenario    = s + "\n\n Streetlights smear across wet asphalt; distant cables hiss.";
```
**Tips**

-   Keep Content Scripts short; put 1--2 sentences per pocket.

-   If you also use a **per-entry Activation Script** (2.6), let that script decide **when**, and let this content script decide **what** to inject.

-   Avoid "double randomness": either roll in activation or set Probability < 100, not both.

---

#### How to use these snippets
1) Edit an entry → choose **Advanced (Script)**.  
2) **Set Probability = 100%** in the UI (avoid double randomness).  
3) Paste **one** of the scripts below into the entry’s **Activation Script** box.  
   - Each script is **self-contained** (includes the small helpers).
4) Return `true` to activate; no `return true` ⇒ treated as `false`.

---

#### A) Gate by turn count only
```js
// Helpers (portable)
function msgCount(){
  var c = (context.chat || {});
  var n = (c.messageCount != null) ? c.messageCount : c.message_count;
  return (n|0);
}

if (msgCount() >= 12) {
  return true;
}
```

#### B) Keyword AND turn count
```js
function msgCount(){
  var c = (context.chat || {});
  var n = (c.messageCount != null) ? c.messageCount : c.message_count;
  return (n|0);
}
function lastMsgLower(){
  var c = (context.chat || {});
  var s = String(c.lastMessage || c.last_message || "");
  return s.toLowerCase();
}

if (msgCount() >= 8 && /\b(origin|backstory|how it started)\b/.test(lastMsgLower())) {
  return true;
}
```

#### C) Windowed reveal (only between turns 10–25)
```js
function msgCount(){
  var c = (context.chat || {});
  var n = (c.messageCount != null) ? c.messageCount : c.message_count;
  return (n|0);
}

var n = msgCount();
if (n >= 10 && n <= 25) {
  return true;
}
```

#### D) Probabilistic gate (roll in script; UI Probability = 100)
```js
function msgCount(){
  var c = (context.chat || {});
  var n = (c.messageCount != null) ? c.messageCount : c.message_count;
  return (n|0);
}

// Optional: add a soft floor so it won't fire too early
if (msgCount() >= 3 && Math.random() < 0.35) {
  return true;
}
```

#### E) Cool-down example (fire at most once every 6 turns)
```js
function msgCount(){
  var c = (context.chat || {});
  var n = (c.messageCount != null) ? c.messageCount : c.message_count;
  return (n|0);
}

var n = msgCount();
context.state = context.state || {};
if (!context.state._lastRevealTurn || (n - context.state._lastRevealTurn) >= 6) {
  context.state._lastRevealTurn = n;  // remember when we last fired
  return true;
}
```

**Cheat sheet --- UI fields vs per-entry script**

-   **UI → Min Messages / Probability / Group / Priority**: keep simple; set **Probability=100%** when your script handles randomness.

-   **Script → Complex gating**: regex, multi-term logic, windows, cool-downs, one-shots.

-   **Groups still resolve** after your script returns `true` (only one entry in a group applies that turn).

**Gotchas**

-   If you don't `return true`, it's treated as `false` (no activation).

-   Avoid **double randomness** (don't also use UI Probability < 100).

-   Keep activation scripts short; put prose in the entry body.

* * * * *

3) Recipe Patterns
------------------

-   **One-per-group**: Give related entries the same `group` label; set Probability to 100% on each and let weight/order decide the winner.

-   **Staged arcs**: Use `min` or `minMessages` thresholds (e.g., Act II keys unlock at ≥20 msgs; finales at ≥40 msgs).

-   **Tag fan-out**: One keyword entry emits a tag (e.g., `triggers:["base_open"]`), then several tag-only entries provide clean checklists.

-   **Suffix wildcards**: `"clos*"` matches `close/closing/closure` --- use sparingly to avoid false positives.

-   **Apply caps**: Keep `APPLY_LIMIT` ~3--6 to avoid prompt bloat.

-   **Name blocks**: Prevent self-trigger loops if your bot's name appears in user text.

* * * * *

4) Debug & QA Checklist
-----------------------

-   **Does a user message actually contain the trigger keys?** (remember: bot replies don't trigger)

-   **Probability rerolls each message** --- try multiple turns or set to 100% while testing.

-   **Min Messages** gates---verify message count is high enough.

-   **Group collisions** --- if several entries are eligible, ensure only one speaks.

-   **Apply limit** --- if exceeded, lower priority entries will be dropped.

-   **Case sensitive** --- only enable when absolutely necessary.

-   **Audit the Debug Panel** --- inspect fired entries and injected deltas.

* * * * *

5) When to keep it Basic vs go Advanced
---------------------------------------

-   **Stay Basic** when a single text chunk covers it and you don't need staging or multi-pocket output.

-   **Go Advanced** when you need: multi-pocket output, staged arcs, daisy-chains, or strict caps.

* * * * *

6) Authoring style tips
-----------------------

-   Write injections as **short, directive sentences** the model can fold into its reply.

-   Prefer **actionable** beats (what to do / what to notice) over long exposition.

-   Keep constants tiny; push scene prose into triggered entries.

* * * * *

### Changelog (v2)

-   Added compact Advanced Lorebook skeleton

-   Added ready-to-use Emotion/Encounter/Preference engines (lite)

-   Added **Hybrid Entries (Basic + per-entry Advanced)** section + UI↔Script cheat sheet

-   Consolidated patterns + QA checklist