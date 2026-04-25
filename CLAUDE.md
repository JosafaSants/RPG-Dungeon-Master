# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**RPG Dungeon Master** — A local single-player RPG system with an AI-controlled Dungeon Master that narrates stories, manages dice rolls, maintains a persistent player character, and displays interactive maps. Portuguese (BR) text interface. Everything runs locally — no cloud, no cost.

Current status: Stage 2 — Book ingestion pipeline (in progress).

---

## Execution Method (never break these rules)

- Read any target file completely before changing it
- Make only the change described in the prompt, nothing more
- Comment every function and every non-obvious line in English
- Do not create new files unless explicitly told to
- Do not install packages unless explicitly told to
- After each change, summarize exactly what was modified and why
- All code comments: English
- All user-facing text: Portuguese (BR)
- One file or command at a time — always wait for the next prompt before continuing
- Never assume anything — if anything is unclear, say so before acting
- Terminal commands (git, pip, curl, etc.) are executed directly by the user — provide the exact command clearly, one at a time
- File reading: ask the user to paste the content into the chat rather than reading from disk
- New code creation: always handled by Claude Code via prompt — never copy-paste existing files

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| LLM | Ollama + gemma3:4b |
| Embeddings | nomic-embed-text via Ollama |
| Vector DB | ChromaDB |
| Database | SQLite + SQLAlchemy ORM |
| Backend API | Python 3.11+ + FastAPI |
| Frontend | React (via Lovable, integrated later) |
| PDF Parsing | pdfplumber + BeautifulSoup |
| Version Control | Git |

---

## Architecture

**Backend** (`backend/`):
- `api/` — FastAPI route handlers
- `core/` — Core DM logic (narrative engine, game mechanics, rules)
- `db/` — SQLAlchemy models and SQLite connection setup
- `models/` — Pydantic schemas for request/response validation
- `services/` — External integrations (Ollama LLM, ChromaDB embeddings, parser, chunker, embedder, catalog)

**Frontend** (`frontend/`) — React SPA for player interaction and visual display (maps, character info). Generated via Lovable.

**Data** (`data/`):
- `books/` — RPG rulebook PDFs and EPUBs
- `vector_store/` — ChromaDB persistent storage (embeddings and metadata)
- `catalog/` — Structured JSON data extracted from adventure books (rooms, NPCs, quests, maps)

**Scripts** (`scripts/`):
- `ingest.py` — Entry point for the book ingestion pipeline

**Tests** (`tests/`) — Automated test suite

---

## DM Response JSON Schema (locked — never change this structure)

Every LLM response must follow this exact structure. The backend must reject any response that does not conform.

```json
{
  "narrative": "string — DM narration in Portuguese (BR)",
  "game_state_delta": {
    "player_moved_to": null,
    "hp_change": 0,
    "items_gained": [],
    "items_lost": [],
    "xp_gained": 0,
    "conditions_added": [],
    "conditions_removed": [],
    "doors_opened": [],
    "entities_revealed": []
  },
  "combat": {
    "active": false,
    "dice_rolled": [],
    "result": null
  },
  "map_update": {
    "reveal_rooms": [],
    "move_entities": []
  }
}
```

---

## Book Ingestion Pipeline

Books are provided as PDF or EPUB files. The pipeline runs once per book.

Steps:
1. `scripts/ingest.py` — Entry point, orchestrates the full pipeline
2. `backend/services/parser.py` — Reads PDF via pdfplumber, reads EPUB via BeautifulSoup, outputs clean text
3. `backend/services/chunker.py` — Splits text into ~500 token chunks with metadata (book title, chapter, section, book type)
4. `backend/services/embedder.py` — Sends chunks to ChromaDB using nomic-embed-text via Ollama
5. `backend/services/catalog.py` — For adventure books only: extracts structured data (rooms, NPCs, quests, encounter tables, maps) into JSON files in `data/catalog/`

---

## Game Systems

### Maps
- Interactive map displayed in the frontend
- Load from adventure catalog JSON if available; generate procedurally otherwise
- Fog of war: rooms revealed as player explores
- Entities: player, enemies, NPCs, chests, doors

### Combat
- Fully automated — system rolls all dice
- Follows D&D 5e rules: initiative, attack rolls, saving throws, damage, conditions
- Player cannot manipulate or override any roll
- Combat state included in every DM response

### Character and Progress
- One persistent character per player across all campaigns
- Tracks: name, race, class, level, XP, HP, STR/DEX/CON/INT/WIS/CHA, inventory, equipment, spells, conditions, quest log, campaign history

---

## SQLite Schema (overview)

Tables:
- `player` — player profile
- `character` — stats and progression
- `inventory` — items owned
- `equipment` — items currently equipped
- `spells` — spells known and slots
- `quest_log` — active and completed quests
- `campaign` — campaigns and their status
- `session_log` — log of every session

---

## Stage Roadmap

| Stage | Deliverable |
|-------|-------------|
| 1 ✅ | Project scaffold + git + requirements.txt + README.md |
| 2 | Book ingestion pipeline (parser, chunker, embedder, catalog) |
| 3 | SQLite schema + character system |
| 4 | Game engine (combat, map, state machine) |
| 5 | DM brain (RAG + Ollama prompt + JSON response parser) |
| 6 | FastAPI backend (all routes + WebSocket for streaming) |
| 7 | Full frontend UI via Lovable |
| 8 | Frontend + backend integration |
| 9 | End-to-end test + Code Review + README final |

---

## Development Commands

### Setup
```bash
pip install -r requirements.txt
# Ensure Ollama is running locally with gemma3:4b and nomic-embed-text
```

### Backend
```bash
uvicorn backend.api.main:app --reload
pytest tests/ -v
pytest tests/ --cov=backend --cov-report=html
```

### Ingestion
```bash
python scripts/ingest.py --file data/books/mybook.pdf --type rules
```
