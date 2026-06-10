# Anvaya CRM – Backend

A REST API for a Customer Relationship Management application. It powers lead tracking, sales agent management, comments, tags, and pipeline reporting, built with Express and MongoDB (Mongoose).

## Live API

https://crm-backend-wlhu.vercel.app

> Base URL for all endpoints below. When running locally, use `http://localhost:3000`.

## Tech Stack

| Layer        | Technology        |
|--------------|-------------------|
| Runtime      | Node.js           |
| Framework    | Express 5         |
| Database     | MongoDB           |
| ODM          | Mongoose 9        |
| Config       | dotenv            |
| CORS         | cors              |
| Module System| CommonJS          |

---

## Features

- **Lead Management** – Create, read, update, delete, filter, and fuzzy-search leads.
- **Sales Agents** – Manage the sales team with unique-email enforcement.
- **Comments** – Attach comments to leads, authored by a sales agent.
- **Tags** – Create and list reusable, uniquely-named tags.
- **Pipeline Reports** – Count open leads and list leads closed in the last 7 days.
- **Input Validation** – Required-field checks, enum validation, and ObjectId format validation on requests.

---

## Data Models

### Lead
| Field         | Type       | Notes                                                                                   |
|---------------|------------|-----------------------------------------------------------------------------------------|
| `name`        | String     | Required                                                                                |
| `source`      | String     | Required. One of: `Website`, `Referral`, `Cold Call`, `Advertisement`, `Email`, `Other` |
| `salesAgent`  | ObjectId   | Required. References a `SalesAgent`                                                      |
| `status`      | String     | One of: `New`, `Contacted`, `Qualified`, `Proposal Sent`, `Closed`. Default: `New`      |
| `tags`        | [String]   | Optional array of tag names                                                              |
| `timeToClose` | Number     | Required. Must be ≥ 1                                                                    |
| `priority`    | String     | One of: `High`, `Medium`, `Low`. Default: `Medium`                                       |
| `closedAt`    | Date       | Set automatically when a lead's status is updated to `Closed`                            |
| `createdAt` / `updatedAt` | Date | Added automatically via timestamps                                              |

### SalesAgent
| Field       | Type   | Notes              |
|-------------|--------|--------------------|
| `name`      | String | Required           |
| `email`     | String | Required, unique   |
| `createdAt` | Date   | Defaults to now    |

### Comment
| Field         | Type     | Notes                               |
|---------------|----------|-------------------------------------|
| `lead`        | ObjectId | Required. References a `Lead`        |
| `author`      | ObjectId | Required. References a `SalesAgent`  |
| `commentText` | String   | Required                            |
| `createdAt`   | Date     | Defaults to now                     |

### Tag
| Field       | Type   | Notes            |
|-------------|--------|------------------|
| `name`      | String | Required, unique |
| `createdAt` | Date   | Defaults to now  |

---

## Getting Started

### Installation

```bash
git clone https://github.com/tanaymurade74/CRMBackend.git
cd CRMBackend
npm install
```

### Environment Variables

Create a `.env` file in the project root:

```env
MONGODB=your_mongodb_connection_string
```

> The app reads the connection string from the `MONGODB` variable. Use a MongoDB Atlas URI or a local instance such as `mongodb://localhost:27017/anvaya-crm`.

### Run the Server

```bash
node index.js
```

The server starts on **port 3000** → `http://localhost:3000`

> **Tip:** There's no `start` script defined yet. Add one to `package.json` so you can use `npm start`, and `nodemon` for auto-reload during development:
> ```json
> "scripts": {
>   "start": "node index.js",
>   "dev": "nodemon index.js"
> }
> ```

---

## Project Structure

```
.
├── db/
│   └── db.connect.js        # Mongoose connection setup (reads process.env.MONGODB)
├── models/
│   ├── lead.model.js        # Lead schema
│   ├── salesAgent.model.js  # SalesAgent schema
│   ├── comment.model.js     # Comment schema
│   └── tag.model.js         # Tag schema
├── index.js                 # Express app: middleware, route handlers, server bootstrap
├── package.json
└── .env                     # Not committed (see .gitignore)
```

---

## API Reference

All responses are JSON. Successful and error responses use varying top-level keys as noted below.

### Leads

| Method | Endpoint                    | Description                                                          |
|--------|-----------------------------|---------------------------------------------------------------------|
| GET    | `/leads`                    | List leads. Supports filtering (see query params below)             |
| GET    | `/leads/search/:searchTerm` | Fuzzy/partial search of leads by name                               |
| POST   | `/leads`                    | Create a lead                                                       |
| PUT    | `/leads/:id`                | Update a lead (sets `closedAt` automatically when status = `Closed`) |
| DELETE | `/leads/:id`                | Delete a lead                                                       |

**`GET /leads` query parameters** (all optional, combinable): `salesAgent` (ObjectId), `status` (must be a valid status), `tags`, `source`, `_id`.

**Create lead — request body:**
```json
{
  "name": "Acme Corp",
  "source": "Website",
  "salesAgent": "64f0c2a1e1b2c3d4e5f6a7b8",
  "tags": ["enterprise", "priority"],
  "timeToClose": 30,
  "priority": "High"
}
```

**Create lead — response (`201`):**
```json
{
  "Lead": {
    "_id": "...",
    "name": "Acme Corp",
    "source": "Website",
    "status": "New",
    "priority": "High",
    "timeToClose": 30,
    "createdAt": "...",
    "updatedAt": "..."
  }
}
```

`GET /leads` returns `{ "Leads": [ ... ] }`. Validation errors return `400`; a referenced sales agent that doesn't exist returns `404`.

### Sales Agents

| Method | Endpoint        | Description                                      |
|--------|-----------------|--------------------------------------------------|
| GET    | `/agents`       | List all sales agents                            |
| POST   | `/agents`       | Create a sales agent (`name`, `email` required)  |
| GET    | `/agents/:id`   | Get a sales agent by ID                          |
| DELETE | `/agents/:id`   | Delete a sales agent                             |

> Creating an agent with an email that already exists returns `409 Conflict`.

### Comments

| Method | Endpoint                  | Description                  |
|--------|---------------------------|------------------------------|
| POST   | `/leads/:id/comments`     | Add a comment to a lead      |
| GET    | `/leads/:id/comments`     | List all comments for a lead |
| DELETE | `/comments/:commentId`    | Delete a comment by ID       |

**Add comment — request body:**
```json
{
  "commentText": "Followed up via email.",
  "author": "64f0c2a1e1b2c3d4e5f6a7b8"
}
```

### Tags

| Method | Endpoint | Description                    |
|--------|----------|--------------------------------|
| POST   | `/tag`   | Create a tag (`name` required) |
| GET    | `/tag`   | List all tags                  |

### Reports

| Method | Endpoint               | Description                                                     |
|--------|------------------------|-----------------------------------------------------------------|
| GET    | `/report/last-week`    | List leads closed in the last 7 days                            |
| GET    | `/report/pipeline`     | Count of leads not yet `Closed` → `{ "totalPipelineLeads": n }` |

---

## Related Repositories

- **Frontend (React):** [https://github.com/tanaymurade74/CRM-Frontend](https://github.com/tanaymurade74/CRM-Frontend)
