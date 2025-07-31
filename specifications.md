# 🧠 MindVault – AI-Powered Note-Taking Assistant

## 1. Overview

### 1.1 Context

MindVault is a smart web-based note-taking application designed to compete with tools like Notion or Obsidian. It features deep integration with customizable AI models, enabling users to centralize, structure, enrich, and retrieve notes easily.

### 1.2 Goals

- Build a powerful yet accessible web app
- Leverage AI for writing assistance, organization, and semantic search
- Deliver a fast, smooth, and intuitive user experience
- Support user customization (themes, folder structures, templates, etc.)
- Remain fully open-source and cloud-provider-independent

### 1.3 Target Audience

- Students and researchers  
- Developers and engineers  
- Writers and journalists  
- Users of Notion / Obsidian / Logseq looking for a web-first, open-source alternative

---

## 2. Core Features

### 2.1 Authentication & User Management

- Secure registration/login
- Email + password auth (JWT + Bcrypt)
- Profile management interface
- API route protection via tokens

### 2.2 Notes System

- Rich Markdown editor (AI-enhanced)
- Folder & subfolder organization
- Keyword and AI-based (vector) search
- Versioning (note edit history)

### 2.3 AI Features

- **Auto-summarize**: summarize long notes
- **Auto-tagging**: suggest relevant tags
- **Q&A**: answer user questions from notes
- **Note assistant**: generate content from prompts
- **Vector search**: semantic similarity matching

### 2.4 UI & UX

- Fully responsive app (desktop/mobile)
- Light/Dark themes
- Dashboard (recent notes, AI suggestions)
- Integrated AI chat panel for contextual help

---

## 3. Proposed Tech Stack

| Component             | Technology                                |
|----------------------|--------------------------------------------|
| Frontend             | Next.js, TailwindCSS, TypeScript           |
| Backend              | Java (Spring Boot), REST API               |
| Database             | PostgreSQL                                 |
| Vector DB (optional) | pgvector or Qdrant                         |
| Auth & Tokens        | JWT, Bcrypt                                |
| Deployment           | Vercel (frontend) + Railway (backend)      |
| File Storage (opt.)  | S3-compatible (e.g. Supabase Storage, MinIO) |

---

## 4. Functional Architecture (Summary)

Users → Web App (Next.js)
→ Calls REST API (Spring Boot)
→ Interacts with PostgreSQL
→ Calls AI engine (OpenAI API / Ollama local)
→ Stores vectors for semantic search

---

## 5. Security

- Passwords hashed using Bcrypt
- Stateless auth with JWT
- Sensitive endpoints protected
- API rate limiting
- Clear separation of DEV/PROD environments

---

## 6. Roadmap (Estimated)

| Phase              | Goal                             | Est. Duration |
|-------------------|----------------------------------|---------------|
| 1. Initialization  | Repos, planning, wireframes       | 2–3 days      |
| 2. Authentication  | Signup/Login with JWT             | 3–5 days      |
| 3. Notes Editor    | CRUD, foldering, Markdown         | 5–7 days      |
| 4. AI Integration  | Summarize, tagging, assistant     | 5–6 days      |
| 5. Vector Search   | Embeddings + semantic lookup      | 4–6 days      |
| 6. UI Polish       | Theming, responsive, animations   | 4 days        |
| 7. Deployment      | Hosting, README, licenses         | 2–3 days      |

---

## 7. Future Features (v2+)

- Real-time collaboration  
- Markdown export/import  
- Graph view (linked notes)  
- Custom AI models via Ollama or LangChain  
- Public API for note automation  
- Plugin system (Markdown extensions)

---

# 🏗️ MindVault – Technical Architecture

## 1. High-Level Architecture

- **Frontend**: Next.js (Vercel)
- **Backend**: Spring Boot (REST API)
- **Database**: PostgreSQL
- **AI Engine**: OpenAI or local via Ollama
- **Vector DB**: pgvector or Qdrant

User Interface (Next.js)
↓
Spring Boot API
↓
PostgreSQL ←→ Vector Search (e.g., pgvector)
↓
AI Service (OpenAI / Ollama)

---

## 2. Functional Domains

| Domain       | Description                                 |
|--------------|---------------------------------------------|
| User         | Auth, profile management                    |
| Notes        | Create, edit, delete, history               |
| Folders      | Organize notes hierarchically               |
| AI Assistant | Summarization, auto-tagging, Q&A            |
| Search       | Full-text and semantic vector search        |

---

## 3. Spring Boot Package Structure

com.mindvault
├── config // Security, CORS, JWT
├── controller // REST API Controllers
├── dto // Input/output DTOs
├── exception // Centralized error handling
├── model // JPA entities
├── repository // JPA repositories
├── security // JWT auth, UserDetails
├── service // Business logic
├── util // Utility classes
└── vector // Embedding & vector search logic


---

## 4. Data Model (PostgreSQL)

### users

| Field       | Type      | Description       |
|-------------|-----------|-------------------|
| id          | UUID (PK) | Unique ID         |
| email       | VARCHAR   | Unique email      |
| password    | VARCHAR   | Hashed            |
| name        | VARCHAR   | Full name         |
| created_at  | TIMESTAMP | Registration date |

### folders

| Field      | Type      | Description         |
|------------|-----------|---------------------|
| id         | UUID (PK) |                     |
| user_id    | UUID (FK) | Owner               |
| name       | VARCHAR   | Folder name         |
| parent_id  | UUID (FK) | Nullable (root)     |
| created_at | TIMESTAMP |                     |

### notes

| Field       | Type      | Description               |
|-------------|-----------|---------------------------|
| id          | UUID (PK) |                           |
| user_id     | UUID (FK) | Owner                     |
| folder_id   | UUID (FK) | Nullable                  |
| title       | VARCHAR   | Note title                |
| content     | TEXT      | Markdown content          |
| summary     | TEXT      | AI-generated summary      |
| created_at  | TIMESTAMP | Creation date             |
| updated_at  | TIMESTAMP | Last updated              |

### tags

| Field   | Type      | Description     |
|---------|-----------|-----------------|
| id      | UUID (PK) |                 |
| user_id | UUID (FK) | Owner           |
| label   | VARCHAR   | Tag label       |

### note_tags (Join Table)

| note_id | UUID (FK) |
|---------|-----------|
| tag_id  | UUID (FK) |

Many-to-Many relationship between notes and tags.

### embeddings (pgvector)

| note_id   | UUID (PK, FK)  |
| embedding | VECTOR(1536)   |

Vector representation for semantic search (compatible with OpenAI, etc.)

---

## 5. REST API Specification

All endpoints are prefixed with `/api/`  
Authentication via Bearer JWT

### 🔐 AuthController

| Method | Endpoint     | Description            |
|--------|--------------|------------------------|
| POST   | /auth/register | Create a new user      |
| POST   | /auth/login    | Get JWT token          |
| GET    | /auth/me       | Get current user data  |

### 📁 FolderController

| Method | Endpoint           | Description             |
|--------|--------------------|-------------------------|
| GET    | /folders           | List user's folders     |
| POST   | /folders           | Create a folder         |
| PUT    | /folders/{id}      | Rename a folder         |
| DELETE | /folders/{id}      | Delete a folder         |

### 📝 NoteController

| Method | Endpoint           | Description            |
|--------|--------------------|------------------------|
| GET    | /notes             | List notes (filterable)|
| GET    | /notes/{id}        | Read a note            |
| POST   | /notes             | Create a note          |
| PUT    | /notes/{id}        | Update a note          |
| DELETE | /notes/{id}        | Delete a note          |

### 🤖 IAController

| Method | Endpoint             | Description             |
|--------|----------------------|-------------------------|
| POST   | /ai/summarize        | Summarize a note        |
| POST   | /ai/autotag          | Suggest tags            |
| POST   | /ai/assistant        | AI chat assistant       |
| POST   | /ai/semantic-search  | Vector-based search     |

### 🔍 SearchController

| Method | Endpoint             | Description                  |
|--------|----------------------|------------------------------|
| GET    | /search?query=...    | Full-text search             |
| POST   | /search/semantic     | Semantic vector search       |

