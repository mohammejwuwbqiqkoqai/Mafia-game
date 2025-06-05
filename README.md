# Mafia-game
لعبة
<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>لعبة المافيا - Mafia Game</title>
<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #f5f7fa;
    color: #333;
    margin: 0; padding: 20px;
    direction: rtl;
  }
  .container {
    max-width: 900px;
    margin: auto;
  }
  h1 {
    text-align: center;
    color: #5c6bc0;
    margin-bottom: 15px;
  }
  .input-section {
    margin-bottom: 20px;
    text-align: center;
  }
  input[type="text"] {
    padding: 10px;
    font-size: 1rem;
    width: 60%;
    max-width: 300px;
    border: 1px solid #ccc;
    border-radius: 6px;
    transition: border-color 0.3s ease;
  }
  input[type="text"]:focus {
    border-color: #5c6bc0;
    outline: none;
  }
  button {
    padding: 10px 20px;
    background: #7986cb;
    border: none;
    border-radius: 6px;
    color: white;
    font-weight: bold;
    cursor: pointer;
    margin-left: 10px;
    transition: background 0.3s ease;
  }
  button:hover:not(:disabled) {
    background: #5c6bc0;
  }
  button:disabled {
    background: #ccc;
    cursor: not-allowed;
  }
  .roles-container {
    display: flex;
    gap: 15px;
    flex-wrap: wrap;
    justify-content: center;
    min-height: 180px;
  }
  .card {
    background: white;
    border-radius: 10px;
    box-shadow: 0 2px 6px rgb(0 0 0 / 0.15);
    width: 140px;
    padding: 15px;
    text-align: center;
    position: relative;
    transition: transform 0.3s ease, background-color 0.3s ease;
    cursor: default;
  }
  .card:hover {
    transform: translateY(-5px);
  }
  .card h3 {
    margin-top: 0;
    color: #3f51b5;
    word-break: break-word;
  }
  .message {
    text-align: center;
    font-weight: bold;
    color: #d32f2f;
    margin-top: 20px;
    min-height: 28px;
  }
  .language-toggle {
    text-align: center;
    margin-bottom: 15px;
  }
  .language-toggle button {
    background: transparent;
    color: #5c6bc0;
    border: 2px solid #5c6bc0;
    border-radius: 20px;
    padding: 5px 15px;
    font-weight: 600;
    margin: 0 5px;
    cursor: pointer;
    transition: background 0.3s ease, color 0.3s ease;
  }
  .language-toggle button.active {
    background: #5c6bc0;
    color: white;
  }
  .night-action {
    text-align: center;
    margin-top: 20px;
  }
  .night-action button {
    background: #4caf50;
    margin-left: 0;
  }
  .protected {
    background-color: #c8e6c9 !important;
  }
  .dead {
    background-color: #ef9a9a !important;
    text-decoration: line-through;
    color: #b71c1c;
  }
</style>
</head>
<body>
<div class="container">
  <h1 id="title">لعبة المافيا</h1>
  
  <div class="language-toggle">
    <button id="btn-ar" class="active" aria-label="اختيار اللغة العربية">عربي</button>
    <button id="btn-en" aria-label="Select English language">English</button>
  </div>
  
  <div class="input-section" id="inputSection">
    <input type="text" id="playerName" placeholder="ادخل اسمك هنا" aria-label="ادخل اسمك هنا" />
    <button id="startBtn">ابدأ اللعبة</button>
  </div>
  
  <div id="rolesArea" class="roles-container" aria-live="polite" aria-label="بطاقات اللاعبين"></div>
  
  <div id="message" class="message" role="alert" aria-live="assertive"></div>
  
  <div class="night-action" id="nightActions" style="display:none;">
    <button id="doctorProtectBtn" disabled>حماية دكتور</button>
    <button id="mafiaKillBtn" disabled>قتل مافيا</button>
    <button id="nextNightBtn" disabled>التالي - ليلة جديدة</button>
  </div>
  
  <audio id="nightSound" src="https://actions.google.com/sounds/v1/ambiences/night_ambience.ogg" preload="auto"></audio>
</div>

<script>
  const rolesArea = document.getElementById('rolesArea');
  const startBtn = document.getElementById('startBtn');
  const playerNameInput = document.getElementById('playerName');
  const message = document.getElementById('message');
  const nightSound = document.getElementById('nightSound');
  
  const doctorProtectBtn = document.getElementById('doctorProtectBtn');
  const mafiaKillBtn = document.getElementById('mafiaKillBtn');
  const nextNightBtn = document.getElementById('nextNightBtn');
  const nightActions = document.getElementById('nightActions');
  const inputSection = document.getElementById('inputSection');
  
  const langButtons = {
    ar: document.getElementById('btn-ar'),
    en: document.getElementById('btn-en'),
  };
  let currentLang = 'ar';

  const texts = {
    ar: {
      title: 'لعبة المافيا',
      placeholder: 'ادخل اسمك هنا',
      start: 'ابدأ اللعبة',
      roles: {
        doctor: 'دكتور',
        mafia: 'مافيا',
        civilian: 'مواطن',
        policeman: 'شرطي',
      },
      messageKillCivilian: 'يا أغبياء شوكت تعرفون القاتل!',
      nightSoundTitle: 'الليل',
      doctorProtectBtn: 'حماية دكتور',
      mafiaKillBtn: 'قتل مافيا',
      nextNightBtn: 'التالي - ليلة جديدة',
      alertNoPlayers: 'رجاءً ادخل اسمك',
      alertNoTarget: 'اختر لاعبًا صحيحًا',
      doctorProtectSelfForbidden: 'الدكتور لا يستطيع حماية نفسه مرتين متتاليتين',
    },
    en: {
      title: 'Mafia Game',
      placeholder: 'Enter your name',
      start: 'Start Game',
      roles: {
        doctor: 'Doctor',
        mafia: 'Mafia',
        civilian: 'Civilian',
        policeman: 'Policeman',
      },
      messageKillCivilian: 'You fools, when will you find the killer!',
      nightSoundTitle: 'Night',
      doctorProtectBtn: 'Doctor Protect',
      mafiaKillBtn: 'Mafia Kill',
      nextNightBtn: 'Next Night',
      alertNoPlayers: 'Please enter your name',
      alertNoTarget: 'Please select a valid player',
      doctorProtectSelfForbidden: 'Doctor cannot protect himself twice in a row',
    },
  };

  // Players array: {name, role, alive, protectedLastNight}
  let players = [];
  let doctorLastProtected = null;
  let currentNight = 1;
  let doctorProtectedPlayer = null;
  let mafiaTarget = null;

  // Language functions
  function translatePage() {
    const t = texts[currentLang];
    document.getElementById('title').textContent = t.title;
    playerNameInput.placeholder = t.placeholder;
    startBtn.textContent = t.start;
    doctorProtectBtn.textContent = t.doctorProtectBtn;
    mafiaKillBtn.textContent = t.mafiaKillBtn;
    nextNightBtn.textContent = t.nextNightBtn;
  }

  langButtons.ar.onclick = () => {
    currentLang = 'ar';
    translatePage();
    langButtons.ar.classList.add('active');
    langButtons.en.classList.remove('active');
    document.body.style.direction = 'rtl';
  };

  langButtons.en.onclick = () => {
    currentLang = 'en';
    translatePage();
    langButtons.en.classList.add('active');
    langButtons.ar.classList.remove('active');
    document.body.style.direction = 'ltr';
  };

  translatePage();

  // Utility shuffle
  function shuffle(array) {
    for(let i = array.length -1; i > 0; i--){
      const j = Math.floor(Math.random() * (i+1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  // Assign roles to players
  function assignRoles(playerNames) {
    const roles = [];
    // Minimum players is 4 to assign all roles properly
    if(playerNames.length < 4){
      alert(currentLang === 'ar' ? 'يجب أن يكون هناك 4 لاعبين على الأقل' : 'At least 4 players are required');
      return [];
    }
    // Always 1 mafia, 1 doctor, 1 policeman, rest civilians
    roles.push('mafia', 'doctor', 'policeman');
    for(let i = 3; i < playerNames.length; i++){
      roles.push('civilian');
    }
    shuffle(roles);

    return playerNames.map((name,i) => ({
      name,
      role: roles[i],
      alive: true,
      protectedLastNight: false,
    }));
  }

  // Create card element for a player
  function createCard(player, index) {
    const t = texts[currentLang];
    const card = document.createElement('div');
    card.classList.add('card');
    if(!player.alive) card.classList.add('dead');
    else if(player.protectedLastNight) card.classList.add('protected');
    card.dataset.index = index;
    card.tabIndex = 0;
    card.innerHTML = `
      <h3>${player.name}</h3>
      <p>${t.roles[player.role]}</p>
    `;
    return card;
  }

  // Show all players cards
  function showPlayers() {
    rolesArea.innerHTML = '';
    players.forEach((player, i) => {
      const card = createCard(player, i);
      rolesArea.appendChild(card);
    });
  }

  // Handle game start
  startBtn.onclick = () => {
    const name = playerNameInput.value.trim();
    if(!name){
      alert(texts[currentLang].alertNoPlayers);
      return;
    }
    // For demo: generate other players with placeholder names
    const playerNames = [name];
    for(let i=1; i < 6; i++){
      playerNames.push(currentLang==='ar' ? `لاعب ${i+1}` : `Player ${i+1}`);
    }
    players = assignRoles(playerNames);
    if(players.length === 0) return; // error
    
    doctorLastProtected = null;
    doctorProtectedPlayer = null;
    mafiaTarget = null;
    currentNight = 1;
    inputSection.style.display = 'none';
    nightActions.style.display = 'block';
    message.textContent = '';
    showPlayers();
    playNightSound();
    updateButtons();
  };

  // Play night sound
  function playNightSound(){
    nightSound.currentTime = 0;
    nightSound.play();
  }

  // Update buttons enable/disable
  function updateButtons(){
    // Doctor protect enabled only if doctor alive
    const doctor = players.find(p => p.role==='doctor' && p.alive);
    doctorProtectBtn.disabled = !doctor;

    // Mafia kill enabled only if mafia alive
    const mafia = players.find(p => p.role==='mafia' && p.alive);
    mafiaKillBtn.disabled = !mafia;

    // Next night enabled only after actions done
    nextNightBtn.disabled = !(doctorProtectedPlayer !== null && mafiaTarget !== null);
  }

  // Card selection logic
  let selectedForProtect = null;
  let selectedForKill = null;

  // Click on card logic
  rolesArea.addEventListener('click', e => {
    if(!e.target.closest('.card')) return;
    const card = e.target.closest('.card');
    const index = +card.dataset.index;
    if(!players[index].alive) return;

    // Depending on which action button is enabled, select target
    if(!doctorProtectBtn.disabled && !doctorProtectBtn.classList.contains('disabled')) {
      // Select for protect if doctor protect button is enabled
      if(selectedForProtect !== null) {
        // Remove previous highlight
        rolesArea.children[selectedForProtect].classList.remove('selected-protect');
      }
      selectedForProtect = index;
      card.classList.add('selected-protect');
    } else if(!mafiaKillBtn.disabled && !mafiaKillBtn.classList.contains('disabled')) {
      // Select for kill if mafia kill button is enabled
      if(selectedForKill !== null) {
        rolesArea.children[selectedForKill].classList.remove('selected-kill');
      }
      selectedForKill = index;
      card.classList.add('selected-kill');
    }
  });

  // Styles for selection
  const style = document.createElement('style');
  style.innerHTML = `
    .selected-protect {
      box-shadow: 0 0 10px 3px #4caf50;
      border: 2px solid #4caf50;
    }
    .selected-kill {
      box-shadow: 0 0 10px 3px #d32f2f;
      border: 2px solid #d32f2f;
    }
  `;
  document.head.appendChild(style);

  // Doctor protect button click
  doctorProtectBtn.onclick = () => {
    if(selectedForProtect === null) {
      alert(texts[currentLang].alertNoTarget);
      return;
    }
    const doctor = players.find(p => p.role==='doctor' && p.alive);
    if(!doctor) return;

    // Doctor cannot protect self twice in a row
    if(doctorLastProtected === selectedForProtect){
      alert(texts[currentLang].doctorProtectSelfForbidden);
      return;
    }

    // Protect the selected player
    doctorProtectedPlayer = selectedForProtect;
    doctorLastProtected = selectedForProtect;

    message.textContent = `${texts[currentLang].roles.doctor} قام بحماية ${players[selectedForProtect].name}`;
    // Disable doctor protect btn and selection
    doctorProtectBtn.disabled = true;
    rolesArea.children[selectedForProtect].classList.remove('selected-protect');
    selectedForProtect = null;

    updateButtons();
  };

  // Mafia kill button click
  mafiaKillBtn.onclick = () => {
    if(selectedForKill === null) {
      alert(texts[currentLang].alertNoTarget);
      return;
    }
    const mafia = players.find(p => p.role==='mafia' && p.alive);
    if(!mafia) return;

    mafiaTarget = selectedForKill;
    message.textContent = `${texts[currentLang].roles.mafia} يستهدف ${players[selectedForKill].name}`;
    mafiaKillBtn.disabled = true;
    rolesArea.children[selectedForKill].classList.remove('selected-kill');
    selectedForKill = null;

    updateButtons();
  };

  // Next night button click - process kills and protections
  nextNightBtn.onclick = () => {
    // Stop night sound
    nightSound.pause();

    if(mafiaTarget === null) {
      alert(texts[currentLang].alertNoTarget);
      return;
    }

    // Process kill
    if(mafiaTarget !== doctorProtectedPlayer) {
      // Player dies
      players[mafiaTarget].alive = false;

      // Show message if civilian dies
      if(players[mafiaTarget].role === 'civilian'){
        message.textContent = texts[currentLang].messageKillCivilian;
      } else {
        message.textContent = `${players[mafiaTarget].name} تم قتله`;
      }
    } else {
      // Protected - no death
      message.textContent = `${players[mafiaTarget].name} تم حمايته بنجاح`;
    }

    // Reset protections for next night
    players.forEach(p => p.protectedLastNight = false);
    if(doctorProtectedPlayer !== null) {
      players[doctorProtectedPlayer].protectedLastNight = true;
    }

    // Reset targets
    doctorProtectedPlayer = null;
    mafiaTarget = null;

    // Show updated players
    showPlayers();

    currentNight++;
    nextNightBtn.disabled = true;
    doctorProtectBtn.disabled = false;
    mafiaKillBtn.disabled = false;

    playNightSound();
  };
</script>
</body>
</html>
