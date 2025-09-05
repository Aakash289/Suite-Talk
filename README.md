# Suite-Talk

Automate hotel review intake, analyze sentiment, and open tickets for negative feedback, all with a config-driven n8n workflow.


Why Suite Talk?

Guest feedback is scattered across Google & TripAdvisor. Managers miss issues, and quarterly reporting takes forever.
Suite Talk pulls reviews automatically, normalizes them, scores sentiment, and (optionally) creates Zendesk tickets for fast follow-up, while storing everything in Postgres for reporting.


What it does

Ingests reviews from Google My Business and a configurable TripAdvisor endpoint

Normalizes different payloads into one schema

Dedupes by offset so each run only processes new items

Sentiment analysis via OpenAI Chat Completions

Auto-tickets negative reviews in Zendesk (toggleable)

Stores all reviews in Postgres for analytics


Workflow at a glance
Cron (hourly)

  → PG Init (idempotent schema)
  
  → PG Load (hotels, sources, settings, last_seen)
  
      ├─ IF provider == google      → HTTP Google Reviews
      
      └─ IF provider == tripadvisor → HTTP TripAdvisor Reviews
      
  → Normalize (Code)  // unify shape + filter by last_seen
  
  → OpenAI (HTTP POST) // sentiment
  
  → Parse (Code)       // score + label
  
  → PG Upsert (reviews)
  
  → PG Offset (advance last_seen)
  
  → IF Negative?
  
       → IF Zendesk enabled?
       
            → Zendesk: Create Ticket


Prerequisites

n8n (1.50+ recommended; uses the Code node)

Postgres database

Credentials in n8n:

Postgres

Google OAuth2 (My Business reviews)

OpenAI (Header Auth: Authorization: Bearer <API_KEY>)

Zendesk (OAuth2 or API token) — optional


Roadmap

Slack/Teams alerts for managers

Auto-assignment in Zendesk by hotel_id

Additional providers (Booking, Facebook, Google Q&A)
