
# Youtube Topic and content idea research — step by step

**What it’s for:** scrape YouTube videos (playlist or keyword), pull stats + transcripts, then log structured rows to Google Sheets. Also includes a chat path that appends LLM outputs to a sheet. 

## A. Prereqs

* n8n running, Google Sheets/Docs OAuth, YouTube Data API key, Apify token, OpenAI API key. 

## B. Import + identify triggers

* Import the JSON. You’ll see a **Manual Trigger** (for scraping) and a **Chat Trigger** (for real-time prompts). 

## C. Configure YouTube scraping (manual path)

1. **Set playlist or keyword source**

   * Node **“Youtube API + Creator/ topic videos to scrape”** → insert your **playlistId** + **API key** in the URL. 
   * (Alt) Node **“…to scrape1”** → set **q=your keyword** + **API key**. 
2. **Structure basic metadata**

   * **Code 1 Structure the scraped info** maps `title, videoId, url, description`. No edits needed. 
3. **Fetch stats**

   * **Get view count, like count data** → keep the videos endpoint; supply your **API key** (you can pass multiple IDs). 
   * **Code 2 – Structure YT video number based data** normalizes `views, likes, comments` and adds an index. 
4. **Merge + write to Sheets**

   * **Merge 1** (combine by position) joins metadata + stats. 
   * **Google Sheets3** → set **Document ID** and **Sheet (e.g., “All” or your target tab)**. Data appends here. 

## D. Transcribe videos

1. **Collect URLs only**

   * **Code 3 – Filter only urls…** produces a clean list of URLs. 
2. **Send to Apify transcripts**

   * **Transcribe video from url** (batch list) → set `apify_api_API_KEY`. 
   * (Alt per-URL) **Transcribe video from url 1** with batching size 1; replace `apify_api_(API KEY)`. 
3. **Combine transcripts + prior data**

   * **Merge 2** (combine by position) merges transcripts with your earlier merged rows. 
4. **Write the final rows**

   * **Google Sheets1** (set **Document ID** + target tab). This holds the full dataset (metadata + stats + transcript). 
   * (Optional) **Code → Google Docs** to push a nicely formatted transcript document. 

## E. Chat-driven append (optional parallel path)

* **When chat message received → OpenAI Chat Model → Basic LLM Chain → “Real time script gets added here”** (append to a sheet you choose). Set OpenAI credential and the Sheet’s **Document ID / tab**. 

## F. Run it

* Click **Test workflow** to run the scraping/transcript→Sheets path end-to-end. Use the chat trigger if you want to send prompts that get appended to a different sheet. 

