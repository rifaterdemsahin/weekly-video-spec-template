# weekly-video-spec-template
Weekly video spec template

## 🔗 Live Prompt Builder
👉 **https://rifaterdemsahin.github.io/weekly-video-spec-template/**

🎬 A form-based tool that helps you build the Claude Code prompt from [`prompt.md`](./prompt.md) without hand-editing the raw `VARIABLES` block.

- 📝 Fill in fields like video slug, video ID, title, hypothesis, GitHub owner, template repo, and your Azure Key Vault name
- 🗄️🔐 One **shared** Supabase database is reused across every weekly video (rows scoped by `VIDEO_ID`) — credentials are fetched from Azure Key Vault at scaffold time, never typed into this form
- ✨ Click **Generate Prompt** to produce the full, ready-to-paste prompt
- 📋 **Copy to Clipboard** or ⬇️ **Download** it as a `.md` file
- 🚀 Paste the result into Claude Code from an empty parent directory to scaffold a new weekly video pre-production repo end-to-end

## ✅ Sanity Check
🎥 See this style in action — a sample video produced with this exact template: **https://www.youtube.com/watch?v=fJjxujMASkQ**

Use it to confirm your scaffolded repo (research → arguments → script → design → previsualisation → assets → todo) is on track to produce the same kind of end result. 🧪👍
