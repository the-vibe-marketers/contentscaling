
# YouTube Trend Analysis & Idea Generation (n8n Workflow)

This workflow automates **marketing research and ideation** using YouTube data and AI models.  
It finds trending videos for a given keyword, analyzes their content to uncover popular themes, and generates new video ideas — all saved in Google Sheets for easy reference.


## 🧠 What This Workflow Does

1. **Find Trending Videos:**  
   Uses the YouTube Data API to search for recent or trending videos based on your keyword.

2. **Extract Video Info:**  
   Fetches title, description, view count, and channel info for each video.

3. **Store Raw Data:**  
   Saves video details (title, views, link, description, channel) into a Google Sheet.

4. **Analyze Content (AI):**  
   Uses an LLM (OpenAI or OpenRouter) to extract themes, hooks, and content angles from each title and description.

5. **Generate Ideas (AI):**  
   Based on the analyzed themes, generates new YouTube video ideas.

6. **Store Results:**  
   Appends or updates the AI-generated content ideas in Google Sheets, linked to each original video.



## ⚙️ Step-by-Step Breakdown

### **1. Search YouTube**
**Node Type:** `HTTP Request (GET)`

**URL Template:**
```

[https://www.googleapis.com/youtube/v3/search?part=snippet&q=YOUR_KEYWORD&type=video&order=date&maxResults=10&key=YOUR_YOUTUBE_API_KEY](https://www.googleapis.com/youtube/v3/search?part=snippet&q=YOUR_KEYWORD&type=video&order=date&maxResults=10&key=YOUR_YOUTUBE_API_KEY)

```

**Replace:**
- `YOUR_KEYWORD` → the search keyword (e.g., “AI marketing”)
- `10` → number of videos to fetch
- `YOUR_YOUTUBE_API_KEY` → your YouTube Data API key

**Output:**  
A list of video items with basic metadata (title, description, videoId, channel).

---

### **2. Get Video Stats & Channel Info**
**Node Type:** `HTTP Request (GET)` inside a loop or using `SplitInBatches`  

**URL Template:**
```

[https://www.googleapis.com/youtube/v3/videos?part=snippet,statistics&id=VIDEO_ID_LIST&key=YOUR_YOUTUBE_API_KEY](https://www.googleapis.com/youtube/v3/videos?part=snippet,statistics&id=VIDEO_ID_LIST&key=YOUR_YOUTUBE_API_KEY)

```

**Input:** Extract `videoId` from the search results.

**Output:**  
Detailed data for each video, including:
- `views`
- `likes`
- `comments`
- `channelTitle`

---

### **3. Store Video Info in Google Sheets**
**Nodes:**
- `Set` or `Code`: structure fields → `title`, `views`, `link`, `description`, `channel`
- `Google Sheets`: operation `Append` or `Update`

**Mapped Columns Example:**
| Title | Views | Link | Description | Channel |
|--------|--------|------|-------------|----------|

**Result:** A database of trending videos.

---

### **4. Analyze Titles + Descriptions**
**Node Type:** `Loop Over Items` + `OpenAI Chat Model` or `OpenRouter Chat Model`

**Prompt Example:**
```

Analyze the following video:
Title: "{{$json.title}}"
Description: "{{$json.description}}"

Extract 2-3 key content angles or themes that make it engaging. Categorize the topic.

````

**Output Example:**
```json
{
  "angle_1": "Automation in marketing",
  "angle_2": "Using AI tools for scaling",
  "category": "Marketing Tech"
}
````

---

### **5. Store Analysis**

**Node Type:** `Google Sheets (Update)` inside the same loop.

Map:

* `Angle 1`
* `Angle 2`
* `Category`

Result: your sheet gets enriched with qualitative insights.

---

### **6. Generate New Ideas**

**Node Type:** another `Loop Over Items` + `OpenAI Chat Model` or `OpenRouter Chat Model`

**Prompt Example:**

```
Based on the content angle "{{$json.angle_1}}", 
suggest 3 creative YouTube video ideas that could perform well.
Each idea should have:
- A title
- A hook
- A format suggestion (e.g., tutorial, case study, reaction)
```

**Output Example:**

```json
{
  "idea_1": "How Marketers Use AI Agents to Double Output",
  "idea_2": "Scaling Content with n8n: Real Workflow Demo",
  "idea_3": "AI Tools Every Agency Should Master in 2025"
}
```

---

### **7. Store Final Ideas**

**Node Type:** `Google Sheets (Append)` or `Google Sheets (Update)`

| Original Title | Angle      | AI-Generated Ideas                      |
| -------------- | ---------- | --------------------------------------- |
| “AI in Ads”    | Automation | “How to Automate Facebook Ads Using AI” |

---

## 🔌 Connecting Your APIs in n8n

### **1. YouTube Data API v3**

**How to Get Access:**

1. Go to [Google Cloud Console](https://console.cloud.google.com/).
2. Create/select a project.
3. Navigate to **APIs & Services → Library** → enable **YouTube Data API v3**.
4. Go to **APIs & Services → Credentials** → click **Create Credentials → API key**.
5. Copy the generated key and restrict it to your domain/IPs.

**In n8n:**
Paste the API key directly into your HTTP Request node URL parameter (`key=YOUR_API_KEY`).

---

### **2. OpenAI**

**How to Connect:**

1. Sign in to [OpenAI Platform](https://platform.openai.com/).
2. Copy your API key.
3. In n8n → **Credentials → OpenAI API**, paste the key.
4. In AI nodes, choose `gpt-4o-mini`, `gpt-4`, or `gpt-3.5-turbo`.

---

### **3. OpenRouter (Alternative to OpenAI)**

**How to Connect:**

1. Go to [OpenRouter](https://openrouter.ai/).
2. Get your API key from the dashboard.
3. In n8n → **Credentials → OpenRouter API**, paste the key.
4. Select models like `anthropic/claude-3.5-sonnet` or `perplexity/sonar`.

---

### **4. Google Sheets**

**How to Connect:**

1. Enable Google Sheets API in the Cloud Console.
2. Create **OAuth 2.0 Client ID** credentials.
3. Add n8n’s redirect URI:

   ```
   https://<your-n8n-domain>/rest/oauth2-credential/callback
   ```
4. In n8n → **Credentials → Google Sheets OAuth2 API**, paste the Client ID and Secret.
5. Authorize and select your Google account.

---

## 🧩 Optional Improvements

* Add a **“Top Performers”** filter (views > 100K) before AI analysis.
* Include an **AI summarizer** node to extract a quick summary from each transcript.
* Connect to **Google Docs** or **Slack** to share results automatically.
* Schedule with a **Cron Trigger** to run weekly.



**Result:**
A dynamic research and ideation pipeline where you automatically:

* Discover what’s trending on YouTube,
* Understand *why* it’s trending, and
* Generate fresh ideas ready for your content calendar.

---

```
```
