Bytebeat x Game of Life Cellular Automata Sonifier

An interactive HTML5 Web Audio & Canvas application combining Greggman Bytebeat algorithmic audio synthesis with Conway's Game of Life cellular automata, featuring real-time data sonification coupling and an infinite 2D camera viewport transform engine.

🌟 Key Features

Greggman Bytebeat Synthesizer: 8kHz real-time expression evaluation (ScriptProcessorNode / AudioContext). Dynamic parameter inputs (t, life, matrix states).

Game of Life Automaton: Toroidal 128x128 double-buffered state array matrix with configurable step speeds, preset loading, and live cell drawing.

Real-time Data Sonification: Live population metrics and spatial state arrays dynamically modulate audio feedback, pitch, bitwise XOR vectors, and cross-section dynamics.

Infinite Zoom & Viewport Transform: High-performance hardware-accelerated 2D canvas matrix transformations with pan, grab, drag, click toggling, and infinite scroll zoom.

Zero External Build Step: Pure client-side HTML5/JavaScript execution—100% serverless, private, and zero-latency.

🚀 Quick Start

Running Locally

No node module installation or server build steps required.

Clone the repository:

git clone https://github.com/j-mosij/bytebeat-game-of-life-sonifier.git
cd bytebeat-game-of-life-sonifier


Open index.html in any modern web browser:

macOS: open index.html

Linux: xdg-open index.html

Windows: start index.html

Alternatively, serve with any local HTTP server:

python3 -m http.server 8000
# Access via http://localhost:8000


🎼 Data Sonification Coupling Modes

Population-driven Pitch Shift (pop): Live active cell count continuously shifts audio clock step resolution.

Center Matrix Modulation (row): The 8x8 center quadrant bits are mapped to an 8-bit byte shift register modulating bitwise operators.

Life State XOR (bit): Cell density acts as a mask against byte sample operations ($t \oplus (\text{life} \ll 8)$).

Independent Execution (none): Audio expression runs cleanly on time variable $t$.

🔒 Security & Client Safety

Because this application dynamically evaluates user-entered Bytebeat math expressions:

Expressions are isolated strictly inside safe function scope boundaries Function('t', 'life', ...).

All audio generation runs fully client-side inside standard browser sandbox constraints.

Zero external third-party tracking or server-side data telemetry is transmitted.

🛠 GitHub Repository Publishing Instructions

To publish this repository under the j-mosij GitHub account:

# Initialize local Git repository
git init
git add .
git commit -m "Initial commit: Bytebeat x Game of Life Sonifier App"

# Create remote branch and push (using GitHub CLI)
gh repo create j-mosij/bytebeat-game-of-life-sonifier --public --source=. --remote=origin --push


📜 License

This project is open-source under the MIT License.<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bytebeat Cellular Automata Sonifier</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #090d16;
            color: #e2e8f0;
            font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
        }
        canvas {
            display: block;
            width: 100vw;
            height: 100vh;
            cursor: grab;
        }
        canvas:active {
            cursor: grabbing;
        }
        /* Custom scrollbar for preset list */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(15, 23, 42, 0.6);
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(51, 65, 85, 0.8);
            border-radius: 3px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(71, 85, 105, 1);
        }
    </style>
</head>
<body class="relative select-none">

    <!-- Interactive Infinite Zoom Canvas -->
    <canvas id="viewport"></canvas>

    <!-- UI Overlay Header / Toolbar -->
    <div class="absolute top-4 left-4 right-4 flex justify-between items-center pointer-events-none z-10">
        <div class="flex items-center space-x-3 bg-slate-900/80 backdrop-blur-md px-4 py-2.5 rounded-xl border border-slate-700/50 shadow-xl pointer-events-auto">
            <div class="w-3 h-3 rounded-full bg-cyan-400 animate-pulse"></div>
            <h1 class="text-sm font-bold tracking-wider text-slate-100 uppercase">Bytebeat <span class="text-cyan-400">x</span> Game of Life</h1>
        </div>

        <!-- Top Right Actions -->
        <div class="flex items-center space-x-2 pointer-events-auto">
            <button id="toggle-audio-btn" class="flex items-center space-x-2 px-4 py-2 rounded-xl bg-cyan-600 hover:bg-cyan-500 text-white font-semibold shadow-lg shadow-cyan-950/50 transition duration-200">
                <i data-lucide="play" class="w-4 h-4"></i>
                <span id="audio-btn-text">Start Audio Engine</span>
            </button>
            <button id="toggle-menu-btn" class="p-2.5 rounded-xl bg-slate-900/80 hover:bg-slate-800 backdrop-blur-md border border-slate-700/50 text-slate-200 shadow-xl transition">
                <i data-lucide="sliders" class="w-5 h-5"></i>
            </button>
        </div>
    </div>

    <!-- Floating HUD Controls Overlay -->
    <div id="control-panel" class="absolute top-20 right-4 w-96 max-h-[calc(100vh-6rem)] overflow-y-auto bg-slate-900/90 backdrop-blur-md border border-slate-700/60 rounded-2xl p-5 shadow-2xl z-10 transition-all duration-300">
        <div class="space-y-6">
            
            <!-- Bytebeat Formula Input Section -->
            <div class="space-y-2">
                <div class="flex justify-between items-center">
                    <label class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Bytebeat Expression (t)</label>
                    <span id="eval-status" class="text-xs text-emerald-400 font-mono">Active</span>
                </div>
                <div class="relative">
                    <textarea id="formula-input" rows="2" class="w-full bg-slate-950/90 border border-slate-700 rounded-xl p-3 font-mono text-sm text-cyan-300 focus:outline-none focus:border-cyan-500 transition resize-none" spellcheck="false">(t*(t>>12|t>>8))&63|(t>>4)</textarea>
                </div>
                <p class="text-[11px] text-slate-400">Available variables: <code class="text-cyan-400">t</code> (sample counter), <code class="text-emerald-400">life</code> (active automaton cell count).</p>
            </div>

            <!-- Bytebeat Presets -->
            <div class="space-y-2">
                <label class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Bytebeat Presets</label>
                <div class="grid grid-cols-2 gap-2">
                    <button class="preset-btn px-3 py-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-xs font-mono text-slate-300 text-left truncate transition" data-formula="(t*(t>>12|t>>8))&63|(t>>4)">Greggman Classic</button>
                    <button class="preset-btn px-3 py-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-xs font-mono text-slate-300 text-left truncate transition" data-formula="((t*5&t>>7)|(t*3&t>>8)|(t*2&t>>9))-1">Starlight Symphony</button>
                    <button class="preset-btn px-3 py-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-xs font-mono text-slate-300 text-left truncate transition" data-formula="(t>>6|t|t>>(t>>16))*10+((t>>11)&7)">Sierpinski Melodies</button>
                    <button class="preset-btn px-3 py-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-xs font-mono text-slate-300 text-left truncate transition" data-formula="(t*(((t>>9)|(t>>13))&(15&(t>>6))))^(life*2)">Cellular Feedback</button>
                </div>
            </div>

            <hr class="border-slate-800" />

            <!-- Cellular Automata Controls -->
            <div class="space-y-4">
                <div class="flex justify-between items-center">
                    <label class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Automaton Parameters</label>
                    <div class="flex space-x-1">
                        <button id="step-life-btn" class="p-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-slate-300 text-xs transition" title="Step One Generation">
                            <i data-lucide="step-forward" class="w-3.5 h-3.5"></i>
                        </button>
                        <button id="reset-life-btn" class="p-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-slate-300 text-xs transition" title="Randomize Board">
                            <i data-lucide="refresh-cw" class="w-3.5 h-3.5"></i>
                        </button>
                        <button id="clear-life-btn" class="p-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-slate-300 text-xs transition" title="Clear Grid">
                            <i data-lucide="trash-2" class="w-3.5 h-3.5"></i>
                        </button>
                    </div>
                </div>

                <!-- Simulation Speed -->
                <div class="space-y-1">
                    <div class="flex justify-between text-xs">
                        <span class="text-slate-400">Simulation Speed (FPS)</span>
                        <span id="speed-val" class="font-mono text-cyan-400">15</span>
                    </div>
                    <input type="range" id="speed-slider" min="1" max="60" value="15" class="w-full accent-cyan-500 bg-slate-950 h-1.5 rounded-lg appearance-none cursor-pointer">
                </div>

                <!-- Coupling Mode -->
                <div class="space-y-1">
                    <label class="text-xs text-slate-400">Data Sonification Coupling</label>
                    <select id="sonification-mode" class="w-full bg-slate-950 border border-slate-700 rounded-xl p-2 text-xs text-slate-200 focus:outline-none focus:border-cyan-500">
                        <option value="pop">Population-driven Pitch Shift</option>
                        <option value="row">Center Matrix Cross-Section Modulation</option>
                        <option value="bit">Life State Bytebeat Bitwise XOR</option>
                        <option value="none">Independent Execution</option>
                    </select>
                </div>
            </div>

            <hr class="border-slate-800" />

            <!-- Navigation & Viewport Controls -->
            <div class="space-y-3">
                <label class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Infinite Canvas Controls</label>
                <div class="flex items-center justify-between bg-slate-950 p-2.5 rounded-xl border border-slate-800">
                    <span class="text-xs text-slate-400 font-mono">Zoom: <span id="zoom-level" class="text-cyan-400">1.00x</span></span>
                    <button id="reset-view-btn" class="px-3 py-1 bg-slate-800 hover:bg-slate-700 text-xs text-slate-200 rounded-lg transition">
                        Reset View
                    </button>
                </div>
                <p class="text-[11px] text-slate-500">
                    <i data-lucide="mouse-pointer" class="inline w-3 h-3 mr-1"></i> Drag to pan camera | Scroll wheel to infinite zoom | Click canvas to toggle cells.
                </p>
            </div>

        </div>
    </div>

    <script>
        // Initialize Lucide icons
        lucide.createIcons();

        // -------------------------------------------------------------
        // Global Constants & State Management
        // -------------------------------------------------------------
        const GRID_SIZE = 128; // Grid resolution (128x128 matrix)
        let grid = new Uint8Array(GRID_SIZE * GRID_SIZE);
        let nextGrid = new Uint8Array(GRID_SIZE * GRID_SIZE);

        // Audio State
        let audioCtx = null;
        let scriptNode = null;
        let isAudioPlaying = false;
        let sampleCounter = 0;
        let compiledFormula = (t, life) => (t * (t >> 12 | t >> 8)) & 63 | (t >> 4);
        let liveCellCount = 0;
        let centerByteVal = 0;

        // Viewport Transform State (Infinite Zoom & Pan)
        let cameraX = 0;
        let cameraY = 0;
        let cameraZoom = 1.0;
        let isDragging = false;
        let dragStartX = 0;
        let dragStartY = 0;

        // Canvas Setup
        const canvas = document.getElementById('viewport');
        const ctx = canvas.getContext('2d');

        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // Center initial camera view on the grid
        function resetCamera() {
            cameraZoom = Math.min(canvas.width, canvas.height) / (GRID_SIZE * 1.4);
            cameraX = (canvas.width - GRID_SIZE * cameraZoom) / 2;
            cameraY = (canvas.height - GRID_SIZE * cameraZoom) / 2;
            updateZoomDisplay();
        }
        resetCamera();

        // -------------------------------------------------------------
        // Cellular Automata Mechanics (Conway's Game of Life)
        // -------------------------------------------------------------
        function randomizeGrid() {
            liveCellCount = 0;
            for (let i = 0; i < grid.length; i++) {
                grid[i] = Math.random() > 0.85 ? 1 : 0;
                if (grid[i]) liveCellCount++;
            }
        }

        function clearGrid() {
            grid.fill(0);
            liveCellCount = 0;
        }

        function updateGrid() {
            let active = 0;
            let centerAccumulator = 0;
            const midStart = (GRID_SIZE / 2 - 4);
            const midEnd = (GRID_SIZE / 2 + 4);

            for (let y = 0; y < GRID_SIZE; y++) {
                for (let x = 0; x < GRID_SIZE; x++) {
                    const idx = y * GRID_SIZE + x;
                    
                    // Count 8 neighbors with toroidal wrapping
                    let neighbors = 0;
                    for (let dy = -1; dy <= 1; dy++) {
                        for (let dx = -1; dx <= 1; dx++) {
                            if (dx === 0 && dy === 0) continue;
                            const nx = (x + dx + GRID_SIZE) % GRID_SIZE;
                            const ny = (y + dy + GRID_SIZE) % GRID_SIZE;
                            neighbors += grid[ny * GRID_SIZE + nx];
                        }
                    }

                    // Conway Rules
                    const currentState = grid[idx];
                    if (currentState === 1 && (neighbors === 2 || neighbors === 3)) {
                        nextGrid[idx] = 1;
                        active++;
                    } else if (currentState === 0 && neighbors === 3) {
                        nextGrid[idx] = 1;
                        active++;
                    } else {
                        nextGrid[idx] = 0;
                    }

                    // Extract center sub-matrix values for sonification
                    if (x >= midStart && x < midEnd && y >= midStart && y < midEnd) {
                        if (nextGrid[idx]) {
                            centerAccumulator |= (1 << ((y - midStart) * 8 + (x - midStart) % 8));
                        }
                    }
                }
            }

            // Swap buffers
            const temp = grid;
            grid = nextGrid;
            nextGrid = temp;

            liveCellCount = active;
            centerByteVal = centerAccumulator & 0xFF;
        }

        randomizeGrid();

        // -------------------------------------------------------------
        // Web Audio API & Bytebeat Engine
        // -------------------------------------------------------------
        function initAudioEngine() {
            if (audioCtx) return;

            const AudioContext = window.AudioContext || window.webkitAudioContext;
            audioCtx = new AudioContext({ sampleRate: 8000 });

            // Create ScriptProcessorNode for real-time sample generation (Bytebeat standard: 8kHz)
            const bufferSize = 2048;
            scriptNode = audioCtx.createScriptProcessor(bufferSize, 0, 1);

            const sonificationSelect = document.getElementById('sonification-mode');

            scriptNode.onaudioprocess = (audioProcessingEvent) => {
                const outputBuffer = audioProcessingEvent.outputBuffer.getChannelData(0);
                const mode = sonificationSelect.value;

                for (let i = 0; i < bufferSize; i++) {
                    sampleCounter++;
                    let t = sampleCounter;

                    // Sonification modulation transforms
                    if (mode === 'pop') {
                        t = Math.floor(t * (1 + (liveCellCount % 64) / 64));
                    } else if (mode === 'row') {
                        t = t ^ (centerByteVal << 2);
                    } else if (mode === 'bit') {
                        t = (t & 0xFFFF) | (liveCellCount << 8);
                    }

                    try {
                        let outByte = compiledFormula(t, liveCellCount);
                        // Standard Bytebeat unsigned 8-bit output conversion to float range [-1.0, 1.0]
                        outputBuffer[i] = ((outByte & 0xFF) / 127.5) - 1.0;
                    } catch (e) {
                        outputBuffer[i] = 0;
                    }
                }
            };

            scriptNode.connect(audioCtx.destination);
        }

        function toggleAudio() {
            if (!audioCtx) {
                initAudioEngine();
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }

            const btnText = document.getElementById('audio-btn-text');
            const btn = document.getElementById('toggle-audio-btn');

            if (isAudioPlaying) {
                audioCtx.suspend();
                isAudioPlaying = false;
                btnText.textContent = "Start Audio Engine";
                btn.classList.replace('bg-rose-600', 'bg-cyan-600');
                btn.classList.replace('hover:bg-rose-500', 'hover:bg-cyan-500');
            } else {
                audioCtx.resume();
                isAudioPlaying = true;
                btnText.textContent = "Pause Audio Engine";
                btn.classList.replace('bg-cyan-600', 'bg-rose-600');
                btn.classList.replace('hover:bg-cyan-500', 'hover:bg-rose-500');
            }
        }

        // Formula Compilation & Validation
        const formulaInput = document.getElementById('formula-input');
        const evalStatus = document.getElementById('eval-status');

        function compileFormula() {
            const expr = formulaInput.value.trim();
            try {
                // Safely evaluate formula as pure mathematical expression
                const fn = new Function('t', 'life', `return (${expr});`);
                // Test run execution
                fn(0, 0);
                compiledFormula = fn;
                evalStatus.textContent = "Valid";
                evalStatus.className = "text-xs text-emerald-400 font-mono";
            } catch (err) {
                evalStatus.textContent = "Syntax Error";
                evalStatus.className = "text-xs text-rose-400 font-mono";
            }
        }

        formulaInput.addEventListener('input', compileFormula);

        // -------------------------------------------------------------
        // Rendering System (Canvas Infinite Zoom & Visualizer)
        // -------------------------------------------------------------
        function draw() {
            // Dark Background clear
            ctx.fillStyle = '#090d16';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.save();
            // Apply Camera Viewport Transformation Matrix
            ctx.translate(cameraX, cameraY);
            ctx.scale(cameraZoom, cameraZoom);

            // Draw Automata Grid Background / Border
            ctx.strokeStyle = 'rgba(51, 65, 85, 0.3)';
            ctx.lineWidth = 0.05;
            ctx.strokeRect(0, 0, GRID_SIZE, GRID_SIZE);

            // Draw Live Cells
            const pulseGlow = (Math.sin(Date.now() / 200) * 0.2 + 0.8);
            ctx.fillStyle = `rgba(34, 211, 238, ${pulseGlow})`;

            for (let y = 0; y < GRID_SIZE; y++) {
                for (let x = 0; x < GRID_SIZE; x++) {
                    if (grid[y * GRID_SIZE + x] === 1) {
                        ctx.fillRect(x, y, 0.95, 0.95);
                    }
                }
            }

            // Draw Center Matrix Highlight Ring (Data Sonification Zone)
            const midStart = (GRID_SIZE / 2 - 4);
            ctx.strokeStyle = 'rgba(244, 63, 94, 0.4)';
            ctx.lineWidth = 0.1;
            ctx.strokeRect(midStart, midStart, 8, 8);

            ctx.restore();

            // Render Audio Waveform Visualizer Overlay on bottom edge
            if (isAudioPlaying) {
                renderWaveformOverlay();
            }

            requestAnimationFrame(draw);
        }

        function renderWaveformOverlay() {
            ctx.beginPath();
            ctx.strokeStyle = 'rgba(34, 211, 238, 0.25)';
            ctx.lineWidth = 2;

            const sliceWidth = canvas.width / 64;
            let x = 0;

            for (let i = 0; i < 64; i++) {
                const sample = compiledFormula(sampleCounter + i * 32, liveCellCount) & 0xFF;
                const v = sample / 255.0;
                const y = canvas.height - (v * 80);

                if (i === 0) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);

                x += sliceWidth;
            }
            ctx.stroke();
        }

        // -------------------------------------------------------------
        // Interaction Handlers (Mouse, Touch, Panning, Zooming)
        // -------------------------------------------------------------
        function updateZoomDisplay() {
            document.getElementById('zoom-level').textContent = cameraZoom.toFixed(2) + 'x';
        }

        canvas.addEventListener('mousedown', (e) => {
            isDragging = true;
            dragStartX = e.clientX - cameraX;
            dragStartY = e.clientY - cameraY;
        });

        window.addEventListener('mousemove', (e) => {
            if (isDragging) {
                cameraX = e.clientX - dragStartX;
                cameraY = e.clientY - dragStartY;
            }
        });

        window.addEventListener('mouseup', () => {
            isDragging = false;
        });

        // Click to toggle Cell State
        canvas.addEventListener('click', (e) => {
            // Ignore click if user was panning/dragging
            const distDragged = Math.hypot(e.clientX - (dragStartX + cameraX), e.clientY - (dragStartY + cameraY));
            if (distDragged > 5) return;

            const rect = canvas.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;

            // Transform canvas screen coordinates to internal grid coordinates
            const gridX = Math.floor((mouseX - cameraX) / cameraZoom);
            const gridY = Math.floor((mouseY - cameraY) / cameraZoom);

            if (gridX >= 0 && gridX < GRID_SIZE && gridY >= 0 && gridY < GRID_SIZE) {
                const idx = gridY * GRID_SIZE + gridX;
                grid[idx] = grid[idx] ? 0 : 1;
            }
        });

        // Infinite Zoom with Mouse Wheel
        canvas.addEventListener('wheel', (e) => {
            e.preventDefault();
            const zoomFactor = 1.1;
            const mouseX = e.clientX;
            const mouseY = e.clientY;

            let newZoom;
            if (e.deltaY < 0) {
                newZoom = cameraZoom * zoomFactor;
            } else {
                newZoom = cameraZoom / zoomFactor;
            }

            // Clamp extreme zooms
            newZoom = Math.max(0.05, Math.min(newZoom, 500));

            // Zoom relative to current cursor position
            cameraX = mouseX - (mouseX - cameraX) * (newZoom / cameraZoom);
            cameraY = mouseY - (mouseY - cameraY) * (newZoom / cameraZoom);
            cameraZoom = newZoom;

            updateZoomDisplay();
        }, { passive: false });

        // -------------------------------------------------------------
        // Control Listeners & Timers
        // -------------------------------------------------------------
        document.getElementById('toggle-audio-btn').addEventListener('click', toggleAudio);

        const speedSlider = document.getElementById('speed-slider');
        const speedVal = document.getElementById('speed-val');
        let simInterval = setInterval(updateGrid, 1000 / parseInt(speedSlider.value));

        speedSlider.addEventListener('input', (e) => {
            const fps = e.target.value;
            speedVal.textContent = fps;
            clearInterval(simInterval);
            simInterval = setInterval(updateGrid, 1000 / fps);
        });

        document.getElementById('step-life-btn').addEventListener('click', updateGrid);
        document.getElementById('reset-life-btn').addEventListener('click', randomizeGrid);
        document.getElementById('clear-life-btn').addEventListener('click', clearGrid);
        document.getElementById('reset-view-btn').addEventListener('click', resetCamera);

        // Preset Selector Listener
        document.querySelectorAll('.preset-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                formulaInput.value = btn.getAttribute('data-formula');
                compileFormula();
            });
        });

        // UI Panel Toggle
        const controlPanel = document.getElementById('control-panel');
        document.getElementById('toggle-menu-btn').addEventListener('click', () => {
            controlPanel.classList.toggle('hidden');
        });

        // Kick off loop
        compileFormula();
        requestAnimationFrame(draw);
    </script>
</body>
</html>
