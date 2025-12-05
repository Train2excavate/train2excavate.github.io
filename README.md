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
            width: 8px;
            height: 8px;
            margin: -4px 0 0 -4px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.8);
            border: 1px solid #000;
            z-index: 1001;
            box-shadow: 0 0 4px #000;
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
            display: none; 
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

        /* Dialogue UI */
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

        /* Tutorial Modal */
        #tutorial-modal {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(17, 24, 39, 0.95);
            border: 2px solid #f59e0b;
            padding: 30px;
            border-radius: 15px;
            color: white;
            z-index: 3000;
            text-align: center;
            max-width: 500px;
            pointer-events: auto;
            display: none;
            box-shadow: 0 0 50px rgba(0,0,0,0.8);
        }

        /* Notification */
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

        /* Progress Bar */
        .progress-bar-container { width: 100%; background-color: #44403c; border-radius: 999px; overflow: hidden; height: 12px; }
        .progress-bar-fill { height: 100%; background-color: #f59e0b; transition: width 0.5s ease-out; box-shadow: 0 0 8px #f59e0b; }
        
        .nav-button.active::after { content: ''; position: absolute; bottom: -8px; left: 0; width: 100%; height: 3px; background: #fcd34d; border-radius: 2px; }
        .animate-fade-in { animation: fadeIn 0.5s ease-out forwards; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
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
        <nav class="bg-stone-900 text-stone-100 p-4 sticky top-0 z-50 shadow-xl border-b-4 border-amber-600">
            <div class="max-w-7xl mx-auto flex flex-col md:flex-row justify-between items-center gap-4">
                <div class="text-3xl font-extrabold text-amber-500 cursor-pointer">⛏️ Train2Excavate</div>
                <div class="flex space-x-8 text-lg font-bold">
                    <button id="nav-home" onclick="switchSection('home')" class="nav-button active">HUB</button>
                    <button id="nav-quiz" onclick="switchSection('quiz')" class="nav-button">ACADEMY</button>
                </div>
                <button onclick="initSimPrompt()" class="bg-amber-600 hover:bg-amber-700 text-white px-6 py-2 rounded-xl font-bold shadow-lg border border-amber-400 transform hover:scale-105 transition">
                    ENTER DIG SITE
                </button>
            </div>
        </nav>

        <main id="content-area" class="flex-grow"></main>

        <footer class="bg-stone-900 text-stone-500 py-6 mt-auto text-center border-t border-stone-800">
            <p>Train2Excavate &copy; 2024</p>
        </footer>
    </div>

    <!-- 3D SIMULATION OVERLAY -->
    <div id="sim-container">
        <div id="crosshair"></div>
        
        <div id="ui-layer">
            <div class="p-6 flex justify-between items-start">
                <div class="p-3 bg-black/40 rounded-lg shadow-xl backdrop-blur-sm interactive-element">
                    <h2 class="text-3xl font-bold text-amber-400 font-serif">Site 42</h2>
                    <p class="text-stone-300 text-sm">Current Objective: <span id="objective-text" class="text-white font-bold">Locate Supervisor</span></p>
                </div>
                <div id="tool-hud">
                    <span class="tool-icon active" data-tool="trowel">⛏️</span>
                    <span class="tool-icon" data-tool="brush">🧹</span>
                    <span class="tool-icon" data-tool="sifter">🧺</span>
                    <span class="tool-icon" data-tool="tape">📏</span>
                </div>
                <button onclick="exitSimulation()" class="interactive-element bg-red-700 hover:bg-red-800 text-white px-5 py-2 rounded-xl font-bold shadow-xl border border-red-400 transition transform hover:scale-105">EXIT SIM</button>
            </div>

            <!-- TUTORIAL MODAL -->
            <div id="tutorial-modal">
                <h2 class="text-2xl font-bold text-amber-500 mb-4">Welcome to Site 42</h2>
                <p class="text-gray-300 mb-6">New to the site? Would you like a quick tutorial on how to move and interact with the environment?</p>
                <div class="flex justify-center gap-4">
                    <button onclick="startTutorial()" class="bg-green-600 hover:bg-green-700 px-6 py-2 rounded text-white font-bold">Yes, Guide Me</button>
                    <button onclick="skipTutorial()" class="bg-stone-600 hover:bg-stone-700 px-6 py-2 rounded text-white font-bold">No, I'm Ready</button>
                </div>
            </div>

            <div id="interaction-prompt" class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 transition-opacity duration-200 opacity-0 pointer-events-none">
                <div class="bg-black/80 backdrop-blur text-white px-8 py-3 rounded-full border-2 border-amber-500 font-bold text-lg shadow-xl">
                    [E] Interact
                </div>
            </div>

            <div id="dialogue-container" class="w-full flex justify-center pb-20 md:pb-8">
                <div id="dialogue-box">
                    <h3 id="npc-name" class="text-2xl font-bold text-amber-500 mb-3 border-b-2 border-stone-600 pb-1">Name</h3>
                    <p id="npc-text" class="text-lg leading-relaxed mb-6 text-stone-200">...</p>
                    <div id="quiz-options" class="flex flex-col gap-3"></div>
                    <div id="dialogue-controls" class="flex justify-end gap-3 mt-4">
                        <button onclick="nextDialogue()" class="interactive-element bg-amber-600 hover:bg-amber-700 px-6 py-2 rounded-lg text-white font-bold transition">Continue</button>
                        <button onclick="closeDialogue()" class="interactive-element bg-stone-700 hover:bg-stone-600 px-4 py-2 rounded-lg text-white font-bold transition">Dismiss</button>
                    </div>
                </div>
            </div>

            <div id="stick-left" class="joystick-zone"><div class="stick-nub" id="nub-left"></div></div>
            <div id="stick-right" class="joystick-zone"><div class="stick-nub" id="nub-right"></div></div>
        </div>
    </div>

    <!-- MAIN LOGIC -->
    <script>
        // --- DATA & STATE ---
        const gameState = {
            xp: 0,
            rank: "Novice Digger",
            quizzesCompleted: [],
            checklist: { boots: false, water: false, sunscreen: false, notebook: false, gloves: false, waiver: false }
        };

        const ranks = [
            { xp: 0, title: "Novice Digger" },
            { xp: 500, title: "Field Assistant I" },
            { xp: 1200, title: "Site Technician" },
            { xp: 2500, title: "Master Archaeologist" }
        ];

        const quizModules = [
            {
                id: "tools_101",
                title: "Module 1: Tool Mastery",
                questions: [
                    { q: "Which tool is for removing loose, dry soil?", a: ["Shovel", "Hand Brush", "Heavy Pick"], correct: 1 },
                    { q: "Correct angle for a trowel?", a: ["45 degrees", "90 degrees", "Flat/Scraping"], correct: 2 },
                    { q: "Purpose of a sieve?", a: ["Artifact transport", "Filter dirt for small finds", "Measure depth"], correct: 1 },
                    { q: "If you find a wall?", a: ["Dig through it", "Stop & Document", "Remove it"], correct: 1 }
                ]
            },
            {
                id: "geo_basics",
                title: "Module 2: Geophysics",
                questions: [
                    { q: "Detects burnt magnetic features?", a: ["Resistivity", "Magnetometry", "GPR"], correct: 1 },
                    { q: "Stone walls show up in resistivity as?", a: ["High resistance", "Low resistance", "No reading"], correct: 0 },
                    { q: "GPR uses?", a: ["Sound", "Radio waves", "Lasers"], correct: 1 }
                ]
            }
        ];

        let currentQuiz = null, currentQuestionIndex = 0, correctAnswers = 0;
        let activeObjective = "Find Dr. Vance";
        let targetObject = null; // The 3D object the trail points to

        // --- HELPER FUNCTIONS ---
        function showNotification(title, message, type = 'info') {
            const box = document.getElementById('notification-box');
            const colors = type === 'success' ? ['bg-green-600', 'border-green-400'] : type === 'error' ? ['bg-red-600', 'border-red-400'] : ['bg-blue-600', 'border-blue-400'];
            box.className = `fixed top-20 right-5 z-20 p-4 border-l-4 ${colors[0]} ${colors[1]} text-white shadow-lg`;
            document.getElementById('notification-title').innerText = title;
            document.getElementById('notification-message').innerText = message;
            box.style.display = 'block';
            box.style.opacity = '1';
            box.style.transform = 'translateX(0)';
            setTimeout(() => {
                box.style.opacity = '0';
                box.style.transform = 'translateX(100%)';
                setTimeout(() => box.style.display = 'none', 300);
            }, 4000);
        }

        // --- GAME LOGIC (Website Side) ---
        const contentArea = document.getElementById('content-area');
        let currentSection = 'home';

        function switchSection(name) {
            window.scrollTo(0,0);
            currentSection = name;
            document.querySelectorAll('.nav-button').forEach(btn => btn.classList.remove('active'));
            const navButton = document.getElementById(`nav-${name}`);
            if (navButton) navButton.classList.add('active');

            let html = "";
            if(name === 'home') {
                html = `
                    <div class="animate-fade-in pb-16">
                        <div class="bg-stone-800 text-white py-20 px-6 text-center shadow-inner">
                            <h1 class="text-5xl md:text-7xl font-serif font-extrabold text-amber-500 mb-2">${gameState.rank}</h1>
                            <p class="text-xl text-stone-300 max-w-3xl mx-auto mb-8">Complete modules to rank up.</p>
                            <div class="max-w-xl mx-auto bg-stone-700 p-4 rounded-xl shadow-lg border border-amber-600">
                                <div class="progress-bar-container h-4"><div id="xp-bar" class="progress-bar-fill" style="width: 0%"></div></div>
                                <div class="text-right text-xs mt-1 text-amber-400 font-mono" id="xp-text"></div>
                            </div>
                        </div>
                        <div class="max-w-6xl mx-auto px-4 -mt-10 grid lg:grid-cols-3 gap-8">
                            <div class="bg-white p-6 rounded-xl shadow-2xl border-t-8 border-blue-600 lg:col-span-2">
                                <h2 class="text-2xl font-bold text-blue-800 mb-4">Site Readiness Checklist</h2>
                                <div id="checklist-ui" class="grid sm:grid-cols-2 gap-4"></div>
                            </div>
                            <div class="bg-white p-6 rounded-xl shadow-2xl border-t-8 border-green-600">
                                <h2 class="text-2xl font-bold text-green-800 mb-4">Performance</h2>
                                <div class="bg-green-50 p-4 rounded-lg text-center">
                                    <div class="text-4xl font-extrabold text-green-600">${gameState.quizzesCompleted.length} / ${quizModules.length}</div>
                                    <div class="text-sm text-stone-600 font-bold">Modules Complete</div>
                                </div>
                            </div>
                        </div>
                    </div>`;
            } else if (name === 'quiz') {
                html = `
                    <div class="animate-fade-in max-w-4xl mx-auto p-8">
                        <h1 class="text-4xl font-serif font-bold text-stone-900 mb-6">Excavation Academy</h1>
                        <div class="space-y-6">
                            ${quizModules.map(m => `
                                <div class="bg-white p-6 rounded-xl shadow-lg border-l-8 ${gameState.quizzesCompleted.includes(m.id) ? 'border-green-500' : 'border-red-500'}">
                                    <h3 class="text-2xl font-bold text-stone-800">${m.title}</h3>
                                    <p class="text-stone-600 mb-4">Status: ${gameState.quizzesCompleted.includes(m.id) ? 'Passed' : 'Pending'}</p>
                                </div>
                            `).join('')}
                        </div>
                    </div>`;
            }
            contentArea.innerHTML = html;
            if (name === 'home') { renderChecklist(); updateStatsUI(); }
        }

        function renderChecklist() {
            const items = { boots: '🥾 Boots', water: '💧 Water', sunscreen: '🧴 Sunscreen', notebook: '📓 Notebook' };
            document.getElementById('checklist-ui').innerHTML = Object.keys(items).map(k => `
                <button onclick="toggleCheck('${k}')" class="flex items-center p-4 rounded-lg border-2 transition ${gameState.checklist[k] ? 'bg-blue-100 border-blue-500' : 'bg-gray-50 border-gray-200'}">
                    <span class="text-xl mr-2">${gameState.checklist[k] ? '✅' : '⬜'}</span> <span class="font-bold">${items[k]}</span>
                </button>
            `).join('');
        }
        
        function toggleCheck(k) { gameState.checklist[k] = !gameState.checklist[k]; renderChecklist(); }
        function updateStatsUI() {
            const pct = (gameState.xp % 500) / 5; 
            const bar = document.getElementById('xp-bar');
            if(bar) bar.style.width = `${pct}%`;
        }

        // --- 3D SIMULATION & GRAPHICS ---
        let scene, camera, renderer;
        let player, npcs = [], guideParticles = [];
        let raycaster = new THREE.Raycaster();
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false;
        let velocity = new THREE.Vector3();
        let prevTime = performance.now();
        let simActive = false;

        // Config
        const PLAYER_HEIGHT = 1.7;
        const SPEED = 6.0;
        
        // Tutorial & Objective State
        let tutorialStep = 0;
        let objectiveNPC = null; 

        function initSimPrompt() {
            startSimulation();
            document.getElementById('tutorial-modal').style.display = 'block';
            document.exitPointerLock(); 
        }

        function startTutorial() {
            document.getElementById('tutorial-modal').style.display = 'none';
            renderer.domElement.requestPointerLock();
            showNotification("Tutorial", "Step 1: Follow the trail of lights to Dr. Vance.", "warning");
            tutorialStep = 1;
            activeObjective = "Talk to Dr. Vance";
            updateObjectiveText();
        }

        function skipTutorial() {
            document.getElementById('tutorial-modal').style.display = 'none';
            renderer.domElement.requestPointerLock();
            showNotification("Started", "Explore the site. Talk to NPCs.", "success");
            tutorialStep = 99; // Skip
        }

        function updateObjectiveText() {
            document.getElementById('objective-text').innerText = activeObjective;
        }

        // --- SCENE GENERATION ---
        function createTerrain() {
            // Creates a hilly terrain with a flat spot for the trench
            const geometry = new THREE.PlaneGeometry(100, 100, 64, 64);
            const pos = geometry.attributes.position;
            
            for (let i = 0; i < pos.count; i++) {
                const x = pos.getX(i);
                const y = pos.getY(i);
                
                // Distance from center (Trench area)
                const dist = Math.sqrt(x*x + y*y);
                
                let zHeight = 0;
                // If outside the trench area (radius 6), add noise height
                if(dist > 8) {
                    zHeight = Math.sin(x * 0.1) * Math.cos(y * 0.1) * 1.5 + Math.random() * 0.2;
                    // Ramp up from the trench
                    zHeight += Math.min((dist - 8) * 0.5, 3); 
                } else if (dist > 5) {
                    // Slope
                     zHeight = (dist - 5) * 0.5;
                }

                pos.setZ(i, zHeight); // Z is Up in generated data, but we rotate plane later
            }
            
            geometry.computeVertexNormals();

            // Simple "Dirt" texture using canvas noise
            const canvas = document.createElement('canvas');
            canvas.width = 512; canvas.height = 512;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#5d4037';
            ctx.fillRect(0,0,512,512);
            // Add noise
            for(let i=0; i<10000; i++){
                ctx.fillStyle = Math.random() > 0.5 ? '#4e342e' : '#6d4c41';
                ctx.fillRect(Math.random()*512, Math.random()*512, 2, 2);
            }
            const tex = new THREE.CanvasTexture(canvas);
            tex.wrapS = THREE.RepeatWrapping;
            tex.wrapT = THREE.RepeatWrapping;
            tex.repeat.set(10, 10);

            const material = new THREE.MeshStandardMaterial({ map: tex, roughness: 0.9 });
            const ground = new THREE.Mesh(geometry, material);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);
            
            // Add decorative rocks/grass
            const rockGeo = new THREE.DodecahedronGeometry(0.4, 0);
            const rockMat = new THREE.MeshStandardMaterial({ color: 0x78716c });
            for(let i=0; i<40; i++) {
                const rock = new THREE.Mesh(rockGeo, rockMat);
                const rX = (Math.random() - 0.5) * 60;
                const rZ = (Math.random() - 0.5) * 60;
                if(Math.sqrt(rX*rX + rZ*rZ) < 9) continue; // Don't put in trench
                rock.position.set(rX, 1.5, rZ); // Approximate height placement
                rock.scale.setScalar(Math.random() * 1.5 + 0.5);
                scene.add(rock);
            }

            // Spoil Heap (Dirt Pile)
            const pileGeo = new THREE.ConeGeometry(3, 2.5, 16);
            const pile = new THREE.Mesh(pileGeo, material);
            pile.position.set(8, 1.25, 0);
            scene.add(pile);
        }

        function createTrench() {
            // The hole is actually created by the terrain going UP around 0,0.
            // We just need the trench floor and walls.
            
            const wallMat = new THREE.MeshStandardMaterial({ color: 0x4e342e }); // Darker dirt
            
            // Floor
            const floor = new THREE.Mesh(new THREE.PlaneGeometry(10, 10), new THREE.MeshStandardMaterial({ color: 0x3e2723 }));
            floor.rotation.x = -Math.PI / 2;
            floor.position.y = 0.05; // Just above implicit zero
            floor.receiveShadow = true;
            scene.add(floor);

            // Walls (Visual only, since terrain slopes up)
            // We put simple boxes to create sharp edges for the trench cut
            const wallGeo = new THREE.BoxGeometry(10, 1, 0.2);
            const backWall = new THREE.Mesh(wallGeo, wallMat);
            backWall.position.set(0, 0.5, -5.1);
            scene.add(backWall);
            
            const frontWall = new THREE.Mesh(wallGeo, wallMat);
            frontWall.position.set(0, 0.5, 5.1);
            scene.add(frontWall);
            
            const sideWallGeo = new THREE.BoxGeometry(0.2, 1, 10);
            const leftWall = new THREE.Mesh(sideWallGeo, wallMat);
            leftWall.position.set(-5.1, 0.5, 0);
            scene.add(leftWall);
            
            const rightWall = new THREE.Mesh(sideWallGeo, wallMat);
            rightWall.position.set(5.1, 0.5, 0);
            scene.add(rightWall);
        }

        function createCharacter(x, z, role, name, shirtColor, pantsColor, dialogue) {
            const group = new THREE.Group();
            group.position.set(x, 0, z);

            // Materials
            const skinMat = new THREE.MeshLambertMaterial({ color: 0xffdbac });
            const shirtMat = new THREE.MeshLambertMaterial({ color: shirtColor });
            const pantsMat = new THREE.MeshLambertMaterial({ color: pantsColor });
            
            // Legs
            const legGeo = new THREE.BoxGeometry(0.25, 0.8, 0.25);
            const legL = new THREE.Mesh(legGeo, pantsMat); legL.position.set(-0.15, 0.4, 0);
            const legR = new THREE.Mesh(legGeo, pantsMat); legR.position.set(0.15, 0.4, 0);
            group.add(legL, legR);

            // Torso
            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.7, 0.35), shirtMat);
            torso.position.y = 1.15;
            group.add(torso);

            // Head
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.35, 0.35, 0.35), skinMat);
            head.position.y = 1.7;
            group.add(head);

            // Arms
            const armGeo = new THREE.BoxGeometry(0.2, 0.7, 0.2);
            const armL = new THREE.Mesh(armGeo, shirtMat); 
            armL.position.set(-0.45, 1.15, 0);
            // Angle left arm slightly like holding something
            armL.rotation.z = 0.1;
            
            const armR = new THREE.Mesh(armGeo, shirtMat); 
            armR.position.set(0.45, 1.15, 0);
            armR.rotation.z = -0.1;

            group.add(armL, armR);

            // Role Indicator (Floating Text approximation - just a colored sphere for now)
            const indicator = new THREE.Mesh(new THREE.OctahedronGeometry(0.1), new THREE.MeshBasicMaterial({color: 0xffff00}));
            indicator.position.y = 2.2;
            // Animation handled in animate loop
            group.userData = { name, isNPC: true, dialogue, indicator, currentDialogueIndex: 0 };
            group.add(indicator);

            npcs.push(group);
            scene.add(group);
            return group;
        }

        function createGuideSystem() {
            // Create pool of particles for the trail
            const geo = new THREE.SphereGeometry(0.08, 4, 4);
            const mat = new THREE.MeshBasicMaterial({ color: 0x00ffff, transparent: true, opacity: 0.6 });
            for(let i=0; i<15; i++) {
                const mesh = new THREE.Mesh(geo, mat);
                mesh.visible = false;
                scene.add(mesh);
                guideParticles.push({ mesh, life: 0, offset: i * 0.1 });
            }
        }

        function initSim() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87ceeb);
            scene.fog = new THREE.Fog(0x87ceeb, 10, 60);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100);
            camera.position.set(0, PLAYER_HEIGHT, 10);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            
            const container = document.getElementById('sim-container');
            const oldCanvas = container.querySelector('canvas');
            if(oldCanvas) oldCanvas.remove();
            container.insertBefore(renderer.domElement, container.firstChild);

            // Lights
            const ambient = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambient);
            const sun = new THREE.DirectionalLight(0xffffff, 0.8);
            sun.position.set(20, 50, 20);
            sun.castShadow = true;
            sun.shadow.mapSize.width = 2048; sun.shadow.mapSize.height = 2048;
            scene.add(sun);

            createTerrain();
            createTrench();
            createGuideSystem();

            // Spawn NPCs
            // Dr. Vance (Supervisor) - Yellow Shirt
            const vance = createCharacter(4, 6, "Supervisor", "Dr. Vance", 0xffeb3b, 0x5d4037, [
                { text: "Ah, the new volunteer. Follow the trail to find your way around.", quizId: null },
                { text: "Ready to prove your worth? Let's test your tool knowledge.", quizId: "tools_101" }
            ]);
            
            // Dr. Jones (Arch) - Brown Jacket/Shirt
            const jones = createCharacter(-4, 4, "Archaeologist", "Dr. Jones", 0x8d6e63, 0x3e2723, [
                { text: "It belongs in a museum! But first, we must document it in situ.", quizId: null }
            ]);
            
            // Set initial objective
            objectiveNPC = vance;
            targetObject = vance;

            // Input Listeners
            document.addEventListener('keydown', e => {
                if(e.code === 'KeyW') moveForward = true;
                if(e.code === 'KeyS') moveBackward = true;
                if(e.code === 'KeyA') moveLeft = true;
                if(e.code === 'KeyD') moveRight = true;
                if(e.code === 'KeyE') checkInteraction();
            });
            document.addEventListener('keyup', e => {
                if(e.code === 'KeyW') moveForward = false;
                if(e.code === 'KeyS') moveBackward = false;
                if(e.code === 'KeyA') moveLeft = false;
                if(e.code === 'KeyD') moveRight = false;
            });
            document.addEventListener('mousemove', e => {
                if (document.pointerLockElement === renderer.domElement) {
                    camera.rotation.y -= e.movementX * 0.002;
                    camera.rotation.x -= e.movementY * 0.002;
                    camera.rotation.x = Math.max(-1.5, Math.min(1.5, camera.rotation.x));
                }
            });
            
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });
        }

        // --- LOOP & ANIMATION ---
        function animate() {
            if (!simActive) return;
            requestAnimationFrame(animate);
            
            const time = performance.now();
            const delta = (time - prevTime) / 1000;
            prevTime = time;

            // Movement
            if (document.pointerLockElement === renderer.domElement) {
                const speed = SPEED * delta;
                const direction = new THREE.Vector3();
                direction.z = Number(moveForward) - Number(moveBackward);
                direction.x = Number(moveRight) - Number(moveLeft);
                direction.normalize();

                if (moveForward || moveBackward) {
                    camera.translateZ(-direction.z * speed);
                }
                if (moveLeft || moveRight) {
                    camera.translateX(direction.x * speed);
                }
                camera.position.y = PLAYER_HEIGHT; // Simple ground clamping (ignore terrain height for simplicity of controls for now)
            }

            // Animate Guide Trail
            if (targetObject) {
                // Lerp source position relative to camera/player
                const startPos = new THREE.Vector3(camera.position.x, camera.position.y - 0.5, camera.position.z);
                const endPos = targetObject.position.clone().add(new THREE.Vector3(0, 1.5, 0));
                const dist = startPos.distanceTo(endPos);

                guideParticles.forEach((p, i) => {
                    p.life += delta * 0.5; // Speed of particles
                    if(p.life > 1) p.life = 0;
                    
                    // Lerp position
                    p.mesh.position.lerpVectors(startPos, endPos, p.life);
                    p.mesh.visible = true;
                    
                    // Scale based on life (fade in/out)
                    const s = Math.sin(p.life * Math.PI);
                    p.mesh.scale.setScalar(s);
                });
            }

            // NPC Indicators
            npcs.forEach(npc => {
                if(npc.userData.indicator) {
                    npc.userData.indicator.rotation.y += delta;
                    npc.userData.indicator.position.y = 2.2 + Math.sin(time * 0.005) * 0.1;
                }
            });
            
            // Raycast for Prompt
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const intersects = raycaster.intersectObjects(npcs, true);
            const prompt = document.getElementById('interaction-prompt');
            
            if (intersects.length > 0) {
                // Find parent group
                let obj = intersects[0].object;
                while(obj.parent && !obj.userData.isNPC) obj = obj.parent;
                
                if (obj.userData.isNPC && obj.position.distanceTo(camera.position) < 4) {
                    prompt.style.opacity = 1;
                } else {
                    prompt.style.opacity = 0;
                }
            } else {
                prompt.style.opacity = 0;
            }

            renderer.render(scene, camera);
        }

        function checkInteraction() {
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const intersects = raycaster.intersectObjects(npcs, true);
            if (intersects.length > 0) {
                let obj = intersects[0].object;
                while(obj.parent && !obj.userData.isNPC) obj = obj.parent;
                
                if (obj.userData.isNPC && obj.position.distanceTo(camera.position) < 4) {
                    openDialogue(obj);
                }
            }
        }

        // --- DIALOGUE SYSTEM ---
        function openDialogue(npc) {
            document.exitPointerLock();
            const box = document.getElementById('dialogue-box');
            const title = document.getElementById('npc-name');
            const text = document.getElementById('npc-text');
            const opts = document.getElementById('quiz-options');
            
            box.style.display = 'block';
            title.innerText = npc.userData.name;
            opts.innerHTML = '';
            
            const lines = npc.userData.dialogue;
            // Basic sequential dialogue logic
            const line = lines[0]; // Simply showing first line for demo
            text.innerText = line.text;
            
            if (line.quizId) {
                document.getElementById('dialogue-controls').style.display = 'none';
                startQuizInSim(line.quizId, opts);
            } else {
                document.getElementById('dialogue-controls').style.display = 'flex';
            }
        }

        function startQuizInSim(qid, container) {
            const mod = quizModules.find(m => m.id === qid);
            container.innerHTML = `<div class="font-bold text-amber-400 mb-2">Quiz: ${mod.title}</div>`;
            
            // Simple one-question demo for the structure
            const q = mod.questions[0];
            container.innerHTML += `<div class="mb-2">${q.q}</div>`;
            q.a.forEach((ans, i) => {
                container.innerHTML += `<button onclick="answerSim(${i === q.correct})" class="block w-full text-left p-2 bg-stone-700 hover:bg-stone-600 mb-1 rounded interactive-element">${ans}</button>`;
            });
            container.style.display = 'block';
        }
        
        // Exposed to global scope for HTML onclick access
        window.answerSim = function(isCorrect) {
            if(isCorrect) {
                showNotification("Correct!", "+50 XP", "success");
                document.getElementById('npc-text').innerText = "Excellent work. You passed.";
            } else {
                showNotification("Incorrect", "Try again.", "error");
            }
            document.getElementById('quiz-options').innerHTML = '';
            document.getElementById('dialogue-controls').style.display = 'flex';
        }

        function closeDialogue() {
            document.getElementById('dialogue-box').style.display = 'none';
            renderer.domElement.requestPointerLock();
        }
        function nextDialogue() { closeDialogue(); } // Simplified for demo

        function startSimulation() {
            if(!renderer) initSim();
            simActive = true;
            document.getElementById('sim-container').style.display = 'block';
            document.getElementById('website-container').style.display = 'none';
            prevTime = performance.now();
            animate();
        }

        function exitSimulation() {
            simActive = false;
            document.getElementById('sim-container').style.display = 'none';
            document.getElementById('website-container').style.display = 'flex';
            document.exitPointerLock();
        }

        window.onload = () => switchSection('home');
    </script>
</body>
</html>
