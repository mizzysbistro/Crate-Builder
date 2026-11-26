<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Interactive 3D Crate Visualizer — Crate Designer 3D</title>

  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- Google Fonts: Inter & JetBrains Mono -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet" />

  <!-- Three.js r128 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

  <!-- QRious & jsQR (Online enhancement; built-in OfflineQR handles offline automatically) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrious/4.0.2/qrious.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>

  <!-- jsPDF & jsPDF-AutoTable for Ready-to-Print Cutting Schedule PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.31/jspdf.plugin.autotable.min.js"></script>

  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'system-ui', 'sans-serif'],
            mono: ['JetBrains Mono', 'monospace'],
          },
          colors: {
            navy: {
              50: '#f0f4f8',
              100: '#d9e2ec',
              700: '#243b53',
              800: '#1E3A5F',
              900: '#102a43',
            }
          }
        }
      }
    };
  </script>

  <style>
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }
    html, body {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      height: 100dvh;
      overflow: hidden;
      font-family: 'Inter', sans-serif;
      background-color: #0f172a;
    }
    #canvas-container {
      width: 100%;
      height: 100%;
      position: relative;
      touch-action: none;
      cursor: grab;
    }
    #canvas-container:active {
      cursor: grabbing;
    }
    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #f1f5f9;
      border-radius: 9999px;
    }
    ::-webkit-scrollbar-thumb {
      background: #cbd5e1;
      border-radius: 9999px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #94a3b8;
    }

    /* Subtle app-wide motion: gentle entrance, responsive controls, and restrained panel lift. */
    #app-shell { animation: appShellEnter 520ms cubic-bezier(.2,.7,.2,1) both; }
    @keyframes appShellEnter {
      from { opacity: 0; transform: translateY(5px); }
      to { opacity: 1; transform: translateY(0); }
    }
    button { transition: transform 150ms ease, box-shadow 180ms ease, background-color 180ms ease, color 180ms ease; }
    button:active { transform: scale(.975); }
    button:focus-visible, input:focus-visible { outline: 2px solid #38bdf8; outline-offset: 2px; }
    #control-sidebar > div { animation: panelContentEnter 300ms ease both; }
    @keyframes panelContentEnter {
      from { opacity: .72; transform: translateY(4px); }
      to { opacity: 1; transform: translateY(0); }
    }
    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after { animation-duration: .01ms !important; animation-iteration-count: 1 !important; transition-duration: .01ms !important; scroll-behavior: auto !important; }
    }
  </style>
</head>
<body class="text-slate-800 antialiased selection:bg-navy-800 selection:text-white">

  <!-- Main App Shell: Full-Bleed 3D Preview First + Slide-Over Controls Menu Drawer -->
  <div id="app-shell" class="relative w-full h-full overflow-hidden">

    <!-- Slide-Over Drawer Backdrop -->
    <div id="drawer-backdrop" onclick="toggleControlDrawer(false)"
      class="hidden fixed inset-0 bg-slate-950/55 backdrop-blur-[2px] z-30 transition-opacity"></div>

    <!-- =================================================================== -->
    <!-- SLIDE-OVER CONTROLS MENU DRAWER (Hidden by Default for Preview First)-->
    <!-- =================================================================== -->
    <aside id="control-sidebar"
      class="fixed top-0 left-0 h-full w-[92vw] max-w-[400px] bg-white border-r border-slate-200 flex flex-col shadow-2xl z-40 transform -translate-x-full transition-transform duration-300 ease-out">

      <!-- Top Navy Branding Header -->
      <div class="bg-navy-800 text-white px-4 py-3 flex items-center justify-between gap-2 shrink-0">
        <div class="flex items-center gap-2.5 min-w-0">
          <div class="w-8 h-8 rounded-lg bg-white/15 flex items-center justify-center border border-white/20 shrink-0">
            <svg class="w-5 h-5 text-sky-200" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M21 7.5l-9-5.25L3 7.5m18 0l-9 5.25m9-5.25v9l-9 5.25M3 7.5l9 5.25M3 7.5v9l9 5.25m0-9v9" />
            </svg>
          </div>
          <div class="min-w-0">
            <h1 class="text-sm sm:text-base font-bold tracking-wide leading-tight truncate">CRATE CONTROLS</h1>
            <p class="text-[10px] sm:text-[11px] text-sky-200 font-medium truncate">Dimensions, Levels &amp; Engineering</p>
          </div>
        </div>
        <div class="flex items-center gap-1.5 shrink-0">
          <span id="header-thickness-badge" class="px-2 py-1 rounded-md bg-white/15 text-xs font-mono font-semibold text-sky-100 border border-white/20">
            19 mm
          </span>
          <button type="button" id="btn-close-drawer" onclick="toggleControlDrawer(false)" title="Close Menu Drawer"
            class="p-1.5 rounded-lg bg-white/15 hover:bg-white/25 text-white border border-white/20 flex items-center justify-center transition">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.4" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>
      </div>

      <!-- Scrollable Controls Body -->
      <div id="sidebar-body" class="p-3.5 sm:p-4 overflow-y-auto flex-1 space-y-3.5">

        <!-- Validation Alert Banner (hidden unless invalid input) -->
        <div id="validation-banner" class="hidden rounded-xl bg-red-50 border border-red-300 p-3 text-xs text-red-800 font-medium"></div>

        <!-- 5. Name (Recipient) & Wood Thickness -->
        <div class="grid grid-cols-3 gap-2 items-center">
          <div class="col-span-2 bg-slate-50 border border-slate-200 rounded-xl px-3 py-1.5 focus-within:border-navy-800">
            <label for="input-recipient" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-400">Name</label>
            <input id="input-recipient" type="text" placeholder="Input Name"
              class="w-full bg-transparent text-xs font-bold uppercase tracking-wide text-slate-800 focus:outline-none mt-0.5" />
          </div>
          <div class="bg-slate-50 border border-slate-200 rounded-xl px-2.5 py-1.5 focus-within:border-navy-800">
            <label for="input-thickness" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-400">Thick (mm)</label>
            <input id="input-thickness" type="number" min="6" max="50" step="1" value="19"
              class="w-full bg-transparent font-mono text-xs font-bold text-slate-900 focus:outline-none text-center mt-0.5" />
          </div>
        </div>

        <!-- 1. Crate Levels (1-Level vs 2-Level Crate) -->
        <div>
          <label class="block text-xs font-semibold uppercase tracking-wider text-slate-500 mb-1.5">Crate Levels</label>
          <div class="grid grid-cols-2 gap-1.5 bg-slate-100 p-1 rounded-xl border border-slate-200">
            <button type="button" id="btn-levels-1" onclick="setCrateLevels(1)"
              class="py-2 px-3 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm">
              1 Level
            </button>
            <button type="button" id="btn-levels-2" onclick="setCrateLevels(2)"
              class="py-2 px-3 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900">
              2 Levels
            </button>
          </div>

          <!-- Level 2 Height & Side-Based Width Controls (Shown when 2 Levels active) -->
          <div id="level2-controls-panel" class="hidden mt-2 bg-indigo-50/90 border border-indigo-200 rounded-xl p-2.5 space-y-2">
            <div class="flex items-center justify-between gap-2">
              <div>
                <label for="input-height-l2" class="block text-[10px] font-bold uppercase tracking-wider text-indigo-900">Level 2 Side Height (Internal mm)</label>
                <span class="text-[10px] text-indigo-700 font-mono">Rear &amp; Sides: H2 • Front: H2 + T • Top 2 flush</span>
              </div>
              <input id="input-height-l2" type="number" min="100" max="10000" step="10" value="800"
                class="w-28 bg-white border border-indigo-300 rounded-lg px-2.5 py-1.5 font-mono text-xs font-bold text-slate-900 text-right focus:outline-none focus:border-navy-800" />
            </div>

            <!-- Custom Level 2 Width (Side-Based) — Locked to Default unless unlocked -->
            <div class="pt-2 border-t border-indigo-200/80 space-y-1.5">
              <div class="flex items-center justify-between gap-2">
                <div>
                  <label for="input-side-l2" class="block text-[10px] font-bold uppercase tracking-wider text-indigo-900">Level 2 Side (Shortside Base)</label>
                  <span id="l2-width-hint" class="text-[10px] text-indigo-700 font-mono">Default Side: 1182 mm (Bottom: 1201, Top 2: 1220)</span>
                </div>
                <button type="button" id="btn-unlock-l2-width" onclick="toggleL2WidthUnlock()"
                  class="px-2 py-1 rounded-lg bg-white border border-indigo-300 text-[10px] font-bold text-indigo-900 hover:bg-indigo-100 transition flex items-center gap-1 shrink-0">
                  <svg id="icon-l2-width-lock" class="w-3 h-3 text-indigo-700" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M16.5 10.5V6.75a4.5 4.5 0 10-9 0v3.75m-.75 11.25h10.5a2.25 2.25 0 002.25-2.25v-6.75a2.25 2.25 0 00-2.25-2.25H6.75a2.25 2.25 0 00-2.25 2.25v6.75a2.25 2.25 0 002.25 2.25z" />
                  </svg>
                  <span id="label-l2-width-lock">Locked: Default</span>
                </button>
              </div>
              <div class="flex items-center justify-between gap-2">
                <span class="text-[10px] font-mono text-slate-600">Side Length (mm):</span>
                <input id="input-side-l2" type="number" min="200" max="10000" step="1" value="1182" disabled
                  class="w-28 bg-slate-100 border border-slate-300 rounded-lg px-2.5 py-1 font-mono text-xs font-bold text-slate-500 text-right focus:outline-none focus:border-navy-800 disabled:cursor-not-allowed" />
              </div>
            </div>
          </div>
        </div>

        <!-- 2. Crate Type Toggle (Exterior vs Internal) -->
        <div>
          <div class="flex items-center justify-between mb-1.5">
            <label class="block text-xs font-semibold uppercase tracking-wider text-slate-500">Measurement View</label>
            <span class="text-[10px] font-mono text-emerald-700 font-semibold">Base L &times; W &times; H Reference</span>
          </div>
          <div class="grid grid-cols-2 gap-1.5 bg-slate-100 p-1 rounded-xl border border-slate-200">
            <button type="button" id="btn-exterior" onclick="setCrateType('Exterior Crate')"
              class="py-2 px-3 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm">
              Exterior Crate
            </button>
            <button type="button" id="btn-internal" onclick="setCrateType('Internal Crate')"
              class="py-2 px-3 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900">
              Internal Crate
            </button>
          </div>
          <p id="crate-type-explanation" class="text-[10px] text-slate-500 mt-1 leading-snug">
            <strong>Exterior (Outside):</strong> Base L &times; W &times; H is the outside box (2440 &times; 1220 &times; 1220 mm). Inside clear space is smaller by 2T.
          </p>
        </div>

        <!-- 3. Level 1 Dimensions Inputs (mm) -->
        <div>
          <div class="flex items-center justify-between mb-1.5">
            <label id="dim-section-label" class="text-xs font-semibold uppercase tracking-wider text-slate-500">Dimensions (mm)</label>
            <span id="dim-reference-hint" class="text-[11px] font-medium text-navy-800 bg-navy-100 px-2 py-0.5 rounded-md">
              Outside Panel Size
            </span>
          </div>
          <div class="grid grid-cols-3 gap-2">
            <div class="bg-slate-50 border border-slate-200 rounded-xl px-2.5 py-1.5 focus-within:border-navy-800">
              <label for="input-length" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-400">Length</label>
              <input id="input-length" type="number" min="200" max="10000" step="1" value="2440"
                class="w-full bg-transparent font-mono text-xs font-bold text-slate-900 focus:outline-none mt-0.5" />
            </div>
            <div class="bg-slate-50 border border-slate-200 rounded-xl px-2.5 py-1.5 focus-within:border-navy-800">
              <label for="input-width" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-400">Width</label>
              <input id="input-width" type="number" min="200" max="10000" step="1" value="1220"
                class="w-full bg-transparent font-mono text-xs font-bold text-slate-900 focus:outline-none mt-0.5" />
            </div>
            <div class="bg-slate-50 border border-slate-200 rounded-xl px-2.5 py-1.5 focus-within:border-navy-800">
              <label for="input-height" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-400">Height</label>
              <input id="input-height" type="number" min="100" max="10000" step="1" value="1220"
                class="w-full bg-transparent font-mono text-xs font-bold text-slate-900 focus:outline-none mt-0.5" />
            </div>
          </div>
          <!-- Paired Companion Readout (Shows the opposite measurement live) -->
          <div id="paired-dim-bar" class="mt-1.5 px-2.5 py-1.5 rounded-lg bg-slate-100 border border-slate-200 flex items-center justify-between text-[10px] font-mono text-slate-600">
            <span id="paired-dim-label" class="font-semibold text-slate-700">Inside Clear Space:</span>
            <span id="paired-dim-value" class="font-bold text-navy-800">2402 &times; 1182 &times; 1182 mm</span>
          </div>
        </div>

        <!-- 4. Split Longside & Internal Joint Controls (Active when Level 1 or Level 2 Wall Height > 1220 mm) -->
        <div id="split-longside-panel" class="hidden bg-sky-50/90 border border-sky-200 rounded-xl p-3 space-y-2.5">
          <div class="flex items-center justify-between">
            <span class="text-xs font-bold text-navy-900 flex items-center gap-1.5">
              <svg class="w-4 h-4 text-navy-800" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 9h16.5m-16.5 6.75h16.5" />
              </svg>
              Split Longside &amp; Internal Joint
            </span>
            <span class="font-mono text-[10px] font-bold text-sky-900 bg-sky-200/70 px-2 py-0.5 rounded">Wall &gt; 1220 mm</span>
          </div>

          <!-- Level 1 Longside Split Row -->
          <div id="l1-split-row" class="grid grid-cols-2 gap-2">
            <div class="bg-white border border-sky-200 rounded-lg px-2.5 py-1.5 focus-within:border-navy-800">
              <label for="input-long1-h" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-500">L1 Longside 1 H</label>
              <input id="input-long1-h" type="number" min="100" max="1220" step="1" value="1220"
                class="w-full bg-transparent font-mono text-xs font-bold text-navy-900 focus:outline-none mt-0.5" />
            </div>
            <div class="bg-slate-100 border border-slate-200 rounded-lg px-2.5 py-1.5">
              <label class="block text-[10px] font-semibold uppercase tracking-wider text-slate-400">L1 Longside 2 H</label>
              <div id="display-long2-h" class="font-mono text-xs font-bold text-slate-700 mt-0.5">261 mm</div>
            </div>
          </div>

          <!-- Level 2 Longside Split Row (Shown when Level 2 Wall Height > 1220 mm) -->
          <div id="l2-split-row" class="hidden grid grid-cols-2 gap-2">
            <div class="bg-white border border-indigo-200 rounded-lg px-2.5 py-1.5 focus-within:border-navy-800">
              <label for="input-l2-long1-h" class="block text-[10px] font-semibold uppercase tracking-wider text-indigo-700">L2 Longside 1 H</label>
              <input id="input-l2-long1-h" type="number" min="100" max="1220" step="1" value="1220"
                class="w-full bg-transparent font-mono text-xs font-bold text-indigo-950 focus:outline-none mt-0.5" />
            </div>
            <div class="bg-indigo-50/70 border border-indigo-200 rounded-lg px-2.5 py-1.5">
              <label class="block text-[10px] font-semibold uppercase tracking-wider text-indigo-500">L2 Longside 2 H</label>
              <div id="display-l2-long2-h" class="font-mono text-xs font-bold text-indigo-900 mt-0.5">261 mm</div>
            </div>
          </div>

          <div class="grid grid-cols-2 gap-2">
            <div class="bg-white border border-sky-200 rounded-lg px-2.5 py-1.5 focus-within:border-navy-800">
              <label for="input-joint-h" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-500">Joint Height (250–400)</label>
              <input id="input-joint-h" type="number" min="250" max="400" step="10" value="300"
                class="w-full bg-transparent font-mono text-xs font-bold text-navy-900 focus:outline-none mt-0.5" />
            </div>
            <div class="bg-white border border-sky-200 rounded-lg px-2.5 py-1.5 focus-within:border-navy-800">
              <label for="input-joint-trim" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-500">Joint L-Trim (min 40)</label>
              <input id="input-joint-trim" type="number" min="40" max="500" step="1" value="40"
                class="w-full bg-transparent font-mono text-xs font-bold text-navy-900 focus:outline-none mt-0.5" />
            </div>
          </div>
          <p id="split-longside-hint" class="text-[10px] text-slate-600 font-mono leading-tight">
            Longside 1 &ge; Longside 2 • Internal Joint L = Longside L &minus; 40 mm
          </p>
          <button type="button" id="btn-toggle-joints" onclick="toggleJointVisibility()"
            class="w-full py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-sky-300 hover:bg-sky-100/70 transition flex items-center justify-center gap-1.5">
            <span>Show Internal Joints</span>
          </button>
        </div>

        <!-- 6. Number of Base Rails -->
        <div>
          <label class="block text-xs font-semibold uppercase tracking-wider text-slate-500 mb-1.5">Number of Base Rails</label>
          <div class="grid grid-cols-2 gap-1.5 bg-slate-100 p-1 rounded-xl border border-slate-200">
            <button type="button" id="btn-rails-3" onclick="setBaseRails(3)"
              class="py-2 px-2 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm">
              3 Rails (Length)
            </button>
            <button type="button" id="btn-rails-4" onclick="setBaseRails(4)"
              class="py-2 px-2 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900">
              4 Rails (Width)
            </button>
          </div>
        </div>

        <!-- Inline Cutting Schedule (Sections 1 & 2 from Schedule Tab) -->
        <div class="bg-slate-50 border border-slate-200 rounded-xl p-3 space-y-2.5">
          <div class="flex items-center justify-between gap-2">
            <span class="text-xs font-bold uppercase tracking-wider text-navy-800">Cutting Schedule</span>
            <span id="drawer-schedule-summary" class="text-[10px] font-mono font-semibold text-slate-600 bg-white border border-slate-200 px-2 py-0.5 rounded">
              6 Panels • 45 Blocks
            </span>
          </div>

          <!-- Section 1: Wooden Panels -->
          <div>
            <div class="text-[10px] font-bold uppercase tracking-wider text-slate-500 mb-1">1. Wooden Panels (mm)</div>
            <div class="overflow-x-auto bg-white border border-slate-200 rounded-lg">
              <table class="w-full text-left border-collapse">
                <thead>
                  <tr class="bg-slate-100 border-b border-slate-200 text-[9px] font-bold uppercase tracking-wider text-slate-600">
                    <th class="py-1.5 px-2">Part</th>
                    <th class="py-1.5 px-1.5 text-right">A</th>
                    <th class="py-1.5 px-1.5 text-right">B</th>
                    <th class="py-1.5 px-2 text-right">Qty</th>
                  </tr>
                </thead>
                <tbody id="drawer-cutlist-panels-body" class="divide-y divide-slate-200 text-[11px]"></tbody>
              </table>
            </div>
          </div>

          <!-- Section 2: Base Rails & 5-Block Skid Footings -->
          <div>
            <div class="text-[10px] font-bold uppercase tracking-wider text-slate-500 mb-1">2. Base Rails &amp; Skid Footings (mm)</div>
            <div class="overflow-x-auto bg-white border border-slate-200 rounded-lg">
              <table class="w-full text-left border-collapse">
                <thead>
                  <tr class="bg-slate-100 border-b border-slate-200 text-[9px] font-bold uppercase tracking-wider text-slate-600">
                    <th class="py-1.5 px-2">Component</th>
                    <th class="py-1.5 px-1.5 text-right">W</th>
                    <th class="py-1.5 px-1.5 text-right">L</th>
                    <th class="py-1.5 px-2 text-right">Qty</th>
                  </tr>
                </thead>
                <tbody id="drawer-cutlist-base-body" class="divide-y divide-slate-200 text-[11px]"></tbody>
              </table>
            </div>
          </div>
        </div>

        <!-- 7. Exploded View & Panel Visibility -->
        <div class="bg-navy-50/70 border border-navy-100 rounded-xl p-3 space-y-2.5">
          <div class="flex items-center justify-between">
            <span class="text-xs font-bold text-navy-900">Exploded Assembly View</span>
            <span class="text-[10px] text-slate-500">Animated</span>
          </div>
          <button type="button" id="btn-exploded-view" onclick="toggleExplodedView()"
            aria-pressed="false"
            class="w-full py-2 px-3 rounded-lg text-xs font-semibold bg-navy-800 text-white hover:bg-navy-700 shadow-sm transition flex items-center justify-center gap-2">
            <span id="exploded-view-button-label">Explode View</span>
          </button>

          <div class="grid grid-cols-2 gap-1.5">
            <button type="button" id="btn-hide-top" onclick="toggleTopLidVisibility()"
              class="py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition flex items-center justify-center gap-1.5">
              <span>Hide Top Lid</span>
            </button>
            <button type="button" id="btn-hide-front" onclick="toggleFrontWallVisibility()"
              class="py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition flex items-center justify-center gap-1.5">
              <span>Hide Front Wall</span>
            </button>
          </div>
        </div>

        <!-- Toggle Button to Show/Hide Advanced Controls (Items 8–12) -->
        <div class="pt-0.5">
          <button type="button" id="btn-toggle-advanced-controls" onclick="toggleAdvancedControls()"
            class="w-full py-2.5 px-3 rounded-xl bg-slate-100 hover:bg-slate-200/80 text-navy-900 border border-slate-300 text-xs font-bold transition flex items-center justify-between">
            <span class="flex items-center gap-1.5">
              <svg class="w-4 h-4 text-navy-800" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" d="M10.5 6h9.75M10.5 6a1.5 1.5 0 11-3 0m3 0a1.5 1.5 0 10-3 0M3.75 6H7.5m3 12h9.75m-9.75 0a1.5 1.5 0 01-3 0m3 0a1.5 1.5 0 00-3 0m-3.75 0H7.5m9-6h3.75m-3.75 0a1.5 1.5 0 01-3 0m3 0a1.5 1.5 0 00-3 0m-9.75 0h9.75" />
              </svg>
              <span id="label-advanced-controls">Show More Tools &amp; Exports</span>
            </span>
            <svg id="icon-advanced-chevron" class="w-4 h-4 text-slate-600 transition-transform duration-200" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
            </svg>
          </button>
        </div>

        <!-- 8–12. Collapsible Section (Hidden by Default) -->
        <div id="advanced-controls-section" class="hidden space-y-3.5 pt-1 border-t border-slate-200">

          <!-- 8. Step-by-Step Assembly Player -->
          <div class="bg-slate-50 border border-slate-200 rounded-xl p-3 space-y-2">
            <div class="flex items-center justify-between">
              <span class="text-xs font-bold text-navy-900">Assembly Step Player</span>
              <span id="step-badge" class="font-mono text-[11px] font-bold text-navy-800 bg-navy-100 px-2 py-0.5 rounded">Step 7 / 7</span>
            </div>
            <div id="step-label" class="text-[11px] font-medium text-slate-600 truncate">7. Top — Complete Crate</div>
            <input id="slider-step" type="range" min="1" max="7" value="7"
              class="w-full accent-navy-800 cursor-pointer h-2 bg-slate-200 rounded-lg" />
            <div class="grid grid-cols-4 gap-1 pt-0.5">
              <button type="button" onclick="changeAssemblyStep(-1)" class="py-1 px-2 rounded-lg text-[11px] font-semibold bg-white border border-slate-200 hover:bg-slate-100 text-slate-700">&larr; Prev</button>
              <button type="button" id="btn-play-steps" onclick="togglePlaySteps()" class="col-span-2 py-1 px-2 rounded-lg text-[11px] font-semibold bg-navy-800 text-white hover:bg-navy-700 transition">Auto-Play Steps</button>
              <button type="button" onclick="changeAssemblyStep(1)" class="py-1 px-2 rounded-lg text-[11px] font-semibold bg-white border border-slate-200 hover:bg-slate-100 text-slate-700">Next &rarr;</button>
            </div>
            <div class="text-[10px] text-slate-500 pt-1">Built-in workshop sounds • Automatic</div>
            <div class="text-[10px] text-slate-500">Normal playback • Smooth panel installation</div>
          </div>

          <!-- 9. Visualization Mode, Dimensions & Guided Pallet-Jack Fork Clearance -->
          <div class="space-y-2">
            <div class="grid grid-cols-2 gap-1.5">
              <button type="button" id="btn-mode-solid" onclick="setRenderMode('solid')"
                class="py-1.5 px-2 rounded-lg text-xs font-semibold bg-navy-800 text-white shadow-sm transition">
                Solid Wood
              </button>
              <button type="button" id="btn-mode-xray" onclick="setRenderMode('xray')"
                class="py-1.5 px-2 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition">
                X-Ray View
              </button>
            </div>
            <div class="grid grid-cols-2 gap-1.5">
              <button type="button" id="btn-toggle-dims" onclick="toggleDimensionLines()"
                class="py-1.5 px-2 rounded-lg text-xs font-semibold bg-navy-800 text-white shadow-sm transition">
                Dimensions: ON
              </button>
              <button type="button" id="btn-toggle-forks" onclick="toggleForkOverlay()"
                class="py-1.5 px-2 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition">
                Fork Guides: OFF
              </button>
            </div>

            <!-- Guided Pallet-Jack / Forklift Clearance Controls -->
            <div id="fork-controls-panel" class="hidden bg-emerald-50/90 border border-emerald-200 rounded-xl p-2.5 space-y-2">
              <div class="flex items-center justify-between">
                <span class="text-[11px] font-bold text-emerald-950">Centered Fork Guide Settings</span>
                <span id="fork-status-badge" class="font-mono text-[10px] font-bold px-2 py-0.5 rounded bg-emerald-200 text-emerald-900">CLEAR</span>
              </div>

              <div>
                <label class="block text-[10px] font-semibold uppercase tracking-wider text-emerald-900 mb-1">Fork Entry Side (Centered)</label>
                <div class="grid grid-cols-2 gap-1 bg-white p-1 rounded-lg border border-emerald-200">
                  <button type="button" id="btn-fork-side-length" onclick="setForkEntrySide('length')"
                    class="py-1 px-2 rounded-md text-[11px] font-semibold bg-emerald-600 text-white shadow-sm transition">
                    Length Side
                  </button>
                  <button type="button" id="btn-fork-side-width" onclick="setForkEntrySide('width')"
                    class="py-1 px-2 rounded-md text-[11px] font-semibold text-slate-600 hover:text-slate-900 transition">
                    Width Side
                  </button>
                </div>
              </div>

              <div class="grid grid-cols-2 gap-2">
                <div class="bg-white border border-emerald-200 rounded-lg px-2 py-1 focus-within:border-emerald-600">
                  <label for="input-fork-width" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-500">Blade Width (mm)</label>
                  <input id="input-fork-width" type="number" min="60" max="450" step="5" value="170"
                    class="w-full bg-transparent font-mono text-xs font-bold text-slate-900 focus:outline-none mt-0.5" />
                </div>
                <div class="bg-white border border-emerald-200 rounded-lg px-2 py-1 focus-within:border-emerald-600">
                  <label for="input-fork-spread" class="block text-[10px] font-semibold uppercase tracking-wider text-slate-500">Fork Spread C-C</label>
                  <input id="input-fork-spread" type="number" min="150" max="2000" step="10" value="550"
                    class="w-full bg-transparent font-mono text-xs font-bold text-slate-900 focus:outline-none mt-0.5" />
                </div>
              </div>
              <p id="fork-clearance-hint" class="text-[10px] font-mono text-emerald-900 leading-tight">
                Outer span: 720 mm | Inner gap: 380 mm | Centered at (0, 0)
              </p>
            </div>
          </div>

          <!-- 10. Live Engineering Metrics Summary -->
          <div class="grid grid-cols-3 gap-2 text-center">
            <div class="bg-slate-50 border border-slate-200 rounded-xl p-2">
              <div class="text-[10px] font-semibold uppercase text-slate-400">Total Height</div>
              <div id="metric-total-height" class="font-mono text-xs font-bold text-slate-800 mt-0.5">1,334 mm</div>
            </div>
            <div class="bg-slate-50 border border-slate-200 rounded-xl p-2">
              <div class="text-[10px] font-semibold uppercase text-slate-400">Skid Blocks</div>
              <div id="metric-blocks" class="font-mono text-xs font-bold text-slate-800 mt-0.5">45 pcs</div>
            </div>
            <div class="bg-slate-50 border border-slate-200 rounded-xl p-2">
              <div class="text-[10px] font-semibold uppercase text-slate-400">Panel Area</div>
              <div id="metric-area" class="font-mono text-xs font-bold text-slate-800 mt-0.5">14.66 m²</div>
            </div>
          </div>

          <!-- 11. Plywood Sheet-Nesting & Saw Kerf Card -->
          <div class="bg-amber-50/80 border border-amber-200 rounded-xl p-2.5 flex items-center justify-between gap-2">
            <div>
              <div class="text-[10px] font-bold uppercase tracking-wider text-amber-900">1220 &times; 2240 mm Sheet Count</div>
              <div id="metric-sheets" class="font-mono text-xs font-bold text-slate-900 mt-0.5">6 sheets (16.40 m²)</div>
            </div>
            <button type="button" id="btn-kerf-toggle" onclick="toggleSawKerf()"
              class="px-2.5 py-1.5 rounded-lg bg-white border border-amber-300 text-[11px] font-mono font-semibold text-amber-900 hover:bg-amber-100 transition shrink-0">
              Kerf: 3 mm
            </button>
          </div>

          <!-- 12. Primary Action Buttons -->
          <div class="grid grid-cols-3 gap-2 pt-0.5 pb-1">
            <button type="button" onclick="toggleControlDrawer(false); toggleCutListModal(true);"
              class="py-2.5 px-2.5 rounded-xl bg-navy-800 hover:bg-navy-700 text-white text-xs font-semibold shadow-sm transition flex items-center justify-center gap-1">
              Schedule
            </button>
            <button type="button" onclick="downloadCuttingSchedulePDF()"
              class="py-2.5 px-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-semibold shadow-sm transition flex items-center justify-center gap-1">
              Export PDF
            </button>
            <button type="button" onclick="downloadSnapshot()"
              class="py-2.5 px-2.5 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-800 border border-slate-300 text-xs font-semibold transition flex items-center justify-center gap-1">
              3D PNG
            </button>
          </div>

        </div>
      </div>
    </aside>

    <!-- =================================================================== -->
    <!-- FULL-BLEED 3D CRATE PREVIEW VIEWPORT (PRIMARY SCREEN)               -->
    <!-- =================================================================== -->
    <main id="viewport-main" class="relative w-full h-full bg-gradient-to-b from-slate-800 via-slate-700 to-slate-800 overflow-hidden">

      <!-- 3D Canvas Host -->
      <div id="canvas-container"></div>

      <!-- Top Floating Header: Menu Drawer Button + Crate Spec Card + Quick Actions -->
      <div class="absolute top-2.5 sm:top-3.5 left-2.5 sm:left-3.5 right-2.5 sm:right-3.5 pointer-events-none flex items-start justify-between gap-2 z-20">

        <!-- Left: Primary Menu Drawer Trigger & Live Crate Spec HUD -->
        <div class="flex items-start gap-2 min-w-0 max-w-xl">
          <button type="button" id="btn-open-drawer" onclick="toggleControlDrawer(true)"
            class="pointer-events-auto px-3 py-2.5 rounded-xl bg-navy-800 hover:bg-navy-700 active:scale-[0.98] text-white border border-sky-300/30 shadow-xl flex items-center gap-2 text-xs sm:text-sm font-bold tracking-wide shrink-0 transition">
            <svg class="w-4 h-4 sm:w-5 sm:h-5 text-sky-200" fill="none" stroke="currentColor" stroke-width="2.3" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 6.75h16.5M3.75 12h16.5m-16.5 5.25h16.5" />
            </svg>
            <span>Controls</span>
          </button>

          <div id="hud-card" class="bg-slate-900/80 backdrop-blur-md border border-white/15 text-white rounded-xl px-3 py-2 shadow-lg pointer-events-auto min-w-0 transition-all">
            <div class="flex items-center justify-between gap-2">
              <div class="flex items-center gap-1.5 min-w-0 flex-wrap">
                <span id="hud-title" class="text-[11px] sm:text-xs font-bold text-sky-300 tracking-wide truncate">EXTERIOR CRATE • 3 RAILS • 19 MM</span>
                <span id="hud-recipient" class="hidden text-[10px] font-mono font-bold bg-navy-800 text-white px-1.5 py-0.5 rounded border border-sky-400/30 shrink-0"></span>
              </div>
              <button type="button" id="btn-toggle-hud-min" onclick="toggleHudMinimize()" title="Minimize or Expand Specs"
                class="p-1 rounded-lg bg-white/10 hover:bg-white/20 text-slate-200 transition shrink-0">
                <svg id="icon-hud-min" class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
                </svg>
              </button>
            </div>

            <div id="hud-details" class="hidden mt-0.5 space-y-0.5">
              <div id="hud-subtitle" class="text-[10px] sm:text-[11px] text-slate-300 font-mono leading-snug">
                Outer: 2440 &times; 1220 &times; 1220 mm | Inner: 2402 &times; 1182 &times; 1182 mm | Footings: 114 mm
              </div>
              <div id="hud-rail-clearance" class="text-[10px] sm:text-[11px] text-amber-300/95 font-mono leading-snug">
                Flush rail len: 2440 mm | Clear gap between rails: 460 mm
              </div>
            </div>
          </div>
        </div>

        <!-- Cut List and Scan QR shortcuts removed from the 3D preview. -->
      </div>

      <!-- Selected Part Inspector Card -->
      <div id="part-inspector" class="hidden absolute top-16 sm:top-16 right-2.5 sm:right-3.5 bg-white/95 backdrop-blur-md border border-slate-200 rounded-2xl p-3 sm:p-3.5 shadow-2xl z-30 w-60 sm:w-72 transition-all">
        <div class="flex items-start justify-between gap-2">
          <div class="min-w-0">
            <span id="insp-mode-badge" class="text-[10px] font-bold uppercase tracking-wider text-navy-800 bg-navy-100 px-2 py-0.5 rounded">Selected Part • Outside</span>
            <h3 id="insp-name" class="text-xs sm:text-sm font-bold text-slate-900 mt-1 truncate">Top Panel</h3>
          </div>
          <button type="button" onclick="clearPartSelection()" class="text-slate-400 hover:text-slate-700 p-1 rounded-lg shrink-0">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>
        <div class="mt-2 space-y-1.5 text-[11px] sm:text-xs font-mono bg-slate-50 p-2 sm:p-2.5 rounded-xl border border-slate-200">
          <div id="insp-primary-row" class="flex justify-between items-center px-1.5 py-1 rounded-lg bg-navy-100/85 border border-navy-700/20">
            <span id="insp-size-label" class="text-navy-900 font-semibold">Outside Size:</span>
            <span id="insp-size" class="font-bold text-navy-900">1220 &times; 2440 mm</span>
          </div>
          <div class="flex justify-between px-1">
            <span class="text-slate-500">Thickness:</span>
            <span id="insp-thick" class="font-bold text-slate-900">19 mm</span>
          </div>
          <div class="flex justify-between px-1">
            <span class="text-slate-500">Quantity:</span>
            <span id="insp-qty" class="font-bold text-slate-900">1 pc</span>
          </div>
        </div>
        <p id="insp-note" class="text-[10px] sm:text-[11px] text-slate-600 mt-2 leading-relaxed"></p>
      </div>

      <!-- Floating On-Canvas Assembly Step Player HUD (Icon-only Prev / Play / Next) -->
      <div id="preview-step-hud" class="hidden absolute bottom-14 sm:bottom-16 left-1/2 -translate-x-1/2 z-30 w-[94vw] max-w-md bg-slate-900/92 backdrop-blur-md border border-white/20 text-white rounded-2xl p-2.5 sm:p-3 shadow-2xl space-y-1.5">
        <div class="flex items-center justify-between gap-2">
          <span id="preview-step-badge" class="font-mono text-[10px] sm:text-[11px] font-bold text-sky-300 bg-white/10 px-2 py-0.5 rounded shrink-0">Step 7 / 7</span>
          <span id="preview-step-label" class="text-[11px] font-medium text-slate-200 truncate flex-1">7. Top — Complete Crate</span>
          <button type="button" onclick="togglePreviewStepPlayer(false)" title="Close Step Player" class="p-1 rounded-lg hover:bg-white/15 text-slate-300 shrink-0">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>
        <div class="flex items-center gap-1.5 pt-0.5">
          <button type="button" onclick="changeAssemblyStep(-1)" title="Previous Step" aria-label="Previous Step"
            class="w-9 h-9 rounded-xl bg-white/10 hover:bg-white/20 active:bg-white/25 text-white flex items-center justify-center shrink-0 transition">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.4" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
            </svg>
          </button>
          <input id="preview-slider-step" type="range" min="1" max="7" value="7"
            class="flex-1 accent-sky-400 cursor-pointer h-2 bg-white/20 rounded-lg" />
          <button type="button" id="btn-preview-play-steps" onclick="togglePlaySteps()" title="Play or Stop Steps" aria-label="Play or Stop Steps"
            class="w-9 h-9 rounded-xl bg-navy-800 hover:bg-navy-700 border border-sky-400/35 text-white flex items-center justify-center shrink-0 transition">
            <svg id="icon-preview-play" class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
              <path d="M5.25 5.653c0-.856.917-1.398 1.667-.986l11.54 6.348a1.125 1.125 0 010 1.971l-11.54 6.347a1.125 1.125 0 01-1.667-.985V5.653z" />
            </svg>
          </button>
          <button type="button" onclick="changeAssemblyStep(1)" title="Next Step" aria-label="Next Step"
            class="w-9 h-9 rounded-xl bg-white/10 hover:bg-white/20 active:bg-white/25 text-white flex items-center justify-center shrink-0 transition">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.4" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
            </svg>
          </button>
        </div>
      </div>

      <!-- Bottom Camera Preset Pill Bar + Steps + X-Ray + Dims + Reset -->
      <div class="absolute bottom-2.5 sm:bottom-3.5 left-1/2 -translate-x-1/2 z-20 flex items-center gap-1 bg-slate-900/90 backdrop-blur-md border border-white/15 p-1.5 rounded-2xl shadow-2xl max-w-[96vw] overflow-x-auto">
        <button type="button" onclick="setCameraPreset('iso')" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-white hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap">Iso</button>
        <button type="button" onclick="setCameraPreset('front')" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-slate-200 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap">Front</button>
        <button type="button" onclick="setCameraPreset('side')" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-slate-200 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap">Side</button>
        <button type="button" onclick="setCameraPreset('top')" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-slate-200 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap">Top</button>
        <button type="button" onclick="setCameraPreset('base')" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-amber-300 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap">Base</button>
        <div class="w-px h-4 sm:h-5 bg-white/20 mx-0.5 shrink-0"></div>
        <button type="button" id="btn-preview-steps-pill" onclick="togglePreviewStepPlayer()" title="Step-by-Step Assembly Player" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-emerald-300 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap flex items-center gap-1">
          <span>Steps</span>
        </button>
        <button type="button" id="btn-pill-xray" onclick="setRenderMode(state.renderMode === 'xray' ? 'solid' : 'xray')" title="Toggle X-Ray Inspection" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-slate-200 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap flex items-center gap-1">
          <span>X-Ray</span>
        </button>
        <button type="button" id="btn-pill-dims" onclick="toggleDimensionLines()" title="Show or Hide 3D Dimensions" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-cyan-200 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap flex items-center gap-1">
          <span>Dims</span>
        </button>
        <button type="button" onclick="resetCameraView()" title="Reset 3D Camera & Exploded View" class="px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-sky-300 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap flex items-center gap-1">
          <span>Reset</span>
        </button>
      </div>
    </main>
  </div>

  <!-- ===================================================================== -->
  <!-- CUTTING SCHEDULE, 2D SHEET NESTING, ISPM-15 & QR SYNC MODAL           -->
  <!-- ===================================================================== -->
  <div id="cutlist-modal" class="hidden fixed inset-0 z-50 bg-slate-950/75 backdrop-blur-sm flex items-center justify-center p-2 sm:p-4">
    <div class="bg-white rounded-2xl shadow-2xl border border-slate-200 w-full max-w-5xl max-h-[92dvh] flex flex-col overflow-hidden">

      <!-- Modal Header -->
      <div class="bg-navy-800 text-white px-4 sm:px-5 py-3.5 flex items-center justify-between gap-2 shrink-0">
        <div>
          <h2 class="text-sm sm:text-base font-bold tracking-wide">WORKSHOP CUTTING SCHEDULE &amp; 2D SHEET NESTING</h2>
          <p id="modal-spec-summary" class="text-[11px] sm:text-xs text-sky-200 font-mono mt-0.5">Exterior Crate • Outer: 2440 &times; 1220 &times; 1220 mm • 3 Base Rails • 19 mm Wood</p>
        </div>
        <button type="button" onclick="toggleCutListModal(false)" class="p-1.5 rounded-lg bg-white/10 hover:bg-white/20 text-white transition shrink-0">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Modal Body -->
      <div class="p-3.5 sm:p-5 overflow-y-auto space-y-5 flex-1">

        <!-- 1. Wooden Panels Table -->
        <div>
          <div class="flex flex-wrap items-center justify-between gap-2 mb-2">
            <h3 class="text-xs font-bold uppercase tracking-wider text-navy-800">1. Wooden Panels (Cut Dimensions)</h3>
            <span id="modal-panel-total" class="text-xs font-mono font-semibold text-slate-600 bg-slate-100 px-2.5 py-0.5 rounded-md">Total: 6 Panels • 14.66 m²</span>
          </div>
          <div class="overflow-x-auto border border-slate-200 rounded-xl">
            <table class="w-full text-left border-collapse">
              <thead>
                <tr class="bg-slate-100 border-b border-slate-200 text-[11px] font-bold uppercase tracking-wider text-slate-600">
                  <th class="py-2.5 px-3">Part</th>
                  <th class="py-2.5 px-3 text-right">Side A (mm)</th>
                  <th class="py-2.5 px-3 text-right">Side B (mm)</th>
                  <th class="py-2.5 px-3 text-right">Thick</th>
                  <th class="py-2.5 px-3 text-right">Qty</th>
                  <th class="py-2.5 px-3 text-right">Area (m²)</th>
                  <th class="py-2.5 px-3">Engineering Note</th>
                </tr>
              </thead>
              <tbody id="cutlist-panels-body" class="divide-y divide-slate-200 text-xs"></tbody>
            </table>
          </div>
        </div>

        <!-- 2. Base Rails & 5-Block Skid Stacks -->
        <div>
          <div class="flex flex-wrap items-center justify-between gap-2 mb-2">
            <h3 class="text-xs font-bold uppercase tracking-wider text-navy-800">2. Base Rails &amp; 5-Block Skid Footings</h3>
            <span id="modal-rail-clearance-badge" class="text-[11px] font-mono font-semibold text-amber-900 bg-amber-100 px-2.5 py-0.5 rounded-md">
              Flush rail len: 2440 mm | Clear gap: 460 mm
            </span>
          </div>
          <div class="overflow-x-auto border border-slate-200 rounded-xl">
            <table class="w-full text-left border-collapse">
              <thead>
                <tr class="bg-slate-100 border-b border-slate-200 text-[11px] font-bold uppercase tracking-wider text-slate-600">
                  <th class="py-2.5 px-3">Component</th>
                  <th class="py-2.5 px-3 text-right">Width (mm)</th>
                  <th class="py-2.5 px-3 text-right">Length (mm)</th>
                  <th class="py-2.5 px-3 text-right">Thick</th>
                  <th class="py-2.5 px-3 text-right">Qty</th>
                  <th class="py-2.5 px-3">Assembly Specification</th>
                </tr>
              </thead>
              <tbody id="cutlist-base-body" class="divide-y divide-slate-200 text-xs"></tbody>
            </table>
          </div>
        </div>

        <!-- 3. 2D Sheet-Nesting Layout (1220 x 2240 mm) & Offcut Record -->
        <div class="bg-slate-50 border border-slate-200 rounded-2xl p-3.5 sm:p-4 space-y-3">
          <div class="flex flex-wrap items-center justify-between gap-2">
            <div>
              <h3 class="text-xs font-bold uppercase tracking-wider text-navy-800">3. 2D Plywood Sheet-Nesting Plan (1220 &times; 2240 mm)</h3>
              <p id="nesting-summary-text" class="text-xs text-slate-600 font-mono mt-0.5">6 Standard Sheets • Yield: 89.4% • Kerf: 3 mm</p>
            </div>
            <div class="flex items-center gap-2">
              <button type="button" onclick="downloadNestingDXF()" class="px-3 py-1.5 rounded-xl bg-amber-600 hover:bg-amber-700 text-white text-xs font-semibold transition">
                Export 2D DXF
              </button>
            </div>
          </div>

          <div id="nesting-oversize-warning" class="hidden rounded-xl bg-amber-100 border border-amber-300 p-2.5 text-xs text-amber-950 font-medium"></div>
          <div id="nesting-sheets-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3"></div>

          <!-- Remaining Offcuts Table -->
          <div class="pt-2 border-t border-slate-200">
            <div class="flex items-center justify-between mb-1.5">
              <span class="text-xs font-bold uppercase tracking-wider text-slate-700">4. Remaining Sheet Offcuts (Remnant Inventory)</span>
              <span id="offcuts-total-badge" class="text-[11px] font-mono font-semibold text-slate-600 bg-white border border-slate-200 px-2 py-0.5 rounded">0 Offcuts</span>
            </div>
            <div class="overflow-x-auto bg-white border border-slate-200 rounded-xl">
              <table class="w-full text-left border-collapse">
                <thead>
                  <tr class="bg-slate-100 border-b border-slate-200 text-[10px] font-bold uppercase tracking-wider text-slate-600">
                    <th class="py-2 px-3">Sheet #</th>
                    <th class="py-2 px-3 text-right">Offcut Width (mm)</th>
                    <th class="py-2 px-3 text-right">Offcut Length (mm)</th>
                    <th class="py-2 px-3 text-right">Area (m²)</th>
                    <th class="py-2 px-3">Usability</th>
                  </tr>
                </thead>
                <tbody id="offcuts-table-body" class="divide-y divide-slate-200 text-xs"></tbody>
              </table>
            </div>
          </div>
        </div>

        <!-- 5. ISPM-15 Stencil & Shop Tablet QR Sync -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="bg-slate-50 border border-slate-200 rounded-2xl p-3.5 sm:p-4 flex flex-col justify-between gap-3">
            <div>
              <h3 class="text-xs font-bold uppercase tracking-wider text-navy-800">5. ISPM-15 Export Stamp &amp; Case Stencil</h3>
              <p class="text-[11px] text-slate-500 mt-0.5">Auto-generated shipping stencil with Name &amp; dimensions.</p>
            </div>
            <div class="bg-amber-100/60 border border-amber-300/80 rounded-xl p-2 flex items-center justify-center">
              <canvas id="stencil-canvas" width="520" height="220" class="w-full max-w-[340px] h-auto rounded"></canvas>
            </div>
            <button type="button" onclick="downloadStencilPNG()" class="w-full py-2 px-3 rounded-xl bg-navy-800 hover:bg-navy-700 text-white text-xs font-semibold transition">
              Download Stencil PNG
            </button>
          </div>

          <div class="bg-slate-50 border border-slate-200 rounded-2xl p-3.5 sm:p-4 flex flex-col justify-between gap-3">
            <div>
              <div class="flex items-center justify-between gap-2">
                <h3 class="text-xs font-bold uppercase tracking-wider text-navy-800">6. Shop Tablet QR Sync (Online &amp; Offline)</h3>
                <span class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-emerald-100 text-emerald-900">Works Offline</span>
              </div>
              <p class="text-[11px] text-slate-500 mt-0.5">Scan on phone/tablet or from printed PDF to open this exact 3D crate.</p>
            </div>
            <div class="flex flex-col sm:flex-row items-center gap-3.5 bg-white border border-slate-200 rounded-xl p-3">
              <canvas id="qr-canvas" width="200" height="200" class="w-28 h-28 shrink-0 rounded-lg border border-slate-100 bg-white"></canvas>
              <div class="text-[11px] text-slate-600 space-y-1.5 min-w-0 flex-1">
                <div class="font-semibold text-slate-800">Instant Offline &amp; Online Sync</div>
                <p class="leading-snug">Works with standard phone cameras, printed PDF sheets, or the built-in scanner—even without internet.</p>
                <div class="flex flex-wrap gap-1.5 pt-0.5">
                  <button type="button" onclick="openQRScannerModal()" class="px-2.5 py-1 rounded-lg bg-emerald-600 hover:bg-emerald-700 text-white font-semibold transition">
                    Open QR Scanner
                  </button>
                  <button type="button" onclick="copyShareURL()" class="px-2.5 py-1 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-800 font-semibold transition">
                    Copy Sync Code
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>

      </div>

      <!-- Modal Footer -->
      <div class="bg-slate-50 border-t border-slate-200 px-4 sm:px-5 py-3 flex flex-wrap items-center justify-between gap-2 shrink-0">
        <span class="text-[11px] text-slate-500">All dimensions in millimeters (mm) • 1220 &times; 2240 mm stock</span>
        <div class="flex flex-wrap items-center gap-2">
          <button type="button" onclick="copyCutListCSV()" class="px-3 py-2 rounded-xl bg-white border border-slate-300 hover:bg-slate-100 text-xs font-semibold text-slate-700 transition">
            Copy CSV
          </button>
          <button type="button" onclick="downloadCuttingSchedulePDF()" class="px-3.5 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-semibold shadow-sm transition">
            Export PDF (With QR)
          </button>
          <button type="button" onclick="toggleCutListModal(false)" class="px-3.5 py-2 rounded-xl bg-navy-800 hover:bg-navy-700 text-white text-xs font-semibold transition">
            Done
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- ===================================================================== -->
  <!-- ONLINE & OFFLINE QR CODE SCANNER MODAL                                -->
  <!-- ===================================================================== -->
  <div id="qr-scanner-modal" class="hidden fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-sm flex items-center justify-center p-2.5 sm:p-4">
    <div class="bg-white rounded-2xl shadow-2xl border border-slate-200 w-full max-w-md flex flex-col overflow-hidden">
      <div class="bg-navy-800 text-white px-4 py-3 flex items-center justify-between">
        <div>
          <h3 class="text-sm font-bold tracking-wide">SCAN CRATE QR CODE</h3>
          <p class="text-[11px] text-sky-200">Works Online &amp; Offline • Live Camera, Printed PDF Photo, or Code</p>
        </div>
        <button type="button" onclick="closeQRScannerModal()" class="p-1.5 rounded-lg bg-white/10 hover:bg-white/20 text-white transition">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <div class="p-4 space-y-3.5 overflow-y-auto max-h-[82dvh]">
        <!-- Live Camera Preview -->
        <div class="relative rounded-xl overflow-hidden bg-slate-900 border border-slate-300 aspect-video flex items-center justify-center">
          <video id="qr-video" playsinline muted class="w-full h-full object-cover hidden"></video>
          <div id="qr-camera-placeholder" class="text-center p-4 space-y-2">
            <p class="text-xs text-slate-300 font-medium">Point camera at a printed PDF or tablet QR code</p>
            <button type="button" onclick="startQRCameraScan()" class="px-3.5 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-semibold shadow transition">
              Start Camera Scan
            </button>
          </div>
          <div id="qr-scan-reticle" class="hidden pointer-events-none absolute inset-6 border-2 border-emerald-400/80 rounded-xl shadow-[0_0_0_9999px_rgba(15,23,42,0.35)]"></div>
        </div>
        <div id="qr-scan-status" class="text-[11px] font-mono text-center text-slate-600 bg-slate-100 py-1.5 px-2.5 rounded-lg">
          Ready to scan via Camera, Image/PDF Snapshot, or Sync Code
        </div>

        <!-- Upload Printed PDF / Screenshot Image -->
        <div class="bg-slate-50 border border-slate-200 rounded-xl p-3 space-y-2">
          <label class="block text-xs font-bold text-slate-800">Scan from Photo / Printed PDF Screenshot</label>
          <input id="qr-file-input" type="file" accept="image/*" onchange="handleQRImageUpload(event)"
            class="block w-full text-xs text-slate-600 file:mr-3 file:py-1.5 file:px-3 file:rounded-lg file:border-0 file:text-xs file:font-semibold file:bg-navy-800 file:text-white hover:file:bg-navy-700 cursor-pointer" />
        </div>

        <!-- Manual Paste Sync Code / URL -->
        <div class="bg-slate-50 border border-slate-200 rounded-xl p-3 space-y-2">
          <label for="qr-manual-input" class="block text-xs font-bold text-slate-800">Or Paste Crate Sync Code / URL</label>
          <div class="flex gap-1.5">
            <input id="qr-manual-input" type="text" placeholder="CRATE3D:type=ext&l=2440..."
              class="flex-1 bg-white border border-slate-300 rounded-lg px-2.5 py-1.5 font-mono text-xs text-slate-800 focus:outline-none focus:border-navy-800" />
            <button type="button" onclick="applyManualQRCode()" class="px-3 py-1.5 rounded-lg bg-navy-800 hover:bg-navy-700 text-white text-xs font-semibold transition">
              Load
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Toast Notification Banner -->
  <div id="toast-banner" class="hidden fixed bottom-16 right-4 z-50 bg-slate-900 text-white text-xs font-medium px-4 py-2.5 rounded-xl shadow-xl border border-white/15"></div>

  <script>
    // =========================================================================
    // 1. PURE-JS OFFLINE & ONLINE ISO/IEC 18004 QR CODE GENERATOR
    // =========================================================================
    const OfflineQR = (() => {
      const GF_EXP = new Uint8Array(512);
      const GF_LOG = new Uint8Array(256);
      let x = 1;
      for (let i = 0; i < 255; i++) {
        GF_EXP[i] = x;
        GF_LOG[x] = i;
        x <<= 1;
        if (x & 0x100) x ^= 0x11d;
      }
      for (let i = 255; i < 512; i++) GF_EXP[i] = GF_EXP[i - 255];

      function gfMul(a, b) {
        if (a === 0 || b === 0) return 0;
        return GF_EXP[GF_LOG[a] + GF_LOG[b]];
      }

      function rsGeneratorPoly(ecLen) {
        let poly = [1];
        for (let i = 0; i < ecLen; i++) {
          const next = new Array(poly.length + 1).fill(0);
          const factor = GF_EXP[i];
          for (let j = 0; j < poly.length; j++) {
            next[j] ^= poly[j];
            next[j + 1] ^= gfMul(poly[j], factor);
          }
          poly = next;
        }
        return poly;
      }

      function rsEncode(data, ecLen) {
        const gen = rsGeneratorPoly(ecLen);
        const res = new Array(ecLen).fill(0);
        for (let i = 0; i < data.length; i++) {
          const coef = data[i] ^ res[0];
          res.shift();
          res.push(0);
          for (let j = 0; j < ecLen; j++) {
            res[j] ^= gfMul(gen[j + 1], coef);
          }
        }
        return res;
      }

      const VERSIONS = [
        null,
        { size: 21, total: 26, ec: 7, data: 19, align: [] },
        { size: 25, total: 44, ec: 10, data: 34, align: [6, 18] },
        { size: 29, total: 70, ec: 15, data: 55, align: [6, 22] },
        { size: 33, total: 100, ec: 20, data: 80, align: [6, 26] },
        { size: 37, total: 134, ec: 26, data: 108, align: [6, 30] },
        { size: 41, total: 172, ec: 36, data: 136, align: [6, 34] },
        { size: 45, total: 196, ec: 40, data: 156, align: [6, 22, 38] },
        { size: 49, total: 242, ec: 48, data: 194, align: [6, 24, 42] },
        { size: 53, total: 292, ec: 60, data: 232, align: [6, 26, 46] },
        { size: 57, total: 346, ec: 72, data: 274, align: [6, 28, 50] }
      ];

      const FORMAT_BITS_L = [
        0x77c4, 0x72f3, 0x7daa, 0x789d, 0x662f, 0x6318, 0x6c41, 0x6976
      ];

      function buildMatrix(text) {
        const bytes = new TextEncoder().encode(text);
        let ver = 1;
        while (ver < VERSIONS.length && VERSIONS[ver].data < bytes.length + 2) ver++;
        if (ver >= VERSIONS.length) ver = VERSIONS.length - 1;
        const spec = VERSIONS[ver];
        const maxBytes = spec.data - 2;
        const payload = bytes.slice(0, maxBytes);

        const bits = [];
        const pushBits = (val, len) => {
          for (let i = len - 1; i >= 0; i--) bits.push((val >> i) & 1);
        };
        pushBits(0b0100, 4);
        pushBits(payload.length, ver <= 9 ? 8 : 16);
        for (const b of payload) pushBits(b, 8);
        pushBits(0, Math.min(4, spec.data * 8 - bits.length));
        while (bits.length % 8 !== 0) bits.push(0);

        const codewords = [];
        for (let i = 0; i < bits.length; i += 8) {
          let byte = 0;
          for (let j = 0; j < 8; j++) byte = (byte << 1) | bits[i + j];
          codewords.push(byte);
        }
        const padBytes = [0xec, 0x11];
        let pIdx = 0;
        while (codewords.length < spec.data) {
          codewords.push(padBytes[pIdx % 2]);
          pIdx++;
        }

        const ecBytes = rsEncode(codewords, spec.ec);
        const allBytes = codewords.concat(ecBytes);
        const bitStream = [];
        for (const b of allBytes) {
          for (let i = 7; i >= 0; i--) bitStream.push((b >> i) & 1);
        }

        const n = spec.size;
        const baseGrid = Array.from({ length: n }, () => new Array(n).fill(null));
        const isFunc = Array.from({ length: n }, () => new Array(n).fill(false));

        const setMod = (r, c, val, fn = true) => {
          if (r >= 0 && r < n && c >= 0 && c < n) {
            baseGrid[r][c] = val ? 1 : 0;
            if (fn) isFunc[r][c] = true;
          }
        };

        const drawFinder = (r0, c0) => {
          for (let dr = -1; dr <= 7; dr++) {
            for (let dc = -1; dc <= 7; dc++) {
              const inBox = dr >= 0 && dr <= 6 && dc >= 0 && dc <= 6;
              const onBorder = dr === 0 || dr === 6 || dc === 0 || dc === 6;
              const inCore = dr >= 2 && dr <= 4 && dc >= 2 && dc <= 4;
              setMod(r0 + dr, c0 + dc, inBox && (onBorder || inCore), true);
            }
          }
        };
        drawFinder(0, 0);
        drawFinder(0, n - 7);
        drawFinder(n - 7, 0);

        for (let i = 8; i < n - 8; i++) {
          setMod(6, i, i % 2 === 0, true);
          setMod(i, 6, i % 2 === 0, true);
        }

        if (spec.align.length >= 2) {
          for (const r of spec.align) {
            for (const c of spec.align) {
              if ((r === 6 && c === 6) || (r === 6 && c === n - 7) || (r === n - 7 && c === 6)) continue;
              for (let dr = -2; dr <= 2; dr++) {
                for (let dc = -2; dc <= 2; dc++) {
                  const val = Math.max(Math.abs(dr), Math.abs(dc)) !== 1;
                  setMod(r + dr, c + dc, val, true);
                }
              }
            }
          }
        }

        for (let i = 0; i < 9; i++) {
          if (i !== 6) {
            isFunc[8][i] = true;
            isFunc[i][8] = true;
          }
        }
        for (let i = 0; i < 8; i++) {
          isFunc[8][n - 1 - i] = true;
          isFunc[n - 1 - i][8] = true;
        }
        setMod(n - 8, 8, 1, true);

        const applyMask = (maskId) => {
          const g = baseGrid.map(row => row.slice());
          let bIdx = 0;
          let dir = -1;
          for (let col = n - 1; col > 0; col -= 2) {
            if (col === 6) col--;
            for (let step = 0; step < n; step++) {
              const row = dir === -1 ? (n - 1 - step) : step;
              for (let cOff = 0; cOff < 2; cOff++) {
                const c = col - cOff;
                if (!isFunc[row][c]) {
                  const rawBit = bIdx < bitStream.length ? bitStream[bIdx++] : 0;
                  let inv = false;
                  if (maskId === 0) inv = (row + c) % 2 === 0;
                  else if (maskId === 1) inv = row % 2 === 0;
                  else if (maskId === 2) inv = c % 3 === 0;
                  else if (maskId === 3) inv = (row + c) % 3 === 0;
                  else if (maskId === 4) inv = (Math.floor(row / 2) + Math.floor(c / 3)) % 2 === 0;
                  else if (maskId === 5) inv = ((row * c) % 2 + (row * c) % 3) === 0;
                  else if (maskId === 6) inv = (((row * c) % 2 + (row * c) % 3) % 2) === 0;
                  else inv = (((row + c) % 2 + (row * c) % 3) % 2) === 0;
                  g[row][c] = rawBit ^ (inv ? 1 : 0);
                }
              }
            }
            dir *= -1;
          }

          const fmt = FORMAT_BITS_L[maskId];
          const fmtCoords1 = [
            [8, 0], [8, 1], [8, 2], [8, 3], [8, 4], [8, 5], [8, 7], [8, 8],
            [7, 8], [5, 8], [4, 8], [3, 8], [2, 8], [1, 8], [0, 8]
          ];
          const fmtCoords2 = [
            [n - 1, 8], [n - 2, 8], [n - 3, 8], [n - 4, 8], [n - 5, 8], [n - 6, 8], [n - 7, 8],
            [8, n - 8], [8, n - 7], [8, n - 6], [8, n - 5], [8, n - 4], [8, n - 3], [8, n - 2], [8, n - 1]
          ];
          for (let i = 0; i < 15; i++) {
            const bit = (fmt >> (14 - i)) & 1;
            g[fmtCoords1[i][0]][fmtCoords1[i][1]] = bit;
            g[fmtCoords2[i][0]][fmtCoords2[i][1]] = bit;
          }
          return g;
        };

        let bestGrid = null;
        let bestPenalty = Infinity;
        for (let m = 0; m < 8; m++) {
          const candidate = applyMask(m);
          let penalty = 0;
          for (let r = 0; r < n; r++) {
            let run = 1;
            for (let c = 1; c < n; c++) {
              if (candidate[r][c] === candidate[r][c - 1]) {
                run++;
                if (run === 5) penalty += 3;
                else if (run > 5) penalty += 1;
              } else {
                run = 1;
              }
            }
          }
          for (let c = 0; c < n; c++) {
            let run = 1;
            for (let r = 1; r < n; r++) {
              if (candidate[r][c] === candidate[r - 1][c]) {
                run++;
                if (run === 5) penalty += 3;
                else if (run > 5) penalty += 1;
              } else {
                run = 1;
              }
            }
          }
          if (penalty < bestPenalty) {
            bestPenalty = penalty;
            bestGrid = candidate;
          }
        }
        return bestGrid;
      }

      function drawToCanvas(canvas, text, size = 240) {
        const matrix = buildMatrix(text);
        const n = matrix.length;
        const quiet = 4;
        const totalModules = n + quiet * 2;
        const scale = Math.max(2, Math.floor(size / totalModules));
        const actualSize = totalModules * scale;
        canvas.width = actualSize;
        canvas.height = actualSize;
        const ctx = canvas.getContext('2d');
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, 0, actualSize, actualSize);
        ctx.fillStyle = '#0f172a';
        for (let r = 0; r < n; r++) {
          for (let c = 0; c < n; c++) {
            if (matrix[r][c]) {
              ctx.fillRect((c + quiet) * scale, (r + quiet) * scale, scale, scale);
            }
          }
        }
      }

      return { drawToCanvas };
    })();

    // =========================================================================
    // 2. STATE & CONSTANTS
    // =========================================================================
    const SHEET_W = 1220;
    const SHEET_L = 2240;

    // Application State
    // Base inputs (baseL, baseW, baseH) are preserved across Exterior and Internal modes:
    // - Exterior Crate: Outer = baseL × baseW × baseH, Inner = Outer - 2T.
    // - Internal Crate: Inner = baseL × baseW × baseH, Outer = Inner + 2T (panels expand outward).
    const state = {
      crateType: 'Exterior Crate',
      recipient: '',
      baseL: 2440,
      baseW: 1220,
      baseH: 1220,
      levels: 1,
      heightL2: 800,
      l2WidthUnlocked: false,
      sideL2: 1182,
      thickness: 19,
      rails: 3,
      longside1Height: 1220,
      l2Longside1Height: 1220,
      jointHeight: 300,
      jointLengthTrim: 40,
      explode: 0,
      explodedViewActive: false,
      explodeAnimationFrame: null,
      assemblyStep: 7,
      isPlayingSteps: false,
      showPreviewStepPlayer: false,
      drawerOpen: false,
      renderMode: 'solid',
      showDimensions: true,
      showForkGuides: false,
      forkEntrySide: 'length',
      forkBladeWidth: 170,
      forkSpread: 550,
      sawKerf: 3,
      hideTopLid: false,
      hideFrontWall: false,
      showJoints: false,
      hudMinimized: true,
      showAdvancedControls: false,
      hiddenParts: new Set(),
      selectedPartName: null,
      selectedPartMesh: null,
      lastCalc: null
    };

    // =========================================================================
    // 2A. BUILT-IN WORKSHOP AUDIO (procedural; no external files or toggle)
    // =========================================================================
    let workshopAudioContext = null;

    function getWorkshopAudioContext() {
      const AudioContextClass = window.AudioContext || window.webkitAudioContext;
      if (!AudioContextClass) return null;
      if (!workshopAudioContext) {
        try { workshopAudioContext = new AudioContextClass(); } catch (err) { return null; }
      }
      if (workshopAudioContext.state === 'suspended') {
        workshopAudioContext.resume().catch(() => {});
      }
      return workshopAudioContext;
    }

    function playNormalTapSound() {
      const ctx = getWorkshopAudioContext();
      if (!ctx) return;
      if (ctx.state !== 'running') {
        ctx.resume().then(() => playNormalTapSound()).catch(() => {});
        return;
      }
      const now = ctx.currentTime;
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      const filter = ctx.createBiquadFilter();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(1050, now);
      osc.frequency.exponentialRampToValueAtTime(620, now + 0.045);
      filter.type = 'lowpass';
      filter.frequency.value = 1500;
      gain.gain.setValueAtTime(0.0001, now);
      gain.gain.linearRampToValueAtTime(0.075, now + 0.004);
      gain.gain.exponentialRampToValueAtTime(0.0001, now + 0.065);
      osc.connect(filter); filter.connect(gain); gain.connect(ctx.destination);
      osc.start(now); osc.stop(now + 0.07);
    }

    function playWorkshopAssemblySound() {
      const ctx = getWorkshopAudioContext();
      if (!ctx) return;
      if (ctx.state !== 'running') {
        ctx.resume().then(() => playWorkshopAssemblySound()).catch(() => {});
        return;
      }
      const now = ctx.currentTime;
      const master = ctx.createGain();
      master.gain.setValueAtTime(0.62, now);
      master.connect(ctx.destination);

      // Short wooden slide: filtered friction noise, followed by two muted wood knocks.
      const length = Math.floor(ctx.sampleRate * 0.34);
      const buffer = ctx.createBuffer(1, length, ctx.sampleRate);
      const data = buffer.getChannelData(0);
      for (let i = 0; i < length; i++) {
        const t = i / length;
        data[i] = (Math.random() * 2 - 1) * Math.pow(1 - t, 1.65);
      }
      const noise = ctx.createBufferSource(); noise.buffer = buffer;
      const filter = ctx.createBiquadFilter(); filter.type = 'lowpass';
      filter.frequency.setValueAtTime(1050, now);
      filter.frequency.exponentialRampToValueAtTime(300, now + 0.31);
      const slideGain = ctx.createGain();
      slideGain.gain.setValueAtTime(0.0001, now);
      slideGain.gain.linearRampToValueAtTime(0.23, now + 0.025);
      slideGain.gain.exponentialRampToValueAtTime(0.0001, now + 0.33);
      noise.connect(filter); filter.connect(slideGain); slideGain.connect(master);
      noise.start(now); noise.stop(now + 0.35);

      const makeKnock = (at, frequency, peak, duration) => {
        const osc = ctx.createOscillator(); const g = ctx.createGain();
        const low = ctx.createBiquadFilter(); low.type = 'lowpass'; low.frequency.value = 520;
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(frequency, now + at);
        osc.frequency.exponentialRampToValueAtTime(frequency * 0.48, now + at + duration);
        g.gain.setValueAtTime(0.0001, now + at);
        g.gain.linearRampToValueAtTime(peak, now + at + 0.012);
        g.gain.exponentialRampToValueAtTime(0.0001, now + at + duration);
        osc.connect(low); low.connect(g); g.connect(master);
        osc.start(now + at); osc.stop(now + at + duration + 0.01);
      };
      makeKnock(0.23, 145, 0.30, 0.19);
      makeKnock(0.34, 92, 0.16, 0.22);
      master.gain.setTargetAtTime(0.0001, now + 0.58, 0.12);
    }

    // Unlock audio on the first genuine user interaction (required by mobile browsers).
    document.addEventListener('pointerdown', () => { getWorkshopAudioContext(); }, { once: true, passive: true });
    document.addEventListener('click', (event) => {
      const button = event.target && event.target.closest ? event.target.closest('button') : null;
      if (!button || button.disabled) return;
      // Ordinary interface tap only. Workshop wood effects are reserved for assembly steps.
      playNormalTapSound();
    }, true);

    // =========================================================================
    // 3. PROCEDURAL WOOD TEXTURE & MATERIALS
    // =========================================================================
    function createPlywoodTexture(baseHex, grainHex, seed = 1) {
      const canvas = document.createElement('canvas');
      canvas.width = 1024;
      canvas.height = 1024;
      const ctx = canvas.getContext('2d');
      ctx.fillStyle = baseHex;
      ctx.fillRect(0, 0, 1024, 1024);

      let s = seed * 9973;
      const rand = () => { s = (s * 16807) % 2147483647; return (s - 1) / 2147483646; };

      // Subtle broad tone variation gives the plywood a natural, non-plastic surface.
      for (let band = 0; band < 34; band++) {
        const y = rand() * 1024;
        const h = 4 + rand() * 34;
        ctx.globalAlpha = 0.018 + rand() * 0.035;
        ctx.fillStyle = rand() > 0.5 ? '#fff0d8' : '#68401f';
        ctx.fillRect(0, y, 1024, h);
      }

      // Long, irregular wood fibers with varied thickness and waviness.
      for (let i = 0; i < 390; i++) {
        const y = rand() * 1024;
        const phase = rand() * Math.PI * 2;
        const amp = 1 + rand() * 5.5;
        ctx.globalAlpha = 0.035 + rand() * 0.095;
        ctx.strokeStyle = rand() > 0.18 ? grainHex : '#f7dfbd';
        ctx.lineWidth = 0.35 + rand() * 2.2;
        ctx.beginPath();
        ctx.moveTo(0, y);
        for (let x = 0; x <= 1024; x += 24) {
          const yy = y + Math.sin(x * (0.004 + rand() * 0.002) + phase) * amp + (rand() - 0.5) * 2.2;
          ctx.lineTo(x, yy);
        }
        ctx.stroke();
      }

      // Occasional fine knots and growth-ring contours.
      for (let k = 0; k < 9; k++) {
        const x = 35 + rand() * 954; const y = 35 + rand() * 954;
        const rx = 7 + rand() * 22; const ry = 2.5 + rand() * 7;
        for (let ring = 0; ring < 4; ring++) {
          ctx.beginPath();
          ctx.globalAlpha = 0.07 + rand() * 0.045;
          ctx.strokeStyle = grainHex; ctx.lineWidth = 0.8 + rand() * 1.1;
          ctx.ellipse(x, y, rx + ring * 4, ry + ring * 1.6, (rand() - 0.5) * 0.12, 0, Math.PI * 2);
          ctx.stroke();
        }
      }

      // Fine pores and tiny tonal flecks.
      for (let i = 0; i < 9000; i++) {
        ctx.globalAlpha = 0.025 + rand() * 0.07;
        ctx.fillStyle = rand() > 0.52 ? '#fff2df' : '#5e3c22';
        const x = rand() * 1024; const y = rand() * 1024;
        const w = 0.4 + rand() * 1.4; const h = 0.3 + rand() * 0.9;
        ctx.fillRect(x, y, w, h);
      }

      // Slight edge-to-edge tonal softness; no geometry or dimensions are altered.
      const vignette = ctx.createLinearGradient(0, 0, 0, 1024);
      vignette.addColorStop(0, 'rgba(70,42,18,0.045)');
      vignette.addColorStop(0.5, 'rgba(255,245,225,0.015)');
      vignette.addColorStop(1, 'rgba(70,42,18,0.055)');
      ctx.globalAlpha = 1; ctx.fillStyle = vignette; ctx.fillRect(0, 0, 1024, 1024);

      const tex = new THREE.CanvasTexture(canvas);
      tex.wrapS = THREE.RepeatWrapping;
      tex.wrapT = THREE.RepeatWrapping;
      tex.anisotropy = 8;
      tex.colorSpace = THREE.SRGBColorSpace;
      tex.needsUpdate = true;
      return tex;
    }

    const woodTextures = {
      plywood: createPlywoodTexture('#dfb786', '#9a6b3b', 11),
      plywoodDark: createPlywoodTexture('#cfa16b', '#805226', 23),
      timber: createPlywoodTexture('#c69258', '#6f4219', 37),
      l2Plywood: createPlywoodTexture('#d9b17f', '#875a2d', 53)
    };

    const materials = {
      bottom: new THREE.MeshStandardMaterial({ map: woodTextures.plywoodDark, roughness: 0.78, metalness: 0.04 }),
      longside: new THREE.MeshStandardMaterial({ map: woodTextures.plywood, roughness: 0.74, metalness: 0.04 }),
      longsideUpper: new THREE.MeshStandardMaterial({ map: woodTextures.plywood, color: 0xfbe5c8, roughness: 0.72, metalness: 0.04 }),
      shortside: new THREE.MeshStandardMaterial({ map: woodTextures.plywood, color: 0xf7dcb8, roughness: 0.76, metalness: 0.04 }),
      top: new THREE.MeshStandardMaterial({ map: woodTextures.plywood, color: 0xffe6c7, roughness: 0.7, metalness: 0.04 }),
      l2Bottom: new THREE.MeshStandardMaterial({ map: woodTextures.plywoodDark, color: 0xf4dec4, roughness: 0.76, metalness: 0.04 }),
      l2Longside: new THREE.MeshStandardMaterial({ map: woodTextures.l2Plywood, roughness: 0.74, metalness: 0.04 }),
      l2LongsideUpper: new THREE.MeshStandardMaterial({ map: woodTextures.l2Plywood, color: 0xfce7cb, roughness: 0.72, metalness: 0.04 }),
      l2Shortside: new THREE.MeshStandardMaterial({ map: woodTextures.l2Plywood, color: 0xf7dfb9, roughness: 0.75, metalness: 0.04 }),
      l2Top: new THREE.MeshStandardMaterial({ map: woodTextures.l2Plywood, color: 0xffe8cb, roughness: 0.7, metalness: 0.04 }),
      joint: new THREE.MeshStandardMaterial({ map: woodTextures.timber, color: 0xd9a76a, roughness: 0.8, metalness: 0.05 }),
      rail: new THREE.MeshStandardMaterial({ map: woodTextures.timber, roughness: 0.85, metalness: 0.03 }),
      block: new THREE.MeshStandardMaterial({ map: woodTextures.timber, color: 0xd8a368, roughness: 0.88, metalness: 0.02 }),
      highlight: new THREE.MeshStandardMaterial({
        color: 0x102a43,
        emissive: 0x1e3a8a,
        emissiveIntensity: 0.45,
        roughness: 0.35,
        metalness: 0.1,
        transparent: true,
        opacity: 0.56
      }),
      internalZoneOverlay: new THREE.MeshStandardMaterial({
        color: 0x102a43,
        emissive: 0x1e3a8a,
        emissiveIntensity: 0.52,
        roughness: 0.32,
        metalness: 0.12,
        transparent: true,
        opacity: 0.54,
        depthWrite: false
      }),
      edgeLine: new THREE.LineBasicMaterial({ color: 0x4b2e14, transparent: true, opacity: 0.42 })
    };

    // =========================================================================
    // 4. PARAMETRIC CRATE ENGINEERING & 2D SHEET NESTING ENGINE
    // =========================================================================
    function computeCrateEngineering() {
      const T = state.thickness;
      const numRails = state.rails;
      const levels = state.levels;
      const isInternal = state.crateType === 'Internal Crate';

      // Base input (baseL × baseW × baseH) stays fixed as the active reference:
      // - Exterior Crate: Outer = baseL × baseW × baseH, Inner = Outer - 2T.
      // - Internal Crate: Inner = baseL × baseW × baseH, Outer = Inner + 2T.
      const outerL = isInternal ? (state.baseL + 2 * T) : state.baseL;
      const outerW = isInternal ? (state.baseW + 2 * T) : state.baseW;
      const boxH = isInternal ? (state.baseH + 2 * T) : state.baseH;
      const innerClearL = outerL - 2 * T;
      const innerClearW = outerW - 2 * T;
      const innerClearH = boxH - 2 * T;

      // Level 1 Structural Panel Dimensions (Full Outer Box)
      const bottomA = outerW - 2 * T;
      const bottomB = outerL - 2 * T;
      const shortA = outerW - 2 * T;
      const shortB = boxH - T;
      const longA = outerL;
      const longB = boxH - T;
      const topA = outerW;
      const topB = outerL;

      const panels = [];
      const addPanel = (name, sideA, sideB, qty, colorKey, note, internalSize = null) => {
        const areaM2 = (sideA * sideB * qty) / 1_000_000;
        panels.push({ name, sideA, sideB, qty, areaM2, colorKey, note, internalSize });
      };

      addPanel('Bottom', bottomA, bottomB, 1, 'bottom', 'Base deck fitted inside walls (W - 2T) × (L - 2T)', `${innerClearW} × ${innerClearL} mm`);

      const isSplitL1 = longB > 1220;
      let long1H = longB;
      let long2H = 0;
      const trim = Math.max(40, state.jointLengthTrim);
      const jointH = Math.min(400, Math.max(250, state.jointHeight));
      const jointL = Math.max(100, longA - trim);

      if (!isSplitL1) {
        addPanel('Longside', longA, longB, 2, 'longside', 'Full-height side walls L × (H - T)', `${innerClearL} × ${innerClearH} mm`);
      } else {
        const minH1 = Math.ceil(longB / 2);
        const maxH1 = Math.min(1220, longB - 50);
        long1H = Math.max(minH1, Math.min(maxH1, state.longside1Height));
        long2H = longB - long1H;
        const innerL1H = Math.max(1, long1H - T);
        const innerL2H = Math.max(0, innerClearH - innerL1H);
        addPanel('Longside 1 (Lower)', longA, long1H, 2, 'longside', `Lower split wall (≥ Longside 2; max 1220 mm)`, `${innerClearL} × ${innerL1H} mm`);
        addPanel('Longside 2 (Upper)', longA, long2H, 2, 'longsideUpper', `Upper split wall (H - T - Longside 1)`, `${innerClearL} × ${innerL2H} mm`);
      }

      addPanel('Shortside', shortA, shortB, 2, 'shortside', 'End walls fitted inside (W - 2T) × (H - T)', `${innerClearW} × ${innerClearH} mm`);
      addPanel('Top', topA, topB, 1, 'top', levels === 2 ? 'Level 1 top / Level 2 base deck W × L' : 'Full-coverage top lid W × L', `${innerClearW} × ${innerClearL} mm`);

      // Level 2 Panel Engineering (Zero Overlap):
      // - Level 2 Bottom (Middle): width equals Shortside width + Longside Rear thickness (Side + T), length equals Top length (L), flush to rear of Level 1 Top.
      // - Level 2 Longside (Rear): exact height of Level 2 Shortsides (H2), resting on rear edge of Level 2 Bottom: L × H2.
      // - Level 2 Shortsides (Left & Right): internal height H2 above Level 2 Bottom: Side × H2.
      // - Level 2 Longside Front: from Level 1 Top surface to the top edge of Level 2 Top: L × (H2 + 2T).
      // - Level 2 Top: same footprint and rear-edge alignment as Level 2 Bottom (Middle): (Side + T) × L.
      let l2WallH = 0;
      let l2ShortLen = shortA;
      let l2OuterW = outerW;
      let l2InnerClearW = innerClearW;
      let l2InnerClearH = 0;
      let l2BottomA = 0;
      let l2BottomB = outerL;
      let l2ShortA = 0;
      let l2ShortB = 0;
      let l2LongRearA = outerL;
      let l2LongRearB = 0;
      let l2LongFrontA = outerL;
      let l2LongFrontB = 0;
      let l2TopA = topA;
      let l2TopB = topB;
      let isSplitL2Rear = false;
      let isSplitL2Front = false;
      let l2Rear1H = 0;
      let l2Rear2H = 0;
      let l2Front1H = 0;
      let l2Front2H = 0;

      if (levels === 2) {
        l2InnerClearH = Math.max(100, state.heightL2);
        l2ShortLen = state.l2WidthUnlocked ? Math.max(200, state.sideL2) : shortA;
        l2OuterW = l2ShortLen + 2 * T;
        l2InnerClearW = l2ShortLen;
        l2WallH = l2InnerClearH + T;

        l2BottomA = l2ShortLen + T;
        l2BottomB = outerL;
        l2ShortA = l2ShortLen;
        l2ShortB = l2InnerClearH;
        l2LongRearA = outerL;
        l2LongRearB = l2InnerClearH;
        l2LongFrontA = outerL;
        l2LongFrontB = l2InnerClearH + 2 * T;
        l2TopA = l2BottomA;
        l2TopB = l2BottomB;

        addPanel('Level 2 Bottom (Middle)', l2BottomA, l2BottomB, 1, 'l2Bottom', 'Flush to rear of L1 Top, same length as Tops: (Shortside W + Rear Thick) × L', `${l2InnerClearW} × ${innerClearL} mm`);

        isSplitL2Rear = l2LongRearB > 1220;
        if (!isSplitL2Rear) {
          addPanel('Level 2 Longside (Rear)', l2LongRearA, l2LongRearB, 1, 'l2Longside', 'Exact height of L2 Shortsides on L2 Bottom: L × H2', `${innerClearL} × ${l2InnerClearH} mm`);
        } else {
          const minR1 = Math.ceil(l2LongRearB / 2);
          const maxR1 = Math.min(1220, l2LongRearB - 50);
          l2Rear1H = Math.max(minR1, Math.min(maxR1, state.l2Longside1Height));
          l2Rear2H = l2LongRearB - l2Rear1H;
          addPanel('Level 2 Rear 1 (Lower)', l2LongRearA, l2Rear1H, 1, 'l2Longside', `L2 Rear lower split wall (${l2Rear1H} mm ≥ Rear 2)`, `${innerClearL} × ${l2Rear1H} mm`);
          addPanel('Level 2 Rear 2 (Upper)', l2LongRearA, l2Rear2H, 1, 'l2LongsideUpper', `L2 Rear upper split wall (${l2Rear2H} mm)`, `${innerClearL} × ${l2Rear2H} mm`);
        }

        addPanel('Level 2 Shortside', l2ShortA, l2ShortB, 2, 'l2Shortside', 'Level 2 internal-height end walls: Side × H2', `${l2InnerClearW} × ${l2InnerClearH} mm`);

        isSplitL2Front = l2LongFrontB > 1220;
        if (!isSplitL2Front) {
          addPanel('Level 2 Longside Front', l2LongFrontA, l2LongFrontB, 1, 'l2Longside', 'From Level 1 Top surface to top edge of Level 2 Top (H2 + 2T)', `${innerClearL} × ${l2InnerClearH} mm`);
        } else {
          const minF1 = Math.ceil(l2LongFrontB / 2);
          const maxF1 = Math.min(1220, l2LongFrontB - 50);
          l2Front1H = Math.max(minF1, Math.min(maxF1, state.l2Longside1Height));
          l2Front2H = l2LongFrontB - l2Front1H;
          const innerF1 = Math.max(1, l2Front1H - T);
          const innerF2 = Math.max(0, l2InnerClearH - innerF1);
          addPanel('Level 2 Front 1 (Lower)', l2LongFrontA, l2Front1H, 1, 'l2Longside', `L2 Front lower split wall (${l2Front1H} mm ≥ Front 2)`, `${innerClearL} × ${innerF1} mm`);
          addPanel('Level 2 Front 2 (Upper)', l2LongFrontA, l2Front2H, 1, 'l2LongsideUpper', `L2 Front upper split wall (${l2Front2H} mm)`, `${innerClearL} × ${innerF2} mm`);
        }

        addPanel('Level 2 Top', l2TopA, l2TopB, 1, 'l2Top', 'Flush to all Level 2 walls: (Side + 2T) × L', `${l2InnerClearW} × ${innerClearL} mm`);
      }

      if (isSplitL1) {
        addPanel('Internal Joint (Seam)', jointH, jointL, 2, 'joint', `L1 internal seam batten (H: ${jointH} mm, L: ${longA} - ${trim} mm)`, `${jointH} × ${Math.min(jointL, innerClearL)} mm`);
      }
      const l2JointQty = (isSplitL2Rear ? 1 : 0) + (isSplitL2Front ? 1 : 0);
      if (l2JointQty > 0) {
        addPanel('Level 2 Internal Joint (Seam)', jointH, jointL, l2JointQty, 'joint', `L2 internal seam batten (H: ${jointH} mm, L: ${outerL} - ${trim} mm)`, `${jointH} × ${Math.min(jointL, innerClearL)} mm`);
      }

      // Base Rails & 5-Block Skid Stacks
      // Rule: Base Rails sit on the ground; 5-Block Skid Stacks sit on top of the Base Rails.
      // Each base rail ALWAYS has 3 skids (2 at ends + 1 at center).
      // 3 Base Rails = 3 × 3 × 5 = 45 skid blocks; 4 Base Rails = 4 × 3 × 5 = 60 skid blocks.
      const railWidth = 100;
      const blockSize = 100;
      const blocksPerStack = 5;
      const stacksPerRail = 3;
      const railLength = numRails === 3 ? outerL : outerW;
      const railSpanDimension = numRails === 3 ? outerW : outerL;
      const clearGapBetweenRails = Math.round((railSpanDimension - numRails * railWidth) / (numRails - 1));
      const totalStacks = numRails * stacksPerRail;
      const totalBlocks = totalStacks * blocksPerStack;

      const supportH = 6 * T;
      const totalBoxStructureH = boxH + (levels === 2 ? (l2InnerClearH + 2 * T) : 0);
      const totalH = totalBoxStructureH + supportH;

      const totalPanelAreaM2 = panels.reduce((acc, p) => acc + p.areaM2, 0);
      const totalPanelCount = panels.reduce((acc, p) => acc + p.qty, 0);

      const baseComponents = [
        {
          name: `Base Rail (${numRails === 3 ? 'Lengthwise' : 'Widthwise'})`,
          width: railWidth,
          length: railLength,
          thickness: T,
          qty: numRails,
          note: `Ground runner flush with ${numRails === 3 ? 'Length' : 'Width'} (${railLength} mm); clear gap: ${clearGapBetweenRails} mm`
        },
        {
          name: 'Skid Block (5-Block Stack)',
          width: blockSize,
          length: blockSize,
          thickness: T,
          qty: totalBlocks,
          note: `Above Base Rails: ${numRails} rails × 3 skids/rail (2 ends + 1 center) × 5 layers = ${totalBlocks} pcs (${5 * T} mm)`
        }
      ];

      const nesting = compute2DSheetNesting(panels, state.sawKerf);

      return {
        T,
        levels,
        numRails,
        outerL,
        outerW,
        boxH,
        l2WallH,
        l2OuterW,
        l2ShortLen,
        l2InnerClearW,
        l2InnerClearH,
        l2BottomA,
        l2BottomB,
        l2ShortA,
        l2ShortB,
        l2LongRearA,
        l2LongRearB,
        l2LongFrontA,
        l2LongFrontB,
        l2TopA,
        l2TopB,
        totalBoxStructureH,
        innerClearL,
        innerClearW,
        innerClearH,
        isSplitL1,
        long1H,
        long2H,
        isSplitL2: isSplitL2Rear || isSplitL2Front,
        isSplitL2Rear,
        isSplitL2Front,
        l2Rear1H,
        l2Rear2H,
        l2Front1H,
        l2Front2H,
        jointH,
        jointL,
        trim,
        bottomA,
        bottomB,
        shortA,
        shortB,
        longA,
        longB,
        topA,
        topB,
        railWidth,
        railLength,
        clearGapBetweenRails,
        blockSize,
        blocksPerStack,
        stacksPerRail,
        totalStacks,
        totalBlocks,
        supportH,
        totalH,
        panels,
        baseComponents,
        totalPanelAreaM2,
        totalPanelCount,
        nesting
      };
    }

    // Guillotine Shelf 2D Sheet-Nesting Planner (1220 x 2240 mm) + Offcut Tracker
    function compute2DSheetNesting(panels, kerf) {
      const items = [];
      const oversizeParts = [];

      panels.forEach(p => {
        for (let i = 0; i < p.qty; i++) {
          const fitsNormal = p.sideA <= SHEET_W && p.sideB <= SHEET_L;
          const fitsRotated = p.sideB <= SHEET_W && p.sideA <= SHEET_L;
          if (!fitsNormal && !fitsRotated) {
            if (!oversizeParts.includes(`${p.name} (${p.sideA}×${p.sideB})`)) {
              oversizeParts.push(`${p.name} (${p.sideA}×${p.sideB})`);
            }
          }
          items.push({
            name: p.name,
            rawW: Math.min(p.sideA, SHEET_W),
            rawH: Math.min(p.sideB, SHEET_L),
            origA: p.sideA,
            origB: p.sideB,
            colorKey: p.colorKey
          });
        }
      });

      items.sort((a, b) => (b.rawW * b.rawH) - (a.rawW * a.rawH));

      const sheets = [];
      const newSheet = () => ({
        index: sheets.length + 1,
        shelves: [],
        placements: [],
        usedArea: 0
      });

      items.forEach(item => {
        const orientations = [];
        if (item.rawW <= SHEET_W && item.rawH <= SHEET_L) {
          orientations.push({ w: item.rawW, h: item.rawH });
        }
        if (item.rawH <= SHEET_W && item.rawW <= SHEET_L && item.rawW !== item.rawH) {
          orientations.push({ w: item.rawH, h: item.rawW });
        }
        if (orientations.length === 0) {
          orientations.push({ w: Math.min(item.rawW, SHEET_W), h: Math.min(item.rawH, SHEET_L) });
        }

        let placed = false;
        for (const sheet of sheets) {
          for (const shelf of sheet.shelves) {
            for (const o of orientations) {
              const needW = shelf.usedW === 0 ? o.w : o.w + kerf;
              if (shelf.usedW + needW <= SHEET_W && o.h <= shelf.height) {
                const px = shelf.usedW === 0 ? 0 : shelf.usedW + kerf;
                sheet.placements.push({
                  name: item.name,
                  x: px,
                  y: shelf.y,
                  w: o.w,
                  h: o.h,
                  origA: item.origA,
                  origB: item.origB,
                  colorKey: item.colorKey
                });
                shelf.usedW = px + o.w;
                sheet.usedArea += o.w * o.h;
                placed = true;
                break;
              }
            }
            if (placed) break;
          }
          if (placed) break;

          const usedH = sheet.shelves.reduce((acc, s) => acc + s.height + kerf, 0);
          for (const o of orientations) {
            if (usedH + o.h <= SHEET_L && o.w <= SHEET_W) {
              const shelf = { y: usedH, height: o.h, usedW: o.w };
              sheet.shelves.push(shelf);
              sheet.placements.push({
                name: item.name,
                x: 0,
                y: shelf.y,
                w: o.w,
                h: o.h,
                origA: item.origA,
                origB: item.origB,
                colorKey: item.colorKey
              });
              sheet.usedArea += o.w * o.h;
              placed = true;
              break;
            }
          }
          if (placed) break;
        }

        if (!placed) {
          const sheet = newSheet();
          orientations.sort((a, b) => a.h - b.h);
          const o = orientations[0];
          sheet.shelves.push({ y: 0, height: o.h, usedW: o.w });
          sheet.placements.push({
            name: item.name,
            x: 0,
            y: 0,
            w: o.w,
            h: o.h,
            origA: item.origA,
            origB: item.origB,
            colorKey: item.colorKey
          });
          sheet.usedArea += o.w * o.h;
          sheets.push(sheet);
        }
      });

      const offcuts = [];
      sheets.forEach(sheet => {
        let cursorY = 0;
        sheet.shelves.forEach(shelf => {
          const remW = SHEET_W - shelf.usedW - (shelf.usedW > 0 ? kerf : 0);
          if (remW >= 80 && shelf.height >= 80) {
            offcuts.push({
              sheetIndex: sheet.index,
              w: Math.round(remW),
              h: Math.round(shelf.height),
              areaM2: (remW * shelf.height) / 1_000_000,
              usableForBlocks: remW >= 100 && shelf.height >= 100
            });
          }
          cursorY = shelf.y + shelf.height + kerf;
        });
        const remH = SHEET_L - cursorY;
        if (remH >= 80) {
          offcuts.push({
            sheetIndex: sheet.index,
            w: SHEET_W,
            h: Math.round(remH),
            areaM2: (SHEET_W * remH) / 1_000_000,
            usableForBlocks: remH >= 100
          });
        }
      });

      const sheetCount = Math.max(1, sheets.length);
      const totalStockAreaM2 = (sheetCount * SHEET_W * SHEET_L) / 1_000_000;
      const totalUsedAreaM2 = sheets.reduce((acc, s) => acc + s.usedArea, 0) / 1_000_000;
      const yieldPct = Math.min(100, (totalUsedAreaM2 / totalStockAreaM2) * 100);

      return {
        sheets,
        sheetCount,
        totalStockAreaM2,
        yieldPct,
        oversizeParts,
        offcuts
      };
    }

    // =========================================================================
    // 5. THREE.JS 3D SCENE, CAMERA, RAYCASTING & INTERACTION
    // =========================================================================
    let scene, camera, renderer, crateGroup, dimensionGroup, forkGuideGroup, highlightOverlayGroup, floorGrid;
    const containerEl = document.getElementById('canvas-container');

    const orbit = {
      theta: Math.PI / 4,
      phi: Math.PI / 3.1,
      radius: 4600,
      target: new THREE.Vector3(0, 720, 0),
      isDragging: false,
      isPanning: false,
      prevX: 0,
      prevY: 0,
      pinchDist: 0,
      downX: 0,
      downY: 0
    };

    function initThreeScene() {
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x1e293b);
      scene.fog = new THREE.Fog(0x1e293b, 9500, 26000);

      camera = new THREE.PerspectiveCamera(42, containerEl.clientWidth / containerEl.clientHeight, 20, 45000);

      renderer = new THREE.WebGLRenderer({ antialias: true, preserveDrawingBuffer: true });
      renderer.setSize(containerEl.clientWidth, containerEl.clientHeight);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
      renderer.shadowMap.enabled = true;
      renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      renderer.outputEncoding = THREE.sRGBEncoding;
      containerEl.appendChild(renderer.domElement);

      const hemiLight = new THREE.HemisphereLight(0xf8fafc, 0x334155, 0.85);
      hemiLight.position.set(0, 5000, 0);
      scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xfffbeb, 0.95);
      dirLight.position.set(2800, 4800, 3200);
      dirLight.castShadow = true;
      dirLight.shadow.mapSize.width = 2048;
      dirLight.shadow.mapSize.height = 2048;
      dirLight.shadow.camera.near = 500;
      dirLight.shadow.camera.far = 14000;
      const d = 3200;
      dirLight.shadow.camera.left = -d;
      dirLight.shadow.camera.right = d;
      dirLight.shadow.camera.top = d;
      dirLight.shadow.camera.bottom = -d;
      dirLight.shadow.bias = -0.0005;
      scene.add(dirLight);

      const rimLight = new THREE.DirectionalLight(0xbae6fd, 0.42);
      rimLight.position.set(-3000, 2200, -2800);
      scene.add(rimLight);

      const floorGeo = new THREE.PlaneGeometry(24000, 24000);
      const floorMat = new THREE.MeshStandardMaterial({ color: 0x0f172a, roughness: 0.92, metalness: 0.05 });
      const floorMesh = new THREE.Mesh(floorGeo, floorMat);
      floorMesh.rotation.x = -Math.PI / 2;
      floorMesh.position.y = -1;
      floorMesh.receiveShadow = true;
      scene.add(floorMesh);

      floorGrid = new THREE.GridHelper(12000, 24, 0x475569, 0x334155);
      floorGrid.position.y = 0;
      scene.add(floorGrid);

      crateGroup = new THREE.Group();
      scene.add(crateGroup);

      dimensionGroup = new THREE.Group();
      scene.add(dimensionGroup);

      forkGuideGroup = new THREE.Group();
      scene.add(forkGuideGroup);

      highlightOverlayGroup = new THREE.Group();
      scene.add(highlightOverlayGroup);

      updateCameraPosition();
      setupPointerControls();
      window.addEventListener('resize', onWindowResize);
    }

    function updateCameraPosition() {
      const sinPhi = Math.sin(orbit.phi);
      const cosPhi = Math.cos(orbit.phi);
      const sinTheta = Math.sin(orbit.theta);
      const cosTheta = Math.cos(orbit.theta);

      camera.position.set(
        orbit.target.x + orbit.radius * sinPhi * sinTheta,
        orbit.target.y + orbit.radius * cosPhi,
        orbit.target.z + orbit.radius * sinPhi * cosTheta
      );
      camera.lookAt(orbit.target);
    }

    function setupPointerControls() {
      const dom = renderer.domElement;

      dom.addEventListener('pointerdown', e => {
        orbit.downX = e.clientX;
        orbit.downY = e.clientY;
        orbit.prevX = e.clientX;
        orbit.prevY = e.clientY;
        if (e.button === 2 || e.shiftKey) {
          orbit.isPanning = true;
        } else {
          orbit.isDragging = true;
        }
      });

      window.addEventListener('pointermove', e => {
        if (!orbit.isDragging && !orbit.isPanning) return;
        const dx = e.clientX - orbit.prevX;
        const dy = e.clientY - orbit.prevY;
        orbit.prevX = e.clientX;
        orbit.prevY = e.clientY;

        if (orbit.isDragging) {
          orbit.theta -= dx * 0.0075;
          orbit.phi = Math.max(0.08, Math.min(Math.PI - 0.08, orbit.phi - dy * 0.0075));
        } else if (orbit.isPanning) {
          const panSpeed = orbit.radius * 0.0012;
          const right = new THREE.Vector3(Math.cos(orbit.theta), 0, -Math.sin(orbit.theta));
          orbit.target.addScaledVector(right, -dx * panSpeed);
          orbit.target.y = Math.max(0, orbit.target.y + dy * panSpeed);
        }
        updateCameraPosition();
      });

      window.addEventListener('pointerup', e => {
        const moved = Math.hypot(e.clientX - orbit.downX, e.clientY - orbit.downY);
        if (orbit.isDragging && moved < 6) {
          handleCanvasTap(e.clientX, e.clientY);
        }
        orbit.isDragging = false;
        orbit.isPanning = false;
      });

      dom.addEventListener('contextmenu', e => e.preventDefault());

      dom.addEventListener('wheel', e => {
        e.preventDefault();
        const zoomFactor = e.deltaY > 0 ? 1.09 : 0.91;
        orbit.radius = Math.max(900, Math.min(18000, orbit.radius * zoomFactor));
        updateCameraPosition();
      }, { passive: false });

      dom.addEventListener('touchstart', e => {
        if (e.touches.length === 2) {
          orbit.isDragging = false;
          orbit.pinchDist = Math.hypot(
            e.touches[0].clientX - e.touches[1].clientX,
            e.touches[0].clientY - e.touches[1].clientY
          );
        }
      }, { passive: true });

      dom.addEventListener('touchmove', e => {
        if (e.touches.length === 2) {
          const dist = Math.hypot(
            e.touches[0].clientX - e.touches[1].clientX,
            e.touches[0].clientY - e.touches[1].clientY
          );
          if (orbit.pinchDist > 0) {
            const ratio = orbit.pinchDist / dist;
            orbit.radius = Math.max(900, Math.min(18000, orbit.radius * ratio));
            updateCameraPosition();
          }
          orbit.pinchDist = dist;
        }
      }, { passive: true });
    }

    const raycaster = new THREE.Raycaster();
    const pointerNDC = new THREE.Vector2();

    function handleCanvasTap(clientX, clientY) {
      const rect = renderer.domElement.getBoundingClientRect();
      pointerNDC.x = ((clientX - rect.left) / rect.width) * 2 - 1;
      pointerNDC.y = -((clientY - rect.top) / rect.height) * 2 + 1;

      raycaster.setFromCamera(pointerNDC, camera);
      const meshes = crateGroup.children.filter(obj => obj.isMesh && obj.visible);
      const hits = raycaster.intersectObjects(meshes, false);

      if (hits.length > 0 && hits[0].object.userData.partMeta) {
        selectCratePart(hits[0].object);
      } else {
        clearPartSelection();
      }
    }

    function createInternalHighlightOverlay(mesh) {
      while (highlightOverlayGroup.children.length > 0) {
        const old = highlightOverlayGroup.children[0];
        if (old.geometry) old.geometry.dispose();
        highlightOverlayGroup.remove(old);
      }

      const meta = mesh && mesh.userData ? mesh.userData.partMeta : null;
      if (!meta || !meta.internalOverlaySpec) return;

      const spec = meta.internalOverlaySpec;
      const overlayGeo = new THREE.BoxGeometry(spec.w, spec.h, spec.d);
      const overlayMesh = new THREE.Mesh(overlayGeo, materials.internalZoneOverlay);
      const localOffset = new THREE.Vector3(spec.offsetX || 0, spec.offsetY || 0, spec.offsetZ || 0);
      overlayMesh.userData = { localOffset };
      overlayMesh.position.copy(mesh.position).add(localOffset);

      const borderEdges = new THREE.EdgesGeometry(overlayGeo);
      const borderMat = new THREE.LineBasicMaterial({ color: 0x0a1e3d, transparent: true, opacity: 0.95 });
      const borderSegs = new THREE.LineSegments(borderEdges, borderMat);
      overlayMesh.add(borderSegs);

      highlightOverlayGroup.add(overlayMesh);
      highlightOverlayGroup.visible = mesh.visible;
    }

    function selectCratePart(targetMesh) {
      const meta = targetMesh.userData.partMeta;
      state.selectedPartName = meta.name;
      state.selectedPartMesh = targetMesh;

      const isInternal = state.crateType === 'Internal Crate';
      const defaultOpacity = state.renderMode === 'xray' ? 0.34 : 1.0;

      crateGroup.children.forEach(m => {
        if (!m.isMesh || !m.userData.partMeta) return;
        m.material = m.userData.baseMaterial;
        m.material.transparent = state.renderMode === 'xray';
        m.material.opacity = defaultOpacity;
        m.material.needsUpdate = true;
      });

      if (isInternal && meta.internalOverlaySpec) {
        createInternalHighlightOverlay(targetMesh);
      } else {
        while (highlightOverlayGroup.children.length > 0) {
          const old = highlightOverlayGroup.children[0];
          if (old.geometry) old.geometry.dispose();
          highlightOverlayGroup.remove(old);
        }
        targetMesh.material = materials.highlight;
      }

      const modeBadge = document.getElementById('insp-mode-badge');
      const sizeLabel = document.getElementById('insp-size-label');
      const sizeValue = document.getElementById('insp-size');

      if (isInternal) {
        modeBadge.textContent = 'Selected Part • Inside Clear';
        sizeLabel.textContent = 'Inside Size:';
        sizeValue.textContent = meta.internalSize || meta.size;
      } else {
        modeBadge.textContent = 'Selected Part • Outside';
        sizeLabel.textContent = 'Outside Size:';
        sizeValue.textContent = meta.size;
      }

      document.getElementById('insp-name').textContent = meta.name;
      document.getElementById('insp-thick').textContent = `${meta.thickness} mm`;
      document.getElementById('insp-qty').textContent = `${meta.qty} pc${meta.qty > 1 ? 's' : ''}`;
      document.getElementById('insp-note').textContent = meta.note;
      document.getElementById('part-inspector').classList.remove('hidden');
    }

    function clearPartSelection() {
      state.selectedPartName = null;
      state.selectedPartMesh = null;
      const defaultOpacity = state.renderMode === 'xray' ? 0.34 : 1.0;
      if (crateGroup) {
        crateGroup.children.forEach(m => {
          if (m.isMesh && m.userData.baseMaterial) {
            m.material = m.userData.baseMaterial;
            m.material.transparent = state.renderMode === 'xray';
            m.material.opacity = defaultOpacity;
            m.material.needsUpdate = true;
          }
        });
      }
      if (highlightOverlayGroup) {
        while (highlightOverlayGroup.children.length > 0) {
          const old = highlightOverlayGroup.children[0];
          if (old.geometry) old.geometry.dispose();
          highlightOverlayGroup.remove(old);
        }
      }
      document.getElementById('part-inspector').classList.add('hidden');
    }

    // =========================================================================
    // 6. 3D CRATE BUILDER, DIMENSION CALLOUTS & FORK CLEARANCE GUIDES
    // =========================================================================
    function createBoxPart(w, h, d, x, y, z, material, explodeVec, stepOrder, partMeta) {
      const geo = new THREE.BoxGeometry(w, h, d);
      const mesh = new THREE.Mesh(geo, material);
      mesh.position.set(x, y, z);
      mesh.castShadow = true;
      mesh.receiveShadow = true;

      const edges = new THREE.EdgesGeometry(geo);
      const lineSegs = new THREE.LineSegments(edges, materials.edgeLine);
      mesh.add(lineSegs);

      mesh.userData = {
        basePos: new THREE.Vector3(x, y, z),
        explodeDir: new THREE.Vector3(explodeVec[0], explodeVec[1], explodeVec[2]),
        stepOrder,
        baseMaterial: material,
        partMeta
      };

      crateGroup.add(mesh);
      return mesh;
    }

    function rebuild3DCrate() {
      const calc = computeCrateEngineering();
      state.lastCalc = calc;

      while (crateGroup.children.length > 0) {
        const obj = crateGroup.children[0];
        if (obj.geometry) obj.geometry.dispose();
        crateGroup.remove(obj);
      }

      const {
        T, levels, numRails, outerL, outerW, boxH, l2WallH, l2OuterW, l2ShortLen,
        l2InnerClearW, l2InnerClearH, l2BottomA, l2BottomB, l2ShortA, l2ShortB,
        l2LongRearA, l2LongRearB, l2LongFrontA, l2LongFrontB, l2TopA, l2TopB,
        innerClearL, innerClearW, innerClearH,
        isSplitL1, long1H, long2H, isSplitL2Rear, isSplitL2Front,
        l2Rear1H, l2Rear2H, l2Front1H, l2Front2H, jointH, jointL,
        bottomA, bottomB, shortA, shortB, longA, longB, topA, topB,
        railWidth, railLength, blockSize, blocksPerStack, stacksPerRail, totalBlocks,
        supportH, totalH
      } = calc;

      const railH = T;
      const stackH = blocksPerStack * T;
      const railCenterY = railH / 2;

      const railPositions = [];
      if (numRails === 3) {
        const offsetZ = outerW / 2 - railWidth / 2;
        railPositions.push(-offsetZ, 0, offsetZ);
      } else {
        const span = outerL - railWidth;
        for (let i = 0; i < 4; i++) {
          railPositions.push(-span / 2 + (span / 3) * i);
        }
      }

      // Step 1: Base Rails on the ground (y = 0 to T)
      railPositions.forEach(pos => {
        if (numRails === 3) {
          createBoxPart(
            railLength, railH, railWidth,
            0, railCenterY, pos,
            materials.rail,
            [0, -260, pos * 0.45],
            1,
            {
              name: 'Base Rail (3-Rail Lengthwise)',
              size: `${railWidth} × ${railLength} mm`,
              internalSize: `${railWidth} × ${innerClearL} mm`,
              thickness: T,
              qty: 3,
              note: `Ground runner flush with Length (${railLength} mm). Clear gap between rails: ${calc.clearGapBetweenRails} mm.`
            }
          );
        } else {
          createBoxPart(
            railWidth, railH, railLength,
            pos, railCenterY, 0,
            materials.rail,
            [pos * 0.45, -260, 0],
            1,
            {
              name: 'Base Rail (4-Rail Widthwise)',
              size: `${railWidth} × ${railLength} mm`,
              internalSize: `${railWidth} × ${innerClearW} mm`,
              thickness: T,
              qty: 4,
              note: `Ground runner flush with Width (${railLength} mm). Clear gap between rails: ${calc.clearGapBetweenRails} mm.`
            }
          );
        }
      });

      // Step 2: 5-Block Skid Stacks on top of Base Rails (y = T to 6T; 3 skids per base rail: 2 ends + 1 center)
      const stackOffsetsAlongRail = [
        -railLength / 2 + blockSize / 2,
        0,
        railLength / 2 - blockSize / 2
      ];

      railPositions.forEach(rPos => {
        stackOffsetsAlongRail.forEach(sPos => {
          const bx = numRails === 3 ? sPos : rPos;
          const bz = numRails === 3 ? rPos : sPos;

          for (let layer = 0; layer < blocksPerStack; layer++) {
            const by = railH + layer * T + T / 2;
            createBoxPart(
              blockSize, T, blockSize,
              bx, by, bz,
              materials.block,
              [bx * 0.35, -180 - (4 - layer) * 28, bz * 0.35],
              2,
              {
                name: 'Skid Block (5-Block Stack)',
                size: `${blockSize} × ${blockSize} mm`,
                internalSize: `${blockSize} × ${blockSize} mm`,
                thickness: T,
                qty: totalBlocks,
                note: `Mounted above Base Rails: 3 skids per rail (2 ends + 1 center) × ${numRails} rails × 5 layers = ${totalBlocks} pcs (${stackH} mm).`
              }
            );
          }
        });
      });

      // Step 3: Bottom Panel (Fitted inside Longsides & Shortsides on top of Skids)
      const bottomY = supportH + T / 2;
      createBoxPart(
        bottomB, T, bottomA,
        0, bottomY, 0,
        materials.bottom,
        [0, -75, 0],
        3,
        {
          name: 'Bottom Panel',
          size: `${bottomA} × ${bottomB} mm`,
          internalSize: `${innerClearW} × ${innerClearL} mm`,
          internalOverlaySpec: { w: innerClearL, h: T + 3, d: innerClearW, offsetX: 0, offsetY: 0, offsetZ: 0 },
          thickness: T,
          qty: 1,
          note: `Base floor panel fitted inside all 4 walls: (Width - 2T) × (Length - 2T) = ${bottomA} × ${bottomB} mm.`
        }
      );

      // Step 4: Longside (Rear)
      const wallCenterY = supportH + longB / 2;
      const rearZ = -outerW / 2 + T / 2;

      if (!isSplitL1) {
        createBoxPart(
          longA, longB, T,
          0, wallCenterY, rearZ,
          materials.longside,
          [0, 30, -340],
          4,
          {
            name: 'Longside (Rear)',
            size: `${longA} × ${longB} mm`,
            internalSize: `${innerClearL} × ${innerClearH} mm`,
            internalOverlaySpec: { w: innerClearL, h: innerClearH, d: T + 3, offsetX: 0, offsetY: T / 2, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Full-height side wall cut to Length × (Height - T) = ${longA} × ${longB} mm.`
          }
        );
      } else {
        const lowerCenterY = supportH + long1H / 2;
        const upperCenterY = supportH + long1H + long2H / 2;
        const innerL1H = Math.max(1, long1H - T);
        const innerL2H = Math.max(1, innerClearH - innerL1H);
        const splitLocalY = -long2H / 2 + innerL2H / 2;

        createBoxPart(
          longA, long1H, T,
          0, lowerCenterY, rearZ,
          materials.longside,
          [0, 15, -340],
          4,
          {
            name: 'Longside 1 Lower (Rear)',
            size: `${longA} × ${long1H} mm`,
            internalSize: `${innerClearL} × ${innerL1H} mm`,
            internalOverlaySpec: { w: innerClearL, h: innerL1H, d: T + 3, offsetX: 0, offsetY: T / 2, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Lower split wall (${long1H} mm >= Longside 2 ${long2H} mm; max 1220 mm).`
          }
        );
        createBoxPart(
          longA, long2H, T,
          0, upperCenterY, rearZ,
          materials.longsideUpper,
          [0, 95, -380],
          4,
          {
            name: 'Longside 2 Upper (Rear)',
            size: `${longA} × ${long2H} mm`,
            internalSize: `${innerClearL} × ${innerL2H} mm`,
            internalOverlaySpec: { w: innerClearL, h: innerL2H, d: T + 3, offsetX: 0, offsetY: splitLocalY, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Upper split wall (${long2H} mm) mounted above Longside 1.`
          }
        );
      }

      // Step 5: Short Sides (Left & Right)
      const shortCenterY = supportH + shortB / 2;
      const leftX = -outerL / 2 + T / 2;
      const rightX = outerL / 2 - T / 2;

      createBoxPart(
        T, shortB, shortA,
        leftX, shortCenterY, 0,
        materials.shortside,
        [-340, 30, 0],
        5,
        {
          name: 'Shortside (Left)',
          size: `${shortA} × ${shortB} mm`,
          internalSize: `${innerClearW} × ${innerClearH} mm`,
          internalOverlaySpec: { w: T + 3, h: innerClearH, d: innerClearW, offsetX: 0, offsetY: T / 2, offsetZ: 0 },
          thickness: T,
          qty: 2,
          note: `End wall fitted inside Longsides: (W - 2T) × (H - T) = ${shortA} × ${shortB} mm.`
        }
      );

      createBoxPart(
        T, shortB, shortA,
        rightX, shortCenterY, 0,
        materials.shortside,
        [340, 30, 0],
        5,
        {
          name: 'Shortside (Right)',
          size: `${shortA} × ${shortB} mm`,
          internalSize: `${innerClearW} × ${innerClearH} mm`,
          internalOverlaySpec: { w: T + 3, h: innerClearH, d: innerClearW, offsetX: 0, offsetY: T / 2, offsetZ: 0 },
          thickness: T,
          qty: 2,
          note: `End wall fitted inside Longsides: (W - 2T) × (H - T) = ${shortA} × ${shortB} mm.`
        }
      );

      // Step 6: Longside (Front) + Internal Joint Battens (when split)
      const frontZ = outerW / 2 - T / 2;
      if (!isSplitL1) {
        createBoxPart(
          longA, longB, T,
          0, wallCenterY, frontZ,
          materials.longside,
          [0, 30, 340],
          6,
          {
            name: 'Longside (Front)',
            size: `${longA} × ${longB} mm`,
            internalSize: `${innerClearL} × ${innerClearH} mm`,
            internalOverlaySpec: { w: innerClearL, h: innerClearH, d: T + 3, offsetX: 0, offsetY: T / 2, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Full-height side wall cut to Length × (Height - T) = ${longA} × ${longB} mm.`
          }
        );
      } else {
        const lowerCenterY = supportH + long1H / 2;
        const upperCenterY = supportH + long1H + long2H / 2;
        const innerL1H = Math.max(1, long1H - T);
        const innerL2H = Math.max(1, innerClearH - innerL1H);
        const splitLocalY = -long2H / 2 + innerL2H / 2;

        createBoxPart(
          longA, long1H, T,
          0, lowerCenterY, frontZ,
          materials.longside,
          [0, 15, 340],
          6,
          {
            name: 'Longside 1 Lower (Front)',
            size: `${longA} × ${long1H} mm`,
            internalSize: `${innerClearL} × ${innerL1H} mm`,
            internalOverlaySpec: { w: innerClearL, h: innerL1H, d: T + 3, offsetX: 0, offsetY: T / 2, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Lower split wall (${long1H} mm >= Longside 2 ${long2H} mm; max 1220 mm).`
          }
        );
        createBoxPart(
          longA, long2H, T,
          0, upperCenterY, frontZ,
          materials.longsideUpper,
          [0, 95, 380],
          6,
          {
            name: 'Longside 2 Upper (Front)',
            size: `${longA} × ${long2H} mm`,
            internalSize: `${innerClearL} × ${innerL2H} mm`,
            internalOverlaySpec: { w: innerClearL, h: innerL2H, d: T + 3, offsetX: 0, offsetY: splitLocalY, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Upper split wall (${long2H} mm) mounted above Longside 1.`
          }
        );

        const seamY = supportH + long1H;
        createBoxPart(
          jointL, jointH, T,
          0, seamY, rearZ + T,
          materials.joint,
          [0, 55, -240],
          4,
          {
            name: 'Internal Joint (Rear)',
            size: `${jointH} × ${jointL} mm`,
            internalSize: `${jointH} × ${Math.min(jointL, innerClearL)} mm`,
            thickness: T,
            qty: 2,
            note: `Internal seam batten centered on Rear Longside 1 & 2 joint (${longA} - ${calc.trim} mm).`
          }
        );
        createBoxPart(
          jointL, jointH, T,
          0, seamY, frontZ - T,
          materials.joint,
          [0, 55, 240],
          6,
          {
            name: 'Internal Joint (Front)',
            size: `${jointH} × ${jointL} mm`,
            internalSize: `${jointH} × ${Math.min(jointL, innerClearL)} mm`,
            thickness: T,
            qty: 2,
            note: `Internal seam batten centered on Front Longside 1 & 2 joint (${longA} - ${calc.trim} mm).`
          }
        );
      }

      // Step 7: Top Lid (Level 1)
      const topY = supportH + boxH - T / 2;
      createBoxPart(
        topB, T, topA,
        0, topY, 0,
        materials.top,
        [0, 260, 0],
        7,
        {
          name: levels === 2 ? 'Level 1 Top (Deck)' : 'Top Panel',
          size: `${topA} × ${topB} mm`,
          internalSize: `${innerClearW} × ${innerClearL} mm`,
          internalOverlaySpec: { w: innerClearL, h: T + 3, d: innerClearW, offsetX: 0, offsetY: 0, offsetZ: 0 },
          thickness: T,
          qty: 1,
          note: levels === 2
            ? `Full-coverage Level 1 top deck (W × L = ${topA} × ${topB} mm) supporting Level 2.`
            : `Full-coverage top lid overlapping all four walls: Width × Length = ${topA} × ${topB} mm.`
        }
      );

      // =====================================================================
      // LEVEL 2 ASSEMBLY (Steps 8–12 — Zero Overlap)
      // - Level 2 Bottom (Middle): width equals Shortside width + Rear Longside thickness (Side + T), length equals Top length (L), flush to rear of Level 1 Top.
      // - Level 2 Longside (Rear): exact height of Level 2 Shortsides (H2), sitting on rear edge of Level 2 Bottom: L × H2.
      // - Level 2 Shortsides (Left & Right): internal height H2 above Level 2 Bottom: Side × H2.
      // - Level 2 Longside Front: from Level 1 Top surface to the top edge of Level 2 Top: L × (H2 + 2T).
      // - Level 2 Top: flush over all 4 Level 2 walls: (Side + 2T) × L.
      // =====================================================================
      if (levels === 2) {
        const l1TopSurfaceY = supportH + boxH;
        const l2RearZ = -outerW / 2 + T / 2;
        const l2FrontZ = l2RearZ + l2ShortLen + T;
        const l2ShortCenterZ = l2RearZ + T / 2 + l2ShortLen / 2;
        const l2BottomCenterZ = -outerW / 2 + l2BottomA / 2;
        const l2TopCenterZ = l2BottomCenterZ;

        // Step 8: Level 2 Bottom (Middle) — width equals Shortside width + Rear Longside thickness (Side + T), same length as Tops (L), flush to rear of L1 Top
        const l2BottomY = l1TopSurfaceY + T / 2;
        createBoxPart(
          l2BottomB, T, l2BottomA,
          0, l2BottomY, l2BottomCenterZ,
          materials.l2Bottom,
          [0, 310, 0],
          8,
          {
            name: 'Level 2 Bottom (Middle)',
            size: `${l2BottomA} × ${l2BottomB} mm`,
            internalSize: `${l2InnerClearW} × ${innerClearL} mm`,
            internalOverlaySpec: { w: innerClearL, h: T + 3, d: l2InnerClearW, offsetX: 0, offsetY: 0, offsetZ: T / 2 },
            thickness: T,
            qty: 1,
            note: `Flush to rear of L1 Top, same length as Tops: (Shortside W + Rear Thick) × L = ${l2BottomA} × ${l2BottomB} mm.`
          }
        );

        // Step 9: Level 2 Longside (Rear) — exact height of Level 2 Shortsides (H2), sitting on Level 2 Bottom
        const l2RearBaseY = l1TopSurfaceY + T;
        const l2RearCenterY = l2RearBaseY + l2LongRearB / 2;

        if (!isSplitL2Rear) {
          createBoxPart(
            l2LongRearA, l2LongRearB, T,
            0, l2RearCenterY, l2RearZ,
            materials.l2Longside,
            [0, 360, -340],
            9,
            {
              name: 'Level 2 Longside (Rear)',
              size: `${l2LongRearA} × ${l2LongRearB} mm`,
              internalSize: `${innerClearL} × ${l2InnerClearH} mm`,
              internalOverlaySpec: { w: innerClearL, h: l2InnerClearH, d: T + 3, offsetX: 0, offsetY: 0, offsetZ: 0 },
              thickness: T,
              qty: 1,
              note: `Exact height of L2 Shortsides on L2 Bottom: L × H2 = ${l2LongRearA} × ${l2LongRearB} mm.`
            }
          );
        } else {
          const l2RearLowerY = l2RearBaseY + l2Rear1H / 2;
          const l2RearUpperY = l2RearBaseY + l2Rear1H + l2Rear2H / 2;

          createBoxPart(
            l2LongRearA, l2Rear1H, T,
            0, l2RearLowerY, l2RearZ,
            materials.l2Longside,
            [0, 340, -340],
            9,
            {
              name: 'Level 2 Rear 1 (Lower)',
              size: `${l2LongRearA} × ${l2Rear1H} mm`,
              internalSize: `${innerClearL} × ${l2Rear1H} mm`,
              internalOverlaySpec: { w: innerClearL, h: l2Rear1H, d: T + 3, offsetX: 0, offsetY: 0, offsetZ: 0 },
              thickness: T,
              qty: 1,
              note: `L2 Rear lower split wall (${l2Rear1H} mm >= Rear 2 ${l2Rear2H} mm).`
            }
          );
          createBoxPart(
            l2LongRearA, l2Rear2H, T,
            0, l2RearUpperY, l2RearZ,
            materials.l2LongsideUpper,
            [0, 410, -380],
            9,
            {
              name: 'Level 2 Rear 2 (Upper)',
              size: `${l2LongRearA} × ${l2Rear2H} mm`,
              internalSize: `${innerClearL} × ${l2Rear2H} mm`,
              internalOverlaySpec: { w: innerClearL, h: l2Rear2H, d: T + 3, offsetX: 0, offsetY: 0, offsetZ: 0 },
              thickness: T,
              qty: 1,
              note: `L2 Rear upper split wall (${l2Rear2H} mm).`
            }
          );

          const l2RearSeamY = l2RearBaseY + l2Rear1H;
          createBoxPart(
            jointL, jointH, T,
            0, l2RearSeamY, l2RearZ + T,
            materials.joint,
            [0, 380, -240],
            9,
            {
              name: 'Level 2 Internal Joint (Rear)',
              size: `${jointH} × ${jointL} mm`,
              internalSize: `${jointH} × ${Math.min(jointL, innerClearL)} mm`,
              thickness: T,
              qty: 1,
              note: `L2 Rear internal seam batten (${l2LongRearA} - ${calc.trim} mm).`
            }
          );
        }

        // Step 10: Level 2 Shortsides (Left & Right) — height equals internal H2 above L2 Bottom
        const l2ShortCenterY = l1TopSurfaceY + T + l2ShortB / 2;
        createBoxPart(
          T, l2ShortB, l2ShortA,
          leftX, l2ShortCenterY, l2ShortCenterZ,
          materials.l2Shortside,
          [-340, 360, 0],
          10,
          {
            name: 'Level 2 Shortside (Left)',
            size: `${l2ShortA} × ${l2ShortB} mm`,
            internalSize: `${l2InnerClearW} × ${l2InnerClearH} mm`,
            internalOverlaySpec: { w: T + 3, h: l2InnerClearH, d: l2InnerClearW, offsetX: 0, offsetY: 0, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Level 2 end wall cut to Side × H2 = ${l2ShortA} × ${l2ShortB} mm.`
          }
        );

        createBoxPart(
          T, l2ShortB, l2ShortA,
          rightX, l2ShortCenterY, l2ShortCenterZ,
          materials.l2Shortside,
          [340, 360, 0],
          10,
          {
            name: 'Level 2 Shortside (Right)',
            size: `${l2ShortA} × ${l2ShortB} mm`,
            internalSize: `${l2InnerClearW} × ${l2InnerClearH} mm`,
            internalOverlaySpec: { w: T + 3, h: l2InnerClearH, d: l2InnerClearW, offsetX: 0, offsetY: 0, offsetZ: 0 },
            thickness: T,
            qty: 2,
            note: `Level 2 end wall cut to Side × H2 = ${l2ShortA} × ${l2ShortB} mm.`
          }
        );

        // Step 12: Level 2 Longside Front — extends from L1 Top surface to the top edge of Level 2 Top
        const l2FrontCenterY = l1TopSurfaceY + l2LongFrontB / 2;
        if (!isSplitL2Front) {
          createBoxPart(
            l2LongFrontA, l2LongFrontB, T,
            0, l2FrontCenterY, l2FrontZ,
            materials.l2Longside,
            [0, 360, 340],
            12,
            {
              name: 'Level 2 Longside Front',
              size: `${l2LongFrontA} × ${l2LongFrontB} mm`,
              internalSize: `${innerClearL} × ${l2InnerClearH} mm`,
              internalOverlaySpec: { w: innerClearL, h: l2InnerClearH, d: T + 3, offsetX: 0, offsetY: 0, offsetZ: 0 },
              thickness: T,
              qty: 1,
              note: `Extends from Level 1 Top surface to top edge of Level 2 Top: L × (H2 + 2T) = ${l2LongFrontA} × ${l2LongFrontB} mm.`
            }
          );
        } else {
          const l2FrontLowerY = l1TopSurfaceY + l2Front1H / 2;
          const l2FrontUpperY = l1TopSurfaceY + l2Front1H + l2Front2H / 2;
          const innerF1 = Math.max(1, l2Front1H - T);
          const innerF2 = Math.max(1, l2InnerClearH - innerF1);
          const l2SplitLocalY = -l2Front2H / 2 + innerF2 / 2;

          createBoxPart(
            l2LongFrontA, l2Front1H, T,
            0, l2FrontLowerY, l2FrontZ,
            materials.l2Longside,
            [0, 340, 340],
            12,
            {
              name: 'Level 2 Front 1 (Lower)',
              size: `${l2LongFrontA} × ${l2Front1H} mm`,
              internalSize: `${innerClearL} × ${innerF1} mm`,
              internalOverlaySpec: { w: innerClearL, h: innerF1, d: T + 3, offsetX: 0, offsetY: T / 2, offsetZ: 0 },
              thickness: T,
              qty: 1,
              note: `L2 Front lower split wall (${l2Front1H} mm >= Front 2 ${l2Front2H} mm).`
            }
          );
          createBoxPart(
            l2LongFrontA, l2Front2H, T,
            0, l2FrontUpperY, l2FrontZ,
            materials.l2LongsideUpper,
            [0, 410, 380],
            12,
            {
              name: 'Level 2 Front 2 (Upper)',
              size: `${l2LongFrontA} × ${l2Front2H} mm`,
              internalSize: `${innerClearL} × ${innerF2} mm`,
              internalOverlaySpec: { w: innerClearL, h: innerF2, d: T + 3, offsetX: 0, offsetY: l2SplitLocalY, offsetZ: 0 },
              thickness: T,
              qty: 1,
              note: `L2 Front upper split wall (${l2Front2H} mm).`
            }
          );

          const l2FrontSeamY = l1TopSurfaceY + l2Front1H;
          createBoxPart(
            jointL, jointH, T,
            0, l2FrontSeamY, l2FrontZ - T,
            materials.joint,
            [0, 380, 240],
            12,
            {
              name: 'Level 2 Internal Joint (Front)',
              size: `${jointH} × ${jointL} mm`,
              internalSize: `${jointH} × ${Math.min(jointL, innerClearL)} mm`,
              thickness: T,
              qty: 1,
              note: `L2 Front internal seam batten (${l2LongFrontA} - ${calc.trim} mm).`
            }
          );
        }

        // Step 11: Level 2 Top — same footprint as Level 2 Bottom (Middle), flush to the rear edge
        const l2TopY = totalH - T / 2;
        createBoxPart(
          l2TopB, T, l2TopA,
          0, l2TopY, l2TopCenterZ,
          materials.l2Top,
          [0, 520, 0],
          11,
          {
            name: 'Level 2 Top',
            size: `${l2TopA} × ${l2TopB} mm`,
            internalSize: `${l2InnerClearW} × ${innerClearL} mm`,
            internalOverlaySpec: { w: innerClearL, h: T + 3, d: l2InnerClearW, offsetX: 0, offsetY: 0, offsetZ: 0 },
            thickness: T,
            qty: 1,
            note: `Same footprint as Level 2 Bottom (Middle), flush to rear edge: ${l2TopA} × ${l2TopB} mm.`
          }
        );
      }

      if (state.selectedPartName) {
        const matchingMesh = crateGroup.children.find(m => m.isMesh && m.userData.partMeta && m.userData.partMeta.name === state.selectedPartName);
        if (matchingMesh) {
          selectCratePart(matchingMesh);
        } else {
          clearPartSelection();
        }
      }

      buildDimensionLines(calc);
      buildForkGuides(calc);
      applyExplodeAndStepState();
      updateHUDAndMetrics(calc);
      refreshCutListModal(calc);
    }

    function syncExplodedViewButton() {
      const button = document.getElementById('btn-exploded-view');
      const label = document.getElementById('exploded-view-button-label');
      if (!button || !label) return;
      const active = state.explodedViewActive;
      label.textContent = active ? 'Assemble View' : 'Explode View';
      button.setAttribute('aria-pressed', active ? 'true' : 'false');
      button.className = active
        ? 'w-full py-2 px-3 rounded-lg text-xs font-semibold bg-emerald-700 text-white hover:bg-emerald-600 shadow-sm transition flex items-center justify-center gap-2'
        : 'w-full py-2 px-3 rounded-lg text-xs font-semibold bg-navy-800 text-white hover:bg-navy-700 shadow-sm transition flex items-center justify-center gap-2';
    }

    function animateExplodedView(target, duration = 950) {
      if (state.explodeAnimationFrame !== null) {
        cancelAnimationFrame(state.explodeAnimationFrame);
        state.explodeAnimationFrame = null;
      }
      const from = state.explode;
      const start = performance.now();
      const tick = now => {
        const raw = Math.min(1, (now - start) / duration);
        const eased = raw < 0.5
          ? 4 * raw * raw * raw
          : 1 - Math.pow(-2 * raw + 2, 3) / 2;
        state.explode = Math.round(from + (target - from) * eased);
        applyExplodeAndStepState({ playSound: false });
        if (raw < 1) {
          state.explodeAnimationFrame = requestAnimationFrame(tick);
        } else {
          state.explode = target;
          state.explodeAnimationFrame = null;
          applyExplodeAndStepState({ playSound: false });
        }
      };
      state.explodeAnimationFrame = requestAnimationFrame(tick);
    }

    function toggleExplodedView() {
      state.explodedViewActive = !state.explodedViewActive;
      syncExplodedViewButton();
      animateExplodedView(state.explodedViewActive ? 100 : 0, 1050);
    }

    function getExplodedViewLift() {
      if (!crateGroup) return 0;
      // Keep the lowest exploded component safely above the preview floor.
      // This is a presentation-only group offset; part dimensions and local geometry stay unchanged.
      let lowestExplodedY = Infinity;
      crateGroup.children.forEach(mesh => {
        const basePos = mesh.userData && mesh.userData.basePos;
        const explodeDir = mesh.userData && mesh.userData.explodeDir;
        const height = mesh.geometry && mesh.geometry.parameters
          ? Number(mesh.geometry.parameters.height) || 0
          : 0;
        if (!basePos || !explodeDir) return;
        lowestExplodedY = Math.min(
          lowestExplodedY,
          basePos.y + explodeDir.y - height / 2
        );
      });
      if (!Number.isFinite(lowestExplodedY)) return 0;
      return Math.max(0, 150 - lowestExplodedY);
    }

    function applyExplodeAndStepState(options = {}) {
      if (!crateGroup) return;
      const factor = state.explode / 100;
      // Float the complete crate assembly as it explodes so the downward-separated
      // skids and base rails remain visible above the preview floor. Return to zero on assemble.
      const explodedLift = getExplodedViewLift() * factor;
      crateGroup.position.y = explodedLift;
      if (highlightOverlayGroup) highlightOverlayGroup.position.y = explodedLift;
      const stepLimit = state.assemblyStep;
      let installedAnyPart = false;
      const previousStep = Number.isFinite(options.previousStep) ? options.previousStep : stepLimit;
      const animateNewParts = options.animateNewParts === true;

      crateGroup.children.forEach(mesh => {
        const { basePos, explodeDir, stepOrder, partMeta } = mesh.userData;
        if (basePos && explodeDir) {
          const targetPos = basePos.clone().addScaledVector(explodeDir, factor);
          const isJointPart = partMeta && partMeta.name && partMeta.name.includes('Internal Joint');
          const hiddenByJointToggle = isJointPart && !state.showJoints;
          const hiddenByToggle = partMeta && state.hiddenParts.has(partMeta.name);
          const hiddenByStep = (stepOrder || 1) > stepLimit;
          const shouldShow = !hiddenByJointToggle && !hiddenByToggle && !hiddenByStep;
          mesh.visible = shouldShow;
          mesh.userData.motion = null;

          if (shouldShow && animateNewParts && (stepOrder || 1) > previousStep && (stepOrder || 1) <= stepLimit) {
            let direction = explodeDir.clone();
            if (direction.lengthSq() < 0.0001) direction.set(basePos.x, Math.max(1, basePos.y), basePos.z);
            if (direction.lengthSq() < 0.0001) direction.set(0, 1, 0);
            direction.normalize();
            const crateSize = state.lastCalc
              ? Math.max(state.lastCalc.outerL || 0, state.lastCalc.outerW || 0, state.lastCalc.totalH || 0)
              : 1000;
            const travel = Math.max(85, Math.min(190, crateSize * 0.065));
            mesh.position.copy(targetPos).addScaledVector(direction, travel);
            mesh.userData.motion = {
              from: mesh.position.clone(),
              to: targetPos,
              start: performance.now(),
              duration: 880
            };
            installedAnyPart = true;
          } else {
            mesh.position.copy(targetPos);
          }
        }
      });

      if (highlightOverlayGroup && state.selectedPartMesh) {
        const activeMesh = state.selectedPartMesh;
        highlightOverlayGroup.visible = activeMesh.visible;
        highlightOverlayGroup.children.forEach(overlay => {
          const localOffset = overlay.userData && overlay.userData.localOffset
            ? overlay.userData.localOffset
            : new THREE.Vector3(0, 0, 0);
          overlay.position.copy(activeMesh.position).add(localOffset);
        });
      }
      if (installedAnyPart && options.playSound !== false) playWorkshopAssemblySound();
    }

    function updatePartMotionAnimations() {
      if (!crateGroup) return;
      const now = performance.now();
      crateGroup.children.forEach(mesh => {
        const motion = mesh.userData && mesh.userData.motion;
        if (!motion) return;
        const raw = Math.min(1, Math.max(0, (now - motion.start) / motion.duration));
        // Smooth ease-out with a tiny settling finish; no geometry or dimensions are changed.
        const eased = 1 - Math.pow(1 - raw, 3);
        mesh.position.lerpVectors(motion.from, motion.to, eased);
        if (raw >= 1) {
          mesh.position.copy(motion.to);
          mesh.userData.motion = null;
        }
      });
      if (highlightOverlayGroup && state.selectedPartMesh) {
        const activeMesh = state.selectedPartMesh;
        highlightOverlayGroup.visible = activeMesh.visible;
        highlightOverlayGroup.children.forEach(overlay => {
          const localOffset = overlay.userData && overlay.userData.localOffset
            ? overlay.userData.localOffset
            : new THREE.Vector3(0, 0, 0);
          overlay.position.copy(activeMesh.position).add(localOffset);
        });
      }
    }

    function makeTextSprite(text, bgColor = 'rgba(15, 23, 42, 0.88)', textColor = '#38bdf8') {
      const canvas = document.createElement('canvas');
      canvas.width = 512;
      canvas.height = 112;
      const ctx = canvas.getContext('2d');

      ctx.fillStyle = bgColor;
      ctx.beginPath();
      ctx.roundRect(12, 12, 488, 88, 22);
      ctx.fill();

      ctx.strokeStyle = 'rgba(255, 255, 255, 0.25)';
      ctx.lineWidth = 3;
      ctx.stroke();

      ctx.fillStyle = textColor;
      ctx.font = 'bold 34px JetBrains Mono, monospace';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(text, 256, 56);

      const tex = new THREE.CanvasTexture(canvas);
      const mat = new THREE.SpriteMaterial({ map: tex, depthTest: false });
      const sprite = new THREE.Sprite(mat);
      sprite.scale.set(560, 122, 1);
      return sprite;
    }

    function buildDimensionLines(calc) {
      while (dimensionGroup.children.length > 0) {
        const obj = dimensionGroup.children[0];
        if (obj.geometry) obj.geometry.dispose();
        dimensionGroup.remove(obj);
      }

      if (!state.showDimensions) return;

      const {
        T, levels, outerL, outerW, boxH, l2WallH, l2OuterW,
        innerClearL, innerClearW, innerClearH, totalH
      } = calc;

      const isInternal = state.crateType === 'Internal Crate';
      const lineMat = new THREE.LineBasicMaterial({ color: isInternal ? 0x34d399 : 0x38bdf8 });
      const addDimSegment = (p1, p2, labelText, labelPos, labelColor) => {
        const geo = new THREE.BufferGeometry().setFromPoints([p1, p2]);
        dimensionGroup.add(new THREE.Line(geo, lineMat));
        const sprite = makeTextSprite(labelText, 'rgba(15, 23, 42, 0.90)', labelColor);
        sprite.position.copy(labelPos);
        dimensionGroup.add(sprite);
      };

      const maxSpanW = levels === 2 ? Math.max(outerW, l2OuterW) : outerW;
      const offsetOut = 240;

      // Length Callout (Front Floor)
      const dispL = isInternal ? innerClearL : outerL;
      const lenLabel = isInternal ? `In L: ${dispL} mm` : `L: ${dispL} mm`;
      const zFront = maxSpanW / 2 + offsetOut;
      addDimSegment(
        new THREE.Vector3(-outerL / 2, 10, zFront),
        new THREE.Vector3(outerL / 2, 10, zFront),
        lenLabel,
        new THREE.Vector3(0, 75, zFront + 40),
        isInternal ? '#6ee7b7' : '#38bdf8'
      );

      // Width Callout (Right Floor)
      const dispW = isInternal ? innerClearW : outerW;
      const widLabel = isInternal ? `In W: ${dispW} mm` : `W: ${dispW} mm`;
      const xRight = outerL / 2 + offsetOut;
      addDimSegment(
        new THREE.Vector3(xRight, 10, -outerW / 2),
        new THREE.Vector3(xRight, 10, outerW / 2),
        widLabel,
        new THREE.Vector3(xRight + 60, 75, 0),
        isInternal ? '#6ee7b7' : '#38bdf8'
      );

      // Height Callout (Front-Right Vertical)
      const dispH = isInternal ? innerClearH : boxH;
      const hLabel = isInternal ? `In H: ${dispH} mm` : `H: ${dispH} mm`;
      const hStartY = isInternal ? (calc.supportH + T) : calc.supportH;
      const hEndY = hStartY + dispH;
      addDimSegment(
        new THREE.Vector3(xRight, hStartY, zFront),
        new THREE.Vector3(xRight, hEndY, zFront),
        hLabel,
        new THREE.Vector3(xRight + 130, (hStartY + hEndY) / 2, zFront),
        isInternal ? '#6ee7b7' : '#38bdf8'
      );

      // Total Height Callout (Rear-Left Vertical)
      const xLeft = -outerL / 2 - offsetOut;
      const zRear = -maxSpanW / 2 - offsetOut;
      const totalLabel = levels === 2 ? `Total (2-Lvl): ${totalH} mm` : `Total: ${totalH} mm`;
      addDimSegment(
        new THREE.Vector3(xLeft, 0, zRear),
        new THREE.Vector3(xLeft, totalH, zRear),
        totalLabel,
        new THREE.Vector3(xLeft - 140, totalH / 2, zRear),
        '#fcd34d'
      );

      if (levels === 2) {
        addDimSegment(
          new THREE.Vector3(xRight, calc.supportH + boxH + T, zFront),
          new THREE.Vector3(xRight, calc.supportH + boxH + T + calc.l2InnerClearH, zFront),
          `L2 In H: ${calc.l2InnerClearH} mm`,
          new THREE.Vector3(xRight + 140, calc.supportH + boxH + T + calc.l2InnerClearH / 2, zFront),
          '#a5b4fc'
        );
      }
    }

    function buildForkGuides(calc) {
      while (forkGuideGroup.children.length > 0) {
        const obj = forkGuideGroup.children[0];
        if (obj.geometry) obj.geometry.dispose();
        forkGuideGroup.remove(obj);
      }

      const statusBadge = document.getElementById('fork-status-badge');
      const hintEl = document.getElementById('fork-clearance-hint');

      const bladeW = Math.max(40, state.forkBladeWidth);
      const spread = Math.max(100, state.forkSpread);
      const halfSpread = spread / 2;
      const outerSpan = spread + bladeW;
      const innerGap = Math.max(0, spread - bladeW);
      const entryAlongLengthSide = state.forkEntrySide === 'length';

      const bladeH = Math.max(28, calc.supportH - 22);
      const bladeLen = (entryAlongLengthSide ? calc.outerW : calc.outerL) + 1250;

      const railWidth = calc.railWidth;
      const numRails = calc.numRails;
      const railPositions = [];
      if (numRails === 3) {
        const offsetZ = calc.outerW / 2 - railWidth / 2;
        railPositions.push(-offsetZ, 0, offsetZ);
      } else {
        const span = calc.outerL - railWidth;
        for (let i = 0; i < 4; i++) railPositions.push(-span / 2 + (span / 3) * i);
      }

      const stackOffsetsAlongRail = [
        -calc.railLength / 2 + calc.blockSize / 2,
        0,
        calc.railLength / 2 - calc.blockSize / 2
      ];

      let hasCollision = false;
      const forkAxes = [-halfSpread, halfSpread];

      forkAxes.forEach(fCenter => {
        const fMin = fCenter - bladeW / 2;
        const fMax = fCenter + bladeW / 2;

        if (entryAlongLengthSide) {
          if (numRails === 3) {
            stackOffsetsAlongRail.forEach(sX => {
              if (fMax > sX - calc.blockSize / 2 && fMin < sX + calc.blockSize / 2) hasCollision = true;
            });
          } else {
            railPositions.forEach(rX => {
              if (fMax > rX - railWidth / 2 && fMin < rX + railWidth / 2) hasCollision = true;
            });
          }
        } else {
          if (numRails === 3) {
            railPositions.forEach(rZ => {
              if (fMax > rZ - railWidth / 2 && fMin < rZ + railWidth / 2) hasCollision = true;
            });
          } else {
            stackOffsetsAlongRail.forEach(sZ => {
              if (fMax > sZ - calc.blockSize / 2 && fMin < sZ + calc.blockSize / 2) hasCollision = true;
            });
          }
        }
      });

      if (statusBadge && hintEl) {
        statusBadge.textContent = hasCollision ? 'BLOCKED' : 'CLEAR';
        statusBadge.className = hasCollision
          ? 'font-mono text-[10px] font-bold px-2 py-0.5 rounded bg-red-200 text-red-900'
          : 'font-mono text-[10px] font-bold px-2 py-0.5 rounded bg-emerald-200 text-emerald-900';
        hintEl.textContent = `Outer span: ${Math.round(outerSpan)} mm | Inner gap: ${Math.round(innerGap)} mm | ${hasCollision ? 'Hits skid/rail!' : 'Centered & clear'}`;
      }

      if (!state.showForkGuides) return;

      const forkMat = new THREE.MeshStandardMaterial({
        color: hasCollision ? 0xef4444 : 0x10b981,
        emissive: hasCollision ? 0x991b1b : 0x065f46,
        emissiveIntensity: 0.35,
        transparent: true,
        opacity: 0.72,
        roughness: 0.4
      });

      forkAxes.forEach(fCenter => {
        const geo = entryAlongLengthSide
          ? new THREE.BoxGeometry(bladeW, bladeH, bladeLen)
          : new THREE.BoxGeometry(bladeLen, bladeH, bladeW);
        const mesh = new THREE.Mesh(geo, forkMat);
        mesh.position.set(
          entryAlongLengthSide ? fCenter : 0,
          calc.T + bladeH / 2 + 4,
          entryAlongLengthSide ? 0 : fCenter
        );
        forkGuideGroup.add(mesh);
      });
    }

    function updateHUDAndMetrics(calc) {
      const {
        T, levels, numRails, outerL, outerW, boxH, l2WallH, l2OuterW, l2ShortLen,
        innerClearL, innerClearW, innerClearH, railLength, clearGapBetweenRails,
        supportH, totalH, totalBlocks, totalPanelAreaM2, nesting
      } = calc;

      document.getElementById('header-thickness-badge').textContent = `${T} mm`;

      const typeLabel = state.crateType.toUpperCase();
      const lvlTag = levels === 2 ? ' • 2 LEVELS' : '';
      document.getElementById('hud-title').textContent = `${typeLabel}${lvlTag} • ${numRails} RAILS • ${T} MM`;

      const recBadge = document.getElementById('hud-recipient');
      if (state.recipient.trim()) {
        recBadge.textContent = state.recipient.trim().toUpperCase();
        recBadge.classList.remove('hidden');
      } else {
        recBadge.classList.add('hidden');
      }

      const l2Sub = levels === 2
        ? ` | L2 In: ${innerClearL}×${calc.l2InnerClearW}×${calc.l2InnerClearH} mm`
        : '';
      document.getElementById('hud-subtitle').textContent =
        `Outer: ${outerL} × ${outerW} × ${boxH} mm${l2Sub} | Inner: ${innerClearL} × ${innerClearW} × ${innerClearH} mm | Footings: ${supportH} mm`;
      document.getElementById('hud-rail-clearance').textContent =
        `Flush rail len: ${railLength} mm | Clear gap between rails: ${clearGapBetweenRails} mm`;

      const pairedLabel = document.getElementById('paired-dim-label');
      const pairedValue = document.getElementById('paired-dim-value');
      if (state.crateType === 'Exterior Crate') {
        pairedLabel.textContent = 'Inside Clear Space:';
        pairedValue.textContent = `${innerClearL} × ${innerClearW} × ${innerClearH} mm`;
      } else {
        pairedLabel.textContent = 'Outside Box Size:';
        pairedValue.textContent = `${outerL} × ${outerW} × ${boxH} mm`;
      }

      // Sync Level 2 side hint & locked default value
      const l2Hint = document.getElementById('l2-width-hint');
      const sideL2Input = document.getElementById('input-side-l2');
      if (l2Hint && sideL2Input) {
        const activeSide = state.l2WidthUnlocked ? l2ShortLen : calc.shortA;
        if (!state.l2WidthUnlocked) {
          state.sideL2 = calc.shortA;
          sideL2Input.value = calc.shortA;
        }
        l2Hint.textContent = `Default Side: ${calc.shortA} mm (Bottom: ${activeSide + T}, Top 2: ${activeSide + 2 * T})`;
      }

      // Update Split Longside panel visibility & calculated Longside 2 values
      const splitPanel = document.getElementById('split-longside-panel');
      const l1SplitRow = document.getElementById('l1-split-row');
      const l2SplitRow = document.getElementById('l2-split-row');
      if (calc.isSplitL1 || calc.isSplitL2) {
        splitPanel.classList.remove('hidden');
        l1SplitRow.classList.toggle('hidden', !calc.isSplitL1);
        l2SplitRow.classList.toggle('hidden', !calc.isSplitL2);
        if (calc.isSplitL1) {
          document.getElementById('input-long1-h').value = calc.long1H;
          document.getElementById('display-long2-h').textContent = `${calc.long2H} mm`;
        }
        if (calc.isSplitL2) {
          const activeL2H1 = calc.isSplitL2Front ? calc.l2Front1H : calc.l2Rear1H;
          const activeL2H2 = calc.isSplitL2Front ? calc.l2Front2H : calc.l2Rear2H;
          document.getElementById('input-l2-long1-h').value = activeL2H1;
          document.getElementById('display-l2-long2-h').textContent = `${activeL2H2} mm`;
        }
        document.getElementById('split-longside-hint').textContent =
          `Longside 1 ≥ Longside 2 • Internal Joint L = ${calc.longA} - ${calc.trim} = ${calc.jointL} mm`;
      } else {
        splitPanel.classList.add('hidden');
      }

      document.getElementById('metric-total-height').textContent = `${totalH.toLocaleString()} mm`;
      document.getElementById('metric-blocks').textContent = `${totalBlocks} pcs`;
      document.getElementById('metric-area').textContent = `${totalPanelAreaM2.toFixed(2)} m²`;
      document.getElementById('metric-sheets').textContent =
        `${nesting.sheetCount} sheet${nesting.sheetCount === 1 ? '' : 's'} (${nesting.totalStockAreaM2.toFixed(2)} m²)`;

      document.getElementById('modal-spec-summary').textContent =
        `${state.crateType}${levels === 2 ? ' (2 Levels)' : ''} • Outer: ${outerL} × ${outerW} × ${boxH} mm • Inner: ${innerClearL} × ${innerClearW} × ${innerClearH} mm • ${numRails} Rails • ${T} mm Wood`;
      document.getElementById('modal-panel-total').textContent =
        `Total: ${calc.totalPanelCount} Panels • ${totalPanelAreaM2.toFixed(2)} m²`;
      document.getElementById('modal-rail-clearance-badge').textContent =
        `Flush rail len: ${railLength} mm | Clear gap: ${clearGapBetweenRails} mm`;

      syncAssemblyStepUI();
    }

    // =========================================================================
    // 7. UI CONTROLS, STEP PLAYER, DRAWER & CAMERA PRESETS
    // =========================================================================
    function toggleControlDrawer(forceState) {
      state.drawerOpen = typeof forceState === 'boolean' ? forceState : !state.drawerOpen;
      const drawer = document.getElementById('control-sidebar');
      const backdrop = document.getElementById('drawer-backdrop');
      if (state.drawerOpen) {
        drawer.classList.remove('-translate-x-full');
        backdrop.classList.remove('hidden');
      } else {
        drawer.classList.add('-translate-x-full');
        backdrop.classList.add('hidden');
      }
    }

    function toggleAdvancedControls(forceState) {
      state.showAdvancedControls = typeof forceState === 'boolean' ? forceState : !state.showAdvancedControls;
      const section = document.getElementById('advanced-controls-section');
      const label = document.getElementById('label-advanced-controls');
      const icon = document.getElementById('icon-advanced-chevron');
      if (state.showAdvancedControls) {
        section.classList.remove('hidden');
        label.textContent = 'Hide More Tools & Exports';
        icon.classList.add('rotate-180');
      } else {
        section.classList.add('hidden');
        label.textContent = 'Show More Tools & Exports';
        icon.classList.remove('rotate-180');
      }
    }

    function toggleHudMinimize() {
      state.hudMinimized = !state.hudMinimized;
      const details = document.getElementById('hud-details');
      const icon = document.getElementById('icon-hud-min');
      if (state.hudMinimized) {
        details.classList.add('hidden');
        icon.innerHTML = '<path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />';
      } else {
        details.classList.remove('hidden');
        icon.innerHTML = '<path stroke-linecap="round" stroke-linejoin="round" d="M19.5 12h-15" />';
      }
    }

    function syncDimensionInputsFromState() {
      document.getElementById('input-length').value = state.baseL;
      document.getElementById('input-width').value = state.baseW;
      document.getElementById('input-height').value = state.baseH;
    }

    function validateAndReadInputs() {
      const l = parseFloat(document.getElementById('input-length').value);
      const w = parseFloat(document.getElementById('input-width').value);
      const h = parseFloat(document.getElementById('input-height').value);
      const h2 = parseFloat(document.getElementById('input-height-l2').value);
      const s2 = parseFloat(document.getElementById('input-side-l2').value);
      const t = parseFloat(document.getElementById('input-thickness').value);
      const l1 = parseFloat(document.getElementById('input-long1-h').value);
      const l2l1 = parseFloat(document.getElementById('input-l2-long1-h').value);
      const jh = parseFloat(document.getElementById('input-joint-h').value);
      const jt = parseFloat(document.getElementById('input-joint-trim').value);

      const banner = document.getElementById('validation-banner');
      if ([l, w, h, t].some(v => isNaN(v) || v <= 0) || t < 4 || t > 80 || l < 150 || w < 150 || h < 80) {
        banner.textContent = 'Enter valid dimensions (Length/Width ≥ 150 mm, Height ≥ 80 mm, Thickness 4–80 mm).';
        banner.classList.remove('hidden');
        return;
      }
      if (state.levels === 2 && (isNaN(h2) || h2 < 100)) {
        banner.textContent = 'Level 2 Height must be at least 100 mm.';
        banner.classList.remove('hidden');
        return;
      }
      if (state.levels === 2 && state.l2WidthUnlocked && (isNaN(s2) || s2 < 150)) {
        banner.textContent = 'Level 2 Side (Shortside Base) must be at least 150 mm.';
        banner.classList.remove('hidden');
        return;
      }

      banner.classList.add('hidden');
      state.baseL = Math.round(l);
      state.baseW = Math.round(w);
      state.baseH = Math.round(h);
      state.thickness = Math.round(t);
      if (!isNaN(h2) && h2 >= 100) state.heightL2 = Math.round(h2);
      if (state.l2WidthUnlocked && !isNaN(s2) && s2 >= 150) state.sideL2 = Math.round(s2);
      if (!isNaN(l1) && l1 >= 100) state.longside1Height = Math.round(l1);
      if (!isNaN(l2l1) && l2l1 >= 100) state.l2Longside1Height = Math.round(l2l1);
      if (!isNaN(jh)) state.jointHeight = Math.min(400, Math.max(250, Math.round(jh)));
      if (!isNaN(jt)) state.jointLengthTrim = Math.max(40, Math.round(jt));

      rebuild3DCrate();
    }

    function setCrateType(type) {
      state.crateType = type;
      const isExt = type === 'Exterior Crate';

      document.getElementById('btn-exterior').className = isExt
        ? 'py-2 px-3 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm'
        : 'py-2 px-3 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900';
      document.getElementById('btn-internal').className = !isExt
        ? 'py-2 px-3 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm'
        : 'py-2 px-3 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900';

      document.getElementById('dim-reference-hint').textContent = isExt ? 'Outside Panel Size' : 'Inside Clear Space';
      document.getElementById('crate-type-explanation').innerHTML = isExt
        ? `<strong>Exterior (Outside):</strong> Base L × W × H is the outside box (${state.baseL} × ${state.baseW} × ${state.baseH} mm). Inside clear space is smaller by 2T.`
        : `<strong>Internal (Inside):</strong> Base L × W × H is the inside clear space (${state.baseL} × ${state.baseW} × ${state.baseH} mm). Outer crate expands by +2T.`;

      syncDimensionInputsFromState();
      rebuild3DCrate();
    }

    function setCrateLevels(lvl) {
      const prevMax = getMaxAssemblySteps();
      const wasAtMax = state.assemblyStep >= prevMax;
      state.levels = lvl === 2 ? 2 : 1;

      document.getElementById('btn-levels-1').className = state.levels === 1
        ? 'py-2 px-3 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm'
        : 'py-2 px-3 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900';
      document.getElementById('btn-levels-2').className = state.levels === 2
        ? 'py-2 px-3 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm'
        : 'py-2 px-3 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900';

      document.getElementById('level2-controls-panel').classList.toggle('hidden', state.levels !== 2);
      syncL2WidthLockUI();

      const newMax = getMaxAssemblySteps();
      if (wasAtMax || state.assemblyStep > newMax) {
        state.assemblyStep = newMax;
      }
      rebuild3DCrate();
    }

    function toggleL2WidthUnlock() {
      state.l2WidthUnlocked = !state.l2WidthUnlocked;
      if (state.l2WidthUnlocked && state.lastCalc) {
        state.sideL2 = state.lastCalc.shortA;
        document.getElementById('input-side-l2').value = state.sideL2;
      }
      syncL2WidthLockUI();
      rebuild3DCrate();
    }

    function syncL2WidthLockUI() {
      const input = document.getElementById('input-side-l2');
      const label = document.getElementById('label-l2-width-lock');
      const icon = document.getElementById('icon-l2-width-lock');
      if (!input || !label || !icon) return;

      if (state.l2WidthUnlocked) {
        input.disabled = false;
        input.className = 'w-28 bg-white border border-indigo-400 rounded-lg px-2.5 py-1 font-mono text-xs font-bold text-slate-900 text-right focus:outline-none focus:border-navy-800';
        label.textContent = 'Custom Side';
        icon.innerHTML = '<path stroke-linecap="round" stroke-linejoin="round" d="M13.5 10.5V6.75a4.5 4.5 0 119 0v3.75M3.75 21.75h10.5a2.25 2.25 0 002.25-2.25v-6.75a2.25 2.25 0 00-2.25-2.25H3.75a2.25 2.25 0 00-2.25 2.25v6.75a2.25 2.25 0 002.25 2.25z" />';
      } else {
        input.disabled = true;
        input.className = 'w-28 bg-slate-100 border border-slate-300 rounded-lg px-2.5 py-1 font-mono text-xs font-bold text-slate-500 text-right focus:outline-none disabled:cursor-not-allowed';
        label.textContent = 'Locked: Default';
        icon.innerHTML = '<path stroke-linecap="round" stroke-linejoin="round" d="M16.5 10.5V6.75a4.5 4.5 0 10-9 0v3.75m-.75 11.25h10.5a2.25 2.25 0 002.25-2.25v-6.75a2.25 2.25 0 00-2.25-2.25H6.75a2.25 2.25 0 00-2.25 2.25v6.75a2.25 2.25 0 002.25 2.25z" />';
      }
    }

    function setBaseRails(count) {
      state.rails = count;
      document.getElementById('btn-rails-3').className = count === 3
        ? 'py-2 px-2 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm'
        : 'py-2 px-2 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900';
      document.getElementById('btn-rails-4').className = count === 4
        ? 'py-2 px-2 rounded-lg text-xs font-semibold transition-all bg-navy-800 text-white shadow-sm'
        : 'py-2 px-2 rounded-lg text-xs font-semibold transition-all text-slate-600 hover:text-slate-900';
      rebuild3DCrate();
    }

    function toggleTopLidVisibility() {
      state.hideTopLid = !state.hideTopLid;
      const topNames = ['Top Panel', 'Level 1 Top (Deck)', 'Level 2 Top'];
      topNames.forEach(n => state.hideTopLid ? state.hiddenParts.add(n) : state.hiddenParts.delete(n));

      const btn = document.getElementById('btn-hide-top');
      btn.textContent = state.hideTopLid ? 'Show Top Lid' : 'Hide Top Lid';
      btn.className = state.hideTopLid
        ? 'py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-navy-800 text-white transition flex items-center justify-center gap-1.5'
        : 'py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition flex items-center justify-center gap-1.5';
      applyExplodeAndStepState();
    }

    function toggleFrontWallVisibility() {
      state.hideFrontWall = !state.hideFrontWall;
      const frontNames = [
        'Longside (Front)',
        'Longside 1 Lower (Front)',
        'Longside 2 Upper (Front)',
        'Internal Joint (Front)',
        'Level 2 Longside Front',
        'Level 2 Front 1 (Lower)',
        'Level 2 Front 2 (Upper)',
        'Level 2 Internal Joint (Front)'
      ];
      frontNames.forEach(n => state.hideFrontWall ? state.hiddenParts.add(n) : state.hiddenParts.delete(n));

      const btn = document.getElementById('btn-hide-front');
      btn.textContent = state.hideFrontWall ? 'Show Front Wall' : 'Hide Front Wall';
      btn.className = state.hideFrontWall
        ? 'py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-navy-800 text-white transition flex items-center justify-center gap-1.5'
        : 'py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition flex items-center justify-center gap-1.5';
      applyExplodeAndStepState();
    }

    function toggleJointVisibility() {
      state.showJoints = !state.showJoints;
      const btn = document.getElementById('btn-toggle-joints');
      if (btn) {
        btn.textContent = state.showJoints ? 'Hide Internal Joints' : 'Show Internal Joints';
        btn.className = state.showJoints
          ? 'w-full py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-navy-800 text-white transition flex items-center justify-center gap-1.5'
          : 'w-full py-1.5 px-2.5 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-sky-300 hover:bg-sky-100/70 transition flex items-center justify-center gap-1.5';
      }
      if (!state.showJoints && state.selectedPartName && state.selectedPartName.includes('Internal Joint')) {
        clearPartSelection();
      }
      applyExplodeAndStepState();
    }

    function setRenderMode(mode) {
      state.renderMode = mode;
      const isXray = mode === 'xray';

      document.getElementById('btn-mode-solid').className = !isXray
        ? 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-navy-800 text-white shadow-sm transition'
        : 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition';
      document.getElementById('btn-mode-xray').className = isXray
        ? 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-navy-800 text-white shadow-sm transition'
        : 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition';

      const pill = document.getElementById('btn-pill-xray');
      if (pill) {
        pill.className = isXray
          ? 'px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold bg-sky-500/30 text-sky-200 border border-sky-400/40 transition whitespace-nowrap flex items-center gap-1'
          : 'px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-slate-200 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap flex items-center gap-1';
      }

      if (crateGroup) {
        crateGroup.children.forEach(m => {
          if (m.isMesh && m.userData.baseMaterial) {
            m.userData.baseMaterial.transparent = isXray;
            m.userData.baseMaterial.opacity = isXray ? 0.34 : 1.0;
            m.userData.baseMaterial.needsUpdate = true;
          }
        });
      }
    }

    function toggleDimensionLines() {
      state.showDimensions = !state.showDimensions;
      const btn = document.getElementById('btn-toggle-dims');
      btn.textContent = `Dimensions: ${state.showDimensions ? 'ON' : 'OFF'}`;
      btn.className = state.showDimensions
        ? 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-navy-800 text-white shadow-sm transition'
        : 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition';

      const pill = document.getElementById('btn-pill-dims');
      if (pill) {
        pill.className = state.showDimensions
          ? 'px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold bg-cyan-500/25 text-cyan-200 border border-cyan-400/35 transition whitespace-nowrap flex items-center gap-1'
          : 'px-2.5 py-1.5 rounded-xl text-[11px] sm:text-xs font-semibold text-slate-300 hover:bg-white/15 active:bg-white/20 transition whitespace-nowrap flex items-center gap-1';
      }

      if (state.lastCalc) buildDimensionLines(state.lastCalc);
    }

    function toggleForkOverlay() {
      state.showForkGuides = !state.showForkGuides;
      syncForkControlsUI();
      if (state.lastCalc) buildForkGuides(state.lastCalc);
    }

    function setForkEntrySide(side) {
      state.forkEntrySide = side;
      syncForkControlsUI();
      if (state.lastCalc) buildForkGuides(state.lastCalc);
    }

    function syncForkControlsUI() {
      const btn = document.getElementById('btn-toggle-forks');
      const panel = document.getElementById('fork-controls-panel');
      btn.textContent = `Fork Guides: ${state.showForkGuides ? 'ON' : 'OFF'}`;
      btn.className = state.showForkGuides
        ? 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-emerald-600 text-white shadow-sm transition'
        : 'py-1.5 px-2 rounded-lg text-xs font-semibold bg-white text-slate-700 border border-slate-200 hover:bg-slate-50 transition';
      panel.classList.toggle('hidden', !state.showForkGuides);

      const isLen = state.forkEntrySide === 'length';
      document.getElementById('btn-fork-side-length').className = isLen
        ? 'py-1 px-2 rounded-md text-[11px] font-semibold bg-emerald-600 text-white shadow-sm transition'
        : 'py-1 px-2 rounded-md text-[11px] font-semibold text-slate-600 hover:text-slate-900 transition';
      document.getElementById('btn-fork-side-width').className = !isLen
        ? 'py-1 px-2 rounded-md text-[11px] font-semibold bg-emerald-600 text-white shadow-sm transition'
        : 'py-1 px-2 rounded-md text-[11px] font-semibold text-slate-600 hover:text-slate-900 transition';
    }

    function toggleSawKerf() {
      state.sawKerf = state.sawKerf === 3 ? 0 : 3;
      document.getElementById('btn-kerf-toggle').textContent = `Kerf: ${state.sawKerf} mm`;
      rebuild3DCrate();
    }

    function getMaxAssemblySteps() {
      return state.levels === 2 ? 12 : 7;
    }

    function getStepNames() {
      const rBlocks = state.rails === 3 ? 45 : 60;
      const base = [
        `1. Base Rails (${state.rails} Rails)`,
        `2. Skids (3 per Rail — ${rBlocks} Blocks)`,
        '3. Bottom Panel',
        '4. Longside (Rear)',
        '5. Short Sides (Left & Right)',
        '6. Longside (Front)',
        state.levels === 2 ? '7. Level 1 Top (Deck)' : '7. Top — Complete Crate'
      ];
      if (state.levels === 2) {
        base.push(
          '8. Level 2 Bottom (Middle)',
          '9. Level 2 Longside (Rear)',
          '10. Level 2 Shortsides',
          '11. Level 2 Top',
          '12. Level 2 Longside Front — Complete'
        );
      }
      return base;
    }

    function syncAssemblyStepUI() {
      const maxS = getMaxAssemblySteps();
      const names = getStepNames();
      const cur = Math.min(maxS, Math.max(1, state.assemblyStep));
      state.assemblyStep = cur;

      const s1 = document.getElementById('slider-step');
      const s2 = document.getElementById('preview-slider-step');
      if (s1) { s1.max = maxS; s1.value = cur; }
      if (s2) { s2.max = maxS; s2.value = cur; }

      const badgeText = `Step ${cur} / ${maxS}`;
      const labelText = names[cur - 1] || '';
      document.getElementById('step-badge').textContent = badgeText;
      document.getElementById('step-label').textContent = labelText;
      document.getElementById('preview-step-badge').textContent = badgeText;
      document.getElementById('preview-step-label').textContent = labelText;
    }

    function setAssemblyStep(step) {
      const maxS = getMaxAssemblySteps();
      const previousStep = state.assemblyStep;
      state.assemblyStep = Math.min(maxS, Math.max(1, step));
      syncAssemblyStepUI();
      applyExplodeAndStepState({
        animateNewParts: state.assemblyStep > previousStep,
        previousStep,
        playSound: true
      });
    }

    function changeAssemblyStep(delta) {
      stopAutoPlaySteps();
      setAssemblyStep(state.assemblyStep + delta);
    }

    let stepPlayTimer = null;
    function togglePlaySteps() {
      if (state.isPlayingSteps) {
        stopAutoPlaySteps();
        return;
      }
      state.isPlayingSteps = true;
      const maxS = getMaxAssemblySteps();
      if (state.assemblyStep >= maxS) setAssemblyStep(1);
      updatePlayButtonsUI();

      stepPlayTimer = setInterval(() => {
        if (state.assemblyStep >= getMaxAssemblySteps()) {
          stopAutoPlaySteps();
        } else {
          setAssemblyStep(state.assemblyStep + 1);
        }
      }, 1450);
    }

    function stopAutoPlaySteps() {
      state.isPlayingSteps = false;
      if (stepPlayTimer) {
        clearInterval(stepPlayTimer);
        stepPlayTimer = null;
      }
      updatePlayButtonsUI();
    }

    function updatePlayButtonsUI() {
      const drawerBtn = document.getElementById('btn-play-steps');
      const previewIcon = document.getElementById('icon-preview-play');
      if (drawerBtn) {
        drawerBtn.textContent = state.isPlayingSteps ? 'Stop Auto-Play' : 'Auto-Play Steps';
      }
      if (previewIcon) {
        previewIcon.innerHTML = state.isPlayingSteps
          ? '<path d="M5.25 5.25h13.5v13.5H5.25V5.25z" />'
          : '<path d="M5.25 5.653c0-.856.917-1.398 1.667-.986l11.54 6.348a1.125 1.125 0 010 1.971l-11.54 6.347a1.125 1.125 0 01-1.667-.985V5.653z" />';
      }
    }

    function togglePreviewStepPlayer(forceState) {
      state.showPreviewStepPlayer = typeof forceState === 'boolean' ? forceState : !state.showPreviewStepPlayer;
      document.getElementById('preview-step-hud').classList.toggle('hidden', !state.showPreviewStepPlayer);
      if (!state.showPreviewStepPlayer) stopAutoPlaySteps();
    }

    function setCameraPreset(preset) {
      const calc = state.lastCalc || computeCrateEngineering();
      const centerY = calc.totalH * 0.45;
      const maxDim = Math.max(calc.outerL, calc.outerW, calc.totalH);
      orbit.target.set(0, centerY, 0);
      orbit.radius = maxDim * 1.75;

      if (preset === 'iso') {
        orbit.theta = Math.PI / 4;
        orbit.phi = Math.PI / 3.1;
      } else if (preset === 'front') {
        orbit.theta = 0;
        orbit.phi = Math.PI / 2.08;
      } else if (preset === 'side') {
        orbit.theta = Math.PI / 2;
        orbit.phi = Math.PI / 2.08;
      } else if (preset === 'top') {
        orbit.theta = 0.001;
        orbit.phi = 0.14;
      } else if (preset === 'base') {
        orbit.target.set(0, calc.supportH * 0.6, 0);
        orbit.theta = Math.PI / 5;
        orbit.phi = Math.PI * 0.62;
      }
      updateCameraPosition();
    }

    function resetCameraView() {
      stopAutoPlaySteps();
      state.explodedViewActive = false;
      syncExplodedViewButton();
      animateExplodedView(0, 520);
      setAssemblyStep(getMaxAssemblySteps());
      setCameraPreset('iso');
    }

    function toggleCutListModal(show) {
      const modal = document.getElementById('cutlist-modal');
      modal.classList.toggle('hidden', !show);
      if (show && state.lastCalc) {
        refreshCutListModal(state.lastCalc);
      }
    }

    function onWindowResize() {
      if (!camera || !renderer || !containerEl) return;
      camera.aspect = containerEl.clientWidth / containerEl.clientHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(containerEl.clientWidth, containerEl.clientHeight);
    }

    function animate() {
      requestAnimationFrame(animate);
      updatePartMotionAnimations();
      if (renderer && scene && camera) {
        renderer.render(scene, camera);
      }
    }

    function refreshCutListModal(calc) {
      const pBody = document.getElementById('cutlist-panels-body');
      pBody.innerHTML = calc.panels.map(p => `
        <tr class="hover:bg-slate-50 transition">
          <td class="py-2.5 px-3 font-semibold text-slate-900">${p.name}</td>
          <td class="py-2.5 px-3 text-right font-mono font-semibold text-slate-800">${p.sideA}</td>
          <td class="py-2.5 px-3 text-right font-mono font-semibold text-slate-800">${p.sideB}</td>
          <td class="py-2.5 px-3 text-right font-mono text-slate-600">${calc.T} mm</td>
          <td class="py-2.5 px-3 text-right font-mono font-bold text-navy-800">${p.qty}</td>
          <td class="py-2.5 px-3 text-right font-mono text-slate-700">${p.areaM2.toFixed(2)}</td>
          <td class="py-2.5 px-3 text-[11px] text-slate-500 font-mono">${p.note}</td>
        </tr>
      `).join('');

      const bBody = document.getElementById('cutlist-base-body');
      bBody.innerHTML = calc.baseComponents.map(b => `
        <tr class="hover:bg-slate-50 transition">
          <td class="py-2.5 px-3 font-semibold text-slate-900">${b.name}</td>
          <td class="py-2.5 px-3 text-right font-mono font-semibold text-slate-800">${b.width}</td>
          <td class="py-2.5 px-3 text-right font-mono font-semibold text-slate-800">${b.length}</td>
          <td class="py-2.5 px-3 text-right font-mono text-slate-600">${b.thickness} mm</td>
          <td class="py-2.5 px-3 text-right font-mono font-bold text-navy-800">${b.qty}</td>
          <td class="py-2.5 px-3 text-[11px] text-slate-500">${b.note}</td>
        </tr>
      `).join('');

      // Sync inline Sections 1 & 2 inside Crate Controls drawer
      const drawerSummary = document.getElementById('drawer-schedule-summary');
      if (drawerSummary) {
        drawerSummary.textContent = `${calc.totalPanelCount} Panels • ${calc.totalBlocks} Blocks`;
      }
      const drawerPBody = document.getElementById('drawer-cutlist-panels-body');
      if (drawerPBody) {
        drawerPBody.innerHTML = calc.panels.map(p => `
          <tr class="hover:bg-slate-50">
            <td class="py-1.5 px-2 font-semibold text-slate-900">${p.name}</td>
            <td class="py-1.5 px-1.5 text-right font-mono text-slate-800">${p.sideA}</td>
            <td class="py-1.5 px-1.5 text-right font-mono text-slate-800">${p.sideB}</td>
            <td class="py-1.5 px-2 text-right font-mono font-bold text-navy-800">${p.qty}</td>
          </tr>
        `).join('');
      }
      const drawerBBody = document.getElementById('drawer-cutlist-base-body');
      if (drawerBBody) {
        drawerBBody.innerHTML = calc.baseComponents.map(b => `
          <tr class="hover:bg-slate-50">
            <td class="py-1.5 px-2 font-semibold text-slate-900">${b.name}</td>
            <td class="py-1.5 px-1.5 text-right font-mono text-slate-800">${b.width}</td>
            <td class="py-1.5 px-1.5 text-right font-mono text-slate-800">${b.length}</td>
            <td class="py-1.5 px-2 text-right font-mono font-bold text-navy-800">${b.qty}</td>
          </tr>
        `).join('');
      }

      render2DSheetNestingDiagrams(calc);
      renderOffcutsTable(calc);
      drawISPM15Stencil(calc);
      drawQRCodeSync(calc);
    }

    function render2DSheetNestingDiagrams(calc) {
      const { nesting } = calc;
      document.getElementById('nesting-summary-text').textContent =
        `${nesting.sheetCount} Standard Sheets (1220 × 2240 mm) • Material Yield: ${nesting.yieldPct.toFixed(1)}% • Saw Kerf: ${state.sawKerf} mm`;

      const warn = document.getElementById('nesting-oversize-warning');
      if (nesting.oversizeParts.length > 0) {
        warn.textContent = `Note: Panels exceeding 1220 × 2240 mm stock require splicing/butt-jointing: ${nesting.oversizeParts.join(', ')}`;
        warn.classList.remove('hidden');
      } else {
        warn.classList.add('hidden');
      }

      const colorMap = {
        bottom: '#d97706',
        longside: '#1e3a5f',
        longsideUpper: '#2563eb',
        shortside: '#0284c7',
        top: '#059669',
        l2Bottom: '#4338ca',
        l2Longside: '#4f46e5',
        l2LongsideUpper: '#6366f1',
        l2Shortside: '#7c3aed',
        l2Top: '#0d9488',
        joint: '#b45309'
      };

      const grid = document.getElementById('nesting-sheets-grid');
      grid.innerHTML = nesting.sheets.map(sheet => {
        const rectsSVG = sheet.placements.map(pl => {
          const fill = colorMap[pl.colorKey] || '#1e3a5f';
          const shortName = pl.name.replace(' (Lower)', ' 1').replace(' (Upper)', ' 2').replace(' (Seam)', '');
          const label = `${shortName} (${pl.w}×${pl.h})`;
          const fontSize = Math.max(34, Math.min(54, Math.floor(Math.min(pl.w, pl.h) * 0.14)));
          return `
            <g>
              <rect x="${pl.x}" y="${pl.y}" width="${pl.w}" height="${pl.h}" fill="${fill}" fill-opacity="0.88" stroke="#ffffff" stroke-width="6" />
              <text x="${pl.x + pl.w / 2}" y="${pl.y + pl.h / 2}" fill="#ffffff" font-family="JetBrains Mono, monospace" font-size="${fontSize}" font-weight="bold" text-anchor="middle" dominant-baseline="middle">
                ${label}
              </text>
            </g>
          `;
        }).join('');

        return `
          <div class="bg-white border border-slate-200 rounded-xl p-2.5 shadow-sm">
            <div class="flex items-center justify-between text-[11px] font-mono font-bold text-slate-700 mb-1.5">
              <span>SHEET #${sheet.index}</span>
              <span class="text-slate-400">1220 × 2240 mm</span>
            </div>
            <svg viewBox="0 0 ${SHEET_W} ${SHEET_L}" class="w-full h-auto max-h-56 bg-slate-100 rounded-lg border border-slate-300">
              <rect x="0" y="0" width="${SHEET_W}" height="${SHEET_L}" fill="#f1f5f9" />
              ${rectsSVG}
            </svg>
          </div>
        `;
      }).join('');
    }

    function renderOffcutsTable(calc) {
      const tbody = document.getElementById('offcuts-table-body');
      const badge = document.getElementById('offcuts-total-badge');
      const offcuts = calc.nesting.offcuts;

      badge.textContent = `${offcuts.length} Offcut${offcuts.length === 1 ? '' : 's'}`;
      if (offcuts.length === 0) {
        tbody.innerHTML = `<tr><td colspan="5" class="py-2.5 px-3 text-center text-slate-500">No significant remnant offcuts (&ge; 80 mm) remaining.</td></tr>`;
        return;
      }

      tbody.innerHTML = offcuts.map(oc => `
        <tr class="hover:bg-slate-50">
          <td class="py-2 px-3 font-mono font-semibold text-slate-800">Sheet #${oc.sheetIndex}</td>
          <td class="py-2 px-3 text-right font-mono text-slate-800">${oc.w}</td>
          <td class="py-2 px-3 text-right font-mono text-slate-800">${oc.h}</td>
          <td class="py-2 px-3 text-right font-mono text-slate-700">${oc.areaM2.toFixed(2)}</td>
          <td class="py-2 px-3 text-[11px] font-medium ${oc.usableForBlocks ? 'text-emerald-700' : 'text-slate-500'}">
            ${oc.usableForBlocks ? 'Usable for 100×100 Skid Blocks / Cleats' : 'Narrow strip / small filler'}
          </td>
        </tr>
      `).join('');
    }

    function drawISPM15Stencil(calc) {
      const canvas = document.getElementById('stencil-canvas');
      if (!canvas) return;
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      ctx.fillStyle = '#e7c391';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      ctx.strokeStyle = '#1e293b';
      ctx.lineWidth = 4;
      ctx.strokeRect(14, 14, canvas.width - 28, canvas.height - 28);

      ctx.beginPath();
      ctx.moveTo(165, 14);
      ctx.lineTo(165, canvas.height - 14);
      ctx.stroke();

      ctx.fillStyle = '#0f172a';
      ctx.font = 'bold 28px Inter, sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText('IPPC', 90, 88);
      ctx.font = 'bold 14px JetBrains Mono, monospace';
      ctx.fillText('ISPM-15', 90, 118);
      ctx.fillText('CERTIFIED', 90, 142);

      ctx.textAlign = 'left';
      ctx.font = 'bold 22px JetBrains Mono, monospace';
      ctx.fillText('PH - 04192 - HT', 190, 66);

      const rec = (state.recipient.trim() || 'STANDARD EXPORT').toUpperCase();
      ctx.font = 'bold 18px Inter, sans-serif';
      ctx.fillText(`NAME: ${rec.slice(0, 22)}`, 190, 108);

      ctx.font = '600 14px JetBrains Mono, monospace';
      ctx.fillText(`DIM: ${calc.outerL}×${calc.outerW}×${calc.totalH} MM`, 190, 142);
      ctx.fillText(`TYPE: ${state.crateType.toUpperCase()} (${calc.T}MM)`, 190, 172);
    }

    function buildSyncPayloadString() {
      const params = new URLSearchParams();
      params.set('type', state.crateType === 'Exterior Crate' ? 'ext' : 'int');
      if (state.recipient.trim()) params.set('name', state.recipient.trim());
      params.set('l', state.baseL);
      params.set('w', state.baseW);
      params.set('h', state.baseH);
      params.set('t', state.thickness);
      params.set('r', state.rails);
      if (state.levels === 2) {
        params.set('lv', 2);
        params.set('h2', state.heightL2);
        if (state.l2WidthUnlocked) {
          params.set('l2u', 1);
          params.set('s2', state.sideL2);
        }
      }
      params.set('l1', state.longside1Height);
      if (state.levels === 2) params.set('l2l1', state.l2Longside1Height);
      params.set('jh', state.jointHeight);
      params.set('jt', state.jointLengthTrim);
      if (state.showForkGuides) {
        params.set('fk', `1:${state.forkEntrySide}:${state.forkBladeWidth}:${state.forkSpread}`);
      }
      return params.toString();
    }

    function buildShareURL() {
      const qs = buildSyncPayloadString();
      const base = window.location.href.split('#')[0];
      if (base.startsWith('http://') || base.startsWith('https://')) {
        return `${base}#${qs}`;
      }
      return `CRATE3D:${qs}`;
    }

    function drawQRCodeSync(calc) {
      const canvas = document.getElementById('qr-canvas');
      if (!canvas) return;
      const qrData = buildShareURL();
      if (typeof QRious !== 'undefined') {
        try {
          new QRious({
            element: canvas,
            value: qrData,
            size: 220,
            background: '#ffffff',
            foreground: '#0f172a',
            level: 'L'
          });
          return;
        } catch (err) {}
      }
      OfflineQR.drawToCanvas(canvas, qrData, 220);
    }

    // =========================================================================
    // 8. ONLINE & OFFLINE QR SCANNER (CAMERA, IMAGE/PDF SNAPSHOT & MANUAL)
    // =========================================================================
    let qrVideoStream = null;
    let qrScanRaf = null;
    let qrOffscreenCanvas = null;

    function openQRScannerModal() {
      document.getElementById('qr-scanner-modal').classList.remove('hidden');
      document.getElementById('qr-scan-status').textContent = 'Ready to scan via Camera, Image/PDF Snapshot, or Sync Code';
    }

    function closeQRScannerModal() {
      stopQRCameraScan();
      document.getElementById('qr-scanner-modal').classList.add('hidden');
    }

    function stopQRCameraScan() {
      if (qrScanRaf) {
        cancelAnimationFrame(qrScanRaf);
        qrScanRaf = null;
      }
      if (qrVideoStream) {
        qrVideoStream.getTracks().forEach(t => t.stop());
        qrVideoStream = null;
      }
      const video = document.getElementById('qr-video');
      const placeholder = document.getElementById('qr-camera-placeholder');
      const reticle = document.getElementById('qr-scan-reticle');
      if (video) {
        video.srcObject = null;
        video.classList.add('hidden');
      }
      if (placeholder) placeholder.classList.remove('hidden');
      if (reticle) reticle.classList.add('hidden');
    }

    async function startQRCameraScan() {
      const statusEl = document.getElementById('qr-scan-status');
      const video = document.getElementById('qr-video');
      const placeholder = document.getElementById('qr-camera-placeholder');
      const reticle = document.getElementById('qr-scan-reticle');

      if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        statusEl.textContent = 'Camera unavailable in this browser. Upload a QR photo/screenshot or paste code below.';
        return;
      }

      try {
        statusEl.textContent = 'Starting camera...';
        qrVideoStream = await navigator.mediaDevices.getUserMedia({
          video: { facingMode: { ideal: 'environment' } },
          audio: false
        });
        video.srcObject = qrVideoStream;
        await video.play();
        placeholder.classList.add('hidden');
        video.classList.remove('hidden');
        reticle.classList.remove('hidden');
        statusEl.textContent = 'Scanning live camera feed (online & offline)...';

        const detector = ('BarcodeDetector' in window)
          ? new window.BarcodeDetector({ formats: ['qr_code'] })
          : null;

        if (!qrOffscreenCanvas) qrOffscreenCanvas = document.createElement('canvas');
        const offCtx = qrOffscreenCanvas.getContext('2d', { willReadFrequently: true });

        const tick = async () => {
          if (!qrVideoStream) return;
          if (video.readyState >= 2) {
            if (detector) {
              try {
                const codes = await detector.detect(video);
                if (codes && codes.length > 0 && codes[0].rawValue) {
                  if (applyConfigFromDecodedPayload(codes[0].rawValue)) {
                    closeQRScannerModal();
                    return;
                  }
                }
              } catch (e) {}
            }
            if (typeof window.jsQR === 'function' && video.videoWidth > 0 && video.videoHeight > 0) {
              qrOffscreenCanvas.width = video.videoWidth;
              qrOffscreenCanvas.height = video.videoHeight;
              offCtx.drawImage(video, 0, 0, qrOffscreenCanvas.width, qrOffscreenCanvas.height);
              const imgData = offCtx.getImageData(0, 0, qrOffscreenCanvas.width, qrOffscreenCanvas.height);
              const decoded = window.jsQR(imgData.data, imgData.width, imgData.height, { inversionAttempts: 'attemptBoth' });
              if (decoded && decoded.data) {
                if (applyConfigFromDecodedPayload(decoded.data)) {
                  closeQRScannerModal();
                  return;
                }
              }
            }
          }
          qrScanRaf = requestAnimationFrame(tick);
        };
        qrScanRaf = requestAnimationFrame(tick);
      } catch (err) {
        statusEl.textContent = 'Camera permission denied or unavailable. Use Photo Upload or Paste Code below.';
      }
    }

    async function handleQRImageUpload(event) {
      const file = event.target.files && event.target.files[0];
      if (!file) return;
      const statusEl = document.getElementById('qr-scan-status');
      statusEl.textContent = 'Decoding QR code from uploaded image...';

      const img = new Image();
      const url = URL.createObjectURL(file);
      img.onload = async () => {
        try {
          if ('BarcodeDetector' in window) {
            const detector = new window.BarcodeDetector({ formats: ['qr_code'] });
            const codes = await detector.detect(img);
            if (codes && codes.length > 0 && codes[0].rawValue) {
              URL.revokeObjectURL(url);
              if (applyConfigFromDecodedPayload(codes[0].rawValue)) {
                closeQRScannerModal();
                return;
              }
            }
          }
          if (typeof window.jsQR === 'function') {
            const tempCanvas = document.createElement('canvas');
            tempCanvas.width = img.naturalWidth || img.width;
            tempCanvas.height = img.naturalHeight || img.height;
            const tCtx = tempCanvas.getContext('2d');
            tCtx.drawImage(img, 0, 0);
            const imgData = tCtx.getImageData(0, 0, tempCanvas.width, tempCanvas.height);
            const decoded = window.jsQR(imgData.data, imgData.width, imgData.height, { inversionAttempts: 'attemptBoth' });
            if (decoded && decoded.data) {
              URL.revokeObjectURL(url);
              if (applyConfigFromDecodedPayload(decoded.data)) {
                closeQRScannerModal();
                return;
              }
            }
          }
          statusEl.textContent = 'No valid Crate QR detected in image. Ensure QR is clear and uncropped.';
        } catch (err) {
          statusEl.textContent = 'Could not decode QR from image. Try pasting the Sync Code below.';
        } finally {
          URL.revokeObjectURL(url);
        }
      };
      img.src = url;
    }

    function applyManualQRCode() {
      const raw = document.getElementById('qr-manual-input').value.trim();
      if (!raw) return;
      if (applyConfigFromDecodedPayload(raw)) {
        closeQRScannerModal();
      } else {
        document.getElementById('qr-scan-status').textContent = 'Invalid Crate Sync Code format.';
      }
    }

    function applyConfigFromDecodedPayload(rawText) {
      if (!rawText) return false;
      let queryStr = rawText.trim();
      if (queryStr.includes('#')) {
        queryStr = queryStr.split('#')[1];
      } else if (queryStr.startsWith('CRATE3D:')) {
        queryStr = queryStr.slice('CRATE3D:'.length);
      } else if (queryStr.includes('?')) {
        queryStr = queryStr.split('?')[1];
      }

      const params = new URLSearchParams(queryStr);
      if (!params.has('l') && !params.has('w') && !params.has('type')) {
        return false;
      }

      const type = params.get('type') === 'int' ? 'Internal Crate' : 'Exterior Crate';
      const name = params.get('name') || '';
      const l = parseInt(params.get('l'), 10);
      const w = parseInt(params.get('w'), 10);
      const h = parseInt(params.get('h'), 10);
      const t = parseInt(params.get('t'), 10);
      const r = parseInt(params.get('r'), 10);
      const lv = parseInt(params.get('lv'), 10);
      const h2 = parseInt(params.get('h2'), 10);
      const l2u = params.get('l2u') === '1';
      const s2 = parseInt(params.get('s2'), 10);
      const l1 = parseInt(params.get('l1'), 10);
      const l2l1 = parseInt(params.get('l2l1'), 10);
      const jh = parseInt(params.get('jh'), 10);
      const jt = parseInt(params.get('jt'), 10);
      const fk = params.get('fk');

      if (!isNaN(l)) state.baseL = l;
      if (!isNaN(w)) state.baseW = w;
      if (!isNaN(h)) state.baseH = h;
      if (!isNaN(t)) { state.thickness = t; document.getElementById('input-thickness').value = t; }
      if (!isNaN(h2) && h2 >= 100) { state.heightL2 = h2; document.getElementById('input-height-l2').value = h2; }
      state.l2WidthUnlocked = l2u;
      if (l2u && !isNaN(s2) && s2 >= 200) {
        state.sideL2 = s2;
        document.getElementById('input-side-l2').value = s2;
      }
      if (!isNaN(l1)) { state.longside1Height = l1; document.getElementById('input-long1-h').value = l1; }
      if (!isNaN(l2l1)) { state.l2Longside1Height = l2l1; document.getElementById('input-l2-long1-h').value = l2l1; }
      if (!isNaN(jh)) { state.jointHeight = jh; document.getElementById('input-joint-h').value = jh; }
      if (!isNaN(jt)) { state.jointLengthTrim = jt; document.getElementById('input-joint-trim').value = jt; }

      if (fk) {
        const parts = fk.split(':');
        if (parts[0] === '1') {
          state.showForkGuides = true;
          if (parts[1]) state.forkEntrySide = parts[1];
          if (parts[2]) state.forkBladeWidth = parseInt(parts[2], 10) || 170; document.getElementById('input-fork-width').value = state.forkBladeWidth;
          if (parts[3]) state.forkSpread = parseInt(parts[3], 10) || 550; document.getElementById('input-fork-spread').value = state.forkSpread;
          syncForkControlsUI();
        }
      }

      state.recipient = name;
      document.getElementById('input-recipient').value = name;
      setCrateType(type);
      if (r === 3 || r === 4) setBaseRails(r);
      setCrateLevels(lv === 2 ? 2 : 1);
      syncL2WidthLockUI();
      rebuild3DCrate();
      showToast('Crate configuration loaded from QR code!');
      return true;
    }

    function loadStateFromURLHash() {
      if (!window.location.hash || window.location.hash.length < 4) return;
      applyConfigFromDecodedPayload(window.location.hash.substring(1));
    }

    // =========================================================================
    // 9. EXPORTS: STREAMLINED PDF CUTTING SCHEDULE (WITH QR), DXF, PNG & CSV
    // =========================================================================
    function downloadCuttingSchedulePDF() {
      const calc = state.lastCalc || computeCrateEngineering();
      if (!window.jspdf || !window.jspdf.jsPDF) {
        showToast('PDF library still loading, please try again in a second.');
        return;
      }

      const { jsPDF } = window.jspdf;
      const doc = new jsPDF({ orientation: 'portrait', unit: 'mm', format: 'a4' });

      const rec = state.recipient.trim() ? state.recipient.trim().toUpperCase() : 'UNASSIGNED';
      const dateStr = new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' });
      const fileSlug = (state.recipient.trim() || 'CRATE').replace(/[^a-zA-Z0-9_-]/g, '_');

      doc.setFillColor(30, 58, 95);
      doc.rect(0, 0, 210, 28, 'F');
      doc.setTextColor(255, 255, 255);
      doc.setFont('helvetica', 'bold');
      doc.setFontSize(15);
      doc.text('WORKSHOP CUTTING SCHEDULE & BUILD SHEET', 14, 12);
      doc.setFont('helvetica', 'normal');
      doc.setFontSize(9.5);
      doc.text(`CRATE DESIGNER 3D  |  Generated: ${dateStr}`, 14, 19);

      const pdfQrCanvas = document.createElement('canvas');
      const qrPayload = buildShareURL();
      if (typeof QRious !== 'undefined') {
        try {
          new QRious({
            element: pdfQrCanvas,
            value: qrPayload,
            size: 260,
            background: '#ffffff',
            foreground: '#0f172a',
            level: 'L'
          });
        } catch (e) {
          OfflineQR.drawToCanvas(pdfQrCanvas, qrPayload, 260);
        }
      } else {
        OfflineQR.drawToCanvas(pdfQrCanvas, qrPayload, 260);
      }

      if (pdfQrCanvas.width > 0) {
        doc.setFillColor(255, 255, 255);
        doc.roundedRect(171, 2.5, 26, 26, 2, 2, 'F');
        doc.addImage(pdfQrCanvas.toDataURL('image/png'), 'PNG', 172.5, 4, 23, 23);
      }

      const l2Summary = calc.levels === 2
        ? ` | Level 2 H: ${calc.l2WallH} mm${state.l2WidthUnlocked ? ` (L2 Side: ${calc.l2ShortLen}, W: ${calc.l2OuterW} mm)` : ''}`
        : '';

      doc.autoTable({
        startY: 33,
        theme: 'grid',
        styles: { fontSize: 8.5, cellPadding: 2.2, textColor: [30, 41, 59] },
        headStyles: { fillColor: [241, 245, 249], textColor: [15, 23, 42], fontStyle: 'bold' },
        body: [
          ['Name / Client:', rec, 'Active View:', `${state.crateType}${calc.levels === 2 ? ' (2 Levels)' : ''}`],
          ['Outside Box (L×W×H):', `${calc.outerL} × ${calc.outerW} × ${calc.boxH} mm${l2Summary}`, 'Wood Thickness:', `${calc.T} mm`],
          ['Inside Clear (L×W×H):', `${calc.innerClearL} × ${calc.innerClearW} × ${calc.innerClearH} mm`, 'Total Height (+Footings):', `${calc.totalH} mm (Base: ${calc.supportH} mm)`],
          ['Base Rail System:', `${calc.numRails} Rails (Flush Len: ${calc.railLength} mm | Gap: ${calc.clearGapBetweenRails} mm)`, '1220×2240 Stock Sheets:', `${calc.nesting.sheetCount} Sheets (${calc.nesting.yieldPct.toFixed(1)}% Yield)`]
        ],
        columnStyles: {
          0: { fontStyle: 'bold', cellWidth: 42, fillColor: [248, 250, 252] },
          1: { cellWidth: 53 },
          2: { fontStyle: 'bold', cellWidth: 42, fillColor: [248, 250, 252] },
          3: { cellWidth: 45 }
        }
      });

      let yPos = doc.lastAutoTable.finalY + 7;
      doc.setTextColor(30, 58, 95);
      doc.setFont('helvetica', 'bold');
      doc.setFontSize(10.5);
      doc.text('1. WOODEN PANELS CUTTING LIST', 14, yPos);

      doc.autoTable({
        startY: yPos + 2.5,
        theme: 'striped',
        head: [['Part', 'Side A (mm)', 'Side B (mm)', 'Qty']],
        body: calc.panels.map(p => [
          p.name,
          String(p.sideA),
          String(p.sideB),
          String(p.qty)
        ]),
        styles: { fontSize: 9, cellPadding: 2.6, textColor: [15, 23, 42] },
        headStyles: { fillColor: [30, 58, 95], textColor: [255, 255, 255], fontStyle: 'bold' },
        columnStyles: {
          0: { fontStyle: 'bold' },
          1: { halign: 'right', font: 'courier', fontStyle: 'bold' },
          2: { halign: 'right', font: 'courier', fontStyle: 'bold' },
          3: { halign: 'right', font: 'courier', fontStyle: 'bold' }
        }
      });

      yPos = doc.lastAutoTable.finalY + 7;
      doc.setTextColor(30, 58, 95);
      doc.setFont('helvetica', 'bold');
      doc.setFontSize(10.5);
      doc.text('2. BASE RAILS & 5-BLOCK SKID FOOTINGS', 14, yPos);

      doc.autoTable({
        startY: yPos + 2.5,
        theme: 'striped',
        head: [['Component', 'Width (mm)', 'Length (mm)', 'Thick (mm)', 'Qty', 'Assembly Specification']],
        body: calc.baseComponents.map(b => [
          b.name,
          String(b.width),
          String(b.length),
          String(b.thickness),
          String(b.qty),
          b.note
        ]),
        styles: { fontSize: 8.5, cellPadding: 2.3, textColor: [15, 23, 42] },
        headStyles: { fillColor: [30, 58, 95], textColor: [255, 255, 255], fontStyle: 'bold' },
        columnStyles: {
          0: { fontStyle: 'bold' },
          1: { halign: 'right', font: 'courier' },
          2: { halign: 'right', font: 'courier' },
          3: { halign: 'right', font: 'courier' },
          4: { halign: 'right', font: 'courier', fontStyle: 'bold' }
        }
      });

      doc.save(`Cutting_Schedule_${fileSlug}_${calc.outerL}x${calc.outerW}x${calc.totalH}.pdf`);
      showToast('Exported Cutting Schedule PDF (with scannable QR)!');
    }

    function downloadNestingDXF() {
      const calc = state.lastCalc || computeCrateEngineering();
      let dxf = '0\nSECTION\n2\nENTITIES\n';
      const spacingX = SHEET_W + 250;

      const addRect = (x, y, w, h, layer) => {
        const pts = [[x, y], [x + w, y], [x + w, y + h], [x, y + h], [x, y]];
        for (let i = 0; i < 4; i++) {
          dxf += `0\nLINE\n8\n${layer}\n10\n${pts[i][0]}\n20\n${pts[i][1]}\n30\n0\n11\n${pts[i + 1][0]}\n21\n${pts[i + 1][1]}\n31\n0\n`;
        }
      };

      calc.nesting.sheets.forEach((sheet, idx) => {
        const ox = idx * spacingX;
        addRect(ox, 0, SHEET_W, SHEET_L, 'STOCK_SHEET');
        sheet.placements.forEach(pl => {
          addRect(ox + pl.x, pl.y, pl.w, pl.h, 'CUT_PANELS');
        });
      });

      dxf += '0\nENDSEC\n0\nEOF\n';
      const blob = new Blob([dxf], { type: 'application/dxf' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = `Crate_Nesting_${calc.outerL}x${calc.outerW}x${calc.boxH}.dxf`;
      a.click();
      showToast('Exported 2D Sheet-Nesting DXF file!');
    }

    function downloadStencilPNG() {
      const canvas = document.getElementById('stencil-canvas');
      if (!canvas) return;
      const a = document.createElement('a');
      a.href = canvas.toDataURL('image/png');
      const slug = (state.recipient.trim() || 'CRATE').replace(/[^a-zA-Z0-9_-]/g, '_');
      a.download = `ISPM15_Stencil_${slug}.png`;
      a.click();
      showToast('Downloaded ISPM-15 Stencil PNG!');
    }

    function downloadSnapshot() {
      renderer.render(scene, camera);
      const calc = state.lastCalc || computeCrateEngineering();
      const a = document.createElement('a');
      a.href = renderer.domElement.toDataURL('image/png');
      const slug = (state.recipient.trim() || 'CRATE').replace(/[^a-zA-Z0-9_-]/g, '_');
      a.download = `3D_Crate_${slug}_${calc.outerL}x${calc.outerW}x${calc.boxH}.png`;
      a.click();
      showToast('Saved 3D Crate Snapshot PNG!');
    }

    function copyShareURL() {
      const url = buildShareURL();
      copyTextFallback(url, 'Copied Crate Sync Code / URL to clipboard!');
    }

    function copyCutListCSV() {
      const calc = state.lastCalc || computeCrateEngineering();
      const lines = [
        `Name,${state.recipient || 'N/A'}`,
        `Crate Reference,${state.crateType}${calc.levels === 2 ? ' (2 Levels)' : ''}`,
        `Outer Box (mm),${calc.outerL} x ${calc.outerW} x ${calc.boxH}`,
        `Inner Clear (mm),${calc.innerClearL} x ${calc.innerClearW} x ${calc.innerClearH}`,
        `Total Height with Footings (mm),${calc.totalH}`,
        `Base Rails,${calc.numRails} (Flush Length: ${calc.railLength} mm | Clear Gap: ${calc.clearGapBetweenRails} mm)`,
        '',
        'Part,Side A (mm),Side B (mm),Thickness (mm),Qty,Area (m2)'
      ];
      calc.panels.forEach(p => {
        lines.push(`${p.name},${p.sideA},${p.sideB},${calc.T},${p.qty},${p.areaM2.toFixed(2)}`);
      });
      calc.baseComponents.forEach(b => {
        lines.push(`${b.name},${b.width},${b.length},${b.thickness},${b.qty},-`);
      });
      copyTextFallback(lines.join('\n'), 'Copied Cutting Schedule CSV to clipboard!');
    }

    function copyTextFallback(text, successMsg) {
      const ta = document.createElement('textarea');
      ta.value = text;
      ta.style.position = 'fixed';
      ta.style.opacity = '0';
      document.body.appendChild(ta);
      ta.select();
      try {
        document.execCommand('copy');
        showToast(successMsg);
      } catch (err) {
        showToast('Unable to copy automatically.');
      }
      document.body.removeChild(ta);
    }

    function showToast(msg) {
      const el = document.getElementById('toast-banner');
      el.textContent = msg;
      el.classList.remove('hidden');
      clearTimeout(el._timer);
      el._timer = setTimeout(() => el.classList.add('hidden'), 2800);
    }

    // =========================================================================
    // 10. EVENT LISTENERS & BOOTSTRAP
    // =========================================================================
    document.getElementById('input-recipient').addEventListener('input', e => {
      state.recipient = e.target.value;
      if (state.lastCalc) {
        updateHUDAndMetrics(state.lastCalc);
        drawISPM15Stencil(state.lastCalc);
        drawQRCodeSync(state.lastCalc);
      }
    });

    ['input-length', 'input-width', 'input-height', 'input-height-l2', 'input-side-l2', 'input-thickness', 'input-long1-h', 'input-l2-long1-h', 'input-joint-h', 'input-joint-trim'].forEach(id => {
      document.getElementById(id).addEventListener('input', validateAndReadInputs);
    });

    document.getElementById('input-fork-width').addEventListener('input', e => {
      const v = parseFloat(e.target.value);
      if (!isNaN(v) && v >= 40) {
        state.forkBladeWidth = v;
        if (state.lastCalc) buildForkGuides(state.lastCalc);
      }
    });

    document.getElementById('input-fork-spread').addEventListener('input', e => {
      const v = parseFloat(e.target.value);
      if (!isNaN(v) && v >= 100) {
        state.forkSpread = v;
        if (state.lastCalc) buildForkGuides(state.lastCalc);
      }
    });

    document.getElementById('slider-step').addEventListener('input', e => {
      stopAutoPlaySteps();
      setAssemblyStep(parseInt(e.target.value, 10) || getMaxAssemblySteps());
    });

    document.getElementById('preview-slider-step').addEventListener('input', e => {
      stopAutoPlaySteps();
      setAssemblyStep(parseInt(e.target.value, 10) || getMaxAssemblySteps());
    });

    document.addEventListener('keydown', e => {
      if (e.key === 'Escape') {
        if (state.drawerOpen) toggleControlDrawer(false);
        if (!document.getElementById('cutlist-modal').classList.contains('hidden')) toggleCutListModal(false);
        if (!document.getElementById('qr-scanner-modal').classList.contains('hidden')) closeQRScannerModal();
      }
    });

    window.addEventListener('hashchange', loadStateFromURLHash);

    function bootCrateApp() {
      loadStateFromURLHash();
      syncDimensionInputsFromState();
      rebuild3DCrate();
      setCameraPreset('iso');
      onWindowResize();
      animate();
    }

    syncExplodedViewButton();
    initThreeScene();
    if (document.readyState === 'complete') {
      bootCrateApp();
    } else {
      window.addEventListener('load', bootCrateApp);
    }
  </script>
</body>
</html>
