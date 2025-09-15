# Sophia’s Lorebary Overview

Sophia’s Lorebary is **middleware**: it processes *after* JanitorAI scripts and before the model sees the prompt.  
Where JanitorAI scripts are reactive and lightweight, Lorebary handles the **heavy lifting of structured lore, triggers, and world state**.

---

## How Lorebary Works

- Lorebary reads user + model messages, parses for trigger keywords, and injects the relevant lorebook entries into context.  
- It operates on JSON-formatted lorebooks, which define **entries** with triggers, content, and metadata:contentReference[oaicite:0]{index=0}:contentReference[oaicite:1]{index=1}.  
- Lorebary sits *downstream* of JanitorAI scripts:  
  - **Scripts** = real-time event logic (probabilities, groups, acts).  
  - **Lorebary** = persistent anchors, NPC/Place descriptions, background lore.  

This separation allows you to keep **cards lean**, **scripts smart**, and **lorebooks deep**.

---

## JSON Structure (Core Fields)

- **_code**: Internal identifier (ignore).  
- **entries**: Numbered dictionary of lore entries. Each entry has:  
  - **uid**: Numeric ID (0, 1, 2 …).  
  - **key**: Primary triggers (names, places, items).  
  - **keysecondary**: Synonyms/alt triggers.  
  - **comment**: Notes for creators (not injected).  
  - **content**: The text injected into context when triggered.  
  - **constant**: If `true`, always active (main anchors).  
  - **selective**: If `true`, requires all keys to be present before triggering.  
  - **disable**: If `true`, entry ignored.  
  - **order**: Priority 1–100 (higher = earlier injection).  
  - **position**: Placement order in prompt (0 = auto).  
  - **category**: Must be one of: `character`, `place`, `item`, `spell`, `letter`, `backgroundinfo`, `worldinfo`, `rule`, `other`:contentReference[oaicite:2]{index=2}.  
- **meta**: Metadata (title, author, description, timestamps, sharing flags).  

---

## Best Practices
- Use **constant anchors** for {{char}} and formatting rules (order 100).  
- Group NPCs, Places, and Rules at order ~90.  
- Background/world info at ~70–80.  
- Keep content **concise but evocative**—scripts handle logic, lorebooks handle flavor.  
- Consider Lorebary Plugins for advanced features (state tracking, memories, dynamic expansions).  

---

# Lorebary JSON Example

json
{
  "_code": "AB12CD34",
  "entries": {
    "0": {
      "uid": 0,
      "key": ["Elara Myles", "Field Epigrapher"],
      "keysecondary": ["Elara", "Myles"],
      "comment": "Core character anchor. Always on.",
      "content": "Elara Myles (24) — archaeologist with dry wit and fragile foundation; quirks: pencil-biting, hums while mapping.",
      "constant": true,
      "selective": false,
      "disable": false,
      "order": 100,
      "position": 0,
      "category": "character"
    },
    "1": {
      "uid": 1,
      "key": ["Kellan Rhys", "Rival Salvager"],
      "keysecondary": ["Kellan", "Rhys"],
      "comment": "Rival NPC with opportunistic motives.",
      "content": "Kellan Rhys runs a lean salvage crew: smiles first, cuts last; shadows firelight to poach paths.",
      "constant": false,
      "selective": false,
      "disable": false,
      "order": 90,
      "position": 0,
      "category": "character"
    },
    "2": {
      "uid": 2,
      "key": ["Storm Temple", "Basalt Terrace"],
      "keysecondary": ["Flooded Stairwell"],
      "comment": "Atmospheric place description.",
      "content": "Wave-battered terrace; salt spray, basalt steps weeping tidewater, lanterns flickering across lagoon.",
      "constant": false,
      "selective": false,
      "disable": false,
      "order": 90,
      "position": 0,
      "category": "place"
    },
    "3": {
      "uid": 3,
      "key": ["Rule", "Consent", "Aftercare"],
      "keysecondary": ["NSFW", "Safety"],
      "comment": "Behavior/safety rails.",
      "content": "Intimacy is allowed only with explicit consent, must be character-driven, and always include aftercare.",
      "constant": false,
      "selective": false,
      "disable": false,
      "order": 80,
      "position": 0,
      "category": "rule"
    }
  },
  "meta": {
    "title": "Elara Myles Lorebook",
    "author": "yourusername",
    "description": "Structured lorebook for Elara, NPCs, places, and rules. Middleware used after JanitorAI scripts.",
    "category": "other",
    "public": false,
    "updatedAt": "2025-09-12T00:00:00.000Z",
    "allowDownloads": false,
    "creatorUsername": "yourusername",
    "creatorId": "00000000-0000-0000-0000-000000000000",
    "createdAt": "2025-09-12T00:00:00.000Z",
    "userId": "00000000-0000-0000-0000-000000000000"
  }
}
