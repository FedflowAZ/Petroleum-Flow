[fuel-flow-map(1).html](https://github.com/user-attachments/files/28429969/fuel-flow-map.1.html)
# Petroleum-Flow
CA AZ Fuel production distribution consumption
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>California, Arizona & Southern Nevada · Petroleum Distribution</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: #07080d; font-family: Georgia, serif; color: #e8dcc8; min-height: 100vh; }
  #app { display: flex; flex-direction: column; min-height: 100vh; }

  /* Header */
  #header {
    background: #0d1117; border-bottom: 1px solid #1e2535;
    padding: 12px 24px; display: flex; justify-content: space-between;
    align-items: center; flex-wrap: wrap; gap: 12px;
  }
  #header-left .sub { font-size: 10px; letter-spacing: 3px; color: #f59e0b; text-transform: uppercase; margin-bottom: 3px; }
  #header-left h1 { font-size: clamp(14px,2vw,20px); font-weight: normal; color: #f0e6d0; }
  #fy-badge { text-align: center; padding: 3px 8px; background: #f59e0b18; border: 1px solid #f59e0b55; border-radius: 6px; }
  #fy-badge .fy-num { font-size: 13px; color: #f59e0b; font-weight: bold; letter-spacing: 1px; }
  #fy-badge .fy-sub { font-size: 8px; color: #7a8099; }
  #stats { display: flex; gap: 8px; flex-wrap: wrap; }
  .stat-box { text-align: center; padding: 2px 4px; border-radius: 4px; }
  .stat-val { font-size: 9px; font-weight: bold; }
  .stat-lbl { font-size: 7px; color: #7a8099; }
  #state-filter { display: flex; gap: 6px; }
  .filter-btn {
    background: transparent; border: 1px solid #1e2535; color: #4a5568;
    padding: 5px 12px; border-radius: 4px; cursor: pointer;
    font-size: 10px; letter-spacing: 1px; font-family: monospace;
  }
  .filter-btn.active { background: #1e2a3a; border-color: #3a5070; color: #a0c4e8; }

  /* Canvas */
  #canvas-wrap { position: relative; flex: 1; }
  #mainCanvas { width: 100%; display: block; max-width: 1140px; margin: 0 auto; cursor: default; }

  /* Tooltips */
  .tooltip {
    position: fixed; background: #0d1117ee; border-radius: 8px;
    padding: 12px 14px; z-index: 1000; pointer-events: none;
    box-shadow: 0 0 20px #38bdf844;
  }
  #vuln-tooltip { border: 1px solid #f59e0b; max-width: 300px; display: none; }
  #county-tooltip { border: 1px solid #2a3a50; width: 220px; display: none; }
  .tt-title { font-size: 12px; font-weight: bold; margin-bottom: 2px; }
  .tt-score { font-size: 10px; color: #a0b0c8; margin-bottom: 8px; font-family: monospace; }
  .tt-incident-box { margin-bottom: 8px; padding: 6px 8px; background: #1a0a0a; border: 1px solid #7f1d1d; border-radius: 4px; }
  .tt-incident-head { font-size: 9px; color: #fca5a5; font-weight: bold; margin-bottom: 3px; }
  .tt-incident-main { font-size: 9px; color: #f87171; margin-bottom: 3px; }
  .tt-incident-detail { font-size: 8px; color: #6b2222; line-height: 1.4; }
  .tt-reasons { border-top: 1px solid #1e2535; padding-top: 8px; }
  .tt-reason { display: flex; gap: 6px; margin-bottom: 5px; align-items: flex-start; }
  .tt-bullet { font-size: 9px; margin-top: 1px; flex-shrink: 0; }
  .tt-reason-text { font-size: 10px; color: #8090a8; line-height: 1.4; }
  .county-name { font-size: 12px; color: #38bdf8; font-weight: bold; margin-bottom: 2px; }
  .county-vol { font-size: 10px; color: #4a6080; margin-bottom: 10px; font-family: monospace; }
  .pie-row { display: flex; gap: 12px; align-items: center; }
  .pie-legend { display: flex; flex-direction: column; gap: 5px; }
  .pie-item { display: flex; align-items: center; gap: 6px; }
  .pie-swatch { width: 10px; height: 10px; border-radius: 2px; flex-shrink: 0; }
  .pie-label { font-size: 10px; color: #a0b0c8; }
  .pie-pct { font-size: 9px; color: #4a6080; font-family: monospace; }
  .tt-note { margin-top: 8px; font-size: 8px; color: #2a3048; border-top: 1px solid #1e2535; padding-top: 6px; }

  /* Controls */
  #controls {
    background: #0d1117; border-top: 1px solid #1e2535;
    padding: 12px 24px; display: flex; gap: 16px; align-items: center; flex-wrap: wrap;
  }
  .ctrl-btn {
    border: 1px solid #1e2535; border-radius: 4px; cursor: pointer;
    font-size: 10px; letter-spacing: 1px; font-family: monospace; padding: 6px 14px;
  }
  #vuln-btn { background: transparent; color: #4a5568; }
  #vuln-btn.on { background: #450a0a; border-color: #dc2626; color: #fca5a5; }
  #play-btn { background: #1a2535; border-color: #2a3a50; color: #8aa0c0; letter-spacing: 2px; }
  #play-btn.playing { background: #7c2d12; border-color: #dc6030; color: #fca070; }
  #fs-btn { background: transparent; color: #4a5568; margin-left: auto; }
  #slider-wrap { flex: 1; min-width: 200px; }
  #yearSlider { width: 100%; accent-color: #f59e0b; cursor: pointer; }
  .slider-labels { display: flex; justify-content: space-between; font-size: 10px; color: #3a4558; margin-top: 3px; }
  #data-note { font-size: 10px; color: #2a3048; max-width: 320px; line-height: 1.6; }

  /* Legend */
  #legend {
    padding: 10px 24px 14px; display: flex; gap: 14px; flex-wrap: wrap;
    border-top: 1px solid #10131c;
  }
  .legend-item { display: flex; align-items: center; gap: 7px; font-size: 10px; color: #5a6580; }
  .legend-swatch { width: 20px; height: 3px; border-radius: 2px; flex-shrink: 0; }

  /* Fullscreen */
  :fullscreen body, :fullscreen #app { background: #07080d; }
  :-webkit-full-screen #mainCanvas { max-width: 100%; }
</style>
</head>
<body>
<div id="app">

<!-- Header -->
<div id="header">
  <div id="header-left">
    <div class="sub">SFPP West Line · Shared Pipeline Infrastructure</div>
    <h1>California, Arizona &amp; Southern Nevada · All Petroleum Products Distribution</h1>
  </div>
  <div id="fy-badge">
    <div class="fy-num" id="fy-display">FY2024</div>
    <div class="fy-sub">Fiscal Year</div>
  </div>
  <div id="stats">
    <div class="stat-box" style="background:#e879f911;border:1px solid #e879f933">
      <div class="stat-val" style="color:#e879f9" id="stat-ca">—</div>
      <div class="stat-lbl">California</div>
    </div>
    <div class="stat-box" style="background:#f59e0b11;border:1px solid #f59e0b33">
      <div class="stat-val" style="color:#f59e0b" id="stat-wl">—</div>
      <div class="stat-lbl">West Line</div>
    </div>
    <div class="stat-box" style="background:#38bdf811;border:1px solid #38bdf833">
      <div class="stat-val" style="color:#38bdf8" id="stat-az">—</div>
      <div class="stat-lbl">Arizona</div>
    </div>
  </div>
  <div id="state-filter">
    <button class="filter-btn active" data-state="both">All</button>
    <button class="filter-btn" data-state="ca">CA</button>
    <button class="filter-btn" data-state="nv">NV</button>
    <button class="filter-btn" data-state="az">AZ</button>
  </div>
</div>

<!-- Canvas -->
<div id="canvas-wrap">
  <canvas id="mainCanvas"></canvas>
  <!-- Vulnerability tooltip -->
  <div class="tooltip" id="vuln-tooltip">
    <div class="tt-title" id="vt-title"></div>
    <div class="tt-score" id="vt-score"></div>
    <div class="tt-incident-box" id="vt-incident-box">
      <div class="tt-incident-head">⚠ Pipeline Incidents</div>
      <div class="tt-incident-main" id="vt-incident-main"></div>
      <div class="tt-incident-detail" id="vt-incident-detail"></div>
    </div>
    <div class="tt-reasons" id="vt-reasons"></div>
  </div>
  <!-- County tooltip -->
  <div class="tooltip" id="county-tooltip">
    <div class="county-name" id="ct-name"></div>
    <div class="county-vol" id="ct-vol"></div>
    <div class="pie-row">
      <svg id="ct-pie" width="88" height="88" style="flex-shrink:0"></svg>
      <div class="pie-legend" id="ct-legend"></div>
    </div>
    <div class="tt-note">Fuel mix estimated · gasoline from ADOT data</div>
  </div>
</div>

<!-- Controls -->
<div id="controls">
  <button class="ctrl-btn" id="fs-btn">⛶ FULLSCREEN</button>
  <button class="ctrl-btn" id="vuln-btn">⚡ VULNERABILITY OFF</button>
  <button class="ctrl-btn playing" id="play-btn">■ PAUSE</button>
  <div id="slider-wrap">
    <input type="range" id="yearSlider" min="0" max="25" value="24">
    <div class="slider-labels"><span>FY2000</span><span>FY2025</span></div>
  </div>
  <div id="data-note">CA volumes: CDTFA/CEC statewide data · all petroleum products. AZ volumes: ADOT gasoline gallonage scaled to all-fuels basis (÷0.45). West Line shared by CA, NV, AZ.</div>
</div>

<!-- Legend -->
<div id="legend">
  <div class="legend-item"><div class="legend-swatch" style="background:#ff3c00"></div>Pulsing ring = vulnerability (5=most exposed/red/fast)</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#1e3a6e"></div>Alaska tankers (domestic · Jones Act · dashed)</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#0ea5e9"></div>Bay Area foreign tankers</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#06b6d4"></div>LA foreign tankers</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#84cc16"></div>Local supply (Kern Co. / SJV)</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#e879f9"></div>LA Basin refineries · Colton ◆ junction</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#fbbf24"></div>CALNEV pipeline + S. Nevada terminal</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#f59e0b"></div>SFPP West Line</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#fb923c"></div>SFPP East Line (El Paso → AZ)</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#38bdf8"></div>AZ Phoenix hub + N/C counties</div>
  <div class="legend-item"><div class="legend-swatch" style="background:#818cf8"></div>AZ Tucson hub + S counties</div>
</div>

</div><!-- #app -->

<script>
// ═══════════════════════════════════════════════════════════════════
// DATA
// ═══════════════════════════════════════════════════════════════════
const AZ_COUNTY_DATA = {
  Apache:      [28053443,28261659,31809628,32021881,31548566,30382536,30353709,27121873,26466346,27478322,26669503,27358248,26674748,25578458,25720189,25667194,26912308,27774900,28321268,28461223,28045463,26780544,28891611,27749010,27536872,26826390],
  Cochise:     [63583911,49701730,54231661,54904384,56306187,58200662,59842797,61280182,57112351,51772212,56692843,53927825,48980997,46450612,47495491,47390982,49612645,51937969,57991690,55635713,55129691,55237677,54362075,54316988,57565384,58041377],
  Coconino:    [123797415,131335993,132991488,125897762,109262146,108012184,103211966,103165038,101104461,92439564,93152706,92921749,90111732,87158046,87678651,92787643,99184874,103784624,105273238,101170962,96897675,96574843,98741536,97044185,102909641,103814820],
  Gila:        [31515458,28050301,27303835,26299476,30556440,34524661,32879444,33687614,31530805,27583201,29284937,29414175,30062275,28726261,28244536,27848879,28410573,29179902,29911017,29643490,30742434,31906207,33150785,33064307,34101691,32197983],
  Graham:      [12668027,10807747,11206254,11909275,10606049,11602453,10528338,12699871,13310878,9820352,12713794,12850487,11428353,11603291,11985569,10372625,10962924,10811879,10924727,10972970,12832210,12620983,13176607,12958320,13436732,13002477],
  Greenlee:    [3998347,3714050,2452414,2372819,2721621,2920583,3577801,3369995,3274659,3325186,2643329,3184067,3609877,4282592,5420171,4761366,4015853,4075513,4380765,4572625,4440115,3958037,2697148,3994629,4132463,4530611],
  "La Paz":    [29178196,29479148,29782126,29728920,31616935,34599286,32123970,35761707,34057950,31684368,32501450,33815395,33763448,33318951,33556188,32269290,36073166,36568059,38366668,40363240,43781041,44480152,49530045,49068514,49424124,50331019],
  Maricopa:    [1338111695,1392390421,1435375342,1490129655,1526283642,1536136249,1596332274,1696777678,1680283509,1574810563,1505131152,1534396052,1552575301,1533111631,1559390102,1614664310,1683833887,1738131988,1763896242,1777338506,1688068380,1635702287,1776212190,1759586248,1775508076,1786493852],
  Mohave:      [103067687,103201634,104290151,108946159,113336400,132264555,121243224,118731966,111106533,104955761,108441384,106029035,104183872,102865129,105674430,106218363,110597068,113024140,114686688,116167207,113410306,115853141,120136939,118633020,118119491,118934900],
  Navajo:      [65075537,63948997,57367157,60973836,66879250,69364603,62232604,63159282,61322038,61263003,58439980,59984493,55797122,55570520,57104722,58151246,62233732,63437414,63355116,65112607,63259267,66541280,68931869,65139483,65714783,69049581],
  Pima:        [410588887,399417225,398316125,404501629,395330549,389341578,371309146,384600547,394428155,356454764,386963156,383686309,382830716,379188156,373075590,378419340,393336245,397732830,401061967,407865784,388944656,365699680,397672674,394435535,391446612,389860283],
  Pinal:       [73038420,70229117,71176660,74661275,85753405,99858478,101597300,109236724,119685805,112917712,114026315,111614712,109706834,108300820,109289018,116282213,122659492,126954134,128848881,131817159,131666646,136052332,142502446,152402373,156472890,163993740],
  "Santa Cruz":[23437330,24811733,26358752,27280949,32206452,31434627,28409612,27914475,24926338,21098973,22489733,20783567,17834076,18006583,21326106,24104593,25035366,27330049,27368517,26148528,25450942,19313708,21865750,21835928,25406226,29148099],
  Yavapai:     [80239390,79184742,83362324,86021861,90689132,91779506,94289890,95388241,88692923,83984020,90349800,90325605,88586724,87249669,89210546,90953422,95268886,98481815,99305883,100814492,97667856,106195444,113354629,112792609,112418024,112157799],
  Yuma:        [91354800,85538800,91659624,95630542,95077644,97593661,98262938,97739564,102114242,91932388,93426284,88803381,88593223,86821597,89326096,92544180,98186531,99297714,103087702,103615568,101097565,100925022,108921719,107944581,110915446,114493490],
};
const CA_STATEWIDE = [14800000000,15100000000,15300000000,15400000000,15500000000,15600000000,15700000000,15800000000,15400000000,14600000000,14200000000,14100000000,14100000000,14400000000,14600000000,15100000000,15300000000,15200000000,15100000000,15000000000,13200000000,13000000000,13600000000,13500000000,13400000000,13300000000];
const CA_REGIONS = {
  "Southern CA": { share:0.55, color:"#e879f9" },
  "Bay Area":    { share:0.18, color:"#34d399" },
  "Central Valley":{ share:0.17, color:"#fb7185" },
  "Northern CA": { share:0.10, color:"#34d399" },
};
const FISCAL_YEARS = Array.from({length:26},(_,i)=>2000+i);
const AZ_PHX_COUNTIES = ["Maricopa","Pinal","Yavapai","Mohave","La Paz","Navajo","Apache","Gila","Graham","Greenlee","Coconino"];
const AZ_TUC_COUNTIES = ["Pima","Santa Cruz","Cochise","Yuma"];
const AZ_FUELS_SCALE = 1/0.45;

const FOREIGN_MIX = [
  ["Saudi Arabia 30%","Ecuador 18%","Iraq 12%","Colombia 10%"],
  ["Saudi Arabia 31%","Ecuador 17%","Iraq 11%","Colombia 11%"],
  ["Saudi Arabia 32%","Ecuador 16%","Iraq 10%","Colombia 11%"],
  ["Saudi Arabia 32%","Ecuador 16%","Iraq 9%","Colombia 12%"],
  ["Saudi Arabia 31%","Ecuador 17%","Iraq 9%","Colombia 12%"],
  ["Saudi Arabia 30%","Ecuador 18%","Iraq 9%","Colombia 12%"],
  ["Saudi Arabia 29%","Ecuador 19%","Iraq 9%","Colombia 13%"],
  ["Saudi Arabia 28%","Ecuador 19%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 27%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 27%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 26%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 26%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 27%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 28%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 29%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 30%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 30%","Ecuador 20%","Iraq 8%","Colombia 14%"],
  ["Saudi Arabia 29%","Ecuador 20%","Colombia 14%","Iraq 8%"],
  ["Saudi Arabia 37%","Ecuador 14%","Colombia 12%","Iraq 8%"],
  ["Saudi Arabia 33%","Ecuador 16%","Colombia 11%","Iraq 10%"],
  ["Ecuador 24%","Saudi Arabia 23%","Iraq 20%","Colombia 8%"],
  ["Ecuador 18%","Saudi Arabia 16%","Iraq 16%","Russia 6%"],
  ["Iraq 22%","Ecuador 17%","Saudi Arabia 16%","Brazil 14%"],
  ["Iraq 21%","Brazil 20%","Guyana 16%","Ecuador 14%"],
  ["Iraq 22%","Brazil 18%","Guyana 14%","Canada 12%"],
  ["Brazil 18%","Iraq 18%","Guyana 14%","Canada 12%"],
];
const SUPPLY_NOTES = {
  18:"⚡ Saudi Arabia peaks at 37%",
  20:"⚡ Saudi Arabia drops sharply",
  21:"⚡ Russia enters at 6%",
  22:"⚡ Russia gone post-sanctions",
  23:"⚡ Guyana surges — Stabroek field",
  24:"⚡ Brazil overtakes Iraq as #1",
  25:"⚡ Brazil #1; Saudi Arabia at 8%",
};
const VULN = {
  nvLasVegas:5.0, azPhx:4.5, azTuc:4.0, azPhxGrp:4.5, azTucGrp:4.0,
  caNorCal:3.5, caCentVal:3.0, caBayArea:2.0, caSoCal:2.5,
};
const VULN_DETAIL = {
  nvLasVegas:{label:"S. Nevada (Las Vegas)",score:"5.0 — Most Vulnerable",incidents:"Jan 2025 shutdown triggered emergency fuel-ups · 88% CA-dependent",incidentDetail:"Jan 2025 wildfire outage: Las Vegas Metro Police preemptively filled all vehicles. CA supplies 88% of NV gasoline market.",reasons:["Single pipeline spur (CALNEV) — no alternative","No local refining","~5 days terminal storage","88% dependent on CA refineries","Tourist demand amplifies shortages"]},
  azPhx:{label:"Phoenix Terminal",score:"4.5 — Very Vulnerable",incidents:"3+ West Line shutdowns since 2022 · +50¢/gal spike Feb 2025",incidentDetail:"Jan 2025: 5-day shutdown absorbed by inventory. Feb 2025: CA refinery fire + Phillips 66 closure → +50¢/gal in 4 weeks. CA supplies ~33% of AZ gasoline market.",reasons:["~60% supply via single West Line","No local refining in Arizona","400 miles from nearest refinery","~12 days terminal storage","CA refinery closures: longer-term structural risk"]},
  azPhxGrp:{label:"N/C Arizona Counties",score:"4.5 — Very Vulnerable",incidents:"Inherits all Phoenix terminal pipeline incidents",incidentDetail:"Any West Line shutdown reduces supply within days. 2003 Tucson incident: $14.4M damage on 1955-era pipe.",reasons:["Entirely dependent on Phoenix terminal","Inherits all Phoenix supply risk","Truck delivery adds final-mile vulnerability","No alternative supply routes"]},
  azTuc:{label:"Tucson Terminal",score:"4.0 — Vulnerable",incidents:"1 PHMSA incident at Tucson (2003)",incidentDetail:"July 2003: 394 barrels lost, $14.4M property damage, environmental cracking on 1955 pipe. East Line provides partial backup.",reasons:["Served by both West and East Lines","East Line from El Paso provides diversification","No local refining","Better positioned than Phoenix for El Paso supply"]},
  azTucGrp:{label:"S. Arizona Counties",score:"4.0 — Vulnerable",incidents:"Inherits Tucson terminal pipeline risk",incidentDetail:"1990 incident near Marana (I-10): 2,444 barrels lost from excavation damage.",reasons:["Dependent on Tucson terminal","Slightly better diversified than Phoenix counties","Yuma has some independent supply options"]},
  caNorCal:{label:"Northern California",score:"3.5 — Moderate-High",incidents:"Valero Benicia closure Apr 2026 directly impacts this region",incidentDetail:"Valero Benicia (145,000 b/d) closing Apr 2026. PBF Martinez fire Feb 2024 still recovering. Phillips 66 Rodeo converted to renewable diesel.",reasons:["Furthest from LA Basin refineries","Valero Benicia closure Apr 2026 — major hit","PBF Martinez still recovering from 2024 fire","Limited terminal storage","Increasing reliance on Asian imports"]},
  caCentVal:{label:"Central Valley",score:"3.0 — Moderate",incidents:"Indirectly affected by West Line incidents",incidentDetail:"West Line outages reduce Central Valley supply within 3-5 days.",reasons:["Pipeline dependent but close to both hubs","Two supply corridors available","No significant local refining","Agricultural demand creates seasonal spikes"]},
  caBayArea:{label:"Bay Area",score:"2.0 — Low-Moderate",incidents:"Valero Benicia closure Apr 2026 · PBF Martinez fire 2024",incidentDetail:"Valero Benicia (145,000 b/d) closing Apr 2026. PBF Martinez fire Feb 2024. Phillips 66 Rodeo converted to renewable diesel.",reasons:["Local refineries (Richmond, Martinez, Rodeo)","Marine import terminals at Richmond","Alaska crude via Jones Act tankers","Valero Benicia closure reduces capacity","Increasing reliance on Asian imports"]},
  caSoCal:{label:"Southern California",score:"2.5 — Low-Moderate (rising)",incidents:"Crisis: 20% CA refining capacity lost since 2020",incidentDetail:"Phillips 66 LA (139,000 b/d) closing end 2025. Chevron El Segundo fire Oct 2025. Valero Benicia closing Apr 2026. EIA warns 'outsized' impact. West Coast prices projected $4.19/gal in 2026.",reasons:["Still has Chevron El Segundo (285,000 b/d)","Multiple marine import terminals","Largest terminal storage in the West","20% CA capacity lost since 2020 — accelerating","CARB specs limit import flexibility"]},
};
const COUNTY_FUEL_MIX = {
  Maricopa:{gasoline:0.47,diesel:0.20,jetfuel:0.22,other:0.11},
  Pima:{gasoline:0.45,diesel:0.22,jetfuel:0.18,other:0.15},
  Pinal:{gasoline:0.44,diesel:0.28,jetfuel:0.12,other:0.16},
  Coconino:{gasoline:0.43,diesel:0.30,jetfuel:0.10,other:0.17},
  Mohave:{gasoline:0.43,diesel:0.30,jetfuel:0.10,other:0.17},
  Yavapai:{gasoline:0.46,diesel:0.26,jetfuel:0.12,other:0.16},
  Navajo:{gasoline:0.43,diesel:0.30,jetfuel:0.10,other:0.17},
  Yuma:{gasoline:0.40,diesel:0.35,jetfuel:0.10,other:0.15},
  "La Paz":{gasoline:0.42,diesel:0.32,jetfuel:0.08,other:0.18},
  Cochise:{gasoline:0.42,diesel:0.30,jetfuel:0.12,other:0.16},
  Gila:{gasoline:0.44,diesel:0.30,jetfuel:0.08,other:0.18},
  Graham:{gasoline:0.40,diesel:0.36,jetfuel:0.07,other:0.17},
  Greenlee:{gasoline:0.38,diesel:0.40,jetfuel:0.05,other:0.17},
  "Santa Cruz":{gasoline:0.44,diesel:0.28,jetfuel:0.10,other:0.18},
  Apache:{gasoline:0.42,diesel:0.32,jetfuel:0.08,other:0.18},
};
const FUEL_COLORS = {gasoline:"#f59e0b",diesel:"#38bdf8",jetfuel:"#a78bfa",other:"#6b7280"};
const FUEL_LABELS = {gasoline:"Gasoline",diesel:"Diesel",jetfuel:"Jet Fuel",other:"Other"};

// ═══════════════════════════════════════════════════════════════════
// STATE
// ═══════════════════════════════════════════════════════════════════
let yearIndex = 24;
let playing = true;
let activeState = "both";
let showVuln = false;
let particles = [];
let frame = 0;
let animId = null;
let playInterval = null;
let yearFloat = 24;

// ═══════════════════════════════════════════════════════════════════
// LAYOUT
// ═══════════════════════════════════════════════════════════════════
const W = 1140, H = 620, H2 = H/2;
const COL = {tanker:60,caRef:210,caReg:370,westLine:540,azTerm:700,azDisp:840,azCo:980,eastLine:1080};

// ═══════════════════════════════════════════════════════════════════
// HELPERS
// ═══════════════════════════════════════════════════════════════════
const fmt = n => n>=1e9?(n/1e9).toFixed(1)+"B":n>=1e6?(n/1e6).toFixed(0)+"M":(n/1e3).toFixed(0)+"K";
const fmtFull = n => n>=1e9?(n/1e9).toFixed(2)+"B gal":n>=1e6?(n/1e6).toFixed(1)+"M gal":(n/1e3).toFixed(0)+"K gal";

function cubicBezier(p0,cp1,cp2,p1,t){
  const mt=1-t;
  return [mt*mt*mt*p0[0]+3*mt*mt*t*cp1[0]+3*mt*t*t*cp2[0]+t*t*t*p1[0],
          mt*mt*mt*p0[1]+3*mt*mt*t*cp1[1]+3*mt*t*t*cp2[1]+t*t*t*p1[1]];
}

function getVolumes(yi){
  const azPhxGas=AZ_PHX_COUNTIES.reduce((s,c)=>s+(AZ_COUNTY_DATA[c]?.[yi]??0),0);
  const azTucGas=AZ_TUC_COUNTIES.reduce((s,c)=>s+(AZ_COUNTY_DATA[c]?.[yi]??0),0);
  const azPhx=azPhxGas*AZ_FUELS_SCALE, azTuc=azTucGas*AZ_FUELS_SCALE;
  const azTotal=azPhx+azTuc;
  const caTotal=CA_STATEWIDE[yi]??CA_STATEWIDE[24];
  return {azPhx,azTuc,azTotal,caTotal,westLineTotal:azTotal*0.60+caTotal*0.12,azWest:azTotal*0.60,azEast:azTotal*0.40};
}

function getNodes(yi){
  const v=getVolumes(yi);
  const mix=FOREIGN_MIX[yi]??FOREIGN_MIX[24];
  return {
    laTanker:{x:COL.tanker,y:H2+80,vol:v.caTotal*0.55,label:"LA Tankers",sub:"Crude Oil",color:"#06b6d4",countries:mix},
    bayTanker:{x:COL.tanker,y:H2-140,vol:v.caTotal*0.20,label:"Bay Tankers",sub:"Crude Oil",color:"#0ea5e9",countries:[...mix]},
    akTanker:{x:COL.tanker,y:H2-260,vol:v.caTotal*0.07,label:"AK Tankers",sub:"Crude Oil · Jones Act",color:"#4a7fd4",countries:["Alaska North Slope","Jones Act"]},
    localSupply:{x:COL.caRef,y:H2+240,vol:v.caTotal*0.23,label:"Local Supply",sub:"Kern Co. / SJV",color:"#84cc16"},
    caRefinery:{x:COL.caRef,y:H2+80,vol:v.caTotal*0.88,label:"LA Basin",sub:"Refineries",color:"#e879f9"},
    caAlaska:{x:COL.caRef,y:H2-220,vol:v.caTotal*0.12,label:"Bay Area",sub:"Refineries",color:"#34d399"},
    caSoCal:{x:COL.caReg,y:H2+200,vol:v.caTotal*0.55,label:"Southern CA",color:"#e879f9"},
    caBayArea:{x:COL.caReg,y:H2-180,vol:v.caTotal*0.18,label:"Bay Area",color:"#34d399"},
    caCentVal:{x:COL.caReg,y:H2-20,vol:v.caTotal*0.17,label:"Cent. Valley",color:"#fb7185"},
    caNorCal:{x:COL.caReg,y:H2-270,vol:v.caTotal*0.10,label:"Northern CA",color:"#34d399"},
    colton:{x:COL.westLine-80,y:H2+30,vol:v.caTotal*0.35,label:"Colton",sub:"Junction",color:"#e879f9"},
    calnev:{x:COL.westLine,y:H2-110,vol:v.caTotal*0.08,label:"CALNEV",sub:"Pipeline",color:"#fbbf24"},
    nvLasVegas:{x:COL.westLine+60,y:H2-160,vol:v.caTotal*0.08,label:"S. Nevada",sub:"Terminal",color:"#fbbf24"},
    westLine:{x:COL.westLine,y:H2+30,vol:v.westLineTotal,label:"SFPP",sub:"West Line",color:"#f59e0b"},
    azPhx:{x:COL.azTerm,y:H2-100,vol:v.azPhx,label:"Phoenix",sub:"Terminal",color:"#38bdf8"},
    azTuc:{x:COL.azTerm,y:H2+100,vol:v.azTuc,label:"Tucson",sub:"Terminal",color:"#818cf8"},
    azPhxGrp:{x:COL.azDisp,y:H2-100,vol:v.azPhx,label:"N/C AZ",color:"#38bdf8"},
    azTucGrp:{x:COL.azDisp,y:H2+100,vol:v.azTuc,label:"S AZ",color:"#818cf8"},
    eastLine:{x:COL.eastLine,y:H2,vol:v.azEast,label:"East Line",sub:"← El Paso",color:"#fb923c"},
  };
}

function getRibbons(yi){
  const n=getNodes(yi),v=getVolumes(yi),maxV=v.caTotal;
  const sc=val=>Math.max(2,(val/maxV)*90);
  return [
    {id:"laTank-ref",x0:n.laTanker.x,y0:n.laTanker.y,x1:n.caRefinery.x,y1:n.caRefinery.y,w:sc(v.caTotal*0.55),vol:v.caTotal*0.55,color:"#06b6d4",state:"ca"},
    {id:"bayTank-ref",x0:n.bayTanker.x,y0:n.bayTanker.y,x1:n.caAlaska.x,y1:n.caAlaska.y,w:sc(v.caTotal*0.20),vol:v.caTotal*0.20,color:"#0ea5e9",state:"ca"},
    {id:"akTank-ref",x0:n.akTanker.x,y0:n.akTanker.y,x1:n.caAlaska.x,y1:n.caAlaska.y,w:sc(v.caTotal*0.07),vol:v.caTotal*0.07,color:"#4a7fd4",state:"ca",dashed:true},
    {id:"local-ref",x0:n.localSupply.x,y0:n.localSupply.y,x1:n.caRefinery.x,y1:n.caRefinery.y,w:sc(v.caTotal*0.23),vol:v.caTotal*0.23,color:"#84cc16",state:"ca"},
    {id:"caRef-colton",x0:n.caRefinery.x,y0:n.caRefinery.y,x1:n.colton.x,y1:n.colton.y,w:sc(v.caTotal*0.35),vol:v.caTotal*0.35,color:"#e879f9",state:"ca"},
    {id:"colton-calnev",x0:n.colton.x,y0:n.colton.y,x1:n.calnev.x,y1:n.calnev.y,w:sc(v.caTotal*0.08),vol:v.caTotal*0.08,color:"#fbbf24",state:"nv"},
    {id:"calnev-nv",x0:n.calnev.x,y0:n.calnev.y,x1:n.nvLasVegas.x,y1:n.nvLasVegas.y,w:sc(v.caTotal*0.08),vol:v.caTotal*0.08,color:"#fbbf24",state:"nv"},
    {id:"colton-wl",x0:n.colton.x,y0:n.colton.y,x1:n.westLine.x,y1:n.westLine.y,w:sc(v.caTotal*0.25),vol:v.caTotal*0.25,color:"#f59e0b",state:"az"},
    {id:"caRef-soCal",x0:n.caRefinery.x,y0:n.caRefinery.y,x1:n.caSoCal.x,y1:n.caSoCal.y,w:sc(v.caTotal*0.55),vol:v.caTotal*0.55,color:"#e879f9",state:"ca"},
    {id:"caRef-wl",x0:n.caRefinery.x,y0:n.caRefinery.y,x1:n.westLine.x,y1:n.westLine.y,w:sc(v.caTotal*0.20),vol:v.caTotal*0.20,color:"#e879f9",state:"ca"},
    {id:"bayRef-bay",x0:n.caAlaska.x,y0:n.caAlaska.y,x1:n.caBayArea.x,y1:n.caBayArea.y,w:sc(v.caTotal*0.08),vol:v.caTotal*0.08,color:"#34d399",state:"ca"},
    {id:"bayRef-nor",x0:n.caAlaska.x,y0:n.caAlaska.y,x1:n.caNorCal.x,y1:n.caNorCal.y,w:sc(v.caTotal*0.04),vol:v.caTotal*0.04,color:"#34d399",state:"ca"},
    {id:"bayRef-cv",x0:n.caAlaska.x,y0:n.caAlaska.y,x1:n.caCentVal.x,y1:n.caCentVal.y,w:sc(v.caTotal*0.12),vol:v.caTotal*0.12,color:"#34d399",state:"ca"},
    {id:"wl-cv",x0:n.westLine.x,y0:n.westLine.y,x1:n.caCentVal.x,y1:n.caCentVal.y,w:sc(v.caTotal*0.05),vol:v.caTotal*0.05,color:"#fb7185",state:"ca"},
    {id:"wl-bay",x0:n.westLine.x,y0:n.westLine.y,x1:n.caBayArea.x,y1:n.caBayArea.y,w:sc(v.caTotal*0.10),vol:v.caTotal*0.10,color:"#fb923c",state:"ca"},
    {id:"wl-azPhx",x0:n.westLine.x,y0:n.westLine.y,x1:n.azPhx.x,y1:n.azPhx.y,w:sc(v.azWest*0.8),vol:v.azWest*0.8,color:"#f59e0b",state:"az"},
    {id:"el-azPhx",x0:n.eastLine.x,y0:n.eastLine.y,x1:n.azPhx.x,y1:n.azPhx.y,w:sc(v.azEast*0.3),vol:v.azEast*0.3,color:"#fb923c",state:"az"},
    {id:"el-azTuc",x0:n.eastLine.x,y0:n.eastLine.y,x1:n.azTuc.x,y1:n.azTuc.y,w:sc(v.azEast*0.7),vol:v.azEast*0.7,color:"#fb923c",state:"az"},
    {id:"wl-azTuc",x0:n.westLine.x,y0:n.westLine.y,x1:n.azTuc.x,y1:n.azTuc.y,w:sc(v.azWest*0.2),vol:v.azWest*0.2,color:"#f59e0b",state:"az"},
    {id:"azPhx-g",x0:n.azPhx.x,y0:n.azPhx.y,x1:n.azPhxGrp.x,y1:n.azPhxGrp.y,w:sc(v.azPhx),vol:v.azPhx,color:"#38bdf8",state:"az"},
    {id:"azTuc-g",x0:n.azTuc.x,y0:n.azTuc.y,x1:n.azTucGrp.x,y1:n.azTucGrp.y,w:sc(v.azTuc),vol:v.azTuc,color:"#818cf8",state:"az"},
  ];
}

function getAzCounties(yi){
  const phx=AZ_PHX_COUNTIES.map(c=>({name:c,vol:(AZ_COUNTY_DATA[c]?.[yi]??0)*AZ_FUELS_SCALE,term:"phx"})).sort((a,b)=>b.vol-a.vol);
  const tuc=AZ_TUC_COUNTIES.map(c=>({name:c,vol:(AZ_COUNTY_DATA[c]?.[yi]??0)*AZ_FUELS_SCALE,term:"tuc"})).sort((a,b)=>b.vol-a.vol);
  const all=[...phx,...tuc];
  return all.map((c,i)=>({...c,x:COL.azCo,y:30+i*((H-60)/(all.length-1))}));
}

function spawnParticles(yi){
  const ribbons=getRibbons(yi), azC=getAzCounties(yi);
  const maxV=CA_STATEWIDE[yi]??CA_STATEWIDE[24];
  const ps=[];
  ribbons.forEach(r=>{
    const count=Math.max(3,Math.round((r.vol/maxV)*120));
    for(let i=0;i<count;i++) ps.push({x0:r.x0,y0:r.y0,x1:r.x1,y1:r.y1,t:Math.random(),speed:0.0003+Math.random()*0.0005,color:r.color,size:1.8+Math.random()*1.5,offset:(Math.random()-0.5)*r.w*0.5,alpha:0.55+Math.random()*0.4,state:r.state,dashed:r.dashed||false});
  });
  azC.forEach(c=>{
    const grpY=c.term==="phx"?H2-100:H2+100;
    const maxAZ=Math.max(...Object.values(AZ_COUNTY_DATA).map(d=>d[yi]??0));
    const count=Math.max(2,Math.round((c.vol/maxAZ)*14));
    for(let i=0;i<count;i++) ps.push({x0:COL.azDisp,y0:grpY,x1:c.x,y1:c.y,t:Math.random(),speed:0.0005+Math.random()*0.0005,color:c.term==="phx"?"#38bdf8":"#818cf8",size:1.2+Math.random(),offset:(Math.random()-0.5)*6,alpha:0.45+Math.random()*0.4,state:"az",dashed:false});
  });
  return ps;
}

// ═══════════════════════════════════════════════════════════════════
// DRAW FUNCTIONS
// ═══════════════════════════════════════════════════════════════════
function drawRibbon(ctx,r,dimmed){
  const dx=r.x1-r.x0,dy=r.y1-r.y0,len=Math.sqrt(dx*dx+dy*dy)||1;
  const nx=-dy/len,ny=dx/len;
  const cp1x=r.x0+dx*0.4,cp2x=r.x0+dx*0.6,hw=r.w*0.5;
  if(r.dashed){
    [[nx,ny],[-nx,-ny]].forEach(([nx2,ny2])=>{
      ctx.beginPath();ctx.moveTo(r.x0+nx2*hw,r.y0+ny2*hw);
      ctx.bezierCurveTo(cp1x+nx2*hw*0.8,r.y0+ny2*hw*0.8,cp2x+nx2*hw*0.8,r.y1+ny2*hw*0.8,r.x1+nx2*hw,r.y1+ny2*hw);
      ctx.strokeStyle=r.color+(dimmed?"22":"99");ctx.lineWidth=hw;ctx.setLineDash([8,5]);ctx.stroke();ctx.setLineDash([]);
    });return;
  }
  ctx.beginPath();ctx.moveTo(r.x0+nx*hw,r.y0+ny*hw);
  ctx.bezierCurveTo(cp1x+nx*hw*0.8,r.y0+ny*hw*0.8,cp2x+nx*hw*0.8,r.y1+ny*hw*0.8,r.x1+nx*hw,r.y1+ny*hw);
  ctx.bezierCurveTo(cp2x-nx*hw*0.8,r.y1-ny*hw*0.8,cp1x-nx*hw*0.8,r.y0-ny*hw*0.8,r.x0-nx*hw,r.y0-ny*hw);
  ctx.closePath();
  const g=ctx.createLinearGradient(r.x0,r.y0,r.x1,r.y1);
  const a=dimmed?"18":"44",b=dimmed?"08":"20";
  g.addColorStop(0,r.color+a);g.addColorStop(0.5,r.color+b);g.addColorStop(1,r.color+a);
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=r.color+(dimmed?"22":"55");ctx.lineWidth=0.5;ctx.stroke();
}

function drawNode(ctx,node,dimmed,maxV){
  const r=4+(node.vol/maxV)*45,alpha=dimmed?0.25:1;
  ctx.globalAlpha=alpha;
  ctx.beginPath();ctx.arc(node.x,node.y,r,0,Math.PI*2);
  const g=ctx.createRadialGradient(node.x,node.y,0,node.x,node.y,r);
  g.addColorStop(0,node.color+"cc");g.addColorStop(1,node.color+"33");
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=node.color;ctx.lineWidth=1.5;ctx.stroke();
  ctx.fillStyle="#e8dcc8";ctx.font="bold 14px Georgia";ctx.textAlign="center";
  ctx.fillText(node.label,node.x,node.y-r-6);
  if(node.sub){ctx.fillStyle="#6a7590";ctx.font="12px Georgia";ctx.fillText(node.sub,node.x,node.y-r-16);}
  ctx.fillStyle=node.color;ctx.font="bold 11px monospace";ctx.fillText(fmt(node.vol)+"gal",node.x,node.y+r+12);
  ctx.globalAlpha=1;
}

function drawRefinery(ctx,node,dimmed,maxV){
  const scale=4+(node.vol/maxV)*40,w=scale*0.6,h=scale*1.8,rx=3,alpha=dimmed?0.25:1;
  ctx.globalAlpha=alpha;
  ctx.beginPath();ctx.moveTo(node.x-w+rx,node.y-h);ctx.lineTo(node.x+w-rx,node.y-h);
  ctx.arcTo(node.x+w,node.y-h,node.x+w,node.y-h+rx,rx);ctx.lineTo(node.x+w,node.y+h-rx);
  ctx.arcTo(node.x+w,node.y+h,node.x+w-rx,node.y+h,rx);ctx.lineTo(node.x-w+rx,node.y+h);
  ctx.arcTo(node.x-w,node.y+h,node.x-w,node.y+h-rx,rx);ctx.lineTo(node.x-w,node.y-h+rx);
  ctx.arcTo(node.x-w,node.y-h,node.x-w+rx,node.y-h,rx);ctx.closePath();
  const g=ctx.createLinearGradient(node.x,node.y-h,node.x,node.y+h);
  g.addColorStop(0,node.color+"dd");g.addColorStop(0.5,node.color+"88");g.addColorStop(1,node.color+"33");
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=node.color;ctx.lineWidth=1.5;ctx.stroke();
  ctx.strokeStyle=node.color+"44";ctx.lineWidth=0.5;
  [-w*0.3,0,w*0.3].forEach(ox=>{ctx.beginPath();ctx.moveTo(node.x+ox,node.y-h+4);ctx.lineTo(node.x+ox,node.y+h-4);ctx.stroke();});
  ctx.fillStyle="#e8dcc8";ctx.font="bold 14px Georgia";ctx.textAlign="center";ctx.fillText(node.label,node.x,node.y-h-6);
  if(node.sub){ctx.fillStyle="#6a7590";ctx.font="12px Georgia";ctx.fillText(node.sub,node.x,node.y-h-16);}
  ctx.fillStyle=node.color;ctx.font="bold 11px monospace";ctx.fillText(fmt(node.vol)+"gal",node.x,node.y+h+12);
  ctx.globalAlpha=1;
}

function drawJunction(ctx,node,dimmed,maxV){
  const scale=4+(node.vol/maxV)*35,r=scale*0.9,alpha=dimmed?0.25:1;
  ctx.globalAlpha=alpha;
  ctx.beginPath();ctx.moveTo(node.x,node.y-r);ctx.lineTo(node.x+r,node.y);ctx.lineTo(node.x,node.y+r);ctx.lineTo(node.x-r,node.y);ctx.closePath();
  const g=ctx.createRadialGradient(node.x,node.y,0,node.x,node.y,r);
  g.addColorStop(0,node.color+"cc");g.addColorStop(1,node.color+"33");
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=node.color;ctx.lineWidth=1.5;ctx.stroke();
  ctx.beginPath();ctx.moveTo(node.x,node.y-r*0.5);ctx.lineTo(node.x+r*0.5,node.y);ctx.lineTo(node.x,node.y+r*0.5);ctx.lineTo(node.x-r*0.5,node.y);ctx.closePath();
  ctx.strokeStyle=node.color+"44";ctx.lineWidth=0.5;ctx.stroke();
  ctx.fillStyle="#e8dcc8";ctx.font="bold 14px Georgia";ctx.textAlign="center";ctx.fillText(node.label,node.x,node.y-r-6);
  if(node.sub){ctx.fillStyle="#6a7590";ctx.font="12px Georgia";ctx.fillText(node.sub,node.x,node.y-r-16);}
  ctx.fillStyle=node.color;ctx.font="bold 11px monospace";ctx.fillText(fmt(node.vol)+"gal",node.x,node.y+r+12);
  ctx.globalAlpha=1;
}

function drawPipe(ctx,node,dimmed,maxV){
  const scale=4+(node.vol/maxV)*40,w=scale*1.8,h=scale*0.5,rx=h,alpha=dimmed?0.25:1;
  ctx.globalAlpha=alpha;
  ctx.beginPath();ctx.moveTo(node.x-w+rx,node.y-h);ctx.lineTo(node.x+w-rx,node.y-h);
  ctx.arcTo(node.x+w,node.y-h,node.x+w,node.y,rx);ctx.arcTo(node.x+w,node.y+h,node.x+w-rx,node.y+h,rx);
  ctx.lineTo(node.x-w+rx,node.y+h);ctx.arcTo(node.x-w,node.y+h,node.x-w,node.y,rx);ctx.arcTo(node.x-w,node.y-h,node.x-w+rx,node.y-h,rx);ctx.closePath();
  const g=ctx.createLinearGradient(node.x-w,node.y,node.x+w,node.y);
  g.addColorStop(0,node.color+"55");g.addColorStop(0.5,node.color+"cc");g.addColorStop(1,node.color+"55");
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=node.color;ctx.lineWidth=1.5;ctx.stroke();
  ctx.strokeStyle=node.color+"33";ctx.lineWidth=0.5;
  [-h*0.3,0,h*0.3].forEach(oy=>{ctx.beginPath();ctx.moveTo(node.x-w+rx+4,node.y+oy);ctx.lineTo(node.x+w-rx-4,node.y+oy);ctx.stroke();});
  ctx.fillStyle="#e8dcc8";ctx.font="bold 14px Georgia";ctx.textAlign="center";ctx.fillText(node.label,node.x,node.y-h-6);
  if(node.sub){ctx.fillStyle="#6a7590";ctx.font="12px Georgia";ctx.fillText(node.sub,node.x,node.y-h-16);}
  ctx.fillStyle=node.color;ctx.font="bold 11px monospace";ctx.fillText(fmt(node.vol)+"gal",node.x,node.y+h+12);
  ctx.globalAlpha=1;
}

function drawTerminal(ctx,node,dimmed,maxV){
  const scale=4+(node.vol/maxV)*45,r=scale*0.9,alpha=dimmed?0.25:1;
  ctx.globalAlpha=alpha;
  ctx.beginPath();ctx.moveTo(node.x,node.y-r);ctx.lineTo(node.x+r*0.866,node.y+r*0.5);ctx.lineTo(node.x-r*0.866,node.y+r*0.5);ctx.closePath();
  const g=ctx.createRadialGradient(node.x,node.y,0,node.x,node.y,r);
  g.addColorStop(0,node.color+"cc");g.addColorStop(1,node.color+"33");
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=node.color;ctx.lineWidth=1.5;ctx.stroke();
  ctx.fillStyle="#e8dcc8";ctx.font="bold 14px Georgia";ctx.textAlign="center";ctx.fillText(node.label,node.x,node.y-r-6);
  if(node.sub){ctx.fillStyle="#6a7590";ctx.font="12px Georgia";ctx.fillText(node.sub,node.x,node.y-r-16);}
  ctx.fillStyle=node.color;ctx.font="bold 11px monospace";ctx.fillText(fmt(node.vol)+"gal",node.x,node.y+r*0.5+12);
  ctx.globalAlpha=1;
}

function drawLocalSupply(ctx,node,dimmed,maxV){
  const scale=4+(node.vol/maxV)*40,tw=scale*0.8,bw=scale*1.4,h=scale*0.8,alpha=dimmed?0.25:1;
  ctx.globalAlpha=alpha;
  ctx.beginPath();ctx.moveTo(node.x-tw,node.y-h);ctx.lineTo(node.x+tw,node.y-h);ctx.lineTo(node.x+bw,node.y+h);ctx.lineTo(node.x-bw,node.y+h);ctx.closePath();
  const g=ctx.createLinearGradient(node.x,node.y+h,node.x,node.y-h);
  g.addColorStop(0,node.color+"dd");g.addColorStop(1,node.color+"44");
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=node.color;ctx.lineWidth=1.5;ctx.stroke();
  ctx.strokeStyle=node.color+"88";ctx.lineWidth=2;ctx.beginPath();ctx.moveTo(node.x-bw-4,node.y+h);ctx.lineTo(node.x+bw+4,node.y+h);ctx.stroke();
  ctx.strokeStyle=node.color+"44";ctx.lineWidth=0.5;ctx.setLineDash([2,3]);
  ctx.beginPath();ctx.moveTo(node.x,node.y+h);ctx.lineTo(node.x,node.y+h+10);ctx.stroke();ctx.setLineDash([]);
  ctx.fillStyle="#e8dcc8";ctx.font="bold 14px Georgia";ctx.textAlign="center";ctx.fillText(node.label,node.x,node.y-h-6);
  if(node.sub){ctx.fillStyle="#6a7590";ctx.font="12px Georgia";ctx.fillText(node.sub,node.x,node.y-h-16);}
  ctx.fillStyle=node.color;ctx.font="bold 11px monospace";ctx.fillText(fmt(node.vol)+"gal",node.x,node.y+h+22);
  ctx.globalAlpha=1;
}

function drawTanker(ctx,node,dimmed,maxV,yi){
  const scale=4+(node.vol/maxV)*40,hw=scale*1.4,bw=scale*1.0,h=scale*0.8,alpha=dimmed?0.25:1;
  ctx.globalAlpha=alpha;
  ctx.beginPath();ctx.moveTo(node.x-hw,node.y-h);ctx.lineTo(node.x+hw,node.y-h);ctx.lineTo(node.x+bw,node.y+h);ctx.lineTo(node.x-bw,node.y+h);ctx.closePath();
  const g=ctx.createLinearGradient(node.x,node.y-h,node.x,node.y+h);
  g.addColorStop(0,node.color+"dd");g.addColorStop(1,node.color+"44");
  ctx.fillStyle=g;ctx.fill();ctx.strokeStyle=node.color;ctx.lineWidth=1.5;ctx.stroke();
  ctx.strokeStyle=node.color+"55";ctx.lineWidth=0.5;
  ctx.beginPath();ctx.moveTo(node.x-hw*0.6,node.y-h+3);ctx.lineTo(node.x+hw*0.6,node.y-h+3);ctx.stroke();
  ctx.beginPath();ctx.moveTo(node.x-hw*0.3,node.y-h+6);ctx.lineTo(node.x+hw*0.3,node.y-h+6);ctx.stroke();
  ctx.fillStyle="#e8dcc8";ctx.font="bold 14px Georgia";ctx.textAlign="center";ctx.fillText(node.label,node.x,node.y-h-6);
  if(node.sub){ctx.fillStyle="#6a7590";ctx.font="12px Georgia";ctx.fillText(node.sub,node.x,node.y-h-16);}
  ctx.fillStyle=node.color;ctx.font="bold 11px monospace";ctx.fillText(fmt(node.vol)+"gal",node.x,node.y+h+12);
  if(node.countries){
    node.countries.slice(0,4).forEach((c,i)=>{ctx.fillStyle=node.color+"cc";ctx.font="9px Georgia";ctx.fillText(c,node.x,node.y+h+22+i*12);});
  }
  const note=SUPPLY_NOTES[yi];
  if(note&&node.label==="LA Tankers"){ctx.fillStyle="#fbbf24cc";ctx.font="bold 8px Georgia";ctx.fillText(note,node.x,node.y+h+74);}
  ctx.globalAlpha=1;
}

function drawVulnerabilityRing(ctx,node,score,frameN,dimmed,maxV){
  if(score<=2.0)return;
  const r=4+(node.vol/maxV)*45,t=(score-2)/3;
  const green=Math.round(60+(1-t)*120),color=`rgb(255,${green},0)`;
  const pulseSpeed=0.04+t*0.06,pulsePhase=(frameN*pulseSpeed)%(Math.PI*2);
  const pulseScale=1+Math.sin(pulsePhase)*0.35,ringR=(r+8+t*6)*pulseScale;
  const alpha=dimmed?0.05:(0.25+Math.sin(pulsePhase)*0.2)*(0.4+t*0.6);
  ctx.globalAlpha=alpha;ctx.beginPath();ctx.arc(node.x,node.y,ringR,0,Math.PI*2);
  ctx.strokeStyle=color;ctx.lineWidth=1.5+t*1.5;ctx.stroke();
  if(score>=4.5){
    const ring2R=ringR+6+Math.sin(pulsePhase+1)*4;
    ctx.globalAlpha=alpha*0.4;ctx.beginPath();ctx.arc(node.x,node.y,ring2R,0,Math.PI*2);ctx.lineWidth=0.8;ctx.stroke();
  }
  ctx.globalAlpha=1;
}

function drawParticle(ctx,p,dimmed){
  const cp1x=p.x0+(p.x1-p.x0)*0.4,cp2x=p.x0+(p.x1-p.x0)*0.6;
  const pos=cubicBezier([p.x0,p.y0],[cp1x,p.y0],[cp2x,p.y1],[p.x1,p.y1],p.t);
  const pos2=cubicBezier([p.x0,p.y0],[cp1x,p.y0],[cp2x,p.y1],[p.x1,p.y1],Math.min(p.t+0.01,1));
  const dx=pos2[0]-pos[0],dy=pos2[1]-pos[1],len=Math.sqrt(dx*dx+dy*dy)||1;
  const x=pos[0]+(-dy/len)*p.offset,y=pos[1]+(dx/len)*p.offset;
  ctx.globalAlpha=dimmed?p.alpha*0.15:p.alpha;
  ctx.beginPath();ctx.arc(x,y,p.size,0,Math.PI*2);ctx.fillStyle=p.color;ctx.fill();
  ctx.beginPath();ctx.arc(x,y,p.size*2,0,Math.PI*2);ctx.fillStyle=p.color+"18";ctx.fill();
  ctx.globalAlpha=1;
}

// ═══════════════════════════════════════════════════════════════════
// MAIN DRAW LOOP
// ═══════════════════════════════════════════════════════════════════
const canvas = document.getElementById("mainCanvas");
const ctx = canvas.getContext("2d");
canvas.width = W; canvas.height = H;

function draw(){
  ctx.clearRect(0,0,W,H);
  ctx.fillStyle="#07080d";ctx.fillRect(0,0,W,H);
  ctx.strokeStyle="#0d0f18";ctx.lineWidth=0.5;
  for(let x=0;x<W;x+=40){ctx.beginPath();ctx.moveTo(x,0);ctx.lineTo(x,H);ctx.stroke();}
  for(let y=0;y<H;y+=40){ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(W,y);ctx.stroke();}

  // Dividers
  ctx.strokeStyle="#1a1f2e";ctx.lineWidth=1;ctx.setLineDash([6,10]);
  [COL.westLine-30,COL.azTerm-30].forEach(x=>{ctx.beginPath();ctx.moveTo(x,40);ctx.lineTo(x,H-10);ctx.stroke();});
  ctx.setLineDash([]);

  // Labels
  ctx.font="bold 11px monospace";ctx.textAlign="center";
  ctx.fillStyle="#e879f922";ctx.fillText("CALIFORNIA",(COL.caRef+COL.westLine-30)/2,22);
  ctx.fillStyle="#38bdf822";ctx.fillText("ARIZONA",(COL.azTerm-30+COL.eastLine)/2,22);
  ctx.fillStyle="#f59e0b44";ctx.fillText("SHARED",COL.westLine-30+30,22);

  const cols=[
    {x:COL.tanker,l:"TANKERS",s:"Marine Imports"},
    {x:COL.caRef,l:"ORIGIN",s:"CA Refineries"},
    {x:COL.caReg,l:"CA REGIONS",s:"Distribution"},
    {x:COL.westLine,l:"WEST LINE",s:"SFPP Trunk"},
    {x:COL.azTerm,l:"TERMINALS",s:"AZ Hubs"},
    {x:COL.azDisp,l:"DISPATCH",s:"Truck Load"},
    {x:COL.azCo,l:"COUNTIES",s:"AZ Delivery"},
    {x:COL.eastLine,l:"EAST LINE",s:"El Paso →"},
  ];
  cols.forEach(c=>{
    ctx.fillStyle="#2a3040";ctx.font="bold 9px monospace";ctx.textAlign="center";ctx.fillText(c.l,c.x,36);
    ctx.fillStyle="#1e2535";ctx.font="12px Georgia";ctx.fillText(c.s,c.x,47);
    ctx.strokeStyle="#1a1f2e";ctx.lineWidth=1;ctx.setLineDash([4,8]);
    ctx.beginPath();ctx.moveTo(c.x,50);ctx.lineTo(c.x,H-10);ctx.stroke();ctx.setLineDash([]);
  });

  const yi=yearIndex;
  const ribbons=getRibbons(yi);
  const nodes=getNodes(yi);
  const azC=getAzCounties(yi);
  const v=getVolumes(yi);
  const maxAZv=Math.max(...azC.map(c=>c.vol));

  // Ribbons
  ribbons.forEach(r=>{
    const dimmed=activeState!=="both"&&r.state!==activeState;
    drawRibbon(ctx,r,dimmed);
  });

  // County arcs
  azC.forEach(c=>{
    const grpY=c.term==="phx"?H2-100:H2+100;
    const color=c.term==="phx"?"#38bdf8":"#818cf8";
    const w=Math.max(1,(c.vol/maxAZv)*16);
    const dimmed=activeState==="ca";
    ctx.globalAlpha=dimmed?0.05:0.25;
    ctx.beginPath();ctx.moveTo(COL.azDisp,grpY);
    ctx.bezierCurveTo(COL.azDisp+40,grpY,COL.azCo-40,c.y,COL.azCo,c.y);
    ctx.strokeStyle=color;ctx.lineWidth=w;ctx.stroke();ctx.globalAlpha=1;
    ctx.globalAlpha=dimmed?0.1:1;
    const r2=2+(c.vol/maxAZv)*11;
    ctx.beginPath();ctx.arc(c.x,c.y,r2,0,Math.PI*2);ctx.fillStyle=color+"44";ctx.fill();
    ctx.strokeStyle=color+"88";ctx.lineWidth=0.8;ctx.stroke();
    ctx.fillStyle="#9ab0c8";ctx.font="12px Georgia";ctx.textAlign="left";ctx.fillText(c.name,c.x+r2+3,c.y+3);
    ctx.globalAlpha=1;
  });

  // Shared maxV
  const allVols=[nodes.laTanker.vol,nodes.bayTanker.vol,nodes.akTanker.vol,nodes.localSupply.vol,nodes.caRefinery.vol,nodes.caAlaska.vol,nodes.caSoCal.vol,nodes.caBayArea.vol,nodes.caCentVal.vol,nodes.caNorCal.vol,nodes.colton.vol,nodes.calnev.vol,nodes.westLine.vol,nodes.nvLasVegas.vol,nodes.azPhx.vol,nodes.azTuc.vol,nodes.azPhxGrp.vol,nodes.azTucGrp.vol,nodes.eastLine.vol];
  const sharedMaxV=Math.max(...allVols);

  // Vulnerability rings
  if(showVuln){
    const vulnList=[["nvLasVegas","nv"],["azPhx","az"],["azTuc","az"],["azPhxGrp","az"],["azTucGrp","az"],["caNorCal","ca"],["caCentVal","ca"],["caBayArea","ca"],["caSoCal","ca"]];
    vulnList.forEach(([key,s])=>{
      const dimmed=activeState!=="both"&&s!==activeState;
      drawVulnerabilityRing(ctx,nodes[key],VULN[key],frame,dimmed,sharedMaxV);
    });
  }

  // Nodes
  const nodeDefs=[
    {n:nodes.laTanker,s:"ca",shape:"tanker"},{n:nodes.bayTanker,s:"ca",shape:"tanker"},
    {n:nodes.akTanker,s:"ca",shape:"tanker"},{n:nodes.localSupply,s:"ca",shape:"local"},
    {n:nodes.caRefinery,s:"ca",shape:"refinery"},{n:nodes.caAlaska,s:"ca",shape:"refinery"},
    {n:nodes.caSoCal,s:"ca",shape:"circle"},{n:nodes.caBayArea,s:"ca",shape:"circle"},
    {n:nodes.caCentVal,s:"ca",shape:"circle"},{n:nodes.caNorCal,s:"ca",shape:"circle"},
    {n:nodes.colton,s:"ca",shape:"junction"},{n:nodes.calnev,s:"nv",shape:"pipe"},
    {n:nodes.westLine,s:"both",shape:"pipe"},{n:nodes.nvLasVegas,s:"nv",shape:"terminal"},
    {n:nodes.azPhx,s:"az",shape:"terminal"},{n:nodes.azTuc,s:"az",shape:"terminal"},
    {n:nodes.azPhxGrp,s:"az",shape:"circle"},{n:nodes.azTucGrp,s:"az",shape:"circle"},
    {n:nodes.eastLine,s:"az",shape:"pipe"},
  ];
  nodeDefs.forEach(({n,s,shape})=>{
    const dimmed=activeState!=="both"&&s!=="both"&&s!==activeState;
    if(shape==="tanker") drawTanker(ctx,n,dimmed,sharedMaxV,yi);
    else if(shape==="local") drawLocalSupply(ctx,n,dimmed,sharedMaxV);
    else if(shape==="refinery") drawRefinery(ctx,n,dimmed,sharedMaxV);
    else if(shape==="junction") drawJunction(ctx,n,dimmed,sharedMaxV);
    else if(shape==="pipe") drawPipe(ctx,n,dimmed,sharedMaxV);
    else if(shape==="terminal") drawTerminal(ctx,n,dimmed,sharedMaxV);
    else drawNode(ctx,n,dimmed,sharedMaxV);
  });

  // Particles
  const speedMult = playing ? 3.0 : 1.0;
  particles.forEach(p=>{
    p.t+=p.speed*speedMult;if(p.t>1)p.t=0;
    const dimmed=activeState!=="both"&&p.state!==activeState;
    drawParticle(ctx,p,dimmed);
  });

  // FY overlay
  ctx.fillStyle="#f59e0b";ctx.font="bold 22px Georgia";ctx.textAlign="center";ctx.globalAlpha=0.9;
  ctx.fillText("FY "+FISCAL_YEARS[yi],W/2,22);ctx.globalAlpha=1;
  ctx.font="bold 48px Georgia";ctx.textAlign="right";ctx.globalAlpha=0.06;
  ctx.fillText("FY"+FISCAL_YEARS[yi],W-16,H-16);ctx.globalAlpha=1;

  frame++;
  animId=requestAnimationFrame(draw);
}

// ═══════════════════════════════════════════════════════════════════
// UI UPDATES
// ═══════════════════════════════════════════════════════════════════
function updateUI(){
  const yi=yearIndex;
  const v=getVolumes(yi);
  document.getElementById("fy-display").textContent="FY"+FISCAL_YEARS[yi];
  document.getElementById("stat-ca").textContent=fmtFull(v.caTotal);
  document.getElementById("stat-wl").textContent=fmtFull(v.westLineTotal);
  document.getElementById("stat-az").textContent=fmtFull(v.azTotal);
}

function setYear(yi){
  yearIndex=Math.round(yi);
  document.getElementById("yearSlider").value=yearIndex;
  updateUI();
}

// ═══════════════════════════════════════════════════════════════════
// AUTOPLAY
// ═══════════════════════════════════════════════════════════════════
function startPlay(){
  playing=true;
  document.getElementById("play-btn").textContent="■ PAUSE";
  document.getElementById("play-btn").classList.add("playing");
  playInterval=setInterval(()=>{
    yearFloat+=0.03;if(yearFloat>=25)yearFloat=0;
    setYear(Math.round(yearFloat));
  },80);
}
function stopPlay(){
  playing=false;
  document.getElementById("play-btn").textContent="▶ PLAY";
  document.getElementById("play-btn").classList.remove("playing");
  clearInterval(playInterval);
}

// ═══════════════════════════════════════════════════════════════════
// TOOLTIPS
// ═══════════════════════════════════════════════════════════════════
function showVulnTooltip(key,clientX,clientY){
  const d=VULN_DETAIL[key];
  const score=VULN[key];
  const t=(score-2)/3;
  const borderColor=`rgb(255,${Math.round(60+(1-t)*120)},0)`;
  const tt=document.getElementById("vuln-tooltip");
  tt.style.borderColor=borderColor;
  tt.style.boxShadow=`0 0 20px ${borderColor}44`;
  document.getElementById("vt-title").style.color=borderColor;
  document.getElementById("vt-title").textContent=d.label;
  document.getElementById("vt-score").textContent="Score "+d.score;
  document.getElementById("vt-incident-main").textContent=d.incidents||"";
  document.getElementById("vt-incident-detail").textContent=d.incidentDetail||"";
  document.getElementById("vt-incident-box").style.display=d.incidents?"block":"none";
  const reasons=document.getElementById("vt-reasons");
  reasons.innerHTML=d.reasons.map(r=>`<div class="tt-reason"><span class="tt-bullet" style="color:${borderColor}">▸</span><span class="tt-reason-text">${r}</span></div>`).join("");
  tt.style.left=(clientX+16)+"px";tt.style.top=(clientY-10)+"px";tt.style.display="block";
}
function hideVulnTooltip(){document.getElementById("vuln-tooltip").style.display="none";}

function showCountyTooltip(county,clientX,clientY){
  const mix=COUNTY_FUEL_MIX[county.name];if(!mix)return;
  document.getElementById("ct-name").textContent=county.name+" County";
  document.getElementById("ct-vol").textContent=fmtFull(county.vol)+" total · all fuels";
  // Pie chart
  const svg=document.getElementById("ct-pie");
  svg.innerHTML="";
  const cx=44,cy=44,r=36;let startAngle=-Math.PI/2;
  const legend=document.getElementById("ct-legend");legend.innerHTML="";
  Object.entries(mix).forEach(([key,pct])=>{
    const angle=pct*Math.PI*2;
    const x1=cx+Math.cos(startAngle)*r,y1=cy+Math.sin(startAngle)*r;
    const x2=cx+Math.cos(startAngle+angle)*r,y2=cy+Math.sin(startAngle+angle)*r;
    const large=angle>Math.PI?1:0;
    const path=document.createElementNS("http://www.w3.org/2000/svg","path");
    path.setAttribute("d",`M ${cx} ${cy} L ${x1} ${y1} A ${r} ${r} 0 ${large} 1 ${x2} ${y2} Z`);
    path.setAttribute("fill",FUEL_COLORS[key]);path.setAttribute("opacity","0.85");
    path.setAttribute("stroke","#0d1117");path.setAttribute("stroke-width","1");
    svg.appendChild(path);
    legend.innerHTML+=`<div class="pie-item"><div class="pie-swatch" style="background:${FUEL_COLORS[key]}"></div><div><div class="pie-label">${FUEL_LABELS[key]}</div><div class="pie-pct">${(pct*100).toFixed(0)}% · ${fmtFull(county.vol*pct)}</div></div></div>`;
    startAngle+=angle;
  });
  const tt=document.getElementById("county-tooltip");
  tt.style.left=(clientX+16)+"px";tt.style.top=(clientY-10)+"px";tt.style.display="block";
}
function hideCountyTooltip(){document.getElementById("county-tooltip").style.display="none";}

// ═══════════════════════════════════════════════════════════════════
// MOUSE HANDLERS
// ═══════════════════════════════════════════════════════════════════
canvas.addEventListener("mousemove",e=>{
  const rect=canvas.getBoundingClientRect();
  const scaleX=W/rect.width;
  const mx=(e.clientX-rect.left)*scaleX,my=(e.clientY-rect.top)*scaleX;
  let foundVuln=false;
  if(showVuln){
    const nodes=getNodes(yearIndex);
    for(const key of Object.keys(VULN_DETAIL)){
      const node=nodes[key];if(!node)continue;
      const dx=mx-node.x,dy=my-node.y;
      if(Math.sqrt(dx*dx+dy*dy)<40){showVulnTooltip(key,e.clientX,e.clientY);foundVuln=true;break;}
    }
  }
  if(!foundVuln)hideVulnTooltip();
  const azC=getAzCounties(yearIndex);
  const maxAZv=Math.max(...azC.map(c=>c.vol));
  let foundCounty=false;
  for(const c of azC){
    const r=2+(c.vol/maxAZv)*11+6,dx=mx-c.x,dy=my-c.y;
    if(Math.sqrt(dx*dx+dy*dy)<r){showCountyTooltip(c,e.clientX,e.clientY);foundCounty=true;break;}
  }
  if(!foundCounty)hideCountyTooltip();
});
canvas.addEventListener("mouseleave",()=>{hideVulnTooltip();hideCountyTooltip();});

// ═══════════════════════════════════════════════════════════════════
// BUTTON HANDLERS
// ═══════════════════════════════════════════════════════════════════
document.getElementById("play-btn").addEventListener("click",()=>{
  if(playing)stopPlay();else startPlay();
});

document.getElementById("yearSlider").addEventListener("input",e=>{
  yearFloat=+e.target.value;
  yearIndex=Math.round(yearFloat);
  document.getElementById("yearSlider").value=yearIndex;
  particles=spawnParticles(yearIndex);
  updateUI();
});

document.getElementById("vuln-btn").addEventListener("click",()=>{
  showVuln=!showVuln;
  const btn=document.getElementById("vuln-btn");
  btn.textContent="⚡ VULNERABILITY "+(showVuln?"ON":"OFF");
  btn.classList.toggle("on",showVuln);
  canvas.style.cursor=showVuln?"crosshair":"default";
});

document.querySelectorAll(".filter-btn").forEach(btn=>{
  btn.addEventListener("click",()=>{
    activeState=btn.dataset.state;
    document.querySelectorAll(".filter-btn").forEach(b=>b.classList.remove("active"));
    btn.classList.add("active");
  });
});

document.getElementById("fs-btn").addEventListener("click",()=>{
  if(!document.fullscreenElement)document.getElementById("app").requestFullscreen();
  else document.exitFullscreen();
});

// ═══════════════════════════════════════════════════════════════════
// INIT
// ═══════════════════════════════════════════════════════════════════
particles=spawnParticles(yearIndex);
updateUI();
draw();
startPlay();
</script>
</body>
</html>
