# Smart Task & Expense Tracker

A full-stack task and budget management web application built with **Python (Flask)**, **SQLite**, **HTML5**, **CSS3**, and **Vanilla JavaScript**.

## Features
- Add, filter (`All`, `Pending`, `Completed`), toggle, and delete tasks and expenses
- Real-time active task counter and total budget calculation
- Persistent local storage powered by SQLite
- Responsive dark-mode UI with embedded vector graphics

## Project Structure
```text
smart_tracker/
├── app.py
├── requirements.txt
├── templates/
│   └── index.html
└── static/
    ├── css/
    │   └── style.css
    ├── js/
    │   └── app.js
    └── images/
        └── logo.svg
```

## Run Locally
1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Start the Flask server:
   ```bash
   python app.py
   ```
3. Open `http://127.0.0.1:5000` in your browser.
