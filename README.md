<!DOCTYPE html>
<html lang="el">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Κατάσκοπος: Ζώα, Χώρες & Φαγητά</title>
<style>
:root {
    --bg: #f4f4f4; --fg: #111; --card-bg: #fff; --card-fg: #111;
    --accent: #0077ff; --danger: #d40000;
    --btn-bg: #fff; --btn-fg: #111; --btn-border: #ccc;
}
.dark {
    --bg: #1b1b1b; --fg: #eee; --card-bg: #2b2b2b; --card-fg: #eee;
    --accent: #4da3ff; --danger: #ff4d4d;
    --btn-bg: #333; --btn-fg: #eee; --btn-border: #555;
}
body {
    font-family: Arial, sans-serif; text-align: center; padding: 20px;
    background: var(--bg); color: var(--fg);
}
h1 { font-size: 2rem; }
button {
    font-size: 1rem; padding: 10px 18px; margin: 8px;
    border-radius: 8px; border: 2px solid var(--btn-border);
    background: var(--btn-bg); color: var(--btn-fg);
    cursor: pointer; transition: 0.2s;
}
button:hover { background: var(--accent); color: #fff; }
button.danger:hover { background: var(--danger); }
.card {
    display: none; font-size: 1.6rem; margin: 20px auto;
    padding: 20px; border-radius: 10px;
    background: var(--card-bg); color: var(--card-fg);
    max-width: 400px; box-shadow: 0 0 10px rgba(0,0,0,0.25);
}
#timer { font-size: 1.3rem; font-weight: bold; margin-top: 10px; }
</style>
</head>
<body>
<h1>🎭 Κατάσκοπος: Ζώα, Χώρες & Φαγητά</h1>

<div>
    <label>Κατηγορία:</label>
    <select id="category">
        <option value="animals">Ζώα</option>
        <option value="countries">Χώρες</option>
        <option value="foods">Φαγητά</option>
    </select><br><br>
    <label>Παίκτες:</label>
    <input type="number" id="players" value="5" min="3" max="20"><br>
    <label>Κατάσκοποι:</label>
    <select id="spies"><option>1</option><option>2</option><option>3</option></select><br><br>
    <button onclick="startGame()">Έναρξη Παιχνιδιού</button>
    <button onclick="restartGame(true)" class="danger">Νέο Παιχνίδι</button>
    <button onclick="toggleDarkMode()">Dark Mode</button>
</div>

<div id="gameArea" style="display:none;">
    <h2 id="playerInfo"></h2>
    <button id="revealBtn" onclick="revealRole()">Δείξε Ρόλο</button>
    <div class="card" id="card"></div>
    <button id="hideBtn" style="display:none;" onclick="hideRole()">Κρύψε & Δώσε</button>
    <div id="afterRoles" style="display:none; margin-top:20px;">
        <button onclick="startTimer()">Ξεκινήστε Χρονόμετρο</button>
        <button onclick="showSecret()">Αποκάλυψη Λέξης</button>
        <p id="timer"></p>
        <p id="revealSecret"></p>
    </div>
</div>

<script>
// ----- Λίστες -----
const animals = [
"Λιοντάρι","Τίγρη","Ελέφαντας","Πάντα","Καμηλοπάρδαλη","Καγκουρό","Λύκος","Αλεπού","Αρκούδα","Ιπποπόταμος",
"Ρινόκερος","Πίθηκος","Γορίλας","Χιμπατζής","Λεοπάρδαλη","Τσιτάχ","Σκίουρος","Ελάφι","Καμηλοπάρδαλη","Λαγός",
"Αγριόγατα","Αγριόχοιρος","Σκαντζόχοιρος","Νυχτερίδα","Πιγκουίνος","Κουκουβάγια","Παπαγάλος","Αετός","Φλαμίνγκο",
"Πελεκάνος","Γλάρος","Δελφίνι","Καρχαρίας","Χελώνα","Χταπόδι","Φάλαινα","Καβουράς","Αστακός","Μέδουσα","Βάτραχος",
"Κροκόδειλος","Χαμαιλέοντας","Σαύρα","Οχιά","Μέλισσα","Πεταλούδα","Αράχνη","Μυρμήγκι","Σφήκα","Σκαθάρι"
];

const countries = [
// Ευρώπη
"Ελλάδα","Γαλλία","Ιταλία","Ισπανία","Πορτογαλία","Γερμανία","Ολλανδία","Βέλγιο","Σουηδία","Νορβηγία","Δανία",
"Φινλανδία","Πολωνία","Ουγγαρία","Αυστρία","Ελβετία","Τσεχία","Σλοβακία","Σερβία","Κροατία","Ρουμανία","Βουλγαρία",
"Λιθουανία","Λετονία","Εσθονία","Αλβανία","Μάλτα","Κύπρος","Ηνωμένο Βασίλειο","Ιρλανδία",
// Άλλες ήπειροι (10 τυχαίες)
"Ηνωμένες Πολιτείες","Καναδάς","Βραζιλία","Αργεντινή","Αυστραλία","Ινδία","Κίνα","Ιαπωνία","Νότια Αφρική","Αίγυπτος"
];

const foods = [
"Πίτσα","Μακαρόνια","Σουβλάκι","Χωριάτικη Σαλάτα","Μουσακάς","Γύρος","Μπιφτέκι","Παστίτσιο","Κεμπάπ","Τυρόπιτα",
"Σαλάτα Καίσαρα","Ριζότο","Χάμπουργκερ","Σπανακόπιτα","Κρέπα","Τάρτα","Λουκουμάδες","Γαριδομακαρονάδα","Φαλάφελ","Σούσι"
];

// ----- Μεταβλητές -----
let roles = [], current = 0, selectedSecret = "", category = "";
let timerInterval, timeLeft = 360, darkMode = false;

// ----- Έναρξη -----
function startGame(){
    const numPlayers = parseInt(document.getElementById('players').value);
    const numSpies = parseInt(document.getElementById('spies').value);
    category = document.getElementById('category').value;

    if(numPlayers<3){alert("Τουλάχιστον 3 παίκτες!");return;}
    if(numSpies>=numPlayers){alert("Οι κατάσκοποι πρέπει να είναι λιγότεροι από τους παίκτες.");return;}

    const list = category==="animals"?animals:category==="countries"?countries:foods;
    selectedSecret = list[Math.floor(Math.random()*list.length)];
    roles = Array(numPlayers).fill(selectedSecret);
    pickUniqueRandomIndices(numSpies,numPlayers).forEach(i=>roles[i]="Κατάσκοπος");

    current=0; clearInterval(timerInterval);
    document.getElementById('timer').textContent="";
    document.getElementById('revealSecret').textContent="";
    document.getElementById('afterRoles').style.display="none";
    document.getElementById('gameArea').style.display="block";
    document.getElementById('playerInfo').textContent="Παίκτης 1: Δείξε Ρόλο";
    document.getElementById('card').style.display="none";
    document.getElementById('revealBtn').style.display="inline-block";
    document.getElementById('hideBtn').style.display="none";
}

// Δείξε / Κρύψε
function revealRole(){
    const card=document.getElementById('card');
    card.style.display="block";
    card.textContent = roles[current]==="Κατάσκοπος" ? "Είσαι ΚΑΤΑΣΚΟΠΟΣ!" : label()+roles[current];
    document.getElementById('revealBtn').style.display="none";
    document.getElementById('hideBtn').style.display="inline-block";
}
function hideRole(){
    document.getElementById('card').style.display="none";
    document.getElementById('hideBtn').style.display="none";
    current++;
    if(current<roles.length){
        document.getElementById('playerInfo').textContent=`Παίκτης ${current+1}: Δείξε Ρόλο`;
        document.getElementById('revealBtn').style.display="inline-block";
    } else {
        document.getElementById('playerInfo').textContent="Όλοι έτοιμοι!";
        document.getElementById('afterRoles').style.display="block";
    }
}
function label(){return category==="animals"?"Ζώο: ":category==="countries"?"Χώρα: ":"Φαγητό: ";}

// Χρονόμετρο
function startTimer(){
    timeLeft=360; updateTimer();
    clearInterval(timerInterval);
    timerInterval=setInterval(()=>{
        timeLeft--; updateTimer();
        if(timeLeft<=0){clearInterval(timerInterval);}
    },1000);
}
function updateTimer(){
    const min=Math.floor(timeLeft/60),sec=timeLeft%60;
    document.getElementById('timer').textContent=`${min}:${sec<10?"0":""}${sec}`;
}

// Αποκάλυψη
function showSecret(){
    document.getElementById('revealSecret').textContent=label()+selectedSecret;
}

// Νέο παιχνίδι
function restartGame(btn){
    if(btn && !confirm("Νέο παιχνίδι;"))return;
    clearInterval(timerInterval);
    document.getElementById('gameArea').style.display="none";
}

// Dark Mode
function toggleDarkMode(){
    darkMode=!darkMode; document.body.classList.toggle('dark',darkMode);
}

// Helper
function pickUniqueRandomIndices(count,max){
    const arr=[]; while(arr.length<count){
        let i=Math.floor(Math.random()*max);
        if(!arr.includes(i))arr.push(i);
    } return arr;
}
</script>
</body>
</html>
