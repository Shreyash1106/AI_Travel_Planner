# ✈️ AI Travel Booking System — Multi-Agent LangGraph

A **Multi-Agent AI System** built with LangGraph that plans complete travel itineraries using 4 specialized AI agents working in a pipeline. It searches real flights, finds hotels, builds day-by-day itineraries, and delivers a final travel plan — all powered by **Groq LLaMA 3.3 70B**.

---

## 🖥️ Demo

![AI Travel Booking System](https://images.unsplash.com/photo-1436491865332-7a61a109cc05?w=1200&q=80)

> Enter a travel query like _"7-day Japan trip under ₹2 Lakhs"_ and 4 AI agents instantly build your complete travel plan.

---

## 🏗️ Architecture

```
User Query
    │
    ▼
┌─────────────────┐
│  Flight Agent   │  ── AviationStack API → Real flight data
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Hotel Agent   │  ── Tavily Search → Hotel recommendations
└────────┬────────┘
         │
         ▼
┌──────────────────────┐
│   Itinerary Agent    │  ── Groq LLaMA 3.3 70B → Day-by-day itinerary
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    Final Agent       │  ── Groq LLaMA 3.3 70B → Complete travel plan
└──────────┬───────────┘
           │
           ▼
  📄 Final Travel Plan
  (Displayed + Auto-saved as .md)
```

**Memory:** PostgreSQL (via LangGraph `PostgresSaver`) — conversation history persists across sessions per User ID.

---

## 🚀 Features

- 🤖 **4 Specialized AI Agents** — Flight, Hotel, Itinerary, Final
- ✈️ **Real Flight Data** — AviationStack API integration
- 🔍 **Live Hotel Search** — Tavily web search
- 🧠 **Long-Term Memory** — PostgreSQL checkpointing per user session
- 💾 **Auto-Save Plans** — Every travel plan saved as `.md` file in `travel_plans/`
- ⬇️ **Download Plan** — One-click download from UI
- 🎨 **Dark UI** — Beautiful Streamlit frontend with custom CSS
- ⚡ **Fast Inference** — Groq LLaMA 3.3 70B (free tier available)

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Agent Framework | LangGraph |
| LLM | Groq · LLaMA 3.3 70B Versatile |
| Flight Data | AviationStack API |
| Web Search | Tavily Search API |
| Memory / Checkpointing | PostgreSQL + psycopg |
| Frontend | Streamlit |
| Language | Python 3.10+ |

---

## 📁 Project Structure

```
AI_Travel_Planner/
│
├── main.py               # Core logic — agents, graph, PostgreSQL setup
├── frontend.py           # Streamlit UI
│
├── tools/
│   ├── flight_tool.py    # AviationStack flight search
│   └── tavily_tool.py    # Tavily hotel/web search
│
├── travel_plans/         # Auto-saved travel plans (.md files)
│
├── .env                  # API keys (not committed)
├── requirements.txt      # Python dependencies
└── README.md
```

---

## ⚙️ Setup & Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/AI_Travel_Planner.git
cd AI_Travel_Planner
```

### 2. Create a virtual environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Mac/Linux
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

Or manually:

```bash
pip install langgraph langchain langchain-groq langchain-community langchain-tavily psycopg[binary] psycopg_pool python-dotenv tavily-python requests streamlit
```

### 4. Setup PostgreSQL

Install PostgreSQL and create the database:

```sql
CREATE DATABASE langgraph_memory;
```

Or open **pgAdmin 4** → right-click Databases → Create → name it `langgraph_memory`.

### 5. Configure environment variables

Create a `.env` file in the root directory:

```env
GROQ_API_KEY=your_groq_api_key_here
TAVILY_API_KEY=your_tavily_api_key_here
AVIATIONSTACK_API_KEY=your_aviationstack_api_key_here
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/langgraph_memory
```

### 6. Run the app

```bash
streamlit run frontend.py
```

---

## 🔑 API Keys — Where to Get Them

| API | Free Tier | Link |
|---|---|---|
| **Groq** | ✅ Yes (fast + generous) | https://console.groq.com |
| **Tavily** | ✅ Yes (1000 req/month) | https://tavily.com |
| **AviationStack** | ✅ Yes (500 req/month) | https://aviationstack.com |

---

## 💡 How to Use

1. Open the app in browser (`http://localhost:8501`)
2. Enter your **User ID** in the sidebar (for memory persistence)
3. Type your travel query OR click a quick destination chip
4. Click **🚀 Generate My Travel Plan**
5. Watch 4 agents work in real-time in the pipeline
6. Download your complete plan as a `.md` file

**Example queries:**
- `Plan a 7-day Japan trip under ₹2 Lakhs`
- `5-day Paris trip for 2 people`
- `Budget backpacking Bali for 10 days`
- `Dubai weekend trip from Mumbai`

---

## 🧠 Agent Details

### ① Flight Agent
- Calls AviationStack API with the user query
- Returns airline name, departure/arrival airports, flight status
- Adds result to shared `TravelState`

### ② Hotel Agent
- Calls Tavily Search for hotel recommendations
- Returns top 5 results with title, URL, and snippet
- Results trimmed to 300 chars to avoid wall-of-text

### ③ Itinerary Agent
- Receives flight + hotel data from state
- Calls Groq LLaMA 3.3 70B to generate a day-by-day itinerary
- Includes budget breakdown per category

### ④ Final Agent
- Combines all results (flights + hotels + itinerary)
- Generates a polished, complete travel response
- This is what gets displayed as the **Final Travel Plan**

---

## 🗄️ Memory Architecture

Each user gets a unique `thread_id` (User ID from sidebar). LangGraph's `PostgresSaver` stores the full conversation state in PostgreSQL, so:

- Previous travel queries are remembered
- Context builds across multiple sessions
- Different users have completely isolated memory

```python
# autocommit=True required for PostgresSaver.setup()
# CREATE INDEX CONCURRENTLY cannot run inside a transaction block
_conn = psycopg.connect(DATABASE_URL, autocommit=True)
checkpointer = PostgresSaver(_conn)
checkpointer.setup()

app = graph.compile(checkpointer=checkpointer)
```

---

## 📄 Sample Output

```
✈️ Flight Information
Airline: IndiGo | Departure: Delhi | Arrival: Tokyo Narita | Status: scheduled

🏨 Hotel Information
1. Budget Hotels in Tokyo from ₹2,000/night
2. Kyoto Guesthouses — best value picks 2026

🗓️ Itinerary
Day 1 — Arrival in Tokyo, Shibuya Crossing, Ramen dinner (₹400)
Day 2 — Tokyo Tower, Asakusa Temple, Sushi lunch (₹800)
...

🧠 Final Travel Plan
Total Estimated Budget: ₹1,75,000 / ₹2,00,000
LLM Calls: 4
```

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/new-agent`)
3. Commit your changes (`git commit -m 'Add visa checker agent'`)
4. Push to the branch (`git push origin feature/new-agent`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Shreyash Dandekar**
- GitHub: [@Shreyashdandekar1106](https://github.com/Shreyashdandekar1106)

---

⭐ **If this project helped you, please give it a star!**
