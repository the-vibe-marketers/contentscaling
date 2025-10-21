

# What this workflow does

* Listens to a Slack channel for any new message → passes that message text to an AI Agent (backed by OpenRouter’s `anthropic/claude-sonnet-4`) with a hard rule to produce **only 300–400 words** → sends the generated content back to Slack and **waits for your reply** in the same DM/thread. 



# 1) Prerequisites

1. **n8n** up and running (cloud or self-hosted).
2. **Slack**: a workspace where you can add an app/bot with OAuth2 and the scopes to read channel messages & send DMs/messages.
3. **OpenRouter** account & API key (the model node is already set to `anthropic/claude-sonnet-4`). 

---

# 2) Import the JSON

* In n8n: **Workflows → Import from file** → select `My workflow 10.json`.
  The canvas will show these nodes: **Slack Trigger1 → AI Agent3 → Send message and wait for response**, plus **OpenRouter Chat Model1** connected into the agent. 

---

# 3) Wire up credentials (one time)

1. **Slack Trigger1**

   * Open node → set **Credentials** to your Slack OAuth (the placeholder name shown is “Slack feedback”).
   * Pick the **Channel** to listen on in **channelId** (it’s empty in the JSON). 
2. **OpenRouter Chat Model1**

   * Open node → attach your **OpenRouter** credential/key. Model is already `anthropic/claude-sonnet-4`. 
3. **AI Agent3**

   * Nothing to paste here; it **uses the upstream model** connection. Verify the prompt field shows:
     `={{ $json.message.text }} ... Only 300 to 400 word output not too long`. 
4. **Send message and wait for response**

   * Attach your Slack OAuth (placeholder “Slack community account”).
   * In **User**, pick who should receive the AI output (it’s blank in the JSON).
   * The message template is preset to:
     `Here is your content - {{ $json.output }}`. 

---

# 4) How the flow runs (end-to-end)

1. **Slack Trigger1** fires on each new message in the chosen channel and exposes it as `$json.message.text`. 
2. **AI Agent3** receives that text as its “topic/content to write,” and strictly asks the model to produce **300–400 words**. 
3. **OpenRouter Chat Model1** provides the LLM (`claude-sonnet-4`) that the agent uses to generate the output. 
4. **Send message and wait for response** DM’s (or messages) the chosen Slack **User** with:
   `Here is your content - {{ $json.output }}` and then **waits for their reply** (so you can branch later if you add more nodes). 

---

# 5) Configure & test

1. **Set the trigger channel** in **Slack Trigger1**.
2. **Select the recipient user** in **Send message and wait for response**.
3. Ensure **both Slack nodes** point to valid OAuth2 credentials and **OpenRouter** is connected. 
4. **Activate** the workflow.
5. Post any message in the configured Slack channel (e.g., “Write a brief on agentic marketing workflows”).
6. You should receive a DM (or message) with the 300–400 word piece; reply there to satisfy the “wait for response” step. 

---

# 6) Useful tweaks

* **Change the recipient**: switch **Send message and wait for response → User** to yourself or a reviewer. 
* **Tighten word count**: in **AI Agent3 → text**, add a harder constraint, e.g., “**Do not exceed 380 words.** Return plain text only.” 
* **Swap models**: change **OpenRouter Chat Model1 → model** to another OpenRouter model if desired. 
* **Threading**: if you prefer a thread reply instead of DM, point the **Send & wait** node to a channel + thread TS in advanced options (or replace it with a normal Slack Send node). 

---

# 7) Troubleshooting

* **No messages captured**: confirm **Slack Trigger1** is subscribed to the correct channel and that your app is installed in that channel. 
* **No AI output**: ensure **OpenRouter** credential is valid and the model name hasn’t been changed to an unavailable one. 
* **Send & wait doesn’t deliver**: verify the **User** field isn’t empty and your bot has permission to DM that user. 

That’s it—plug in the two Slack fields (channel + user), attach your OpenRouter key, activate, and you’re rolling. 
