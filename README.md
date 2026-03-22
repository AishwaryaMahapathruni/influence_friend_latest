# Vibely — Influence-Aware Social Platform

A full-stack social media platform that uses **graph ML algorithms** (PageRank + Adamic-Adar on the live follow network) to power a Smart Score ranking system and explainable "Suggested for You" recommendations.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Django 4.2 · Django REST Framework · MongoEngine |
| Database | MongoDB Atlas (cloud) |
| ML Engine | NumPy · SciPy · NetworkX (Adamic-Adar, Temporal PageRank) |
| Auth | JWT (custom MongoJWT backend) |
| Frontend | React 18 · Vite · Tailwind CSS |
| Real-time | Django Channels · Redis (WebSocket layer) |

---

## Project Structure

```
vibely/
├── vibely-backend/
│   ├── apps/
│   │   ├── accounts/       # Auth, registration, JWT, user model
│   │   ├── social/         # Follow, friend requests, recommendations
│   │   ├── posts/          # Feed, likes, comments, shares, favorites
│   │   ├── messaging/      # Conversations, messages, unread counts
│   │   ├── notifications/  # Notification feed, unread badge
│   │   ├── analytics/      # Smart Score computation, graph cache
│   │   ├── trending/       # Trending topics & viral posts
│   │   └── search/         # Unified user + post search
│   ├── ml/
│   │   ├── temporal_centrality.py      # TemporalPageRank + ActivityCentrality
│   │   ├── link_prediction_baselines.py # Adamic-Adar recommender
│   │   └── recommendation_explainer.py # Natural-language explanation engine
│   ├── vibely/
│   │   ├── settings/
│   │   │   ├── base.py     # Core settings, MongoDB connection, JWT config
│   │   │   └── local.py    # Dev overrides
│   │   └── urls.py         # Root URL routing
│   ├── manage.py
│   └── requirements.txt
│
└── vibely-frontend/
    ├── src/
    │   ├── pages/
    │   │   ├── FeedPage.jsx        # Home — posts feed + discover sidebar
    │   │   ├── DiscoverPage.jsx    # Suggested for You + Discover Creators
    │   │   ├── FriendsPage.jsx     # Friend requests, suggestions, all friends
    │   │   ├── MessagesPage.jsx    # Conversations + live user search
    │   │   ├── ProfilePage.jsx     # User profile, posts, follow/message
    │   │   ├── TrendingPage.jsx    # Trending topics & viral posts
    │   │   ├── AnalyticsPage.jsx   # Smart Score breakdown & stats
    │   │   └── SettingsPage.jsx    # Profile edit, password change
    │   ├── components/
    │   │   ├── layout/             # Navbar (search, notifications, badges)
    │   │   ├── posts/              # PostCard, CreatePostForm, CommentSection
    │   │   ├── discover/           # InfluencerCard, RecommendationCard, FilterTabs
    │   │   ├── profile/            # ProfileHeader
    │   │   └── common/             # Avatar, FollowButton, SmartScoreRing, ...
    │   ├── services/               # API clients (axios)
    │   └── utils/                  # formatters, constants
    └── package.json
```

---

## Features

### Social
- **Feed** — posts from followed users, own posts, paginated with Load More
- **Posts** — create with optional image, like (toggle), comment, share, bookmark
- **Follow system** — follow / unfollow with live follower counts
- **Friend requests** — send, accept, reject; mutual-request auto-accept
- **Messaging** — 1-on-1 conversations, unread badges, new-chat user search

### ML-Powered
- **Smart Score (0–100)** — 6-component weighted score:
  - Engagement (25%) · Reach (20%) · Activity (15%) · Growth (15%) · Diffusion (10%) · **Graph Influence (15%)**
  - Graph Influence uses Temporal PageRank + Activity Centrality on the follow graph
- **"Suggested for You"** — Adamic-Adar link prediction with natural-language explanations
  - Primary reason: *"👥 8 mutual connections"*, *"⚡ Top voice in Tech"*
  - Confidence badge: High match / Good match / Possible match
  - Excludes already-followed users and existing friends
- **Graph cache** — 15-minute TTL, invalidated on every follow/unfollow

### UX
- Real-time notification bell with unread count (polls every 30 s)
- Friend request badge on Navbar
- "Request Sent" state persists across page refreshes (loaded from server)
- Debounced search in Navbar and Messages new-chat
- Timestamps always shown in local time (UTC `Z` suffix on all API responses)

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js 18+
- MongoDB Atlas account (or local MongoDB 6+)
- Redis (for Django Channels; can skip for HTTP-only mode)

---

### Backend Setup

```bash
cd vibely/vibely-backend

# Create and activate virtual environment
python -m venv venv
# Windows
venv\Scripts\activate
# Mac / Linux
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

Create `vibely/.env`:

```env
SECRET_KEY=your-django-secret-key-here
DEBUG=True

# MongoDB Atlas
MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/?retryWrites=true&w=majority&tls=true
MONGODB_NAME=vibely_db

# Redis (optional — needed for WebSocket channels)
REDIS_HOST=127.0.0.1
REDIS_PORT=6379

# CORS — allow the frontend dev server
CORS_ALLOWED_ORIGINS=http://localhost:5173
```

> **MongoDB Atlas tip:** Go to *Network Access* → Add your current IP (or `0.0.0.0/0` for dev).

```bash
# Start the Django dev server
python manage.py runserver
```

API is available at `http://localhost:8000/api/v1/`

---

### Frontend Setup

```bash
cd vibely/vibely-frontend

npm install
npm run dev
```

Frontend runs at `http://localhost:5173`

Create `vibely-frontend/.env` if you need a different API URL:

```env
VITE_API_URL=http://localhost:8000/api/v1
```

---

## API Reference

### Auth — `/api/v1/auth/`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/register/` | Register a new user |
| POST | `/login/` | Login, returns `access` + `refresh` tokens |
| POST | `/refresh/` | Refresh access token |
| GET/PUT | `/me/` | Get / update own profile |
| POST | `/change-password/` | Change password |

### Users — `/api/v1/users/`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/discover/` | Browse creators sorted by Smart Score |
| GET | `/recommendations/` | ML-powered "Suggested for You" with reasons |
| GET | `/<id>/` | Get user profile |
| POST/DELETE | `/<id>/follow/` | Follow / unfollow |
| GET | `/<id>/followers/` | List followers |
| GET | `/<id>/following/` | List following |
| GET | `/friend-requests/` | Incoming pending friend requests |
| GET | `/friend-requests/?direction=sent` | Outgoing pending requests |
| POST | `/friend-requests/` | Send a friend request `{ user_id }` |
| POST | `/friend-requests/<id>/accept/` | Accept a friend request |
| POST | `/friend-requests/<id>/reject/` | Reject a friend request |
| GET | `/friends/` | All accepted friends |
| GET | `/<id>/friend-status/` | Friendship status with a user |

### Posts — `/api/v1/posts/`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Feed (followed + own posts) |
| POST | `/` | Create post (text or multipart with image) |
| GET/PUT/DELETE | `/<id>/` | Get / edit / delete post |
| POST | `/<id>/like/` | Toggle like |
| POST | `/<id>/share/` | Share |
| POST | `/<id>/favorite/` | Toggle bookmark |
| GET/POST | `/<id>/comments/` | List / add comments |
| GET | `/user/<user_id>/` | Get a user's posts |

### Messages — `/api/v1/messages/`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/conversations/` | List conversations |
| POST | `/conversations/` | Create or get conversation `{ user_id }` |
| GET/POST | `/conversations/<id>/` | Get messages / send message |
| POST | `/conversations/<id>/read/` | Mark as read |
| GET | `/unread-count/` | Total unread message count |

### Notifications — `/api/v1/notifications/`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | List notifications |
| GET | `/unread-count/` | Unread notification count |
| POST | `/mark-read/` | Mark specific IDs as read |
| POST | `/mark-all-read/` | Mark all as read |
| DELETE | `/clear/` | Delete all notifications |

### Analytics — `/api/v1/analytics/`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/overview/` | Smart Score breakdown for current user |

### Search — `/api/v1/search/`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/?q=<query>&type=users\|posts\|all` | Unified search |

---

## Smart Score — How It Works

```
Smart Score (0–100) = weighted sum of 6 components
  ├── Engagement Rate  × 0.25   (likes + comments per post)
  ├── Reach Score      × 0.20   (followers relative to category peers)
  ├── Activity Score   × 0.15   (posting frequency & consistency)
  ├── Growth Rate      × 0.15   (follower growth over time)
  ├── Diffusion Score  × 0.10   (shares & re-engagement)
  └── Graph Influence  × 0.15   (Temporal PageRank on follow graph)
```

Graph Influence is computed by `apps/analytics/graph_cache.py`:
1. Reads all `Follow` documents from MongoDB
2. Builds numpy edge arrays with timestamps
3. Runs `TemporalPageRank` + `ActivityBasedInfluence` (`ml/temporal_centrality.py`)
4. Results are cached for 15 minutes; cache is invalidated on every follow/unfollow

---

## Recommendations — How They Work

```
MongoDB Follow documents
  ↓  graph_builder.py
numpy edge arrays + neighbor dict
  ↓  ml/link_prediction_baselines.py (AdamicAdar)
Scored candidate pairs (users not yet followed)
  ↓  ml/recommendation_explainer.py
Human-readable reasons per recommendation
  ↓  GET /api/v1/users/recommendations/
"Suggested for You" cards with reason pills
```

Fallback: if the user is not yet in the graph (new account), returns top users by Smart Score excluding already-followed users and existing friends.

---

## Notification Types

| Type | Trigger |
|---|---|
| `follow` | Someone follows you |
| `like` | Someone likes your post |
| `comment` | Someone comments on your post |
| `share` | Someone shares your post |
| `message` | New message received |
| `friend_request` | Someone sends you a friend request |
| `friend_accept` | Someone accepts your friend request |

---

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `SECRET_KEY` | (required) | Django secret key |
| `DEBUG` | `False` | Enable debug mode |
| `MONGODB_URI` | `''` | MongoDB Atlas connection string |
| `MONGODB_NAME` | `vibely_db` | Database name |
| `MONGODB_HOST` | `localhost` | Local MongoDB host (if no URI) |
| `MONGODB_PORT` | `27017` | Local MongoDB port |
| `REDIS_HOST` | `127.0.0.1` | Redis host for Channels |
| `REDIS_PORT` | `6379` | Redis port |

---

## License

MIT




# for simple execution 

frontend

1.cd vibely
cd vibely-frontend
npm install
npm run dev


2.backend  --->another terminal 

cd vibely-backend

python -m venv venv
venv\scripts\activate
pip install -r requirements.txt

python manage.py makemigrations
python manage.py migrate
python manage.py runserver

