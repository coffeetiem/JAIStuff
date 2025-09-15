# JAIStuff
---
## Intro
I put together this repo to share some of the docs I use for creating roleplay bots with ChatGPT/Gemini, JanitorAI, and (optionally) Sophia’s Lorebary.  

The goal is to provide a **structured workflow**:  
- `anchorindex.md` defines the schema and how the docs fit together.  
- `gptprompt.md` sets the master system context and runtime rules.  
- Supporting docs (template, DeepseekRails, scripts, Lorebary) provide detail and examples.  

> ⚠️ Note: Bots should always function with **JanitorAI Scripts alone**. Lorebary is middleware for persistence and depth, not a requirement.

---

## What you should know
When starting a new character build, I typically tell ChatGPT or Gemini to reference `anchorindex.md` as the master index and then attach the following files:

- `/GPT_Prompts/anchorindex.md` → Index/schema for how to use all the other docs in combination.  
- `/GPT_Prompts/gptprompt.md` → Master system prompt with formatting rules, pacing, continuity, and implementation notes.  
- `/Process/template.md` → Character Card template. Includes personality structure, scene constraints, and system notes. View in raw markdown or an editor like Notepad++ for best results.  
- `/Process/DeepseekRails.md` → Roleplay rails for DeepSeek v3.1. Covers formatting, continuity beacons, NSFW rules, and golden rules.  
- `/ScriptingAndLorebooks/JanitorAIscriptsoverview.md` → Core reference for writing and structuring JanitorAI Scripts. Explains both lorebook-style and advanced JavaScript modes, with examples.  
- `/ScriptingAndLorebooks/LorebaryJSONReadme.md` → Guide for creating JSON lorebooks in Sophia’s Lorebary. Optional middleware for structured lore, NPCs, and rules.  

---

## Other things to consider
AI helpers forget context over time. You can mitigate this by:  
- Re-uploading these reference docs occasionally into your chat session with GPT.  
- Reminding GPT what each file is for during long sessions.  

Also:  
- Vanilla ChatGPT isn’t great at writing scripts by default. I offload script creation to ChatGPT Codex (a coding focused GPT model) connected to GitHub. This workflow produces stronger JavaScript and JSON for advanced use.  
- Keep your GitHub repo in sync with your evolving characters, scenarios, and worldbuilding, then use prompts like: *“How would you tell ChatGPT Codex to create `X`?”*  

This way, your workflow balances **creativity in ChatGPT** with **precision in Codex/GitHub**.
