<!DOCTYPE html>
<html lang="el">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Κατάσκοπος με Ζώα</title>
<meta name="description" content="Spyfall-τύπου παιχνίδι στα ελληνικά με ζώα. Παίξτε με φίλους!">
<style>
:root {
    --bg: #f4f4f4;
    --fg: #111;
    --card-bg: #fff;
    --card-fg: #111;
    --accent: #0077ff;
    --danger: #d40000;
    --btn-bg: #fff;
    --btn-fg: #111;
    --btn-border: #ccc;
}
.dark {
    --bg: #1b1b1b;
    --fg: #eee;
    --card-bg: #2b2b2b;
    --card-fg: #eee;
    --accent: #4da3ff;
    --danger: #ff4d4d;
    --btn-bg: #333;
    --btn-fg: #eee;
    --btn-border: #555;
}
html,body {
    margin:0;
    padding:0;
}
body {
    font-family: Arial, sans-serif;
    text-align: center;
    padding: 20px;
    background: var(--bg);
    color: var(--fg);
    transition: background 0.2s ease, color 0.2s ease;
}
h1 {
    margin-top:0;
    font-size: clamp(1.8rem, 4vw, 2.5rem);
}
p, label {
    font-size: clamp(1rem, 3vw, 1.2rem);
}
input, select {
    font-size: 1.1rem;
    padding: 6px 8px;
    margin: 5px;
}
button {
    font-size: 1.1rem;
    padding: 12px 24px;
    margin: 8px;
    cursor: pointer;
    border-radius: 8px;
    border: 2px solid var(--btn-border);
    background: var(--btn-bg);
    color: var(--btn-fg);
    transition: transform 0.1s, background 0.2s;
    max-width: 90%;
}
button:hover {
    transform: scale(1.03);
    background: var(--accent);
    color: #fff;
}
button.danger:hover {
    background: var(--danger);
}
.controls {
    margin-bottom:20px;
}
#gameArea {
    max-width: 600px;
    margin: 0 auto;
}
.card {
    display: none;
    font-size: 1.8rem;
    margin-top: 30px;
    padding: 20px;
    border-radius: 10px;
    background: var(--card-bg);
    color: var(--card-fg);
    box-shadow: 0 0 10px rgba(0,0,0,0.25);
    word-wrap: break-word;
}
#timer {
    font-size: 1.5rem;
    margin-top: 15px;
    font-weight: bold;
}
.flex-row {
    display:flex;
    justify-content:center;
    align-items:center;
    flex-wrap:wrap;
    gap:8px;
}
@media (max-width:480px){
    button {
        font-size:1rem;
        padding:10px 16px;
    }
    .card {
        font-size:1.5rem;
        padding:16px;
    }
}
</style>
</head>
<body>
<h1>🐾 Κατάσκοπος με Ζώα</h1>

<div class="controls">
    <label for="players">Παίκτες:</label>
    <input type="number" id="players" min="3" max="20" value="5">
    <br>
    <label for="spies">Κατάσκοποι:</label>
    <select id="spies">
        <option value="1" selected>1</option>
        <option value="2">2</option>
        <option value="3">3</option>
    </select>
    <br>
    <button onclick="startGame()">Έναρξη Παιχνιδιού</button>
    <button onclick="restartGame(true)" class="danger">Νέο Παιχνίδι</button>
    <button onclick="toggleDarkMode()">Dark Mode</button>
</div>

<div id="gameArea" style="display:none;">
    <h2 id="playerInfo"></h2>
    <button id="revealBtn" onclick="revealRole()">Δείξε τον Ρόλο</button>
    <div class="card" id="card"></div>
    <button id="hideBtn" style="display:none;" onclick="hideRole()">Κρύψε & Δώσε Στον Επόμενο</button>
    <br>
    <div id="afterRoles" style="display:none;">
        <button onclick="startTimer()">Ξεκινήστε το Χρονόμετρο</button>
        <button onclick="showAnimal()">Αποκάλυψη Ζώου</button>
        <p id="timer"></p>
        <p id="revealAnimal"></p>
    </div>
</div>

<script>
// ---- ΔΕΔΟΜΕΝΑ ΖΩΩΝ ----
const animals = [
    // Θηλαστικά
    "Λιοντάρι","Ελέφαντας","Τίγρη","Πάντα","Καμηλοπάρδαλη","Καγκουρό","Λύκος","Αλεπού","Αρκούδα",
    "Λεοπάρδαλη","Τσιτάχ","Ιπποπόταμος","Ρινόκερος","Γορίλας","Χιμπατζής","Λαγός","Σκίουρος","Ελάφι",
    "Αγριόχοιρος","Σκαντζόχοιρος","Νυχτερίδα",
    // Πουλιά
    "Πιγκουίνος","Κουκουβάγια","Φλαμίνγκο","Παπαγάλος","Αετός","Γεράκι","Σπουργίτι","Πελαργός","Γλάρος","Παγώνι",
    // Θαλάσσια
    "Δελφίνι","Καρχαρίας","Χταπόδι","Χελώνα","Ιππόκαμπος","Φάλαινα","Μέδουσα","Σφουγγάρι","Καβουράς","Αστακός",
    // Ερπετά & Αμφίβια
    "Κροκόδειλος","Βάτραχος","Σαύρα","Χαμαιλέοντας","Οχιά",
    // Έντομα
    "Μέλισσα","Πεταλούδα","Αράχνη","Μυρμήγκι","Σκαθάρι","Τζιτζίκι","Ακρίδα","Σφήκα"
];

// ---- ΚΑΤΑΣΤΑΣΗ ΠΑΙΧΝΙΔΙΟΥ ----
let roles = [];
let current = 0;
let selectedAnimal = "";
let timerInterval;
let timeLeft = 360; // 6 λεπτά
let darkMode = false;

// ---- ΕΝΑΡΞΗ ----
function startGame() {
    const numPlayers = parseInt(document.getElementById("players").value, 10);
    const numSpies = parseInt(document.getElementById("spies").value, 10);

    if (numPlayers < 3) {
        alert("Χρειάζονται τουλάχιστον 3 παίκτες!");
        return;
    }
    if (numSpies >= numPlayers) {
        alert("Οι κατάσκοποι πρέπει να είναι λιγότεροι από τους παίκτες!");
        return;
    }

    // Επιλογή ζώου
    selectedAnimal = animals[Math.floor(Math.random() * animals.length)];

    // Όλοι αρχικά παίρνουν το ζώο
    roles = Array(numPlayers).fill(selectedAnimal);

    // Επιλογή τυχαίων κατασκόπων
    const spyIndices = pickUniqueRandomIndices(numSpies, numPlayers);
    spyIndices.forEach(i => roles[i] = "Κατάσκοπος");

    // Μηδενισμοί
    current = 0;
    clearInterval(timerInterval);
    document.getElementById("timer").textContent = "";
    document.getElementById("revealAnimal").textContent = "";
    document.getElementById("afterRoles").style.display = "none";

    // Εμφάνιση περιοχής παιχνιδιού
    document.getElementById("gameArea").style.display = "block";
    document.getElementById("playerInfo").textContent = "Παίκτης 1: Πάτησε 'Δείξε τον Ρόλο'";
    document.getElementById("card").style.display = "none";
    document.getElementById("revealBtn").style.display = "inline-block";
    document.getElementById("hideBtn").style.display = "none";
}

// ---- ΛΕΙΤΟΥΡΓΙΕΣ ΚΑΡΤΑΣ ΡΟΛΟΥ ----
function revealRole() {
    const card = document.getElementById("card");
    card.style.display = "block";
    card.textContent = (roles[current] === "Κατάσκοπος")
        ? "Είσαι ΚΑΤΑΣΚΟΠΟΣ!"
        : "Ζώο: " + roles[current];
    document.getElementById("revealBtn").style.display = "none";
    document.getElementById("hideBtn").style.display = "inline-block";
}

function hideRole() {
    document.getElementById("card").style.display = "none";
    document.getElementById("hideBtn").style.display = "none";
    current++;
    if (current < roles.length) {
        document.getElementById("playerInfo").textContent = `Παίκτης ${current+1}: Πάτησε 'Δείξε τον Ρόλο'`;
        document.getElementById("revealBtn").style.display = "inline-block";
    } else {
        document.getElementById("playerInfo").textContent = "Όλοι οι παίκτες είδαν τον ρόλο τους! Ξεκινήστε!";
        document.getElementById("revealBtn").style.display = "none";
        document.getElementById("afterRoles").style.display = "block";
    }
}

// ---- ΧΡΟΝΟΜΕΤΡΟ ----
function startTimer() {
    timeLeft = 360; // 6 λεπτά
    document.getElementById("timer").textContent = formatTime(timeLeft);
    clearInterval(timerInterval);
    timerInterval = setInterval(() => {
        timeLeft--;
        document.getElementById("timer").textContent = formatTime(timeLeft);
        if (timeLeft <= 0) {
            clearInterval(timerInterval);
            document.getElementById("timer").textContent = "Τέλος χρόνου!";
        }
    }, 1000);
}

function formatTime(seconds) {
    const min = Math.floor(seconds / 60);
    const sec = seconds % 60;
    return `${min}:${sec < 10 ? '0' : ''}${sec}`;
}

// ---- ΑΠΟΚΑΛΥΨΗ ΖΩΟΥ ----
function showAnimal() {
    document.getElementById("revealAnimal").textContent = "Το ζώο ήταν: " + selectedAnimal;
}

// ---- ΝΕΟ ΠΑΙΧΝΙΔΙ (Restart) ----
function restartGame(fromButton=false) {
    // Αν πατήθηκε το κουμπί "Νέο Παιχνίδι" ενώ παίζεται γύρος, ρώτα επιβεβαίωση
    if (fromButton) {
        const ok = confirm("Νέο παιχνίδι; Θα χαθεί ο τρέχων γύρος.");
        if (!ok) return;
    }
    // Καθάρισμα κατάστασης
    clearInterval(timerInterval);
    roles = [];
    selectedAnimal = "";
    current = 0;
    document.getElementById("timer").textContent = "";
    document.getElementById("revealAnimal").textContent = "";
    document.getElementById("afterRoles").style.display = "none";
    document.getElementById("card").style.display = "none";
    document.getElementById("hideBtn").style.display = "none";
    document.getElementById("revealBtn").style.display = "none";
    document.getElementById("playerInfo").textContent = "Ξεκίνα νέο παιχνίδι παραπάνω.";
    document.getElementById("gameArea").style.display = "none";
}

// ---- DARK MODE ----
function toggleDarkMode() {
    darkMode = !darkMode;
    document.body.classList.toggle('dark', darkMode);
}

// ---- ΒΟΗΘΗΤΙΚΕΣ ----
function pickUniqueRandomIndices(count, max) {
    const indices = [];
    while (indices.length < count) {
        const i = Math.floor(Math.random() * max);
        if (!indices.includes(i)) indices.push(i);
    }
    return indices;
}
</script>
</body>
</html>
