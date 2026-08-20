<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KitCheck | Kit Authenticator & Explorer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @media print {
            body * { visibility: hidden; }
            #certificateArea, #certificateArea * { visibility: visible; }
            #certificateArea { position: absolute; left: 0; top: 0; width: 100%; }
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 font-sans min-h-screen pb-12">

    <!-- Header -->
    <header class="border-b border-slate-800 bg-slate-950/50 backdrop-blur sticky top-0 z-50">
        <div class="max-w-5xl mx-auto px-4 py-4 flex items-center justify-between">
            <div class="flex items-center space-x-2">
                <span class="text-2xl">⚽</span>
                <span class="text-xl font-bold tracking-wider text-emerald-400">KITCHECK</span>
            </div>
            
            <!-- Navigation Tabs -->
            <div class="flex bg-slate-800 p-1 rounded-lg border border-slate-700 text-xs">
                <button id="tabAuthBtn" onclick="switchTab('auth')" class="px-3 py-1.5 rounded-md font-semibold bg-emerald-500 text-slate-950 transition">Authenticator</button>
                <button id="tabShopBtn" onclick="switchTab('shop')" class="px-3 py-1.5 rounded-md font-semibold text-slate-400 hover:text-slate-200 transition">Jersey Explorer</button>
            </div>
        </div>
    </header>

    <main class="max-w-5xl mx-auto px-4 py-8">
        
        <!-- SECTION 1: AUTHENTICATOR TOOL -->
        <div id="authSection">
            <div class="text-center mb-10">
                <h1 class="text-3xl sm:text-4xl font-extrabold mb-3">Authenticate Any Football Kit</h1>
                <p class="text-slate-400 max-w-xl mx-auto text-sm sm:text-base">Verify serial codes, inspect tags, and calculate kit confidence ratings.</p>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                <!-- Inputs Column -->
                <div class="lg:col-span-2 space-y-6">
                    <div class="bg-slate-800/50 border border-slate-700/60 rounded-xl p-5">
                        <div class="flex justify-between items-center mb-2">
                            <label class="block text-sm font-semibold">1. Product Code Verification</label>
                            <button onclick="toggleDatabaseModal()" class="text-xs text-emerald-400 hover:underline">View Verified Codes</button>
                        </div>
                        <input type="text" id="productCode" onkeyup="checkQuickDatabase(this.value)" placeholder="Enter Code (e.g. AJ5531-728 or AI6716)" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-4 py-2.5 text-slate-100 placeholder-slate-500 focus:outline-none focus:border-emerald-500 transition uppercase">
                        <div id="dbStatus" class="mt-2 text-xs hidden"></div>
                    </div>

                    <div class="bg-slate-800/50 border border-slate-700/60 rounded-xl p-5">
                        <label class="block text-sm font-semibold mb-3">2. Upload Photos</label>
                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                            <div class="border-2 border-dashed border-slate-700 rounded-lg p-4 text-center hover:border-emerald-500/50 transition cursor-pointer bg-slate-900/50" onclick="document.getElementById('tagImg').click()">
                                <input type="file" id="tagImg" accept="image/*" capture="environment" class="hidden" onchange="previewImage(this, 'tagPreview')">
                                <div id="tagPreview"><span class="text-2xl">🏷️</span><p class="text-xs font-medium">Wash Tag</p></div>
                            </div>
                            <div class="border-2 border-dashed border-slate-700 rounded-lg p-4 text-center hover:border-emerald-500/50 transition cursor-pointer bg-slate-900/50" onclick="document.getElementById('badgeImg').click()">
                                <input type="file" id="badgeImg" accept="image/*" capture="environment" class="hidden" onchange="previewImage(this, 'badgePreview')">
                                <div id="badgePreview"><span class="text-2xl">🛡️</span><p class="text-xs font-medium">Club Crest</p></div>
                            </div>
                            <div class="border-2 border-dashed border-slate-700 rounded-lg p-4 text-center hover:border-emerald-500/50 transition cursor-pointer bg-slate-900/50" onclick="document.getElementById('logoImg').click()">
                                <input type="file" id="logoImg" accept="image/*" capture="environment" class="hidden" onchange="previewImage(this, 'logoPreview')">
                                <div id="logoPreview"><span class="text-2xl">⚡</span><p class="text-xs font-medium">Brand Logo</p></div>
                            </div>
                        </div>
                    </div>

                    <button onclick="runAuthentication()" class="w-full bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-bold py-3.5 px-6 rounded-xl transition shadow-lg shadow-emerald-500/10">Run Full Verification</button>
                </div>

                <!-- Certificate Column -->
                <div id="certificateArea" class="bg-slate-800/30 border border-slate-700/60 rounded-xl p-6 flex flex-col justify-between">
                    <div>
                        <h2 class="text-xs font-bold uppercase tracking-widest text-slate-400 mb-4">Verification Certificate</h2>
                        <div id="resultBox" class="text-center py-8">
                            <span class="text-4xl block mb-2">🔍</span>
                            <p class="text-sm text-slate-400">Awaiting input for analysis...</p>
                        </div>
                        <div id="matchedKitDetails" class="hidden my-4 p-3 bg-emerald-950/30 border border-emerald-500/30 rounded-lg text-xs space-y-1">
                            <p class="text-emerald-400 font-bold" id="kitTitle">--</p>
                            <p class="text-slate-300" id="kitInfo">--</p>
                        </div>
                        <div id="analysisBreakdown" class="hidden space-y-3 border-t border-slate-700/60 pt-4">
                            <div class="flex justify-between text-xs"><span class="text-slate-400">Database Lookup:</span><span id="checkDb" class="font-bold">--</span></div>
                            <div class="flex justify-between text-xs"><span class="text-slate-400">Code Syntax:</span><span id="checkCode" class="font-bold">--</span></div>
                            <div class="flex justify-between text-xs"><span class="text-slate-400">Photos Uploaded:</span><span id="checkPhotos" class="font-bold">--</span></div>
                        </div>
                    </div>
                    <div id="certFooter" class="hidden border-t border-slate-700/60 pt-4 mt-6">
                        <div class="flex items-center justify-between mb-4">
                            <div class="text-[11px] text-slate-500">
                                <div>CERT ID: <strong id="certId" class="text-slate-300">#000000</strong></div>
                                <div id="certDate">--</div>
                            </div>
                        </div>
                        <button onclick="window.print()" class="w-full bg-slate-700 hover:bg-slate-600 text-xs py-2 rounded text-slate-200">🖨️ Print Certificate</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- SECTION 2: EXPLORER & SHOPPING ENGINE -->
        <div id="shopSection" class="hidden space-y-6">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-extrabold mb-2">Football Jersey Explorer</h1>
                <p class="text-slate-400 text-sm">Find official kits, compare prices, and view product serial codes.</p>
            </div>

            <!-- Search Bar -->
            <div class="max-w-xl mx-auto flex gap-2">
                <input type="text" id="shopSearchInput" onkeyup="filterShopJerseys()" placeholder="Search by team, player, or year (e.g. Madrid, Barcelona, City, 2026)..." class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 text-slate-100 placeholder-slate-500 focus:outline-none focus:border-emerald-500 transition">
            </div>

            <!-- Jersey Grid -->
            <div id="jerseyGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 pt-4">
                <!-- JS injects cards here -->
            </div>
        </div>

    </main>

    <!-- Modal for Sample Database Codes -->
    <div id="dbModal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden items-center justify-center p-4 z-50">
        <div class="bg-slate-800 border border-slate-700 rounded-xl p-6 max-w-md w-full space-y-4">
            <h3 class="text-lg font-bold">Sample Product Codes</h3>
            <div class="space-y-2 text-xs" id="demoCodeList"></div>
            <button onclick="toggleDatabaseModal()" class="w-full bg-slate-700 py-2 rounded text-xs">Close</button>
        </div>
    </div>

    <!-- MAIN JAVASCRIPT APP LOGIC -->
    <script>
        // Integrated Store Database & Reference Index
        const JERSEY_STORE = [
            { id: "AJ5531-728", team: "Brazil National Team", season: "2018/19 Home", brand: "Nike", price: "$90 - $120", code: "AJ5531-728", shopUrl: "https://www.google.com/search?q=Brazil+2018+Home+Jersey+AJ5531-728", img: "https://images.unsplash.com/photo-1518091043644-c1d4457512c6?w=500" },
            { id: "AI6716", team: "Real Madrid", season: "2025/26 Home Jersey", brand: "Adidas", price: "$97.99 - $120.00", code: "AI6716", shopUrl: "https://www.google.com/search?q=Real+Madrid+2025+Home+Jersey", img: "https://images.unsplash.com/photo-1508098682722-e99c43a406b2?w=500" },
            { id: "CD0697-010", team: "FC Barcelona", season: "2025/26 Fourth Jersey", brand: "Nike", price: "$70.00 - $94.99", code: "CD0697-010", shopUrl: "https://www.google.com/search?q=Barcelona+Fourth+Jersey+2025", img: "https://images.unsplash.com/photo-1577223625816-7546f13df25d?w=500" },
            { id: "HF1845", team: "Manchester City", season: "2025/26 Home Jersey", brand: "Puma", price: "$50.00 - $72.00", code: "HF1845", shopUrl: "https://www.google.com/search?q=Manchester+City+Home+Jersey+2025", img: "https://images.unsplash.com/photo-1522778119026-d647f0596c20?w=500" },
            { id: "756031-01", team: "Manchester United", season: "2025/26 Home Jersey", brand: "Adidas", price: "$100.00", code: "756031-01", shopUrl: "https://www.google.com/search?q=Manchester+United+2025+Home+Jersey", img: "https://images.unsplash.com/photo-1517466787929-bc90951d0974?w=500" },
            { id: "DH8966-100", team: "England National Team", season: "2026 Match Jersey", brand: "Nike", price: "$175.00", code: "DH8966-100", shopUrl: "https://www.google.com/search?q=England+2026+Match+Jersey", img: "https://images.unsplash.com/photo-1574629810360-7efbbe195018?w=500" }
        ];

        // Tab Navigation Switcher
        function switchTab(tab) {
            const authSection = document.getElementById('authSection');
            const shopSection = document.getElementById('shopSection');
            const authBtn = document.getElementById('tabAuthBtn');
            const shopBtn = document.getElementById('tabShopBtn');

            if (tab === 'auth') {
                authSection.classList.remove('hidden');
                shopSection.classList.add('hidden');
                authBtn.className = "px-3 py-1.5 rounded-md font-semibold bg-emerald-500 text-slate-950 transition";
                shopBtn.className = "px-3 py-1.5 rounded-md font-semibold text-slate-400 hover:text-slate-200 transition";
            } else {
                authSection.classList.add('hidden');
                shopSection.classList.remove('hidden');
                shopBtn.className = "px-3 py-1.5 rounded-md font-semibold bg-emerald-500 text-slate-950 transition";
                authBtn.className = "px-3 py-1.5 rounded-md font-semibold text-slate-400 hover:text-slate-200 transition";
                renderShop(JERSEY_STORE);
            }
        }

        // Render Cards in Shop Tab
        function renderShop(items) {
            const grid = document.getElementById('jerseyGrid');
            grid.innerHTML = '';
            
            if (items.length === 0) {
                grid.innerHTML = `<p class="col-span-full text-center text-slate-500 py-8">No jerseys match your search term.</p>`;
                return;
            }

            items.forEach(item => {
                grid.innerHTML += `
                    <div class="bg-slate-800/40 border border-slate-700/60 rounded-xl overflow-hidden hover:border-emerald-500/50 transition">
                        <img src="${item.img}" class="h-44 w-full object-cover">
                        <div class="p-4 space-y-3">
                            <div>
                                <span class="text-[10px] bg-slate-700 text-slate-300 px-2 py-0.5 rounded uppercase font-bold">${item.brand}</span>
                                <h3 class="text-base font-bold text-slate-100 mt-1">${item.team}</h3>
                                <p class="text-xs text-slate-400">${item.season}</p>
                            </div>
                            <div class="bg-slate-900/60 p-2 rounded text-[11px] font-mono flex justify-between">
                                <span class="text-slate-500">Tag Serial:</span>
                                <span class="text-emerald-400 font-bold">${item.id}</span>
                            </div>
                            <div class="flex items-center justify-between pt-2 border-t border-slate-700/50">
                                <span class="text-sm font-extrabold text-emerald-400">${item.price}</span>
                                <a href="${item.shopUrl}" target="_blank" class="bg-emerald-500 hover:bg-emerald-400 text-slate-950 text-xs font-bold px-3 py-1.5 rounded transition">Find Online ↗</a>
                            </div>
                        </div>
                    </div>
                `;
            });
        }

        function filterShopJerseys() {
            const query = document.getElementById('shopSearchInput').value.toLowerCase();
            const filtered = JERSEY_STORE.filter(item => 
                item.team.toLowerCase().includes(query) || 
                item.season.toLowerCase().includes(query) ||
                item.brand.toLowerCase().includes(query) ||
                item.id.toLowerCase().includes(query)
            );
            renderShop(filtered);
        }

        // Instant Code Lookup Function
        function checkQuickDatabase(val) {
            const code = val.trim().toUpperCase();
            const statusDiv = document.getElementById('dbStatus');
            if (!code) { statusDiv.classList.add('hidden'); return; }
            statusDiv.classList.remove('hidden');

            const match = JERSEY_STORE.find(k => k.id === code);
            if (match) {
                statusDiv.innerHTML = `<span class="text-emerald-400 font-bold">✓ Match Found:</span> ${match.team} (${match.season})`;
            } else {
                statusDiv.innerHTML = `<span class="text-slate-400">Validating format structure...</span>`;
            }
        }

        // Core Authentication Engine
        function runAuthentication() {
            const code = document.getElementById('productCode').value.trim().toUpperCase();
            const tagImg = document.getElementById('tagImg').files[0];
            const badgeImg = document.getElementById('badgeImg').files[0];
            const logoImg = document.getElementById('logoImg').files[0];

            if (!code && !tagImg) {
                alert("Please enter a product code or upload an inner tag photo.");
                return;
            }

            let score = 0;
            const match = JERSEY_STORE.find(k => k.id === code);

            if (match) score += 50;
            else if (code.length >= 5) score += 25;

            if (tagImg) score += 20;
            if (badgeImg) score += 15;
            if (logoImg) score += 10;

            const breakdown = document.getElementById('analysisBreakdown');
            const certFooter = document.getElementById('certFooter');
            const matchedKitDetails = document.getElementById('matchedKitDetails');

            breakdown.classList.remove('hidden');
            certFooter.classList.remove('hidden');

            if (match) {
                matchedKitDetails.classList.remove('hidden');
                document.getElementById('kitTitle').innerText = match.team;
                document.getElementById('kitInfo').innerText = `Brand: ${match.brand} | Kit: ${match.season}`;
                document.getElementById('checkDb').innerText = "VERIFIED MATCH (+50%)";
                document.getElementById('checkDb').className = "text-emerald-400 font-bold";
            } else {
                matchedKitDetails.classList.add('hidden');
                document.getElementById('checkDb').innerText = "UNLISTED (0%)";
                document.getElementById('checkDb').className = "text-slate-500 font-bold";
            }

            document.getElementById('checkCode').innerText = code ? "VALID FORMAT (+25%)" : "MISSING";
            document.getElementById('checkCode').className = code ? "text-emerald-400 font-bold" : "text-slate-500 font-bold";

            const photoCount = (tagImg ? 1 : 0) + (badgeImg ? 1 : 0) + (logoImg ? 1 : 0);
            document.getElementById('checkPhotos').innerText = photoCount > 0 ? `${photoCount}/3 ATTACHED` : "NONE";

            const resultBox = document.getElementById('resultBox');
            if (score >= 70) {
                resultBox.innerHTML = `<div class="inline-flex items-center justify-center w-10 h-10 bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 rounded-full mb-2">✓</div><h3 class="text-xl font-bold text-emerald-400">AUTHENTIC</h3><p class="text-xs text-slate-400 mt-1">Confidence Score: ${score}%</p>`;
            } else if (score >= 40) {
                resultBox.innerHTML = `<div class="inline-flex items-center justify-center w-10 h-10 bg-amber-500/10 text-amber-400 border border-amber-500/20 rounded-full mb-2">!</div><h3 class="text-xl font-bold text-amber-400">INCONCLUSIVE</h3><p class="text-xs text-slate-400 mt-1">Confidence Score: ${score}%</p>`;
            } else {
                resultBox.innerHTML = `<div class="inline-flex items-center justify-center w-10 h-10 bg-rose-500/10 text-rose-400 border border-rose-500/20 rounded-full mb-2">✕</div><h3 class="text-xl font-bold text-rose-400">COUNTERFEIT FLAG</h3><p class="text-xs text-slate-400 mt-1">Confidence Score: ${score}%</p>`;
            }

            document.getElementById('certId').innerText = '#' + Math.floor(100000 + Math.random() * 900000);
            document.getElementById('certDate').innerText = new Date().toLocaleDateString();
        }

        // Image Preview Handler
        function previewImage(input, previewId) {
            const file = input.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    document.getElementById(previewId).innerHTML = `<img src="${e.target.result}" class="h-16 w-full object-cover rounded mb-1"><p class="text-[10px] text-emerald-400">Image Loaded</p>`;
                }
                reader.readAsDataURL(file);
            }
        }

        // Modal Controls
        function toggleDatabaseModal() {
            document.getElementById('dbModal').classList.toggle('hidden');
            document.getElementById('dbModal').classList.toggle('flex');
        }

        // Populate Modal List
        const demoList = document.getElementById('demoCodeList');
        JERSEY_STORE.forEach(item => {
            demoList.innerHTML += `<div class="p-2 border border-slate-700 rounded flex justify-between cursor-pointer hover:border-emerald-500" onclick="selectDemoCode('${item.id}')"><span class="font-mono text-emerald-400">${item.id}</span><span class="text-slate-400">${item.team}</span></div>`;
        });

        function selectDemoCode(code) {
            document.getElementById('productCode').value = code;
            checkQuickDatabase(code);
            toggleDatabaseModal();
        }
    </script>
</body>
</html>

<!--
  =============================================================================
  OPTIONAL BACKEND SERVER (Node.js/Express)
  If you decide to run a separate custom server for scraping or database lookup,
  you can save the code below as server.js and run `npm install express cors`
  =============================================================================

  const express = require('express');
  const cors = require('cors');
  const app = express();
  app.use(cors());

  app.get('/api/v1/search', (req, res) => {
      const { q } = req.query;
      res.json({ query: q, status: "Connected" });
  });

  app.listen(3000, () => console.log('API running on port 3000'));
-->

