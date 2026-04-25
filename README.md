# RPG Dungeon Master 🎲

Sistema local de RPG com Dungeon Master controlado por IA.

## Descrição

O jogador interage via texto livre em português (BR) com um Dungeon Master inteligente que:
- Narra a história e reage às ações do jogador
- Gerencia todas as rolagens de dados automaticamente
- Mantém um personagem persistente entre campanhas
- Exibe mapas e informações visuais interativas

## Stack Tecnológica

| Camada | Tecnologia |
|--------|-----------|
| LLM | Ollama (mistral-nemo ou llama3.1:8b) |
| Embeddings | nomic-embed-text via Ollama |
| Vector DB | ChromaDB |
| Banco de dados | SQLite + SQLAlchemy |
| Backend | Python + FastAPI |
| Frontend | React |
| Parser de livros | pdfplumber + BeautifulSoup |

## Estrutura do Projeto
RPG-Dungeon-Master/
├── backend/
│   ├── api/          # Rotas FastAPI
│   ├── core/         # Lógica central (DM, dados, regras)
│   ├── db/           # Modelos e conexão SQLite
│   ├── models/       # Schemas Pydantic
│   └── services/     # Serviços (Ollama, ChromaDB, etc.)
├── frontend/         # Interface React
├── data/
│   ├── books/        # PDFs dos livros de RPG
│   └── vector_store/ # Base vetorial ChromaDB
├── scripts/          # Scripts utilitários
└── tests/            # Testes automatizados

## Pré-requisitos

- Python 3.11+
- Ollama instalado e rodando localmente
- Node.js 18+ (para o frontend)

## Como rodar

```bash
# Instalar dependências Python
pip install -r requirements.txt

# Iniciar o backend
uvicorn backend.api.main:app --reload
```

## Status

🚧 Em desenvolvimento — Estágio 1: Estrutura base