# NearMe
Near Me app

Near Me — Social Food Discovery Through Trusted Circles

> *"Not the most reviewed restaurant. The one your people actually love."*

Anonymous crowds lie. Five-star ratings are gamed. **Near Me** solves the fundamental trust problem in restaurant discovery by replacing crowd-sourced reviews with your personal social graph — only people you trust, rating places they've genuinely experienced.

---

## 🧠 The Core Idea

Every major discovery app (Yelp, Google Maps, TripAdvisor) surfaces restaurants based on **volume of reviews**. But volume ≠ relevance. A place with 4,000 reviews from strangers is less useful than three recommendations from friends who share your taste.

Near Me flips the model:

```
Traditional Apps                    Near Me
────────────────────────────────────────────────────────────
Anonymous strangers rate            Only your trusted circle rates
Volume determines ranking           Trust-weight determines ranking
You scroll 50 reviews               You see 3 friends' opinions
No taste-palette context            Matched by actual taste similarity
Generic "good food" signal          "Sarah loved it — Sarah = your food twin"
```to fill 

---

## ✨ Features

### 🔍 Trust-Graph Discovery
- Restaurant recommendations ranked by **social proximity** — closer friends carry more weight
- **Taste-palette matching** — follow people whose preferences overlap with yours (cuisine type, price range, ambiance)
- Multi-hop discovery: see what friends-of-friends love when your direct network is sparse

### 👥 Social Planning
- **Group polls** — propose restaurants, let your group vote, lock in the plan
- **Shared wishlists** — collaborative "want to try" lists with friends
- Group event coordination (date, time, headcount) within the app

### 📸 Rich Content
- Photo posts from real visits — your feed is your trusted circle's dining diary
- Honest reviews attached to real relationships, not anonymous handles
- Short-form ratings (quick 1–5 + vibe tags) designed for mobile-first, low-friction input

### 👤 Taste Profiles
- Personal bio and cuisine preference tags
- Automatically inferred taste-palette from rating history
- Public profiles — follow anyone whose palate resonates with yours
- Taste-similarity score shown between you and any user

### 🗺️ Integrated Discovery Map
- Map view combining venue locations with trust-weighted recommendation scores
- Filters: cuisine type, price range, distance, "friends been here" flag
- Inspired by Google Maps navigation + Yelp reviews + Instagram social layer

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
│            Mobile App (iOS / Android)  +  Web App            │
└──────────────────────────┬───────────────────────────────────┘
                           │  REST / WebSocket
┌──────────────────────────▼───────────────────────────────────┐
│                        API LAYER                             │
│                    FastAPI Backend                           │
│                                                              │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │  Auth &     │  │  Feed &      │  │  Planning &         │ │
│  │  Profiles   │  │  Discovery   │  │  Polls Service      │ │
│  └─────────────┘  └──────┬───────┘  └─────────────────────┘ │
└─────────────────────────┬┴──────────────────────────────────┘
                          │
┌─────────────────────────▼──────────────────────────────────┐
│                   GRAPH ENGINE LAYER                        │
│                                                            │
│   Social Graph (NetworkX / Neo4j)                          │
│   ┌─────────────────────────────────────────────────────┐  │
│   │  Nodes:  Users  ──────────────  Restaurants         │  │
│   │  Edges:  Follows, Reviews, Visits, Taste-similarity  │  │
│   │  Weights: Trust score, Recency, Interaction depth   │  │
│   └─────────────────────────────────────────────────────┘  │
│                                                            │
│   Recommendation Engine                                    │
│   ┌─────────────────────────────────────────────────────┐  │
│   │  1. Graph traversal (BFS / weighted DFS)            │  │
│   │  2. Trust-weight scoring per hop                    │  │
│   │  3. Taste-palette similarity (cosine similarity)    │  │
│   │  4. Recency decay (exponential time weighting)      │  │
│   │  5. Constraint filtering (distance, price, hours)   │  │
│   │  6. Ranked recommendation list                      │  │
│   └─────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────▼──────────────────────────────────┐
│                     DATA LAYER                             │
│   PostgreSQL (users, restaurants, reviews, relationships)  │
│   Redis (feed cache, session, real-time poll state)        │
│   S3 / Cloud Storage (photos, media)                       │
└────────────────────────────────────────────────────────────┘
```

---

## 🔬 The Graph Recommendation Algorithm

This is the core IP of Near Me. Every recommendation is a function of the social graph.

```python
def score_restaurant(restaurant, user, graph, alpha=0.7, beta=0.2, gamma=0.1):
    """
    Score a restaurant for a given user based on their trust graph.

    Components:
        trust_score      — weighted average rating from social graph neighbors
        taste_match      — cosine similarity between user and reviewer taste vectors
        recency_weight   — exponential decay on review age
    """
    total_score = 0.0
    total_weight = 0.0

    for reviewer, relationship_strength in graph.neighbors(user):
        review = get_review(reviewer, restaurant)
        if not review:
            continue

        # Trust weight: direct friends count more than distant connections
        hop_distance  = graph.shortest_path_length(user, reviewer)
        trust_weight  = relationship_strength / (hop_distance ** 2)

        # Taste similarity: how aligned are their palates?
        taste_sim = cosine_similarity(
            user.taste_vector,
            reviewer.taste_vector
        )

        # Recency: fresh reviews matter more
        days_ago      = (today - review.date).days
        recency       = math.exp(-0.05 * days_ago)

        # Combined weight
        weight        = alpha * trust_weight + beta * taste_sim + gamma * recency
        total_score  += weight * review.rating
        total_weight += weight

    return total_score / total_weight if total_weight > 0 else None
```

---

## 🛠️ Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Backend** | Python + FastAPI | Async-first, clean type hints, fast prototyping |
| **Graph Engine** | NetworkX (dev) / Neo4j (prod) | Native graph traversal, Cypher query language |
| **Database** | PostgreSQL | Relational data for users, venues, reviews |
| **Cache** | Redis | Real-time feed, poll state, session management |
| **ML / Vectors** | NumPy + scikit-learn | Taste-palette cosine similarity |
| **Maps** | Google Maps API | Venue geocoding + map view |
| **Media** | AWS S3 / GCP Cloud Storage | Photo uploads |
| **Auth** | JWT + OAuth2 (Google, Apple) | Mobile-friendly auth |

---

## 🚀 Getting Started

```bash
# Clone the repo
git clone https://github.com/avikshath/near-me.git
cd near-me

# Set up virtual environment
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Fill in: DATABASE_URL, REDIS_URL, GOOGLE_MAPS_API_KEY, SECRET_KEY

# Run database migrations
alembic upgrade head

# Seed sample data (optional)
python scripts/seed_data.py

# Start the API server
uvicorn app.main:app --reload --port 8000

# API docs available at:
# http://localhost:8000/docs
```

---

## 📁 Project Structure

```
near-me/
├── app/
│   ├── api/
│   │   ├── routes/
│   │   │   ├── auth.py          # Login, signup, OAuth
│   │   │   ├── users.py         # Profiles, follow, taste palette
│   │   │   ├── restaurants.py   # Venue search, detail, reviews
│   │   │   ├── feed.py          # Personalized social feed
│   │   │   └── polls.py         # Group planning & voting
│   ├── core/
│   │   ├── graph/
│   │   │   ├── engine.py        # Graph traversal + scoring
│   │   │   ├── similarity.py    # Taste-palette cosine similarity
│   │   │   └── weights.py       # Trust weight + recency decay
│   │   ├── recommendations.py   # Main recommendation orchestrator
│   │   └── config.py
│   ├── models/                  # SQLAlchemy ORM models
│   ├── schemas/                 # Pydantic request/response schemas
│   └── main.py
├── tests/
│   ├── test_graph_engine.py
│   ├── test_recommendations.py
│   └── test_api.py
├── scripts/
│   └── seed_data.py
├── alembic/                     # DB migrations
├── requirements.txt
└── README.md
```

---

## 🗺️ Roadmap

- [x] Core graph recommendation engine
- [x] User profiles + taste-palette system
- [x] Restaurant rating + review flow
- [x] Social follow + feed
- [ ] Group polls + planning coordination
- [ ] Mobile apps (React Native)
- [ ] Multi-hop discovery (friends-of-friends)
- [ ] Taste-palette ML model (collaborative filtering upgrade)
- [ ] Real-time notifications (WebSocket)
- [ ] Venue onboarding / partnerships

---

## 🆚 Why Not Just Use Yelp?

| | Yelp / Google Maps | Near Me |
|---|---|---|
| **Trust source** | Anonymous strangers | Your actual social circle |
| **Ranking signal** | Review count + recency | Relationship strength + taste match |
| **Discovery quality** | High noise, low relevance | Low noise, high personal relevance |
| **Social layer** | None | Native — polls, planning, shared lists |
| **Taste modeling** | None | Explicit taste-palette profiles |
| **Manipulation risk** | High (fake reviews) | Low (social accountability) |

---

## 👤 Author

**Avikshath Reddy Velapati**
[LinkedIn](https://linkedin.com/in/avikshath) · [Email](mailto:velapatiavikshathreddy@gmail.com)

---

*Near Me is a personal project exploring trust-graph recommendation systems and social-first product design.*
