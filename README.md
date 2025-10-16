<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Watershed Investigator</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: #f0fdf4; /* Light Green/Bay feel */
        }
        .site-pin {
            position: absolute;
            width: 32px;
            height: 32px;
            background-color: #ef4444; /* Red pin */
            border-radius: 50%;
            cursor: pointer;
            border: 3px solid #fca5a5;
            transition: all 0.2s ease-in-out;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            color: white;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            z-index: 10;
        }
        .site-pin:hover {
            transform: scale(1.1);
            box-shadow: 0 8px 10px rgba(0, 0, 0, 0.2);
        }
        .site-pin.complete {
            background-color: #10b981; /* Green when complete */
            border-color: #a7f3d0;
        }
        .bay-map {
            background-image: url('https://placehold.co/800x600/bbf7d0/14532d?text=Galveston+Bay+Watershed+Map');
            background-size: cover;
            background-position: center;
            position: relative;
            min-height: 400px;
            border-radius: 1rem;
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
        }
        .modal-overlay {
            background-color: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(4px);
        }
    </style>
</head>
<body class="p-4 sm:p-8">

    <!-- Game Container -->
    <div id="game-container" class="max-w-4xl mx-auto bg-white rounded-2xl shadow-2xl p-6 md:p-10 border-t-8 border-emerald-600">
        
        <!-- Header and Controls -->
        <header class="mb-6 flex flex-col md:flex-row justify-between items-center space-y-4 md:space-y-0">
            <h1 id="header-title" class="text-3xl font-bold text-emerald-800">Watershed Investigator</h1>
            <div class="flex space-x-3 items-center">
                <!-- NEW DOWNLOAD BUTTON ADDED HERE -->
                <button id="download-btn" onclick="downloadHtml()" class="p-2 bg-sky-500 text-white rounded-lg hover:bg-sky-600 transition duration-150 text-sm font-semibold">
                    Download HTML
                </button>
                <button id="lang-switch" onclick="toggleLanguage()" class="p-2 bg-slate-200 text-slate-700 rounded-lg hover:bg-slate-300 transition duration-150 text-sm font-semibold">
                    Español
                </button>
                <div id="score-display" class="p-2 bg-sky-100 text-sky-700 rounded-lg font-semibold text-sm">
                    Score: 0/5
                </div>
            </div>
        </header>

        <!-- Main Game Area -->
        <div id="main-view" class="space-y-6">
            
            <!-- Instructions Panel -->
            <div id="instructions" class="bg-emerald-50 border-l-4 border-emerald-500 text-emerald-800 p-4 rounded-lg">
                <p id="inst-text" class="text-sm"></p>
            </div>

            <!-- Map View (Initial View) -->
            <div id="map-view" class="bay-map w-full h-96 relative">
                <!-- Site Pins will be added here by JS -->
            </div>
            
            <!-- Result/Final Message Container -->
            <div id="result-message" class="hidden text-center p-6 bg-yellow-50 rounded-xl border-2 border-yellow-300">
                <h2 id="result-title" class="text-2xl font-bold text-yellow-800 mb-2"></h2>
                <p id="result-text" class="text-lg text-yellow-700"></p>
                <p id="final-score" class="text-3xl font-extrabold mt-4 text-emerald-700"></p>
            </div>

        </div>

        <!-- Submission Feedback Message -->
        <div id="feedback-message" class="hidden mt-4 p-3 rounded-lg text-white font-semibold"></div>

    </div>

    <!-- Site Investigation Modal -->
    <div id="investigation-modal" class="hidden fixed inset-0 z-50 flex items-center justify-center p-4 modal-overlay transition-opacity duration-300">
        <div class="bg-white rounded-xl w-full max-w-2xl shadow-2xl transform transition-transform duration-300 scale-100 p-6 space-y-5">
            
            <h2 id="modal-title" class="text-2xl font-bold text-emerald-700 border-b pb-2"></h2>
            
            <!-- Data Panel -->
            <div class="bg-blue-50 p-4 rounded-lg">
                <h3 id="data-header" class="text-xl font-semibold text-blue-800 mb-3"></h3>
                <div id="data-list" class="grid grid-cols-2 gap-3 text-sm">
                    <!-- Data points loaded here -->
                </div>
            </div>

            <!-- Analysis Panel -->
            <form id="analysis-form" class="space-y-4">
                
                <!-- 1. Cause of Pollution (Now Multiple Choice) -->
                <div class="space-y-2">
                    <label id="cause-label" class="block text-md font-medium text-gray-700"></label>
                    <div id="mcq-cause-container" class="space-y-3 p-3 bg-gray-50 rounded-lg border border-gray-200">
                        <!-- Radio buttons loaded here -->
                    </div>
                </div>

                <!-- 2. Proposed Solution (Still Open-Ended) -->
                <div class="space-y-2">
                    <label for="solution-input" id="solution-label" class="block text-md font-medium text-gray-700"></label>
                    <!-- Added onpaste handler to prevent copy-pasting -->
                    <textarea id="solution-input" rows="2" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-emerald-500 focus:border-emerald-500 shadow-sm"></textarea>
                </div>

                <!-- Action Buttons -->
                <div class="flex justify-end space-x-3 pt-3">
                    <button type="button" onclick="closeModal()" id="cancel-btn" class="px-4 py-2 text-sm font-semibold text-gray-700 bg-gray-200 rounded-lg hover:bg-gray-300 transition duration-150"></button>
                    <button type="submit" id="submit-btn" class="px-4 py-2 text-sm font-semibold text-white bg-emerald-600 rounded-lg hover:bg-emerald-700 transition duration-150 shadow-md"></button>
                </div>
            </form>

        </div>
    </div>

<script>
    // --- Configuration and Data ---

    const SITES = [
        {
            id: 1, name_en: "Agricultural Runoff Creek", name_es: "Arroyo de Escorrentía Agrícola",
            coords: { top: '20%', left: '30%' }, // Top-Left of the map
            data: [
                { param: "Nitrates (NO₃⁻)", value: "High (>10 mg/L)", unit: "mg/L" },
                { param: "Dissolved Oxygen (DO)", value: "Low (3 mg/L)", unit: "mg/L" },
                { param: "pH", value: "Normal (6.8)", unit: "" },
                { param: "Phosphates (PO₄³⁻)", value: "High (0.5 mg/L)", unit: "mg/L" }
            ],
            // MCQ Setup
            mcq_options: [
                "Farm fertilizer runoff",
                "Broken oil pipelines",
                "Factory chemical discharge"
            ],
            correct_mcq_answer: "Farm fertilizer runoff",
            // Solution Keywords (for open-ended scoring)
            solution_keywords: ["buffer zone", "planting trees/grass", "cover crops", "slow down runoff", "better farming"],
            completed: false, score: 0
        },
        {
            id: 2, name_en: "Industrial Discharge Zone", name_es: "Zona de Descarga Industrial",
            coords: { top: '45%', left: '15%' },
            data: [
                { param: "pH", value: "Very High (9.5)", unit: "" },
                { param: "Heavy Metals (Pb, Hg)", value: "Trace Detected", unit: "" },
                { param: "Dissolved Oxygen (DO)", value: "Normal (7 mg/L)", unit: "mg/L" },
                { param: "Temperature", value: "High (35°C)", unit: "°C" }
            ],
            // MCQ Setup
            mcq_options: [
                "Leaking boat fuel",
                "Hot chemical waste from a factory",
                "Storm drain trash"
            ],
            correct_mcq_answer: "Hot chemical waste from a factory",
            // Solution Keywords
            solution_keywords: ["clean the water", "filters", "government laws", "cooling tower", "strict rules"],
            completed: false, score: 0
        },
        {
            id: 3, name_en: "Suburban Wastewater Outlet", name_es: "Salida de Aguas Residuales Suburbanas",
            coords: { top: '70%', left: '40%' },
            data: [
                { param: "Fecal Coliform", value: "Very High", unit: "CFU/100mL" },
                { param: "Dissolved Oxygen (DO)", value: "Very Low (2 mg/L)", unit: "mg/L" },
                { param: "Turbidity", value: "High", unit: "NTU" },
                { param: "Chlorine Residual", value: "Trace", unit: "mg/L" }
            ],
            // MCQ Setup
            mcq_options: [
                "Ocean salt mixing with freshwater",
                "Leaking sewer pipes and pet waste",
                "Too much clean, hot water"
            ],
            correct_mcq_answer: "Leaking sewer pipes and pet waste",
            // Solution Keywords
            solution_keywords: ["fix pipes", "sewer upgrades", "check septic tanks", "cleanup education", "proper disposal"],
            completed: false, score: 0
        },
        {
            id: 4, name_en: "Ship Channel Marina", name_es: "Puerto Deportivo del Canal de Navegación",
            coords: { top: '60%', left: '75%' },
            data: [
                { param: "Hydrocarbons", value: "High (Oil Sheen visible)", unit: "" },
                { param: "Turbidity", value: "Moderate", unit: "NTU" },
                { param: "Temperature", value: "Normal (25°C)", unit: "°C" },
                { param: "Copper", value: "Trace (from antifouling paint)", unit: "" }
            ],
            // MCQ Setup
            mcq_options: [
                "Natural decomposition of fish",
                "Oil leaks and toxic boat paint",
                "Too many fishing net fragments"
            ],
            correct_mcq_answer: "Oil leaks and toxic boat paint",
            // Solution Keywords
            solution_keywords: ["recycle oil", "no-spill pump", "educational signs", "better maintenance", "containment boom"],
            completed: false, score: 0
        },
        {
            id: 5, name_en: "Urban Storm Drain Outlet", name_es: "Salida de Drenaje Pluvial Urbano",
            coords: { top: '35%', left: '70%' },
            data: [
                { param: "Temperature", value: "Extremely High (38°C)", unit: "°C" },
                { param: "Litter", value: "High (Plastics, Trash)", unit: "" },
                { param: "Dissolved Oxygen (DO)", value: "Low (4.5 mg/L)", unit: "mg/L" },
                { param: "Suspended Solids", value: "Very High", unit: "mg/L" }
            ],
            // MCQ Setup
            mcq_options: [
                "Litter and heat from paved roads",
                "Natural volcanic activity heating the water",
                "Deep sea oil drilling waste"
            ],
            correct_mcq_answer: "Litter and heat from paved roads",
            // Solution Keywords
            solution_keywords: ["rain gardens", "porous pavement", "street sweeping", "trash cleanups", "stop dumping", "reduce heat"],
            completed: false, score: 0
        }
    ];

    const LANGUAGE_PACKS = {
        en: {
            title: "Watershed Investigator",
            instructions: "Welcome, future environmental scientist! Your job is to check out 5 polluted spots in the Galveston Bay area. For each spot, look at the data, figure out **what caused the pollution** (Multiple Choice), and suggest a **real solution** (Open-Ended, minimum 20 words). Click a red pin to begin!",
            score: "Score:",
            modal_header: "Site Investigation: ",
            data_header: "Collected Data Analysis",
            // Updated labels for MCQ
            cause_label: "1. Select the main cause of pollution:", 
            solution_label: "2. Proposed Solution (Write how it can be fixed - min. 20 words):",
            cancel_btn: "Close",
            submit_btn: "Submit Analysis",
            // Updated feedback for 50/50 scoring
            feedback_success: "Great work! You got the Cause (MCQ) correct and earned 1 point for a good Solution.",
            feedback_partial: "You got partial credit! Cause Score: {cause}/1, Solution Score: {solution}/1. Review your answers and think about how to fix the problem!",
            feedback_fail: "Keep trying! Cause Score: 0/1, Solution Score: 0/1. Look at the data and the site name again.",
            result_title_complete: "Mission Complete!",
            result_text_complete: "You have successfully investigated all 5 sites and proposed solutions to protect the watershed. Your efforts make a difference!",
            lang_switch_btn: "Español",
            paste_blocked: "Copy-pasting is disabled. Please type your original solution (minimum 20 words)."
        },
        es: {
            title: "Investigador de Cuencas",
            instructions: "¡Bienvenido, futuro científico ambiental! Tu trabajo es investigar 5 puntos contaminados en la Bahía de Galveston. Para cada punto, mira los datos, averigua **qué causó la contaminación** (Opción Múltiple) y sugiere una **solución real** (Respuesta Abierta, mínimo 20 palabras). ¡Haz clic en un pin rojo para comenzar!",
            score: "Puntuación:",
            modal_header: "Investigación del Sitio: ",
            data_header: "Análisis de Datos Recolectados",
            // Updated labels for MCQ (Spanish)
            cause_label: "1. Selecciona la causa principal de la contaminación:",
            solution_label: "2. Solución Propuesta (Escribe cómo se puede arreglar - mín. 20 palabras):",
            cancel_btn: "Cerrar",
            submit_btn: "Enviar Análisis",
            // Updated feedback for 50/50 scoring (Spanish)
            feedback_success: "¡Buen trabajo! Obtuviste la Causa (Opción Múltiple) correcta y ganaste 1 punto por una buena Solución.",
            feedback_partial: "¡Obtuviste crédito parcial! Puntuación de la Causa: {cause}/1, Puntuación de la Solución: {solution}/1. ¡Revisa tus respuestas y piensa en cómo solucionar el problema!",
            feedback_fail: "¡Sigue intentándolo! Puntuación de la Causa: 0/1, Puntuación de la Solución: 0/1. Mira los datos y el nombre del sitio de nuevo.",
            result_title_complete: "¡Misión Cumplida!",
            result_text_complete: "Has investigado con éxito los 5 sitios y propuesto soluciones para proteger la cuenca hidrográfica. ¡Tus esfuerzos marcan la diferencia!",
            lang_switch_btn: "English",
            paste_blocked: "La copia y pega está deshabilitada. Por favor, escribe tu solución original (mínimo 20 palabras)."
        }
    };

    let currentLanguage = 'en';
    let currentSite = null;

    // --- Utility Functions ---

    /**
     * Checks if the user's input contains any of the required keywords.
     * @param {string} input - The user's text input.
     * @param {string[]} keywords - The required list of keywords.
     * @returns {number} 1 if score is earned, 0 otherwise.
     */
    function checkAnswer(input, keywords) {
        const normalizedInput = input.toLowerCase().replace(/[^\w\s]/g, '');
        const found = keywords.some(keyword => normalizedInput.includes(keyword));
        return found ? 1 : 0;
    }

    /**
     * Calculates the word count of a given string.
     * @param {string} text - The input string.
     * @returns {number} The word count.
     */
    function getWordCount(text) {
        if (!text) return 0;
        // Match sequences of non-whitespace characters
        const matches = text.trim().match(/\S+/g);
        return matches ? matches.length : 0;
    }

    /**
     * Downloads the current HTML content as a file.
     */
    function downloadHtml() {
        // Get the entire outer HTML of the document
        const htmlContent = '<!DOCTYPE html>\n' + document.documentElement.outerHTML;
        
        // Create a Blob from the HTML content
        const blob = new Blob([htmlContent], { type: 'text/html' });
        
        // Create an anchor element for the download
        const a = document.createElement('a');
        a.href = URL.createObjectURL(blob);
        a.download = 'watershed_investigator.html'; // Suggested file name
        
        // Append to body, click it, and remove it
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);

        // Show feedback message
        showFeedback(
            currentLanguage === 'en' ? 'File downloaded successfully!' : 'Archivo descargado exitosamente!', 
            'bg-sky-600'
        );
    }
    // --- Rendering Functions ---

    function updateText() {
        const text = LANGUAGE_PACKS[currentLanguage];
        document.getElementById('header-title').textContent = text.title;
        document.getElementById('inst-text').textContent = text.instructions;
        document.getElementById('lang-switch').textContent = text.lang_switch_btn;
        
        // Set the download button text
        document.getElementById('download-btn').textContent = currentLanguage === 'en' ? 'Download HTML' : 'Descargar HTML';

        updateScoreDisplay();

        if (currentSite) {
            const siteName = currentLanguage === 'en' ? currentSite.name_en : currentSite.name_es;
            document.getElementById('modal-title').textContent = text.modal_header + siteName;
            document.getElementById('data-header').textContent = text.data_header;
            document.getElementById('cause-label').textContent = text.cause_label;
            document.getElementById('solution-label').textContent = text.solution_label;
            document.getElementById('cancel-btn').textContent = text.cancel_btn;
            document.getElementById('submit-btn').textContent = text.submit_btn;
        }

        if (document.getElementById('result-message').classList.contains('complete')) {
            document.getElementById('result-title').textContent = text.result_title_complete;
            document.getElementById('result-text').textContent = text.result_text_complete;
        }
    }

    function toggleLanguage() {
        currentLanguage = currentLanguage === 'en' ? 'es' : 'en';
        updateText();
    }

    function updateScoreDisplay() {
        const totalScore = SITES.reduce((sum, site) => sum + site.score, 0);
        const totalPossible = SITES.length * 2;
        const text = LANGUAGE_PACKS[currentLanguage];

        document.getElementById('score-display').textContent = `${text.score} ${totalScore}/${totalPossible}`;

        const completedSites = SITES.filter(s => s.completed).length;
        if (completedSites === SITES.length) {
            showFinalResults(totalScore, totalPossible);
        }
    }

    function renderMap() {
        const mapView = document.getElementById('map-view');
        mapView.innerHTML = '';

        SITES.forEach(site => {
            const pin = document.createElement('div');
            pin.className = `site-pin transition-all ${site.completed ? 'complete' : ''}`;
            pin.style.top = site.coords.top;
            pin.style.left = site.coords.left;
            pin.textContent = site.id;
            pin.onclick = () => openInvestigationModal(site.id);

            // Add hover effect with site name (English/Spanish based on current lang)
            pin.title = currentLanguage === 'en' ? site.name_en : site.name_es;

            mapView.appendChild(pin);
        });
    }

    // --- Modal Logic ---

    function openInvestigationModal(siteId) {
        currentSite = SITES.find(s => s.id === siteId);
        
        const causeContainer = document.getElementById('mcq-cause-container');
        const solutionInput = document.getElementById('solution-input');
        const submitBtn = document.getElementById('submit-btn');

        // Reset/Update Modal Content
        updateText();
        renderDataList(currentSite.data);
        renderCauseMCQ(currentSite.mcq_options);
        
        // Handle completed sites (Read-only mode)
        if (currentSite.completed) {
            // Re-render MCQ to show previous selection, then disable all inputs
            renderCauseMCQ(currentSite.mcq_options, currentSite.lastCause);
            causeContainer.querySelectorAll('input').forEach(input => input.disabled = true);
            
            solutionInput.value = currentSite.lastSolution || '';
            solutionInput.disabled = true;
            submitBtn.classList.add('hidden');
        } else {
            // Enable inputs for new attempts
            causeContainer.querySelectorAll('input').forEach(input => input.disabled = false);
            solutionInput.value = '';
            solutionInput.disabled = false;
            submitBtn.classList.remove('hidden');
        }

        // Show modal
        document.getElementById('investigation-modal').classList.remove('hidden');
    }
    
    // Function to render the multiple choice questions
    function renderCauseMCQ(options, lastSelectedValue = null) {
        const container = document.getElementById('mcq-cause-container');
        container.innerHTML = ''; // Clear previous options

        options.forEach((option, index) => {
            const id = `mcq-option-${currentSite.id}-${index}`;
            
            const div = document.createElement('div');
            div.className = 'flex items-start';

            const input = document.createElement('input');
            input.type = 'radio';
            input.name = 'cause-mcq';
            input.id = id;
            input.value = option;
            input.className = 'mt-1 h-4 w-4 text-emerald-600 border-gray-300 focus:ring-emerald-500';
            
            // Set checked state if the user previously selected this option
            if (lastSelectedValue && lastSelectedValue === option) {
                input.checked = true;
            }

            const label = document.createElement('label');
            label.htmlFor = id;
            label.className = 'ml-3 text-sm font-medium text-gray-700 cursor-pointer';
            label.textContent = option;

            div.appendChild(input);
            div.appendChild(label);
            container.appendChild(div);
        });
    }


    function renderDataList(data) {
        const list = document.getElementById('data-list');
        list.innerHTML = '';
        data.forEach(d => {
            const item = document.createElement('div');
            item.className = 'flex justify-between items-center bg-white p-2 rounded-md shadow-sm border border-blue-100';
            const label = document.createElement('span');
            label.className = 'font-semibold text-gray-600';
            label.textContent = d.param;
            const value = document.createElement('span');
            value.className = 'font-bold text-blue-900';
            value.textContent = `${d.value} ${d.unit || ''}`;
            item.appendChild(label);
            item.appendChild(value);
            list.appendChild(item);
        });
    }

    function closeModal() {
        document.getElementById('investigation-modal').classList.add('hidden');
        currentSite = null;
        // Hide feedback when closing
        document.getElementById('feedback-message').classList.add('hidden');
    }

    // --- Game Logic ---

    function handleSubmit(event) {
        event.preventDefault();
        if (!currentSite) return;

        const solutionInput = document.getElementById('solution-input').value.trim();
        
        // Get selected MCQ value
        const selectedMcq = document.querySelector('input[name="cause-mcq"]:checked');
        const causeInput = selectedMcq ? selectedMcq.value : '';

        // Calculate word count for the solution
        const solutionWordCount = getWordCount(solutionInput);
        const MIN_WORDS = 20;

        if (!selectedMcq) {
            const text_en = "Please select the main cause of pollution (Multiple Choice).";
            const text_es = "Por favor, selecciona la causa principal de la contaminación (Opción Múltiple).";
            showFeedback(currentLanguage === 'en' ? text_en : text_es, 'bg-red-500');
            return;
        }

        if (solutionWordCount < MIN_WORDS) {
            const text_en = `Your solution must be at least ${MIN_WORDS} words. Your current solution has ${solutionWordCount} words. Please elaborate.`;
            const text_es = `Tu solución debe tener al menos ${MIN_WORDS} palabras. Tu solución actual tiene ${solutionWordCount} palabras. Por favor, sé más detallado.`;
            showFeedback(currentLanguage === 'en' ? text_en : text_es, 'bg-red-500');
            return;
        }

        // Store user input (MCQ answer is stored in lastCause)
        currentSite.lastCause = causeInput;
        currentSite.lastSolution = solutionInput;

        // Scoring: 
        // 1. Cause Score (MCQ): Exact match against the correct_mcq_answer
        const causeScore = (causeInput === currentSite.correct_mcq_answer) ? 1 : 0;

        // 2. Solution Score (Open-Ended): Keyword matching
        const solutionScore = checkAnswer(solutionInput, currentSite.solution_keywords);
        
        const totalScore = causeScore + solutionScore;

        currentSite.score = totalScore;
        currentSite.completed = true;

        showSubmissionFeedback(causeScore, solutionScore);
        
        // Update game state
        updateScoreDisplay();
        renderMap(); // Re-render map to update pin color
        
        // Set to read-only mode after submission
        document.getElementById('mcq-cause-container').querySelectorAll('input').forEach(input => input.disabled = true);
        document.getElementById('solution-input').disabled = true;
        document.getElementById('submit-btn').classList.add('hidden');
    }

    function showSubmissionFeedback(causeScore, solutionScore) {
        const text = LANGUAGE_PACKS[currentLanguage];
        const feedbackEl = document.getElementById('feedback-message');
        
        let message;
        let colorClass;

        if (causeScore === 1 && solutionScore === 1) {
            message = text.feedback_success;
            colorClass = 'bg-green-600';
        } else if (causeScore === 0 && solutionScore === 0) {
            message = text.feedback_fail;
            colorClass = 'bg-red-600';
        } else {
            message = text.feedback_partial.replace('{cause}', causeScore).replace('{solution}', solutionScore);
            colorClass = 'bg-orange-600';
        }

        feedbackEl.className = `mt-4 p-3 rounded-lg text-white font-semibold ${colorClass}`;
        feedbackEl.textContent = message;
        feedbackEl.classList.remove('hidden');
    }

    function showFeedback(message, colorClass) {
        const feedbackEl = document.getElementById('feedback-message');
        feedbackEl.className = `mt-4 p-3 rounded-lg text-white font-semibold ${colorClass}`;
        feedbackEl.textContent = message;
        feedbackEl.classList.remove('hidden');
        // Optionally hide after a few seconds
        setTimeout(() => feedbackEl.classList.add('hidden'), 5000);
    }


    function showFinalResults(totalScore, totalPossible) {
        const text = LANGUAGE_PACKS[currentLanguage];
        document.getElementById('instructions').classList.add('hidden');
        document.getElementById('map-view').classList.add('hidden');
        
        const resultEl = document.getElementById('result-message');
        resultEl.classList.add('complete');
        resultEl.classList.remove('hidden');
        
        document.getElementById('result-title').textContent = text.result_title_complete;
        document.getElementById('result-text').textContent = text.result_text_complete;
        document.getElementById('final-score').textContent = `${text.score} ${totalScore}/${totalPossible}`;
    }

    // --- Initialization ---

    document.addEventListener('DOMContentLoaded', () => {
        // Initial setup
        updateText();
        renderMap();
        
        // Attach form listener
        document.getElementById('analysis-form').addEventListener('submit', handleSubmit);

        // --- PASTE PREVENTION LOGIC ---
        const solutionInput = document.getElementById('solution-input');
        solutionInput.addEventListener('paste', function(e) {
            e.preventDefault();
            // Show feedback using the centralized function
            showFeedback(
                LANGUAGE_PACKS[currentLanguage].paste_blocked, 
                'bg-yellow-600'
            );
        });
        // --- END PASTE PREVENTION LOGIC ---
    });

</script>

</body>
</html>
