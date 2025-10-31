# 🏥 MediQuery — AI-Powered Medical Analytics & Insights Platform

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.68+-green.svg)](https://fastapi.tiangolo.com)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.0+-red.svg)](https://streamlit.io)
[![MindsDB](https://img.shields.io/badge/MindsDB-Latest-orange.svg)](https://mindsdb.com)

MediQuery is an intelligent healthcare analytics copilot that transforms natural language questions into actionable medical insights. Built on MindsDB's AI infrastructure, it enables hospitals, clinics, and researchers to analyze patient feedback, satisfaction surveys, and treatment effectiveness without writing complex SQL queries.

## 🌟 Why MediQuery?

Traditional healthcare analytics require technical expertise and manual data processing. MediQuery democratizes medical data analysis by allowing anyone to ask questions like:

- *"What is the average patient satisfaction by department?"*
- *"Which departments have the highest wait times and lowest ratings?"*
- *"Show me reviews mentioning excellent doctor communication"*

The platform instantly transforms these questions into SQL queries, analyzes your data, and presents insights through an intuitive web interface.

## ✨ Key Features

- **🤖 Conversational AI Agent** — Natural language to SQL query transformation
- **🧠 Semantic Knowledge Base** — Intelligent search through patient feedback with context understanding
- **📊 Real-time Analytics** — Live insights from patient satisfaction surveys and reviews
- **🔍 Metadata-Aware Filtering** — Filter by department, patient type, date ranges, and more
- **⚡ Zero-ETL Architecture** — Works directly with live data sources (Google Sheets, databases)
- **🎯 Interactive Dashboard** — User-friendly Streamlit interface with charts and visualizations
- **🔌 RESTful API** — FastAPI backend for easy integration with existing systems

## 🏗️ Architecture

```
┌──────────────────────────┐
│  Data Sources            │
│  (Google Sheets/DB)      │
└───────────┬──────────────┘
            │ CREATE DATABASE
┌───────────▼──────────────┐
│  MindsDB Knowledge Base  │
│  (Semantic Search)       │
└───────────┬──────────────┘
            │
┌───────────▼──────────────┐
│  MindsDB AI Agent        │
│  (NL → SQL)              │
└───────────┬──────────────┘
            │ REST API
┌───────────▼──────────────┐
│ FastAPI Backend          │
│ (/ask_agent, /kb_search) │
└───────────┬──────────────┘
            │
┌───────────▼──────────────┐
│ Streamlit Frontend       │
│ (Interactive Dashboard)  │
└──────────────────────────┘
```

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- MindsDB instance (local or cloud)
- Git

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/medquery.git
cd medquery
```

### 2. Set Up Virtual Environment

```bash
python -m venv .venv

# On Windows
.venv\Scripts\activate

# On macOS/Linux
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
# Backend dependencies
cd backend
pip install -r requirements.txt

# Frontend dependencies
cd ../frontend
pip install -r requirements.txt
```

### 4. Configure Environment

```bash
# Copy and edit backend environment file
cd ../backend
cp .env .env.local
```

Edit `.env.local` with your MindsDB configuration:

```env
MINDSDB_URL=http://localhost:47334
MINDSDB_API_KEY=your_api_key_if_needed
```

### 5. Start the Services

**Terminal 1 - Backend:**
```bash
cd backend
uvicorn main:app --reload
```

**Terminal 2 - Frontend:**
```bash
cd frontend
streamlit run app.py
```

### 6. Access the Application

- 🖥️ **Frontend Dashboard**: http://localhost:8501
- 🔧 **Backend API**: http://localhost:8000
- 📚 **API Documentation**: http://localhost:8000/docs

## 📋 MindsDB Setup

### 1. Create Data Source

Connect your patient feedback data (Google Sheets example):

```sql
CREATE DATABASE patient_feedback_source
WITH ENGINE = 'sheets',
PARAMETERS = {
    "spreadsheet_id": "your_google_sheet_id",
    "sheet_name": "patient_reviews"
};
```

### 2. Create Knowledge Base

Set up semantic search capabilities:

```sql
CREATE KNOWLEDGE_BASE patient_reviews_kb
USING
    embedding_model = {
        "provider": "google",
        "model_name": "text-embedding-004"
    },
    reranking_model = {
        "provider": "google", 
        "model_name": "gemini-2.5-flash"
    },
    metadata_columns = [
        "patient_id", "department", "visit_date", 
        "patient_type", "overall_rating", "doctor_communication",
        "cleanliness", "wait_time", "treatment_effectiveness",
        "recommendation", "verified"
    ],
    content_columns = ["review_text"],
    id_column = "review_id";
```

### 3. Create AI Agent

Configure the conversational analytics agent:

```sql
CREATE AGENT department_analysis
USING
    model = {
        "provider": "google",
        "model_name": "gemini-2.5-flash"
    },
    data = {
        "tables": ["mindsdb.patient_reviews_kb"]
    },
    prompt_template = 'You have access to patient_reviews_kb with columns: department, overall_rating, treatment_effectiveness, doctor_communication, wait_time, recommendation. Analyze satisfaction trends, detect concerns, and summarize outcomes by department.';
```

### 4. Set Up Auto-Sync (Optional)

Keep your knowledge base updated automatically:

```sql
CREATE JOB sync_patient_reviews (
    INSERT INTO patient_reviews_kb
    SELECT * FROM patient_feedback_source.patient_reviews
    WHERE review_id NOT IN (
        SELECT id FROM patient_reviews_kb
    )
) EVERY 6 hours;
```

## 💡 Usage Examples

### Agent Queries

Ask natural language questions:

```sql
SELECT answer FROM department_analysis 
WHERE question = 'What is the average patient satisfaction by department?';
```

### Knowledge Base Search

Perform semantic searches:

```sql
SELECT review_text, department, similarity_score
FROM mindsdb.patient_reviews_kb 
WHERE content = 'excellent doctor communication and clean facilities'
AND department = 'Cardiology'
LIMIT 10;
```

## 🛠️ API Reference

### POST `/ask_agent`

Query the AI agent with natural language:

```json
{
    "agent_name": "department_analysis",
    "question": "What is the average patient satisfaction by department?"
}
```

### POST `/kb_search`

Perform semantic search on the knowledge base:

```json
{
    "kb_full_name": "project.patient_reviews_kb",
    "query_text": "excellent doctor communication",
    "limit": 10,
    "metadata_filters": {
        "department": "Cardiology"
    }
}
```

### GET `/health`

Check service health and configuration:

```json
{
    "status": "ok",
    "mindsdb_url": "http://localhost:47334"
}
```

## 📊 Sample Data Schema

Your patient feedback data should include these columns:

| Column | Type | Description |
|--------|------|-------------|
| `review_id` | String | Unique identifier |
| `patient_id` | String | Patient identifier |
| `department` | String | Hospital department |
| `visit_date` | Date | Visit date |
| `overall_rating` | Integer | 1-5 rating scale |
| `doctor_communication` | Integer | 1-5 rating |
| `cleanliness` | Integer | 1-5 rating |
| `wait_time` | Integer | Minutes waited |
| `treatment_effectiveness` | Integer | 1-5 rating |
| `review_text` | Text | Free-form feedback |
| `recommendation` | Boolean | Would recommend |

## 🔧 Tech Stack

| Layer | Technology |
|-------|------------|
| **AI Engine** | MindsDB Agents + Knowledge Bases |
| **LLM** | Google Gemini 2.5 Flash |
| **Embeddings** | Google text-embedding-004 |
| **Backend** | FastAPI + Python |
| **Frontend** | Streamlit |
| **Data Sources** | Google Sheets, PostgreSQL, MySQL |
| **Deployment** | Docker, Uvicorn |

## 🚢 Deployment

### Docker Deployment

```bash
# Build and run with Docker Compose
docker-compose up --build
```

### Cloud Deployment

Deploy to your preferred cloud platform:

- **Heroku**: Use the included `Procfile`
- **AWS**: Deploy with ECS or Lambda
- **Google Cloud**: Use Cloud Run
- **Azure**: Deploy with Container Instances

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- 📖 **Documentation**: [Wiki](https://github.com/yourusername/medquery/wiki)
- 🐛 **Bug Reports**: [Issues](https://github.com/yourusername/medquery/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/yourusername/medquery/discussions)
- 📧 **Email**: support@medquery.com

## 🔮 Roadmap

- [ ] **Multi-Agent Collaboration** — Complex medical insights through agent coordination
- [ ] **Advanced Analytics Dashboard** — Trend analysis and correlation detection  
- [ ] **Real-time Alerts** — Automated notifications for satisfaction drops
- [ ] **Mobile App** — iOS/Android companion app
- [ ] **Integration APIs** — Connect with EMR systems (Epic, Cerner)
- [ ] **Multi-language Support** — Analyze feedback in multiple languages
- [ ] **Predictive Analytics** — ML models for satisfaction prediction

## 🙏 Acknowledgments

- [MindsDB](https://mindsdb.com) for the AI infrastructure
- [FastAPI](https://fastapi.tiangolo.com) for the excellent web framework
- [Streamlit](https://streamlit.io) for the rapid UI development
- The healthcare analytics community for inspiration and feedback

---

**Made with ❤️ for better healthcare analytics**