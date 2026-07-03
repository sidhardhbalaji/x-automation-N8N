# Sid's X Content Engine

A fully automated, AI-driven content pipeline built on **n8n**. This workflow acts as your elite, autonomous social media team: discovering trending tech news, scoring it for virality, writing targeted X (Twitter) posts, self-correcting grammar, generating images, asking for your final approval via Telegram, and publishing it directly to your account.

## ✨ Features

- **Multi-Source Data Ingestion**: Automatically pulls and merges the latest articles from TechCrunch, Hacker News, Google News, OpenAI Blog, and Reddit (AI & SaaS).
- **Optional Xquik Source Feed**: Add a reviewed TweetClaw or Xquik export as another public X/Twitter source before virality scoring, using fields such as `url`, `text`, `author`, `source`, `checkedAt`, and `reason`.
- **AI Virality Scoring**: Uses OpenAI (`gpt-4o`) to score every discovered story (1-100) based on virality, novelty, and relevance to tech founders, passing only the top 3 forward.
- **Strict Deduplication**: Connects to a PostgreSQL database to ensure you never post about the same URL twice.
- **AI Copywriting Agent**: Generates 3 distinct X post variations under 280 characters for the winning story:
  - Variation A: The "Founder" angle.
  - Variation B: The "Contrarian" angle.
  - Variation C: The "Educational" angle.
- **AI Quality Control Agent**: A secondary editor agent reviews the generated copy for clarity, grammar, and generic AI jargon (like "delve" or "revolutionize").
- **Dynamic Image Generation**: Attempts to extract the article's OpenGraph (OG) image. If missing, it uses DALL-E 3 to generate a custom, high-quality corporate tech graphic.
- **Human-in-the-Loop Telegram Approval**: Pauses the workflow and sends the 3 variations to your phone via Telegram. Click interactive buttons to approve, reject, or regenerate the post.
- **Automated Publishing & Logging**: Uploads the image and text to X (Twitter) and logs the success and analytics into the PostgreSQL database.

## 🛠️ Prerequisites

To run this workflow, you need:
1. An **n8n** instance (must be accessible via a public tunnel for Telegram webhooks to function).
2. A **PostgreSQL** database.
3. An **OpenAI API Key**.
4. A **Telegram Bot Token** and your **Chat ID**.
5. **X (Twitter) OAuth2 Credentials**.

Keep any TweetClaw or Xquik source export separate from n8n credentials. Import only public content fields into the scoring step, then keep Telegram approval as the gate before publishing.

## 🚀 Setup Instructions

### 1. Database Setup
You will need a PostgreSQL database. Execute the following SQL commands to set up the required tables:

```sql
CREATE TABLE IF NOT EXISTS posted_content (
    id SERIAL PRIMARY KEY,
    url TEXT UNIQUE NOT NULL,
    headline TEXT NOT NULL,
    summary TEXT,
    generated_post TEXT,
    x_post_id TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS workflow_activity (
    id SERIAL PRIMARY KEY,
    execution_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    discovery_count INTEGER,
    selected_stories JSONB,
    approved_posts JSONB,
    published_posts JSONB,
    errors JSONB
);
```

### 1. Import Workflow
The safest and most reliable way to install this workflow is to download the JSON file and import it directly into n8n.

1. **Download the file**: Click here to download [**`x_content_engine_workflow.json`**](https://raw.githubusercontent.com/sidhardhbalaji/x-automation-N8N/main/workflows/x_content_engine_workflow.json) (Right-click the page and select "Save As...").
2. Open your n8n dashboard and go to your Workflows.
3. Click the **Add Workflow** button in the top right.
4. Click the **`...` (Options)** menu in the top right of the canvas.
5. Select **Import from File...** and choose the JSON file you just downloaded.
6. The entire workflow will instantly populate!

### 2. Database Setup
Once imported, you will need to open the nodes that have missing credentials and create them:
- **OpenAI API**: Enter your OpenAI API key.
- **Postgres DB**: Enter your database host (e.g., `localhost`), user, password, and database name.
- **Telegram Bot**: Provide your bot token. Update the `Chat ID` in the "Telegram Approval Form" node to your personal Telegram Chat ID.
- **Twitter OAuth2 API**: Connect your X developer account credentials.

### 4. Enable Public Tunnel
For the Telegram "Wait for Approval" node to work, your n8n instance *must* have a public internet URL (Telegram cannot send button clicks to `localhost`). 

Start n8n using a tunnel service like Pinggy or n8n's built in tunnel:
```bash
# Example using Pinggy
ssh -p 443 -R0:localhost:5678 a.pinggy.io

# Then start n8n with the generated URL
WEBHOOK_URL="https://your-pinggy-url.pinggy.link" npx n8n start
```

### 5. Test & Activate
- Click **Test Workflow** to run the pipeline manually.
- Check your Telegram for the approval buttons.
- Once verified, toggle the workflow to **Active** in the top right corner of n8n.

## 🤝 Contributing
Feel free to fork this project and add new data sources, agentic workflows, or different output channels (like LinkedIn or a Newsletter).
