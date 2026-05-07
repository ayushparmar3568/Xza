# AI-Based Social Issue Detection (AI Incident Manager)

## 📌 Project Overview
This project is an **AI-powered Incident Management System** designed to automatically ingest, analyze, and prioritize social media posts or user-submitted text regarding civic issues, emergencies, and online harassment. 

By combining Natural Language Processing (NLP) and rule-based risk engines, the application reads the unstructured text of a post and determines:
1. **What is the post about?** (Topic)
2. **How is the user feeling?** (Emotion)
3. **Is this a real emergency or just an exaggeration?** (Context Filters)
4. **How urgently should authorities respond?** (Severity/Priority)

---

## 🛠️ Technology Stack
* **Frontend:** React.js, Vite, Axios, React Router (Port 5173)
* **Backend:** Node.js, Express.js, Mongoose (Port 5001)
* **Database:** MongoDB (Local instance at Port 27017)
* **AI/Machine Learning:** Hugging Face Inference API for advanced NLP (Zero-Shot Classification and Emotion Detection)

---

## ⚙️ How It Works: The 4-Phase Pipeline

When a post is submitted to the system (either individually or via a bulk dataset upload), it goes through a master orchestrator (`analysisService.js`) which executes the following 4-phase pipeline:

### Phase 1: Preprocessing (`preprocessor.js`)
* **Text Cleaning:** The raw text is stripped of unnecessary characters, URLs, and special symbols to prepare it for AI analysis.
* **Tokenization:** The text is broken down into readable tokens (words).

### Phase 2: Dual Engine Assessment
Two powerful AI engines run in parallel to analyze the core meaning of the post:
1. **Topic Classifier (`topicClassifier.js`):** 
   * Uses AI (Zero-Shot Classification) to categorize the post into one of six predefined categories: *Civic Issue, Crime / Violence, Self-Harm / Depression, Cyberbullying / Harassment, Emergency / Disaster, or General Complaint.*
2. **Emotion Scorer (`emotionScorer.js`):**
   * Uses AI models to detect the dominant emotion in the text (Anger, Fear, Sadness, Despair, Disgust, Joy, Surprise, Neutral).
   * Generates a "Volatility Flag" if multiple intense negative emotions are detected simultaneously.

### Phase 3: Filter Layer / Context Validation (`filterLayer.js`)
This layer looks for specific nuances to prevent false alarms:
* **Hyperbole Detection:** Checks if the user is exaggerating (e.g., "I'm literally dying of boredom" vs "I am dying, send an ambulance").
* **Entity Detection:** Identifies if specific targets (like a person's name) or emergency entities (like "Police", "Fire", "Ambulance") are mentioned.

### Phase 4: Risk Engine & Severity Calculation (`riskEngine.js`)
This is the final decision-making layer. It takes the outputs from Phases 1, 2, and 3, and calculates a final **Severity Level**:
* **Critical (Score 4):** Active emergencies, severe self-harm, or violent crimes with high fear/despair emotions.
* **High (Score 3):** Significant civic issues, harassment, or crimes with anger/fear, but no immediate threat to life.
* **Medium (Score 2):** Standard complaints, civic issues with moderate emotion.
* **Low (Score 1):** General complaints, highly hyperbolic statements, or posts with neutral/joyful emotions.

---

## 🚀 Application Flow (User Journey)

1. **Authentication:** An admin logs into the web application using their credentials (e.g., Email: `aryanpatidar@example.com`, Password: `admin`). Authentication is secured using JWT cookies.
2. **Dashboard:** The admin views the main dashboard containing metrics and a list of processed incidents, sorted by severity.
3. **Data Ingestion:** 
   * The admin navigates to the Upload/Analyze page.
   * They can upload a JSON or CSV dataset of scraped social media posts (e.g., tweets about local issues).
4. **Processing:** The frontend sends the dataset to the `/api/upload/dataset` endpoint. The backend processes the posts sequentially through the 4-Phase Pipeline.
5. **Review & Action:** The processed posts appear on the admin dashboard. The admin can review the AI's verdict, see the extracted entities and emotions, add manual notes, and mark the incident as "Resolved."

---

## 🧠 Why this approach?
* **Parallel Processing:** By running the Topic and Emotion engines at the same time, the system significantly reduces the latency of external AI API calls.
* **Hyperbole Filtering:** Simple keyword matchers fail because humans use dramatic language online. Using a dedicated NLP filter layer ensures that a tweet saying "This traffic makes me want to jump off a bridge" isn't incorrectly flagged as a Critical Self-Harm incident.
* **Scalability:** The architecture separates the AI engines from the core REST API logic, meaning the AI models can be easily swapped or upgraded in the future without rewriting the backend.
