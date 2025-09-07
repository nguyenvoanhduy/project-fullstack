# project-fullstack
# 📂 Project Template: Node.js + Express + PostgreSQL + React

```
project-template/
│── backend/
│   ├── src/
│   │   ├── index.js            # entry point Express app
│   │   ├── routes/userRoutes.js
│   │   ├── controllers/userController.js
│   │   └── db.js               # PostgreSQL connection
│   ├── package.json
│   └── .env.example            # DB connection env vars
│
│── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── components/UserList.jsx
│   │   └── main.jsx
│   ├── package.json
│   └── vite.config.js
│
│── docker-compose.yml          # FE + BE + DB services
│── .gitignore
│── README.md                   # setup guide
```

---

## 📌 Backend (Express + PostgreSQL)
**backend/package.json**
```json
{
  "name": "backend",
  "version": "1.0.0",
  "main": "src/index.js",
  "scripts": {
    "dev": "nodemon src/index.js"
  },
  "dependencies": {
    "dotenv": "^16.4.0",
    "express": "^4.19.2",
    "pg": "^8.12.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

**backend/.env.example**
```
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=db
DB_PORT=5432
DB_NAME=app_db
```

**backend/src/db.js**
```js
const { Pool } = require("pg");
require("dotenv").config();

const pool = new Pool({
  user: process.env.DB_USER,
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  password: process.env.DB_PASSWORD,
  port: process.env.DB_PORT,
});

module.exports = pool;
```

**backend/src/index.js**
```js
const express = require("express");
const cors = require("cors");
const userRoutes = require("./routes/userRoutes");

const app = express();
app.use(cors());
app.use(express.json());

app.get("/health", (req, res) => res.json({ status: "ok" }));
app.use("/api/users", userRoutes);

const PORT = 5000;
app.listen(PORT, () => console.log(`Backend running on port ${PORT}`));
```

**backend/src/routes/userRoutes.js**
```js
const express = require("express");
const { getUsers } = require("../controllers/userController");

const router = express.Router();
router.get("/", getUsers);

module.exports = router;
```

**backend/src/controllers/userController.js**
```js
const pool = require("../db");

async function getUsers(req, res) {
  try {
    const result = await pool.query("SELECT id, name FROM users");
    res.json(result.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Database error" });
  }
}

module.exports = { getUsers };
```

---

## 📌 Frontend (React + Vite + Tailwind)
**frontend/package.json**
```json
{
  "name": "frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.0",
    "vite": "^5.2.0",
    "tailwindcss": "^3.4.0"
  }
}
```

**frontend/vite.config.js**
```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
  },
});
```

**frontend/tailwind.config.js**
```js
export default {
  content: ["./index.html", "./src/**/*.{js,jsx}"],
  theme: { extend: {} },
  plugins: [],
};
```

**frontend/src/App.jsx**
```jsx
import UserList from "./components/UserList";

function App() {
  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold">User List</h1>
      <UserList />
    </div>
  );
}

export default App;
```

**frontend/src/components/UserList.jsx**
```jsx
import { useEffect, useState } from "react";

function UserList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch("http://localhost:5000/api/users")
      .then((res) => res.json())
      .then((data) => setUsers(data));
  }, []);

  return (
    <ul className="mt-2 list-disc pl-5">
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}

export default UserList;
```

**frontend/src/main.jsx**
```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

**frontend/index.css**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## 📌 Docker Compose
**docker-compose.yml**
```yaml
version: "3.9"
services:
  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_db
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

  backend:
    build: ./backend
    command: npm run dev
    ports:
      - "5000:5000"
    environment:
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - DB_HOST=db
      - DB_PORT=5432
      - DB_NAME=app_db
    volumes:
      - ./backend:/app
    depends_on:
      - db

  frontend:
    build: ./frontend
    command: npm run dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
    depends_on:
      - backend

volumes:
  db_data:
```

---

## 📌 README.md (hướng dẫn ngắn)
```md
# Fullstack Template

## Cài đặt
```bash
git clone <this-repo>
cd project-template
cp backend/.env.example backend/.env
docker-compose up --build
```

- FE chạy tại: http://localhost:3000
- BE chạy tại: http://localhost:5000
- DB chạy tại: localhost:5432 (Postgres)
