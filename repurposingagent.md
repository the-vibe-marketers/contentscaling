🧩 Repurposing Agent

**Purpose:**
This workflow automates **social content ideation → research → LinkedIn post creation → AI image generation → Google Doc saving → human review → LinkedIn publishing**.

It brings together:

* YouTube & X (Twitter) scraping via **Apify**
* Multiple **AI agents (OpenAI, Perplexity, Claude)** for ideation, research, writing, and prompt generation
* **Google Docs** for storing final content
* **Slack** for review approval
* **LinkedIn** posting automation

---

## ⚙️ Step 1: Prerequisites

Before you import:

1. ✅ Have an **n8n instance** (self-hosted or cloud).
2. ✅ Create API tokens:

   * **Apify API Key**
   * **OpenAI API Key**
   * **OpenRouter API Key**
   * **Google OAuth2 Credential**
   * **Slack OAuth Credential**
   * **LinkedIn OAuth Credential**
3. ✅ Create a **Google Drive Folder** named “Linkedin posts” (or use the ID already in the JSON).
4. ✅ Ensure your Slack app can post messages & wait for approval (via `sendAndWait` action).

---

## 🧠 Step 2: Trigger – Start via Chat Input

**Node:** `When chat message received`

* Triggers the workflow when you send a message (can be in n8n chat, or custom chat trigger).
* The message is split by the **Split Out** node to process the keyword or query separately.

---

## 🎥 Step 3: YouTube Data Collection

**Nodes:**

* `HTTP Request` → searches YouTube for videos with keyword `"n8n"` using Apify.
* `Loop Over Items` → processes each video result.
* `HTTP Request2` → fetches the transcript for each video.
* `Code` → joins all transcript text into a single “full_caption” field.

📊 Output:
Each YouTube video now has a full English transcript ready for text analysis.

---

## 🐦 Step 4: Twitter Data Collection

**Nodes:**

* `HTTP Request3` → scrapes a Twitter account’s latest 10 posts (Apify’s `danek~twitter-scraper-ppr`).
* `Code1` → filters tweets from the past 10 days and removes replies.

📊 Output:
A list of recent original tweets for content analysis.

---

## 🔗 Step 5: Combine Video + Tweet Data

**Node:** `Merge5`

* Combines YouTube transcripts (from Code) and Twitter posts (from Code1).
* Ensures the `full_caption` and `text` fields are aligned by position for the next step.

---

## 💡 Step 6: Generate Content Ideas

**Node:** `Content idea generator`

* Uses **OpenAI (Grok model)** to analyze the transcripts and tweets.
* Prompt focuses on **marketing-focused ideas** for non-technical marketers.
* Outputs:

  * Title
  * Hook
  * Format (video/thread/one-pager)
  * Angle

🧠 Memory: Not persisted (single pass).
💬 Model: `gpt-4o-mini` via OpenAI Chat Model1.

---

## 🔍 Step 7: Deep Research Agent

**Node:** `Research agent`

* Uses **Perplexity Sonar** via OpenRouter.
* Conducts deep research around:

  * Trends, insights, case studies, and expert takes.
  * Focused on how **marketers can scale marketing using AI with n8n**.
* Structured output with sections like:

  * Topic
  * Key Insights
  * Expert Takes
  * Supporting Data
  * Citations

🧠 Memory: `Simple Memory` (session key `content_idea`).

---

## 🧾 Step 8: Convert Research to Text for Docs

**Node:** `Code2`

* Merges all research output into formatted Markdown-like structure:

  ```
  # Research
  ## [Section]
  - **key insight**: ...
  - **citations**: ...
  ```
* Returns `document_name` and `content` for Google Docs.

---

## 🧩 Step 9: Merge Research + Content Ideas

**Node:** `Merge1`

* Combines research text and content ideas for use by the post-generation agent.

---

## ✍️ Step 10: Generate LinkedIn Post

**Node:** `Linkedin post generating agent`

* Uses **Claude 3.7 Sonnet (via OpenRouter)**.
* Prompt enforces:

  * 400-word post
  * Brand voice (authentic, expert, conversational)
  * Format: Title + Content only
  * No hashtags/emojis
  * Must sound like a professional insight-sharing post

🧠 Memory: `Simple Memory1` (session key `perplexity_research`).

---

## 🪶 Step 11: Format LinkedIn Output

**Node:** `Linkedin formatted output1`

* Cleans up Markdown headers, removes unnecessary tags, and formats paragraphs.
* Output field: `linkedin_post`.

---

## 📄 Step 12: Create Google Doc Draft

**Node:** `Linkedin doc creation1`

* Creates a new Google Doc titled “Linkedin posts” in the specified Drive folder.
* Document ID is passed downstream.

---

## 🧩 Step 13: Merge Content + Doc ID

**Node:** `Merge8`

* Combines `linkedin_post` with the Google Doc info.

---

## 📝 Step 14: Write LinkedIn Content into Doc

**Node:** `Linkedin content doc1`

* Inserts the LinkedIn post text into the newly created Google Doc.

---

## 👀 Step 15: Human Review (Slack)

**Node:** `Human in the loop1`

* Sends a message to Slack channel `slacktrigger_n8n_blog`:

  ```
  Here is the post - https://docs.google.com/document/d/{{documentId}}/edit
  ```
* Waits for approval (double confirm).

---

## ✅ Step 16: Approval Check

**Node:** `If2`

* Checks whether the Slack message was **approved** (boolean true).
* If approved → continues to image + publishing pipeline.

---

## 🖼️ Step 17: Generate Image Prompts

**Node:** `Image prompt generator`

* Uses **OpenAI (gpt-4o-mini)**.
* Generates 1–2 vivid image prompts (under 20 words) from post text, e.g.:

  ```
  {"image_prompt": "marketer automating workflows using AI agents", "style": "flat vector", "format": "wide"}
  ```

🧠 Memory: `Simple Memory2` (session key `linkedin_post`).

---

## 🎨 Step 18: Create Image via OpenAI DALL·E

**Node:** `Linkedin image generation using openAI1`

* Calls OpenAI Images API (`gpt-image-1`) using the prompt.
* Generates one `1536x1024` image.

---

## 🗂️ Step 19: Convert Image Output

**Node:** `Convert to File2`

* Converts the base64 JSON image data into a binary image file for upload.

---

## 🪄 Step 20: Final Merge Before Posting

**Node:** `Merge9`

* Combines:

  1. Approved Slack output
  2. LinkedIn post text
  3. Image binary

---

## 📢 Step 21: Post on LinkedIn

**Node:** `Post on linkedin1`

* Publishes the final post to your LinkedIn account (person ID `PnQwcsps5V`).
* Shares both the post text and the generated image.

---

## 🧩 Step 22: (Optional) Memories & Models Summary

| Purpose       | Node                             | Model               | Memory Key            |
| ------------- | -------------------------------- | ------------------- | --------------------- |
| Content ideas | `Content idea generator`         | `gpt-4o-mini`       | —                     |
| Research      | `Research agent`                 | `perplexity/sonar`  | `content_idea`        |
| Post writing  | `Linkedin post generating agent` | `claude-3.7-sonnet` | `perplexity_research` |
| Image prompt  | `Image prompt generator`         | `gpt-4o-mini`       | `linkedin_post`       |

---

## 🚀 Step 23: To Run It

1. **Import JSON** into n8n.
2. **Set credentials** for:

   * Apify
   * OpenAI
   * OpenRouter
   * Google OAuth
   * Slack
   * LinkedIn
3. **Replace placeholder tokens:**

   * `apify_api_YOUR_API_KEY` (in all 3 Apify URLs)
4. **Activate workflow.**
5. **Send a chat message** (via Chat Trigger) to start the process.
6. Watch:

   * Docs get created in Google Drive
   * Slack approval message appear
   * Once approved → post goes live on LinkedIn with AI-generated image.

---

## 💡 Optional Improvements

* Add error-handling around Apify HTTP nodes (`onError: continueRegularOutput`).
* Store all generated docs’ links in a Google Sheet for content tracking.
* Add a "Re-run for edits" Slack command for rejected posts.



