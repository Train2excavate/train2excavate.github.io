# Imakeprojects.github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>3D Pizza Factory Tycoon - Advanced</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Bungee+Inline&family=Poppins:wght@400;700&display=swap');

        :root {
            --felt-gray: #7f8c8d;
            --pizza-red: #e74c3c;
            --pizza-yellow: #f1c40f;
            --hud-bg: rgba(0, 0, 0, 0.8);
        }

        body {
            font-family: 'Poppins', sans-serif;
            margin: 0; padding: 0; overflow: hidden;
            background-color: #1a1a1a;
            user-select: none; -webkit-user-select: none;
            touch-action: none;
        }

        #game-container { position: relative; width: 100vw; height: 100vh; }
        #canvas-container { width: 100%; height: 100%; }
        #ui-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }

        /* HUD */
        .hud-stat-container {
            pointer-events: none; position: absolute; top: 10px; left: 10px;
            display: flex; flex-direction: column; gap: 8px; z-index: 10;
        }

        .hud-item {
            display: flex; align-items: center; padding: 5px 10px;
            background-color: var(--hud-bg); border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5); font-family: 'Bungee Inline', cursive;
            min-width: 150px;
        }

        .hud-label { font-size: 0.7rem; color: #ccc; margin-right: 8px; }
        .hud-value { font-size: 1.1rem; font-weight: bold; color: white; text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5); }

        /* Tutorial Overlay */
        #tutorial-overlay {
            pointer-events: auto;
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(30, 41, 59, 0.95); border: 4px solid #f1c40f;
            padding: 2rem; border-radius: 1rem; color: white; text-align: center;
            z-index: 100; max-width: 90%; width: 400px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.8);
            transition: opacity 0.3s;
        }
        #tutorial-overlay h2 { font-family: 'Bungee Inline', cursive; color: #f1c40f; font-size: 1.8rem; margin-bottom: 1rem; }
        .key-badge { background: #333; border: 1px solid #666; padding: 2px 6px; border-radius: 4px; font-weight: bold; color: #f1c40f; }
        .tut-btn { 
            margin-top: 1.5rem; background: #e74c3c; color: white; border: none; 
            padding: 10px 20px; font-weight: bold; font-family: 'Bungee Inline', cursive; 
            cursor: pointer; border-radius: 8px; transition: transform 0.1s;
        }
        .tut-btn:hover { background: #c0392b; }
        .tut-btn:active { transform: scale(0.95); }

        /* Rebirth Button (Top Right) */
        #rebirth-container {
            pointer-events: auto; position: absolute; top: 10px; right: 10px;
            z-index: 20; display: flex; flex-direction: column; align-items: flex-end;
        }
        #rebirth-btn-ui {
            background: linear-gradient(135deg, #8e44ad, #9b59b6);
            border: 3px solid #fff; border-radius: 12px;
            padding: 8px 16px; color: white; font-family: 'Bungee Inline', cursive;
            cursor: pointer; box-shadow: 0 4px 6px rgba(0,0,0,0.5);
            transition: transform 0.1s; text-align: center;
        }
        #rebirth-btn-ui:active { transform: scale(0.95); }
        .rebirth-cost { font-size: 0.8rem; display: block; color: #f1c40f; }

        /* Inventory Bar - Fixed 8 slots */
        #inventory-bar {
            position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%);
            display: flex; gap: 8px; background-color: rgba(0, 0, 0, 0.6);
            padding: 10px; border-radius: 12px; border: 2px solid rgba(255, 255, 255, 0.2);
            pointer-events: auto; z-index: 10; transition: all 0.3s;
        }

        .inv-slot {
            width: 50px; height: 50px; 
            background-color: rgba(255, 255, 255, 0.1); 
            border: 2px solid rgba(255, 255, 255, 0.3); 
            border-radius: 8px;
            display: flex; 
            flex-direction: column; align-items: center; justify-content: center;
            position: relative;
            transition: all 0.2s;
        }
        .inv-slot.active-slot {
             background-color: rgba(255, 255, 255, 0.25);
             border-color: #f1c40f;
             box-shadow: 0 0 5px #f1c40f;
        }

        .inv-icon { font-size: 24px; margin-bottom: 2px; opacity: 0.6; transition: opacity 0.2s; }
        .inv-slot.active-slot .inv-icon { opacity: 1.0; }

        .inv-count {
            position: absolute; bottom: 2px; right: 4px; font-weight: bold;
            color: #f1c40f; font-size: 12px; text-shadow: 1px 1px 0 #000;
            font-family: 'Bungee Inline', cursive; opacity: 0; transition: opacity 0.2s;
        }
        .inv-slot.active-slot .inv-count { opacity: 1.0; }


        /* Controls */
        #crosshair {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 8px; height: 8px; border: 2px solid white; border-radius: 50%;
            pointer-events: none; z-index: 10;
        }

        #mobile-controls {
            position: absolute; right: 20px; bottom: 20px; display: flex; flex-direction: column; gap: 15px;
            pointer-events: none; z-index: 15;
        }
        
        #mobile-right-controls {
            display: flex; flex-direction: column; gap: 15px; align-items: flex-end;
        }

        .action-button {
            pointer-events: auto; width: 70px; height: 70px; border-radius: 50%;
            color: white; font-weight: bold; font-family: 'Bungee Inline', cursive;
            box-shadow: 0 4px #000; transition: all 0.1s; display: flex; align-items: center; justify-content: center;
            font-size: 0.8rem; border: 2px solid rgba(255,255,255,0.3);
            user-select: none;
        }
        .action-button:active { transform: translateY(4px); box-shadow: 0 0 #000; }
        
        #view-toggle-button { background-color: #3498db; }
        #jump-button { background-color: #e74c3c; }
        #interact-btn-mobile { background-color: #f1c40f; color: #000; }

        /* Joystick */
        #joystick-zone {
            position: absolute; top: 0; left: 0; width: 50%; height: 100%;
            z-index: 14; 
        }
        .joystick-base {
            position: absolute; width: 100px; height: 100px; border-radius: 50%;
            background: rgba(255, 255, 255, 0.1); border: 2px solid rgba(255, 255, 255, 0.3);
            display: none; pointer-events: none; transform: translate(-50%, -50%);
        }
        .joystick-stick {
            position: absolute; width: 40px; height: 40px; border-radius: 50%;
            background: rgba(255, 255, 255, 0.8); top: 50%; left: 50%;
            transform: translate(-50%, -50%);
        }

        /* Interaction Prompt & Progress */
        #interaction-overlay {
            position: absolute; top: 55%; left: 50%; transform: translate(-50%, -50%);
            display: none; flex-direction: column; align-items: center; z-index: 100;
        }
        
        #progress-bar-container {
            width: 120px; height: 10px; background: rgba(0,0,0,0.5);
            border: 2px solid white; border-radius: 5px; margin-top: 10px;
            overflow: hidden; 
        }
        #progress-bar-fill {
            width: 0%; height: 100%; background: #f1c40f; transition: none;
        }

        .prompt-key-bg {
            width: 30px; height: 30px; background-color: #000; color: #fff;
            border: 2px solid #fff; border-radius: 5px; display: flex; align-items: center; justify-content: center;
            font-weight: bold; margin-bottom: 5px;
        }
        #interaction-label {
            background-color: rgba(0,0,0,0.8); color: white; padding: 4px 12px;
            border-radius: 4px; font-size: 0.9rem; font-weight: bold; white-space: nowrap;
        }

        #notification-area {
            position: absolute; top: 20%; left: 50%; transform: translate(-50%, -50%);
            pointer-events: none; text-align: center; width: 100%; z-index: 100;
        }
        .notif {
            font-size: 1.5rem; font-weight: bold; color: #f1c40f;
            text-shadow: 2px 2px 0 #000; opacity: 0; transition: opacity 0.5s;
        }

        #pc-hint {
            position: absolute; bottom: 10px; left: 10px; color: rgba(255,255,255,0.5); font-size: 0.8rem; pointer-events: none;
        }
        
        @media (min-width: 1024px) {
            #mobile-controls { display: none; }
            #joystick-zone { display: none; }
        }
    </style>
</head>
<body>

    <div id="loading-screen" class="fixed inset-0 bg-gray-900 z-50 flex flex-col justify-center items-center text-white">
        <div class="text-4xl font-bold mb-4 font-mono text-yellow-400">PIZZA TYCOON 3D</div>
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-yellow-400"></div>
        <p class="mt-4 text-gray-400">Connecting...</p>
    </div>

    <!-- Tutorial Overlay -->
    <div id="tutorial-overlay">
        <h2>HOW TO PLAY</h2>
        <div class="text-left space-y-2 text-sm text-gray-200">
            <p><span class="key-badge">W</span><span class="key-badge">A</span><span class="key-badge">S</span><span class="key-badge">D</span> to Move</p>
            <p><span class="key-badge">SHIFT</span> to Run, <span class="key-badge">SPACE</span> to Jump</p>
            <p><span class="key-badge">E</span> (Hold) to Interact with Stations</p>
            <p><span class="key-badge">1</span> <span class="key-badge">2</span> <span class="key-badge">3</span> to Equip Tools</p>
            <p><span class="key-badge">Q</span> (Shift+Q) to Lock/Unlock Mouse</p>
            <p><span class="key-badge">P</span> to Toggle this Tutorial</p>
            <hr class="border-gray-600 my-2">
            <p class="text-yellow-400">Goal: Make Pizzas! Dough -> Ingredients -> Prep -> Oven -> Sell.</p>
        </div>
        <button class="tut-btn" onclick="document.getElementById('tutorial-overlay').style.display='none'">GOT IT!</button>
    </div>

    <div id="game-container" class="hidden">
        <div id="canvas-container"></div>

        <div id="ui-overlay">
            <div id="crosshair"></div>

            <!-- Joystick (Mobile) -->
            <div id="joystick-zone">
                <div id="virtual-joystick" class="joystick-base">
                    <div class="joystick-stick"></div>
                </div>
            </div>

            <div id="interaction-overlay">
                <div class="prompt-key-bg" id="prompt-key">E</div>
                <div id="interaction-label">Interact</div>
                <div id="progress-bar-container">
                    <div id="progress-bar-fill"></div>
                </div>
            </div>

            <div id="notification-area">
                <div id="status-message" class="notif"></div>
            </div>

            <div id="rebirth-container">
                <button id="rebirth-btn-ui">
                    REBIRTH
                    <span class="rebirth-cost" id="ui-rebirth-cost">$100,000</span>
                </button>
            </div>

            <div class="hud-stat-container">
                <div class="hud-item bg-green-900 border-2 border-green-600">
                    <span class="hud-label">CASH:</span>
                    <span id="money-stat" class="hud-value text-green-300">$0.00</span>
                </div>
                <div class="hud-item bg-purple-900 border-2 border-purple-600">
                    <span class="hud-label">REBIRTHS:</span>
                    <span id="rebirth-stat" class="hud-value text-purple-300">0</span>
                </div>
                 <div class="hud-item bg-blue-900 border-2 border-blue-600">
                    <span class="hud-label">EQUIPMENT:</span>
                    <span id="equip-stat" class="hud-value text-blue-300">1</span>
                </div>
            </div>

            <!-- Inventory Slots -->
            <div id="inventory-bar">
                <div class="inv-slot" id="slot-dough">
                    <span class="inv-icon">🍚</span>
                    <span class="inv-count" id="inv-dough-count">0</span>
                </div>
                <div class="inv-slot" id="slot-ingred">
                    <span class="inv-icon">🍅</span>
                    <span class="inv-count" id="inv-ingred-count">0</span>
                </div>
                <div class="inv-slot" id="slot-raw">
                    <span class="inv-icon">🍕</span>
                    <span class="inv-count" id="inv-raw-count">0</span>
                </div>
                <div class="inv-slot" id="slot-cooked">
                    <span class="inv-icon">🔥</span>
                    <span class="inv-count" id="inv-cooked-count">0</span>
                </div>
                 <div class="inv-slot" id="slot-burger">
                    <span class="inv-icon">🍔</span>
                    <span class="inv-count" id="inv-burger-count">0</span>
                </div>
                 <!-- Empty Slots for Capacity Visualization -->
                <div class="inv-slot empty-slot">
                    <span class="inv-icon text-gray-400">_</span>
                    <span class="inv-count">0</span>
                </div>
                <div class="inv-slot empty-slot">
                    <span class="inv-icon text-gray-400">_</span>
                    <span class="inv-count">0</span>
                </div>
                <div class="inv-slot empty-slot">
                    <span class="inv-icon text-gray-400">_</span>
                    <span class="inv-count">0</span>
                </div>
            </div>

            <div id="mobile-controls">
                <button id="interact-btn-mobile" class="action-button">USE</button>
                <div id="mobile-right-controls">
                    <button id="view-toggle-button" class="action-button">VIEW</button>
                    <button id="jump-button" class="action-button">JUMP</button>
                </div>
            </div>
            
            <div id="pc-hint">
                SHIFT+Q to Toggle Mouse • WASD to Move • HOLD E to Interact • 1/2/3 Tools
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        setLogLevel('Debug');

        // --- FIREBASE CONFIG ---
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let app, db, auth, userId;
        let gameActive = false;
        let unsubscribeSave = null;

        // --- GAME STATE ---
        let gameState = {
            money: 0,
            rebirths: 0,
            // Inventory
            rawDough: 0,
            ingredients: 0,
            rawPizzas: 0,
            cookedPizzas: 0,
            burgers: 0,
            // Upgrades
            equipmentLevel: 1, 
            workerCount: 0,
            standLevel: 1,
            hasCashier: false,
            hasConveyor: false,
            hasFastFood: false,
            // Stats
            pizzaValue: 10.00,
        };
        
        let equippedTool = 0; // 0: None, 1: Rolling Pin, 2: Peel, 3: Drink

        const UPGRADE_COSTS = {
            EQUIPMENT: (level) => 200 * Math.pow(2, level - 1), 
            WORKER: (count) => 1500 + 500 * count,
            STAND: (level) => 3000 * Math.pow(1.5, level - 1),
            CASHIER: 5000, 
            CONVEYOR: 10000, 
            EXPAND_FOOD: 20000 
        };
        
        const INTERACTION_DURATIONS = {
            dough: 2.0,
            ingred: 2.0,
            prep_roll: 0.5,
            prep_top: 1.5,
            prep_grab: 0.5,
            oven: 2.0,
            sell: 1.0,
            fryer: 2.0,
            upgrade: 0.1,
        }

        // --- THREE.JS VARIABLES ---
        let scene, camera, renderer, clock;
        let playerGroup, cameraRig, heldItemGroup;
        let raycaster = new THREE.Raycaster();
        let interactables = [];
        let npcList = [];
        let customers = [];
        
        // Physics
        let velocity = new THREE.Vector3();
        let direction = new THREE.Vector3();
        let moveState = { fwd: false, bwd: false, left: false, right: false, jump: false, run: false };
        let isGrounded = true;
        let isThirdPerson = false;
        let canJump = false;
        
        // Interaction
        let currentLookTarget = null;
        let isInteracting = false;
        let interactionStartTime = 0;
        let currentInteractionDuration = 2.0; 
        let activeVisualItem = null; 
        
        // --- MATERIALS ---
        const materials = {
            dough: new THREE.MeshStandardMaterial({ color: 0xFCE4EC, roughness: 0.7 }),
            sauce: new THREE.MeshStandardMaterial({ color: 0xB71C1C, roughness: 0.8 }),
            cheese: new THREE.MeshStandardMaterial({ color: 0xFFD700 }),
            crust: new THREE.MeshStandardMaterial({ color: 0xD7CCC8 }),
            metal: new THREE.MeshStandardMaterial({ color: 0x95a5a6, metalness: 0.8, roughness: 0.2 }),
            wood: new THREE.MeshStandardMaterial({ color: 0x5D4037 }),
            brick: new THREE.MeshStandardMaterial({ color: 0x8B4513 }),
            brickDark: new THREE.MeshStandardMaterial({ color: 0x5B2C00 }),
            cooked: new THREE.MeshStandardMaterial({ color: 0x8D6E63 }),
            burgerBun: new THREE.MeshStandardMaterial({ color: 0xE67E22 }),
            burgerMeat: new THREE.MeshStandardMaterial({ color: 0x3E2723 }),
            skin: new THREE.MeshStandardMaterial({ color: 0xffccaa }),
            shirt: new THREE.MeshStandardMaterial({ color: 0x3498db }),
            toolHandle: new THREE.MeshStandardMaterial({ color: 0x8B4513 }), // Wood handle
            toolMetal: new THREE.MeshStandardMaterial({ color: 0xBDC3C7, metalness: 0.7 }),
            drinkRed: new THREE.MeshStandardMaterial({ color: 0xFF0000 }),
        };

        // --- INIT ---
        async function initGame() {
            app = initializeApp(firebaseConfig);
            auth = getAuth(app);
            db = getFirestore(app);

            const initAuth = async () => {
                if (initialAuthToken) await signInWithCustomToken(auth, initialAuthToken);
                else await signInAnonymously(auth);
            };
            await initAuth();

            onAuthStateChanged(auth, (user) => {
                if (user) {
                    userId = user.uid;
                    loadGame();
                    unsubscribeSave = setInterval(saveGame, 10000); 
                } else {
                    if (unsubscribeSave) clearInterval(unsubscribeSave);
                }
            });

            setupScene();
            setupControls();
            gameActive = true;
            document.getElementById('loading-screen').classList.add('hidden');
            document.getElementById('game-container').classList.remove('hidden');
            animate();
        }

        // --- SCENE SETUP ---
        function setupScene() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87CEEB); 
            scene.fog = new THREE.Fog(0x87CEEB, 10, 60);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            document.getElementById('canvas-container').appendChild(renderer.domElement);
            
            clock = new THREE.Clock();

            // Lighting
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);
            const dirLight = new THREE.DirectionalLight(0xffffff, 0.9);
            dirLight.position.set(15, 30, 20);
            dirLight.castShadow = true;
            dirLight.shadow.mapSize.width = 2048;
            dirLight.shadow.mapSize.height = 2048;
            dirLight.shadow.camera.top = 30; dirLight.shadow.camera.bottom = -30;
            dirLight.shadow.camera.left = -30; dirLight.shadow.camera.right = 30;
            scene.add(dirLight);

            // Floor
            const floorGeo = new THREE.PlaneGeometry(100, 100);
            const floorMat = new THREE.MeshStandardMaterial({ color: 0x34495e, roughness: 0.8 });
            const floor = new THREE.Mesh(floorGeo, floorMat);
            floor.rotation.x = -Math.PI / 2;
            floor.receiveShadow = true;
            scene.add(floor);

            // Player Setup
            playerGroup = new THREE.Group();
            scene.add(playerGroup);
            
            // Character Body
            const charGroup = new THREE.Group();
            const charBody = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.4, 1.4, 16), materials.shirt);
            charBody.position.y = 0.7;
            charGroup.add(charBody);
            const charHead = new THREE.Mesh(new THREE.SphereGeometry(0.35, 16, 16), materials.skin);
            charHead.position.y = 1.6;
            charGroup.add(charHead);
            
            charGroup.name = "CharacterMesh";
            charGroup.visible = false; // Hidden in 1st person initially
            charGroup.castShadow = true;
            playerGroup.add(charGroup);

            // Camera Rig
            cameraRig = new THREE.Object3D();
            playerGroup.add(cameraRig);
            cameraRig.position.y = 1.6; 
            cameraRig.add(camera);

            // Held Item Container
            heldItemGroup = new THREE.Group();
            heldItemGroup.position.set(0.5, -0.4, -0.8); 
            heldItemGroup.rotation.set(0.2, -0.2, 0);
            camera.add(heldItemGroup);

            // --- STATIONS ---
            createDetailedStation('Dough Mixer', -10, 0, 5, materials.metal, 'dough', 'mixer');
            createDetailedStation('Ingredients', 10, 0, 5, materials.wood, 'ingred', 'crates');
            
            const prepStation = createDetailedStation('Prep Table', 0, 0, 5, materials.metal, 'prep', 'table');
            prepStation.userData.prepState = 'empty'; 
            prepStation.userData.currentDuration = INTERACTION_DURATIONS.prep_roll;

            createDetailedStation('Oven', 0, 0, 10, materials.brick, 'oven', 'oven');
            createDetailedStation('Sell Stand', 0, 0, -8, materials.wood, 'sell', 'stand');
            
            const fryerStation = createDetailedStation('Fryer', 12, 0, 0, materials.metal, 'fryer', 'fryer');
            fryerStation.visible = false;
            
            // --- UPGRADES ---
            createUpgradePad('Equipment', -15, 5, UPGRADE_COSTS.EQUIPMENT(gameState.equipmentLevel), 'equip', gameState.equipmentLevel);
            createUpgradePad('Hire Worker', -15, 0, UPGRADE_COSTS.WORKER(gameState.workerCount), 'worker', gameState.workerCount);
            createUpgradePad('Marketing', -15, -5, UPGRADE_COSTS.STAND(gameState.standLevel), 'stand', gameState.standLevel);

            createUpgradePad('Cashier (Auto-Sell)', 15, 5, UPGRADE_COSTS.CASHIER, 'cashier', gameState.hasCashier ? 1 : 0);
            createUpgradePad('Conveyor (Auto-Transport)', 15, 0, UPGRADE_COSTS.CONVEYOR, 'conveyor', gameState.hasConveyor ? 1 : 0);
            createUpgradePad('Expand Food (Burgers)', 15, -5, UPGRADE_COSTS.EXPAND_FOOD, 'expand', gameState.hasFastFood ? 1 : 0);

            // Decor
            const wallGeo = new THREE.BoxGeometry(60, 6, 1);
            const wallMat = new THREE.MeshStandardMaterial({color: 0x444444});
            const backWall = new THREE.Mesh(wallGeo, wallMat);
            backWall.position.set(0, 3, 18);
            scene.add(backWall);

            spawnCustomer(-2, 0, -10);
            spawnCustomer(2, 0, -10);
        }

        function createDetailedStation(label, x, y, z, mat, type, modelType) {
            const group = new THREE.Group();
            group.position.set(x, 0, z);
            group.receiveShadow = true;
            group.castShadow = true;
            
            let mainMesh;

            if (modelType === 'mixer') {
                mainMesh = new THREE.Mesh(new THREE.SphereGeometry(1.2, 16, 16), materials.dough);
                mainMesh.position.y = 1.2;
                mainMesh.name = "DoughBlob";
                group.add(mainMesh);
                const stand = new THREE.Mesh(new THREE.CylinderGeometry(1.5, 1.5, 0.5, 32), mat);
                stand.position.y = 0.25;
                group.add(stand);
            } 
            else if (modelType === 'crates') {
                const table = new THREE.Mesh(new THREE.BoxGeometry(3.5, 1, 1.8), materials.wood);
                table.position.y = 0.5;
                group.add(table);
                const sauceBowl = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.4, 0.3), materials.metal);
                sauceBowl.position.set(-1.0, 1.25, 0.5);
                group.add(sauceBowl);
                const sauceFill = new THREE.Mesh(new THREE.CylinderGeometry(0.45, 0.35, 0.1), materials.sauce);
                sauceFill.position.set(-1.0, 1.35, 0.5);
                group.add(sauceFill);
                const cheeseBlock = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.4, 0.4), materials.cheese);
                cheeseBlock.position.set(0, 1.3, 0.5);
                group.add(cheeseBlock);
                const toppingsJar = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 0.8), new THREE.MeshStandardMaterial({color: 0xcccccc, transparent: true, opacity: 0.5}));
                toppingsJar.position.set(1, 1.4, 0.5);
                group.add(toppingsJar);
            } 
            else if (modelType === 'oven') {
                const base = new THREE.Mesh(new THREE.BoxGeometry(4, 1.5, 3), materials.brick);
                base.position.y = 0.75;
                group.add(base);
                const holeGeo = new THREE.CylinderGeometry(1, 1, 3.1, 16, 1, false, 0, Math.PI);
                const hole = new THREE.Mesh(holeGeo, materials.brickDark);
                hole.rotation.x = Math.PI/2;
                hole.rotation.y = Math.PI/2;
                hole.position.set(0, 1.2, 0);
                group.add(hole);
                const fire = new THREE.PointLight(0xff7700, 2, 5);
                fire.position.set(0, 1.2, 0.5);
                group.add(fire);
            }
            else if (modelType === 'table' || modelType === 'fryer') {
                const tableGeo = new THREE.BoxGeometry(3, 0.2, 2);
                mainMesh = new THREE.Mesh(tableGeo, mat);
                mainMesh.position.y = 1.1;
                group.add(mainMesh);
                
                if(modelType === 'table') {
                    const prepSurface = new THREE.Mesh(new THREE.PlaneGeometry(1.5, 1.5), new THREE.MeshStandardMaterial({ color: 0x95a5a6, side: THREE.DoubleSide }));
                    prepSurface.rotation.x = -Math.PI / 2;
                    prepSurface.position.y = 1.21;
                    prepSurface.name = "PrepSurfaceMesh";
                    prepSurface.visible = false;
                    group.add(prepSurface);
                }

                const legGeo = new THREE.BoxGeometry(0.2, 1, 0.2);
                const legPositions = [[-1.3, -0.8], [1.3, -0.8], [-1.3, 0.8], [1.3, 0.8]];
                legPositions.forEach(pos => {
                    const l = new THREE.Mesh(legGeo, materials.wood);
                    l.position.set(pos[0], 0.5, pos[1]);
                    group.add(l);
                });
            }
            else {
                const base = new THREE.Mesh(new THREE.BoxGeometry(4, 1.2, 1.5), materials.wood);
                base.position.y = 0.6;
                group.add(base);
                const poleGeo = new THREE.CylinderGeometry(0.05, 0.05, 2.5);
                const p1 = new THREE.Mesh(poleGeo, materials.metal); p1.position.set(-1.8, 1.8, 0.6); group.add(p1);
                const p2 = new THREE.Mesh(poleGeo, materials.metal); p2.position.set(1.8, 1.8, 0.6); group.add(p2);
                const canopy = new THREE.Mesh(new THREE.BoxGeometry(4.2, 0.1, 2), new THREE.MeshStandardMaterial({color: 0xe74c3c}));
                canopy.position.set(0, 3, 0.4);
                canopy.rotation.x = 0.1;
                group.add(canopy);
            }

            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = 256; canvas.height = 64;
            ctx.fillStyle = 'rgba(0,0,0,0.8)'; ctx.fillRect(0,0,256,64);
            ctx.fillStyle = 'white'; ctx.font = 'bold 30px Arial'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(label, 128, 32);
            
            const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: new THREE.CanvasTexture(canvas) }));
            sprite.scale.set(3, 0.75, 1);
            sprite.position.y = 3.5;
            group.add(sprite);

            group.userData = { isInteractable: true, type: type, name: label, modelType: modelType };
            scene.add(group);
            interactables.push(group);
            
            const itemMount = new THREE.Group();
            itemMount.position.y = (modelType === 'mixer') ? 2.5 : 1.3;
            itemMount.name = "ItemMount";
            group.add(itemMount);
            
            return group;
        }

        function createUpgradePad(label, x, z, cost, type, level) {
            const group = new THREE.Group();
            group.position.set(x, 0.05, z);
            
            const padGeo = new THREE.CircleGeometry(1.5, 32);
            const pad = new THREE.Mesh(padGeo, new THREE.MeshStandardMaterial({ color: 0x2ecc71, transparent: true, opacity: 0.6 }));
            pad.rotation.x = -Math.PI/2;
            group.add(pad);
            
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = 256; canvas.height = 128;
            ctx.fillStyle = 'white'; ctx.font = 'bold 30px Arial'; ctx.textAlign = 'center';
            ctx.fillText(label, 128, 40);
            ctx.fillStyle = '#f1c40f';
            const costText = (type === 'cashier' || type === 'conveyor' || type === 'expand') ? `$${cost.toLocaleString()}` : `Lvl ${level + 1} | $${cost.toLocaleString()}`;
            ctx.fillText(costText, 128, 90);
            
            const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: new THREE.CanvasTexture(canvas) }));
            sprite.scale.set(3, 1.5, 1);
            sprite.position.y = 1.5;
            group.add(sprite);
            
            group.userData = { isInteractable: true, type: 'upgrade', upgradeType: type, cost: cost, level: level };
            interactables.push(group);
            scene.add(group);
        }

        function spawnCustomer(x, y, z) {
            const group = new THREE.Group();
            group.position.set(x, y, z);
            const body = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.8, 0.3), new THREE.MeshStandardMaterial({color: Math.random() * 0xffffff}));
            body.position.y = 0.9;
            group.add(body);
            const head = new THREE.Mesh(new THREE.SphereGeometry(0.25), materials.skin);
            head.position.y = 1.45;
            group.add(head);
            scene.add(group);
            customers.push(group);
        }

        // --- HELD ITEM VISUALS ---
        function updateHeldItemVisual() {
            while(heldItemGroup.children.length > 0) heldItemGroup.remove(heldItemGroup.children[0]); 

            let mesh = null;
            // 1. Check if holding actual Factory Resources (Priority)
            let holdingResource = true;
            if(gameState.burgers > 0) {
                mesh = new THREE.Group();
                const b1 = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 0.05), materials.burgerBun);
                const m = new THREE.Mesh(new THREE.CylinderGeometry(0.21, 0.21, 0.05), materials.burgerMeat); m.position.y = 0.06;
                const b2 = new THREE.Mesh(new THREE.SphereGeometry(0.2), materials.burgerBun); b2.position.y = 0.09; b2.scale.y = 0.5;
                mesh.add(b1, m, b2);
            } else if (gameState.cookedPizzas > 0) {
                mesh = new THREE.Group();
                const box = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.05, 0.6), materials.wood);
                const pie = new THREE.Mesh(new THREE.CylinderGeometry(0.25, 0.25, 0.02), materials.cooked); pie.position.y = 0.04;
                mesh.add(box, pie);
            } else if (gameState.rawPizzas > 0) {
                mesh = new THREE.Group();
                const c = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 0.02), materials.dough);
                const s = new THREE.Mesh(new THREE.CylinderGeometry(0.25, 0.25, 0.03), materials.sauce);
                mesh.add(c, s);
            } else if (gameState.ingredients > 0) {
                mesh = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.3, 0.4), materials.wood);
            } else if (gameState.rawDough > 0) {
                mesh = new THREE.Mesh(new THREE.SphereGeometry(0.2), materials.dough);
            } else {
                holdingResource = false;
            }

            // 2. If no resource, show equipped Tool (Cosmetic)
            if (!holdingResource && equippedTool > 0) {
                if (equippedTool === 1) { // Rolling Pin
                    mesh = new THREE.Group();
                    const pin = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 0.6), materials.toolHandle);
                    pin.rotation.z = Math.PI / 2;
                    mesh.add(pin);
                } else if (equippedTool === 2) { // Peel
                    mesh = new THREE.Group();
                    const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.03, 0.8), materials.toolHandle);
                    handle.position.z = 0.4;
                    handle.rotation.x = Math.PI / 2;
                    const paddle = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.02, 0.4), materials.toolMetal);
                    mesh.add(handle, paddle);
                } else if (equippedTool === 3) { // Drink
                    mesh = new THREE.Group();
                    const cup = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.08, 0.25), materials.drinkRed);
                    const straw = new THREE.Mesh(new THREE.CylinderGeometry(0.01, 0.01, 0.3), new THREE.MeshStandardMaterial({color:0xffffff}));
                    straw.position.set(0.05, 0.1, 0);
                    straw.rotation.z = -0.2;
                    mesh.add(cup, straw);
                }
            }

            if(mesh) heldItemGroup.add(mesh);
        }

        // --- CONTROLS ---
        function setupControls() {
            document.addEventListener('keydown', (e) => {
                switch(e.code) {
                    case 'KeyW': moveState.fwd = true; break;
                    case 'KeyS': moveState.bwd = true; break;
                    case 'KeyA': moveState.left = true; break;
                    case 'KeyD': moveState.right = true; break;
                    case 'Space': if(canJump) moveState.jump = true; break;
                    case 'ShiftLeft': moveState.run = true; break;
                    case 'KeyE': startInteraction(); break;
                    case 'KeyV': toggleView(); break;
                    case 'KeyP': 
                        const tut = document.getElementById('tutorial-overlay');
                        tut.style.display = tut.style.display === 'none' ? 'block' : 'none';
                        break;
                    case 'Digit1': equippedTool = 1; updateHeldItemVisual(); notify("Equipped: Rolling Pin", false); break;
                    case 'Digit2': equippedTool = 2; updateHeldItemVisual(); notify("Equipped: Peel", false); break;
                    case 'Digit3': equippedTool = 3; updateHeldItemVisual(); notify("Equipped: Drink", false); break;
                    case 'KeyQ': 
                        if(e.shiftKey) {
                            if(document.pointerLockElement) document.exitPointerLock();
                            else document.body.requestPointerLock();
                        }
                        break;
                }
            });
            document.addEventListener('keyup', (e) => {
                switch(e.code) {
                    case 'KeyW': moveState.fwd = false; break;
                    case 'KeyS': moveState.bwd = false; break;
                    case 'KeyA': moveState.left = false; break;
                    case 'KeyD': moveState.right = false; break;
                    case 'Space': moveState.jump = false; break;
                    case 'ShiftLeft': moveState.run = false; break;
                    case 'KeyE': endInteraction(); break;
                }
            });

            document.addEventListener('mousemove', (e) => {
                if (document.pointerLockElement) {
                    playerGroup.rotation.y -= e.movementX * 0.002;
                    // Standard: Moving mouse UP (negative Y) should rotate camera UP (negative X rotation)
                    // Note: In ThreeJS, rotation around X: Positive is looking DOWN, Negative is looking UP.
                    cameraRig.rotation.x -= e.movementY * 0.002;
                    cameraRig.rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, cameraRig.rotation.x));
                }
            });

            const interactBtn = document.getElementById('interact-btn-mobile');
            interactBtn.addEventListener('touchstart', (e) => { e.preventDefault(); startInteraction(); }, {passive: false});
            interactBtn.addEventListener('touchend', (e) => { e.preventDefault(); endInteraction(); });

            document.getElementById('jump-button').addEventListener('touchstart', (e) => { e.preventDefault(); moveState.jump = true; canJump = true; });
            document.getElementById('jump-button').addEventListener('touchend', (e) => { e.preventDefault(); moveState.jump = false; canJump = false; });
            document.getElementById('view-toggle-button').addEventListener('click', toggleView);
            document.getElementById('rebirth-btn-ui').addEventListener('click', doRebirthAttempt);

            // Joystick Logic
            const joystickZone = document.getElementById('joystick-zone');
            const stickBase = document.getElementById('virtual-joystick');
            const stickKnob = stickBase.querySelector('.joystick-stick');
            let jOriginX, jOriginY;

            joystickZone.addEventListener('touchstart', (e) => {
                e.preventDefault();
                const touch = e.touches[0];
                jOriginX = touch.clientX;
                jOriginY = touch.clientY;
                stickBase.style.display = 'block';
                stickBase.style.left = jOriginX + 'px';
                stickBase.style.top = jOriginY + 'px';
                stickKnob.style.transform = `translate(-50%, -50%)`;
            }, {passive: false});

            joystickZone.addEventListener('touchmove', (e) => {
                e.preventDefault();
                const touch = e.touches[0];
                const deltaX = touch.clientX - jOriginX;
                const deltaY = touch.clientY - jOriginY;
                const distance = Math.min(50, Math.sqrt(deltaX*deltaX + deltaY*deltaY));
                const angle = Math.atan2(deltaY, deltaX);
                const moveX = Math.cos(angle) * distance;
                const moveY = Math.sin(angle) * distance;
                stickKnob.style.transform = `translate(calc(-50% + ${moveX}px), calc(-50% + ${moveY}px))`;
                moveState.fwd = deltaY < -10;
                moveState.bwd = deltaY > 10;
                moveState.left = deltaX < -10;
                moveState.right = deltaX > 10;
            }, {passive: false});

            joystickZone.addEventListener('touchend', (e) => {
                e.preventDefault();
                stickBase.style.display = 'none';
                moveState.fwd = false; moveState.bwd = false;
                moveState.left = false; moveState.right = false;
            });
        }

        function toggleView() {
            isThirdPerson = !isThirdPerson;
            const charMesh = playerGroup.getObjectByName("CharacterMesh");
            if(charMesh) charMesh.visible = isThirdPerson;
            heldItemGroup.visible = !isThirdPerson;
            // When switching to 1st person, reset camera rig to center
            if(!isThirdPerson) {
                camera.position.set(0, 0, 0);
                camera.lookAt(playerGroup.position.clone().add(new THREE.Vector3(0, 0, -10))); // Look forward
            }
            document.getElementById('view-toggle-button').textContent = isThirdPerson ? "1st VIEW" : "3rd VIEW";
        }

        // ... [Interaction Logic same as before] ...
        function updatePrepVisuals(target) {
            const surface = target.getObjectByName("PrepSurfaceMesh");
            if (!surface) return;
            while(surface.children.length > 0) surface.remove(surface.children[0]);
            
            surface.visible = true;
            let mesh = null;
            if (target.userData.prepState === 'rolled') {
                mesh = new THREE.Mesh(new THREE.CylinderGeometry(0.8, 0.8, 0.05, 32), materials.dough);
                mesh.rotation.x = Math.PI / 2;
                mesh.position.y = 0.03;
            } else if (target.userData.prepState === 'topped') {
                mesh = new THREE.Group();
                const base = new THREE.Mesh(new THREE.CylinderGeometry(0.8, 0.8, 0.05, 32), materials.crust);
                const sauce = new THREE.Mesh(new THREE.CylinderGeometry(0.75, 0.75, 0.03, 32), materials.sauce); sauce.position.y = 0.04;
                const cheese = new THREE.Mesh(new THREE.CylinderGeometry(0.7, 0.7, 0.02, 32), materials.cheese); cheese.position.y = 0.055;
                mesh.add(base, sauce, cheese);
                mesh.rotation.x = Math.PI / 2;
                mesh.position.y = 0.03;
            } else { surface.visible = false; }
            if (mesh) surface.add(mesh);
        }

        function determineInteractionDuration(target) {
            const type = target.userData.type;
            const prepState = target.userData.prepState;
            if (type === 'prep') {
                if (prepState === 'empty' && gameState.rawDough > 0) return INTERACTION_DURATIONS.prep_roll;
                if (prepState === 'rolled' && gameState.ingredients > 0) return INTERACTION_DURATIONS.prep_top;
                if (prepState === 'topped') return INTERACTION_DURATIONS.prep_grab;
            }
            if (type === 'upgrade') return INTERACTION_DURATIONS.upgrade; 
            return INTERACTION_DURATIONS[type] || 2.0;
        }

        function startInteraction() {
            if(!currentLookTarget || isInteracting) return;
            currentInteractionDuration = determineInteractionDuration(currentLookTarget);
            isInteracting = true;
            interactionStartTime = clock.getElapsedTime();
            
            const mount = currentLookTarget.getObjectByName("ItemMount");
            const type = currentLookTarget.userData.type;
            if(mount) {
                while(mount.children.length > 0) mount.remove(mount.children[0]);
                let tempMesh;
                if(type === 'dough') tempMesh = new THREE.Mesh(new THREE.SphereGeometry(0.3), materials.dough);
                if(type === 'ingred') tempMesh = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.3, 0.4), materials.wood);
                if(type === 'prep') {
                    if (currentLookTarget.userData.prepState === 'empty') tempMesh = new THREE.Mesh(new THREE.SphereGeometry(0.3), materials.dough);
                    else if (currentLookTarget.userData.prepState === 'rolled') tempMesh = new THREE.Mesh(new THREE.SphereGeometry(0.3), materials.cheese);
                    else if (currentLookTarget.userData.prepState === 'topped') tempMesh = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.05, 0.6), materials.wood);
                }
                if(type === 'oven' && gameState.rawPizzas > 0) {
                    tempMesh = new THREE.Group();
                    const c = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 0.02), materials.dough);
                    const s = new THREE.Mesh(new THREE.CylinderGeometry(0.25, 0.25, 0.03), materials.sauce);
                    tempMesh.add(c, s);
                }
                if(tempMesh) {
                    mount.add(tempMesh);
                    activeVisualItem = tempMesh;
                    activeVisualItem.scale.setScalar(0.01);
                }
            }
            document.getElementById('progress-bar-container').style.display = 'block';
        }

        function endInteraction() {
            if(!isInteracting) return;
            isInteracting = false;
            document.getElementById('progress-bar-container').style.display = 'none';
            document.getElementById('progress-bar-fill').style.width = '0%';
            if(activeVisualItem && activeVisualItem.parent && (clock.getElapsedTime() - interactionStartTime) < currentInteractionDuration - 0.01) {
                activeVisualItem.parent.remove(activeVisualItem);
                activeVisualItem = null;
            }
        }

        function completeInteraction() {
            endInteraction(); 
            if(!currentLookTarget) return;
            const type = currentLookTarget.userData.type;
            const target = currentLookTarget; 
            const mount = target.getObjectByName("ItemMount");
            if(mount) { while(mount.children.length > 0) mount.remove(mount.children[0]); activeVisualItem = null; }

             if (type === 'dough') {
                if (gameState.rawDough < 5) {
                    gameState.rawDough++;
                    const doughBlob = target.getObjectByName("DoughBlob");
                    if (doughBlob) { doughBlob.scale.set(0.9, 0.9, 0.9); setTimeout(() => doughBlob.scale.set(1.0, 1.0, 1.0), 100); }
                    notify(`Ripped off Dough! (${gameState.rawDough})`);
                } else notify("Inventory Full!", true);
            }
            else if (type === 'ingred') {
                if (gameState.ingredients < 5) {
                    gameState.ingredients++;
                    notify(`Grabbed Toppings! (${gameState.ingredients})`);
                } else notify("Inventory Full!", true);
            }
            else if (type === 'prep') {
                let prepState = target.userData.prepState;
                if (prepState === 'empty' && gameState.rawDough > 0) { gameState.rawDough--; target.userData.prepState = 'rolled'; notify("Dough Rolled!"); }
                else if (prepState === 'rolled' && gameState.ingredients > 0) { gameState.ingredients--; target.userData.prepState = 'topped'; notify("Pizza Topped!"); }
                else if (prepState === 'topped') {
                    if (gameState.rawPizzas < 5) { gameState.rawPizzas++; target.userData.prepState = 'empty'; notify("Raw Pizza Ready!"); } 
                    else notify("Inventory Full!", true);
                } else notify("Missing Items!", true);
                updatePrepVisuals(target);
            }
            else if (type === 'oven') {
                if (gameState.rawPizzas > 0) { gameState.rawPizzas--; gameState.cookedPizzas++; notify("Pizza Cooked!"); } else notify("Need Raw Pizza!", true);
            }
            else if (type === 'sell') {
                let earnings = 0;
                const multiplier = (1 + gameState.rebirths * 0.5);
                const valueBoost = gameState.standLevel;
                if (gameState.cookedPizzas > 0) { earnings += gameState.cookedPizzas * gameState.pizzaValue * valueBoost * multiplier; gameState.cookedPizzas = 0; }
                if (gameState.burgers > 0) { earnings += gameState.burgers * (gameState.pizzaValue * 1.5) * valueBoost * multiplier; gameState.burgers = 0; }
                if (earnings > 0) { gameState.money += earnings; notify(`Sold for $${earnings.toFixed(2)}!`); } else notify("Nothing to sell!", true);
            }
            else if (type === 'fryer') { if(!gameState.hasFastFood) return; gameState.burgers++; notify("Fried a Burger!"); }
            
            else if (type === 'upgrade') {
                const cost = target.userData.cost;
                const uType = target.userData.upgradeType;
                if (gameState.money >= cost) {
                    gameState.money -= cost;
                    let newLevel = target.userData.level + 1;
                    if(uType === 'equip') { gameState.equipmentLevel = newLevel; target.userData.cost = UPGRADE_COSTS.EQUIPMENT(newLevel); notify(`Equipment Level ${newLevel}!`); }
                    else if(uType === 'worker') {
                        if (gameState.equipmentLevel > target.userData.level) { gameState.workerCount = newLevel; spawnWorker(newLevel); target.userData.cost = UPGRADE_COSTS.WORKER(newLevel); notify(`Hired Worker #${newLevel}!`); }
                        else { notify("Upgrade Equipment First!", true); gameState.money += cost; }
                    }
                    else if(uType === 'stand') { gameState.standLevel = newLevel; target.userData.cost = UPGRADE_COSTS.STAND(newLevel); notify(`Marketing Level ${newLevel}!`); }
                    else if(uType === 'cashier') { gameState.hasCashier = true; notify("Cashier Hired (Auto-Sell)!"); target.visible = false; }
                    else if(uType === 'conveyor') { gameState.hasConveyor = true; notify("Conveyor Belt Installed!"); target.visible = false; }
                    else if(uType === 'expand') { gameState.hasFastFood = true; interactables.forEach(obj => { if(obj.userData.type === 'fryer') obj.visible = true; }); notify("FAST FOOD UNLOCKED!"); target.visible = false; }
                    target.userData.level = newLevel;
                } else notify(`Need $${cost}!`, true);
            }
            updateUI();
            updateHeldItemVisual();
            saveGame();
        }

        // ... [Rebirth/Worker Logic/Animate same as before with fix] ...
        function doRebirthAttempt() {
            const cost = 100000 * (gameState.rebirths + 1);
            if(gameState.money >= cost) {
                gameState.rebirths++;
                gameState.money = gameState.rebirths * 10000;
                gameState.rawDough = 0; gameState.ingredients = 0; gameState.rawPizzas = 0; gameState.cookedPizzas = 0; gameState.burgers = 0;
                gameState.equipmentLevel = 1; gameState.workerCount = 0; gameState.standLevel = 1;
                gameState.hasCashier = false; gameState.hasConveyor = false; gameState.hasFastFood = false;
                npcList.forEach(npc => scene.remove(npc.mesh)); npcList = [];
                interactables.forEach(obj => {
                    if(obj.userData.type === 'fryer') obj.visible = false;
                    if(obj.userData.type === 'upgrade') {
                        obj.visible = true;
                        const uType = obj.userData.upgradeType;
                        let level = (uType === 'cashier' || uType === 'conveyor' || uType === 'expand') ? 0 : 1;
                        obj.userData.level = level;
                        if (uType === 'equip') obj.userData.cost = UPGRADE_COSTS.EQUIPMENT(1);
                        else if (uType === 'worker') obj.userData.cost = UPGRADE_COSTS.WORKER(0);
                        else if (uType === 'stand') obj.userData.cost = UPGRADE_COSTS.STAND(1);
                        else if (uType === 'cashier') obj.userData.cost = UPGRADE_COSTS.CASHIER;
                        else if (uType === 'conveyor') obj.userData.cost = UPGRADE_COSTS.CONVEYOR;
                        else if (uType === 'expand') obj.userData.cost = UPGRADE_COSTS.EXPAND_FOOD;
                    }
                    if(obj.userData.type === 'prep') { obj.userData.prepState = 'empty'; updatePrepVisuals(obj); }
                });
                notify(`REBIRTH #${gameState.rebirths}! Earn Multiplier Active!`, false);
                updateUI(); updateHeldItemVisual(); saveGame();
            } else notify(`Need $${cost.toLocaleString()} to Rebirth!`, true);
        }

        function notify(msg, isErr) {
            const el = document.getElementById('status-message');
            el.innerText = msg;
            el.style.color = isErr ? '#e74c3c' : '#f1c40f';
            el.style.opacity = 1;
            setTimeout(() => el.style.opacity = 0, 2000);
        }

        function runWorkerLogic(delta) {
            if (gameState.workerCount > 0) {
                const productionRate = gameState.equipmentLevel * gameState.workerCount;
                if (Math.random() < productionRate * delta * 0.1) {
                    if (Math.random() < 0.5) { if (gameState.rawDough < 10 + gameState.equipmentLevel * 5) gameState.rawDough++; } 
                    else { if (gameState.ingredients < 10 + gameState.equipmentLevel * 5) gameState.ingredients++; }
                    updateUI();
                }
            }
            if (gameState.hasConveyor) {
                if (Math.random() < 0.5 * delta) {
                    if(gameState.rawPizzas > 0 && gameState.cookedPizzas < 10) { gameState.rawPizzas--; gameState.cookedPizzas++; notify("Conveyor Cooked Pizza", false); }
                }
            }
            if (gameState.hasCashier) {
                if (Math.random() < 0.2 * delta && (gameState.cookedPizzas > 0 || gameState.burgers > 0)) {
                    let earnings = 0;
                    const multiplier = (1 + gameState.rebirths * 0.5);
                    const valueBoost = gameState.standLevel;
                    const sellAmount = 1;
                    if (gameState.cookedPizzas >= sellAmount) { earnings += sellAmount * gameState.pizzaValue * valueBoost * multiplier; gameState.cookedPizzas -= sellAmount; } 
                    else if (gameState.burgers >= sellAmount) { earnings += sellAmount * (gameState.pizzaValue * 1.5) * valueBoost * multiplier; gameState.burgers -= sellAmount; }
                    if (earnings > 0) { gameState.money += earnings; notify(`Cashier Sold $${earnings.toFixed(2)}`, false); }
                    updateUI();
                }
            }
        }

        function animate() {
            requestAnimationFrame(animate);
            const delta = clock.getDelta();

            // FIXED 3RD PERSON MOVEMENT (Camera Relative)
            const speed = moveState.run ? 15 : 8;
            
            // Get camera direction (ignoring Y)
            const camDir = new THREE.Vector3();
            camera.getWorldDirection(camDir);
            camDir.y = 0; camDir.normalize();
            
            // Get camera side direction
            const camSide = new THREE.Vector3();
            camSide.crossVectors(camera.up, camDir);
            camSide.normalize();

            // Calculate movement vector based on WASD
            const moveVec = new THREE.Vector3(0, 0, 0);
            if (moveState.fwd) moveVec.add(camDir); // W = Forward in Camera View
            if (moveState.bwd) moveVec.add(camDir.clone().negate()); // S = Back
            if (moveState.left) moveVec.add(camSide); // A = Left
            if (moveState.right) moveVec.add(camSide.clone().negate()); // D = Right
            
            if (moveVec.lengthSq() > 0) moveVec.normalize();

            velocity.x = moveVec.x * speed * delta;
            velocity.z = moveVec.z * speed * delta;

            if (!isGrounded) velocity.y -= 20 * delta;
            else {
                velocity.y = 0;
                if (moveState.jump) { velocity.y = 8; isGrounded = false; canJump = false; }
            }

            playerGroup.position.add(velocity.clone().multiplyScalar(0.5));
            if (playerGroup.position.y < 0) { playerGroup.position.y = 0; isGrounded = true; canJump = true; }

            // Rotate player to face movement direction if moving
            if (moveVec.lengthSq() > 0.1) {
                const targetRotation = Math.atan2(moveVec.x, moveVec.z);
                const currentRotation = playerGroup.rotation.y;
                // Simple lerp for smooth rotation
                let diff = targetRotation - currentRotation;
                while (diff > Math.PI) diff -= Math.PI * 2;
                while (diff < -Math.PI) diff += Math.PI * 2;
                playerGroup.rotation.y += diff * 0.1;
            }

            // Camera Logic
            if (isThirdPerson) {
                const offset = new THREE.Vector3(0, 3, 6);
                // In 3rd person, we usually want camera behind player, 
                // OR freely rotating. 
                // If we want free rotation (like RPG), we don't force camera position based on player rotation.
                // We keep the camera logic from the previous request (CameraRig rotates with mouse), 
                // but since we made movement camera-relative, we just follow position.
                
                // Position CameraRig at Player
                cameraRig.position.copy(playerGroup.position).add(new THREE.Vector3(0, 1.6, 0));
                
                // Camera itself is offset inside rig
                camera.position.set(0, 1, 4); // Local offset within Rig
                camera.lookAt(cameraRig.position); // Look at Rig center (Player head)
            } else {
                // 1st Person: Camera inside Rig
                cameraRig.position.copy(playerGroup.position).add(new THREE.Vector3(0, 1.6, 0));
                camera.position.set(0, 0, 0);
                // Camera rotation is controlled by mouse directly on CameraRig in 1st person
                // No extra lookAt needed as Rig rotation handles it
            }

            // --- Raycasting ---
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const intersects = raycaster.intersectObjects(interactables, true);
            const prompt = document.getElementById('interaction-overlay');
            const mobileBtn = document.getElementById('interact-btn-mobile');

            if (intersects.length > 0 && intersects[0].distance < 4) {
                let target = intersects[0].object;
                while(target.parent && target.parent.type !== 'Scene' && !target.userData.isInteractable) target = target.parent;
                if(target.userData.isInteractable && !target.visible) target = null;

                if(target) {
                    currentLookTarget = target;
                    prompt.style.display = 'flex';
                    mobileBtn.style.display = 'flex';
                    let label = "Interact";
                    if(target.userData.type === 'upgrade') { label = target.userData.name.split('(')[0]; document.getElementById('progress-bar-container').style.display = 'none'; } 
                    else { document.getElementById('progress-bar-container').style.display = 'block'; }

                    if(target.userData.type === 'prep') {
                        if (target.userData.prepState === 'empty' && gameState.rawDough > 0) label = "Roll Dough (0.5s)";
                        else if (target.userData.prepState === 'rolled' && gameState.ingredients > 0) label = "Add Toppings (1.5s)";
                        else if (target.userData.prepState === 'topped') label = "Grab Raw Pizza (0.5s)";
                        else label = "Need Items";
                    } else if (target.userData.type === 'dough') label = "Grab Dough (2.0s)";
                    else if (target.userData.type === 'ingred') label = "Grab Toppings (2.0s)";
                    else if (target.userData.type === 'oven') label = gameState.rawPizzas > 0 ? "Bake Pizza (2.0s)" : "Need Raw Pizza";
                    else if (target.userData.type === 'sell') label = (gameState.cookedPizzas > 0 || gameState.burgers > 0) ? "Sell Goods (1.0s)" : "Nothing to Sell";
                    else if (target.userData.type === 'fryer') label = "Fry Burger (2.0s)";

                    document.getElementById('interaction-label').innerText = label;

                    if(isInteracting && currentInteractionDuration > 0.1) {
                        const elapsed = clock.getElapsedTime() - interactionStartTime;
                        const pct = Math.min(100, (elapsed / currentInteractionDuration) * 100);
                        document.getElementById('progress-bar-fill').style.width = pct + '%';
                        if(activeVisualItem) activeVisualItem.scale.setScalar(Math.min(1, pct / 100));
                        if(elapsed >= currentInteractionDuration) completeInteraction();
                    } else if(isInteracting && currentInteractionDuration <= 0.1) { completeInteraction(); }
                } else resetInteractUI(prompt, mobileBtn);
            } else resetInteractUI(prompt, mobileBtn);
            
            runWorkerLogic(delta);
            renderer.render(scene, camera);
        }

        function resetInteractUI(prompt, mobileBtn) {
            currentLookTarget = null;
            prompt.style.display = 'none';
            mobileBtn.style.display = 'none';
            if(isInteracting) endInteraction();
        }

        function updateUI() {
            document.getElementById('money-stat').textContent = `$${gameState.money.toFixed(2)}`;
            document.getElementById('rebirth-stat').textContent = gameState.rebirths;
            document.getElementById('equip-stat').textContent = gameState.equipmentLevel;
            const rCost = 100000 * (gameState.rebirths + 1);
            document.getElementById('ui-rebirth-cost').textContent = `$${rCost.toLocaleString()}`;

            const inventoryData = [
                { id: 'slot-dough', count: gameState.rawDough, elId: 'inv-dough-count', max: 5 },
                { id: 'slot-ingred', count: gameState.ingredients, elId: 'inv-ingred-count', max: 5 },
                { id: 'slot-raw', count: gameState.rawPizzas, elId: 'inv-raw-count', max: 5 },
                { id: 'slot-cooked', count: gameState.cookedPizzas, elId: 'inv-cooked-count', max: 5 },
                { id: 'slot-burger', count: gameState.burgers, elId: 'inv-burger-count', max: 5, visible: gameState.hasFastFood }
            ];

            inventoryData.forEach(item => {
                const el = document.getElementById(item.elId);
                const parent = document.getElementById(item.id);
                el.textContent = item.count;
                if (item.count > 0 || (item.visible && item.count === 0)) parent.classList.add('active-slot');
                else parent.classList.remove('active-slot');
            });
            const bSlot = document.getElementById('slot-burger');
            if(gameState.hasFastFood) bSlot.classList.add('active-slot');
            else bSlot.classList.remove('active-slot');

            interactables.forEach(obj => {
                if (obj.userData.type === 'upgrade' && obj.visible) {
                    const uType = obj.userData.upgradeType;
                    let cost = obj.userData.cost;
                    let level = obj.userData.level;
                    if (uType === 'equip') { cost = UPGRADE_COSTS.EQUIPMENT(gameState.equipmentLevel); level = gameState.equipmentLevel; } 
                    else if (uType === 'worker') { cost = UPGRADE_COSTS.WORKER(gameState.workerCount); level = gameState.workerCount; } 
                    else if (uType === 'stand') { cost = UPGRADE_COSTS.STAND(gameState.standLevel); level = gameState.standLevel; }

                    if (uType !== 'cashier' && uType !== 'conveyor' && uType !== 'expand') {
                        const canvas = obj.children[1].material.map.image;
                        const ctx = canvas.getContext('2d');
                        ctx.clearRect(0, 0, canvas.width, canvas.height);
                        ctx.fillStyle = 'white'; ctx.font = 'bold 30px Arial'; ctx.textAlign = 'center';
                        ctx.fillText(obj.userData.name.split('(')[0], 128, 40);
                        ctx.fillStyle = '#f1c40f';
                        ctx.fillText(`Lvl ${level + 1} | $${cost.toLocaleString()}`, 128, 90);
                        obj.children[1].material.map.needsUpdate = true;
                    }
                }
            });
        }

        async function saveGame() { if(!userId) return; try { const dataToSave = JSON.parse(JSON.stringify(gameState)); await setDoc(doc(db, 'artifacts', appId, 'users', userId, 'pizza_tycoon_save'), dataToSave); } catch(e) {} }
        async function loadGame() {
            try {
                const snap = await getDoc(doc(db, 'artifacts', appId, 'users', userId, 'pizza_tycoon_save'));
                if(snap.exists()) {
                    gameState = { ...gameState, ...snap.data() };
                    for(let i=0; i<gameState.workerCount; i++) spawnWorker(i+1);
                    interactables.forEach(obj => {
                         if(obj.userData.type === 'fryer' && gameState.hasFastFood) obj.visible = true;
                         if(obj.userData.upgradeType === 'cashier' && gameState.hasCashier) obj.visible = false;
                         if(obj.userData.upgradeType === 'conveyor' && gameState.hasConveyor) obj.visible = false;
                         if(obj.userData.upgradeType === 'expand' && gameState.hasFastFood) obj.visible = false;
                         if(obj.userData.type === 'prep') updatePrepVisuals(obj);
                    });
                }
                updateUI(); updateHeldItemVisual();
            } catch(e) {}
        }
        
        function spawnWorker(id) {
            const worker = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.8, 0.3), materials.shirt); body.position.y = 0.9; worker.add(body);
            const head = new THREE.Mesh(new THREE.SphereGeometry(0.25), materials.skin); head.position.y = 1.45; worker.add(head);
            const hat = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.25, 0.3), new THREE.MeshStandardMaterial({color: 0xffffff})); hat.position.y = 1.7; worker.add(hat);
            const targetX = Math.random() * 20 - 10; const targetZ = Math.random() * 20 - 10;
            worker.position.set(targetX, 0, targetZ); scene.add(worker);
            npcList.push({ mesh: worker, state: 'idle', timer: 0, target: new THREE.Vector3(targetX, 0, targetZ) });
        }

        initGame();
        window.addEventListener('resize', () => { if(camera && renderer) { camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight); } });
    </script>
</body>
</html>
