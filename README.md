# FinTrack
Predictive budgeting, auto-parsing bank SMS, stock watchlists, and an AI finance copilot — all in one fast, privacy-first app.

FinTrack is a cross-platform personal finance app (Flutter) with a Python/Flask backend that:

Parses bank SMS and normalizes transactions automatically.

Builds spend forecasts with an XGBoost model (reported ~~90% accuracy in prior runs).

Provides an AI finance copilot (Gemini) for Q&A (“Why did groceries spike?”, “How much can I save if I cut coffee?”).

Tracks stocks (Tingo API) and couples them with your cashflow trends.

Stores data locally (SQLite) with optional cloud sync (Supabase/other, optional).

✨ Features

Auto-Ingest

SMS / notifications parsing → date, merchant, amount, category, account.

CSV import (banks & wallets) with header mapping.

Smart Categorization

Rule-based + ML-assisted category suggestions.

Forecasting & Budgets

XGBoost models for next-month spend and category-level projections.

“What-if” scenarios (e.g., cut eating-out by 20%).

AI Copilot (Gemini)

Natural-language chat over your finance data (secure, scoped prompts).

Explanations with citations to raw transactions.

Stocks & Watchlists

Tingo quotes, basic metrics, and alert rules (spikes/drops).

Dashboards

Trend lines, category pies, cash-flow waterfall, and savings rate.

Privacy 1st

Local SQLite; environment-keyed cloud/LLM access.

🧱 Architecture
fintrack/
├─ app/                        # Flutter app (Dart)
│  ├─ lib/
│  │  ├─ screens/              # Home, Insights, Budgets, Stocks, Chat
│  │  ├─ widgets/              # Charts, cards, loaders
│  │  ├─ services/             # REST client, local DB helpers
│  │  ├─ models/               # Dart models (Transaction, Budget, Watchlist)
│  │  └─ state/                # Riverpod/Provider/Bloc (TODO: confirm)
│  └─ assets/                  # Icons, fonts, sample CSVs
├─ server/                     # Python 3.10+ (Flask API)
│  ├─ app.py                   # API entry
│  ├─ routes/                  # /transactions, /forecast, /chat, /stocks
│  ├─ services/                # sms_parser.py, model.py, tingo_client.py
│  ├─ db/                      # schema.sql, migrations/, sqlite helpers
│  ├─ models/                  # xgboost.pkl, encoder.pkl (saved artifacts)
│  └─ notebooks/               # EDA & training (Jupyter)
├─ docs/                       # API docs, architecture diagrams
└─ README.md


TODO (you): adjust folders to match your zip if paths differ.

🗃️ Data Model (SQLite)

Tables (proposed):

transactions(id, ts, account, merchant, amount, currency, raw_text, category, source)

budgets(id, month, category, limit_amount)

forecasts(id, month, category, predicted_amount, model_version)

watchlist(id, symbol, created_at)

users(id, created_at) (optional; for multi-profile)

TODO: Replace with your actual schema (server/db/schema.sql) after upload.

🤖 Machine Learning

Task: next-period spend prediction (overall + per-category).

Model: XGBoost (regression).

Features (typical): rolling sums, category proportions, week-of-month, pay-cycle flags, outlier caps, lag features.

Reported perf: ~90% accuracy in prior runs (interpret as directionally strong; supply your exact metric like R² / MAPE once you re-train).

Artifacts saved via joblib:

server/models/xgboost.pkl
server/models/encoder.pkl
server/models/feature_spec.json

Training locally
cd server
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab  # open notebooks/01_train.ipynb, adjust data path
# export final artifacts to server/models/

🔌 API (Flask)

Base: http://localhost:8000 (dev) or your deployed URL.

Transactions

POST /transactions/ingest/sms → parse & insert an SMS body

body: { "message": "Your AECB Card ****1234 spent AED 24.50 at STARBUCKS..." }

POST /transactions/import/csv → upload CSV

GET /transactions?from=YYYY-MM-DD&to=YYYY-MM-DD&category=Food

Forecasting

POST /forecast/run → recompute forecasts (returns summary)

GET /forecast/:month → projections by category

Chat (Gemini)

POST /chat → { "prompt": "Why did groceries spike in July?" }

Stocks

GET /stocks/quote?symbol=AAPL

POST /stocks/watchlist → { "symbol": "TSLA" }

TODO: Replace/confirm paths from your actual routes/ files.

📱 Flutter App

State mgmt: (Provider, Riverpod, or Bloc — confirm in your code)

HTTP: dio or http

Local DB: sqflite or drift

Charts: fl_chart or similar

Run (Dev)
cd app
flutter pub get
flutter run -d chrome   # or your device


Tip: Ensure the server URL is set in a config file (see Environment).

🔐 Environment & Secrets

Create .env in server/:

# Flask
FLASK_ENV=development
FLASK_DEBUG=1
PORT=8000

# Gemini
GEMINI_API_KEY=<TODO>

# Tingo (stocks)
TINGO_API_KEY=<TODO>

# Database
SQLITE_PATH=./db/fintrack.db


Flutter config (e.g., lib/config.dart or .env via flutter_dotenv):

class AppConfig {
  static const apiBase = String.fromEnvironment(
    'API_BASE',
    defaultValue: 'http://localhost:8000',
  );
}


Build with:

flutter run --dart-define=API_BASE=https://your-backend.example.com

🧪 Testing

Server: pytest for routes + parser unit tests (server/tests/).

Mobile: widget tests for charts, list virtualization, empty states.

# Server
cd server
pytest -q

# App (Dart)
cd app
flutter test

🔒 Security & Privacy

All parsing runs locally; no financial PII sent to the LLM.

The Gemini copilot receives aggregates and anonymized snippets with explicit system prompts to avoid data leakage.

Keys loaded from environment only; never commit .env.

Optional: enable row-level encryption for SQLite (e.g., SQLCipher) — add a KMS flow if you deploy multi-tenant.

🚀 Deployment
Option A — Single VM/Docker

Run Flask behind Nginx/Gunicorn.

Serve Flutter Web (if using web build) via Nginx.

# Server
docker build -t fintrack-api ./server
docker run -p 8000:8000 --env-file server/.env fintrack-api

# App (Web)
cd app && flutter build web
# copy build/web/ to your web host or serve with Nginx

Option B — Split Hosting

API: Render/Fly.io/Heroku (Docker).

App: Firebase Hosting, Vercel, or Netlify (Flutter Web).

TODO: Add your actual deploy commands once you choose a target.

📈 Screenshots

Place images in app/assets/screens/ and reference them:

Home	Insights	Chat

	
	

TODO: Export real screenshots once the app runs on your device.

🗺️ Roadmap

 Bank-grade PDF statement ingestion (multi-bank templates).

 Auto-budget recommendations based on seasonality.

 Multi-currency normalization and FX impacts.

 Goal tracking (“3-month emergency fund”, “Pay off AED 5k card”).

 Cloud sync & shared household budgets.

 On-device ML (TensorFlow Lite) for categorization.

🤝 Contributing

PRs welcome! Please:

Open an issue describing the change.

Keep commits scoped and tested.

Update docs if you touch API or config.

📜 License

MIT — see LICENSE.

🙌 Acknowledgements

XGBoost, scikit-learn for modeling.

Gemini for natural-language insights.

Tingo for market data.

Flutter/Flask community for great tooling.

Quick Start (TL;DR)
# Backend
cd server
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # or create as above
python app.py

# Frontend
cd ../app
flutter pub get
flutter run --dart-define=API_BASE=http://localhost:8000
