
# 0) What this workflow does (at a glance)

* Listens to a Slack channel for a keyword → scrapes recent X/Twitter posts via Apify → filters >10k-follower profiles → fetches their latest posts → clusters tweets into themes with LLMs → writes themes to Google Sheets → summarizes top opportunities → sends a Slack summary. 

---

# 1) Prerequisites

1. **n8n** running (cloud or self-hosted).
2. **Slack**: one bot app with:

   * `chat:write`, and event subscription for messages in the target channel.
   * OAuth installed to your workspace; keep the bot token handy.
3. **Apify**: an API token with access to actor `danek/twitter-scraper-ppr`.

   * You’ll paste it where the JSON has `apify_api_(YOUR API KEY)`. 
4. **Google Sheets**: a spreadsheet with three tabs:

   * **Profiles with > 10k followers for the keyword** (gid `1492699236`)
   * **General theme around keyword** (gid `1559775025`)
   * **Profiles > 10k followers discuss** (gid `0`)
     The JSON already references those GIDs; you’ll just point to the **Document ID** in each Sheets node. 
5. **LLM access**:

   * OpenAI key (for `gpt-4o-mini` nodes)
   * OpenRouter key (for `perplexity/sonar-reasoning`, `anthropic/claude-sonnet-4`)
     Set these in n8n Credentials used by the nodes. 

---

# 2) Import the workflow JSON

1. In n8n → **Workflows** → **Import from File** → select your JSON.
2. After import, open the workflow to wire credentials (next step). 

---

# 3) Connect credentials in each node (one time)

* **Slack Trigger** & **Slack (send)** nodes

  * Attach your Slack OAuth2 credential (the JSON shows placeholders named “Slack feedback” & “Slack community account”). Also pick the **Channel** or **User** where needed. 
* **HTTP Request / HTTP Request2** (Apify)

  * Replace `token=apify_api_(YOUR API KEY)` with your real token.
  * Keep method `POST`, and the JSON bodies as-is (they interpolate Slack text/handles). 
* **Google Sheets** nodes (3 of them)

  * Select your **Google Sheets OAuth** credential.
  * Set **Document ID** to your spreadsheet (the JSON leaves it blank).
  * Keep **SheetName** as already configured (GIDs match the three tabs listed above). 
* **OpenAI Chat Model / OpenAI Chat Model1**

  * Point to your OpenAI credential. Model value is `gpt-4o-mini`. 
* **OpenRouter Chat Model / OpenRouter Chat Model1**

  * Point to your OpenRouter credential (keeps models `perplexity/sonar-reasoning` and `anthropic/claude-sonnet-4`). 

---

# 4) How the data flows (end-to-end)

1. **Slack Trigger** fires when a message arrives in the configured channel. The message text becomes your **keyword**. 
2. **HTTP Request (Apify search)** queries Twitter “Latest” with that keyword (up to 300 posts). 
3. **Filter** keeps only tweets whose `user_info.followers_count > 10000`. 
4. **Edit Fields1** extracts `name`, `followers`, `bio` from `user_info` for each qualifying profile. 
5. **HTTP Request2 (Apify by username)** fetches up to 20 latest posts for each found profile (batched 3 users per 12s to be polite with rate limits). Feeds two paths: `Merge` and `Merge1`. 
6. **Merge** + **Code** (analytics) roll up per-user stats (active days, best weekday/hour UTC, engagement, content mix). (Optional output—kept for insights.) 
7. **Edit Fields → Code2 → Code6 → AI Agent1 → Code3 → Google Sheets2**

   * Builds “All text,” chunks to ~20 sets, clusters themes/sentiment via OpenAI, parses JSON, and appends or updates **General theme around keyword** (themes + sentiment + summary + opportunities). 
8. **Merge1 → Edit Fields2 → Code4 → Code1 → AI Agent → Code5 → Google Sheets**

   * Formats tweets again (profile-focused), chunks, clusters with another agent prompt, parses JSON, then appends to **Profiles > 10k followers discuss** (theme/sentiment/summary). 
9. **Google Sheets2 → Code7 → AI Agent2**

   * Reads the general themes just written, condenses to **Top 5 opportunities for today** (200-ish words cap), using OpenAI/OpenRouter as configured. 
10. **AI Agent2 → Slack (Send a message)**

* Posts the final summary back to Slack (and provides a Telegram CTA string included in the node’s text). 

11. (Optional second Slack flow) **Slack Trigger1 → AI Agent3 → Slack (send and wait)** produces a 300–400 word content piece from a Slack DM/mention and awaits a reply, separate from the main keyword-to-themes pipeline. 

---

# 5) Node-specific edits you’ll likely make

* **HTTP Request / HTTP Request2**: paste your **Apify token**. 
* **Google Sheets (all 3)**: set the **Document ID** (the JSON leaves it empty). Keep the provided GIDs. 
* **Slack Trigger(s)**: pick the channel(s) to listen to; for the “send and wait” node, pick the **User** you want to DM. 
* **Credentials**: attach your OpenAI and OpenRouter creds to their respective model nodes. 

---

# 6) Test it

1. **Activate** the workflow.
2. In the chosen Slack channel, type a keyword (e.g., `account abstraction`).
3. Watch the execution: you should see Sheets fill:

   * Tab “Profiles with > 10k followers for the keyword” with name/followers/bio.
   * Tab “General theme around keyword” with theme/sentiment/summary/opportunities.
   * Tab “Profiles > 10k followers discuss” with theme/sentiment/summary.
     Then a summary message appears in Slack. 

---

# 7) Tips & gotchas

* **Rate limits**: `HTTP Request2` batches 3 users every 12s; keep if you hit Apify/remote throttling. 
* **Parsing safety**: `Code3`/`Code5` strip ```json fences and validate arrays before writing to Sheets. If you see “⚠️ No valid themes parsed.” the LLM likely produced non-JSON text—re-run or shorten chunks. 
* **Time zones**: “Most active hour” is **UTC** by design in the analytics code; convert downstream if you want IST. 
* **Security**: don’t hardcode tokens; store them in n8n Credentials/Env and template into nodes.
* **Sheet structure**: keep the 3 tabs named as in the JSON or update the nodes’ sheet selectors if you rename. 

---

# 8) Quick map of key nodes (for orientation)

* **Slack Trigger** → **HTTP Request (Apify search)** → **Filter (>10k)** →
  **Edit Fields1** → **HTTP Request2 (by username)** →
  a) **Merge → Code (analytics)**
  b) **Merge1 → (Edit/Code/AI/Parse) → Sheets (General & Profiles discuss)** → **Summarize (AI Agent2)** → **Slack send**. 

Note: When you are testing make sure you are doing it with manual trigger
