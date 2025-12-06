<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Train2Excavate: Volunteer Academy</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Using Three.js for 3D simulation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        /* --- General Styles --- */
        body { margin: 0; overflow-x: hidden; touch-action: none; font-family: 'Inter', sans-serif; }
        
        /* --- 3D Sim Styles --- */
        #sim-container {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1000;
            display: none;
            background-color: #000;
        }
        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }
        .interactive-element { pointer-events: auto; }
        
        /* Crosshair */
        #crosshair {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 6px;
            height: 6px;
            margin: -3px 0 0 -3px;
            border-radius: 50%;
            background: #fcd34d;
            border: 1px solid #78350f;
            z-index: 1001;
            box-shadow: 0 0 5px #000;
        }

        /* Mobile Joysticks */
        .joystick-zone {
            position: absolute;
            bottom: 40px;
            width: 140px;
            height: 140px;
            background: rgba(0, 0, 0, 0.2);
            border: 2px solid rgba(255, 255, 255, 0.4);
            border-radius: 50%;
            pointer-events: auto;
            touch-action: none;
            display: none; /* JS enables this on touch devices */
        }
        #stick-left { left: 20px; }
        #stick-right { right: 20px; }
        
        .stick-nub {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 60px;
            height: 60px;
            background: rgba(245, 158, 11, 0.8);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
            box-shadow: 0 0 15px rgba(0,0,0,0.7);
        }

        /* Dialogue & Quiz UI */
        #dialogue-box {
            background: rgba(28, 25, 23, 0.95);
            border: 2px solid #d97706;
            color: #fff;
            padding: 24px;
            border-radius: 12px;
            max-width: 90%;
            width: 700px;
            margin: 0 auto 30px auto;
            pointer-events: auto;
            display: none;
            font-family: 'Inter', sans-serif;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
        }

        /* Message Box Notification */
        #notification-box {
            position: fixed;
            top: 100px;
            right: 20px;
            z-index: 2000;
            padding: 15px 25px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.3);
            display: none;
            transition: opacity 0.3s ease-in-out, transform 0.3s ease-in-out;
            opacity: 0;
            transform: translateX(100%);
        }

        /* Tool HUD */
        #tool-hud {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 10px;
            background: rgba(0, 0, 0, 0.6);
            padding: 10px 15px;
            border-radius: 12px;
            border: 1px solid #fcd34d;
            box-shadow: 0 0 10px rgba(252, 211, 77, 0.5);
            pointer-events: none;
        }
        .tool-icon {
            font-size: 2rem;
            opacity: 0.7;
            transition: opacity 0.3s;
        }
        .tool-icon.active {
            opacity: 1.0;
            transform: scale(1.1);
            filter: drop-shadow(0 0 5px #fcd34d);
        }

        /* --- Website Styles --- */
        .progress-bar-container {
            width: 100%;
            background-color: #44403c;
            border-radius: 999px;
            overflow: hidden;
            height: 12px;
        }
        .progress-bar-fill {
            height: 100%;
            background-color: #f59e0b;
            transition: width 0.5s ease-out;
            box-shadow: 0 0 8px #f59e0b;
        }
        .animate-fade-in { animation: fadeIn 0.5s ease-out forwards; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        /* Professional Buttons */
        .nav-button {
            transition: all 0.3s;
            position: relative;
        }
        .nav-button:hover {
            color: #fcd34d;
            transform: translateY(-2px);
        }
        .nav-button.active::after {
            content: '';
            position: absolute;
            bottom: -8px;
            left: 0;
            width: 100%;
            height: 3px;
            background: #fcd34d;
            border-radius: 2px;
        }
    </style>
</head>
<body class="bg-gray-50 text-stone-800">

    <!-- NOTIFICATION BOX -->
    <div id="notification-box" class="text-white">
        <div id="notification-title" class="font-bold text-lg"></div>
        <p id="notification-message" class="text-sm"></p>
    </div>

    <!-- WEBSITE UI -->
    <div id="website-container" class="min-h-screen flex flex-col">
        
        <!-- Navbar with Progress -->
        <nav class="bg-stone-900 text-stone-100 p-4 sticky top-0 z-50 shadow-xl border-b-4 border-amber-600">
            <div class="max-w-7xl mx-auto">
                <div class="flex flex-col md:flex-row justify-between items-center gap-4">
                    <div class="flex items-center space-x-2 font-serif text-3xl font-extrabold text-amber-500 cursor-pointer">
                        <span>⛏️ Train2Excavate Academy</span>
                    </div>
                    
                    <!-- Nav Buttons -->
                    <div class="flex space-x-8 text-lg font-bold">
                        <button id="nav-home" onclick="switchSection('home')" class="nav-button active">HUB</button>
                        <button id="nav-guides" onclick="switchSection('guides')" class="nav-button">GUIDES</button>
                        <button id="nav-quiz" onclick="switchSection('quiz')" class="nav-button">ACADEMY</button>
                    </div>

                    <!-- Action Button -->
                    <button onclick="startSimulation()" class="bg-amber-600 hover:bg-amber-700 text-white px-6 py-2 rounded-xl transition font-bold shadow-lg border border-amber-400 flex items-center transform hover:scale-105">
                        <span class="mr-2">ENTRANCE SIMULATION</span>
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 8l4 4m0 0l-4 4m4-4H3"></path></svg>
                    </button>
                </div>
            </div>
        </nav>

        <!-- Dynamic Content -->
        <main id="content-area" class="flex-grow">
            <!-- Content Injected via JS -->
        </main>

        <footer class="bg-stone-900 text-stone-500 py-6 mt-auto text-center border-t border-stone-800">
            <p>Train2Excavate &copy; 2024 • Professional Volunteer Training Systems</p>
        </footer>
    </div>

    <!-- 3D SIMULATION OVERLAY -->
    <div id="sim-container">
        <div id="crosshair"></div>
        <!-- Canvas injected here -->
        
        <div id="ui-layer">
            <!-- HUD Top Left & Right -->
            <div class="p-6 flex justify-between items-start">
                <!-- Site Info (Left) -->
                <div class="p-3 bg-black/40 rounded-lg shadow-xl backdrop-blur-sm interactive-element">
                    <h2 class="text-3xl font-bold text-amber-400 font-serif drop-shadow-lg">Site 42</h2>
                    <p class="text-stone-300 text-sm drop-shadow-md">Objective: Explore & Document</p>
                </div>
                <!-- Tools HUD (Center) -->
                <div id="tool-hud">
                    <span class="tool-icon active" data-tool="trowel">⛏️</span>
                    <span class="tool-icon" data-tool="brush">🧹</span>
                    <span class="tool-icon" data-tool="sifter">🧺</span>
                    <span class="tool-icon" data-tool="tape">📏</span>
                </div>
                <!-- Exit Button (Right) -->
                <button onclick="exitSimulation()" class="interactive-element bg-red-700 hover:bg-red-800 text-white px-5 py-2 rounded-xl font-bold shadow-xl border border-red-400 transition transform hover:scale-105">
                    EXIT SIM
                </button>
            </div>

            <!-- Player Stats in Sim (Bottom Center) -->
            <div class="p-4 bg-black/40 backdrop-blur-sm text-white rounded-t-lg mx-auto mb-0 interactive-element">
                <div class="text-xs text-stone-400 uppercase font-bold text-center">Current Rank</div>
                <div class="text-amber-400 font-bold text-xl leading-none" id="sim-rank-display">Novice</div>
                <div class="progress-bar-container mt-1 w-48">
                    <div id="sim-xp-bar" class="progress-bar-fill" style="width: 0%"></div>
                </div>
            </div>

            <!-- Interaction Prompt -->
            <div id="interaction-prompt" class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 transition-opacity duration-200 opacity-0 pointer-events-none">
                <div class="bg-black/80 backdrop-blur text-white px-8 py-3 rounded-full border-2 border-amber-500 font-bold text-lg shadow-xl hover:scale-105 transition">
                    [Click / Tap] to Interact (E Key)
                </div>
            </div>

            <!-- Dialogue -->
            <div id="dialogue-container" class="w-full flex justify-center pb-20 md:pb-8">
                <div id="dialogue-box">
                    <h3 id="npc-name" class="text-2xl font-bold text-amber-500 mb-3 border-b-2 border-stone-600 pb-1">Name</h3>
                    <p id="npc-text" class="text-lg leading-relaxed mb-6 text-stone-200">...</p>
                    <div id="quiz-options" class="flex flex-col gap-3">
                        <!-- Quiz options injected here -->
                    </div>
                    <div id="dialogue-controls" class="flex justify-end gap-3 mt-4">
                        <button onclick="nextDialogue()" class="interactive-element bg-amber-600 hover:bg-amber-700 px-6 py-2 rounded-lg text-white font-bold transition">Continue</button>
                        <button onclick="closeDialogue()" class="interactive-element bg-stone-700 hover:bg-stone-600 px-4 py-2 rounded-lg text-white font-bold transition">Dismiss</button>
                    </div>
                </div>
            </div>

            <!-- Mobile Controls -->
            <div id="stick-left" class="joystick-zone"><div class="stick-nub" id="nub-left"></div></div>
            <div id="stick-right" class="joystick-zone"><div class="stick-nub" id="nub-right"></div></div>
        </div>
    </div>

    <!-- MAIN LOGIC -->
    <script>
        // --- DATA & STATE MANAGEMENT ---
        const gameState = {
            xp: 0,
            rank: "Novice Digger",
            quizzesCompleted: [],
            checklist: {
                boots: false, water: false, sunscreen: false, 
                notebook: false, gloves: false, waiver: false
            }
        };

        const ranks = [
            { xp: 0, title: "Novice Digger" },
            { xp: 500, title: "Field Assistant I" },
            { xp: 1200, title: "Site Technician" },
            { xp: 2500, title: "Master Archaeologist" }
        ];

        // Comprehensive Quiz Database
        const quizModules = [
            {
                id: "tools_101",
                title: "Module 1: Tool Mastery & Excavation Technique",
                questions: [
                    { q: "Which tool is best for removing loose, dry soil from a trench wall?", a: ["Shovel", "Hand Brush (Dustpan & Broom)", "Heavy Pick"], correct: 1 },
                    { q: "What angle should you typically hold a trowel to scrape a context surface?", a: ["45 degrees, flat to the surface", "90 degrees, straight down", "Varies based on context, but usually flat for scraping"], correct: 2 },
                    { q: "What is the primary function of a builder's sieve ('sifter') on site?", a: ["To carry artifacts to the washing station", "To filter all excavated dirt for small, missed artifacts", "To measure the depth of the trench"], correct: 1 },
                    { q: "If you encounter a stone wall in your square, what is the immediate next step?", a: ["Dig through it to see what's on the other side", "Stop, photograph, document its location and structure", "Call the press for a major find"], correct: 1 },
                    { q: "True or False: All excavated soil, including sterile topsoil, must be screened.", a: ["True, always screen everything", "False, only known artifact-rich layers need screening"], correct: 0 },
                    { q: "What is a line level used for?", a: ["Measuring the amount of light in the trench", "Ensuring the base of the trench is horizontal", "Measuring the thickness of artifacts"], correct: 1 },
                    { q: "When using a shovel, how should you lift the load to protect your back?", a: ["Bend at the waist and twist", "Keep your back straight and lift with your legs", "Ask someone else to do it"], correct: 1 },
                    { q: "What is a bucket or container used to transport samples called?", a: ["A wheelbarrow", "A bulk bag", "A context bag or sample tub"], correct: 2 },
                    { q: "The term 'context sheet' refers to:", a: ["A piece of plastic to cover the site", "The primary form for documenting a single layer or feature", "A map of the nearby area"], correct: 1 }
                ]
            },
            {
                id: "geo_basics",
                title: "Module 2: Basic Geophysical Survey",
                questions: [
                    { q: "Which method detects buried magnetic materials like kilns or hearths?", a: ["Resistivity Survey", "Magnetometry", "Ground Penetrating Radar (GPR)"], correct: 1 },
                    { q: "Resistivity surveys measure the ground's ability to resist what?", a: ["Light penetration", "Water absorption", "Electrical current"], correct: 2 },
                    { q: "GPR works by sending what into the ground and recording the echoes?", a: ["Sound waves", "Radio waves", "Magnetic pulses"], correct: 1 },
                    { q: "The main benefit of a geophysical survey is to:", a: ["Dig the trench faster", "Identify potential buried features without digging", "Determine the age of artifacts"], correct: 1 },
                    { q: "Features filled with stone rubble tend to show up as what in a resistivity survey?", a: ["High resistance anomalies", "Low resistance anomalies", "Magnetic anomalies"], correct: 0 },
                    { q: "Features filled with damp organic soil tend to show up as what in a resistivity survey?", a: ["High resistance anomalies", "Low resistance anomalies", "No anomaly"], correct: 1 }
                ]
            }
        ];

        let currentQuiz = null;
        let currentQuestionIndex = 0;
        let correctAnswers = 0;

        // --- LOCAL STORAGE FUNCTIONS & UTILITIES ---
        function showNotification(title, message, type = 'info') {
            const box = document.getElementById('notification-box');
            const titleEl = document.getElementById('notification-title');
            const messageEl = document.getElementById('notification-message');

            let bgColor = 'bg-blue-600';
            let borderColor = 'border-blue-400';

            if (type === 'success') {
                bgColor = 'bg-green-600';
                borderColor = 'border-green-400';
            } else if (type === 'error') {
                bgColor = 'bg-red-600';
                borderColor = 'border-red-400';
            } else if (type === 'warning') {
                bgColor = 'bg-amber-600';
                borderColor = 'border-amber-400';
            }

            box.className = `fixed top-20 right-5 z-20 p-4 border-l-4 ${bgColor} ${borderColor} text-white shadow-lg transition-all duration-300`;
            titleEl.innerText = title;
            messageEl.innerText = message;
            box.style.display = 'block';
            
            setTimeout(() => {
                box.style.opacity = '1';
                box.style.transform = 'translateX(0)';
            }, 50);

            setTimeout(() => {
                box.style.opacity = '0';
                box.style.transform = 'translateX(100%)';
                setTimeout(() => { box.style.display = 'none'; }, 300);
            }, 4000);
        }

        function loadGame() {
            const saved = localStorage.getItem('t2e_save_v2');
            if(saved) {
                const parsed = JSON.parse(saved);
                Object.assign(gameState, {
                    ...gameState, 
                    ...parsed,    
                    checklist: { ...gameState.checklist, ...parsed.checklist } 
                });
            }
            const currentRankEntry = ranks.slice().reverse().find(r => gameState.xp >= r.xp);
            gameState.rank = currentRankEntry ? currentRankEntry.title : ranks[0].title;
            updateStatsUI();
            switchSection(currentSection);
        }

        function saveGame() {
            localStorage.setItem('t2e_save_v2', JSON.stringify(gameState));
            updateStatsUI();
        }

        function addXP(amount) {
            const oldRank = gameState.rank;
            gameState.xp += amount;
            
            let newRankTitle = oldRank;
            for(let i = ranks.length - 1; i >= 0; i--) {
                if(gameState.xp >= ranks[i].xp) {
                    newRankTitle = ranks[i].title;
                    break;
                }
            }

            if(newRankTitle !== oldRank) {
                gameState.rank = newRankTitle;
                showNotification("Promotion Achieved!", `You have been promoted to: ${gameState.rank}!`, 'success');
            }
            saveGame();
        }

        function updateStatsUI() {
            const currentRank = ranks.find(r => r.title === gameState.rank) || ranks[0];
            const currentRankIndex = ranks.indexOf(currentRank);
            const nextRank = ranks[currentRankIndex + 1];
            
            const xpNeeded = nextRank ? (nextRank.xp - currentRank.xp) : 1;
            const xpProgress = gameState.xp - currentRank.xp;
            
            let pct = 100;
            if (nextRank) {
                pct = (xpProgress / xpNeeded) * 100;
            }

            const xpBarWeb = document.getElementById('xp-bar');
            const xpTextWeb = document.getElementById('xp-text');
            const rankDisplayWeb = document.getElementById('rank-display');

            if(xpBarWeb) xpBarWeb.style.width = `${pct}%`;
            if(xpTextWeb) xpTextWeb.innerText = nextRank ? `${xpProgress}/${xpNeeded} XP` : `${gameState.xp} XP (MAX)`;
            if(rankDisplayWeb) rankDisplayWeb.innerText = gameState.rank;

            const xpBarSim = document.getElementById('sim-xp-bar');
            const rankDisplaySim = document.getElementById('sim-rank-display');

            if(xpBarSim) xpBarSim.style.width = `${pct}%`;
            if(rankDisplaySim) rankDisplaySim.innerText = gameState.rank;
        }
        
        function toggleChecklistItem(item) {
            gameState.checklist[item] = !gameState.checklist[item];
            saveGame();
            renderChecklist();
        }

        function renderChecklist() {
            const checklistUI = document.getElementById('checklist-ui');
            if (!checklistUI) return;

            const items = {
                boots: { icon: '🥾', name: 'Steel-Toe Boots' },
                water: { icon: '💧', name: 'Water Bottle/Hydration Pack' },
                sunscreen: { icon: '🧴', name: 'Sunscreen & Hat' },
                notebook: { icon: '📓', name: 'Field Notebook & Pencil' },
                gloves: { icon: '🧤', name: 'Work Gloves' },
                waiver: { icon: '📝', name: 'Signed Waiver' }
            };

            checklistUI.innerHTML = Object.keys(items).map(key => {
                const item = items[key];
                const checked = gameState.checklist[key];
                return `
                    <button onclick="toggleChecklistItem('${key}')" 
                        class="flex items-center p-4 rounded-lg transition text-left 
                        ${checked ? 'bg-blue-100 border-2 border-blue-500' : 'bg-gray-100 border-2 border-gray-300 hover:bg-gray-200'}"
                    >
                        <span class="text-2xl mr-3">${item.icon}</span>
                        <div class="flex-grow">
                            <div class="font-bold ${checked ? 'text-blue-700 line-through' : 'text-stone-800'}">${item.name}</div>
                            <div class="text-sm text-stone-500">${checked ? 'Ready' : 'Missing'}</div>
                        </div>
                        <span class="text-xl">${checked ? '✅' : '⬜'}</span>
                    </button>
                `;
            }).join('');
        }

        // --- SECTION LOGIC ---
        const contentArea = document.getElementById('content-area');
        let currentSection = 'home';

        function switchSection(name) {
            window.scrollTo(0,0);
            currentSection = name;

            document.querySelectorAll('.nav-button').forEach(btn => {
                btn.classList.remove('active');
            });
            const navButton = document.getElementById(`nav-${name}`);
            if (navButton) navButton.classList.add('active');

            let html = "";
            
            if(name === 'home') {
                const currentRank = ranks.find(r => r.title === gameState.rank) || ranks[0];
                const currentRankIndex = ranks.indexOf(currentRank);
                const nextRank = ranks[currentRankIndex + 1];

                html = `
                    <div class="animate-fade-in pb-16">
                        <div class="bg-stone-800 text-white py-20 px-6 text-center shadow-inner">
                            <h1 class="text-5xl md:text-7xl font-serif font-extrabold text-amber-500 mb-2">${gameState.rank}</h1>
                            <p class="text-xl text-stone-300 max-w-3xl mx-auto mb-8">
                                Welcome. Your current goal is to advance to 
                                ${nextRank ? `**${nextRank.title}**` : '**Maximum Rank**'}
                                by mastering the modules below.
                            </p>
                            <div class="max-w-xl mx-auto bg-stone-700 p-4 rounded-xl shadow-lg border border-amber-600">
                                <div class="flex justify-between text-sm text-stone-300 mb-2 font-bold">
                                    <span id="rank-display">${gameState.rank}</span>
                                    <span>${nextRank ? nextRank.title : 'MASTER'}</span>
                                </div>
                                <div class="progress-bar-container h-4">
                                    <div id="xp-bar" class="progress-bar-fill" style="width: 0%"></div>
                                </div>
                                <div class="text-right text-xs mt-1 text-amber-400 font-mono" id="xp-text"></div>
                            </div>
                        </div>
                        <div class="max-w-6xl mx-auto px-4 -mt-10 grid lg:grid-cols-3 gap-8">
                            <div class="bg-white p-6 rounded-xl shadow-2xl border-t-8 border-blue-600 lg:col-span-2">
                                <h2 class="text-3xl font-serif font-bold text-blue-800 mb-4">Site Readiness Protocol</h2>
                                <p class="text-stone-600 mb-4">Ensure all essential items are prepared before entering the field.</p>
                                <div id="checklist-ui" class="grid sm:grid-cols-2 gap-4"></div>
                            </div>
                            <div class="bg-white p-6 rounded-xl shadow-2xl border-t-8 border-green-600">
                                <h2 class="text-3xl font-serif font-bold text-green-800 mb-4">Academy Performance</h2>
                                <div class="space-y-4">
                                    <div class="bg-green-50 p-4 rounded-lg">
                                        <div class="text-4xl font-extrabold text-green-600">${gameState.quizzesCompleted.length} / ${quizModules.length}</div>
                                        <div class="text-sm text-stone-600 uppercase font-bold">Modules Mastered</div>
                                    </div>
                                    <button onclick="switchSection('quiz')" class="w-full bg-green-500 hover:bg-green-600 text-white font-bold py-3 rounded-lg transition shadow-md">Go to Academy</button>
                                </div>
                            </div>
                        </div>
                    </div>
                `;
            } else if (name === 'guides') {
                html = `
                    <div class="animate-fade-in max-w-5xl mx-auto p-8">
                        <h1 class="text-4xl font-serif font-bold text-stone-900 mb-6 border-b pb-2">Field Guides & Resources</h1>
                        <p class="text-lg text-stone-700 mb-8">Access detailed documentation on archaeological practices.</p>
                        <div class="bg-white p-6 rounded-xl shadow-lg border-l-8 border-amber-500">
                            <h3 class="text-2xl font-bold text-amber-700 mb-2">Guides Coming Soon</h3>
                            <p class="text-stone-600">Detailed text guides are being excavated...</p>
                        </div>
                    </div>
                `;
            } else if (name === 'quiz') {
                html = `
                    <div class="animate-fade-in max-w-4xl mx-auto p-8">
                        <h1 class="text-4xl font-serif font-bold text-stone-900 mb-6 border-b pb-2">Excavation Academy Quizzes</h1>
                        <p class="text-lg text-stone-700 mb-8">Test your knowledge. Passing a module earns you 150 XP.</p>
                        <div class="space-y-6">
                            ${quizModules.map(module => {
                                const isComplete = gameState.quizzesCompleted.includes(module.id);
                                return `
                                    <div class="bg-white p-6 rounded-xl shadow-lg ${isComplete ? 'border-l-8 border-green-500' : 'border-l-8 border-red-500'}">
                                        <h3 class="text-2xl font-bold ${isComplete ? 'text-green-700' : 'text-red-700'} mb-2">${module.title}</h3>
                                        <div class="text-stone-600 mb-4 flex justify-between items-center">
                                            <span>Questions: ${module.questions.length} | XP Reward: 150</span>
                                            <span class="font-bold text-lg">${isComplete ? 'STATUS: MASTERED' : 'STATUS: PENDING'}</span>
                                        </div>
                                        <button onclick="startQuiz('${module.id}')" ${isComplete ? 'disabled' : ''} class="px-6 py-2 rounded-lg font-bold transition ${isComplete ? 'bg-gray-400 cursor-not-allowed' : 'bg-amber-600 hover:bg-amber-700 text-white shadow-md'}">
                                            ${isComplete ? 'Module Complete' : 'Start Quiz'}
                                        </button>
                                    </div>
                                `;
                            }).join('')}
                        </div>
                    </div>
                `;
            }

            contentArea.innerHTML = html;
            if (name === 'home') {
                renderChecklist();
                updateStatsUI();
            }
        }
        
        // --- QUIZ LOGIC ---
        function startQuiz(quizId) {
            currentQuiz = quizModules.find(m => m.id === quizId);
            currentQuestionIndex = 0;
            correctAnswers = 0;
            if (currentQuiz) {
                openDialogue("Quiz Master", `Welcome to the ${currentQuiz.title}! Answer 8/10 to pass.`);
                document.getElementById('dialogue-controls').style.display = 'none'; 
                showQuestion();
            }
        }
        
        function showQuestion() {
            if (currentQuestionIndex >= currentQuiz.questions.length) {
                endQuiz();
                return;
            }
            const q = currentQuiz.questions[currentQuestionIndex];
            document.getElementById('npc-name').innerText = currentQuiz.title;
            document.getElementById('npc-text').innerHTML = `Question ${currentQuestionIndex + 1}/${currentQuiz.questions.length}:<br><strong class="text-amber-300">${q.q}</strong>`;
            
            const optionsContainer = document.getElementById('quiz-options');
            optionsContainer.innerHTML = q.a.map((option, index) => `
                <button onclick="submitAnswer(${index})" class="text-left p-3 rounded-lg bg-stone-700 hover:bg-stone-600 transition text-stone-100 interactive-element">
                    ${String.fromCharCode(65 + index)}. ${option}
                </button>
            `).join('');
            optionsContainer.style.display = 'flex';
            document.getElementById('dialogue-controls').style.display = 'none'; 
        }

        function submitAnswer(selectedIndex) {
            const q = currentQuiz.questions[currentQuestionIndex];
            const isCorrect = selectedIndex === q.correct;
            
            document.querySelectorAll('#quiz-options button').forEach((btn, index) => {
                btn.disabled = true;
                if (index === q.correct) btn.classList.add('bg-green-700');
                else if (index === selectedIndex) btn.classList.add('bg-red-700');
                else btn.classList.add('opacity-50');
            });

            if (isCorrect) correctAnswers++;
            currentQuestionIndex++;
            
            const nextButton = document.createElement('button');
            nextButton.innerText = 'Next Question';
            nextButton.onclick = showQuestion;
            nextButton.className = "interactive-element bg-amber-600 hover:bg-amber-700 px-6 py-2 rounded-lg text-white font-bold transition mt-4 self-end";
            
            const controls = document.getElementById('dialogue-controls');
            controls.style.display = 'flex';
            controls.innerHTML = '';
            controls.appendChild(nextButton);
        }

        function endQuiz() {
            const minQuestionsToPass = currentQuiz.id === 'geo_basics' ? 4 : 8; // Adjust threshold for shorter quizzes
            const passed = correctAnswers >= minQuestionsToPass;
            const xpEarned = passed ? 150 : 0;
            const totalQuestions = currentQuiz.questions.length;

            if (passed && !gameState.quizzesCompleted.includes(currentQuiz.id)) {
                gameState.quizzesCompleted.push(currentQuiz.id);
                addXP(xpEarned);
                showNotification("Module Mastered!", `${currentQuiz.title} passed! +150 XP awarded.`, 'success');
            } else if (passed) {
                showNotification("Already Mastered", `You passed again.`, 'info');
            } else {
                showNotification("Module Failed", `Score: ${correctAnswers}/${totalQuestions}. Need ${minQuestionsToPass} to pass.`, 'error');
            }
            closeDialogue();
            switchSection('quiz');
        }

        // --- DIALOGUE SYSTEM ---
        let currentNPC = null;
        
        function openDialogue(name, text) {
            document.getElementById('npc-name').innerText = name;
            document.getElementById('npc-text').innerText = text;
            document.getElementById('quiz-options').style.display = 'none';
            document.getElementById('dialogue-controls').style.display = 'flex';
            document.getElementById('dialogue-box').style.display = 'block';
            document.getElementById('crosshair').style.opacity = '0';
        }

        function closeDialogue() {
            document.getElementById('dialogue-box').style.display = 'none';
            document.getElementById('crosshair').style.opacity = '1';
            currentNPC = null;
            if (simContainer.style.display === 'block') enableControls();
        }

        function nextDialogue() {
            if (!currentNPC) return;
            const dialogueData = currentNPC.userData.dialogue;
            let index = currentNPC.userData.currentDialogueIndex;
            index++;
            if (index < dialogueData.length) {
                currentNPC.userData.currentDialogueIndex = index;
                const nextLine = dialogueData[index];
                document.getElementById('npc-text').innerText = nextLine.text;
                if (nextLine.quizId) {
                    document.getElementById('dialogue-controls').style.display = 'none';
                    startQuiz(nextLine.quizId);
                    return;
                }
            } else {
                closeDialogue();
                currentNPC.userData.currentDialogueIndex = 0;
            }
        }
        
        function interactWithNPC(npc) {
            currentNPC = npc;
            npc.userData.currentDialogueIndex = 0; 
            const dialogueData = npc.userData.dialogue;
            if (dialogueData && dialogueData.length > 0) {
                disableControls();
                const firstLine = dialogueData[0];
                openDialogue(npc.userData.name, firstLine.text);
                if (firstLine.quizId) {
                    document.getElementById('dialogue-controls').style.display = 'none';
                    startQuiz(firstLine.quizId);
                } else {
                    document.getElementById('dialogue-controls').style.display = 'flex';
                }
            }
        }


        // --- 3D SIMULATION LOGIC ---
        let scene, camera, renderer;
        let player, npcs = [];
        let raycaster = new THREE.Raycaster();
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false;
        let canJump = false;
        let velocity = new THREE.Vector3();
        let direction = new THREE.Vector3();
        let prevTime = performance.now();
        let simActive = false;
        
        const simContainer = document.getElementById('sim-container');
        const interactionPrompt = document.getElementById('interaction-prompt');

        // --- Player Configuration ---
        const playerHeight = 1.8;
        const playerSpeed = 5.0; // FIXED: Reduced speed for better control
        const playerGravity = 9.8;
        
        // --- Input Handling ---
        function lockPointer() {
            if (document.pointerLockElement !== renderer.domElement) {
                renderer.domElement.requestPointerLock();
            }
        }
        
        function setupControls() {
            document.addEventListener('keydown', onKeyDown, false);
            document.addEventListener('keyup', onKeyUp, false);
            simContainer.addEventListener('mousedown', onMouseDown, false);
            simContainer.addEventListener('touchstart', onTouchStart, false);
            document.addEventListener('mousemove', onMouseMove, false);
            
            simContainer.addEventListener('click', (event) => {
                if(document.getElementById('dialogue-box').style.display === 'block') {
                    event.stopPropagation();
                }
            });

            setupMobileJoystick();
            if ('ontouchstart' in window) {
                document.getElementById('stick-left').style.display = 'block';
                document.getElementById('stick-right').style.display = 'block';
            }
        }

        function onKeyDown(event) {
            if (!simActive) return;
            switch (event.code) {
                case 'KeyW': moveForward = true; break;
                case 'KeyS': moveBackward = true; break;
                case 'KeyA': moveLeft = true; break;
                case 'KeyD': moveRight = true; break;
                case 'Space': if (canJump === true) velocity.y += 150; canJump = false; break;
                case 'KeyE': 
                    if (currentNPC) nextDialogue(); 
                    else checkInteraction(); 
                    break;
            }
        }

        function onKeyUp(event) {
            if (!simActive) return;
            switch (event.code) {
                case 'KeyW': moveForward = false; break;
                case 'KeyS': moveBackward = false; break;
                case 'KeyA': moveLeft = false; break;
                case 'KeyD': moveRight = false; break;
            }
        }

        function onMouseDown(event) {
            if (!simActive || document.getElementById('dialogue-box').style.display === 'block') return;
            lockPointer();
            if (document.pointerLockElement === renderer.domElement) checkInteraction();
        }
        
        function onMouseMove(event) {
            if (!simActive || document.pointerLockElement !== renderer.domElement || document.getElementById('dialogue-box').style.display === 'block') return;
            
            const sensitivity = 0.002; 
            
            // Y rotation (Horizontal look)
            camera.rotation.y -= event.movementX * sensitivity * 1.5; 
            
            // X rotation (Vertical look)
            camera.rotation.x -= event.movementY * sensitivity * 1.5; 
            
            // Clamp vertical rotation (look up/down limit)
            camera.rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, camera.rotation.x));
        }
        
        // --- Mobile Joystick Logic (Simplified) ---
        const joystick = { left: { active: false, x: 0, y: 0, nub: null, zone: null, identifier: null }, right: { active: false, x: 0, y: 0, nub: null, zone: null, identifier: null } };
        const maxDist = 70; 

        function setupMobileJoystick() {
            joystick.left.nub = document.getElementById('nub-left');
            joystick.left.zone = document.getElementById('stick-left');
            joystick.right.nub = document.getElementById('nub-right');
            joystick.right.zone = document.getElementById('stick-right');
            if (!joystick.left.zone) return;

            document.addEventListener('touchstart', onTouchStart, { passive: false });
            document.addEventListener('touchmove', onTouchMove, { passive: false });
            document.addEventListener('touchend', onTouchEnd, { passive: false });
            document.addEventListener('touchcancel', onTouchEnd, { passive: false }); // Added touchcancel
        }

        function onTouchStart(event) {
            if (!simActive || document.getElementById('dialogue-box').style.display === 'block') return;
            let isJoystickTouch = false;
            for (let i = 0; i < event.changedTouches.length; i++) {
                const touch = event.changedTouches[i];
                if (joystick.left.zone.contains(touch.target) && !joystick.left.active) {
                    isJoystickTouch = true;
                    joystick.left.active = true; 
                    joystick.left.identifier = touch.identifier;
                    break;
                } else if (joystick.right.zone.contains(touch.target) && !joystick.right.active) {
                    isJoystickTouch = true;
                    joystick.right.active = true; 
                    joystick.right.identifier = touch.identifier;
                    break;
                }
            }
            if (!isJoystickTouch) checkInteraction();
        }

        function onTouchMove(event) {
            if (!simActive) return;
            event.preventDefault(); // Prevent scrolling on touch move
            for (let i = 0; i < event.changedTouches.length; i++) {
                const touch = event.changedTouches[i];
                if (joystick.left.active && touch.identifier === joystick.left.identifier) {
                    updateJoystick(joystick.left, touch, joystick.left.zone);
                }
                else if (joystick.right.active && touch.identifier === joystick.right.identifier) {
                    updateJoystick(joystick.right, touch, joystick.right.zone, true);
                }
            }
        }

        function updateJoystick(stick, touch, zone, isCamera = false) {
            const rect = zone.getBoundingClientRect();
            const center = { x: rect.width / 2 + rect.left, y: rect.height / 2 + rect.top };
            let x = touch.clientX - center.x;
            let y = touch.clientY - center.y;
            const distance = Math.sqrt(x * x + y * y);
            
            // Clamp position if outside maxDist
            if (distance > maxDist) {
                const angle = Math.atan2(y, x);
                x = Math.cos(angle) * maxDist;
                y = Math.sin(angle) * maxDist;
            }
            
            // Update stick position (for visual feedback)
            stick.nub.style.transform = `translate(${x}px, ${y}px)`;
            
            // Normalize stick position for control (range -1.0 to 1.0)
            stick.x = x / maxDist;
            stick.y = y / maxDist;
            
            if (isCamera) {
                // Control camera rotation (look)
                camera.rotation.y -= stick.x * 0.04;
                camera.rotation.x -= stick.y * 0.04;
                camera.rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, camera.rotation.x));
            } else {
                // Control movement flags
                moveForward = stick.y < -0.1;
                moveBackward = stick.y > 0.1;
                moveLeft = stick.x < -0.1;
                moveRight = stick.x > 0.1;
            }
        }

        function onTouchEnd(event) {
            for (let i = 0; i < event.changedTouches.length; i++) {
                const touch = event.changedTouches[i];
                if (joystick.left.active && touch.identifier === joystick.left.identifier) {
                    joystick.left.active = false; 
                    joystick.left.identifier = null;
                    joystick.left.nub.style.transform = 'translate(-50%, -50%)';
                    // Reset movement flags
                    moveForward = moveBackward = moveLeft = moveRight = false;
                } else if (joystick.right.active && touch.identifier === joystick.right.identifier) {
                    joystick.right.active = false; 
                    joystick.right.identifier = null;
                    joystick.right.nub.style.transform = 'translate(-50%, -50%)';
                }
            }
        }

        function disableControls() {
            moveForward = moveBackward = moveLeft = moveRight = false;
            if (document.pointerLockElement === renderer.domElement) document.exitPointerLock();
        }

        function enableControls() {
             if (!('ontouchstart' in window)) lockPointer();
        }
        
        // --- NPC Interaction ---
        function checkInteraction() {
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const intersects = raycaster.intersectObjects(npcs, true);

            if (intersects.length > 0) {
                let npcGroup = intersects[0].object;
                // Traverse up to find the main NPC group object
                while (npcGroup.parent && !npcGroup.userData.isNPC) {
                    npcGroup = npcGroup.parent;
                }
                // Check if it's an NPC and within interaction distance (5 units)
                if (npcGroup.userData.isNPC && npcGroup.position.distanceTo(camera.position) < 5) {
                    interactWithNPC(npcGroup);
                }
            }
        }

        // --- FIXED INIT SIM FUNCTION ---
        function initSim() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87ceeb); 
            scene.fog = new THREE.Fog(0x87ceeb, 10, 50);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.y = playerHeight;
            camera.position.z = 8; // Spawn point moved back from trench (z=0)
            
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;

            const simDiv = document.getElementById('sim-container');
            
            const existingCanvas = simDiv.querySelector('canvas');
            if (existingCanvas) {
                simDiv.removeChild(existingCanvas);
            }
            simDiv.insertBefore(renderer.domElement, simDiv.firstChild);

            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);

            const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0);
            directionalLight.position.set(50, 100, 50); 
            directionalLight.castShadow = true;
            directionalLight.shadow.mapSize.width = 2048; 
            directionalLight.shadow.mapSize.height = 2048;
            directionalLight.shadow.camera.near = 0.5;
            directionalLight.shadow.camera.far = 500;
            directionalLight.shadow.camera.left = -50;
            directionalLight.shadow.camera.right = 50;
            directionalLight.shadow.camera.top = 50;
            directionalLight.shadow.camera.bottom = -50;
            scene.add(directionalLight);

            // Ground
            const groundGeometry = new THREE.PlaneGeometry(100, 100);
            const groundMaterial = new THREE.MeshLambertMaterial({ color: 0x5c4033 }); // Brown dirt
            const ground = new THREE.Mesh(groundGeometry, groundMaterial);
            ground.rotation.x = -Math.PI / 2; 
            ground.receiveShadow = true;
            scene.add(ground);
            
            // Trench (centered at x=0, z=0)
            createTrench(0, -0.5, 0, 10, 10, 1); 

            npcs = [];
            // Dr. Eleanor Vance (Original Supervisor) - Near Trench Corner
            addNPC(6, 0, 6, "Dr. Eleanor Vance", 0x8B4513, 0xFAD5A5, [
                { text: `Welcome, Volunteer. I'm Dr. Vance, the Site Supervisor. Your primary task is tool proficiency.`, quizId: null },
                { text: "Before you start digging, you must pass the Tool Mastery module. Speak to Dr. Jones for site history.", quizId: "tools_101" }
            ]);
            
            // Dr. Henry Jones Jr. (Archaeologist) - Near Trench Side
            addNPC(-5, 0, 1, "Dr. Henry Jones Jr.", 0xCC7722, 0xFAD5A5, [
                { text: "Well met, junior archaeologist. I'm Dr. Jones. We're currently excavating a Late Roman-era villa.", quizId: null },
                { text: "Remember: 'It belongs in a museum,' but first, it belongs in a context bag with documentation. Any questions?" },
                { text: "Excellent. Pass the Tool Mastery quiz with Dr. Vance, then talk to Maya about geophysical results." }
            ]);

            // Maya Sharma (Geophysicist) - Far from Trench, near a 'machine' (represented by a cube)
            const machineGeometry = new THREE.BoxGeometry(2, 1, 2);
            const machineMaterial = new THREE.MeshPhongMaterial({ color: 0x4B0082 });
            const machine = new THREE.Mesh(machineGeometry, machineMaterial);
            machine.position.set(-10, 0.5, -10);
            machine.castShadow = true;
            scene.add(machine);

            addNPC(-8, 0, -10, "Maya Sharma (Geophysicist)", 0x228B22, 0xFFA07A, [
                { text: "Hi! I'm Maya. I ran the survey here. See that machine? It helps us map anomalies before we dig.", quizId: null },
                { text: "It's crucial to understand what lies beneath. Ready for a quick test on the basics of remote sensing?", quizId: "geo_basics" }
            ]);


            window.addEventListener('resize', onWindowResize, false);
        }
        
        function onWindowResize() {
            if (camera) {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
            }
            if (renderer) {
                renderer.setSize(window.innerWidth, window.innerHeight);
            }
        }

        function createTrench(x, y, z, width, depth, height) {
            const trenchWallMaterial = new THREE.MeshPhongMaterial({ color: 0x807663 });
            
            // Front Wall
            let geometry = new THREE.BoxGeometry(width, height, 0.1);
            let wall = new THREE.Mesh(geometry, trenchWallMaterial);
            wall.position.set(x, y + height / 2, z - depth / 2);
            wall.castShadow = true;
            wall.receiveShadow = true;
            scene.add(wall);
            
            // Back Wall
            wall = new THREE.Mesh(geometry, trenchWallMaterial);
            wall.position.set(x, y + height / 2, z + depth / 2);
            wall.castShadow = true;
            wall.receiveShadow = true;
            scene.add(wall);

            // Left Wall
            geometry = new THREE.BoxGeometry(0.1, height, depth);
            wall = new THREE.Mesh(geometry, trenchWallMaterial);
            wall.position.set(x - width / 2, y + height / 2, z);
            wall.castShadow = true;
            wall.receiveShadow = true;
            scene.add(wall);
            
            // Right Wall
            wall = new THREE.Mesh(geometry, trenchWallMaterial);
            wall.position.set(x + width / 2, y + height / 2, z);
            wall.castShadow = true;
            wall.receiveShadow = true;
            scene.add(wall);

            // Floor
            const floorGeometry = new THREE.PlaneGeometry(width - 0.1, depth - 0.1);
            const floorMaterial = new THREE.MeshLambertMaterial({ color: 0x3d2b1f }); 
            const floor = new THREE.Mesh(floorGeometry, floorMaterial);
            floor.rotation.x = -Math.PI / 2;
            floor.position.set(x, y, z);
            floor.receiveShadow = true;
            scene.add(floor);
        }

        // Added color arguments for different NPCs
        function addNPC(x, y, z, name, bodyColor, headColor, dialogue) {
            const npcGroup = new THREE.Group();
            npcGroup.name = name;
            npcGroup.position.set(x, y, z);

            const bodyMaterial = new THREE.MeshPhongMaterial({ color: bodyColor });
            const headMaterial = new THREE.MeshPhongMaterial({ color: headColor });

            const bodyGeometry = new THREE.CylinderGeometry(0.3, 0.4, 1.6, 16);
            const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
            body.position.y = 0.8;
            body.castShadow = true;
            npcGroup.add(body);
            
            const headGeometry = new THREE.SphereGeometry(0.35, 32, 16);
            const head = new THREE.Mesh(headGeometry, headMaterial);
            head.position.y = 1.8;
            head.castShadow = true;
            npcGroup.add(head);

            npcGroup.userData = { 
                name: name, 
                isNPC: true, 
                dialogue: dialogue,
                currentDialogueIndex: 0
            };
            
            npcs.push(npcGroup);
            scene.add(npcGroup);
        }
        
        // Tool HUD Logic
        let currentToolIndex = 0;
        const toolIcons = document.querySelectorAll('.tool-icon');
        
        function updateToolHUD() {
            toolIcons.forEach((icon, index) => {
                icon.classList.remove('active');
                if (index === currentToolIndex) {
                    icon.classList.add('active');
                }
            });
        }
        
        function switchTool(index) {
            currentToolIndex = index % toolIcons.length;
            updateToolHUD();
            showNotification("Tool Switched", `Active Tool: ${toolIcons[currentToolIndex].dataset.tool}`, 'info');
        }

        function animate() {
            if (!simActive) return;
            requestAnimationFrame(animate);

            const time = performance.now();
            const delta = (time - prevTime) / 1000;
            prevTime = time;

            // Apply friction/drag and gravity
            velocity.x -= velocity.x * 10.0 * delta;
            velocity.z -= velocity.z * 10.0 * delta;
            velocity.y -= playerGravity * 50.0 * delta; 

            // Calculate movement vector (local to camera)
            direction.z = Number(moveForward) - Number(moveBackward);
            direction.x = Number(moveRight) - Number(moveLeft);
            
            if (direction.length() > 0) {
                direction.normalize(); 
                // Apply velocity from input
                velocity.z = direction.z * playerSpeed;
                velocity.x = direction.x * playerSpeed;
            } else {
                velocity.x = 0;
                velocity.z = 0;
            }

            // Apply translation based on camera's local axes (this handles the rotation automatically)
            // Use camera.position.add and rotate the camera's directional vector instead of translate for WASD movement relative to camera look direction
            if (moveForward || moveBackward || moveLeft || moveRight) {
                const rotation = new THREE.Euler(0, camera.rotation.y, 0, 'YXZ');
                const forward = new THREE.Vector3(0, 0, -1).applyEuler(rotation).multiplyScalar(velocity.z * delta);
                const right = new THREE.Vector3(1, 0, 0).applyEuler(rotation).multiplyScalar(velocity.x * delta);
                camera.position.add(forward).add(right);
            }

            camera.position.y += velocity.y * delta;
            
            // Ground Clamp (Gravity)
            if (camera.position.y < playerHeight) {
                velocity.y = 0;
                camera.position.y = playerHeight;
                canJump = true;
            }

            // Prompt Logic
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const intersects = raycaster.intersectObjects(npcs, true);
            if (intersects.length > 0) {
                let npcGroup = intersects[0].object;
                while (npcGroup.parent && !npcGroup.userData.isNPC) npcGroup = npcGroup.parent;
                if (npcGroup.userData.isNPC && npcGroup.position.distanceTo(camera.position) < 5) {
                    interactionPrompt.style.opacity = '1';
                } else {
                    interactionPrompt.style.opacity = '0';
                }
            } else {
                interactionPrompt.style.opacity = '0';
            }

            renderer.render(scene, camera);
        }

        function startSimulation() {
            if (!renderer) {
                initSim();
                // Attach tool switching listener
                document.addEventListener('keydown', (event) => {
                    if (simActive && event.key >= '1' && event.key <= '4') {
                        switchTool(parseInt(event.key) - 1);
                    }
                });
                toolIcons.forEach((icon, index) => {
                    icon.addEventListener('click', () => switchTool(index));
                });
            }
            simActive = true;
            simContainer.style.display = 'block';
            document.getElementById('website-container').style.display = 'none';
            prevTime = performance.now();
            animate();
            if (!('ontouchstart' in window)) lockPointer();
            updateStatsUI();
            updateToolHUD();
            showNotification("Simulation Started", "WASD/Joystick to Move, Mouse/Right Joystick to Look, E/Click/Tap to Interact. Press 1-4 to switch tools.", 'warning');
        }

        function exitSimulation() {
            simActive = false;
            simContainer.style.display = 'none';
            document.getElementById('website-container').style.display = 'flex';
            if (document.pointerLockElement) document.exitPointerLock();
            closeDialogue();
            saveGame();
        }

        window.onload = function () {
            loadGame();
            setupControls();
        }
    </script>
</body>
</html>
