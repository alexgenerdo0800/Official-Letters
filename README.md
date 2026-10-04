
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Our Letterbox</title>
    <style>
        :root {
            --forest-green: #1b4332;
            --mint-green: #40916c;
            --soft-pink: #ffb703;
            --rose-pink: #ffb5a7;
            --light-pink: #fcd5ce;
            --cream: #f8edeb;
        }

        body {
            font-family: 'Georgia', serif;
            background-color: var(--cream);
            color: var(--forest-green);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            position: relative;
        }

        /* Decorative Lily Flowers placement */
        .lily-decoration {
            font-size: 3.5rem;
            position: absolute;
            opacity: 0.8;
            user-select: none;
            z-index: -1;
        }

        .lily-top-left {
            top: 20px;
            left: 20px;
        }

        .lily-top-right {
            top: 20px;
            right: 20px;
        }

        .container {
            max-width: 600px;
            width: 100%;
            background-color: white;
            border: 3px solid var(--forest-green);
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 8px 16px rgba(27, 67, 50, 0.15);
            box-sizing: border-box;
            margin-top: 40px;
        }

        h1 {
            text-align: center;
            color: var(--forest-green);
            font-size: 2.5rem;
            margin-top: 0;
            border-bottom: 2px dashed var(--rose-pink);
            padding-bottom: 10px;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 10px;
        }

        .form-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            font-weight: bold;
            margin-bottom: 8px;
        }

        input[type="text"], input[type="password"], textarea {
            width: 100%;
            padding: 12px;
            border: 2px solid var(--rose-pink);
            border-radius: 8px;
            box-sizing: border-box;
            font-family: inherit;
            color: var(--forest-green);
            background-color: var(--cream);
        }

        input[type="text"]:focus, input[type="password"]:focus, textarea:focus {
            outline: none;
            border-color: var(--forest-green);
        }

        textarea {
            resize: vertical;
            min-height: 150px;
        }

        .btn {
            background-color: var(--forest-green);
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 20px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s ease;
        }

        .btn:hover {
            background-color: var(--mint-green);
            transform: translateY(-1px);
        }

        .letter-box {
            margin-top: 30px;
            border-top: 2px dashed var(--rose-pink);
            padding-top: 20px;
        }

        .letter {
            background-color: var(--cream);
            border-left: 5px solid var(--rose-pink);
            padding: 15px;
            border-radius: 4px;
            margin-bottom: 15px;
            animation: fadeIn 0.5s ease-in-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .letter-author {
            font-weight: bold;
            color: var(--forest-green);
            margin-bottom: 5px;
        }

        .letter-date {
            font-size: 0.8rem;
            color: #666;
            margin-bottom: 10px;
        }

        .letter-body {
            white-space: pre-wrap;
            line-height: 1.5;
        }

        .error {
            color: #d90429;
            font-weight: bold;
            margin-top: 5px;
            display: none;
        }
    </style>
</head>
<body>

    <!-- Decorative Lily Flower Background Elements -->
    <div class="lily-decoration lily-top-left">🪷</div>
    <div class="lily-decoration lily-top-right">🪷</div>

    <div class="container">
        <h1>📬 Our Secret Mailbox <span style="font-size: 2rem;">🪷</span></h1>

        <!-- Letter Composition Form -->
        <form id="letter-form" onsubmit="handleFormSubmit(event)">
            <div class="form-group">
                <label for="author">Your Name</label>
                <input type="text" id="author" placeholder="E.g., Me / Her" required>
            </div>

            <div class="form-group">
                <label for="message">Your Letter</label>
                <textarea id="message" placeholder="Write something sweet..." required></textarea>
            </div>

            <div class="form-group">
                <label for="pin">Secret PIN</label>
                <input type="password" id="pin" placeholder="Enter the 8-digit PIN" required>
                <div id="pin-error" class="error">Incorrect Secret PIN. Access Denied.</div>
            </div>

            <button type="submit" class="btn" style="width: 100%;">Send Letter</button>
        </form>

        <!-- Archived/Sent Letters Container -->
        <div class="letter-box">
            <h2>Recent Letters</h2>
            <div id="letters-container">
                <!-- Letters will update dynamically here -->
            </div>
        </div>
    </div>

    <script>
        const REQUIRED_PIN = "03250904";
        let db;

        // Initialize Database and UI elements on load
        document.addEventListener('DOMContentLoaded', () => {
            initDatabase();
            requestAutomaticNotificationPermission();
        });

        // Creates structured local browser database equivalent to SQL table architecture
        function initDatabase() {
            const request = indexedDB.open("SecretMailboxDB", 1);

            request.onupgradeneeded = (event) => {
                const database = event.target.result;
                if (!database.objectStoreNames.contains("letters")) {
                    // Equivalent to: CREATE TABLE letters (id SERIAL PRIMARY KEY, author TEXT, message TEXT, date TEXT)
                    database.createObjectStore("letters", { keyPath: "id", autoIncrement: true });
                }
            };

            request.onsuccess = (event) => {
                db = event.target.result;
                loadLetters(); // Load database logs instantly once active
            };

            request.onerror = (event) => {
                console.error("Database initialization failed:", event.target.error);
            };
        }

        function requestAutomaticNotificationPermission() {
            if ("Notification" in window && Notification.permission === "default") {
                Notification.requestPermission();
            }
        }

        // Equivalent to: SELECT * FROM letters ORDER BY id DESC
        function loadLetters() {
            if (!db) return;

            const transaction = db.transaction(["letters"], "readonly");
            const store = transaction.objectStore("letters");
            const request = store.getAll();

            request.onsuccess = () => {
                const container = document.getElementById('letters-container');
                container.innerHTML = '';

                // Reverse sorting arrays to place newest elements at the top
                const letters = request.result.reverse();

                letters.forEach(letter => {
                    const letterEl = document.createElement('div');
                    letterEl.className = 'letter';
                    letterEl.innerHTML = `
                        <div class="letter-author">${escapeHTML(letter.author)}</div>
                        <div class="letter-date">${escapeHTML(letter.date)}</div>
                        <div class="letter-body">${escapeHTML(letter.message)}</div>
                    `;
                    container.appendChild(letterEl);
                });
            };
        }

        // Equivalent to: INSERT INTO letters (author, message, date) VALUES (...)
        function handleFormSubmit(event) {
            event.preventDefault();

            const authorInput = document.getElementById('author');
            const messageInput = document.getElementById('message');
            const pinInput = document.getElementById('pin');
            const pinError = document.getElementById('pin-error');

            // Verify PIN security requirement
            if (pinInput.value !== REQUIRED_PIN) {
                pinError.style.display = "block";
                return;
            }

            pinError.style.display = "none";

            const newLetter = {
                author: authorInput.value,
                message: messageInput.value,
                date: new Date().toLocaleString()
            };

            const transaction = db.transaction(["letters"], "readwrite");
            const store = transaction.objectStore("letters");
            const request = store.add(newLetter);

            request.onsuccess = () => {
                // Trigger Browser Notification automatically if system permission is allowed
                if ("Notification" in window && Notification.permission === "granted") {
                    new Notification(`New Letter from ${newLetter.author}!`, {
                        body: newLetter.message.substring(0, 60) + (newLetter.message.length > 60 ? '...' : ''),
                    });
                }

                // Force layout update immediately to show the submitted letter
                loadLetters();
