# Agent IA de prospection et réservation (n8n) — Génération Prospect, Bali

Workflow n8n qui déploie un assistant conversationnel sur **Telegram** pour une agence d'activités à Bali. L'agent présente les activités avec les prix exacts, collecte les informations du prospect, enregistre la réservation et envoie une confirmation par email.

## Fonctionnalités

- **Conversation Telegram** : l'agent répond aux questions sur les activités, les prix et les disponibilités, avec une mémoire de conversation par utilisateur.
- **RAG sur base de connaissance** : les documents déposés dans un dossier Google Drive sont automatiquement indexés (découpage en chunks de 400 tokens, embeddings Gemini, vector store en mémoire). L'agent s'appuie uniquement sur ces données pour répondre.
- **Recherche web** (Tavily MCP) pour les compléments d'information.
- **Collecte des prospects** : nom, email, téléphone, activité et date souhaitées sont ajoutés dans Google Sheets.
- **Réservation** : création d'un événement dans Google Calendar.
- **Confirmation** : email récapitulatif envoyé au client via Gmail.

## Architecture

```
Flux 1 — Conversation
Telegram Trigger ─► AI Agent ─► Send a text message (Telegram)
                      ├─ Modèle : Groq (openai/gpt-oss-120b)
                      ├─ Mémoire : Simple Memory (fenêtre de 3 messages, clé = id Telegram)
                      └─ Outils : Vector Store (RAG) · Tavily MCP · Google Sheets · Google Calendar · Gmail

Flux 2 — Indexation de la base de connaissance
Google Drive Trigger (nouveau fichier, toutes les minutes)
   ─► Download file ─► Vector Store (insert) ◄─ Embeddings Gemini · Data Loader · Token Splitter
```

## Stack

| Rôle | Outil |
|---|---|
| Orchestration | n8n |
| LLM | Groq — `openai/gpt-oss-120b` |
| Embeddings | Google Gemini (`gemini-embedding-2`) |
| Canal | Telegram Bot |
| Données / agenda / email | Google Sheets, Google Calendar, Gmail, Google Drive |
| Recherche web | Tavily (MCP) |

## Installation

1. Importer `workflow.json` dans n8n (**Workflows → Import from file**).
2. Créer les credentials : Telegram (token du bot via @BotFather), Groq, Google Gemini, Google OAuth (Drive, Sheets, Calendar, Gmail), Tavily.
3. Adapter les ressources à votre compte :
   - le dossier Google Drive surveillé (documents sur les activités et les prix) ;
   - le Google Sheet de destination (colonnes : `Nom`, `Email`, `Téléphone`, `Activité_souhaitée`, `Date_activité`) ;
   - l'agenda Google Calendar.
4. Déposer un document dans le dossier Drive pour alimenter la base de connaissance (le vector store est en mémoire : il faut le réalimenter après un redémarrage de n8n).
5. Activer le workflow et écrire au bot sur Telegram.

## Limites et pistes d'amélioration

- Vector store en mémoire : passer à une base persistante (Qdrant, Supabase pgvector…).
- Ajouter une vérification des disponibilités avant la création de l'événement.
- Gérer les doublons de prospects dans le Sheet.

## Sécurité

Aucun identifiant n'est stocké dans ce dépôt : les credentials restent dans n8n. Vérifiez qu'aucun ID de Sheet, d'agenda ou de dossier personnel ne reste dans `workflow.json` avant de le publier.


