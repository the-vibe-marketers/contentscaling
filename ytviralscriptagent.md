# Youtube Script agent — step by step

**What it’s for:** a very small chat flow that takes your message, runs an LLM, and appends the result to Google Sheets. Ideal as a “script-on-demand” chat endpoint. 

## A. Prereqs

* n8n running, OpenAI API key, Google Sheets OAuth. 

## B. Import + wire credentials

1. **When chat message received**: no credential needed; this is your entry point. 
2. **OpenAI Chat Model**: attach your OpenAI credential; model is `gpt-4o-mini`. 
3. **Basic LLM Chain**: uses the model above; you’ll set your prompt template here. 
4. **Google Sheets (Real time script gets added here)**: set **Document ID** and **Sheet** where outputs should append. 

## C. Flow & run

* **Chat message → Basic LLM Chain → Google Sheets append.** Activate and send a message; your generated script lands in the configured sheet. 

# 3) Connect the two (use the scraped sheet as context in the chat agent)

If you want your **chat-triggered Script agent** to **read from the sheet produced by the YT workflow** and then generate a script *based on that data*, add these two steps **inside Script agent**, between **When chat message received** and **Basic LLM Chain**:

1. **Google Sheets (Read)**

   * Operation: **Read rows**
   * Document ID: **same Google Sheet** used by the YT workflow’s **Google Sheets1** (or **Google Sheets3**, depending on where your final unified data lives).
   * Range: pick columns you need (e.g., `title, views, captions/transcript`).  
2. **Adjust the LLM prompt** in **Basic LLM Chain** to include the rows:

   ```
   Use the following YouTube dataset (titles, views, transcripts) as research context:
   {{ $json.sheetData }}

   Now write a viral YouTube script matching this request:
   {{ $json.message.text }}
   ```

   * Then keep the existing **Google Sheets append** to log the generated script. 

> Tip: If the sheet is large, insert a small **Code** node to sort by `views` and pass only the top 3–5 rows into the prompt to keep tokens low. 

---

## Quick maps

**YT Script Viral Script Generating Agent (builder)**
Manual Trigger → YouTube API (playlist/keyword) → **Code (structure)** → Videos stats → **Code (stats structure)** → **Merge 1** → **Google Sheets3** → URLs → Apify transcripts (batch or per-URL) → **Merge 2** → **Google Sheets1** (+ optional Code → Google Docs). Chat Trigger → OpenAI → LLM Chain → **Sheet append**. 

**Script agent (chat → script)**
Chat Trigger → **(add: Google Sheets Read)** → **Basic LLM Chain (prompt includes sheet data)** → **Sheet append**. 

