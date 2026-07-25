<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pantry Inventory & Dynamic Grocery List</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Firebase Compatibility Libraries -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        :root {
            --app-bg: #fdfbf7;
            --card-bg: #ffffff;
            --text-main: #2d3748;
            --accent-color: #d97706;
            --accent-hover: #b45309;
            --row-select-bg: #fef3c7;
            --border-color: #e5e7eb;
            --item-text-color: #111827;
        }

        .dark-mode {
            --app-bg: #121827;
            --card-bg: #1f2937;
            --text-main: #f3f4f6;
            --accent-color: #f59e0b;
            --accent-hover: #d97706;
            --row-select-bg: #374151;
            --border-color: #374151;
            --item-text-color: #f9fafb;
        }

        body {
            background-color: var(--app-bg);
            color: var(--text-main);
            font-family: 'Inter', sans-serif;
            transition: background-color 0.3s ease, color 0.3s ease;
        }

        .item-name-text {
            color: var(--item-text-color) !important;
        }

        .custom-card {
            background-color: var(--card-bg);
            transition: background-color 0.3s ease;
        }

        .selected-row {
            background-color: var(--row-select-bg) !important;
        }

        /* Dynamic Accent Utilities */
        .btn-accent {
            background-color: var(--accent-color) !important;
            color: #ffffff !important;
        }
        .btn-accent:hover {
            background-color: var(--accent-hover) !important;
        }
        .text-accent {
            color: var(--accent-color) !important;
        }
        .border-accent {
            border-color: var(--accent-color) !important;
        }
        .bg-accent-light {
            background-color: var(--row-select-bg) !important;
        }

        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.05);
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 4px;
        }
    </style>
</head>
<body class="min-h-full flex flex-col p-4 md:p-8">

    <header class="max-w-6xl w-full mx-auto mb-6 flex flex-col md:flex-row md:items-center justify-between gap-4 border-b border-gray-200 dark:border-gray-700 pb-4">
        <div>
            <h1 class="text-2xl md:text-3xl font-bold tracking-tight">🍱 Family Pantry & Grocery</h1>
            <p class="text-xs md:text-sm text-gray-500 dark:text-gray-400 mt-1">Audit pantry inventory, customize units & tags, and generate your dynamic grocery list.</p>
        </div>

        <!-- Global Navigation Controls -->
        <div class="flex items-center gap-2 flex-wrap">
            <div class="inline-flex rounded-xl p-1 bg-gray-200/60 dark:bg-gray-800 border border-gray-300/50 dark:border-gray-700">
                <button id="navPantryBtn" onclick="switchWindow('pantry')" class="px-3 py-1.5 rounded-lg text-xs font-semibold transition-all btn-accent shadow-sm">
                    📦 Pantry Inventory
                </button>
                <button id="navGroceryBtn" onclick="switchWindow('grocery')" class="px-3 py-1.5 rounded-lg text-xs font-semibold text-gray-600 dark:text-gray-300 hover:text-amber-600 transition-all">
                    🛒 Dynamic Grocery List
                </button>
            </div>

            <!-- Customizer & Cloud Buttons -->
            <button onclick="openModal('themeModal')" class="p-2 rounded-xl border border-gray-300 dark:border-gray-700 hover:bg-gray-100 dark:hover:bg-gray-800 transition text-sm" title="Customize Theme Colors">
                🎨
            </button>
            <button onclick="toggleDarkMode()" class="p-2 rounded-xl border border-gray-300 dark:border-gray-700 hover:bg-gray-100 dark:hover:bg-gray-800 transition text-sm" id="darkModeBtn" title="Toggle Light/Dark Mode">
                🌙
            </button>
            <button onclick="openModal('cloudModal')" id="syncStatusBadge" class="inline-flex items-center px-3 py-1.5 rounded-xl text-xs font-medium border border-gray-300 dark:border-gray-700 hover:bg-gray-100 dark:hover:bg-gray-800 transition">
                <span class="w-2 h-2 mr-1.5 rounded-full bg-gray-400"></span> ☁️ Cloud Sync
            </button>
        </div>
    </header>

    <main class="max-w-6xl w-full mx-auto flex-1 flex flex-col gap-6">

        <!-- ==================== WINDOW 1: PANTRY INVENTORY ==================== -->
        <div id="pantryWindow" class="space-y-6">
            
            <!-- SECTION 1: PANTRY OVERVIEW -->
            <section class="custom-card rounded-2xl p-4 md:p-6 shadow-sm border border-gray-200/80 dark:border-gray-800 space-y-4">
                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                    <div>
                        <h2 class="text-xl font-bold tracking-tight">Pantry Overview</h2>
                    </div>

                    <!-- Search Input -->
                    <div class="relative w-full sm:w-64">
                        <input type="text" id="searchInput" oninput="renderTable()" placeholder="Search food items..." 
                               class="w-full pl-9 pr-3 py-1.5 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent focus:outline-none focus:ring-2 focus:ring-amber-500">
                        <span class="absolute left-3 top-2 text-gray-400 text-xs">🔍</span>
                    </div>
                </div>

                <!-- Tag Filter Tabs -->
                <div id="tagTabsContainer" class="flex gap-2 border-b border-gray-200 dark:border-gray-700 pb-2 overflow-x-auto custom-scrollbar">
                    <!-- Dynamic Tag Tabs Injected Here -->
                </div>

                <!-- Section 1 Inventory Table -->
                <div class="overflow-x-auto rounded-xl border border-gray-200 dark:border-gray-700 custom-scrollbar">
                    <table class="w-full text-left text-xs border-collapse">
                        <thead class="bg-gray-100/80 dark:bg-gray-800/80 text-gray-600 dark:text-gray-300 font-semibold border-b border-gray-200 dark:border-gray-700">
                            <tr>
                                <th class="p-3 w-10 text-center">Select</th>
                                <th class="p-3 cursor-pointer hover:text-amber-600 transition" onclick="sortByName()" title="Click to sort alphabetically">
                                    Name <span id="sortIndicator" class="text-gray-400">↕</span>
                                </th>
                                <th class="p-3 text-center">Quantity Needed</th>
                                <th class="p-3">
                                    <div class="flex items-center gap-1">
                                        <span>Unit</span>
                                        <button onclick="openUnitsModal()" class="text-gray-400 hover:text-amber-600 text-xs p-0.5 rounded" title="Manage Custom Units">⚙️</button>
                                    </div>
                                </th>
                                <th class="p-3 text-center">Quantity in Stock</th>
                                <th class="p-3">
                                    <div class="flex items-center gap-1">
                                        <span>Tag</span>
                                        <button onclick="openTagsModal()" class="text-gray-400 hover:text-amber-600 text-xs p-0.5 rounded" title="Manage Tag Names & Colors">⚙️</button>
                                    </div>
                                </th>
                            </tr>
                        </thead>
                        <tbody id="inventoryTableBody" class="divide-y divide-gray-200 dark:divide-gray-800">
                            <!-- Table Rows Injected Here -->
                        </tbody>
                    </table>
                </div>
            </section>

            <!-- SECTION 2: USER ACTION BUTTONS -->
            <section class="custom-card rounded-2xl p-4 shadow-sm border border-gray-200/80 dark:border-gray-800 flex flex-wrap gap-3 items-center justify-between">
                <div class="flex items-center gap-2">
                    <button onclick="openAddModal()" class="px-4 py-2 btn-accent rounded-xl text-xs font-bold transition shadow-sm flex items-center gap-1.5 cursor-pointer">
                        <span>➕</span> ADD
                    </button>
                    <button id="modifyBtn" onclick="openModifyModal()" disabled class="px-4 py-2 bg-gray-200 dark:bg-gray-800 text-gray-400 rounded-xl text-xs font-bold transition cursor-not-allowed flex items-center gap-1.5">
                        <span>✏️</span> MODIFY
                    </button>
                    <button id="deleteBtn" onclick="openDeleteModal()" disabled class="px-4 py-2 bg-gray-200 dark:bg-gray-800 text-gray-400 rounded-xl text-xs font-bold transition cursor-not-allowed flex items-center gap-1.5">
                        <span>🗑️</span> DELETE
                    </button>
                </div>
                <div class="text-[11px] text-gray-400 italic">
                    Select 1 row to MODIFY. Select 1 or more rows to DELETE.
                </div>
            </section>
        </div>

        <!-- ==================== WINDOW 2: DYNAMIC GROCERY LIST ==================== -->
        <div id="groceryWindow" class="space-y-6 hidden">
            <section class="custom-card rounded-2xl p-4 md:p-6 shadow-sm border border-gray-200/80 dark:border-gray-800 space-y-6">
                <div>
                    <h2 class="text-xl font-bold tracking-tight">🛒 Dynamic Grocery List</h2>
                    <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Select which tags to include in your shopping trip. Essential items are active by default.</p>
                </div>

                <!-- Tag Filter Controls -->
                <div class="space-y-2">
                    <label class="text-xs font-bold text-gray-500 dark:text-gray-400 uppercase tracking-wider">Include Tags in List:</label>
                    <div id="groceryTagFilters" class="flex flex-wrap gap-2">
                        <!-- Dynamic Tag Checkboxes Injected Here -->
                    </div>
                </div>

                <!-- Generated Grocery Items View -->
                <div class="border border-gray-200 dark:border-gray-700 rounded-xl p-4 bg-gray-50/50 dark:bg-gray-800/40 space-y-3">
                    <div class="flex justify-between items-center pb-2 border-b border-gray-200 dark:border-gray-700">
                        <span class="text-xs font-bold text-gray-600 dark:text-gray-300">Items To Buy</span>
                        <button onclick="copyGroceryList()" class="px-3 py-1 bg-amber-500/10 text-accent hover:bg-amber-500/20 rounded-lg text-xs font-semibold transition border border-amber-500/20">
                            📋 Copy List
                        </button>
                    </div>

                    <div id="groceryItemsList" class="space-y-2 max-h-96 overflow-y-auto custom-scrollbar pr-1">
                        <!-- Grocery items rendered here -->
                    </div>
                </div>
            </section>
        </div>
    </main>

    <!-- ==================== MODALS ==================== -->

    <!-- ADD / MODIFY FOOD ITEM MODAL -->
    <div id="itemModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="custom-card rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center">
                <h3 id="itemModalTitle" class="text-lg font-bold">Add New Food Item</h3>
                <button onclick="closeModal('itemModal')" class="text-gray-400 hover:text-gray-600 text-xl font-bold">&times;</button>
            </div>

            <form id="itemForm" onsubmit="saveItem(event)" class="space-y-3">
                <div>
                    <label class="block text-xs font-medium mb-1">Name of Food Item <span class="text-red-500">*</span></label>
                    <input type="text" id="modalName" required class="w-full p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent focus:outline-none focus:ring-2 focus:ring-amber-500">
                </div>

                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-medium mb-1">Desired Quantity <span class="text-red-500">*</span></label>
                        <input type="number" id="modalNeeded" step="any" min="0" required class="w-full p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent focus:outline-none focus:ring-2 focus:ring-amber-500">
                    </div>
                    <div>
                        <label class="block text-xs font-medium mb-1">Unit of Measurement <span class="text-red-500">*</span></label>
                        <select id="modalUnit" required class="w-full p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent focus:outline-none focus:ring-2 focus:ring-amber-500 dark:bg-gray-800">
                            <!-- Injected dynamically -->
                        </select>
                    </div>
                </div>

                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-medium mb-1">Quantity in Stock</label>
                        <input type="number" id="modalStock" step="any" min="0" value="0" class="w-full p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent focus:outline-none focus:ring-2 focus:ring-amber-500">
                    </div>
                    <div>
                        <label class="block text-xs font-medium mb-1">Tag Attached <span class="text-red-500">*</span></label>
                        <select id="modalTag" required class="w-full p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent focus:outline-none focus:ring-2 focus:ring-amber-500 dark:bg-gray-800">
                            <!-- Injected dynamically -->
                        </select>
                    </div>
                </div>

                <div class="flex justify-end gap-2 pt-2">
                    <button type="button" onclick="closeModal('itemModal')" class="px-4 py-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700">Cancel</button>
                    <button type="submit" class="px-4 py-2 text-xs rounded-xl btn-accent font-bold">Save Item</button>
                </div>
            </form>
        </div>
    </div>

    <!-- DELETE CONFIRMATION MODAL -->
    <div id="deleteModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="custom-card rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <h3 class="text-lg font-bold text-red-600 dark:text-red-400">Confirm Removal</h3>
            <p class="text-xs text-gray-500 dark:text-gray-400">Are you sure you want to remove the following food items from your pantry?</p>

            <ul id="deleteListSummary" class="max-h-40 overflow-y-auto space-y-1.5 p-3 rounded-xl bg-gray-100 dark:bg-gray-800 text-xs custom-scrollbar">
                <!-- Injected dynamically -->
            </ul>

            <div class="flex justify-end gap-2 pt-2">
                <button onclick="closeModal('deleteModal')" class="px-4 py-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700">Cancel</button>
                <button onclick="confirmDelete()" class="px-4 py-2 text-xs rounded-xl bg-red-600 text-white font-bold hover:bg-red-700">Delete Permanently</button>
            </div>
        </div>
    </div>

    <!-- CUSTOM UNITS MANAGER MODAL -->
    <div id="unitsModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="custom-card rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center border-b border-gray-200 dark:border-gray-700 pb-2">
                <h3 class="text-lg font-bold">⚙️ Units of Measurement</h3>
                <button onclick="closeModal('unitsModal')" class="text-gray-400 hover:text-gray-600 text-xl font-bold">&times;</button>
            </div>

            <div id="unitsListContainer" class="space-y-2 max-h-60 overflow-y-auto custom-scrollbar pr-1">
                <!-- Dynamic units list -->
            </div>

            <!-- Add New Unit Row -->
            <div class="pt-2 border-t border-gray-200 dark:border-gray-700 space-y-2">
                <div class="text-xs font-bold text-gray-500">Add Custom Unit</div>
                <div class="grid grid-cols-3 gap-2">
                    <input type="text" id="newUnitSymbol" maxlength="3" placeholder="Symbol (e.g. Crt)" class="p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent">
                    <input type="text" id="newUnitName" maxlength="10" placeholder="Name (e.g. Crate)" class="col-span-2 p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent">
                </div>
                <button onclick="addCustomUnit()" class="w-full py-2 btn-accent rounded-xl text-xs font-bold">➕ Add Unit</button>
            </div>
        </div>
    </div>

    <!-- TAG MANAGER MODAL -->
    <div id="tagsModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="custom-card rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center border-b border-gray-200 dark:border-gray-700 pb-2">
                <h3 class="text-lg font-bold">⚙️ Manage Tags</h3>
                <button onclick="closeModal('tagsModal')" class="text-gray-400 hover:text-gray-600 text-xl font-bold">&times;</button>
            </div>

            <div id="tagsListContainer" class="space-y-2 max-h-60 overflow-y-auto custom-scrollbar pr-1">
                <!-- Dynamic tags list -->
            </div>

            <!-- Add Custom Tag Form -->
            <div class="pt-2 border-t border-gray-200 dark:border-gray-700 space-y-2">
                <div class="text-xs font-bold text-gray-500">Add Custom Tag</div>
                <div class="flex gap-2">
                    <input type="text" id="newTagName" placeholder="Tag Name" class="flex-1 p-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700 bg-transparent">
                    <input type="color" id="newTagBg" value="#fef3c7" class="h-8 w-10 p-0.5 rounded-lg border border-gray-300 cursor-pointer" title="Badge Background Color">
                    <input type="color" id="newTagText" value="#92400e" class="h-8 w-10 p-0.5 rounded-lg border border-gray-300 cursor-pointer" title="Badge Text Color">
                </div>
                <button onclick="addCustomTag()" class="w-full py-2 btn-accent rounded-xl text-xs font-bold">➕ Add Tag</button>
            </div>
        </div>
    </div>

    <!-- THEME CUSTOMIZER MODAL -->
    <div id="themeModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="custom-card rounded-2xl max-w-md w-full p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center border-b border-gray-200 dark:border-gray-700 pb-2">
                <h3 class="text-lg font-bold">🎨 Color Theme Palette</h3>
                <button onclick="closeModal('themeModal')" class="text-gray-400 hover:text-gray-600 text-xl font-bold">&times;</button>
            </div>

            <!-- Preset Palettes -->
            <div>
                <label class="block text-xs font-bold text-gray-500 mb-2">Default Theme Presets</label>
                <div class="grid grid-cols-5 gap-2">
                    <button onclick="applyPresetTheme('amber')" class="h-10 rounded-xl bg-amber-500 hover:opacity-90 text-white text-xs font-bold">Amber</button>
                    <button onclick="applyPresetTheme('mint')" class="h-10 rounded-xl bg-emerald-600 hover:opacity-90 text-white text-xs font-bold">Mint</button>
                    <button onclick="applyPresetTheme('rose')" class="h-10 rounded-xl bg-rose-500 hover:opacity-90 text-white text-xs font-bold">Rose</button>
                    <button onclick="applyPresetTheme('lavender')" class="h-10 rounded-xl bg-indigo-500 hover:opacity-90 text-white text-xs font-bold">Lavender</button>
                    <button onclick="applyPresetTheme('peach')" class="h-10 rounded-xl bg-orange-400 hover:opacity-90 text-white text-xs font-bold">Peach</button>
                </div>
            </div>

            <!-- Custom Element Pickers -->
            <div class="space-y-3 pt-2 border-t border-gray-200 dark:border-gray-700">
                <label class="block text-xs font-bold text-gray-500">Custom Colors</label>
                
                <div class="grid grid-cols-2 gap-3 text-xs">
                    <div class="flex items-center justify-between p-2 rounded-xl border border-gray-200 dark:border-gray-700">
                        <span>App Background</span>
                        <input type="color" id="themeAppBg" onchange="applyCustomColorsFromPickers()" class="w-8 h-8 rounded border border-gray-300">
                    </div>
                    <div class="flex items-center justify-between p-2 rounded-xl border border-gray-200 dark:border-gray-700">
                        <span>Card Background</span>
                        <input type="color" id="themeCardBg" onchange="applyCustomColorsFromPickers()" class="w-8 h-8 rounded border border-gray-300">
                    </div>
                    <div class="flex items-center justify-between p-2 rounded-xl border border-gray-200 dark:border-gray-700">
                        <span>Food Item Text</span>
                        <input type="color" id="themeItemText" onchange="applyCustomColorsFromPickers()" class="w-8 h-8 rounded border border-gray-300">
                    </div>
                    <div class="flex items-center justify-between p-2 rounded-xl border border-gray-200 dark:border-gray-700">
                        <span>Accent Color</span>
                        <input type="color" id="themeAccent" onchange="applyCustomColorsFromPickers()" class="w-8 h-8 rounded border border-gray-300">
                    </div>
                    <div class="flex items-center justify-between p-2 rounded-xl border border-gray-200 dark:border-gray-700">
                        <span>Selected Row</span>
                        <input type="color" id="themeRowSelect" onchange="applyCustomColorsFromPickers()" class="w-8 h-8 rounded border border-gray-300">
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- CLOUD SYNC MODAL -->
    <div id="cloudModal" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="custom-card rounded-2xl max-w-lg w-full p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center">
                <h3 class="text-lg font-bold">☁️ Family Cloud Sync (Firebase)</h3>
                <button onclick="closeModal('cloudModal')" class="text-gray-400 hover:text-gray-600 text-xl font-bold">&times;</button>
            </div>

            <p class="text-xs text-gray-500 dark:text-gray-400">Paste your Firebase web app configuration below (e.g. <code>const firebaseConfig = { ... };</code>) to sync pantry updates across family members in real time.</p>
            
            <div id="cloudErrorBox" class="hidden text-xs text-red-600 bg-red-100 dark:bg-red-900/30 p-3 rounded-xl border border-red-200 font-mono whitespace-pre-wrap"></div>

            <div>
                <textarea id="firebaseConfigInput" rows="6" placeholder='const firebaseConfig = { apiKey: "...", authDomain: "...", projectId: "..." };'
                          class="w-full p-3 font-mono text-xs rounded-xl border border-gray-300 dark:border-gray-600 bg-transparent focus:outline-none focus:ring-2 focus:ring-amber-500"></textarea>
            </div>

            <div class="flex justify-between items-center">
                <button onclick="disconnectFirebase()" class="text-xs text-red-500 underline">Disconnect Cloud</button>
                <div class="flex gap-2">
                    <button onclick="closeModal('cloudModal')" class="px-4 py-2 text-xs rounded-xl border border-gray-300 dark:border-gray-700">Cancel</button>
                    <button onclick="saveFirebaseConfig()" class="px-4 py-2 text-xs rounded-xl btn-accent font-bold">Connect & Sync</button>
                </div>
            </div>
        </div>
    </div>

    <!-- APPLICATION JAVASCRIPT LOGIC -->
    <script>
        // Global Modal Visibility Scope Helpers
        function openModal(modalId) {
            const modal = document.getElementById(modalId);
            if (modal) {
                modal.classList.remove('hidden');
                if (modalId === 'themeModal') {
                    // Sync current picker values with root computed CSS variables
                    const cs = getComputedStyle(document.documentElement);
                    const hexFromRgb = (rgb) => {
                        if (!rgb || !rgb.startsWith('rgb')) return '#111827';
                        const parts = rgb.match(/\d+/g);
                        if (!parts || parts.length < 3) return '#111827';
                        return '#' + parts.slice(0, 3).map(x => parseInt(x).toString(16).padStart(2, '0')).join('');
                    };
                    const itemTextVal = cs.getPropertyValue('--item-text-color').trim();
                    if (itemTextVal && document.getElementById('themeItemText')) {
                        document.getElementById('themeItemText').value = itemTextVal.startsWith('#') ? itemTextVal : hexFromRgb(itemTextVal);
                    }
                }
            }
        }

        function closeModal(modalId) {
            const modal = document.getElementById(modalId);
            if (modal) modal.classList.add('hidden');
        }

        window.openModal = openModal;
        window.closeModal = closeModal;

        // Application State
        let state = {
            activeWindow: 'pantry',
            activeTagTab: 'ALL',
            sortDirection: 'default', // 'default', 'asc', 'desc'
            selectedItemIds: [],
            editingItemId: null,
            groceryActiveTags: { 'Essential': true }, // Essential active by default
            
            units: [
                { symbol: 'Kg', name: 'Kilogram', isDefault: true },
                { symbol: 'L', name: 'Liter', isDefault: true },
                { symbol: '#', name: 'Unit', isDefault: true }
            ],

            tagsList: [
                { id: 'Essential', name: 'Essential', bg: '#fee2e2', text: '#991b1b', isDefault: true },
                { id: 'Tier 1', name: 'Tier 1', bg: '#fef3c7', text: '#92400e', isDefault: true },
                { id: 'Tier 2', name: 'Tier 2', bg: '#e0e7ff', text: '#3730a3', isDefault: true },
                { id: 'Tier 3', name: 'Tier 3', bg: '#dcfce7', text: '#166534', isDefault: true },
                { id: 'Tier 4', name: 'Tier 4', bg: '#f3e8ff', text: '#6b21a8', isDefault: true }
            ],

            items: [
                { id: '1', name: 'Milk', needed: 4, stock: 1, unit: 'L', tag: 'Essential' },
                { id: '2', name: 'Jasmine Rice', needed: 5, stock: 2.5, unit: 'Kg', tag: 'Essential' },
                { id: '3', name: 'Olive Oil', needed: 2, stock: 0.5, unit: 'L', tag: 'Tier 1' },
                { id: '4', name: 'Eggs', needed: 12, stock: 4, unit: '#', tag: 'Essential' },
                { id: '5', name: 'Dark Chocolate', needed: 3, stock: 1, unit: '#', tag: 'Tier 3' }
            ]
        };

        let db = null;

        function initApp() {
            // Load state from localStorage if available
            const savedItems = localStorage.getItem('pantry_items');
            if (savedItems) state.items = JSON.parse(savedItems);

            const savedUnits = localStorage.getItem('pantry_units');
            if (savedUnits) state.units = JSON.parse(savedUnits);

            const savedTags = localStorage.getItem('pantry_tags');
            if (savedTags) state.tagsList = JSON.parse(savedTags);

            renderTagTabs();
            renderTable();
            renderGroceryList();
            updateButtonStates();

            // Auto-connect Firebase if credentials saved
            const savedConfig = localStorage.getItem('pantry_firebase_config');
            if (savedConfig) {
                try {
                    initFirebaseSync(JSON.parse(savedConfig));
                } catch (e) {
                    console.error("Firebase auto-connect failed:", e);
                }
            }
        }

        function saveStateLocally() {
            localStorage.setItem('pantry_items', JSON.stringify(state.items));
            localStorage.setItem('pantry_units', JSON.stringify(state.units));
            localStorage.setItem('pantry_tags', JSON.stringify(state.tagsList));
            if (db) pushStateToCloud();
        }

        // Window Switcher
        function switchWindow(target) {
            state.activeWindow = target;
            const pantryWin = document.getElementById('pantryWindow');
            const groceryWin = document.getElementById('groceryWindow');
            const navPantryBtn = document.getElementById('navPantryBtn');
            const navGroceryBtn = document.getElementById('navGroceryBtn');

            if (target === 'pantry') {
                pantryWin.classList.remove('hidden');
                groceryWin.classList.add('hidden');
                navPantryBtn.className = "px-3 py-1.5 rounded-lg text-xs font-semibold transition-all btn-accent shadow-sm";
                navGroceryBtn.className = "px-3 py-1.5 rounded-lg text-xs font-semibold text-gray-600 dark:text-gray-300 hover:text-amber-600 transition-all";
            } else {
                pantryWin.classList.add('hidden');
                groceryWin.classList.remove('hidden');
                navGroceryBtn.className = "px-3 py-1.5 rounded-lg text-xs font-semibold transition-all btn-accent shadow-sm";
                navPantryBtn.className = "px-3 py-1.5 rounded-lg text-xs font-semibold text-gray-600 dark:text-gray-300 hover:text-amber-600 transition-all";
                renderGroceryList();
            }
        }

        function renderTagTabs() {
            const container = document.getElementById('tagTabsContainer');
            if (!container) return;

            let html = `
                <button onclick="switchTagTab('ALL')" class="px-3 py-1 rounded-xl text-xs font-bold transition whitespace-nowrap ${state.activeTagTab === 'ALL' ? 'btn-accent' : 'bg-gray-100 dark:bg-gray-800 text-gray-600 dark:text-gray-300'}">
                    ALL (${state.items.length})
                </button>
            `;

            state.tagsList.forEach(t => {
                const count = state.items.filter(i => i.tag === t.name).length;
                const isActive = state.activeTagTab === t.name;
                html += `
                    <button onclick="switchTagTab('${t.name}')" class="px-3 py-1 rounded-xl text-xs font-bold transition whitespace-nowrap flex items-center gap-1.5" style="background-color: ${isActive ? t.bg : ''}; color: ${isActive ? t.text : ''}">
                        <span>${t.name}</span>
                        <span class="text-[10px] px-1.5 py-0.2 rounded-full opacity-80" style="background-color: ${t.text}20">${count}</span>
                    </button>
                `;
            });

            container.innerHTML = html;
        }

        function switchTagTab(tagName) {
            state.activeTagTab = tagName;
            renderTagTabs();
            renderTable();
        }

        function sortByName() {
            if (state.sortDirection === 'default') state.sortDirection = 'asc';
            else if (state.sortDirection === 'asc') state.sortDirection = 'desc';
            else state.sortDirection = 'default';

            const indicator = document.getElementById('sortIndicator');
            if (indicator) {
                indicator.innerText = state.sortDirection === 'asc' ? '▲' : state.sortDirection === 'desc' ? '▼' : '↕';
            }
            renderTable();
        }

        function renderTable() {
            const tbody = document.getElementById('inventoryTableBody');
            if (!tbody) return;

            const searchTerm = (document.getElementById('searchInput')?.value || '').toLowerCase();

            let filtered = state.items.filter(item => {
                const matchesTag = state.activeTagTab === 'ALL' || item.tag === state.activeTagTab;
                const matchesSearch = item.name.toLowerCase().includes(searchTerm);
                return matchesTag && matchesSearch;
            });

            if (state.sortDirection === 'asc') {
                filtered.sort((a, b) => a.name.localeCompare(b.name));
            } else if (state.sortDirection === 'desc') {
                filtered.sort((a, b) => b.name.localeCompare(a.name));
            }

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="6" class="p-6 text-center text-gray-400">No food items found.</td></tr>`;
                return;
            }

            tbody.innerHTML = filtered.map(item => {
                const isSelected = state.selectedItemIds.includes(item.id);
                const tagObj = state.tagsList.find(t => t.name === item.tag) || { bg: '#e5e7eb', text: '#374151' };

                return `
                    <tr class="${isSelected ? 'selected-row' : ''} hover:bg-gray-50/50 dark:hover:bg-gray-800/50 transition">
                        <td class="p-3 text-center">
                            <input type="checkbox" ${isSelected ? 'checked' : ''} onchange="toggleSelectRow('${item.id}')" class="w-4 h-4 accent-amber-500 rounded cursor-pointer">
                        </td>
                        <td class="p-3 font-semibold">
                            <div class="flex items-center gap-1.5" id="nameContainer_${item.id}">
                                <span id="nameText_${item.id}" class="item-name-text">${escapeHtml(item.name)}</span>
                                <button onclick="enableInlineNameEdit('${item.id}')" class="text-gray-400 hover:text-amber-600 text-xs p-0.5" title="Inline Edit Name">✏️</button>
                            </div>
                        </td>
                        <td class="p-3 text-center">
                            <div class="inline-flex items-center border border-gray-200 dark:border-gray-700 rounded-lg overflow-hidden">
                                <button onclick="adjustQuantity('${item.id}', 'needed', -1)" class="px-2 py-0.5 hover:bg-gray-200 dark:hover:bg-gray-700 font-bold">-</button>
                                <span class="px-3 py-0.5 font-bold min-w-[36px] text-center">${item.needed}</span>
                                <button onclick="adjustQuantity('${item.id}', 'needed', 1)" class="px-2 py-0.5 hover:bg-gray-200 dark:hover:bg-gray-700 font-bold">+</button>
                            </div>
                        </td>
                        <td class="p-3"><span class="px-2.5 py-1 bg-gray-200/80 dark:bg-gray-700/80 text-gray-800 dark:text-gray-100 font-bold font-mono text-[11px] rounded-md border border-gray-300/60 dark:border-gray-600/60 shadow-xs">${item.unit}</span></td>
                        <td class="p-3 text-center">
                            <div class="inline-flex items-center border border-gray-200 dark:border-gray-700 rounded-lg overflow-hidden">
                                <button onclick="adjustQuantity('${item.id}', 'stock', -1)" class="px-2 py-0.5 hover:bg-gray-200 dark:hover:bg-gray-700 font-bold">-</button>
                                <span class="px-3 py-0.5 font-bold min-w-[36px] text-center">${item.stock}</span>
                                <button onclick="adjustQuantity('${item.id}', 'stock', 1)" class="px-2 py-0.5 hover:bg-gray-200 dark:hover:bg-gray-700 font-bold">+</button>
                            </div>
                        </td>
                        <td class="p-3" id="tagCell_${item.id}">
                            <div class="flex items-center gap-1">
                                <span class="px-2.5 py-0.5 rounded-full font-bold text-[11px]" style="background-color: ${tagObj.bg}; color: ${tagObj.text}">${escapeHtml(item.tag)}</span>
                                <button onclick="enableInlineTagEdit('${item.id}')" class="text-gray-400 hover:text-amber-600 text-xs p-0.5" title="Change Tag">✏️</button>
                            </div>
                        </td>
                    </tr>
                `;
            }).join('');
        }

        function adjustQuantity(id, field, delta) {
            const item = state.items.find(i => i.id === id);
            if (!item) return;

            const isDecimalUnit = item.unit === 'Kg' || item.unit === 'L';
            const step = isDecimalUnit ? 0.5 : 1;
            let val = (field === 'needed' ? item.needed : item.stock) + (delta * step);

            if (val < 0) val = 0;
            if (!isDecimalUnit) val = Math.round(val);

            if (field === 'needed') item.needed = val;
            else item.stock = val;

            saveStateLocally();
            renderTable();
            renderTagTabs();
        }

        function toggleSelectRow(id) {
            const index = state.selectedItemIds.indexOf(id);
            if (index > -1) state.selectedItemIds.splice(index, 1);
            else state.selectedItemIds.push(id);

            updateButtonStates();
            renderTable();
        }

        function updateButtonStates() {
            const modifyBtn = document.getElementById('modifyBtn');
            const deleteBtn = document.getElementById('deleteBtn');

            if (modifyBtn) {
                if (state.selectedItemIds.length === 1) {
                    modifyBtn.disabled = false;
                    modifyBtn.className = "px-4 py-2 btn-accent rounded-xl text-xs font-bold transition shadow-sm flex items-center gap-1.5 cursor-pointer";
                } else {
                    modifyBtn.disabled = true;
                    modifyBtn.className = "px-4 py-2 bg-gray-200 dark:bg-gray-800 text-gray-400 rounded-xl text-xs font-bold transition cursor-not-allowed flex items-center gap-1.5";
                }
            }

            if (deleteBtn) {
                if (state.selectedItemIds.length >= 1) {
                    deleteBtn.disabled = false;
                    deleteBtn.className = "px-4 py-2 bg-red-600 hover:bg-red-700 text-white rounded-xl text-xs font-bold transition shadow-sm flex items-center gap-1.5 cursor-pointer";
                } else {
                    deleteBtn.disabled = true;
                    deleteBtn.className = "px-4 py-2 bg-gray-200 dark:bg-gray-800 text-gray-400 rounded-xl text-xs font-bold transition cursor-not-allowed flex items-center gap-1.5";
                }
            }
        }

        function enableInlineNameEdit(id) {
            const container = document.getElementById(`nameContainer_${id}`);
            const item = state.items.find(i => i.id === id);
            if (!container || !item) return;

            container.innerHTML = `
                <input type="text" id="inlineInput_${id}" value="${escapeHtml(item.name)}" 
                       onblur="saveInlineName('${id}')" onkeydown="if(event.key==='Enter') saveInlineName('${id}')"
                       class="p-1 text-xs rounded border border-amber-500 bg-transparent focus:outline-none dark:bg-gray-800">
            `;
            document.getElementById(`inlineInput_${id}`)?.focus();
        }

        function saveInlineName(id) {
            const input = document.getElementById(`inlineInput_${id}`);
            const item = state.items.find(i => i.id === id);
            if (input && item && input.value.trim() !== '') {
                item.name = input.value.trim();
                saveStateLocally();
            }
            renderTable();
        }

        function enableInlineTagEdit(id) {
            const tagCell = document.getElementById(`tagCell_${id}`);
            const item = state.items.find(i => i.id === id);
            if (!tagCell || !item) return;

            const options = state.tagsList.map(t => `<option value="${t.name}" ${t.name === item.tag ? 'selected' : ''}>${t.name}</option>`).join('');
            tagCell.innerHTML = `
                <select onchange="saveInlineTag('${id}', this.value)" class="text-xs p-1 rounded-lg border border-amber-500 bg-transparent focus:outline-none dark:bg-gray-800">
                    ${options}
                </select>
            `;
        }

        function saveInlineTag(id, newTag) {
            const item = state.items.find(i => i.id === id);
            if (item) {
                item.tag = newTag;
                saveStateLocally();
                renderTable();
                renderTagTabs();
            }
        }

        function populateDropdowns() {
            const unitSelect = document.getElementById('modalUnit');
            const tagSelect = document.getElementById('modalTag');

            if (unitSelect) {
                unitSelect.innerHTML = state.units.map(u => `<option value="${u.symbol}">${u.symbol} (${u.name})</option>`).join('');
            }
            if (tagSelect) {
                tagSelect.innerHTML = state.tagsList.map(t => `<option value="${t.name}">${t.name}</option>`).join('');
            }
        }

        function openAddModal() {
            state.editingItemId = null;
            document.getElementById('itemModalTitle').innerText = 'Add New Food Item';
            document.getElementById('itemForm').reset();
            populateDropdowns();
            openModal('itemModal');
        }

        function openModifyModal() {
            if (state.selectedItemIds.length !== 1) return;
            const item = state.items.find(i => i.id === state.selectedItemIds[0]);
            if (!item) return;

            state.editingItemId = item.id;
            document.getElementById('itemModalTitle').innerText = 'Modify Food Item';
            populateDropdowns();

            document.getElementById('modalName').value = item.name;
            document.getElementById('modalNeeded').value = item.needed;
            document.getElementById('modalUnit').value = item.unit;
            document.getElementById('modalStock').value = item.stock;
            document.getElementById('modalTag').value = item.tag;

            openModal('itemModal');
        }

        function saveItem(event) {
            event.preventDefault();
            const name = document.getElementById('modalName').value.trim();
            const needed = parseFloat(document.getElementById('modalNeeded').value);
            const unit = document.getElementById('modalUnit').value;
            const stock = parseFloat(document.getElementById('modalStock').value || '0');
            const tag = document.getElementById('modalTag').value;

            if (!name || isNaN(needed) || !unit || !tag) return;

            if (state.editingItemId) {
                const item = state.items.find(i => i.id === state.editingItemId);
                if (item) {
                    item.name = name;
                    item.needed = needed;
                    item.unit = unit;
                    item.stock = stock;
                    item.tag = tag;
                }
            } else {
                state.items.push({
                    id: Date.now().toString(),
                    name, needed, unit, stock, tag
                });
            }

            saveStateLocally();
            closeModal('itemModal');
            renderTable();
            renderTagTabs();
        }

        function openDeleteModal() {
            if (state.selectedItemIds.length === 0) return;
            const list = document.getElementById('deleteListSummary');
            if (list) {
                list.innerHTML = state.selectedItemIds.map(id => {
                    const item = state.items.find(i => i.id === id);
                    return item ? `<li class="flex justify-between"><span>${escapeHtml(item.name)}</span><span class="text-gray-400">${item.tag}</span></li>` : '';
                }).join('');
            }
            openModal('deleteModal');
        }

        function confirmDelete() {
            state.items = state.items.filter(i => !state.selectedItemIds.includes(i.id));
            state.selectedItemIds = [];
            saveStateLocally();
            closeModal('deleteModal');
            updateButtonStates();
            renderTable();
            renderTagTabs();
        }

        function renderGroceryList() {
            const filterContainer = document.getElementById('groceryTagFilters');
            const itemsContainer = document.getElementById('groceryItemsList');

            if (filterContainer) {
                filterContainer.innerHTML = state.tagsList.map(t => {
                    const isChecked = state.groceryActiveTags[t.name] === true;
                    return `
                        <label class="inline-flex items-center gap-1.5 px-3 py-1 rounded-xl text-xs font-bold border cursor-pointer transition" style="background-color: ${isChecked ? t.bg : 'transparent'}; color: ${isChecked ? t.text : 'currentColor'}">
                            <input type="checkbox" ${isChecked ? 'checked' : ''} onchange="toggleGroceryTag('${t.name}')" class="accent-amber-500 rounded">
                            <span>${t.name}</span>
                        </label>
                    `;
                }).join('');
            }

            if (itemsContainer) {
                const neededItems = state.items.filter(i => {
                    const deficit = i.needed - i.stock;
                    const tagActive = state.groceryActiveTags[i.tag] === true;
                    return deficit > 0 && tagActive;
                });

                if (neededItems.length === 0) {
                    itemsContainer.innerHTML = `<div class="p-4 text-center text-xs text-gray-400">No grocery items needed for selected tags!</div>`;
                    return;
                }

                itemsContainer.innerHTML = neededItems.map(i => {
                    const deficit = i.needed - i.stock;
                    return `
                        <div class="flex items-center justify-between p-3 rounded-xl custom-card border border-gray-200 dark:border-gray-700/80 shadow-xs hover:border-amber-400/50 transition-all">
                            <label class="flex items-center gap-2.5 cursor-pointer flex-1">
                                <input type="checkbox" class="w-4 h-4 accent-amber-500 rounded cursor-pointer">
                                <span class="text-xs font-bold item-name-text">${escapeHtml(i.name)}</span>
                            </label>
                            <span class="text-xs font-extrabold text-amber-900 dark:text-amber-100 bg-amber-100 dark:bg-amber-900/60 px-2.5 py-1 rounded-lg border border-amber-300/60 dark:border-amber-700/60 shadow-xs">
                                Need: ${deficit} ${i.unit}
                            </span>
                        </div>
                    `;
                }).join('');
            }
        }

        function toggleGroceryTag(tagName) {
            state.groceryActiveTags[tagName] = !state.groceryActiveTags[tagName];
            renderGroceryList();
        }

        function copyGroceryList(evt) {
            const neededItems = state.items.filter(i => {
                const deficit = i.needed - i.stock;
                const tagActive = state.groceryActiveTags[i.tag] === true;
                return deficit > 0 && tagActive;
            });

            if (neededItems.length === 0) return;

            const listText = neededItems.map(i => `• ${i.name}: ${i.needed - i.stock} ${i.unit}`).join('\n');
            
            const textarea = document.createElement('textarea');
            textarea.value = listText;
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand('copy');
            document.body.removeChild(textarea);

            const btn = (evt && evt.target) ? evt.target : document.activeElement;
            if (btn && btn.tagName === 'BUTTON') {
                const origText = btn.innerText;
                btn.innerText = '✓ Copied!';
                setTimeout(() => { btn.innerText = origText; }, 2000);
            }
        }

        function openUnitsModal() {
            renderUnitsList();
            openModal('unitsModal');
        }

        function renderUnitsList() {
            const container = document.getElementById('unitsListContainer');
            if (!container) return;

            container.innerHTML = state.units.map(u => `
                <div class="flex items-center justify-between p-2 rounded-xl border border-gray-200 dark:border-gray-700 text-xs">
                    <div>
                        <span class="font-bold">${escapeHtml(u.symbol)}</span>
                        <span class="text-gray-400 text-[11px] ml-1">(${escapeHtml(u.name)})</span>
                    </div>
                    ${u.isDefault ? '<span class="text-[10px] text-gray-400">Default</span>' : `
                        <button onclick="removeUnit('${u.symbol}')" class="text-red-500 hover:text-red-700 text-xs font-bold">🗑️</button>
                    `}
                </div>
            `).join('');
        }

        function addCustomUnit() {
            const symbol = document.getElementById('newUnitSymbol').value.trim();
            const name = document.getElementById('newUnitName').value.trim();

            if (!symbol || !name || symbol.length > 3 || name.length > 10) return;
            if (state.units.some(u => u.symbol.toLowerCase() === symbol.toLowerCase())) return;

            state.units.push({ symbol, name, isDefault: false });
            document.getElementById('newUnitSymbol').value = '';
            document.getElementById('newUnitName').value = '';
            saveStateLocally();
            renderUnitsList();
            renderTable();
        }

        function removeUnit(symbol) {
            state.units = state.units.filter(u => u.symbol !== symbol);
            saveStateLocally();
            renderUnitsList();
            renderTable();
        }

        function openTagsModal() {
            renderTagsManagerList();
            openModal('tagsModal');
        }

        function renderTagsManagerList() {
            const container = document.getElementById('tagsListContainer');
            if (!container) return;

            container.innerHTML = state.tagsList.map(t => `
                <div class="flex items-center justify-between p-2 rounded-xl border border-gray-200 dark:border-gray-700 text-xs">
                    <span class="px-2 py-0.5 rounded-full font-bold" style="background-color: ${t.bg}; color: ${t.text}">${t.name}</span>
                    <div class="flex items-center gap-2">
                        <input type="color" value="${t.bg}" onchange="updateTagColor('${t.name}', 'bg', this.value)" class="w-6 h-6 rounded cursor-pointer" title="Background Color">
                        <input type="color" value="${t.text}" onchange="updateTagColor('${t.name}', 'text', this.value)" class="w-6 h-6 rounded cursor-pointer" title="Text Color">
                        ${t.isDefault ? '<span class="text-[10px] text-gray-400">Default</span>' : `
                            <button onclick="removeTag('${t.name}')" class="text-red-500 hover:text-red-700 text-xs font-bold">🗑️</button>
                        `}
                    </div>
                </div>
            `).join('');
        }

        function updateTagColor(tagName, field, color) {
            const tag = state.tagsList.find(t => t.name === tagName);
            if (tag) {
                tag[field] = color;
                saveStateLocally();
                renderTagsManagerList();
                renderTagTabs();
                renderTable();
            }
        }

        function addCustomTag() {
            const name = document.getElementById('newTagName').value.trim();
            const bg = document.getElementById('newTagBg').value;
            const text = document.getElementById('newTagText').value;

            if (!name || state.tagsList.some(t => t.name.toLowerCase() === name.toLowerCase())) return;

            state.tagsList.push({ id: name, name, bg, text, isDefault: false });
            document.getElementById('newTagName').value = '';
            saveStateLocally();
            renderTagsManagerList();
            renderTagTabs();
            renderTable();
        }

        function removeTag(tagName) {
            state.tagsList = state.tagsList.filter(t => t.name !== tagName);
            saveStateLocally();
            renderTagsManagerList();
            renderTagTabs();
            renderTable();
        }

        function toggleDarkMode() {
            document.body.classList.toggle('dark-mode');
            const btn = document.getElementById('darkModeBtn');
            if (btn) btn.innerText = document.body.classList.contains('dark-mode') ? '☀️' : '🌙';
        }

        function applyPresetTheme(preset) {
            const themes = {
                amber: { bg: '#fdfbf7', card: '#ffffff', itemText: '#111827', accent: '#d97706', select: '#fef3c7' },
                mint: { bg: '#f0fdf4', card: '#ffffff', itemText: '#065f46', accent: '#059669', select: '#d1fae5' },
                rose: { bg: '#fff1f2', card: '#ffffff', itemText: '#9f1239', accent: '#e11d48', select: '#ffe4e6' },
                lavender: { bg: '#f5f3ff', card: '#ffffff', itemText: '#4c1d95', accent: '#7c3aed', select: '#ede9fe' },
                peach: { bg: '#fff7ed', card: '#ffffff', itemText: '#9a3412', accent: '#ea580c', select: '#ffedd5' }
            };

            const t = themes[preset];
            if (!t) return;

            document.getElementById('themeAppBg').value = t.bg;
            document.getElementById('themeCardBg').value = t.card;
            if (document.getElementById('themeItemText')) document.getElementById('themeItemText').value = t.itemText;
            document.getElementById('themeAccent').value = t.accent;
            document.getElementById('themeRowSelect').value = t.select;

            applyCustomColorsFromPickers();
        }

        function adjustColorBrightness(hex, percent) {
            let num = parseInt(hex.replace('#', ''), 16),
                amt = Math.round(2.55 * percent),
                R = (num >> 16) + amt,
                G = (num >> 8 & 0x00FF) + amt,
                B = (num & 0x0000FF) + amt;
            return '#' + (0x1000000 + (R < 255 ? R < 1 ? 0 : R : 255) * 0x10000 + (G < 255 ? G < 1 ? 0 : G : 255) * 0x100 + (B < 255 ? B < 1 ? 0 : B : 255)).toString(16).slice(1);
        }

        function applyCustomColorsFromPickers() {
            const r = document.documentElement;
            const accent = document.getElementById('themeAccent').value;
            r.style.setProperty('--app-bg', document.getElementById('themeAppBg').value);
            r.style.setProperty('--card-bg', document.getElementById('themeCardBg').value);
            const itemTextEl = document.getElementById('themeItemText');
            if (itemTextEl) r.style.setProperty('--item-text-color', itemTextEl.value);
            r.style.setProperty('--accent-color', accent);
            r.style.setProperty('--accent-hover', adjustColorBrightness(accent, -15));
            r.style.setProperty('--row-select-bg', document.getElementById('themeRowSelect').value);
        }

        function parseFirebaseConfigInput(raw) {
            if (!raw || !raw.trim()) throw new Error("Empty configuration snippet.");
            
            // Try standard JSON parse first
            try {
                return JSON.parse(raw);
            } catch (e) {
                // Handle raw JavaScript code snippets (e.g. const firebaseConfig = { ... };)
                const start = raw.indexOf('{');
                const end = raw.lastIndexOf('}');
                if (start === -1 || end === -1 || end <= start) {
                    throw new Error("No valid configuration object found. Paste the full firebaseConfig snippet from Firebase Console.");
                }

                const objStr = raw.substring(start, end + 1);
                // Use Function constructor to safely evaluate object literal notation
                const evaluator = new Function(`"use strict"; return (${objStr});`);
                const parsed = evaluator();

                if (parsed && typeof parsed === 'object' && parsed.apiKey && parsed.projectId) {
                    return parsed;
                } else {
                    throw new Error("Parsed object is missing required fields like 'apiKey' or 'projectId'.");
                }
            }
        }

        function saveFirebaseConfig() {
            const raw = document.getElementById('firebaseConfigInput').value;
            const errBox = document.getElementById('cloudErrorBox');
            if (errBox) errBox.classList.add('hidden');

            try {
                const configObj = parseFirebaseConfigInput(raw);
                localStorage.setItem('pantry_firebase_config', JSON.stringify(configObj));
                initFirebaseSync(configObj);
                closeModal('cloudModal');
            } catch (err) {
                if (errBox) {
                    errBox.innerText = "Firebase Config Error:\n" + err.message;
                    errBox.classList.remove('hidden');
                }
            }
        }

        function initFirebaseSync(config) {
            try {
                if (!firebase.apps.length) firebase.initializeApp(config);
                db = firebase.firestore();

                try {
                    firebase.auth().signInAnonymously().catch(e => console.warn("Firebase Auth Notice:", e.message));
                } catch (e) { }

                db.collection("pantry").doc("shared_state").onSnapshot(doc => {
                    if (doc.exists) {
                        const data = doc.data();
                        if (data.items) state.items = data.items;
                        if (data.units) state.units = data.units;
                        if (data.tagsList) state.tagsList = data.tagsList;

                        renderTagTabs();
                        renderTable();
                        if (state.activeWindow === 'grocery') renderGroceryList();
                    }
                }, err => {
                    const errBox = document.getElementById('cloudErrorBox');
                    if (errBox) {
                        errBox.innerText = "Firestore Connection Error: " + err.message;
                        errBox.classList.remove('hidden');
                    }
                });

                const badge = document.getElementById('syncStatusBadge');
                if (badge) {
                    badge.innerHTML = `<span class="w-2 h-2 mr-1.5 rounded-full bg-green-500"></span> ☁️ Live Synced`;
                    badge.className = "inline-flex items-center px-3 py-1.5 rounded-xl text-xs font-medium bg-green-100 text-green-800 border border-green-300";
                }
            } catch (err) {
                console.error("Firebase Init Error:", err);
            }
        }

        function pushStateToCloud() {
            if (!db) return;
            db.collection("pantry").doc("shared_state").set({
                items: state.items,
                units: state.units,
                tagsList: state.tagsList,
                updatedAt: firebase.firestore.FieldValue.serverTimestamp()
            }, { merge: true }).catch(err => console.error("Cloud push error:", err));
        }

        function disconnectFirebase() {
            localStorage.removeItem('pantry_firebase_config');
            db = null;
            const badge = document.getElementById('syncStatusBadge');
            if (badge) {
                badge.innerHTML = `<span class="w-2 h-2 mr-1.5 rounded-full bg-gray-400"></span> ☁️ Cloud Sync`;
                badge.className = "inline-flex items-center px-3 py-1.5 rounded-xl text-xs font-medium border border-gray-300 dark:border-gray-700 hover:bg-gray-100 dark:hover:bg-gray-800 transition";
            }
            closeModal('cloudModal');
        }

        function escapeHtml(str) {
            return String(str).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');
        }

        // Initialize application on DOM ready
        window.addEventListener('DOMContentLoaded', initApp);
    </script>
</body>
</html>
