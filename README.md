# -Smart-Task-Expense-Tracker

 Add a README.md File
Here is a complete, full-stack Smart Task & Expense Tracker built using Python (Flask) for the backend, SQLite for persistent storage, and HTML, CSS, and Vanilla JavaScript for a responsive, modern frontend—complete with embedded SVG graphics so no external image downloads are required.

1. Project Folder Structure

Create a folder named smart_tracker and organize your files exactly like this:
Show Code
Download diagram SVG

2. Visual Assets (static/images/logo.svg)

Save this crisp vector graphic as static/images/logo.svg. It serves as your application's brand icon and hero badge without relying on broken external image links.
xml
<svg xmlns="http://w3.org" viewBox="0 0 120 120" width="120" height="120">
  <defs>
    <linearGradient id="grad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#4f46e5" />
      <stop offset="100%" stop-color="#06b6d4" />
    </linearGradient>
  </defs>
  <rect width="120" height="120" rx="28" fill="url(#grad)" />
  <path d="M36 62 L52 78 L86 42" fill="none" stroke="#ffffff" stroke-width="10" stroke-linecap="round" stroke-linejoin="round" />
  <circle cx="88" cy="84" r="10" fill="#fbbf24" />
</svg>
Use code with caution.

3. Backend Code (Python + Flask + SQLite)


requirements.txt

text
Flask==3.0.3
Use code with caution.

app.py

This file sets up the REST API endpoints (GET, POST, PUT, DELETE), automatically initializes a local SQLite database (tracker.db), and serves the frontend.
python
import sqlite3
from flask import Flask, render_template, request, jsonify

app = Flask(__name__)
DB_NAME = "tracker.db"

def init_db():
    with sqlite3.connect(DB_NAME) as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS items (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL,
                category TEXT NOT NULL,
                amount REAL DEFAULT 0,
                completed INTEGER DEFAULT 0
            )
        """)
        conn.commit()

@app.route("/")
def index():
    return render_template("index.html")

@app.route("/api/items", methods=["GET"])
def get_items():
    with sqlite3.connect(DB_NAME) as conn:
        conn.row_factory = sqlite3.Row
        rows = conn.execute("SELECT * FROM items ORDER BY id DESC").fetchall()
        return jsonify([dict(row) for row in rows])

@app.route("/api/items", methods=["POST"])
def add_item():
    data = request.get_json()
    title = data.get("title", "").strip()
    category = data.get("category", "General")
    amount = float(data.get("amount", 0))

    if not title:
        return jsonify({"error": "Title is required"}), 400

    with sqlite3.connect(DB_NAME) as conn:
        cursor = conn.execute(
            "INSERT INTO items (title, category, amount, completed) VALUES (?, ?, ?, 0)",
            (title, category, amount)
        )
        conn.commit()
        new_id = cursor.lastrowid

    return jsonify({"id": new_id, "title": title, "category": category, "amount": amount, "completed": 0}), 201

@app.route("/api/items/<int:item_id>", methods=["PUT"])
def toggle_item(item_id):
    with sqlite3.connect(DB_NAME) as conn:
        conn.execute("UPDATE items SET completed = 1 - completed WHERE id = ?", (item_id,))
        conn.commit()
    return jsonify({"status": "updated"})

@app.route("/api/items/<int:item_id>", methods=["DELETE"])
def delete_item(item_id):
    with sqlite3.connect(DB_NAME) as conn:
        conn.execute("DELETE FROM items WHERE id = ?", (item_id,))
        conn.commit()
    return jsonify({"status": "deleted"})

if __name__ == "__main__":
    init_db()
    app.run(debug=True, port=5000)
Use code with caution.

4. Frontend Structure (templates/index.html)

Uses Jinja2 static asset routing to load the SVG logo, stylesheet, and JavaScript controller cleanly.
html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Task & Budget Pulse</title>
  <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}" />
</head>
<body>
  <div class="app-container">
    <header class="app-header">
      <div class="brand">
        <img src="{{ url_for('static', filename='images/logo.svg') }}" alt="App Logo" class="logo" />
        <div>
          <h1>Task & Budget Pulse</h1>
          <p>Full-Stack Python, HTML, CSS & JS Manager</p>
        </div>
      </div>
      <div class="summary-cards">
        <div class="card">
          <span>Active Tasks</span>
          <strong id="active-count">0</strong>
        </div>
        <div class="card">
          <span>Total Budget</span>
          <strong id="total-amount">₹0.00</strong>
        </div>
      </div>
    </header>

    <section class="form-section">
      <form id="item-form">
        <input type="text" id="title" placeholder="Enter task or expense title..." required />
        <select id="category">
          <option value="Work">Work</option>
          <option value="Personal">Personal</option>
          <option value="Shopping">Shopping</option>
          <option value="Travel">Travel</option>
        </select>
        <input type="number" id="amount" placeholder="Cost (₹)" min="0" step="0.01" value="0" />
        <button type="submit">Add Entry</button>
      </form>
    </section>

    <section class="filter-bar">
      <button class="filter-btn active" data-filter="ALL">All</button>
      <button class="filter-btn" data-filter="PENDING">Pending</button>
      <button class="filter-btn" data-filter="COMPLETED">Completed</button>
    </section>

    <main>
      <ul id="item-list" class="item-list"></ul>
    </main>
  </div>

  <script src="{{ url_for('static', filename='js/app.js') }}"></script>
</body>
</html>
Use code with caution.

5. Styling (static/css/style.css)

Modern CSS Grid and Flexbox layout with clean card elevation, responsive mobile breakpoints, and visual state indicators.
css
:root {
  --bg: #0f172a;
  --surface: #1e293b;
  --surface-hover: #334155;
  --primary: #4f46e5;
  --primary-hover: #4338ca;
  --accent: #06b6d4;
  --text: #f8fafc;
  --muted: #94a3b8;
  --danger: #ef4444;
  --success: #10b981;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
}

body {
  background-color: var(--bg);
  color: var(--text);
  min-height: 100vh;
  padding: 2rem 1rem;
}

.app-container {
  max-width: 850px;
  margin: 0 auto;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1.5rem;
  background: var(--surface);
  padding: 1.5rem;
  border-radius: 16px;
  margin-bottom: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.08);
}

.brand {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo {
  width: 56px;
  height: 56px;
}

.brand h1 {
  font-size: 1.4rem;
}

.brand p {
  color: var(--muted);
  font-size: 0.9rem;
}

.summary-cards {
  display: flex;
  gap: 1rem;
}

.card {
  background: var(--bg);
  padding: 0.8rem 1.2rem;
  border-radius: 12px;
  text-align: right;
  min-width: 120px;
}

.card span {
  display: block;
  font-size: 0.75rem;
  color: var(--muted);
  text-transform: uppercase;
}

.card strong {
  font-size: 1.25rem;
  color: var(--accent);
}

.form-section form {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr auto;
  gap: 0.75rem;
  background: var(--surface);
  padding: 1rem;
  border-radius: 12px;
  margin-bottom: 1.25rem;
}

input, select, button {
  padding: 0.75rem 1rem;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: var(--bg);
  color: var(--text);
  font-size: 0.95rem;
}

button[type="submit"] {
  background: var(--primary);
  border: none;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

button[type="submit"]:hover {
  background: var(--primary-hover);
}

.filter-bar {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.filter-btn {
  background: var(--surface);
  cursor: pointer;
  padding: 0.5rem 1rem;
  font-size: 0.85rem;
}

.filter-btn.active {
  background: var(--primary);
  border-color: var(--primary);
}

.item-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.item-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--surface);
  padding: 1rem 1.25rem;
  border-radius: 12px;
  transition: transform 0.15s;
}

.item-card:hover {
  background: var(--surface-hover);
}

.item-card.completed .item-title {
  text-decoration: line-through;
  color: var(--muted);
}

.item-info {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.badge {
  font-size: 0.75rem;
  padding: 0.25rem 0.6rem;
  border-radius: 999px;
  background: rgba(6, 182, 212, 0.15);
  color: var(--accent);
}

.item-actions {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.amount-tag {
  font-weight: 600;
  color: #fbbf24;
}

.btn-icon {
  padding: 0.4rem 0.7rem;
  font-size: 0.85rem;
  cursor: pointer;
}

.btn-check {
  color: var(--success);
  border-color: var(--success);
}

.btn-delete {
  color: var(--danger);
  border-color: var(--danger);
}

@media (max-width: 680px) {
  .form-section form {
    grid-template-columns: 1fr;
  }
}
Use code with caution.

6. Client-Side Logic (static/js/app.js)

Handles asynchronous fetch() calls to the Python Flask API, updates the DOM without page reloads, and filters tasks dynamically.
javascript
let allItems = [];
let currentFilter = "ALL";

const itemForm = document.getElementById("item-form");
const itemList = document.getElementById("item-list");
const activeCountEl = document.getElementById("active-count");
const totalAmountEl = document.getElementById("total-amount");
const filterButtons = document.querySelectorAll(".filter-btn");

async function fetchItems() {
  const res = await fetch("/api/items");
  allItems = await res.json();
  renderItems();
}

function renderItems() {
  itemList.innerHTML = "";

  const filtered = allItems.filter((item) => {
    if (currentFilter === "PENDING") return item.completed === 0;
    if (currentFilter === "COMPLETED") return item.completed === 1;
    return true;
  });

  let activeCount = 0;
  let totalBudget = 0;

  allItems.forEach((item) => {
    if (item.completed === 0) activeCount++;
    totalBudget += Number(item.amount || 0);
  });

  activeCountEl.textContent = activeCount;
  totalAmountEl.textContent = `₹${totalBudget.toFixed(2)}`;

  filtered.forEach((item) => {
    const li = document.createElement("li");
    li.className = `item-card ${item.completed ? "completed" : ""}`;

    li.innerHTML = `
      <div class="item-info">
        <span class="badge">${item.category}</span>
        <span class="item-title">${escapeHTML(item.title)}</span>
      </div>
      <div class="item-actions">
        <span class="amount-tag">₹${Number(item.amount).toFixed(2)}</span>
        <button class="btn-icon btn-check" onclick="toggleItem(${item.id})">
          ${item.completed ? "Undo" : "Done"}
        </button>
        <button class="btn-icon btn-delete" onclick="deleteItem(${item.id})">
          Delete
        </button>
      </div>
    `;
    itemList.appendChild(li);
  });
}

itemForm.addEventListener("submit", async (e) => {
  e.preventDefault();
  const title = document.getElementById("title").value;
  const category = document.getElementById("category").value;
  const amount = document.getElementById("amount").value;

  await fetch("/api/items", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title, category, amount })
  });

  itemForm.reset();
  fetchItems();
});

async function toggleItem(id) {
  await fetch(`/api/items/${id}`, { method: "PUT" });
  fetchItems();
}

async function deleteItem(id) {
  await fetch(`/api/items/${id}`, { method: "DELETE" });
  fetchItems();
}

filterButtons.forEach((btn) => {
  btn.addEventListener("click", () => {
    filterButtons.forEach((b) => b.classList.remove("active"));
    btn.classList.add("active");
    currentFilter = btn.dataset.filter;
    renderItems();
  });
});

function escapeHTML(str) {
  const div = document.createElement("div");
  div.textContent = str;
  return div.innerHTML;
}

fetchItems();
Use code with caution.

7. How to Run Locally

1. Open your terminal inside the smart_tracker folder.
2. Install Flask using pip:bash
pip install -r requirements.txt
Use code with caution.
3. Start the Flask server:bash
python app.py
Use code with caution.
4. Open http://127.0.0.1:5000 in your browser.
If you'd like to expand this project, let me know if you want:
• A Dockerfile and CI/CD deployment setup (such as Azure App Service or Render)
• User authentication (login/register with hashed passwords)
• Data visualization charts using Chart.js for expense breakdowns
Try without personalization

mern full stack project

Here is a complete, full-stack MERN (MongoDB, Express, React, Node.js) Inventory & Order Manager—including database schemas, REST APIs, React UI components, styling, and embedded SVG graphics so every visual loads out of the box.

1. Project Folder Structure

Create a root directory named mern_pulse and organize it into server and client folders:
Show Code
Download diagram SVG

2. Backend (MongoDB + Express + Node.js)


server/package.json

json
{
  "name": "mern-pulse-server",
  "version": "1.0.0",
  "main": "server.js",
  "type": "module",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "mongoose": "^8.4.0"
  }
}
Use code with caution.

server/.env

env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/mern_pulse
Use code with caution.

server/models/Product.js

Defines the Mongoose schema with stock validation and automatic timestamps.
javascript
import mongoose from "mongoose";

const productSchema = new mongoose.Schema(
  {
    name: { type: String, required: true, trim: true },
    category: { type: String, required: true, default: "Electronics" },
    price: { type: Number, required: true, min: 0 },
    stock: { type: Number, required: true, min: 0, default: 1 }
  },
  { timestamps: true }
);

export default mongoose.model("Product", productSchema);
Use code with caution.

server/routes/productRoutes.js

Handles CRUD operations (GET, POST, PATCH for stock updates, and DELETE).
javascript
import express from "express";
import Product from "../models/Product.js";

const router = express.Router();

// Get all products
router.get("/", async (req, res) => {
  try {
    const products = await Product.find().sort({ createdAt: -1 });
    res.json(products);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Create a product
router.post("/", async (req, res) => {
  try {
    const { name, category, price, stock } = req.body;
    const product = await Product.create({ name, category, price, stock });
    res.status(201).json(product);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// Adjust stock count (+1 or -1)
router.patch("/:id/stock", async (req, res) => {
  try {
    const { delta } = req.body;
    const product = await Product.findById(req.params.id);
    if (!product) return res.status(404).json({ error: "Product not found" });

    product.stock = Math.max(0, product.stock + Number(delta));
    await product.save();
    res.json(product);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// Delete a product
router.delete("/:id", async (req, res) => {
  try {
    await Product.findByIdAndDelete(req.params.id);
    res.json({ message: "Deleted" });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

export default router;
Use code with caution.

server/server.js

javascript
import express from "express";
import mongoose from "mongoose";
import cors from "cors";
import dotenv from "dotenv";
import productRoutes from "./routes/productRoutes.js";

dotenv.config();
const app = express();

app.use(cors());
app.use(express.json());
app.use("/api/products", productRoutes);

mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    console.log("Connected to MongoDB");
    app.listen(process.env.PORT || 5000, () =>
      console.log(`Server running on port ${process.env.PORT || 5000}`)
    );
  })
  .catch((err) => console.error("MongoDB connection error:", err));
Use code with caution.

3. Visual Asset (client/public/logo.svg)

Save this crisp vector logo inside client/public/logo.svg:
xml
<svg xmlns="http://w3.org" viewBox="0 0 120 120" width="120" height="120">
  <defs>
    <linearGradient id="mernGrad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#10b981" />
      <stop offset="100%" stop-color="#3b82f6" />
    </linearGradient>
  </defs>
  <rect width="120" height="120" rx="26" fill="url(#mernGrad)" />
  <polygon points="60,26 92,44 92,76 60,94 28,76 28,44" fill="none" stroke="#ffffff" stroke-width="7" stroke-linejoin="round" />
  <polyline points="28,44 60,62 92,44" fill="none" stroke="#ffffff" stroke-width="7" stroke-linejoin="round" />
  <line x1="60" y1="62" x2="60" y2="94" stroke="#ffffff" stroke-width="7" />
</svg>
Use code with caution.

4. Frontend (React + Vite)


client/package.json

json
{
  "name": "mern-pulse-client",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.1",
    "vite": "^5.3.1"
  }
}
Use code with caution.

client/vite.config.js

Configures a proxy so /api requests automatically route to your Express server without CORS issues during development.
javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      "/api": "http://localhost:5000"
    }
  }
});
Use code with caution.

client/index.html

html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>MERN Inventory Pulse</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
Use code with caution.

client/src/main.jsx

jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import "./App.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
Use code with caution.

client/src/App.jsx

jsx
import { useEffect, useState } from "react";

export default function App() {
  const [products, setProducts] = useState([]);
  const [search, setSearch] = useState("");
  const [form, setForm] = useState({
    name: "",
    category: "Electronics",
    price: "",
    stock: 1
  });

  const fetchProducts = async () => {
    const res = await fetch("/api/products");
    const data = await res.json();
    setProducts(data);
  };

  useEffect(() => {
    fetchProducts();
  }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!form.name.trim()) return;

    await fetch("/api/products", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        ...form,
        price: Number(form.price),
        stock: Number(form.stock)
      })
    });

    setForm({ name: "", category: "Electronics", price: "", stock: 1 });
    fetchProducts();
  };

  const adjustStock = async (id, delta) => {
    await fetch(`/api/products/${id}/stock`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ delta })
    });
    fetchProducts();
  };

  const deleteProduct = async (id) => {
    await fetch(`/api/products/${id}`, { method: "DELETE" });
    fetchProducts();
  };

  const filtered = products.filter((p) =>
    p.name.toLowerCase().includes(search.toLowerCase())
  );

  const totalValue = products.reduce((sum, p) => sum + p.price * p.stock, 0);
  const totalUnits = products.reduce((sum, p) => sum + p.stock, 0);

  return (
    <div className="container">
      <header className="header">
        <div className="brand">
          <img src="/logo.svg" alt="MERN Pulse Logo" className="logo" />
          <div>
            <h1>MERN Inventory Pulse</h1>
            <p>MongoDB • Express • React • Node.js</p>
          </div>
        </div>
        <div className="stats">
          <div className="stat-box">
            <span>Total Units</span>
            <strong>{totalUnits}</strong>
          </div>
          <div className="stat-box">
            <span>Stock Value</span>
            <strong>₹{totalValue.toFixed(2)}</strong>
          </div>
        </div>
      </header>

      <form className="product-form" onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Product name..."
          value={form.name}
          onChange={(e) => setForm({ ...form, name: e.target.value })}
          required
        />
        <select
          value={form.category}
          onChange={(e) => setForm({ ...form, category: e.target.value })}
        >
          <option value="Electronics">Electronics</option>
          <option value="Accessories">Accessories</option>
          <option value="Office">Office</option>
          <option value="Hardware">Hardware</option>
        </select>
        <input
          type="number"
          placeholder="Price (₹)"
          min="0"
          step="0.01"
          value={form.price}
          onChange={(e) => setForm({ ...form, price: e.target.value })}
          required
        />
        <input
          type="number"
          placeholder="Stock"
          min="0"
          value={form.stock}
          onChange={(e) => setForm({ ...form, stock: e.target.value })}
          required
        />
        <button type="submit">Add Product</button>
      </form>

      <div className="toolbar">
        <input
          type="search"
          placeholder="Search inventory by name..."
          value={search}
          onChange={(e) => setSearch(e.target.value)}
        />
      </div>

      <div className="grid">
        {filtered.map((item) => (
          <div key={item._id} className="card">
            <div className="card-top">
              <span className="tag">{item.category}</span>
              <span className={`stock-badge ${item.stock === 0 ? "out" : ""}`}>
                {item.stock === 0 ? "Out of Stock" : `${item.stock} in stock`}
              </span>
            </div>
            <h3>{item.name}</h3>
            <p className="price">₹{item.price.toFixed(2)}</p>
            <div className="card-actions">
              <div className="qty-controls">
                <button onClick={() => adjustStock(item._id, -1)}>-</button>
                <span>{item.stock}</span>
                <button onClick={() => adjustStock(item._id, 1)}>+</button>
              </div>
              <button
                className="btn-delete"
                onClick={() => deleteProduct(item._id)}
              >
                Delete
              </button>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
Use code with caution.

client/src/App.css

css
:root {
  --bg: #0b1120;
  --card: #1e293b;
  --border: rgba(255, 255, 255, 0.08);
  --primary: #10b981;
  --primary-hover: #059669;
  --text: #f8fafc;
  --muted: #94a3b8;
  --danger: #ef4444;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', system-ui, sans-serif;
}

body {
  background: var(--bg);
  color: var(--text);
  padding: 2rem 1rem;
}

.container {
  max-width: 960px;
  margin: 0 auto;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1.5rem;
  background: var(--card);
  padding: 1.5rem;
  border-radius: 16px;
  border: 1px solid var(--border);
  margin-bottom: 1.5rem;
}

.brand {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo {
  width: 54px;
  height: 54px;
}

.brand h1 {
  font-size: 1.4rem;
}

.brand p {
  color: var(--muted);
  font-size: 0.85rem;
}

.stats {
  display: flex;
  gap: 1rem;
}

.stat-box {
  background: var(--bg);
  padding: 0.75rem 1.25rem;
  border-radius: 12px;
  text-align: right;
}

.stat-box span {
  display: block;
  font-size: 0.75rem;
  color: var(--muted);
  text-transform: uppercase;
}

.stat-box strong {
  font-size: 1.25rem;
  color: var(--primary);
}

.product-form {
  display: grid;
  grid-template-columns: 2fr 1.2fr 1fr 0.8fr auto;
  gap: 0.75rem;
  background: var(--card);
  padding: 1rem;
  border-radius: 12px;
  margin-bottom: 1rem;
}

input, select, button {
  padding: 0.7rem 0.9rem;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: var(--bg);
  color: var(--text);
  font-size: 0.95rem;
}

button[type="submit"] {
  background: var(--primary);
  color: #052e16;
  font-weight: 700;
  border: none;
  cursor: pointer;
}

button[type="submit"]:hover {
  background: var(--primary-hover);
}

.toolbar {
  margin-bottom: 1.25rem;
}

.toolbar input {
  width: 100%;
  background: var(--card);
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 1rem;
}

.card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.card-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.tag {
  font-size: 0.75rem;
  padding: 0.2rem 0.6rem;
  border-radius: 999px;
  background: rgba(59, 130, 246, 0.15);
  color: #60a5fa;
}

.stock-badge {
  font-size: 0.75rem;
  color: var(--primary);
}

.stock-badge.out {
  color: var(--danger);
}

.price {
  font-size: 1.35rem;
  font-weight: 700;
  color: #fbbf24;
}

.card-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: auto;
  padding-top: 0.75rem;
  border-top: 1px solid var(--border);
}

.qty-controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.qty-controls button {
  padding: 0.25rem 0.65rem;
  cursor: pointer;
}

.btn-delete {
  color: var(--danger);
  border-color: var(--danger);
  padding: 0.35rem 0.75rem;
  font-size: 0.85rem;
  cursor: pointer;
}

@media (max-width: 720px) {
  .product-form {
    grid-template-columns: 1fr;
  }
}
Use code with caution.

5. How to Run Locally

1. Start MongoDB: Ensure MongoDB Community Server is running locally (or paste a MongoDB Atlas connection string into server/.env).
2. Start the Express Backend:bash
cd server
npm install
npm start
Use code with caution.
3. Start the React Frontend (in a second terminal):bash
cd client
npm install
npm run dev
Use code with caution.
4. Open http://localhost:5173 in your browser.
If you'd like to extend this MERN application, let me know if you want:
• JWT Authentication (user signup/login with protected routes and role-based access)
• Image uploads using Multer or Cloudinary so each product displays a custom photo
• Docker Compose configuration to run MongoDB, Express, and React with a single command
using python,html,css,and js project with everything aslo code and images - Google Search.html
File
github
To push your MERN Inventory Pulse project to GitHub cleanly (without uploading heavy node_modules or sensitive .env credentials), follow these steps:

1. Create a .gitignore File

Create a file named .gitignore inside the root mern_pulse/ directory so Git ignores dependencies and environment secrets:
text
# Dependencies
node_modules/

# Environment variables
.env

# Build outputs
client/dist/
.vite/

# OS & Editor files
.DS_Store
.vscode/
Use code with caution.

2. Add a README.md File

Create README.md in mern_pulse/ to document your project on your GitHub repository page:
markdown
# MERN Inventory Pulse

A full-stack inventory and stock management web application built with **MongoDB, Express, React, and Node.js**.

## Tech Stack
- **Frontend:** React 18, Vite, CSS3
- **Backend:** Node.js, Express
- **Database:** MongoDB & Mongoose

## Local Setup
1. **Backend:**
   ```bash
   cd server
   npm install
   # Create a .env file with PORT=5000 and MONGO_URI=your_mongodb_uri
   npm start
   ```
2. **Frontend:**
   ```bash
   cd client
   npm install
   npm run dev
   ```
Use code with caution.

3. Initialize Git & Push to GitHub

1. Create a new repository on GitHub (do not initialize it with a README or .gitignore since you just created them).
2. Open your terminal inside the root mern_pulse/ folder and run:
bash
git init
git add .
git commit -m "Initial commit: MERN Inventory Pulse full-stack app"
git branch -M main
git remote add origin https://github.com<YOUR_USERNAME>/<YOUR_REPO_NAME>.git
git push -u origin main
Use code with caution.
(Replace <YOUR_USERNAME> and <YOUR_REPO_NAME> with your actual GitHub username and repository URL.)
If you'd like to automate or enhance your GitHub repository, let me know if you want:
• A GitHub Actions CI workflow (.github/workflows/ci.yml) to automatically test and build your React & Node code on every push
• A .env.example template and Docker Compose setup for one-click deployment
• Troubleshooting help if you hit an authentication or push error in your terminal
