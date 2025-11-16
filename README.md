# BetterVue — Lancement rapide

BetterVue fournit une visibilité en temps réel sur la qualité des entretiens techniques menés par vos recruteurs : pertinence et difficulté des questions, couverture des sujets clés, ton et recommandations post-entretien. Cette documentation rapide explique comment installer, configurer et lancer l'application telle qu'implémentée dans ce dépôt (répertoire principal : quantcoach-livekit).

IMPORTANT — ne commitez jamais vos clés API dans le dépôt. Utilisez un fichier `.env` local ou un gestionnaire de secrets.

---

Table des matières
- Aperçu
- Pré-requis
- Arborescence importante
- Installation et configuration
  - Backend (FastAPI + agent)
  - Frontend (React / Vite)
- Variables d'environnement principales
- Commandes courantes
- Utilisation (création de salle / flux SSE)
- Débogage et résolution de problèmes
- Notes de déploiement et bonnes pratiques
- Ressources et contact

---

Aperçu
BetterVue :
- Analyse le CV d'un candidat et suggère des questions.
- Évalue chaque question (pertinence, difficulté, sujets couverts).
- Fournit des indicateurs en temps réel : barre de difficulté, radar de couverture des sujets, indicateur de ton, timeline, red-flags.
- Génère un résumé d'entretien AI (score, points forts, recommandations).

Dans ce dépôt, la fonctionnalité principale est implémentée dans quantcoach-livekit/ :
- Backend : gestion d'agents, SSE (Server-Sent Events), API FastAPI.
- Frontend : dashboard React/TypeScript affichant visualisations temps réel.

---

Prérequis
- Python 3.11+ (le projet mentionne 3.12 dans la doc, 3.11+ devrait fonctionner)
- Node.js 18+ et npm ou pnpm
- LiveKit (instance ou cloud) — pour la vidéo/voix en temps réel
- Clés API pour les services LLM / STT / TTS (OpenAI, Anthropic, Deepgram, ElevenLabs, etc.)
- (Optionnel) env vars et secrets configurés via .env ou gestionnaire de secrets

---

Arborescence importante
- quantcoach-livekit/
  - backend/  → FastAPI, agent et scripts Python
  - frontend/ → app React / Vite (TypeScript)
- .env.example (exemples d'env vars globales)
- quantcoach-livekit/backend/.env.example
- quantcoach-livekit/frontend/.env.example
- IMPLEMENTATION_GUIDE.md → guide détaillé, endpoints et workflow

---

Installation & configuration

1) Cloner le dépôt
```bash
git clone https://github.com/eliasbr26/iterateHackathon.git
cd iterateHackathon
```

2) Backend — quantcoach-livekit/backend
```bash
cd quantcoach-livekit/backend
# Créez/activez un env Python
python -m venv .venv
source .venv/bin/activate    # Linux / macOS
# ou .venv\Scripts\activate   # Windows

pip install -r requirements.txt
```

Créez un fichier `.env` dans quantcoach-livekit/backend/ en vous basant sur quantcoach-livekit/backend/.env.example :
- LIVEKIT_URL (ex: wss://your-livekit-instance.livekit.cloud)
- LIVEKIT_API_KEY
- LIVEKIT_API_SECRET
- ELEVENLABS_API_KEY (ou DEEPGRAM_API_KEY selon l'implémentation)
- ANTHROPIC_API_KEY (ou OPENAI_API_KEY, selon les intégrations utilisées)

Exemple (NE PAS utiliser les valeurs réelles affichées dans le dépôt) :
```dotenv
LIVEKIT_URL=wss://your-livekit-instance.livekit.cloud
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret
ELEVENLABS_API_KEY=your_elevenlabs_key
ANTHROPIC_API_KEY=your_anthropic_key
# Optionnel
# SERVER_HOST=0.0.0.0
# SERVER_PORT=8000
```

3) Frontend — quantcoach-livekit/frontend
```bash
cd quantcoach-livekit/frontend
npm install
# ou pnpm install
```

Créez un fichier `.env` (Vite) en vous basant sur quantcoach-livekit/frontend/.env.example :
```env
VITE_API_URL=http://localhost:8000
```

---

Lancement en local

1) Démarrer le backend
Depuis quantcoach-livekit/backend :
```bash
# Activez l'environnement Python si nécessaire
python server.py
```
Par défaut le serveur écoute souvent sur http://0.0.0.0:8000 (voir messages de démarrage).

2) Démarrer le frontend
Depuis quantcoach-livekit/frontend :
```bash
npm run dev
```
Le serveur de développement Vite démarrera (typiquement http://localhost:5173).

---

Commandes courantes
- Backend :
  - pip install -r requirements.txt
  - python server.py
- Frontend :
  - npm install
  - npm run dev
  - npm run build (pour production)
- API (exemples curl) :
  - Créer une salle (l'agent démarre automatiquement) :
    curl -X POST http://localhost:8000/rooms/create -H "Content-Type: application/json" -d '{"room_name":"my-interview"}'
  - Vérifier le statut de l'agent :
    curl http://localhost:8000/rooms/my-interview/status
  - Stream SSE (ouvrir dans navigateur ou EventSource) :
    http://localhost:8000/rooms/my-interview/stream
  - Récupérer analytics :
    http://localhost:8000/rooms/my-interview/analytics

---

Flux et comportement
- Quand une salle est créée via l'API ou le frontend, l'AgentManager démarre automatiquement un agent pour cette salle.
- L'agent traite l'audio (STT), l'envoie aux évaluateurs LLM, et publie des événements en mémoire.
- Le backend expose un flux SSE (`/rooms/{room_name}/stream`) consommé par le frontend pour mises à jour temps réel (transcript, évaluations, métriques).
- Les visualisations (barre difficulté, radar, timeline, red-flags, ton, etc.) se mettent à jour via ces événements.

---

Variables d'environnement principales (récapitulatif)
- LIVEKIT_URL — URL de votre instance LiveKit (wss://...)
- LIVEKIT_API_KEY — clé API LiveKit
- LIVEKIT_API_SECRET — secret LiveKit
- ELEVENLABS_API_KEY — (ou DEEPGRAM_API_KEY) pour transcription/audio
- ANTHROPIC_API_KEY — clé Anthropic (ou OPENAI_API_KEY) pour LLM
- VITE_API_URL — URL backend (frontend)

N'oubliez pas : ne stockez pas ces clefs dans le dépôt.

---

Dépannage rapide

- "Agent doesn't start automatically"
  - Vérifiez les logs de server.py (AgentManager).
  - Vérifiez que les variables d'environnement requises sont présentes.

- "SSE stream not connecting"
  - Vérifiez la console navigateur (erreurs CORS / URL).
  - Assurez-vous que le port backend est accessible et que l'URL VITE_API_URL est correcte.

- Erreur : "No module named 'sse_starlette'"
  - pip install sse-starlette>=2.1.0

- Frontend : composants non rendus / erreurs TypeScript
  - npm install, redémarrez l'IDE/serveur TS.

---

Notes de déploiement / production
- Utiliser un service d'orchestration/process manager pour le backend (systemd, docker-compose, Kubernetes).
- Exécuter le backend sous un serveur ASGI (uvicorn/gunicorn + uvicorn workers) :
  - uvicorn server:app --host 0.0.0.0 --port 8000 --workers 4
- Frontend : builder la version de production (npm run build) et servir via CDN ou serveur statique (nginx).
- SSL / HTTPS : indispensable pour LiveKit et la sécurité des échanges.
- Gestion des secrets : use vaults / services gérés (AWS Secrets Manager, GitHub Secrets, etc.).
- Persistance : l'implémentation actuelle utilise de la mémoire pour file d'événements — prévoir stockage (DB) pour historique, redémarrage et audits.
- Scalabilité : séparer gestion des agents (workers), broker d'événements (Redis, Kafka), et autoscaling des agents si besoin.

---

Sécurité & confidentialité
- Chiffrez / protégez les clefs API.
- Assurez la confidentialité des entretiens (stockage chiffré, retention policies).
- Avertissez les utilisateurs candidats/recruteurs que l'entretien est analysé par une IA (conformité RGPD/CEA selon juridiction).

---

Ressources & liens rapides
- Fichiers d'exemple d'env :
  - ./quantcoach-livekit/backend/.env.example
  - ./quantcoach-livekit/frontend/.env.example
- Guide d'implémentation détaillé : IMPLEMENTATION_GUIDE.md
- Notes pour assistants : CLAUDE.md (documents internes)

---

Contact
Pour questions sur le code ou l'architecture du projet, consultez les fichiers IMPLEMENTATION_GUIDE.md et CLAUDE.md du dépôt, ou contactez l'équipe responsable du dépôt.

Bonne installation et bon débogage !  
BetterVue — rendre chaque entretien plus juste, pertinent et professionnel.
