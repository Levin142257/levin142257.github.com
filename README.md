<!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Blockfront — 2D Tower Defense</title>
  <style>
    * { box-sizing: border-box; }

    html, body {
      width: 100%;
      height: 100%;
      margin: 0;
      overflow: hidden;
      background: #172430;
      color: #f3f6ff;
      font-family: system-ui, sans-serif;
    }

    canvas {
      display: block;
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      touch-action: none;
    }

    .panel {
      background: linear-gradient(145deg, rgba(28,38,48,.94), rgba(15,23,31,.96));
      border: 1px solid rgba(215,229,220,.16);
      border-radius: 12px;
      box-shadow: 0 14px 34px rgba(0,0,0,.32), inset 0 1px rgba(255,255,255,.04);
      backdrop-filter: blur(10px);
    }

    #menu {
      position: fixed;
      z-index: 100;
      inset: 0;
      display: grid;
      place-items: center;
      overflow-y: auto;
      padding: 14px 0;
      background:
        radial-gradient(circle at 50% 15%, rgba(132,164,169,.35), transparent 28%),
        linear-gradient(145deg, #29414b 0%, #172430 58%, #101923 100%);
    }

    #menu-card {
      width: min(920px, calc(100vw - 28px));
      max-height: calc(100vh - 28px);
      overflow-y: auto;
      padding: 24px;
      text-align: center;
    }

    #menu-card h1 {
      margin: 0;
      font-size: clamp(30px, 5vw, 48px);
      letter-spacing: .12em;
    }

    #menu-card p {
      margin: 8px 0 20px;
      color: #aab7c7;
    }

    #difficulty-control {
      display: grid;
      justify-items: center;
      gap: 6px;
      margin: 0 0 14px;
    }

    #difficulty-control label {
      color: #ffd166;
      font-size: 11px;
      font-weight: 900;
      letter-spacing: .1em;
      text-transform: uppercase;
    }

    #difficulty-select {
      min-width: 180px;
      padding: 8px 12px;
      border: 1px solid rgba(255,255,255,.2);
      border-radius: 8px;
      background: #222e3b;
      color: #fff;
      font: inherit;
      font-size: 13px;
    }

    #difficulty-description {
      margin: 0 !important;
      font-size: 11px;
    }

    #visual-controls {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 12px;
      margin: 0 0 16px;
    }

    .setting-control {
      display: grid;
      gap: 5px;
      min-width: 150px;
      text-align: left;
    }

    .setting-control label {
      color: #b6c7ca;
      font-size: 10px;
      font-weight: 850;
      letter-spacing: .08em;
      text-transform: uppercase;
    }

    .setting-control select {
      width: 100%;
      padding: 8px 10px;
      border: 1px solid rgba(255,255,255,.16);
      border-radius: 8px;
      background: #222e3b;
      color: #fff;
      font: inherit;
      font-size: 12px;
    }

    #best-records {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 8px;
      margin-top: 14px;
      color: #aab7c7;
      font-size: 10px;
    }

    .record-chip {
      padding: 6px 9px;
      border: 1px solid rgba(255,255,255,.09);
      border-radius: 999px;
      background: rgba(5,12,16,.22);
    }

    .record-chip strong { color: #ffd166; }

    #menu-navigation {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 8px;
      margin: 0 0 16px;
    }

    .menu-tab {
      padding: 9px 13px;
      border: 1px solid rgba(255,255,255,.14);
      border-radius: 9px;
      background: rgba(255,255,255,.06);
      color: #e7f0ec;
      cursor: pointer;
      font: inherit;
      font-size: 11px;
      font-weight: 850;
      transition: background .15s, border-color .15s, transform .15s;
    }

    .menu-tab:hover, .menu-tab:focus-visible {
      transform: translateY(-2px);
      border-color: #84e58b;
      outline: none;
    }

    .menu-tab.active {
      border-color: #84e58b;
      background: rgba(132,229,139,.16);
      color: #a6f2a0;
    }

    .menu-view {
      display: none;
      text-align: left;
    }

    .menu-view.active { display: block; }
    .menu-view h2 { margin: 0 0 6px; font-size: 18px; }
    .menu-view > p { margin: 0 0 14px; color: #aab7c7; font-size: 12px; }

    #campaign-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(190px, 1fr));
      gap: 9px;
      max-height: 48vh;
      overflow-y: auto;
      padding: 2px 4px 8px 0;
    }

    .mission-card {
      display: grid;
      gap: 5px;
      padding: 12px;
      border: 1px solid rgba(255,255,255,.13);
      border-radius: 11px;
      background: rgba(255,255,255,.045);
      color: #f3f6ff;
      text-align: left;
    }

    .mission-card.unlocked { cursor: pointer; }
    .mission-card.unlocked:hover, .mission-card.unlocked:focus-visible {
      border-color: #84e58b;
      background: rgba(132,229,139,.11);
      outline: none;
    }

    .mission-card.locked { opacity: .48; }
    .mission-title { font-size: 12px; font-weight: 900; }
    .mission-meta { color: #ffd166; font-size: 10px; font-weight: 800; }
    .mission-brief { color: #aab7c7; font-size: 10px; line-height: 1.45; }
    .mission-status { color: #84e58b; font-size: 9px; font-weight: 850; }

    #editor-layout {
      display: grid;
      grid-template-columns: minmax(220px, .7fr) minmax(280px, 1.3fr);
      gap: 14px;
      align-items: start;
    }

    #editor-form { display: grid; gap: 9px; }
    #editor-form label { display: grid; gap: 4px; color: #b6c7ca; font-size: 10px; font-weight: 800; }
    #editor-form input, #editor-form select {
      width: 100%;
      padding: 8px 9px;
      border: 1px solid rgba(255,255,255,.16);
      border-radius: 7px;
      background: #222e3b;
      color: #fff;
      font: inherit;
      font-size: 12px;
    }

    .editor-actions { display: flex; flex-wrap: wrap; gap: 6px; }
    .editor-actions button {
      flex: 1 1 100px;
      padding: 8px;
      border: 1px solid rgba(255,255,255,.14);
      border-radius: 7px;
      background: #344956;
      color: #fff;
      cursor: pointer;
      font: inherit;
      font-size: 10px;
      font-weight: 850;
    }

    .editor-actions button.primary { background: #84e58b; color: #17251a; }
    .editor-actions button.danger { background: #8f4a52; }

    #editor-preview {
      width: 100%;
      aspect-ratio: 3 / 2;
      border: 1px solid rgba(255,255,255,.13);
      border-radius: 10px;
      background: #31453e;
      cursor: crosshair;
      touch-action: manipulation;
    }

    #editor-preview .editor-route {
      fill: none;
      stroke: #b49b70;
      stroke-linecap: round;
      stroke-linejoin: round;
      stroke-width: 24;
    }

    #editor-preview .editor-point { fill: #ffd166; stroke: #172430; stroke-width: 4; }
    #editor-preview .editor-start { fill: #84e58b; }
    #editor-preview .editor-end { fill: #ff7681; }
    #editor-message { min-height: 18px; margin: 0; color: #ffd166; font-size: 10px; }

    #custom-maps {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 7px;
      margin-top: 12px;
    }

    .custom-map-chip {
      padding: 6px 9px;
      border: 1px solid rgba(132,229,139,.25);
      border-radius: 999px;
      background: rgba(132,229,139,.08);
      color: #d7e5d3;
      cursor: pointer;
      font: inherit;
      font-size: 10px;
      font-weight: 800;
    }

    #codex-controls {
      display: flex;
      flex-wrap: wrap;
      gap: 6px;
      margin-bottom: 10px;
    }

    .codex-filter {
      padding: 7px 10px;
      border: 1px solid rgba(255,255,255,.13);
      border-radius: 999px;
      background: rgba(255,255,255,.05);
      color: #e7f0ec;
      cursor: pointer;
      font: inherit;
      font-size: 10px;
      font-weight: 800;
    }

    .codex-filter.active { border-color: #84e58b; color: #a6f2a0; }

    #codex-entries {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
      gap: 8px;
      max-height: 50vh;
      overflow-y: auto;
      padding-right: 4px;
    }

    .codex-entry {
      padding: 11px;
      border: 1px solid rgba(255,255,255,.11);
      border-radius: 9px;
      background: rgba(255,255,255,.04);
    }

    .codex-entry h3 { margin: 0 0 5px; font-size: 12px; }
    .codex-entry p { margin: 0; color: #b3c0c5; font-size: 10px; line-height: 1.5; }
    .codex-entry small { display: block; margin-top: 6px; color: #ffd166; font-size: 9px; }

    #maps {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 12px;
    }

    .map-card {
      min-height: 220px;
      border: 1px solid rgba(255,255,255,.12);
      border-radius: 14px;
      background: rgba(255,255,255,.05);
      color: #fff;
      cursor: pointer;
      font: inherit;
      padding: 12px;
      transition: .15s;
    }

    .map-card:hover, .map-card:focus-visible {
      transform: translateY(-4px);
      border-color: #84e58b;
      background: rgba(132,229,139,.12);
      outline: none;
    }

    .map-preview {
      height: 110px;
      margin-bottom: 12px;
      border-radius: 10px;
      overflow: hidden;
    }

    .map-preview svg { width: 100%; height: 100%; display: block; }
    .map-name { display: block; font-size: 16px; font-weight: 900; }
    .map-description { display: block; margin-top: 5px; color: #aab7c7; font-size: 11px; }
    .map-lives { display: block; margin-top: 7px; color: #ffd166; font-size: 12px; font-weight: 900; }

    #game-ui { display: none; }

    #hud {
      position: fixed;
      z-index: 10;
      top: 12px;
      left: 12px;
      right: 12px;
      display: flex;
      justify-content: space-between;
      pointer-events: none;
    }

    #brand {
      padding: 11px 14px;
      font-weight: 900;
      letter-spacing: .12em;
      pointer-events: auto;
    }

    #brand small {
      display: block;
      margin-top: 3px;
      color: #aab7c7;
      font-size: 9px;
    }

    #stats {
      display: flex;
      gap: 16px;
      padding: 10px 14px;
      pointer-events: auto;
    }

    .stat-label { color: #aab7c7; font-size: 9px; text-transform: uppercase; }
    .stat-value { margin-top: 2px; font-size: 16px; font-weight: 900; }

    #wave-progress {
      position: fixed;
      z-index: 10;
      top: 72px;
      left: 50%;
      width: min(320px, 42vw);
      height: 7px;
      overflow: hidden;
      transform: translateX(-50%);
      border: 1px solid rgba(255,255,255,.22);
      border-radius: 99px;
      background: rgba(9,16,19,.65);
      pointer-events: none;
    }

    #wave-progress-fill {
      width: 0%;
      height: 100%;
      border-radius: inherit;
      background: linear-gradient(90deg, #84e58b, #ffd166);
      box-shadow: 0 0 12px rgba(132,229,139,.55);
      transition: width .18s ease-out;
    }

    #wave-progress-label {
      position: fixed;
      z-index: 10;
      top: 82px;
      left: 50%;
      transform: translateX(-50%);
      color: #d7e5d3;
      font-size: 9px;
      font-weight: 800;
      text-shadow: 0 2px 6px #101923;
      pointer-events: none;
    }

    #shop {
      position: fixed;
      z-index: 10;
      top: 88px;
      left: 12px;
      width: 205px;
      max-height: calc(100vh - 155px);
      overflow-y: auto;
      padding: 11px;
      scrollbar-width: thin;
    }

    #shop h2 {
      margin: 0 0 7px;
      font-size: 11px;
      letter-spacing: .12em;
    }

    .tower-button {
      width: 100%;
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 7px;
      margin-top: 5px;
      padding: 7px 8px;
      border: 1px solid rgba(255,255,255,.12);
      border-radius: 9px;
      background: rgba(255,255,255,.05);
      color: #fff;
      cursor: pointer;
      font: inherit;
      text-align: left;
    }

    .tower-button:hover, .tower-button.active {
      border-color: #84e58b;
      background: rgba(132,229,139,.13);
    }

    .tower-name { font-size: 11px; font-weight: 850; }
    .tower-desc { display: block; margin-top: 2px; color: #aab7c7; font-size: 9px; }
    .tower-cost { color: #ffd166; font-size: 10px; font-weight: 900; white-space: nowrap; }

    #tower-panel {
      display: none;
      margin-top: 10px;
      padding-top: 10px;
      border-top: 1px solid rgba(255,255,255,.12);
    }

    #tower-panel h2 { margin: 0 0 5px; font-size: 9px; }
    #selected-name { font-size: 12px; font-weight: 900; }
    #selected-info { margin: 5px 0 8px; color: #aab7c7; font-size: 9px; line-height: 1.5; }
    .actions { display: flex; flex-wrap: wrap; gap: 6px; }

    .actions button { min-width: 0; }

    button.action, #start-wave, #pause-game, #back-menu, #speed-game, #airstrike-game {
      border: 0;
      border-radius: 8px;
      padding: 9px 10px;
      font: inherit;
      font-size: 10px;
      font-weight: 900;
      cursor: pointer;
      touch-action: manipulation;
    }

    button.action { flex: 1; background: #84e58b; color: #17251a; }
    button.sell { background: #bd5964; color: #fff; }
    button.target-mode { background: #4a6372; color: #fff; }
    button:disabled { opacity: .5; cursor: not-allowed; }

    #bottom {
      position: fixed;
      z-index: 10;
      bottom: 12px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 8px;
    }

    #start-wave { background: #ffd166; color: #222812; }
    #pause-game { background: #84e58b; color: #17251a; }
    #back-menu { background: #222e3b; color: #fff; border: 1px solid rgba(255,255,255,.12); }
    #speed-game { background: #4a6372; color: #fff; }
    #airstrike-game { background: #ef8b58; color: #28170f; }

    #airstrike-game.ready {
      animation: ability-pulse 1.8s ease-in-out infinite;
    }

    @keyframes ability-pulse {
      50% { box-shadow: 0 0 18px rgba(239,139,88,.6); }
    }

    #toast {
      position: fixed;
      z-index: 20;
      top: 80px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 11px;
      font-weight: 900;
      letter-spacing: .08em;
      text-shadow: 0 2px 8px #172430;
      pointer-events: none;
      text-align: center;
    }

    #toast[data-kind="success"] { color: #a6f2a0; }
    #toast[data-kind="warning"] { color: #ff9a91; }

    #combat-feed {
      position: fixed;
      z-index: 9;
      right: 12px;
      bottom: 62px;
      display: grid;
      justify-items: end;
      gap: 4px;
      pointer-events: none;
    }

    .feed-message {
      padding: 5px 8px;
      border: 1px solid rgba(255,255,255,.1);
      border-radius: 7px;
      background: rgba(14,22,27,.76);
      color: #d8e2df;
      font-size: 9px;
      font-weight: 750;
      text-shadow: 0 1px 3px #101923;
      animation: feed-enter .2s ease-out, feed-leave .3s ease-in 3.2s forwards;
    }

    .feed-message.boss { color: #ff9a91; }
    .feed-message.reward { color: #ffd166; }

    @keyframes feed-enter {
      from { opacity: 0; transform: translateX(10px); }
      to { opacity: 1; transform: translateX(0); }
    }

    @keyframes feed-leave {
      to { opacity: 0; transform: translateY(-4px); }
    }

    #map-vignette {
      position: fixed;
      z-index: 2;
      inset: 0;
      background:
        radial-gradient(ellipse at center, transparent 53%, rgba(10,17,20,.16) 100%),
        linear-gradient(180deg, rgba(20,33,39,.08), transparent 30%, transparent 76%, rgba(13,19,21,.18));
      pointer-events: none;
    }

    #game-ui[data-weather="rain"] #map-vignette {
      background:
        linear-gradient(120deg, rgba(106,151,177,.08), rgba(20,35,48,.18)),
        radial-gradient(ellipse at center, transparent 48%, rgba(12,21,29,.28) 100%);
    }

    #game-ui[data-weather="snow"] #map-vignette {
      background:
        linear-gradient(180deg, rgba(214,231,236,.12), transparent 40%),
        radial-gradient(ellipse at center, transparent 52%, rgba(194,214,220,.2) 100%);
    }

    #game-ui[data-weather="mist"] #map-vignette {
      background:
        linear-gradient(180deg, rgba(196,212,210,.16), rgba(190,207,204,.05) 55%, rgba(196,212,210,.2)),
        radial-gradient(ellipse at center, transparent 35%, rgba(189,205,202,.3) 100%);
    }

    #game-ui[data-quality="low"] #map-vignette {
      background: linear-gradient(180deg, rgba(20,33,39,.04), transparent 60%, rgba(13,19,21,.1));
    }

    #game-ui[data-airstrike="targeting"] canvas { cursor: crosshair; }

    #screen-flash {
      position: fixed;
      z-index: 8;
      inset: 0;
      opacity: 0;
      background: #ffe3a2;
      pointer-events: none;
    }

    #screen-flash.active {
      animation: screen-flash .28s ease-out;
    }

    @keyframes screen-flash {
      0% { opacity: .38; }
      100% { opacity: 0; }
    }

    #overlay {
      position: fixed;
      z-index: 30;
      inset: 0;
      display: none;
      place-items: center;
      background: rgba(9,15,20,.76);
    }

    #overlay-card {
      width: min(360px, calc(100vw - 28px));
      padding: 24px;
      text-align: center;
    }

    #overlay-card h2 { margin: 0 0 8px; color: #ff7681; }
    #overlay-card p { color: #aab7c7; }

    #pause-indicator {
      position: fixed;
      z-index: 9;
      inset: 0;
      display: none;
      place-items: center;
      background: rgba(8,14,20,.48);
      color: #fff;
      font-size: clamp(24px, 4vw, 36px);
      font-weight: 900;
      letter-spacing: .12em;
      pointer-events: none;
      text-align: center;
      text-shadow: 0 3px 12px #172430;
    }

    #pause-indicator small {
      display: block;
      margin-top: 10px;
      color: #c5d0dc;
      font-size: 13px;
      letter-spacing: normal;
    }

    @media (max-width: 760px) {
      #maps { grid-template-columns: repeat(2, 1fr); }
      .map-card { min-height: 180px; }
      .map-preview { height: 80px; }
      #shop { top: 78px; left: 7px; width: 166px; padding: 8px; }
      #hud { top: 7px; left: 7px; right: 7px; }
      #stats { gap: 8px; }
      #stats { padding: 8px; }
      #wave-progress { top: 71px; }
      #wave-progress-label { top: 81px; }
      #bottom { width: calc(100vw - 12px); bottom: 6px; gap: 5px; }
      #bottom button { padding: 8px; font-size: 9px; }
      #combat-feed { right: 7px; bottom: 94px; }
      #brand { padding: 8px; font-size: 12px; }
      .stat-value { font-size: 13px; }
    }

    @media (max-width: 430px) {
      #menu-card { padding: 14px; }
      #maps { gap: 7px; }
      .map-card { min-height: 154px; padding: 8px; }
      .map-preview { height: 64px; margin-bottom: 7px; }
      .map-description { font-size: 9px; }
      #shop { top: 96px; width: 148px; }
      #stats { gap: 5px; }
      .stat-label { font-size: 8px; }
      .stat-value { font-size: 12px; }
      #wave-progress { top: 81px; width: 45vw; }
      #wave-progress-label { top: 91px; }
      #bottom { left: 7px; transform: none; justify-content: flex-start; }
      #bottom button { padding: 7px 8px; }
      #screen-flash { z-index: 8; }
      #editor-layout { grid-template-columns: 1fr; }
      #campaign-grid { grid-template-columns: 1fr; max-height: 42vh; }
      #codex-entries { grid-template-columns: 1fr; max-height: 42vh; }
    }

    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after {
        animation-duration: .01ms !important;
        animation-iteration-count: 1 !important;
        scroll-behavior: auto !important;
        transition-duration: .01ms !important;
      }
    }
  </style>
</head>
<body>
  <section id="menu">
    <div class="panel" id="menu-card">
      <h1>BLOCKFRONT</h1>
      <p>Kies een map om te beginnen.</p>

      <nav id="menu-navigation" aria-label="Spelmenu">
        <button class="menu-tab active" type="button" data-view="maps-view">Vrij spelen</button>
        <button class="menu-tab" type="button" data-view="campaign-view">Campagne</button>
        <button class="menu-tab" type="button" data-view="editor-view">Kaarteditor</button>
        <button class="menu-tab" type="button" data-view="codex-view">Veldhandboek</button>
      </nav>

      <div id="difficulty-control">
        <label for="difficulty-select">Moeilijkheid</label>
        <select id="difficulty-select">
          <option value="easy">Makkelijk</option>
          <option value="normal" selected>Normaal</option>
          <option value="hard">Moeilijk</option>
        </select>
        <p id="difficulty-description">Standaardbalans voor een eerlijke uitdaging.</p>
      </div>

      <div id="visual-controls">
        <div class="setting-control">
          <label for="weather-select">Weerbeeld</label>
          <select id="weather-select">
            <option value="clear">Helder</option>
            <option value="rain">Regen</option>
            <option value="snow">Sneeuw</option>
            <option value="mist">Ochtendmist</option>
          </select>
        </div>
        <div class="setting-control">
          <label for="quality-select">Terreindetail</label>
          <select id="quality-select">
            <option value="high">Hoog</option>
            <option value="medium">Gebalanceerd</option>
            <option value="low">Lichtgewicht</option>
          </select>
        </div>
      </div>

      <section class="menu-view active" id="maps-view">
      <div id="maps">
        <button class="map-card" type="button" data-map="meadow">
          <div class="map-preview">
            <svg viewBox="0 0 200 130">
              <rect width="200" height="130" fill="#527e4c"/>
              <polyline points="10,100 45,100 45,45 100,45 100,75 155,75 155,25 190,25"
                fill="none" stroke="#b49b70" stroke-width="18"/>
            </svg>
          </div>
          <span class="map-name">Meadow</span>
          <span class="map-description">Bochtige beginnerskaart.</span>
          <span class="map-lives">20 levens</span>
        </button>

        <button class="map-card" type="button" data-map="crossroads">
          <div class="map-preview">
            <svg viewBox="0 0 200 130">
              <rect width="200" height="130" fill="#557b55"/>
              <polyline points="10,65 55,65 55,20 105,20 105,110 155,110 155,55 190,55"
                fill="none" stroke="#9e9273" stroke-width="18"/>
            </svg>
          </div>
          <span class="map-name">Crossroads</span>
          <span class="map-description">Lange route, weinig bouwruimte.</span>
          <span class="map-lives">15 levens</span>
        </button>

        <button class="map-card" type="button" data-map="swamp">
          <div class="map-preview">
            <svg viewBox="0 0 200 130">
              <rect width="200" height="130" fill="#49685a"/>
              <polyline points="10,110 45,110 45,70 90,70 90,30 130,30 130,85 190,85"
                fill="none" stroke="#8d8468" stroke-width="18"/>
            </svg>
          </div>
          <span class="map-name">Swamp</span>
          <span class="map-description">Korte en gevaarlijke route.</span>
          <span class="map-lives">12 levens</span>
        </button>

        <button class="map-card" type="button" data-map="desert">
          <div class="map-preview">
            <svg viewBox="0 0 200 130">
              <rect width="200" height="130" fill="#b78d58"/>
              <polyline points="10,25 55,25 55,75 105,75 105,35 150,35 150,105 190,105"
                fill="none" stroke="#d8c58a" stroke-width="18"/>
            </svg>
          </div>
          <span class="map-name">Desert</span>
          <span class="map-description">Kronkelige route door het zand.</span>
          <span class="map-lives">18 levens</span>
        </button>

        <button class="map-card" type="button" data-map="highlands">
          <div class="map-preview">
            <svg viewBox="0 0 200 130">
              <rect width="200" height="130" fill="#65755d"/>
              <path d="M0 0h60l25 24-20 20H0zm120 0h80v42h-44l-22-20zM0 88h38l22 18-14 24H0zm141 0h59v42h-76l18-20z"
                fill="#7e8d70" opacity=".8"/>
              <polyline points="0,65 45,65 45,25 105,25 105,100 160,100 160,52 200,52"
                fill="none" stroke="#b09b78" stroke-width="16"/>
            </svg>
          </div>
          <span class="map-name">Highlands</span>
          <span class="map-description">Rotsige passen en steile plateaus.</span>
          <span class="map-lives">16 levens</span>
        </button>
      </div>

      <div id="custom-maps"></div>
      <div id="best-records" aria-live="polite"></div>
      </section>

      <section class="menu-view" id="campaign-view">
        <h2>De Grensoorlog</h2>
        <p>Voltooi operaties op vijf fronten. Je voortgang wordt lokaal bewaard.</p>
        <div id="campaign-summary"></div>
        <div id="campaign-grid"></div>
      </section>

      <section class="menu-view" id="editor-view">
        <h2>Kaartwerkplaats</h2>
        <p>Klik in het veld om bochten toe te voegen. De route loopt van groen naar rood.</p>
        <div id="editor-layout">
          <div id="editor-form">
            <label for="editor-name">Kaartnaam<input id="editor-name" maxlength="28" value="Nieuwe route"></label>
            <label for="editor-biome">Landschap
              <select id="editor-biome">
                <option value="meadow">Weide</option>
                <option value="forest">Bos</option>
                <option value="swamp">Moeras</option>
                <option value="desert">Woestijn</option>
                <option value="highlands">Hoogland</option>
              </select>
            </label>
            <label for="editor-lives">Basislevens<input id="editor-lives" type="number" min="5" max="40" value="18"></label>
            <div class="editor-actions">
              <button type="button" id="editor-undo">Bocht terug</button>
              <button type="button" id="editor-reset" class="danger">Route wissen</button>
              <button type="button" id="editor-save" class="primary">Kaart bewaren</button>
            </div>
            <p id="editor-message" aria-live="polite"></p>
          </div>
          <svg id="editor-preview" viewBox="0 0 960 640" role="img" aria-label="Klik om de route te tekenen">
            <rect width="960" height="640" fill="#527e4c"/>
            <polyline class="editor-route" id="editor-route"></polyline>
            <g id="editor-points"></g>
            <circle class="editor-point editor-start" cx="0" cy="320" r="15"></circle>
            <circle class="editor-point editor-end" cx="960" cy="320" r="15"></circle>
          </svg>
        </div>
      </section>

      <section class="menu-view" id="codex-view">
        <h2>Veldhandboek</h2>
        <p>Bekijk de eenheden, kaarten en tactieken die je verdediging sterker maken.</p>
        <div id="codex-controls">
          <button class="codex-filter active" type="button" data-codex="all">Alles</button>
          <button class="codex-filter" type="button" data-codex="tower">Torens</button>
          <button class="codex-filter" type="button" data-codex="enemy">Vijanden</button>
          <button class="codex-filter" type="button" data-codex="map">Kaarten</button>
          <button class="codex-filter" type="button" data-codex="tactic">Tactiek</button>
        </div>
        <div id="codex-entries"></div>
      </section>
    </div>
  </section>

  <div id="game-ui">
    <canvas id="game"></canvas>

    <header id="hud">
      <div class="panel" id="brand">
        BLOCKFRONT
        <small id="map-name">MEADOW</small>
        <small id="difficulty-name">NORMAAL</small>
      </div>
      <div class="panel" id="stats">
        <div><div class="stat-label">Cash</div><div class="stat-value" id="cash">$700</div></div>
        <div><div class="stat-label">Base</div><div class="stat-value" id="lives">20</div></div>
        <div><div class="stat-label">Wave</div><div class="stat-value" id="wave">1</div></div>
      </div>
    </header>

    <div id="wave-progress" aria-hidden="true"><div id="wave-progress-fill"></div></div>
    <div id="wave-progress-label" aria-live="polite">Klaar voor de volgende aanval</div>
    <div id="map-vignette" aria-hidden="true"></div>
    <div id="screen-flash" aria-hidden="true"></div>

    <aside class="panel" id="shop">
      <h2>TOWER SHOP</h2>

      <button class="tower-button active" type="button" data-type="scout">
        <span><span class="tower-name">Scout</span><span class="tower-desc">Snel · allround</span></span>
        <span class="tower-cost">$250</span>
      </button>
      <button class="tower-button" type="button" data-type="sniper">
        <span><span class="tower-name">Sniper</span><span class="tower-desc">Lange afstand</span></span>
        <span class="tower-cost">$400</span>
      </button>
      <button class="tower-button" type="button" data-type="cannon">
        <span><span class="tower-name">Cannon</span><span class="tower-desc">Gebiedsschade</span></span>
        <span class="tower-cost">$500</span>
      </button>
      <button class="tower-button" type="button" data-type="frost">
        <span><span class="tower-name">Frost</span><span class="tower-desc">Vertraagt vijanden</span></span>
        <span class="tower-cost">$350</span>
      </button>
      <button class="tower-button" type="button" data-type="laser">
        <span><span class="tower-name">Laser</span><span class="tower-desc">Directe straal</span></span>
        <span class="tower-cost">$650</span>
      </button>
      <button class="tower-button" type="button" data-type="tesla">
        <span><span class="tower-name">Tesla</span><span class="tower-desc">Kettingaanval</span></span>
        <span class="tower-cost">$550</span>
      </button>
      <button class="tower-button" type="button" data-type="mortar">
        <span><span class="tower-name">Mortar</span><span class="tower-desc">Grote explosie</span></span>
        <span class="tower-cost">$600</span>
      </button>
      <button class="tower-button" type="button" data-type="rapid">
        <span><span class="tower-name">Rapid</span><span class="tower-desc">Heel snel vuren</span></span>
        <span class="tower-cost">$450</span>
      </button>

      <div id="tower-panel">
        <h2>GESELECTEERDE TOREN</h2>
        <div id="selected-name"></div>
        <div id="selected-info"></div>
        <div class="actions">
          <button class="action" type="button" id="upgrade">Upgrade</button>
          <button class="action target-mode" type="button" id="target-mode">Doel: Voorste</button>
          <button class="action sell" type="button" id="sell">Verkoop</button>
        </div>
      </div>
    </aside>

    <div id="toast"></div>
    <div id="combat-feed" aria-live="polite"></div>
    <div id="pause-indicator">GEPAUZEERD<small>P of Hervat · A luchtsteun · 1–3 snelheid · Esc menu</small></div>

    <div id="bottom">
      <button id="start-wave" type="button">Start wave 1</button>
      <button id="speed-game" type="button" aria-label="Spelsnelheid wijzigen">Snelheid 1×</button>
      <button id="airstrike-game" type="button" aria-label="Luchtsteun inzetten">Luchtsteun $300</button>
      <button id="pause-game" type="button" aria-pressed="false">Pauze</button>
      <button id="back-menu" type="button">Hoofdmenu</button>
    </div>

    <div id="overlay">
      <div class="panel" id="overlay-card">
        <h2 id="overlay-title">Base vernietigd!</h2>
        <p id="final-score"></p>
        <div class="actions">
          <button class="action" type="button" id="next-mission" style="display:none">Volgende operatie</button>
          <button class="action" type="button" id="restart-map">Opnieuw</button>
          <button class="action sell" type="button" id="overlay-menu">Hoofdmenu</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    (() => {
      const canvas = document.getElementById('game');
      const gl = canvas.getContext('webgl', {antialias:true,alpha:false});
      if (!gl) throw new Error('WebGL is niet beschikbaar in deze browser.');
      const W = 960;
      const H = 640;
      const CELL = 40;
      const WORLD_WIDTH = 30;
      const WORLD_DEPTH = 20;
      const shaderProgram = createShaderProgram();
      const staticBuffer = gl.createBuffer();
      const dynamicBuffer = gl.createBuffer();
      let staticVertexCount = 0;
      let viewProjectionMatrix = null;
      const shaderLocations = {
        position:gl.getAttribLocation(shaderProgram,'a_position'),
        normal:gl.getAttribLocation(shaderProgram,'a_normal'),
        color:gl.getAttribLocation(shaderProgram,'a_color'),
        matrix:gl.getUniformLocation(shaderProgram,'u_viewProjection'),
        light:gl.getUniformLocation(shaderProgram,'u_lightDirection'),
        camera:gl.getUniformLocation(shaderProgram,'u_cameraPosition'),
        fog:gl.getUniformLocation(shaderProgram,'u_fogColor'),
        fogAmount:gl.getUniformLocation(shaderProgram,'u_fogAmount')
      };
      gl.enable(gl.DEPTH_TEST);
      gl.depthFunc(gl.LEQUAL);
      gl.disable(gl.CULL_FACE);

      const maps = {
        meadow: {
          name: 'MEADOW', biome: 'meadow', lives: 20, ground: '#527e4c', road: '#b49b70',
          path: [{x:0,y:170},{x:180,y:170},{x:180,y:370},{x:410,y:370},{x:410,y:200},{x:650,y:200},{x:650,y:450},{x:850,y:450},{x:960,y:520}]
        },
        crossroads: {
          name: 'CROSSROADS', biome: 'forest', lives: 15, ground: '#557b55', road: '#9e9273',
          path: [{x:0,y:300},{x:130,y:300},{x:130,y:90},{x:330,y:90},{x:330,y:490},{x:560,y:490},{x:560,y:250},{x:760,y:250},{x:760,y:540},{x:960,y:540}]
        },
        swamp: {
          name: 'SWAMP', biome: 'swamp', lives: 12, ground: '#49685a', road: '#8d8468',
          path: [{x:0,y:500},{x:130,y:500},{x:130,y:330},{x:300,y:330},{x:300,y:130},{x:520,y:130},{x:520,y:400},{x:720,y:400},{x:720,y:210},{x:960,y:210}]
        },
        desert: {
          name: 'DESERT', biome: 'desert', lives: 18, ground: '#b78d58', road: '#d8c58a',
          path: [{x:0,y:110},{x:180,y:110},{x:180,y:300},{x:390,y:300},{x:390,y:150},{x:610,y:150},{x:610,y:470},{x:820,y:470},{x:820,y:270},{x:960,y:270}]
        },
        highlands: {
          name: 'HIGHLANDS', biome: 'highlands', lives: 16, ground: '#64725e', road: '#b09b78',
          path: [{x:0,y:320},{x:170,y:320},{x:170,y:150},{x:390,y:150},{x:390,y:500},{x:610,y:500},{x:610,y:250},{x:810,y:250},{x:810,y:430},{x:960,y:430}]
        }
      };

      const difficultySettings = {
        easy: {
          name:'Makkelijk',
          description:'Meer startgeld en levens; vijanden zijn zwakker en langzamer.',
          startingCash:900,livesMultiplier:1.5,enemyHealthMultiplier:.8,enemySpeedMultiplier:.9
        },
        normal: {
          name:'Normaal',
          description:'Standaardbalans voor een eerlijke uitdaging.',
          startingCash:700,livesMultiplier:1,enemyHealthMultiplier:1,enemySpeedMultiplier:1
        },
        hard: {
          name:'Moeilijk',
          description:'Minder startgeld en levens; vijanden zijn sterker en sneller.',
          startingCash:550,livesMultiplier:.7,enemyHealthMultiplier:1.3,enemySpeedMultiplier:1.12
        }
      };

      const types = {
        scout:  {name:'Scout',  cost:250, damage:12, range:125, rate:.48, speed:500, color:'#67b7ff', splash:0,   slow:0,   chain:0, radius:4},
        sniper: {name:'Sniper', cost:400, damage:42, range:215, rate:1.45,speed:720, color:'#ffd166', splash:0,   slow:0,   chain:0, radius:4},
        cannon: {name:'Cannon', cost:500, damage:28, range:145, rate:1.2, speed:300, color:'#ff795f', splash:48,  slow:0,   chain:0, radius:7},
        frost:  {name:'Frost',  cost:350, damage:8,  range:150, rate:.8,  speed:430, color:'#8beaff', splash:0,   slow:1.8, chain:0, radius:4},
        laser:  {name:'Laser',  cost:650, damage:17, range:165, rate:.18, speed:0,   color:'#d895ff', splash:0,   slow:0,   chain:0, radius:4},
        tesla:  {name:'Tesla',  cost:550, damage:20, range:145, rate:1,   speed:480, color:'#70f0d0', splash:0,   slow:0,   chain:2, radius:4},
        mortar: {name:'Mortar', cost:600, damage:48, range:245, rate:2.1, speed:250, color:'#ffae55', splash:72,  slow:0,   chain:0, radius:8},
        rapid:  {name:'Rapid',  cost:450, damage:6,  range:115, rate:.13, speed:570, color:'#f06f9e', splash:0,   slow:0,   chain:0, radius:3}
      };

      let currentMap = null;
      let path = [];
      let pathLength = 0;
      let enemies = [];
      let towers = [];
      let shots = [];
      let effects = [];
      let beams = [];
      let weatherParticles = [];
      let cash = 700;
      let lives = 20;
      let wave = 1;
      let waveActive = false;
      let paused = false;
      let gameOver = false;
      let difficulty = 'normal';
      let buildType = 'scout';
      let selectedTower = null;
      let spawned = 0;
      let spawnTotal = 0;
      let spawnTimer = 0;
      let lastTime = 0;
      let visualTime = 0;
      let weather = 'clear';
      let quality = 'high';
      let gameSpeed = 1;
      let airstrikeCooldown = 0;
      let airstrikeTargeting = false;
      let waveStartLives = 0;
      let mouse = {x:0,y:0,inside:false};
      const targetingModes = ['leading','strongest','nearest'];
      const targetingLabels = {
        leading:'Voorste',
        strongest:'Sterkste',
        nearest:'Dichtstbij'
      };
      const recordStorageKey = 'blockfront-records-v1';
      const campaignStorageKey = 'blockfront-campaign-v1';
      const customMapStorageKey = 'blockfront-custom-maps-v1';
      const missionBlueprints = [
        {
          chapter:1,
          stage:1,
          title:"De groene grens · Stille dagen",
          mapId:"meadow",
          weather:"clear",
          waveGoal:3,
          startingCash:930,
          lives:18,
          enemyHealthMultiplier:0.96,
          enemySpeedMultiplier:0.94,
          extraEnemies:0,
          enemyRewardMultiplier:1.15,
          spawnIntervalMultiplier:1.18,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.0,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 1 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:2,
          title:"De groene grens · Eerste contact",
          mapId:"crossroads",
          weather:"rain",
          waveGoal:4,
          startingCash:910,
          lives:17,
          enemyHealthMultiplier:0.969,
          enemySpeedMultiplier:0.94,
          extraEnemies:0,
          enemyRewardMultiplier:1.15,
          spawnIntervalMultiplier:1.16,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.04,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 2 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:3,
          title:"De groene grens · Verloren zendmast",
          mapId:"swamp",
          weather:"snow",
          waveGoal:5,
          startingCash:890,
          lives:16,
          enemyHealthMultiplier:0.978,
          enemySpeedMultiplier:0.94,
          extraEnemies:0,
          enemyRewardMultiplier:1.16,
          spawnIntervalMultiplier:1.14,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.09,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 3 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:4,
          title:"De groene grens · Nachtelijke wacht",
          mapId:"desert",
          weather:"mist",
          waveGoal:6,
          startingCash:870,
          lives:18,
          enemyHealthMultiplier:0.987,
          enemySpeedMultiplier:0.94,
          extraEnemies:1,
          enemyRewardMultiplier:1.16,
          spawnIntervalMultiplier:1.13,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.14,
          objective:"Versla de baas en houd de route open.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 4 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:5,
          title:"De groene grens · Gebroken brug",
          mapId:"highlands",
          weather:"clear",
          waveGoal:7,
          startingCash:850,
          lives:17,
          enemyHealthMultiplier:0.996,
          enemySpeedMultiplier:0.94,
          extraEnemies:1,
          enemyRewardMultiplier:1.17,
          spawnIntervalMultiplier:1.11,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.18,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 5 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:6,
          title:"De groene grens · Rode rook",
          mapId:"meadow",
          weather:"rain",
          waveGoal:3,
          startingCash:830,
          lives:16,
          enemyHealthMultiplier:1.005,
          enemySpeedMultiplier:0.94,
          extraEnemies:1,
          enemyRewardMultiplier:1.17,
          spawnIntervalMultiplier:1.09,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.23,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 6 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:7,
          title:"De groene grens · Lange mars",
          mapId:"crossroads",
          weather:"snow",
          waveGoal:4,
          startingCash:810,
          lives:18,
          enemyHealthMultiplier:1.014,
          enemySpeedMultiplier:0.94,
          extraEnemies:2,
          enemyRewardMultiplier:1.17,
          spawnIntervalMultiplier:1.07,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.27,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 7 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:8,
          title:"De groene grens · Schaduwlinie",
          mapId:"swamp",
          weather:"mist",
          waveGoal:5,
          startingCash:790,
          lives:17,
          enemyHealthMultiplier:1.023,
          enemySpeedMultiplier:0.94,
          extraEnemies:2,
          enemyRewardMultiplier:1.18,
          spawnIntervalMultiplier:1.05,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.31,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 8 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:9,
          title:"De groene grens · Stormsein",
          mapId:"desert",
          weather:"clear",
          waveGoal:6,
          startingCash:770,
          lives:16,
          enemyHealthMultiplier:1.032,
          enemySpeedMultiplier:0.94,
          extraEnemies:2,
          enemyRewardMultiplier:1.18,
          spawnIntervalMultiplier:1.04,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.36,
          objective:"Versla de baas en houd de route open.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 9 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:1,
          stage:10,
          title:"De groene grens · Laatste licht",
          mapId:"highlands",
          weather:"rain",
          waveGoal:7,
          startingCash:750,
          lives:18,
          enemyHealthMultiplier:1.041,
          enemySpeedMultiplier:0.94,
          extraEnemies:3,
          enemyRewardMultiplier:1.19,
          spawnIntervalMultiplier:1.02,
          airstrikeDamageMultiplier:1.25,
          perfectBonusMultiplier:1.41,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 10 in De groene grens: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:1,
          title:"De oude kruising · Stille dagen",
          mapId:"crossroads",
          weather:"snow",
          waveGoal:3,
          startingCash:904,
          lives:17,
          enemyHealthMultiplier:0.992,
          enemySpeedMultiplier:0.953,
          extraEnemies:0,
          enemyRewardMultiplier:1.12,
          spawnIntervalMultiplier:1.15,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.0,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 1 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:2,
          title:"De oude kruising · Eerste contact",
          mapId:"swamp",
          weather:"mist",
          waveGoal:4,
          startingCash:884,
          lives:16,
          enemyHealthMultiplier:1.001,
          enemySpeedMultiplier:0.953,
          extraEnemies:0,
          enemyRewardMultiplier:1.13,
          spawnIntervalMultiplier:1.13,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.04,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 2 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:3,
          title:"De oude kruising · Verloren zendmast",
          mapId:"desert",
          weather:"clear",
          waveGoal:5,
          startingCash:864,
          lives:15,
          enemyHealthMultiplier:1.01,
          enemySpeedMultiplier:0.953,
          extraEnemies:0,
          enemyRewardMultiplier:1.13,
          spawnIntervalMultiplier:1.11,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.09,
          objective:"Versla de baas en houd de route open.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 3 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:4,
          title:"De oude kruising · Nachtelijke wacht",
          mapId:"highlands",
          weather:"rain",
          waveGoal:6,
          startingCash:844,
          lives:17,
          enemyHealthMultiplier:1.019,
          enemySpeedMultiplier:0.953,
          extraEnemies:1,
          enemyRewardMultiplier:1.14,
          spawnIntervalMultiplier:1.09,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.14,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 4 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:5,
          title:"De oude kruising · Gebroken brug",
          mapId:"meadow",
          weather:"snow",
          waveGoal:7,
          startingCash:824,
          lives:16,
          enemyHealthMultiplier:1.028,
          enemySpeedMultiplier:0.953,
          extraEnemies:1,
          enemyRewardMultiplier:1.14,
          spawnIntervalMultiplier:1.07,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.18,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 5 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:6,
          title:"De oude kruising · Rode rook",
          mapId:"crossroads",
          weather:"mist",
          waveGoal:3,
          startingCash:804,
          lives:15,
          enemyHealthMultiplier:1.037,
          enemySpeedMultiplier:0.953,
          extraEnemies:1,
          enemyRewardMultiplier:1.15,
          spawnIntervalMultiplier:1.05,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.23,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 6 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:7,
          title:"De oude kruising · Lange mars",
          mapId:"swamp",
          weather:"clear",
          waveGoal:4,
          startingCash:784,
          lives:17,
          enemyHealthMultiplier:1.046,
          enemySpeedMultiplier:0.953,
          extraEnemies:2,
          enemyRewardMultiplier:1.15,
          spawnIntervalMultiplier:1.04,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.27,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 7 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:8,
          title:"De oude kruising · Schaduwlinie",
          mapId:"desert",
          weather:"rain",
          waveGoal:5,
          startingCash:764,
          lives:16,
          enemyHealthMultiplier:1.055,
          enemySpeedMultiplier:0.953,
          extraEnemies:2,
          enemyRewardMultiplier:1.15,
          spawnIntervalMultiplier:1.02,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.31,
          objective:"Versla de baas en houd de route open.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 8 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:9,
          title:"De oude kruising · Stormsein",
          mapId:"highlands",
          weather:"snow",
          waveGoal:6,
          startingCash:744,
          lives:15,
          enemyHealthMultiplier:1.064,
          enemySpeedMultiplier:0.953,
          extraEnemies:2,
          enemyRewardMultiplier:1.16,
          spawnIntervalMultiplier:1.0,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.36,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 9 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:2,
          stage:10,
          title:"De oude kruising · Laatste licht",
          mapId:"meadow",
          weather:"mist",
          waveGoal:7,
          startingCash:724,
          lives:17,
          enemyHealthMultiplier:1.073,
          enemySpeedMultiplier:0.953,
          extraEnemies:3,
          enemyRewardMultiplier:1.16,
          spawnIntervalMultiplier:0.98,
          airstrikeDamageMultiplier:1.22,
          perfectBonusMultiplier:1.41,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 10 in De oude kruising: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:1,
          title:"Het stille moeras · Stille dagen",
          mapId:"swamp",
          weather:"clear",
          waveGoal:3,
          startingCash:878,
          lives:16,
          enemyHealthMultiplier:1.024,
          enemySpeedMultiplier:0.966,
          extraEnemies:1,
          enemyRewardMultiplier:1.1,
          spawnIntervalMultiplier:1.11,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.0,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 1 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:2,
          title:"Het stille moeras · Eerste contact",
          mapId:"desert",
          weather:"rain",
          waveGoal:4,
          startingCash:858,
          lives:15,
          enemyHealthMultiplier:1.033,
          enemySpeedMultiplier:0.966,
          extraEnemies:1,
          enemyRewardMultiplier:1.1,
          spawnIntervalMultiplier:1.09,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.04,
          objective:"Versla de baas en houd de route open.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 2 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:3,
          title:"Het stille moeras · Verloren zendmast",
          mapId:"highlands",
          weather:"snow",
          waveGoal:5,
          startingCash:838,
          lives:14,
          enemyHealthMultiplier:1.042,
          enemySpeedMultiplier:0.966,
          extraEnemies:1,
          enemyRewardMultiplier:1.11,
          spawnIntervalMultiplier:1.07,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.09,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 3 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:4,
          title:"Het stille moeras · Nachtelijke wacht",
          mapId:"meadow",
          weather:"mist",
          waveGoal:6,
          startingCash:818,
          lives:16,
          enemyHealthMultiplier:1.051,
          enemySpeedMultiplier:0.966,
          extraEnemies:2,
          enemyRewardMultiplier:1.11,
          spawnIntervalMultiplier:1.06,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.14,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 4 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:5,
          title:"Het stille moeras · Gebroken brug",
          mapId:"crossroads",
          weather:"clear",
          waveGoal:7,
          startingCash:798,
          lives:15,
          enemyHealthMultiplier:1.06,
          enemySpeedMultiplier:0.966,
          extraEnemies:2,
          enemyRewardMultiplier:1.12,
          spawnIntervalMultiplier:1.04,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.18,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 5 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:6,
          title:"Het stille moeras · Rode rook",
          mapId:"swamp",
          weather:"rain",
          waveGoal:3,
          startingCash:778,
          lives:14,
          enemyHealthMultiplier:1.069,
          enemySpeedMultiplier:0.966,
          extraEnemies:2,
          enemyRewardMultiplier:1.12,
          spawnIntervalMultiplier:1.02,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.23,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 6 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:7,
          title:"Het stille moeras · Lange mars",
          mapId:"desert",
          weather:"snow",
          waveGoal:4,
          startingCash:758,
          lives:16,
          enemyHealthMultiplier:1.078,
          enemySpeedMultiplier:0.966,
          extraEnemies:3,
          enemyRewardMultiplier:1.12,
          spawnIntervalMultiplier:1.0,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.27,
          objective:"Versla de baas en houd de route open.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 7 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:8,
          title:"Het stille moeras · Schaduwlinie",
          mapId:"highlands",
          weather:"mist",
          waveGoal:5,
          startingCash:738,
          lives:15,
          enemyHealthMultiplier:1.087,
          enemySpeedMultiplier:0.966,
          extraEnemies:3,
          enemyRewardMultiplier:1.13,
          spawnIntervalMultiplier:0.98,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.31,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 8 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:9,
          title:"Het stille moeras · Stormsein",
          mapId:"meadow",
          weather:"clear",
          waveGoal:6,
          startingCash:718,
          lives:14,
          enemyHealthMultiplier:1.096,
          enemySpeedMultiplier:0.966,
          extraEnemies:3,
          enemyRewardMultiplier:1.13,
          spawnIntervalMultiplier:0.97,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.36,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 9 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:3,
          stage:10,
          title:"Het stille moeras · Laatste licht",
          mapId:"crossroads",
          weather:"rain",
          waveGoal:7,
          startingCash:698,
          lives:16,
          enemyHealthMultiplier:1.105,
          enemySpeedMultiplier:0.966,
          extraEnemies:4,
          enemyRewardMultiplier:1.14,
          spawnIntervalMultiplier:0.95,
          airstrikeDamageMultiplier:1.18,
          perfectBonusMultiplier:1.41,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 10 in Het stille moeras: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:1,
          title:"De koperen woestijn · Stille dagen",
          mapId:"desert",
          weather:"snow",
          waveGoal:4,
          startingCash:852,
          lives:15,
          enemyHealthMultiplier:1.056,
          enemySpeedMultiplier:0.979,
          extraEnemies:1,
          enemyRewardMultiplier:1.07,
          spawnIntervalMultiplier:1.07,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.0,
          objective:"Versla de baas en houd de route open.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 1 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:2,
          title:"De koperen woestijn · Eerste contact",
          mapId:"highlands",
          weather:"mist",
          waveGoal:5,
          startingCash:832,
          lives:14,
          enemyHealthMultiplier:1.065,
          enemySpeedMultiplier:0.979,
          extraEnemies:1,
          enemyRewardMultiplier:1.08,
          spawnIntervalMultiplier:1.06,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.04,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 2 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:3,
          title:"De koperen woestijn · Verloren zendmast",
          mapId:"meadow",
          weather:"clear",
          waveGoal:6,
          startingCash:812,
          lives:13,
          enemyHealthMultiplier:1.074,
          enemySpeedMultiplier:0.979,
          extraEnemies:1,
          enemyRewardMultiplier:1.08,
          spawnIntervalMultiplier:1.04,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.09,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 3 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:4,
          title:"De koperen woestijn · Nachtelijke wacht",
          mapId:"crossroads",
          weather:"rain",
          waveGoal:7,
          startingCash:792,
          lives:15,
          enemyHealthMultiplier:1.083,
          enemySpeedMultiplier:0.979,
          extraEnemies:2,
          enemyRewardMultiplier:1.09,
          spawnIntervalMultiplier:1.02,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.14,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 4 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:5,
          title:"De koperen woestijn · Gebroken brug",
          mapId:"swamp",
          weather:"snow",
          waveGoal:8,
          startingCash:772,
          lives:14,
          enemyHealthMultiplier:1.092,
          enemySpeedMultiplier:0.979,
          extraEnemies:2,
          enemyRewardMultiplier:1.09,
          spawnIntervalMultiplier:1.0,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.18,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 5 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:6,
          title:"De koperen woestijn · Rode rook",
          mapId:"desert",
          weather:"mist",
          waveGoal:4,
          startingCash:752,
          lives:13,
          enemyHealthMultiplier:1.101,
          enemySpeedMultiplier:0.979,
          extraEnemies:2,
          enemyRewardMultiplier:1.09,
          spawnIntervalMultiplier:0.98,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.23,
          objective:"Versla de baas en houd de route open.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 6 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:7,
          title:"De koperen woestijn · Lange mars",
          mapId:"highlands",
          weather:"clear",
          waveGoal:5,
          startingCash:732,
          lives:15,
          enemyHealthMultiplier:1.11,
          enemySpeedMultiplier:0.979,
          extraEnemies:3,
          enemyRewardMultiplier:1.1,
          spawnIntervalMultiplier:0.97,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.27,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 7 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:8,
          title:"De koperen woestijn · Schaduwlinie",
          mapId:"meadow",
          weather:"rain",
          waveGoal:6,
          startingCash:712,
          lives:14,
          enemyHealthMultiplier:1.119,
          enemySpeedMultiplier:0.979,
          extraEnemies:3,
          enemyRewardMultiplier:1.1,
          spawnIntervalMultiplier:0.95,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.31,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 8 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:9,
          title:"De koperen woestijn · Stormsein",
          mapId:"crossroads",
          weather:"snow",
          waveGoal:7,
          startingCash:692,
          lives:13,
          enemyHealthMultiplier:1.128,
          enemySpeedMultiplier:0.979,
          extraEnemies:3,
          enemyRewardMultiplier:1.11,
          spawnIntervalMultiplier:0.93,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.36,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 9 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:4,
          stage:10,
          title:"De koperen woestijn · Laatste licht",
          mapId:"swamp",
          weather:"mist",
          waveGoal:8,
          startingCash:672,
          lives:15,
          enemyHealthMultiplier:1.137,
          enemySpeedMultiplier:0.979,
          extraEnemies:4,
          enemyRewardMultiplier:1.11,
          spawnIntervalMultiplier:0.91,
          airstrikeDamageMultiplier:1.15,
          perfectBonusMultiplier:1.41,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 10 in De koperen woestijn: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:1,
          title:"De hoge passen · Stille dagen",
          mapId:"highlands",
          weather:"clear",
          waveGoal:4,
          startingCash:826,
          lives:14,
          enemyHealthMultiplier:1.088,
          enemySpeedMultiplier:0.992,
          extraEnemies:2,
          enemyRewardMultiplier:1.05,
          spawnIntervalMultiplier:1.04,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.0,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 1 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:2,
          title:"De hoge passen · Eerste contact",
          mapId:"meadow",
          weather:"rain",
          waveGoal:5,
          startingCash:806,
          lives:13,
          enemyHealthMultiplier:1.097,
          enemySpeedMultiplier:0.992,
          extraEnemies:2,
          enemyRewardMultiplier:1.05,
          spawnIntervalMultiplier:1.02,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.04,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 2 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:3,
          title:"De hoge passen · Verloren zendmast",
          mapId:"crossroads",
          weather:"snow",
          waveGoal:6,
          startingCash:786,
          lives:12,
          enemyHealthMultiplier:1.106,
          enemySpeedMultiplier:0.992,
          extraEnemies:2,
          enemyRewardMultiplier:1.06,
          spawnIntervalMultiplier:1.0,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.09,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 3 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:4,
          title:"De hoge passen · Nachtelijke wacht",
          mapId:"swamp",
          weather:"mist",
          waveGoal:7,
          startingCash:766,
          lives:14,
          enemyHealthMultiplier:1.115,
          enemySpeedMultiplier:0.992,
          extraEnemies:3,
          enemyRewardMultiplier:1.06,
          spawnIntervalMultiplier:0.99,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.14,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 4 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:5,
          title:"De hoge passen · Gebroken brug",
          mapId:"desert",
          weather:"clear",
          waveGoal:8,
          startingCash:746,
          lives:13,
          enemyHealthMultiplier:1.124,
          enemySpeedMultiplier:0.992,
          extraEnemies:3,
          enemyRewardMultiplier:1.07,
          spawnIntervalMultiplier:0.97,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.18,
          objective:"Versla de baas en houd de route open.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 5 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:6,
          title:"De hoge passen · Rode rook",
          mapId:"highlands",
          weather:"rain",
          waveGoal:4,
          startingCash:726,
          lives:12,
          enemyHealthMultiplier:1.133,
          enemySpeedMultiplier:0.992,
          extraEnemies:3,
          enemyRewardMultiplier:1.07,
          spawnIntervalMultiplier:0.95,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.23,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 6 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:7,
          title:"De hoge passen · Lange mars",
          mapId:"meadow",
          weather:"snow",
          waveGoal:5,
          startingCash:706,
          lives:14,
          enemyHealthMultiplier:1.142,
          enemySpeedMultiplier:0.992,
          extraEnemies:4,
          enemyRewardMultiplier:1.07,
          spawnIntervalMultiplier:0.93,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.27,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 7 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:8,
          title:"De hoge passen · Schaduwlinie",
          mapId:"crossroads",
          weather:"mist",
          waveGoal:6,
          startingCash:686,
          lives:13,
          enemyHealthMultiplier:1.151,
          enemySpeedMultiplier:0.992,
          extraEnemies:4,
          enemyRewardMultiplier:1.08,
          spawnIntervalMultiplier:0.91,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.31,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 8 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:9,
          title:"De hoge passen · Stormsein",
          mapId:"swamp",
          weather:"clear",
          waveGoal:7,
          startingCash:666,
          lives:12,
          enemyHealthMultiplier:1.16,
          enemySpeedMultiplier:0.992,
          extraEnemies:4,
          enemyRewardMultiplier:1.08,
          spawnIntervalMultiplier:0.9,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.36,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 9 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:5,
          stage:10,
          title:"De hoge passen · Laatste licht",
          mapId:"desert",
          weather:"rain",
          waveGoal:8,
          startingCash:646,
          lives:14,
          enemyHealthMultiplier:1.169,
          enemySpeedMultiplier:0.992,
          extraEnemies:5,
          enemyRewardMultiplier:1.09,
          spawnIntervalMultiplier:0.88,
          airstrikeDamageMultiplier:1.11,
          perfectBonusMultiplier:1.41,
          objective:"Versla de baas en houd de route open.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 10 in De hoge passen: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:1,
          title:"De vergeten route · Stille dagen",
          mapId:"meadow",
          weather:"snow",
          waveGoal:4,
          startingCash:800,
          lives:18,
          enemyHealthMultiplier:1.12,
          enemySpeedMultiplier:1.005,
          extraEnemies:2,
          enemyRewardMultiplier:1.02,
          spawnIntervalMultiplier:1.0,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.0,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 1 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:2,
          title:"De vergeten route · Eerste contact",
          mapId:"crossroads",
          weather:"mist",
          waveGoal:5,
          startingCash:780,
          lives:17,
          enemyHealthMultiplier:1.129,
          enemySpeedMultiplier:1.005,
          extraEnemies:2,
          enemyRewardMultiplier:1.03,
          spawnIntervalMultiplier:0.99,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.04,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 2 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:3,
          title:"De vergeten route · Verloren zendmast",
          mapId:"swamp",
          weather:"clear",
          waveGoal:6,
          startingCash:760,
          lives:16,
          enemyHealthMultiplier:1.138,
          enemySpeedMultiplier:1.005,
          extraEnemies:2,
          enemyRewardMultiplier:1.03,
          spawnIntervalMultiplier:0.97,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.09,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 3 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:4,
          title:"De vergeten route · Nachtelijke wacht",
          mapId:"desert",
          weather:"rain",
          waveGoal:7,
          startingCash:740,
          lives:18,
          enemyHealthMultiplier:1.147,
          enemySpeedMultiplier:1.005,
          extraEnemies:3,
          enemyRewardMultiplier:1.04,
          spawnIntervalMultiplier:0.95,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.14,
          objective:"Versla de baas en houd de route open.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 4 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:5,
          title:"De vergeten route · Gebroken brug",
          mapId:"highlands",
          weather:"snow",
          waveGoal:8,
          startingCash:720,
          lives:17,
          enemyHealthMultiplier:1.156,
          enemySpeedMultiplier:1.005,
          extraEnemies:3,
          enemyRewardMultiplier:1.04,
          spawnIntervalMultiplier:0.93,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.18,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 5 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:6,
          title:"De vergeten route · Rode rook",
          mapId:"meadow",
          weather:"mist",
          waveGoal:4,
          startingCash:700,
          lives:16,
          enemyHealthMultiplier:1.165,
          enemySpeedMultiplier:1.005,
          extraEnemies:3,
          enemyRewardMultiplier:1.04,
          spawnIntervalMultiplier:0.91,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.23,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 6 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:7,
          title:"De vergeten route · Lange mars",
          mapId:"crossroads",
          weather:"clear",
          waveGoal:5,
          startingCash:680,
          lives:18,
          enemyHealthMultiplier:1.174,
          enemySpeedMultiplier:1.005,
          extraEnemies:4,
          enemyRewardMultiplier:1.05,
          spawnIntervalMultiplier:0.9,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.27,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 7 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:8,
          title:"De vergeten route · Schaduwlinie",
          mapId:"swamp",
          weather:"rain",
          waveGoal:6,
          startingCash:660,
          lives:17,
          enemyHealthMultiplier:1.183,
          enemySpeedMultiplier:1.005,
          extraEnemies:4,
          enemyRewardMultiplier:1.05,
          spawnIntervalMultiplier:0.88,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.31,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 8 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:9,
          title:"De vergeten route · Stormsein",
          mapId:"desert",
          weather:"snow",
          waveGoal:7,
          startingCash:640,
          lives:16,
          enemyHealthMultiplier:1.192,
          enemySpeedMultiplier:1.005,
          extraEnemies:4,
          enemyRewardMultiplier:1.06,
          spawnIntervalMultiplier:0.86,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.36,
          objective:"Versla de baas en houd de route open.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 9 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:6,
          stage:10,
          title:"De vergeten route · Laatste licht",
          mapId:"highlands",
          weather:"mist",
          waveGoal:8,
          startingCash:620,
          lives:18,
          enemyHealthMultiplier:1.201,
          enemySpeedMultiplier:1.005,
          extraEnemies:5,
          enemyRewardMultiplier:1.06,
          spawnIntervalMultiplier:0.84,
          airstrikeDamageMultiplier:1.07,
          perfectBonusMultiplier:1.41,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 10 in De vergeten route: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:1,
          title:"De noordelijke linie · Stille dagen",
          mapId:"crossroads",
          weather:"clear",
          waveGoal:5,
          startingCash:774,
          lives:17,
          enemyHealthMultiplier:1.152,
          enemySpeedMultiplier:1.018,
          extraEnemies:3,
          enemyRewardMultiplier:1.0,
          spawnIntervalMultiplier:0.97,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.0,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 1 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:2,
          title:"De noordelijke linie · Eerste contact",
          mapId:"swamp",
          weather:"rain",
          waveGoal:6,
          startingCash:754,
          lives:16,
          enemyHealthMultiplier:1.161,
          enemySpeedMultiplier:1.018,
          extraEnemies:3,
          enemyRewardMultiplier:1.0,
          spawnIntervalMultiplier:0.95,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.04,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 2 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:3,
          title:"De noordelijke linie · Verloren zendmast",
          mapId:"desert",
          weather:"snow",
          waveGoal:7,
          startingCash:734,
          lives:15,
          enemyHealthMultiplier:1.17,
          enemySpeedMultiplier:1.018,
          extraEnemies:3,
          enemyRewardMultiplier:1.01,
          spawnIntervalMultiplier:0.93,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.09,
          objective:"Versla de baas en houd de route open.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 3 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:4,
          title:"De noordelijke linie · Nachtelijke wacht",
          mapId:"highlands",
          weather:"mist",
          waveGoal:8,
          startingCash:714,
          lives:17,
          enemyHealthMultiplier:1.179,
          enemySpeedMultiplier:1.018,
          extraEnemies:4,
          enemyRewardMultiplier:1.01,
          spawnIntervalMultiplier:0.92,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.14,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 4 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:5,
          title:"De noordelijke linie · Gebroken brug",
          mapId:"meadow",
          weather:"clear",
          waveGoal:9,
          startingCash:694,
          lives:16,
          enemyHealthMultiplier:1.188,
          enemySpeedMultiplier:1.018,
          extraEnemies:4,
          enemyRewardMultiplier:1.02,
          spawnIntervalMultiplier:0.9,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.18,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 5 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:6,
          title:"De noordelijke linie · Rode rook",
          mapId:"crossroads",
          weather:"rain",
          waveGoal:5,
          startingCash:674,
          lives:15,
          enemyHealthMultiplier:1.197,
          enemySpeedMultiplier:1.018,
          extraEnemies:4,
          enemyRewardMultiplier:1.02,
          spawnIntervalMultiplier:0.88,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.23,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 6 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:7,
          title:"De noordelijke linie · Lange mars",
          mapId:"swamp",
          weather:"snow",
          waveGoal:6,
          startingCash:654,
          lives:17,
          enemyHealthMultiplier:1.206,
          enemySpeedMultiplier:1.018,
          extraEnemies:5,
          enemyRewardMultiplier:1.02,
          spawnIntervalMultiplier:0.86,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.27,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 7 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:8,
          title:"De noordelijke linie · Schaduwlinie",
          mapId:"desert",
          weather:"mist",
          waveGoal:7,
          startingCash:634,
          lives:16,
          enemyHealthMultiplier:1.215,
          enemySpeedMultiplier:1.018,
          extraEnemies:5,
          enemyRewardMultiplier:1.03,
          spawnIntervalMultiplier:0.84,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.31,
          objective:"Versla de baas en houd de route open.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 8 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:9,
          title:"De noordelijke linie · Stormsein",
          mapId:"highlands",
          weather:"clear",
          waveGoal:8,
          startingCash:614,
          lives:15,
          enemyHealthMultiplier:1.224,
          enemySpeedMultiplier:1.018,
          extraEnemies:5,
          enemyRewardMultiplier:1.03,
          spawnIntervalMultiplier:0.83,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.36,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 9 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:7,
          stage:10,
          title:"De noordelijke linie · Laatste licht",
          mapId:"meadow",
          weather:"rain",
          waveGoal:9,
          startingCash:594,
          lives:17,
          enemyHealthMultiplier:1.233,
          enemySpeedMultiplier:1.018,
          extraEnemies:6,
          enemyRewardMultiplier:1.04,
          spawnIntervalMultiplier:0.81,
          airstrikeDamageMultiplier:1.04,
          perfectBonusMultiplier:1.41,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 10 in De noordelijke linie: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:1,
          title:"De laatste bevoorrading · Stille dagen",
          mapId:"swamp",
          weather:"snow",
          waveGoal:5,
          startingCash:748,
          lives:16,
          enemyHealthMultiplier:1.184,
          enemySpeedMultiplier:1.031,
          extraEnemies:3,
          enemyRewardMultiplier:0.97,
          spawnIntervalMultiplier:0.93,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.0,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 1 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:2,
          title:"De laatste bevoorrading · Eerste contact",
          mapId:"desert",
          weather:"mist",
          waveGoal:6,
          startingCash:728,
          lives:15,
          enemyHealthMultiplier:1.193,
          enemySpeedMultiplier:1.031,
          extraEnemies:3,
          enemyRewardMultiplier:0.98,
          spawnIntervalMultiplier:0.92,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.04,
          objective:"Versla de baas en houd de route open.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 2 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:3,
          title:"De laatste bevoorrading · Verloren zendmast",
          mapId:"highlands",
          weather:"clear",
          waveGoal:7,
          startingCash:708,
          lives:14,
          enemyHealthMultiplier:1.202,
          enemySpeedMultiplier:1.031,
          extraEnemies:3,
          enemyRewardMultiplier:0.98,
          spawnIntervalMultiplier:0.9,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.09,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 3 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:4,
          title:"De laatste bevoorrading · Nachtelijke wacht",
          mapId:"meadow",
          weather:"rain",
          waveGoal:8,
          startingCash:688,
          lives:16,
          enemyHealthMultiplier:1.211,
          enemySpeedMultiplier:1.031,
          extraEnemies:4,
          enemyRewardMultiplier:0.99,
          spawnIntervalMultiplier:0.88,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.14,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 4 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:5,
          title:"De laatste bevoorrading · Gebroken brug",
          mapId:"crossroads",
          weather:"snow",
          waveGoal:9,
          startingCash:668,
          lives:15,
          enemyHealthMultiplier:1.22,
          enemySpeedMultiplier:1.031,
          extraEnemies:4,
          enemyRewardMultiplier:0.99,
          spawnIntervalMultiplier:0.86,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.18,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 5 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:6,
          title:"De laatste bevoorrading · Rode rook",
          mapId:"swamp",
          weather:"mist",
          waveGoal:5,
          startingCash:648,
          lives:14,
          enemyHealthMultiplier:1.229,
          enemySpeedMultiplier:1.031,
          extraEnemies:4,
          enemyRewardMultiplier:0.99,
          spawnIntervalMultiplier:0.84,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.23,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 6 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:7,
          title:"De laatste bevoorrading · Lange mars",
          mapId:"desert",
          weather:"clear",
          waveGoal:6,
          startingCash:628,
          lives:16,
          enemyHealthMultiplier:1.238,
          enemySpeedMultiplier:1.031,
          extraEnemies:5,
          enemyRewardMultiplier:1.0,
          spawnIntervalMultiplier:0.83,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.27,
          objective:"Versla de baas en houd de route open.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 7 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:8,
          title:"De laatste bevoorrading · Schaduwlinie",
          mapId:"highlands",
          weather:"rain",
          waveGoal:7,
          startingCash:608,
          lives:15,
          enemyHealthMultiplier:1.247,
          enemySpeedMultiplier:1.031,
          extraEnemies:5,
          enemyRewardMultiplier:1.0,
          spawnIntervalMultiplier:0.81,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.31,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 8 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:9,
          title:"De laatste bevoorrading · Stormsein",
          mapId:"meadow",
          weather:"snow",
          waveGoal:8,
          startingCash:588,
          lives:14,
          enemyHealthMultiplier:1.256,
          enemySpeedMultiplier:1.031,
          extraEnemies:5,
          enemyRewardMultiplier:1.01,
          spawnIntervalMultiplier:0.79,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.36,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 9 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:8,
          stage:10,
          title:"De laatste bevoorrading · Laatste licht",
          mapId:"crossroads",
          weather:"mist",
          waveGoal:9,
          startingCash:568,
          lives:16,
          enemyHealthMultiplier:1.265,
          enemySpeedMultiplier:1.031,
          extraEnemies:6,
          enemyRewardMultiplier:1.01,
          spawnIntervalMultiplier:0.77,
          airstrikeDamageMultiplier:1.0,
          perfectBonusMultiplier:1.41,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 10 in De laatste bevoorrading: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:1,
          title:"De vallende sterren · Stille dagen",
          mapId:"desert",
          weather:"clear",
          waveGoal:5,
          startingCash:722,
          lives:15,
          enemyHealthMultiplier:1.216,
          enemySpeedMultiplier:1.044,
          extraEnemies:4,
          enemyRewardMultiplier:0.95,
          spawnIntervalMultiplier:0.9,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.0,
          objective:"Versla de baas en houd de route open.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 1 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:2,
          title:"De vallende sterren · Eerste contact",
          mapId:"highlands",
          weather:"rain",
          waveGoal:6,
          startingCash:702,
          lives:14,
          enemyHealthMultiplier:1.225,
          enemySpeedMultiplier:1.044,
          extraEnemies:4,
          enemyRewardMultiplier:0.95,
          spawnIntervalMultiplier:0.88,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.04,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 2 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:3,
          title:"De vallende sterren · Verloren zendmast",
          mapId:"meadow",
          weather:"snow",
          waveGoal:7,
          startingCash:682,
          lives:13,
          enemyHealthMultiplier:1.234,
          enemySpeedMultiplier:1.044,
          extraEnemies:4,
          enemyRewardMultiplier:0.96,
          spawnIntervalMultiplier:0.86,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.09,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 3 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:4,
          title:"De vallende sterren · Nachtelijke wacht",
          mapId:"crossroads",
          weather:"mist",
          waveGoal:8,
          startingCash:662,
          lives:15,
          enemyHealthMultiplier:1.243,
          enemySpeedMultiplier:1.044,
          extraEnemies:5,
          enemyRewardMultiplier:0.96,
          spawnIntervalMultiplier:0.85,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.14,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 4 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:5,
          title:"De vallende sterren · Gebroken brug",
          mapId:"swamp",
          weather:"clear",
          waveGoal:9,
          startingCash:642,
          lives:14,
          enemyHealthMultiplier:1.252,
          enemySpeedMultiplier:1.044,
          extraEnemies:5,
          enemyRewardMultiplier:0.97,
          spawnIntervalMultiplier:0.83,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.18,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 5 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:6,
          title:"De vallende sterren · Rode rook",
          mapId:"desert",
          weather:"rain",
          waveGoal:5,
          startingCash:622,
          lives:13,
          enemyHealthMultiplier:1.261,
          enemySpeedMultiplier:1.044,
          extraEnemies:5,
          enemyRewardMultiplier:0.97,
          spawnIntervalMultiplier:0.81,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.23,
          objective:"Versla de baas en houd de route open.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 6 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:7,
          title:"De vallende sterren · Lange mars",
          mapId:"highlands",
          weather:"snow",
          waveGoal:6,
          startingCash:602,
          lives:15,
          enemyHealthMultiplier:1.27,
          enemySpeedMultiplier:1.044,
          extraEnemies:6,
          enemyRewardMultiplier:0.97,
          spawnIntervalMultiplier:0.79,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.27,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 7 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:8,
          title:"De vallende sterren · Schaduwlinie",
          mapId:"meadow",
          weather:"mist",
          waveGoal:7,
          startingCash:582,
          lives:14,
          enemyHealthMultiplier:1.279,
          enemySpeedMultiplier:1.044,
          extraEnemies:6,
          enemyRewardMultiplier:0.98,
          spawnIntervalMultiplier:0.77,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.31,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 8 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:9,
          title:"De vallende sterren · Stormsein",
          mapId:"crossroads",
          weather:"clear",
          waveGoal:8,
          startingCash:562,
          lives:13,
          enemyHealthMultiplier:1.288,
          enemySpeedMultiplier:1.044,
          extraEnemies:6,
          enemyRewardMultiplier:0.98,
          spawnIntervalMultiplier:0.76,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.36,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 9 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:9,
          stage:10,
          title:"De vallende sterren · Laatste licht",
          mapId:"swamp",
          weather:"rain",
          waveGoal:9,
          startingCash:542,
          lives:15,
          enemyHealthMultiplier:1.297,
          enemySpeedMultiplier:1.044,
          extraEnemies:7,
          enemyRewardMultiplier:0.99,
          spawnIntervalMultiplier:0.74,
          airstrikeDamageMultiplier:0.97,
          perfectBonusMultiplier:1.41,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 10 in De vallende sterren: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:1,
          title:"De verre vesting · Stille dagen",
          mapId:"highlands",
          weather:"snow",
          waveGoal:6,
          startingCash:696,
          lives:14,
          enemyHealthMultiplier:1.248,
          enemySpeedMultiplier:1.057,
          extraEnemies:4,
          enemyRewardMultiplier:0.92,
          spawnIntervalMultiplier:0.86,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.0,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 1 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:2,
          title:"De verre vesting · Eerste contact",
          mapId:"meadow",
          weather:"mist",
          waveGoal:7,
          startingCash:676,
          lives:13,
          enemyHealthMultiplier:1.257,
          enemySpeedMultiplier:1.057,
          extraEnemies:4,
          enemyRewardMultiplier:0.93,
          spawnIntervalMultiplier:0.85,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.04,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 2 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:3,
          title:"De verre vesting · Verloren zendmast",
          mapId:"crossroads",
          weather:"clear",
          waveGoal:8,
          startingCash:656,
          lives:12,
          enemyHealthMultiplier:1.266,
          enemySpeedMultiplier:1.057,
          extraEnemies:4,
          enemyRewardMultiplier:0.93,
          spawnIntervalMultiplier:0.83,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.09,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 3 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:4,
          title:"De verre vesting · Nachtelijke wacht",
          mapId:"swamp",
          weather:"rain",
          waveGoal:9,
          startingCash:636,
          lives:14,
          enemyHealthMultiplier:1.275,
          enemySpeedMultiplier:1.057,
          extraEnemies:5,
          enemyRewardMultiplier:0.94,
          spawnIntervalMultiplier:0.81,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.14,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 4 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:5,
          title:"De verre vesting · Gebroken brug",
          mapId:"desert",
          weather:"snow",
          waveGoal:10,
          startingCash:616,
          lives:13,
          enemyHealthMultiplier:1.284,
          enemySpeedMultiplier:1.057,
          extraEnemies:5,
          enemyRewardMultiplier:0.94,
          spawnIntervalMultiplier:0.79,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.18,
          objective:"Versla de baas en houd de route open.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 5 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:6,
          title:"De verre vesting · Rode rook",
          mapId:"highlands",
          weather:"mist",
          waveGoal:6,
          startingCash:596,
          lives:12,
          enemyHealthMultiplier:1.293,
          enemySpeedMultiplier:1.057,
          extraEnemies:5,
          enemyRewardMultiplier:0.94,
          spawnIntervalMultiplier:0.77,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.23,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 6 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:7,
          title:"De verre vesting · Lange mars",
          mapId:"meadow",
          weather:"clear",
          waveGoal:7,
          startingCash:576,
          lives:14,
          enemyHealthMultiplier:1.302,
          enemySpeedMultiplier:1.057,
          extraEnemies:6,
          enemyRewardMultiplier:0.95,
          spawnIntervalMultiplier:0.76,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.27,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 7 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:8,
          title:"De verre vesting · Schaduwlinie",
          mapId:"crossroads",
          weather:"rain",
          waveGoal:8,
          startingCash:556,
          lives:13,
          enemyHealthMultiplier:1.311,
          enemySpeedMultiplier:1.057,
          extraEnemies:6,
          enemyRewardMultiplier:0.95,
          spawnIntervalMultiplier:0.74,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.31,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 8 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:9,
          title:"De verre vesting · Stormsein",
          mapId:"swamp",
          weather:"snow",
          waveGoal:9,
          startingCash:536,
          lives:12,
          enemyHealthMultiplier:1.32,
          enemySpeedMultiplier:1.057,
          extraEnemies:6,
          enemyRewardMultiplier:0.96,
          spawnIntervalMultiplier:0.72,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.36,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 9 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:10,
          stage:10,
          title:"De verre vesting · Laatste licht",
          mapId:"desert",
          weather:"mist",
          waveGoal:10,
          startingCash:516,
          lives:14,
          enemyHealthMultiplier:1.329,
          enemySpeedMultiplier:1.057,
          extraEnemies:7,
          enemyRewardMultiplier:0.96,
          spawnIntervalMultiplier:0.7,
          airstrikeDamageMultiplier:0.93,
          perfectBonusMultiplier:1.41,
          objective:"Versla de baas en houd de route open.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 10 in De verre vesting: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:1,
          title:"De ijszee · Stille dagen",
          mapId:"meadow",
          weather:"clear",
          waveGoal:6,
          startingCash:670,
          lives:18,
          enemyHealthMultiplier:1.28,
          enemySpeedMultiplier:1.07,
          extraEnemies:5,
          enemyRewardMultiplier:0.9,
          spawnIntervalMultiplier:0.83,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.0,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 1 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:2,
          title:"De ijszee · Eerste contact",
          mapId:"crossroads",
          weather:"rain",
          waveGoal:7,
          startingCash:650,
          lives:17,
          enemyHealthMultiplier:1.289,
          enemySpeedMultiplier:1.07,
          extraEnemies:5,
          enemyRewardMultiplier:0.9,
          spawnIntervalMultiplier:0.81,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.04,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 2 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:3,
          title:"De ijszee · Verloren zendmast",
          mapId:"swamp",
          weather:"snow",
          waveGoal:8,
          startingCash:630,
          lives:16,
          enemyHealthMultiplier:1.298,
          enemySpeedMultiplier:1.07,
          extraEnemies:5,
          enemyRewardMultiplier:0.91,
          spawnIntervalMultiplier:0.79,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.09,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 3 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:4,
          title:"De ijszee · Nachtelijke wacht",
          mapId:"desert",
          weather:"mist",
          waveGoal:9,
          startingCash:610,
          lives:18,
          enemyHealthMultiplier:1.307,
          enemySpeedMultiplier:1.07,
          extraEnemies:6,
          enemyRewardMultiplier:0.91,
          spawnIntervalMultiplier:0.78,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.14,
          objective:"Versla de baas en houd de route open.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 4 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:5,
          title:"De ijszee · Gebroken brug",
          mapId:"highlands",
          weather:"clear",
          waveGoal:10,
          startingCash:590,
          lives:17,
          enemyHealthMultiplier:1.316,
          enemySpeedMultiplier:1.07,
          extraEnemies:6,
          enemyRewardMultiplier:0.92,
          spawnIntervalMultiplier:0.76,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.18,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 5 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:6,
          title:"De ijszee · Rode rook",
          mapId:"meadow",
          weather:"rain",
          waveGoal:6,
          startingCash:570,
          lives:16,
          enemyHealthMultiplier:1.325,
          enemySpeedMultiplier:1.07,
          extraEnemies:6,
          enemyRewardMultiplier:0.92,
          spawnIntervalMultiplier:0.74,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.23,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 6 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:7,
          title:"De ijszee · Lange mars",
          mapId:"crossroads",
          weather:"snow",
          waveGoal:7,
          startingCash:550,
          lives:18,
          enemyHealthMultiplier:1.334,
          enemySpeedMultiplier:1.07,
          extraEnemies:7,
          enemyRewardMultiplier:0.92,
          spawnIntervalMultiplier:0.72,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.27,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 7 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:8,
          title:"De ijszee · Schaduwlinie",
          mapId:"swamp",
          weather:"mist",
          waveGoal:8,
          startingCash:530,
          lives:17,
          enemyHealthMultiplier:1.343,
          enemySpeedMultiplier:1.07,
          extraEnemies:7,
          enemyRewardMultiplier:0.93,
          spawnIntervalMultiplier:0.7,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.31,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 8 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:9,
          title:"De ijszee · Stormsein",
          mapId:"desert",
          weather:"clear",
          waveGoal:9,
          startingCash:510,
          lives:16,
          enemyHealthMultiplier:1.352,
          enemySpeedMultiplier:1.07,
          extraEnemies:7,
          enemyRewardMultiplier:0.93,
          spawnIntervalMultiplier:0.69,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.36,
          objective:"Versla de baas en houd de route open.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 9 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:11,
          stage:10,
          title:"De ijszee · Laatste licht",
          mapId:"highlands",
          weather:"rain",
          waveGoal:10,
          startingCash:490,
          lives:18,
          enemyHealthMultiplier:1.361,
          enemySpeedMultiplier:1.07,
          extraEnemies:8,
          enemyRewardMultiplier:0.94,
          spawnIntervalMultiplier:0.67,
          airstrikeDamageMultiplier:0.9,
          perfectBonusMultiplier:1.41,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 10 in De ijszee: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:1,
          title:"Het laatste bastion · Stille dagen",
          mapId:"crossroads",
          weather:"snow",
          waveGoal:6,
          startingCash:644,
          lives:17,
          enemyHealthMultiplier:1.312,
          enemySpeedMultiplier:1.083,
          extraEnemies:5,
          enemyRewardMultiplier:0.87,
          spawnIntervalMultiplier:0.79,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.0,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De bevoorrading is schaars. Kies torens die meerdere doelen raken.",
          specialRule:"Zware colonne: vijanden hebben extra gezondheid.",
          debrief:"Operatie 1 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:2,
          title:"Het laatste bastion · Eerste contact",
          mapId:"swamp",
          weather:"mist",
          waveGoal:7,
          startingCash:624,
          lives:16,
          enemyHealthMultiplier:1.321,
          enemySpeedMultiplier:1.083,
          extraEnemies:5,
          enemyRewardMultiplier:0.88,
          spawnIntervalMultiplier:0.78,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.04,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een veldarts ondersteunt de aanval. Geef prioriteit aan kwetsbare groepen.",
          specialRule:"Bevoorradingssteun: vijanden leveren meer geld op.",
          debrief:"Operatie 2 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:3,
          title:"Het laatste bastion · Verloren zendmast",
          mapId:"desert",
          weather:"clear",
          waveGoal:8,
          startingCash:604,
          lives:15,
          enemyHealthMultiplier:1.33,
          enemySpeedMultiplier:1.083,
          extraEnemies:5,
          enemyRewardMultiplier:0.88,
          spawnIntervalMultiplier:0.76,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.09,
          objective:"Versla de baas en houd de route open.",
          briefing:"De route is lang, maar de vijand wordt sterker na elke aanvalsgolf.",
          specialRule:"Dichte formaties: extra vijanden per aanvalsgolf.",
          debrief:"Operatie 3 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:4,
          title:"Het laatste bastion · Nachtelijke wacht",
          mapId:"highlands",
          weather:"rain",
          waveGoal:9,
          startingCash:584,
          lives:17,
          enemyHealthMultiplier:1.339,
          enemySpeedMultiplier:1.083,
          extraEnemies:6,
          enemyRewardMultiplier:0.89,
          spawnIntervalMultiplier:0.74,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.14,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Zware eenheden naderen. Combineer bereik met gebiedsschade.",
          specialRule:"Snel offensief: aanvallen volgen elkaar sneller op.",
          debrief:"Operatie 4 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:5,
          title:"Het laatste bastion · Gebroken brug",
          mapId:"meadow",
          weather:"snow",
          waveGoal:10,
          startingCash:564,
          lives:16,
          enemyHealthMultiplier:1.348,
          enemySpeedMultiplier:1.083,
          extraEnemies:6,
          enemyRewardMultiplier:0.89,
          spawnIntervalMultiplier:0.72,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.18,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"Mist beperkt het zicht. Plaats torens dicht bij de route.",
          specialRule:"Luchtdekking: luchtsteun veroorzaakt extra schade.",
          debrief:"Operatie 5 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:6,
          title:"Het laatste bastion · Rode rook",
          mapId:"crossroads",
          weather:"mist",
          waveGoal:6,
          startingCash:544,
          lives:15,
          enemyHealthMultiplier:1.357,
          enemySpeedMultiplier:1.083,
          extraEnemies:6,
          enemyRewardMultiplier:0.89,
          spawnIntervalMultiplier:0.7,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.23,
          objective:"Verdedig de basis en voltooi de operatie.",
          briefing:"De basis heeft versterking nodig. Overleef de volledige operatie.",
          specialRule:"Zuinig bouwen: een schone golf geeft een grotere bonus.",
          debrief:"Operatie 6 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:7,
          title:"Het laatste bastion · Lange mars",
          mapId:"swamp",
          weather:"clear",
          waveGoal:7,
          startingCash:524,
          lives:17,
          enemyHealthMultiplier:1.366,
          enemySpeedMultiplier:1.083,
          extraEnemies:7,
          enemyRewardMultiplier:0.9,
          spawnIntervalMultiplier:0.69,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.27,
          objective:"Houd minstens de helft van je basislevens over.",
          briefing:"Een baas leidt de aanval. Bewaar je luchtsteun voor een dicht cluster.",
          specialRule:"Natte munitie: de zichtbaarheid is beperkt door regen.",
          debrief:"Operatie 7 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:8,
          title:"Het laatste bastion · Schaduwlinie",
          mapId:"desert",
          weather:"rain",
          waveGoal:8,
          startingCash:504,
          lives:16,
          enemyHealthMultiplier:1.375,
          enemySpeedMultiplier:1.083,
          extraEnemies:7,
          enemyRewardMultiplier:0.9,
          spawnIntervalMultiplier:0.67,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.31,
          objective:"Versla de baas en houd de route open.",
          briefing:"Verken de toegangsweg en houd de basis onbeschadigd.",
          specialRule:"Bevroren passen: de basis heeft minder reservelevens.",
          debrief:"Operatie 8 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:9,
          title:"Het laatste bastion · Stormsein",
          mapId:"highlands",
          weather:"snow",
          waveGoal:9,
          startingCash:484,
          lives:15,
          enemyHealthMultiplier:1.384,
          enemySpeedMultiplier:1.083,
          extraEnemies:7,
          enemyRewardMultiplier:0.91,
          spawnIntervalMultiplier:0.65,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.36,
          objective:"Gebruik je geld zorgvuldig en blijf de golven overleven.",
          briefing:"Een verkenner heeft vijandelijke beweging gezien. Bereid je verdediging voor.",
          specialRule:"Commandopost: overleef meer golven om te winnen.",
          debrief:"Operatie 9 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
        {
          chapter:12,
          stage:10,
          title:"Het laatste bastion · Laatste licht",
          mapId:"meadow",
          weather:"mist",
          waveGoal:10,
          startingCash:464,
          lives:17,
          enemyHealthMultiplier:1.393,
          enemySpeedMultiplier:1.083,
          extraEnemies:8,
          enemyRewardMultiplier:0.91,
          spawnIntervalMultiplier:0.63,
          airstrikeDamageMultiplier:0.86,
          perfectBonusMultiplier:1.41,
          objective:"Overleef de aangegeven aanvalsgolven.",
          briefing:"De tegenstander test onze linie. Spreid je vuurkracht over de bochten.",
          specialRule:"Verkenningsgolf: snelle eenheden verschijnen vaker.",
          debrief:"Operatie 10 in Het laatste bastion: houd de linie vast en bereid de volgende sector voor.",
        },
      ];
      const campaignMissions = missionBlueprints.map((mission,index)=>({
        ...mission,
        id:index+1
      }));
      let activeMission=null;
      let campaignUnlocked=1;
      let editorPoints=[{x:0,y:320},{x:960,y:320}];
      let activeCodexFilter='all';
      const customMapIds=[];

      const ui = {
        cash: document.getElementById('cash'),
        lives: document.getElementById('lives'),
        wave: document.getElementById('wave'),
        toast: document.getElementById('toast'),
        start: document.getElementById('start-wave'),
        pause: document.getElementById('pause-game'),
        towerPanel: document.getElementById('tower-panel'),
        selectedName: document.getElementById('selected-name'),
        selectedInfo: document.getElementById('selected-info'),
        upgrade: document.getElementById('upgrade'),
        targetMode: document.getElementById('target-mode'),
        sell: document.getElementById('sell'),
        difficulty: document.getElementById('difficulty-select'),
        difficultyDescription: document.getElementById('difficulty-description'),
        weather: document.getElementById('weather-select'),
        quality: document.getElementById('quality-select'),
        difficultyName: document.getElementById('difficulty-name'),
        pauseIndicator: document.getElementById('pause-indicator'),
        overlay: document.getElementById('overlay'),
        finalScore: document.getElementById('final-score'),
        mapName: document.getElementById('map-name'),
        waveProgress: document.getElementById('wave-progress-fill'),
        waveProgressLabel: document.getElementById('wave-progress-label'),
        speed: document.getElementById('speed-game'),
        airstrike: document.getElementById('airstrike-game'),
        combatFeed: document.getElementById('combat-feed'),
        screenFlash: document.getElementById('screen-flash'),
        bestRecords: document.getElementById('best-records'),
        campaignGrid: document.getElementById('campaign-grid'),
        campaignSummary: document.getElementById('campaign-summary'),
        editorName: document.getElementById('editor-name'),
        editorBiome: document.getElementById('editor-biome'),
        editorLives: document.getElementById('editor-lives'),
        editorRoute: document.getElementById('editor-route'),
        editorPoints: document.getElementById('editor-points'),
        editorMessage: document.getElementById('editor-message'),
        editorPreview: document.getElementById('editor-preview'),
        customMaps: document.getElementById('custom-maps'),
        codexEntries: document.getElementById('codex-entries'),
        overlayTitle: document.getElementById('overlay-title'),
        nextMission: document.getElementById('next-mission')
      };

      function readRecords() {
        try {
          const parsed = JSON.parse(localStorage.getItem(recordStorageKey) || '{}');
          return parsed && typeof parsed === 'object' && !Array.isArray(parsed) ? parsed : {};
        } catch (error) {
          console.warn('Blockfront-records konden niet worden gelezen.', error);
          return {};
        }
      }

      const bestRecords = readRecords();

      function readCampaignProgress() {
        try {
          const value=Number(localStorage.getItem(campaignStorageKey));
          return Number.isInteger(value)?Math.max(1,Math.min(campaignMissions.length+1,value)):1;
        } catch (error) {
          console.warn('Campagnevoortgang kon niet worden gelezen.',error);
          return 1;
        }
      }

      function saveCampaignProgress() {
        try {
          localStorage.setItem(campaignStorageKey,String(campaignUnlocked));
        } catch (error) {
          console.warn('Campagnevoortgang kon niet worden opgeslagen.',error);
        }
      }

      campaignUnlocked=readCampaignProgress();

      function renderCampaign() {
        ui.campaignGrid.replaceChildren();
        const completed=Math.max(0,campaignUnlocked-1);
        ui.campaignSummary.textContent=completed+
          ' van '+campaignMissions.length+' operaties voltooid · volgende vrijgespeeld: '+
          Math.min(campaignUnlocked,campaignMissions.length);

        for (const mission of campaignMissions) {
          const unlocked=mission.id<=campaignUnlocked;
          const card=document.createElement('button');
          const title=document.createElement('span');
          const meta=document.createElement('span');
          const briefing=document.createElement('span');
          const status=document.createElement('span');
          card.type='button';
          card.className='mission-card '+(unlocked?'unlocked':'locked');
          card.disabled=!unlocked;
          title.className='mission-title';
          title.textContent=mission.title;
          meta.className='mission-meta';
          meta.textContent=maps[mission.mapId].name+' · '+mission.waveGoal+' golven · '+
            weatherLabel(mission.weather);
          briefing.className='mission-brief';
          briefing.textContent=mission.briefing+' Doel: '+mission.objective+
            ' Regel: '+mission.specialRule;
          status.className='mission-status';
          status.textContent=mission.id<campaignUnlocked?'Voltooid':
            unlocked?'Nu beschikbaar':'Vergrendeld';
          card.append(title,meta,briefing,status);
          if (unlocked) card.addEventListener('click',()=>startMission(mission.id));
          ui.campaignGrid.append(card);
        }
      }

      function startMission(missionId) {
        const mission=campaignMissions.find(item=>item.id===missionId);
        if (!mission||mission.id>campaignUnlocked) return;
        activeMission=mission;
        ui.weather.value=mission.weather;
        startMap(mission.mapId);
      }

      function completeMission(mission) {
        waveActive=false;
        gameOver=true;
        campaignUnlocked=Math.max(campaignUnlocked,mission.id+1);
        saveCampaignProgress();
        ui.overlayTitle.textContent='Operatie geslaagd!';
        ui.finalScore.textContent=mission.title+' voltooid. '+mission.debrief;
        ui.nextMission.style.display=mission.id<campaignMissions.length?'block':'none';
        ui.nextMission.disabled=mission.id>=campaignMissions.length;
        ui.overlay.style.display='grid';
        addFeedMessage('Operatie voltooid · volgende missie vrijgespeeld','reward');
        renderCampaign();
      }

      function showMenuView(viewId) {
        document.querySelectorAll('.menu-view').forEach(view=>{
          view.classList.toggle('active',view.id===viewId);
        });
        document.querySelectorAll('.menu-tab').forEach(button=>{
          const active=button.dataset.view===viewId;
          button.classList.toggle('active',active);
          button.setAttribute('aria-pressed',String(active));
        });
        if (viewId==='campaign-view') renderCampaign();
        if (viewId==='codex-view') renderCodex(activeCodexFilter);
      }

      const codexEntries=[
        {category:'tower',title:'Scout',description:'Een betaalbare allroundtoren. Plaats hem bij bochten om veel route af te dekken.',detail:'Snelheid: hoog · bereik: gemiddeld · geschikt als eerste aankoop.'},
        {category:'tower',title:'Sniper',description:'Schiet langzaam, maar raakt doelen van ver met zware enkeldoelschade.',detail:'Zet hem op open plekken en kies Sterkste tegen grote vijanden.'},
        {category:'tower',title:'Cannon',description:'Explosieve projectielen raken het doel en vijanden vlak eromheen.',detail:'Werkzaam tegen groepen; minder sterk als een vijand alleen loopt.'},
        {category:'tower',title:'Frost',description:'Vertraagt een geraakt doel zodat andere torens langer kunnen aanvallen.',detail:'Combineer met mortier of kanon voor extra tijd in de explosiezone.'},
        {category:'tower',title:'Laser',description:'Raakt onmiddellijk zonder projectielreistijd.',detail:'De straal is betrouwbaar tegen snelle vijanden en korte afstanden.'},
        {category:'tower',title:'Tesla',description:'Elektrische schoten springen door naar meerdere nabije doelen.',detail:'Plaats hem waar vijanden dicht bij elkaar langs de route bewegen.'},
        {category:'tower',title:'Mortar',description:'Heeft groot bereik en veroorzaakt een brede explosie.',detail:'Richt op drukke routekruisingen; de vuursnelheid is laag.'},
        {category:'tower',title:'Rapid',description:'Vuurt lichte kogels op hoge snelheid.',detail:'Een betaalbare manier om snelle vijanden consequent te raken.'},
        {category:'enemy',title:'Verkenner',description:'De gewone vijand heeft gemiddelde snelheid en gezondheid.',detail:'Een enkele Scout kan vroege golven meestal aan.'},
        {category:'enemy',title:'Loper',description:'Snelle vijand die in vroege golven verschijnt.',detail:'Gebruik snelle torens, vertraging of een toren met directe inslag.'},
        {category:'enemy',title:'Zware eenheid',description:'Langzaam en bestand tegen meer schade dan een gewone vijand.',detail:'Sniper en Mortar zijn efficiënt wanneer het doelwit alleen loopt.'},
        {category:'enemy',title:'Veldarts',description:'Verschijnt vanaf golf vijf en herstelt nabije beschadigde vijanden.',detail:'Geef de veldarts voorrang voordat hij een groep kan genezen.'},
        {category:'enemy',title:'Baas',description:'Verschijnt aan het begin van elke vijfde golf met extra gezondheid.',detail:'Spaar cash voor upgrades en gebruik luchtsteun bij een groep.'},
        {category:'map',title:'Meadow',description:'Een toegankelijke route met meerdere rechte stukken en ruime bouwzones.',detail:'Plaats torens bij de U-bochten voor maximaal bereik.'},
        {category:'map',title:'Crossroads',description:'Een lange route met opeenvolgende kruisingen.',detail:'Bereik en doelprioriteit zijn hier belangrijk.'},
        {category:'map',title:'Swamp',description:'Een korte, gevaarlijke route met weinig reactietijd.',detail:'Zet schade vroeg neer en versterk de laatste bocht.'},
        {category:'map',title:'Desert',description:'Een kronkelende route door open zand.',detail:'Mortier en Tesla bestrijken meerdere routevakken.'},
        {category:'map',title:'Highlands',description:'Een rotsachtig terrein met scherpe bochten en smalle passen.',detail:'De lange rechte stukken zijn geschikt voor Sniper-torens.'},
        {category:'tactic',title:'Voorste doelwit',description:'De toren kiest de vijand die het verst over de route is gekomen.',detail:'Gebruik dit om ontsnappende lopers te onderscheppen.'},
        {category:'tactic',title:'Sterkste doelwit',description:'De toren richt zich op de vijand met de meeste huidige gezondheid.',detail:'Handig om zware eenheden en bazen vroeg te verzwakken.'},
        {category:'tactic',title:'Dichtstbijzijnde doelwit',description:'De toren kiest de vijand die geometrisch het dichtstbij staat.',detail:'Werkt goed wanneer een toren een klein gebied bewaakt.'},
        {category:'tactic',title:'Golven plannen',description:'Bekijk het golfbericht boven in beeld voordat je een nieuwe golf start.',detail:'Baasgolven staan elke vijfde golf op het programma.'},
        {category:'tactic',title:'Schone verdediging',description:'Verlies tijdens een golf geen basislevens om extra geld te verdienen.',detail:'De bonus groeit mee met het golfnummer.'},
        {category:'tactic',title:'Luchtsteun',description:'Richt de vaardigheid op een groep; het kost $300 en herlaadt daarna.',detail:'Toets A activeert het richten. Escape annuleert.'},
        {category:'tactic',title:'Terreindetail',description:'Verlaag de terreininstelling als je apparaat moeite heeft met veel objecten.',detail:'De spelregels blijven gelijk; alleen decor en weerdeeltjes veranderen.'},
        {category:'tactic',title:'Kaartwerkplaats',description:'Maak een route met minstens twee bochten en bewaar hem voor later.',detail:'Zelfgemaakte kaarten blijven lokaal in deze browser opgeslagen.'}
      ];

      function renderCodex(filter='all') {
        activeCodexFilter=filter;
        ui.codexEntries.replaceChildren();
        const entries=codexEntries.filter(entry=>filter==='all'||entry.category===filter);
        for (const entry of entries) {
          const card=document.createElement('article');
          const title=document.createElement('h3');
          const description=document.createElement('p');
          const detail=document.createElement('small');
          card.className='codex-entry';
          title.textContent=entry.title;
          description.textContent=entry.description;
          detail.textContent=entry.detail;
          card.append(title,description,detail);
          ui.codexEntries.append(card);
        }
        document.querySelectorAll('.codex-filter').forEach(button=>{
          button.classList.toggle('active',button.dataset.codex===filter);
        });
      }

      function validCustomMap(map) {
        return map&&typeof map.id==='string'&&/^custom-[a-z0-9-]+$/i.test(map.id)&&
          typeof map.name==='string'&&map.name.trim().length>=3&&map.name.length<=28&&
          typeof map.biome==='string'&&['meadow','forest','swamp','desert','highlands'].includes(map.biome)&&
          typeof map.ground==='string'&&/^#[0-9a-f]{6}$/i.test(map.ground)&&
          typeof map.road==='string'&&/^#[0-9a-f]{6}$/i.test(map.road)&&
          Number.isInteger(map.lives)&&map.lives>=5&&map.lives<=40&&
          Array.isArray(map.path)&&map.path.length>=2&&map.path.length<=20&&
          map.path.every(point=>point&&Number.isFinite(point.x)&&Number.isFinite(point.y)&&
            point.x>=0&&point.x<=W&&point.y>=0&&point.y<=H)&&
          map.path[0].x===0&&map.path[map.path.length-1].x===W&&
          map.path.every((point,index)=>index===0||
            Math.hypot(point.x-map.path[index-1].x,point.y-map.path[index-1].y)>=1);
      }

      function loadCustomMaps() {
        let saved=[];
        try {
          const parsed=JSON.parse(localStorage.getItem(customMapStorageKey)||'[]');
          if (Array.isArray(parsed)) saved=parsed.filter(validCustomMap);
        } catch (error) {
          console.warn('Eigen kaarten konden niet worden gelezen.',error);
        }
        for (const map of saved) {
          maps[map.id]={
            name:map.name.toUpperCase(),
            biome:map.biome,
            lives:map.lives,
            ground:map.ground,
            road:map.road,
            path:map.path
          };
          customMapIds.push(map.id);
        }
        renderCustomMaps();
      }

      function renderCustomMaps() {
        ui.customMaps.replaceChildren();
        for (const id of customMapIds) {
          const map=maps[id];
          if (!map) continue;
          const button=document.createElement('button');
          button.type='button';
          button.className='custom-map-chip';
          button.textContent='Eigen kaart: '+map.name;
          button.addEventListener('click',()=>{
            activeMission=null;
            startMap(id);
          });
          ui.customMaps.append(button);
        }
      }

      function drawEditorRoute() {
        ui.editorRoute.setAttribute('points',editorPoints.map(point=>point.x+','+point.y).join(' '));
        ui.editorPoints.replaceChildren();
        editorPoints.slice(1,-1).forEach((point,index)=>{
          const marker=document.createElementNS('http://www.w3.org/2000/svg','circle');
          marker.setAttribute('cx',String(point.x));
          marker.setAttribute('cy',String(point.y));
          marker.setAttribute('r','13');
          marker.setAttribute('class','editor-point');
          marker.setAttribute('aria-label','Bocht '+(index+1));
          ui.editorPoints.append(marker);
        });
        const colors={
          meadow:'#527e4c',forest:'#557b55',swamp:'#49685a',
          desert:'#b78d58',highlands:'#64725e'
        };
        ui.editorPreview.querySelector('rect').setAttribute('fill',colors[ui.editorBiome.value]);
      }

      function saveCustomMap() {
        const name=ui.editorName.value.trim().slice(0,28);
        const lives=Number(ui.editorLives.value);
        if (name.length<3) {
          ui.editorMessage.textContent='Gebruik een kaartnaam van minstens drie tekens.';
          return;
        }
        if (!Number.isInteger(lives)||lives<5||lives>40) {
          ui.editorMessage.textContent='Kies tussen 5 en 40 basislevens.';
          return;
        }
        if (editorPoints.length<3) {
          ui.editorMessage.textContent='Voeg minstens één bocht toe aan de route.';
          return;
        }
        if (editorPoints.some((point,index)=>index>0&&
            Math.hypot(point.x-editorPoints[index-1].x,point.y-editorPoints[index-1].y)<42)) {
          ui.editorMessage.textContent='Twee routepunten liggen te dicht bij elkaar.';
          return;
        }

        const id='custom-'+Date.now().toString(36)+'-'+
          Math.random().toString(36).slice(2,7);
        if (customMapIds.length>=20) {
          ui.editorMessage.textContent='Je kunt maximaal 20 eigen kaarten bewaren.';
          return;
        }
        const map={
          id,name,biome:ui.editorBiome.value,lives,
          ground:({
            meadow:'#527e4c',forest:'#557b55',swamp:'#49685a',
            desert:'#b78d58',highlands:'#64725e'
          })[ui.editorBiome.value],
          road:({
            meadow:'#b49b70',forest:'#9e9273',swamp:'#8d8468',
            desert:'#d8c58a',highlands:'#b09b78'
          })[ui.editorBiome.value],
          path:editorPoints.map(point=>({x:Math.round(point.x),y:Math.round(point.y)}))
        };
        const saved=customMapIds.map(mapId=>({
          ...maps[mapId],
          id:mapId,
          name:maps[mapId].name,
          path:maps[mapId].path
        })).concat(map);
        try {
          localStorage.setItem(customMapStorageKey,JSON.stringify(saved));
        } catch (error) {
          console.error('Eigen kaart kon niet worden opgeslagen.',error);
          ui.editorMessage.textContent='Opslaan is mislukt. Controleer de browseropslag.';
          return;
        }
        maps[id]={name:name.toUpperCase(),biome:map.biome,lives,ground:map.ground,road:map.road,path:map.path};
        customMapIds.push(id);
        renderCustomMaps();
        ui.editorMessage.textContent='Kaart bewaard. Start hem via Vrij spelen.';
      }

      function recordKey(mapName, selectedDifficulty) {
        return mapName.toLowerCase() + ':' + selectedDifficulty;
      }

      function renderRecords() {
        ui.bestRecords.replaceChildren();
        const entries = Object.entries(maps).map(([id,map]) => {
          const best = Number(bestRecords[recordKey(id,difficulty)] || 0);
          return {name:map.name,best};
        }).filter(record => record.best > 0);

        if (!entries.length) {
          const hint = document.createElement('span');
          hint.className = 'record-chip';
          hint.textContent = 'Records verschijnen zodra je een golf overleeft.';
          ui.bestRecords.append(hint);
          return;
        }

        for (const record of entries.sort((a,b) => b.best-a.best)) {
          const chip = document.createElement('span');
          const label = document.createElement('strong');
          chip.className = 'record-chip';
          label.textContent = 'Golf ' + record.best;
          chip.append(record.name + ' · ',label);
          ui.bestRecords.append(chip);
        }
      }

      function saveRecord(completedWave=wave-1) {
        if (!currentMap) return;
        const id = Object.keys(maps).find(key => maps[key] === currentMap);
        if (!id) return;
        const key = recordKey(id,difficulty);
        const previous = Number(bestRecords[key] || 0);
        if (completedWave <= previous) return;

        bestRecords[key] = completedWave;
        try {
          localStorage.setItem(recordStorageKey,JSON.stringify(bestRecords));
        } catch (error) {
          console.warn('Blockfront-record kon niet worden opgeslagen.', error);
        }
        renderRecords();
        addFeedMessage('Nieuw record: golf ' + completedWave,'reward');
      }

      function addFeedMessage(message,kind='') {
        const entry = document.createElement('div');
        entry.className = 'feed-message' + (kind ? ' ' + kind : '');
        entry.textContent = message;
        ui.combatFeed.append(entry);
        while (ui.combatFeed.children.length > 4) {
          ui.combatFeed.firstElementChild.remove();
        }
        setTimeout(() => entry.remove(),3600);
      }

      function flashScreen() {
        ui.screenFlash.classList.remove('active');
        void ui.screenFlash.offsetWidth;
        ui.screenFlash.classList.add('active');
      }

      function resetWeatherParticles() {
        weatherParticles = [];
        if (weather !== 'rain' && weather !== 'snow') return;

        const density = quality === 'high' ? 180 : quality === 'medium' ? 100 : 48;
        for (let i=0;i<density;i++) {
          weatherParticles.push({
            x:Math.random()*W,
            y:Math.random()*H,
            height:.5+Math.random()*4,
            drift:(Math.random()-.5)*35,
            size:.6+Math.random()*1.5
          });
        }
      }

      function weatherLabel(value) {
        return {
          clear:'helder weer',
          rain:'regen',
          snow:'sneeuw',
          mist:'ochtendmist'
        }[value] || 'helder weer';
      }

      function setWeather(value) {
        weather=value;
        document.getElementById('game-ui').dataset.weather=weather;
        resetWeatherParticles();
        if (currentMap) {
          const mapId=Object.keys(maps).find(key=>maps[key]===currentMap);
          if (mapId) uploadMapGeometry(currentMap);
        }
        addFeedMessage('Weerbeeld gewijzigd: '+weatherLabel(weather));
      }

      function setQuality(value) {
        quality=value;
        resetWeatherParticles();
        if (currentMap) uploadMapGeometry(currentMap);
      }

      function resize() {
        const dpr = Math.min(window.devicePixelRatio || 1, 2);
        canvas.width = Math.floor(innerWidth * dpr);
        canvas.height = Math.floor(innerHeight * dpr);
        canvas.style.width = innerWidth + 'px';
        canvas.style.height = innerHeight + 'px';
        gl.viewport(0,0,canvas.width,canvas.height);
        updateCameraMatrix();
      }

      function say(text, color = '#fff') {
        ui.toast.textContent = text;
        ui.toast.style.color = color;
        clearTimeout(say.timer);
        say.timer = setTimeout(() => ui.toast.textContent = '', 1800);
      }

      function updateUI() {
        ui.cash.textContent = '$' + cash;
        ui.lives.textContent = lives;
        ui.wave.textContent = wave;
        ui.start.textContent = waveActive ? 'Wave bezig…' : 'Start wave ' + wave;
        ui.start.disabled = waveActive || gameOver || paused;
        ui.pause.textContent = paused ? 'Hervat' : 'Pauze';
        ui.pause.disabled = !currentMap || gameOver;
        ui.pause.setAttribute('aria-pressed',String(paused));
        ui.difficultyName.textContent=difficultySettings[difficulty].name.toUpperCase();
        if (activeMission) ui.mapName.textContent=activeMission.title;
        ui.speed.textContent='Snelheid '+gameSpeed+'×';
        ui.waveProgress.style.width=(waveActive&&spawnTotal
          ?Math.max(0,Math.min(1,(spawned-enemies.length)/spawnTotal))*100
          :0)+'%';
        ui.waveProgressLabel.textContent=waveActive
          ?Math.max(0,spawnTotal-spawned)+enemies.length+' vijanden over'
          :wavePreviewLabel();
        ui.targetMode.disabled=!selectedTower;
        ui.targetMode.textContent='Doel: '+(selectedTower
          ?targetingLabels[selectedTower.targetMode]||targetingLabels.leading
          :'Voorste');
        updateAbilityUI();

        document.querySelectorAll('.tower-button').forEach(button => {
          button.classList.toggle('active', button.dataset.type === buildType);
        });

        if (!selectedTower) {
          ui.towerPanel.style.display = 'none';
          return;
        }

        const upgradePrice = upgradeCost(selectedTower);
        ui.towerPanel.style.display = 'block';
        ui.selectedName.textContent =
          types[selectedTower.type].name + ' · Level ' + selectedTower.level;
        ui.selectedInfo.textContent =
          'Schade ' + Math.round(selectedTower.damage) +
          ' · Bereik ' + Math.round(selectedTower.range) +
          ' · Upgrade $' + upgradePrice;
        ui.upgrade.disabled = cash < upgradePrice || selectedTower.level >= 5;
      }

      function updateAbilityUI() {
        ui.airstrike.classList.toggle('ready',airstrikeCooldown<=0&&!airstrikeTargeting);
        ui.airstrike.textContent=airstrikeTargeting
          ?'Kies doel · Esc annuleert'
          :airstrikeCooldown>0
            ?'Luchtsteun '+airstrikeCooldown.toFixed(1)+'s'
            :'Luchtsteun $300';
        ui.airstrike.disabled=paused||gameOver||!currentMap||
          (!airstrikeTargeting&&(airstrikeCooldown>0||cash<300));
        ui.airstrike.setAttribute('aria-pressed',String(airstrikeTargeting));
      }

      function upgradeCost(tower) {
        return Math.floor(tower.cost * .65 + tower.level * 125);
      }

      function wavePreview() {
        const total=7+wave*3+(wave%5===0?4:0);
        const counts={normal:0,runner:0,tank:0,medic:0,boss:0};
        for (let index=0;index<total;index++) {
          let kind='normal';
          if (wave%5===0&&index===0) kind='boss';
          else if (wave>=2&&index%3===2) kind='runner';
          if (wave%5!==0||index!==0) {
            if (wave>=3&&index%4===3) kind='tank';
            if (wave>=5&&index%6===4) kind='medic';
          }
          counts[kind]++;
        }
        return {total,counts};
      }

      function wavePreviewLabel() {
        const preview=wavePreview();
        const boss=preview.counts.boss?' · baas':'';
        const special=preview.counts.medic?' · '+preview.counts.medic+' veldarts':'';
        return (preview.counts.boss?'Baasgolf ':'Golf ')+wave+': '+
          preview.total+' vijanden'+special+boss;
      }

      function calculatePathLength() {
        pathLength = 0;
        for (let i=0; i<path.length-1; i++) {
          pathLength += Math.hypot(path[i+1].x-path[i].x, path[i+1].y-path[i].y);
        }
      }

      function pointAt(distance) {
        let remaining = distance;
        for (let i=0; i<path.length-1; i++) {
          const a=path[i], b=path[i+1];
          const length=Math.hypot(b.x-a.x,b.y-a.y);
          if (remaining <= length) {
            const t=length ? remaining/length : 0;
            return {x:a.x+(b.x-a.x)*t,y:a.y+(b.y-a.y)*t};
          }
          remaining-=length;
        }
        return {...path[path.length-1]};
      }

      function distanceToPath(x,y) {
        let best=Infinity;
        for (let i=0;i<path.length-1;i++) {
          const a=path[i],b=path[i+1];
          const dx=b.x-a.x,dy=b.y-a.y;
          const len=dx*dx+dy*dy;
          const t=len?Math.max(0,Math.min(1,((x-a.x)*dx+(y-a.y)*dy)/len)):0;
          best=Math.min(best,Math.hypot(x-(a.x+dx*t),y-(a.y+dy*t)));
        }
        return best;
      }

      function startMap(mapId) {
        currentMap=maps[mapId];
        if (!currentMap) return;
        path=currentMap.path;
        weather=activeMission?.weather||ui.weather.value;
        ui.weather.value=weather;
        quality=ui.quality.value;
        calculatePathLength();
        uploadMapGeometry(currentMap);

        enemies=[];
        towers=[];
        shots=[];
        effects=[];
        beams=[];
        airstrikeCooldown=0;
        airstrikeTargeting=false;
        gameSpeed=1;
        const rules=difficultySettings[difficulty];
        cash=activeMission?activeMission.startingCash:rules.startingCash;
        lives=Math.ceil((activeMission?activeMission.lives:currentMap.lives)*rules.livesMultiplier);
        waveStartLives=lives;
        wave=1;
        waveActive=false;
        paused=false;
        gameOver=false;
        buildType='scout';
        selectedTower=null;
        spawned=0;
        spawnTotal=0;
        spawnTimer=0;

        ui.mapName.textContent=activeMission?activeMission.title:currentMap.name;
        document.getElementById('game-ui').dataset.weather=weather;
        document.getElementById('game-ui').dataset.quality=quality;
        document.getElementById('game-ui').dataset.airstrike='';
        ui.pauseIndicator.style.display='none';
        ui.overlay.style.display='none';
        ui.overlayTitle.textContent='Base vernietigd!';
        ui.nextMission.style.display='none';
        document.getElementById('menu').style.display='none';
        document.getElementById('game-ui').style.display='block';
        ui.combatFeed.replaceChildren();
        resetWeatherParticles();
        resize();
        updateUI();
        addFeedMessage('Weer: '+weatherLabel(weather));
        say('Kies een toren en klik op een leeg vak');
      }

      function openMenu() {
        paused=true;
        airstrikeTargeting=false;
        document.getElementById('game-ui').dataset.airstrike='';
        ui.pauseIndicator.style.display='none';
        document.getElementById('menu').style.display='grid';
        document.getElementById('game-ui').style.display='none';
        ui.overlay.style.display='none';
        renderRecords();
      }

      function togglePause() {
        if (!currentMap || gameOver) return;
        paused=!paused;
        if (paused&&airstrikeTargeting) {
          airstrikeTargeting=false;
          document.getElementById('game-ui').dataset.airstrike='';
        }
        ui.pauseIndicator.style.display=paused?'grid':'none';
        updateUI();
        say(paused?'Spel gepauzeerd':'Spel hervat',paused?'#ffd166':'#84e58b');
      }

      function canPlace(x,y) {
        if (x<25 || x>W-25 || y<25 || y>H-25) return false;
        if (distanceToPath(x,y)<48) return false;
        return !towers.some(t=>Math.hypot(t.x-x,t.y-y)<38);
      }

      function createTower(type,x,y) {
        const cfg=types[type];
        towers.push({
          type,x,y,level:1,
          damage:cfg.damage,range:cfg.range,rate:cfg.rate,
          speed:cfg.speed,splash:cfg.splash,slow:cfg.slow,
          chain:cfg.chain,cost:cfg.cost,cooldown:.2,
          targetMode:'leading',
          aimAngle:-Math.PI/2,
          sellValue:Math.floor(cfg.cost*.7)
        });
      }

      function placeTower(x,y) {
        const cfg=types[buildType];
        if (cash<cfg.cost) return say('Niet genoeg cash','#ff7681');
        if (!canPlace(x,y)) return say('Hier kun je geen toren plaatsen','#ff7681');

        cash-=cfg.cost;
        createTower(buildType,x,y);
        selectedTower=null;
        updateUI();
        say(cfg.name+' geplaatst','#84e58b');
      }

      function createEnemy() {
        const boss=wave%5===0 && spawned===0;
        let kind='normal';
        if (!boss && wave>=2 && spawned%3===2) kind='runner';
        if (!boss && wave>=3 && spawned%4===3) kind='tank';
        if (!boss && wave>=5 && spawned%6===4) kind='medic';

        const multiplier=boss?3:kind==='tank'?2.1:kind==='medic'?1.25:1;
        const speed=kind==='runner'?115:kind==='tank'?48:kind==='medic'?62:70;
        const hp=(38+wave*15)*multiplier;
        const start=pointAt(0);

        enemies.push({
          x:start.x,y:start.y,progress:0,
          speed:(speed+wave*2.5)*difficultySettings[difficulty].enemySpeedMultiplier*
            (activeMission?activeMission.enemySpeedMultiplier:1),
          hp:hp*difficultySettings[difficulty].enemyHealthMultiplier*
            (activeMission?activeMission.enemyHealthMultiplier:1),
          maxHp:hp*difficultySettings[difficulty].enemyHealthMultiplier*
            (activeMission?activeMission.enemyHealthMultiplier:1),
          reward:Math.round((10+wave*2+(boss?40:kind==='tank'?10:0))*
            (activeMission?activeMission.enemyRewardMultiplier:1)),
          slow:0,boss,kind,alive:true,healTimer:1
        });
        if (boss) {
          flashScreen();
          addFeedMessage('Baas nadert de basis!','boss');
        } else if (kind==='medic') {
          addFeedMessage('Vijandelijke veldarts gesignaleerd');
        }
      }

      function startWave() {
        if (waveActive || paused || gameOver) return;
        waveActive=true;
        waveStartLives=lives;
        spawned=0;
        spawnTotal=7+wave*3+(wave%5===0?4:0)+(activeMission?activeMission.extraEnemies:0);
        spawnTimer=.2*(activeMission?activeMission.spawnIntervalMultiplier:1);
        say(wave%5===0?'Baasgolf '+wave+'!':'Golf '+wave+' gestart','#ffd166');
        addFeedMessage(wave%5===0?'Baasgolf '+wave+' gestart':'Golf '+wave+' gestart',
          wave%5===0?'boss':'');
        updateUI();
      }

      function removeEnemy(enemy) {
        if (!enemy.alive) return;
        enemy.alive=false;
        const index=enemies.indexOf(enemy);
        if (index>=0) enemies.splice(index,1);
        cash+=enemy.reward;
      }

      function damageEnemy(enemy,damage,splash=0,slow=0,chain=0) {
        if (!enemy || !enemy.alive) return;

        enemy.hp-=damage;
        if (slow>0) enemy.slow=Math.max(enemy.slow,slow);

        if (splash>0) {
          for (const other of [...enemies]) {
            if (other!==enemy && other.alive &&
                Math.hypot(other.x-enemy.x,other.y-enemy.y)<splash) {
              other.hp-=damage*.45;
              if (other.hp<=0) removeEnemy(other);
            }
          }
        }

        if (chain>0) {
          const nearby=enemies
            .filter(other=>other!==enemy && other.alive &&
              Math.hypot(other.x-enemy.x,other.y-enemy.y)<85)
            .sort((a,b)=>
              Math.hypot(a.x-enemy.x,a.y-enemy.y)-
              Math.hypot(b.x-enemy.x,b.y-enemy.y))
            .slice(0,chain);

          for (const other of nearby) {
            other.hp-=damage*.65;
            beams.push({x1:enemy.x,y1:enemy.y,target:other,color:types.tesla.color,life:.16,width:3});
            if (other.hp<=0) removeEnemy(other);
          }
        }

        if (enemy.hp<=0) removeEnemy(enemy);
        updateUI();
      }

      function deployAirstrike(x,y) {
        if (!airstrikeTargeting) return;
        if (x<0||x>W||y<0||y>H) {
          say('Kies een plek op het slagveld','#ff7681');
          return;
        }

        const radius=108;
        const targets=enemies.filter(enemy=>Math.hypot(enemy.x-x,enemy.y-y)<=radius);
        if (!targets.length) {
          airstrikeTargeting=false;
          document.getElementById('game-ui').dataset.airstrike='';
          updateAbilityUI();
          say('Geen vijanden binnen bereik','#ff7681');
          return;
        }

        cash-=300;
        airstrikeCooldown=38;
        airstrikeTargeting=false;
        document.getElementById('game-ui').dataset.airstrike='';
        const damage=(115+wave*9)*
          (activeMission?activeMission.airstrikeDamageMultiplier:1);
        for (const enemy of targets) damageEnemy(enemy,damage);
        effects.push({x,y,radius:28,life:.65,maxLife:.65,color:'#ffb45c'});
        effects.push({x,y,radius:48,life:.85,maxLife:.85,color:'#f27b48'});
        effects.push({x,y,radius:68,life:1,maxLife:1,color:'#f5d277'});
        flashScreen();
        addFeedMessage('Luchtsteun raakt '+targets.length+' vijand'+
          (targets.length===1?'':'en'),'reward');
        updateUI();
      }

      function toggleAirstrike() {
        if (airstrikeTargeting) {
          airstrikeTargeting=false;
          document.getElementById('game-ui').dataset.airstrike='';
          updateAbilityUI();
          say('Luchtsteun geannuleerd');
          return;
        }
        if (cash<300) return say('Luchtsteun kost $300','#ff7681');
        if (airstrikeCooldown>0) return say('Luchtsteun herlaadt nog','#ff7681');
        if (!currentMap||paused||gameOver) return;
        airstrikeTargeting=true;
        document.getElementById('game-ui').dataset.airstrike='targeting';
        updateAbilityUI();
        say('Klik op de kaart om de luchtsteun te richten','#ffd166');
      }

      function shoot(tower,target) {
        const cfg=types[tower.type];
        tower.aimAngle=Math.atan2(target.y-tower.y,target.x-tower.x);

        if (tower.type==='laser') {
          damageEnemy(target,tower.damage);
          beams.push({
            x1:tower.x,y1:tower.y-12,target,
            color:cfg.color,life:.13,width:4
          });
          effects.push({x:target.x,y:target.y,radius:5,life:.2,maxLife:.2,color:cfg.color});
          return;
        }

        shots.push({
          x:tower.x,y:tower.y-12,target,
          speed:tower.speed,damage:tower.damage,
          splash:tower.splash,slow:tower.slow,chain:tower.chain,
          color:cfg.color,radius:cfg.radius,type:tower.type
        });
      }

      function finishWave() {
        waveActive=false;
        cash+=Math.round((15+wave*6)*
          (activeMission?activeMission.enemyRewardMultiplier:1));
        const clearedWave=wave;
        if (lives===waveStartLives) {
          const perfectBonus=Math.round((25+wave*3)*
            (activeMission?activeMission.perfectBonusMultiplier:1));
          cash+=perfectBonus;
          addFeedMessage('Schone verdediging · bonus $'+perfectBonus,'reward');
        }
        saveRecord(wave);
        if (activeMission&&wave>=activeMission.waveGoal) {
          completeMission(activeMission);
          updateUI();
          return;
        }
        wave++;
        updateUI();
        say('Golf '+clearedWave+' verslagen!','#84e58b');
      }

      function createShaderProgram() {
        const vertexSource=`
          attribute vec3 a_position;
          attribute vec3 a_normal;
          attribute vec3 a_color;
          uniform mat4 u_viewProjection;
          varying vec3 v_normal;
          varying vec3 v_color;
          varying float v_distance;
          varying vec3 v_worldPosition;
          void main() {
            vec4 worldPosition=vec4(a_position,1.0);
            gl_Position=u_viewProjection*worldPosition;
            v_normal=a_normal;
            v_color=a_color;
            v_distance=gl_Position.w;
            v_worldPosition=a_position;
          }
        `;
        const fragmentSource=`
          precision mediump float;
          uniform vec3 u_lightDirection;
          uniform vec3 u_cameraPosition;
          uniform vec3 u_fogColor;
          uniform float u_fogAmount;
          varying vec3 v_normal;
          varying vec3 v_color;
          varying float v_distance;
          varying vec3 v_worldPosition;
          void main() {
            vec3 normal=normalize(v_normal);
            vec3 light=normalize(u_lightDirection);
            vec3 viewDirection=normalize(u_cameraPosition-v_worldPosition);
            vec3 halfVector=normalize(light+viewDirection);
            float diffuse=max(dot(normal,light),0.0);
            float backLight=max(dot(normal,normalize(vec3(-light.x,.35,-light.z))),0.0);
            float specular=pow(max(dot(normal,halfVector),0.0),28.0)*0.16;
            float hemisphere=clamp(normal.y*.5+.5,0.0,1.0);
            float illumination=.34+hemisphere*.12+diffuse*.62+backLight*.08;
            vec3 color=v_color*illumination;
            color+=vec3(1.0,.91,.76)*specular;
            float rim=pow(1.0-max(dot(normal,viewDirection),0.0),3.0);
            color+=vec3(.22,.38,.34)*rim*.12;
            float contact=1.0-smoothstep(-.15,1.0,v_worldPosition.y);
            color*=.94+contact*.06;
            float fog=smoothstep(30.0,58.0,v_distance)*u_fogAmount;
            color=mix(color,u_fogColor,fog);
            float dither=fract(sin(dot(gl_FragCoord.xy,vec2(12.9898,78.233)))*43758.5453);
            color+=(dither-.5)/255.0;
            gl_FragColor=vec4(color,1.0);
          }
        `;
        const compile=(type,source)=>{
          const shader=gl.createShader(type);
          gl.shaderSource(shader,source);
          gl.compileShader(shader);
          if (!gl.getShaderParameter(shader,gl.COMPILE_STATUS)) {
            const message=gl.getShaderInfoLog(shader);
            gl.deleteShader(shader);
            throw new Error('WebGL shader compile failed: '+message);
          }
          return shader;
        };
        const program=gl.createProgram();
        gl.attachShader(program,compile(gl.VERTEX_SHADER,vertexSource));
        gl.attachShader(program,compile(gl.FRAGMENT_SHADER,fragmentSource));
        gl.linkProgram(program);
        if (!gl.getProgramParameter(program,gl.LINK_STATUS)) {
          const message=gl.getProgramInfoLog(program);
          gl.deleteProgram(program);
          throw new Error('WebGL program link failed: '+message);
        }
        return program;
      }

      function multiplyMatrix(a,b) {
        const result=new Float32Array(16);
        for (let column=0;column<4;column++) {
          for (let row=0;row<4;row++) {
            result[column*4+row]=
              a[row]*b[column*4]+
              a[4+row]*b[column*4+1]+
              a[8+row]*b[column*4+2]+
              a[12+row]*b[column*4+3];
          }
        }
        return result;
      }

      function perspectiveMatrix(fieldOfView,aspect,near,far) {
        const f=1/Math.tan(fieldOfView/2);
        const range=1/(near-far);
        return new Float32Array([
          f/aspect,0,0,0,
          0,f,0,0,
          0,0,(near+far)*range,-1,
          0,0,2*near*far*range,0
        ]);
      }

      function lookAtMatrix(eye,target,up) {
        const normalize=value=>{
          const length=Math.hypot(value[0],value[1],value[2])||1;
          return value.map(component=>component/length);
        };
        const cross=(a,b)=>[
          a[1]*b[2]-a[2]*b[1],
          a[2]*b[0]-a[0]*b[2],
          a[0]*b[1]-a[1]*b[0]
        ];
        const dot=(a,b)=>a[0]*b[0]+a[1]*b[1]+a[2]*b[2];
        const z=normalize([eye[0]-target[0],eye[1]-target[1],eye[2]-target[2]]);
        const x=normalize(cross(up,z));
        const y=cross(z,x);
        return new Float32Array([
          x[0],y[0],z[0],0,
          x[1],y[1],z[1],0,
          x[2],y[2],z[2],0,
          -dot(x,eye),-dot(y,eye),-dot(z,eye),1
        ]);
      }

      function invertMatrix(matrix) {
        const rows=Array.from({length:4},(_,row)=>[
          matrix[row],matrix[4+row],matrix[8+row],matrix[12+row],
          row===0?1:0,row===1?1:0,row===2?1:0,row===3?1:0
        ]);
        for (let column=0;column<4;column++) {
          let pivot=column;
          for (let row=column+1;row<4;row++) {
            if (Math.abs(rows[row][column])>Math.abs(rows[pivot][column])) pivot=row;
          }
          if (Math.abs(rows[pivot][column])<1e-10) return null;
          [rows[column],rows[pivot]]=[rows[pivot],rows[column]];
          const divisor=rows[column][column];
          for (let i=0;i<8;i++) rows[column][i]/=divisor;
          for (let row=0;row<4;row++) {
            if (row===column) continue;
            const factor=rows[row][column];
            for (let i=0;i<8;i++) rows[row][i]-=factor*rows[column][i];
          }
        }
        const inverse=new Float32Array(16);
        for (let row=0;row<4;row++) {
          for (let column=0;column<4;column++) {
            inverse[column*4+row]=rows[row][column+4];
          }
        }
        return inverse;
      }

      function transformPoint(matrix,x,y,z,w=1) {
        const result=[
          matrix[0]*x+matrix[4]*y+matrix[8]*z+matrix[12]*w,
          matrix[1]*x+matrix[5]*y+matrix[9]*z+matrix[13]*w,
          matrix[2]*x+matrix[6]*y+matrix[10]*z+matrix[14]*w,
          matrix[3]*x+matrix[7]*y+matrix[11]*z+matrix[15]*w
        ];
        const divisor=result[3]||1;
        return result.slice(0,3).map(component=>component/divisor);
      }

      function updateCameraMatrix() {
        const aspect=canvas.width/Math.max(1,canvas.height);
        const projection=perspectiveMatrix(42*Math.PI/180,aspect,.1,100);
        const view=lookAtMatrix([0,20,28],[0,0,0],[0,1,0]);
        viewProjectionMatrix=multiplyMatrix(projection,view);
      }

      const colorCache=new Map();
      function colorVector(color) {
        if (Array.isArray(color)) return color;
        if (!colorCache.has(color)) {
          colorCache.set(color,[
            parseInt(color.slice(1,3),16)/255,
            parseInt(color.slice(3,5),16)/255,
            parseInt(color.slice(5,7),16)/255
          ]);
        }
        return colorCache.get(color);
      }

      function addTriangle(mesh,a,b,c,color) {
        const ab=[b[0]-a[0],b[1]-a[1],b[2]-a[2]];
        const ac=[c[0]-a[0],c[1]-a[1],c[2]-a[2]];
        let normal=[
          ab[1]*ac[2]-ab[2]*ac[1],
          ab[2]*ac[0]-ab[0]*ac[2],
          ab[0]*ac[1]-ab[1]*ac[0]
        ];
        const length=Math.hypot(normal[0],normal[1],normal[2]);
        normal=length<1e-8?[0,1,0]:normal.map(component=>component/length);
        const rgb=colorVector(color);
        for (const point of [a,b,c]) {
          mesh.push(point[0],point[1],point[2],normal[0],normal[1],normal[2],rgb[0],rgb[1],rgb[2]);
        }
      }

      function addQuad(mesh,a,b,c,d,color) {
        addTriangle(mesh,a,b,c,color);
        addTriangle(mesh,a,c,d,color);
      }

      function addBox(mesh,x,y,z,width,height,depth,color,yaw=0) {
        const c=Math.cos(yaw),s=Math.sin(yaw);
        const point=(px,py,pz)=>[
          x+px*c-pz*s,
          y+py,
          z+px*s+pz*c
        ];
        const corners=[
          point(-width/2,-height/2,-depth/2),point(width/2,-height/2,-depth/2),
          point(width/2,height/2,-depth/2),point(-width/2,height/2,-depth/2),
          point(-width/2,-height/2,depth/2),point(width/2,-height/2,depth/2),
          point(width/2,height/2,depth/2),point(-width/2,height/2,depth/2)
        ];
        for (const face of [[4,5,6,7],[1,0,3,2],[0,4,7,3],[5,1,2,6],[3,7,6,2],[0,1,5,4]]) {
          addQuad(mesh,corners[face[0]],corners[face[1]],corners[face[2]],corners[face[3]],color);
        }
      }

      function addSphere(mesh,x,y,z,rx,ry,rz,color,segments=8,rings=5) {
        for (let ring=0;ring<rings;ring++) {
          const top=Math.PI*ring/rings;
          const bottom=Math.PI*(ring+1)/rings;
          for (let segment=0;segment<segments;segment++) {
            const start=Math.PI*2*segment/segments;
            const end=Math.PI*2*(segment+1)/segments;
            const point=(latitude,longitude)=>[
              x+Math.sin(latitude)*Math.cos(longitude)*rx,
              y+Math.cos(latitude)*ry,
              z+Math.sin(latitude)*Math.sin(longitude)*rz
            ];
            addQuad(mesh,point(top,start),point(top,end),point(bottom,end),point(bottom,start),color);
          }
        }
      }

      function addCylinder(mesh,x,y,z,radius,height,color,segments=8) {
        const bottom=y-height/2;
        const top=y+height/2;
        for (let i=0;i<segments;i++) {
          const a=Math.PI*2*i/segments;
          const b=Math.PI*2*(i+1)/segments;
          const p1=[x+Math.cos(a)*radius,bottom,z+Math.sin(a)*radius];
          const p2=[x+Math.cos(b)*radius,bottom,z+Math.sin(b)*radius];
          const p3=[x+Math.cos(b)*radius,top,z+Math.sin(b)*radius];
          const p4=[x+Math.cos(a)*radius,top,z+Math.sin(a)*radius];
          addQuad(mesh,p1,p4,p3,p2,color);
          addTriangle(mesh,[x,bottom,z],p1,p2,color);
          addTriangle(mesh,[x,top,z],p3,p4,color);
        }
      }

      function addRing(mesh,x,y,z,radius,color,segments=48,thickness=.035) {
        for (let i=0;i<segments;i++) {
          const a=Math.PI*2*i/segments;
          const b=Math.PI*2*(i+1)/segments;
          const point=(angle,r)=>[x+Math.cos(angle)*r,y,z+Math.sin(angle)*r];
          addQuad(mesh,point(a,radius-thickness),point(b,radius-thickness),
            point(b,radius+thickness),point(a,radius+thickness),color);
        }
      }

      function gameToWorld(x,y,height=0) {
        return {
          x:(x/W-.5)*WORLD_WIDTH,
          y:height,
          z:(y/H-.5)*WORLD_DEPTH
        };
      }

      function buildMapGeometry(map) {
        const mesh=[];
        const detailFactor=quality==='high'?1:quality==='medium'?.68:.38;
        let seed=map.name.length*73856093;
        const random=()=>{
          seed=(seed*1664525+1013904223)>>>0;
          return seed/4294967296;
        };
        const ground=colorVector(map.ground);
        const groundColor=[
          ground[0]*.97,
          ground[1]*.97,
          ground[2]*.97
        ];
        addQuad(mesh,[-WORLD_WIDTH/2,0,-WORLD_DEPTH/2],[-WORLD_WIDTH/2,0,WORLD_DEPTH/2],
          [WORLD_WIDTH/2,0,WORLD_DEPTH/2],[WORLD_WIDTH/2,0,-WORLD_DEPTH/2],groundColor);

        for (let i=0;i<Math.round((map.biome==='desert'?18:24)*detailFactor);i++) {
          const x=18+random()*(W-36);
          const y=18+random()*(H-36);
          if (distanceToPath(x,y)<58) continue;
          const point=gameToWorld(x,y,.018);
          const rx=.55+random()*.85;
          const rz=.35+random()*.7;
          const patch=map.biome==='desert'
            ?(i%2?'#ad8457':'#c0a171')
            :map.biome==='swamp'
              ?(i%2?'#526d5a':'#3b5a4d')
              :(i%2?'#4d7545':'#66804a');
          addSphere(mesh,point.x,point.y,point.z,rx,.012,rz,patch,12,3);
        }

        const grassColors=map.biome==='swamp'
          ?['#405d50','#52705c','#62795d','#374d46']
          :['#477143','#557d45','#66834a','#3e653e'];
        const terrainMarks=Math.round(
          (map.biome==='desert'?280:map.biome==='swamp'?330:map.biome==='highlands'?520:850)*
          detailFactor
        );
        for (let i=0;i<terrainMarks;i++) {
          const x=18+random()*(W-36);
          const y=18+random()*(H-36);
          if (distanceToPath(x,y)<54) continue;
          const point=gameToWorld(x,y,.012+random()*.01);
          const width=.025+random()*.035;
          const depth=.03+random()*.07;
          const lean=(random()-.5)*.11;
          addTriangle(mesh,
            [point.x-width,point.y,point.z],
            [point.x+width,point.y,point.z+depth],
            [point.x+lean,point.y+.12+random()*.13,point.z+depth*.35],
            grassColors[Math.floor(random()*grassColors.length)]);
        }

        if (map.biome==='swamp') {
          for (let i=0;i<26;i++) {
            const x=24+random()*(W-48);
            const y=24+random()*(H-48);
            if (distanceToPath(x,y)<70) continue;
            const point=gameToWorld(x,y,.04);
            addSphere(mesh,point.x,point.y,point.z,.25+random()*.4,.035,.16+random()*.19,
              i%2?'#314e49':'#3e5b50',8,4);
            if (i%2===0) {
              for (let reed=0;reed<3;reed++) {
                addBox(mesh,point.x+(reed-1)*.12,.28,point.z+(reed%2)*.08,
                  .035,.5+random()*.25,.035,'#84905e', (random()-.5)*.2);
              }
            }
          }
        } else if (map.biome==='desert') {
          for (let i=0;i<Math.round(28*detailFactor);i++) {
            const x=22+random()*(W-44);
            const y=22+random()*(H-44);
            if (distanceToPath(x,y)<65) continue;
            const point=gameToWorld(x,y,.12);
            addSphere(mesh,point.x,point.y,point.z,.18+random()*.38,.12+random()*.22,
              .16+random()*.3,i%2?'#9b7952':'#c09a64',7,4);
            if (i%5===0) {
              addCylinder(mesh,point.x,point.y+.45,point.z,.11,.9,'#557649',7);
              addBox(mesh,point.x+.16,point.y+.48,point.z,.36,.1,.1,'#557649',-.2);
              addBox(mesh,point.x+.29,point.y+.6,point.z,.1,.32,.1,'#557649',-.2);
              addBox(mesh,point.x-.13,point.y+.31,point.z,.3,.1,.1,'#557649',.18);
              addBox(mesh,point.x-.26,point.y+.43,point.z,.1,.28,.1,'#557649',.18);
            }
          }
        } else if (map.biome==='highlands') {
          const rockColors=['#68736a','#788276','#59645d','#899083'];
          for (let i=0;i<Math.round(36*detailFactor);i++) {
            const x=22+random()*(W-44);
            const y=22+random()*(H-44);
            if (distanceToPath(x,y)<76) continue;
            const point=gameToWorld(x,y,.12);
            const width=.22+random()*.42;
            const depth=.24+random()*.45;
            const height=.18+random()*.42;
            addBox(mesh,point.x,point.y+height/2,point.z,width,height,depth,
              rockColors[Math.floor(random()*rockColors.length)],(random()-.5)*.8);
            if (i%3===0) {
              addSphere(mesh,point.x+width*.24,point.y+height*.92,point.z-depth*.12,
                width*.28,.14,depth*.28,'#87917b',7,4);
            }
          }

          for (let i=0;i<Math.round(22*detailFactor);i++) {
            const x=22+random()*(W-44);
            const y=22+random()*(H-44);
            if (distanceToPath(x,y)<66) continue;
            const point=gameToWorld(x,y,.035);
            const height=.42+random()*.48;
            addCylinder(mesh,point.x,height*.38,point.z,.09,.75,'#584b3c',7);
            addSphere(mesh,point.x,height,point.z,.38,.48,.34,
              i%2?'#576d53':'#718067',8,5);
            addSphere(mesh,point.x-.18,height+.12,point.z+.08,.25,.3,.25,
              i%2?'#6c7d61':'#809073',7,4);
          }
        } else {
          for (let i=0;i<Math.round(28*detailFactor);i++) {
            const x=20+random()*(W-40);
            const y=20+random()*(H-40);
            if (distanceToPath(x,y)<68) continue;
            const point=gameToWorld(x,y);
            const size=.52+random()*.35;
            addSphere(mesh,point.x,.055,point.z,size,.045,size*.72,'#263b2b',8,4);
            addCylinder(mesh,point.x,.42,point.z,.14,.76,
              i%2?'#624a35':'#76583c',7);
            const foliage=i%3===0?'#416c3d':i%3===1?'#507947':'#365f39';
            addSphere(mesh,point.x,1.05,point.z,size*.82,.72,size*.78,foliage,9,6);
            addSphere(mesh,point.x-.3,1.18,point.z-.12,size*.57,.53,size*.58,
              i%2?'#527d45':'#47723f',8,5);
            addSphere(mesh,point.x+.29,.98,point.z+.14,size*.55,.48,size*.54,
              i%2?'#355f39':'#46723e',8,5);
          }
        }

        const worldPath=map.path.map(point=>gameToWorld(point.x,point.y,.045));
        for (let i=0;i<worldPath.length-1;i++) {
          const a=worldPath[i],b=worldPath[i+1];
          const dx=b.x-a.x,dz=b.z-a.z;
          const length=Math.hypot(dx,dz)||1;
          const nx=-dz/length,nz=dx/length;
          const road=(width,y,color)=>{
            const half=width/2;
            addQuad(mesh,
              [a.x+nx*half,y,a.z+nz*half],
              [b.x+nx*half,y,b.z+nz*half],
              [b.x-nx*half,y,b.z-nz*half],
              [a.x-nx*half,y,a.z-nz*half],
              color);
          };
          road(2.05,.035,'#373832');
          road(1.72,.052,'#554c3c');
          road(1.5,.068,map.road);
          road(1.12,.072,map.biome==='desert'?'#d8bd8b':'#b09a72');
        }
        for (let i=1;i<worldPath.length-1;i++) {
          const point=worldPath[i];
          addCylinder(mesh,point.x,.065,point.z,.76,.035,map.road,12);
        }

        const roadLength=pathLength;
        for (let distance=12;distance<roadLength;distance+=21) {
          const point=pointAt(distance);
          const ahead=pointAt(Math.min(roadLength,distance+2));
          const angle=Math.atan2(ahead.y-point.y,ahead.x-point.x);
          const side=(random()-.5)*34;
          const px=point.x-Math.sin(angle)*side;
          const py=point.y+Math.cos(angle)*side;
          const stone=gameToWorld(px,py,.084);
          addSphere(mesh,stone.x,stone.y,stone.z,.035+random()*.045,.018,
            .025+random()*.035,random()>.5?'#75664e':'#c4ab7f',6,3);
          if (distance%42<21) {
            const rutAngle=Math.atan2(ahead.y-point.y,ahead.x-point.x);
            for (const rutSide of [-.28,.28]) {
              const rx=point.x-Math.sin(rutAngle)*rutSide*30;
              const ry=point.y+Math.cos(rutAngle)*rutSide*30;
              const rut=gameToWorld(rx,ry,.087);
              addSphere(mesh,rut.x,rut.y,rut.z,.10,.008,.025,
                random()>.5?'#8d7a5a':'#a18a64',6,3);
            }
          }
        }

        const start=worldPath[0];
        const end=worldPath[worldPath.length-1];
        addCylinder(mesh,end.x,.18,end.z,1.02,.3,'#27383b',10);
        addCylinder(mesh,end.x,.38,end.z,.82,.18,'#6c7c76',10);
        addBox(mesh,end.x,.55,end.z,1.18,.18,.92,'#38484a');
        addBox(mesh,end.x,.72,end.z,.92,.32,.68,'#56727a');
        addSphere(mesh,end.x,.96,end.z,.28,.16,.28,'#bdd5cd',8,5);
        addCylinder(mesh,end.x,1.18,end.z,.09,.42,'#1e292b',8);
        addSphere(mesh,end.x,1.42,end.z,.13,.13,.13,'#d8be69',8,5);
        addCylinder(mesh,start.x,.12,start.z,.63,.24,'#5d3434',10);
        addCylinder(mesh,start.x,.28,start.z,.48,.12,'#8a4e46',10);
        addSphere(mesh,start.x,.48,start.z,.34,.25,.34,'#9c5a4d',8,5);
        addBox(mesh,start.x,.68,start.z,.08,.52,.08,'#5b4e3d');
        addBox(mesh,start.x-.05,.9,start.z,.38,.24,.025,'#b7473f');
        return mesh;
      }

      function uploadMapGeometry(map) {
        const mesh=buildMapGeometry(map);
        gl.bindBuffer(gl.ARRAY_BUFFER,staticBuffer);
        gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(mesh),gl.STATIC_DRAW);
        staticVertexCount=mesh.length/9;
      }

      function addUnit(mesh,enemy) {
        const current=gameToWorld(enemy.x,enemy.y);
        const next=pointAt(Math.min(pathLength,enemy.progress+5));
        const nextWorld=gameToWorld(next.x,next.y);
        const yaw=Math.atan2(nextWorld.z-current.z,nextWorld.x-current.x);
        const scale=enemy.boss?1.32:enemy.kind==='tank'?1.12:enemy.kind==='runner'?.9:1;
        const gait=enemy.progress*.12+visualTime*(enemy.kind==='runner'?10:7);
        const color=enemy.slow>0?'#5798a5':
          enemy.boss?'#642c35':
          enemy.kind==='runner'?'#a87c3d':
          enemy.kind==='tank'?'#4b5c42':
          enemy.kind==='medic'?'#47755e':'#784044';
        const highlight=enemy.slow>0?'#94d9df':
          enemy.boss?'#a55758':
          enemy.kind==='runner'?'#d2ad68':
          enemy.kind==='tank'?'#74815d':
          enemy.kind==='medic'?'#83c497':'#b36561';
        const stride=Math.sin(gait)*.12*scale;
        const bob=Math.abs(Math.sin(gait))*.025*scale;
        addSphere(mesh,current.x,.035,current.z,.46*scale,.025,.34*scale,'#242927',10,4);
        addSphere(mesh,current.x-.12*scale,.055,current.z-.04*scale,.18*scale,.018,.12*scale,'#59604f',7,3);
        addSphere(mesh,current.x+.18*scale,.055,current.z+.06*scale,.16*scale,.018,.11*scale,'#59604f',7,3);

        for (const side of [-1,1]) {
          addBox(mesh,current.x-.04+stride*side,(.22+bob)*scale,current.z+side*.11*scale,
            .15*scale,.34*scale,.14*scale,'#343b3c',yaw);
          addBox(mesh,current.x+.02+stride*side,(.075+bob)*scale,current.z+side*.11*scale,
            .24*scale,.1*scale,.18*scale,'#282e30',yaw);
          addBox(mesh,current.x-.04*scale,.47*scale,current.z+side*.24*scale,
            .14*scale,.35*scale,.14*scale,'#3c4644',yaw);
        }

        addBox(mesh,current.x-.2*scale,.57*scale,current.z,.2*scale,.34*scale,.32*scale,
          enemy.boss?'#512832':'#3f443d',yaw);
        addBox(mesh,current.x,.55*scale,current.z,.42*scale,.5*scale,.38*scale,color,yaw);
        addBox(mesh,current.x+.055*scale,.58*scale,current.z,.31*scale,.28*scale,.395*scale,
          highlight,yaw);
        addBox(mesh,current.x+.08*scale,.51*scale,current.z,.2*scale,.08*scale,.405*scale,
          '#363d3b',yaw);

        for (const side of [-1,1]) {
          addSphere(mesh,current.x-.06*scale,.76*scale,current.z+side*.23*scale,
            .16*scale,.15*scale,.15*scale,highlight,7,4);
          addBox(mesh,current.x+.15*scale,.48*scale,current.z+side*.27*scale,
            .27*scale,.13*scale,.13*scale,'#343a39',yaw);
          addBox(mesh,current.x+.28*scale,.49*scale,current.z+side*.3*scale,
            .11*scale,.1*scale,.12*scale,highlight,yaw);
        }
        addSphere(mesh,current.x+.18*scale,(.91+bob)*scale,current.z,
          .17*scale,.19*scale,.17*scale,'#987b61',8,5);
        if (enemy.kind==='medic') {
          addBox(mesh,current.x-.22*scale,.91*scale,current.z,
            .08*scale,.32*scale,.07*scale,'#d7f0cb',yaw);
          addBox(mesh,current.x-.22*scale,.91*scale,current.z,
            .28*scale,.08*scale,.07*scale,'#d7f0cb',yaw);
        }
        addSphere(mesh,current.x+.19*scale,1.03*scale,current.z,
          .2*scale,.11*scale,.2*scale,enemy.boss?'#3e4140':'#485047',8,4);
        addBox(mesh,current.x+.34*scale,.9*scale,current.z,
          .08*scale,.08*scale,.23*scale,'#283336',yaw);
        addBox(mesh,current.x+.38*scale,.62*scale,current.z,
          .36*scale,.075*scale,.07*scale,'#222b2e',yaw);
        addBox(mesh,current.x+.47*scale,.68*scale,current.z,
          .08*scale,.14*scale,.08*scale,'#1d2526',yaw);
        addCylinder(mesh,current.x+.5*scale,.69*scale,current.z,.025,.32,'#151a1b',6);

        if (enemy.boss) {
          addBox(mesh,current.x-.3*scale,.56*scale,current.z,.12*scale,.62*scale,.4*scale,
            '#672d39',yaw);
          addSphere(mesh,current.x,.83*scale,current.z+.22*scale,.045,.045,.045,'#e1b45f',6,3);
        }

        const ratio=Math.max(0,Math.min(1,enemy.hp/enemy.maxHp));
        const barY=1.38*scale;
        addBox(mesh,current.x,barY,current.z,.82*scale,.085*scale,.045,'#202727');
        if (ratio>0) {
          const barColor=ratio>.5?'#6ba85a':ratio>.25?'#c3a34d':'#ad4642';
          addBox(mesh,current.x-(1-ratio)*.39*scale,barY+.003,current.z+.026,
            .76*ratio*scale,.04*scale,.025,barColor);
        }
      }

      function buildDynamicGeometry() {
        const mesh=[];
        for (const tower of towers) {
          const point=gameToWorld(tower.x,tower.y);
          const aim=tower.aimAngle;
          const range=tower.range/W*WORLD_WIDTH;
          if (tower===selectedTower) {
            const pulse=.5+.5*Math.sin(visualTime*3);
            addRing(mesh,point.x,.035,point.z,range,'#d7e5d3',48,.02+pulse*.012);
          }
          addSphere(mesh,point.x,.035,point.z,.82,.025,.72,'#202725',10,3);
          addSphere(mesh,point.x-.22,.055,point.z-.2,.16,.018,.12,'#555e58',7,3);
          addCylinder(mesh,point.x,.12,point.z,.68,.18,'#2d3535',12);
          addCylinder(mesh,point.x,.22,point.z,.58,.16,'#565f5b',12);
          addCylinder(mesh,point.x,.34,point.z,.49,.16,'#3a4442',12);
          addBox(mesh,point.x,.43,point.z,.78,.24,.72,'#4d5754');
          addCylinder(mesh,point.x,.58,point.z,.40,.24,'#303a3b',12);
          addCylinder(mesh,point.x,.68,point.z,.31,.08,types[tower.type].color,12);
          const barrelLength=tower.type==='sniper'?.8:
            tower.type==='cannon'||tower.type==='mortar'?.62:
            tower.type==='laser'?.73:.53;
          const barrelWidth=tower.type==='cannon'||tower.type==='mortar'?.18:
            tower.type==='sniper'?.075:.11;
          addBox(mesh,point.x+Math.cos(aim)*barrelLength*.5,.62,
            point.z+Math.sin(aim)*barrelLength*.5,barrelLength,.12,barrelWidth,
            types[tower.type].color,aim);
          addSphere(mesh,point.x,.65,point.z,.28,.2,.29,types[tower.type].color,8,5);
          const muzzleX=point.x+Math.cos(aim)*barrelLength;
          const muzzleZ=point.z+Math.sin(aim)*barrelLength;
          addCylinder(mesh,muzzleX,.62,muzzleZ,
            barrelWidth*.72,.16,'#20292a',10);
          if (tower.cooldown>tower.rate-.09) {
            const flash=.08+.045*(.5+.5*Math.sin(visualTime*45+tower.x+tower.y));
            addSphere(mesh,muzzleX,.62,muzzleZ,flash,flash,flash,
              tower.type==='laser'?'#f4d9ff':'#ffe0a0',7,4);
          }
          for (const side of [-1,1]) {
            addBox(mesh,point.x+Math.cos(aim)*.28,
              .43+side*.09,point.z+Math.sin(aim)*.28,
              .12,.07,.12,'#77817c',aim);
          }
          if (tower.type==='frost'||tower.type==='tesla'||tower.type==='laser') {
            const pulse=.5+.5*Math.sin(visualTime*4+tower.x*.08+tower.y*.05);
            const coreSize=.09+pulse*.045;
            addSphere(mesh,point.x,.96,point.z,
              coreSize,.24+pulse*.06,coreSize,types[tower.type].color,8,5);
            addRing(mesh,point.x,.96,point.z,.23+pulse*.035,
              types[tower.type].color,16,.018);
          }
          if (tower.type==='sniper') {
            addBox(mesh,point.x+Math.cos(aim)*.12,.79,
              point.z+Math.sin(aim)*.12,.2,.08,.18,'#d7bc72',aim);
            addSphere(mesh,point.x+Math.cos(aim)*.12,.84,
              point.z+Math.sin(aim)*.12,.045,.045,.045,'#fff0b0',6,3);
          } else if (tower.type==='cannon'||tower.type==='mortar') {
            for (const offset of [.18,.48]) {
              addBox(mesh,point.x+Math.cos(aim)*offset,.62,
                point.z+Math.sin(aim)*offset,.055,.17,barrelWidth*1.35,
                '#89918a',aim);
            }
          } else if (tower.type==='rapid') {
            for (const side of [-1,1]) {
              addBox(mesh,point.x+Math.cos(aim)*.2,.57,
                point.z+Math.sin(aim)*.2+side*.12,.34,.075,.055,
                '#f29ab8',aim);
            }
          } else if (tower.type==='scout') {
            addCylinder(mesh,point.x,.88,point.z,.055,.32,'#a8c9d4',8);
            addSphere(mesh,point.x,.99,point.z,.075,.055,.075,'#d7efff',7,4);
          }
          if (tower.level>1) {
            addCylinder(mesh,point.x,.83,point.z,.19,.04,'#e7c56e',10);
          }
        }

        for (const enemy of enemies) addUnit(mesh,enemy);
        for (const shot of shots) {
          const point=gameToWorld(shot.x,shot.y,.38);
          addSphere(mesh,point.x,point.y,point.z,.13,.13,.13,shot.color,8,5);
          addSphere(mesh,point.x,point.y-.06,point.z,.08,.08,.08,'#e8e7c6',7,4);
        }
        for (const effect of effects) {
          const point=gameToWorld(effect.x,effect.y,.045);
          addRing(mesh,point.x,point.y,point.z,
            effect.radius/W*WORLD_WIDTH,effect.color,28,.025);
          addSphere(mesh,point.x,effect.radius/W*WORLD_WIDTH*.12+point.y,
            point.z,effect.radius/W*WORLD_WIDTH*.55,
            effect.radius/W*WORLD_WIDTH*.18,effect.radius/W*WORLD_WIDTH*.55,
            effect.life/effect.maxLife>.45?'#9da09a':'#5e625e',10,5);
        }
        for (const beam of beams) {
          if (!beam.target.alive) continue;
          const target=gameToWorld(beam.target.x,beam.target.y,.6);
          const source=gameToWorld(beam.x1,beam.y1,.72);
          const dx=target.x-source.x,dz=target.z-source.z;
          const length=Math.hypot(dx,dz);
          addBox(mesh,(source.x+target.x)/2,(source.y+target.y)/2,(source.z+target.z)/2,
            length,.035,.035,beam.color,Math.atan2(dz,dx));
        }

        if (mouse.inside&&buildType&&!airstrikeTargeting) {
          const position=snappedMouse();
          const point=gameToWorld(position.x,position.y,.04);
          const valid=canPlace(position.x,position.y)&&cash>=types[buildType].cost;
          const range=types[buildType].range/W*WORLD_WIDTH;
          addRing(mesh,point.x,.04,point.z,range,valid?'#74d781':'#e76767',56,.025);
          addCylinder(mesh,point.x,.22,point.z,.38,.32,
            valid?'#4f9560':'#a74646',8);
        }
        if (mouse.inside&&airstrikeTargeting) {
          const point=gameToWorld(mouse.x,mouse.y,.055);
          addRing(mesh,point.x,.055,point.z,108/W*WORLD_WIDTH,'#ffb45c',56,.045);
          addCylinder(mesh,point.x,.16,point.z,.17,.22,'#f27b48',10);
          addSphere(mesh,point.x,.34,point.z,.2,.08,.2,'#ffd166',8,4);
        }
        for (const particle of weatherParticles) {
          const point=gameToWorld(particle.x,particle.y,particle.height);
          if (weather==='rain') {
            addBox(mesh,point.x,point.y,point.z,.025,.22,.025,
              '#b4d7e8',-.38);
          } else {
            addSphere(mesh,point.x,point.y,point.z,
              .035*particle.size,.035*particle.size,.035*particle.size,
              '#e5f1f0',6,4);
          }
        }
        return mesh;
      }

      function bindGeometryBuffer(buffer,data,usage) {
        gl.bindBuffer(gl.ARRAY_BUFFER,buffer);
        if (data) gl.bufferData(gl.ARRAY_BUFFER,data,usage);
        const stride=9*Float32Array.BYTES_PER_ELEMENT;
        gl.enableVertexAttribArray(shaderLocations.position);
        gl.vertexAttribPointer(shaderLocations.position,3,gl.FLOAT,false,stride,0);
        gl.enableVertexAttribArray(shaderLocations.normal);
        gl.vertexAttribPointer(shaderLocations.normal,3,gl.FLOAT,false,stride,3*Float32Array.BYTES_PER_ELEMENT);
        gl.enableVertexAttribArray(shaderLocations.color);
        gl.vertexAttribPointer(shaderLocations.color,3,gl.FLOAT,false,stride,6*Float32Array.BYTES_PER_ELEMENT);
      }

      function screenToGame(clientX,clientY) {
        const rect=canvas.getBoundingClientRect();
        const x=(clientX-rect.left)/rect.width*2-1;
        const y=1-(clientY-rect.top)/rect.height*2;
        const inverse=invertMatrix(viewProjectionMatrix);
        if (!inverse) return {x:-CELL,y:-CELL};
        const near=transformPoint(inverse,x,y,-1);
        const far=transformPoint(inverse,x,y,1);
        const denominator=far[1]-near[1];
        if (Math.abs(denominator)<1e-8) return {x:-CELL,y:-CELL};
        const distance=-near[1]/denominator;
        if (distance<0) return {x:-CELL,y:-CELL};
        const worldX=near[0]+(far[0]-near[0])*distance;
        const worldZ=near[2]+(far[2]-near[2])*distance;
        return {
          x:(worldX/WORLD_WIDTH+.5)*W,
          y:(worldZ/WORLD_DEPTH+.5)*H
        };
      }

      function update(dt) {
        airstrikeCooldown=Math.max(0,airstrikeCooldown-dt);
        for (const particle of weatherParticles) {
          if (weather==='rain') {
            particle.x+=particle.drift*dt;
            particle.y+=330*dt;
            particle.height-=5*dt;
            if (particle.y>H+8||particle.height<.2) {
              particle.x=Math.random()*W;
              particle.y=-Math.random()*H*.35;
              particle.height=3+Math.random()*3;
            }
          } else {
            particle.x+=particle.drift*dt+Math.sin(particle.y*.025)*8*dt;
            particle.y+=45*dt;
            particle.height=Math.max(.2,particle.height-.22*dt);
            if (particle.y>H+8) {
              particle.x=Math.random()*W;
              particle.y=-8;
              particle.height=1+Math.random()*3;
            }
          }
          if (particle.x<0) particle.x=W;
          if (particle.x>W) particle.x=0;
        }

        if (waveActive) {
          spawnTimer-=dt;
          if (spawned<spawnTotal && spawnTimer<=0) {
            createEnemy();
            spawned++;
            const spawnInterval=(.72-wave*.025)*
              (activeMission?activeMission.spawnIntervalMultiplier:1);
            spawnTimer=Math.max(.24,spawnInterval);
            updateUI();
          }
        }

        for (let i=enemies.length-1;i>=0;i--) {
          const enemy=enemies[i];
          enemy.slow=Math.max(0,enemy.slow-dt);
          enemy.progress+=enemy.speed*(enemy.slow>0?.55:1)*dt;
          if (enemy.kind==='medic') {
            enemy.healTimer-=dt;
            if (enemy.healTimer<=0) {
              const patient=enemies
                .filter(other=>other!==enemy&&other.alive&&other.hp<other.maxHp&&
                  Math.hypot(other.x-enemy.x,other.y-enemy.y)<115)
                .sort((a,b)=>a.hp/a.maxHp-b.hp/b.maxHp)[0];
              if (patient) {
                patient.hp=Math.min(patient.maxHp,patient.hp+patient.maxHp*.09);
                beams.push({
                  x1:enemy.x,y1:enemy.y,target:patient,
                  color:'#83edaa',life:.25,width:2
                });
              }
              enemy.healTimer=1.2;
            }
          }

          const p=pointAt(enemy.progress);
          enemy.x=p.x;
          enemy.y=p.y;

          if (enemy.progress>=pathLength) {
            enemy.alive=false;
            enemies.splice(i,1);
            lives--;

            if (lives<=0) {
              lives=0;
              gameOver=true;
              airstrikeTargeting=false;
              document.getElementById('game-ui').dataset.airstrike='';
              saveRecord();
              flashScreen();
              addFeedMessage('De basis is gevallen','boss');
              ui.finalScore.textContent='Je bereikte wave '+wave+' op '+currentMap.name+'.';
              ui.overlay.style.display='grid';
            }
            updateUI();
          }
        }

        for (const tower of towers) {
          tower.cooldown-=dt;
          if (tower.cooldown>0) continue;

          let target=null;
          for (const enemy of enemies) {
            if (Math.hypot(enemy.x-tower.x,enemy.y-tower.y)>tower.range) continue;
            if (!target) {
              target=enemy;
              continue;
            }
            if (tower.targetMode==='strongest'&&enemy.hp>target.hp) target=enemy;
            else if (tower.targetMode==='nearest'&&
                Math.hypot(enemy.x-tower.x,enemy.y-tower.y)<
                Math.hypot(target.x-tower.x,target.y-tower.y)) target=enemy;
            else if (tower.targetMode==='leading'&&enemy.progress>target.progress) target=enemy;
          }

          if (target) {
            shoot(tower,target);
            tower.cooldown=tower.rate;
          }
        }

        for (let i=shots.length-1;i>=0;i--) {
          const shot=shots[i];
          if (!shot.target.alive || !enemies.includes(shot.target)) {
            shots.splice(i,1);
            continue;
          }

          const dx=shot.target.x-shot.x;
          const dy=shot.target.y-shot.y;
          const distance=Math.hypot(dx,dy);

          if (distance<8) {
            damageEnemy(shot.target,shot.damage,shot.splash,shot.slow,shot.chain);
            effects.push({
              x:shot.target.x,y:shot.target.y,
              radius:shot.type==='mortar'?12:6,
              life:.3,maxLife:.3,color:shot.color
            });
            shots.splice(i,1);
          } else {
            shot.x+=dx/distance*shot.speed*dt;
            shot.y+=dy/distance*shot.speed*dt;
          }
        }

        for (let i=effects.length-1;i>=0;i--) {
          effects[i].life-=dt;
          effects[i].radius+=90*dt;
          if (effects[i].life<=0) effects.splice(i,1);
        }

        for (let i=beams.length-1;i>=0;i--) {
          beams[i].life-=dt;
          if (beams[i].life<=0 || !beams[i].target.alive) beams.splice(i,1);
        }

        if (waveActive && spawned>=spawnTotal && enemies.length===0) finishWave();
        updateAbilityUI();
      }

      function snappedMouse() {
        return {
          x:Math.floor(mouse.x/CELL)*CELL+CELL/2,
          y:Math.floor(mouse.y/CELL)*CELL+CELL/2
        };
      }

      function draw() {
        const baseClear=currentMap ? {
          desert:[.58,.66,.66],
          swamp:[.31,.42,.43],
          forest:[.39,.52,.53],
          highlands:[.48,.57,.6],
          meadow:[.48,.61,.65]
        }[currentMap.biome] : [.48,.61,.65];
        const weatherTint={
          clear:[0,0,0],
          rain:[-.1,-.07,-.04],
          snow:[.11,.12,.13],
          mist:[.12,.13,.12]
        }[weather];
        const clear=baseClear.map((channel,index)=>
          Math.max(0,Math.min(1,channel+weatherTint[index])));
        gl.clearColor(clear[0],clear[1],clear[2],1);
        gl.clear(gl.COLOR_BUFFER_BIT|gl.DEPTH_BUFFER_BIT);
        gl.useProgram(shaderProgram);
        gl.uniformMatrix4fv(shaderLocations.matrix,false,viewProjectionMatrix);
        const lighting=weather==='rain'?[-.3,.72,.56]:
          weather==='snow'?[-.38,.9,.22]:[-.45,.82,.36];
        const fogColor=weather==='rain'?[.38,.48,.53]:
          weather==='snow'?[.72,.79,.81]:
          weather==='mist'?[.7,.76,.73]:
          currentMap?.biome==='desert'?[.67,.63,.54]:
          currentMap?.biome==='swamp'?[.35,.47,.45]:
          currentMap?.biome==='highlands'?[.5,.57,.56]:[.56,.61,.59];
        const fogAmount=weather==='mist'?.78:weather==='rain'?.44:
          weather==='snow'?.36:.32;
        gl.uniform3f(shaderLocations.light,lighting[0],lighting[1],lighting[2]);
        gl.uniform3f(shaderLocations.camera,0,20,28);
        gl.uniform3f(shaderLocations.fog,fogColor[0],fogColor[1],fogColor[2]);
        gl.uniform1f(shaderLocations.fogAmount,fogAmount);
        if (staticVertexCount) {
          bindGeometryBuffer(staticBuffer,null,gl.STATIC_DRAW);
          gl.drawArrays(gl.TRIANGLES,0,staticVertexCount);
        }
        const mesh=buildDynamicGeometry();
        if (mesh.length) {
          bindGeometryBuffer(dynamicBuffer,new Float32Array(mesh),gl.DYNAMIC_DRAW);
          gl.drawArrays(gl.TRIANGLES,0,mesh.length/9);
        }
      }

      function canvasPoint(event) {
        return screenToGame(event.clientX,event.clientY);
      }

      canvas.addEventListener('pointermove',event=>{
        const p=canvasPoint(event);
        mouse={x:p.x,y:p.y,inside:p.x>=0&&p.x<=W&&p.y>=0&&p.y<=H};
      });

      canvas.addEventListener('pointerleave',()=>mouse.inside=false);

      canvas.addEventListener('pointerdown',event=>{
        if (gameOver) return;
        const p=canvasPoint(event);
        if (airstrikeTargeting) {
          deployAirstrike(p.x,p.y);
          return;
        }
        const tower=[...towers].reverse().find(t=>Math.hypot(t.x-p.x,t.y-p.y)<24);

        if (tower) {
          selectedTower=tower;
          updateUI();
          return;
        }

        placeTower(
          Math.floor(p.x/CELL)*CELL+CELL/2,
          Math.floor(p.y/CELL)*CELL+CELL/2
        );
      });

      document.querySelectorAll('.map-card').forEach(card=>{
        card.addEventListener('click',()=>{
          activeMission=null;
          startMap(card.dataset.map);
        });
      });

      document.querySelectorAll('.menu-tab').forEach(button=>{
        button.addEventListener('click',()=>showMenuView(button.dataset.view));
      });

      document.querySelectorAll('.codex-filter').forEach(button=>{
        button.addEventListener('click',()=>renderCodex(button.dataset.codex));
      });

      ui.editorPreview.addEventListener('pointerdown',event=>{
        const bounds=ui.editorPreview.getBoundingClientRect();
        const point={
          x:Math.max(20,Math.min(W-20,(event.clientX-bounds.left)/bounds.width*W)),
          y:Math.max(20,Math.min(H-20,(event.clientY-bounds.top)/bounds.height*H))
        };
        editorPoints.splice(editorPoints.length-1,0,point);
        drawEditorRoute();
        ui.editorMessage.textContent='Routepunten: '+(editorPoints.length-2)+
          ' · gebruik Bocht terug om een punt te verwijderen.';
      });

      document.getElementById('editor-undo').addEventListener('click',()=>{
        if (editorPoints.length>2) editorPoints.splice(editorPoints.length-2,1);
        drawEditorRoute();
        ui.editorMessage.textContent='Routepunten: '+(editorPoints.length-2);
      });

      document.getElementById('editor-reset').addEventListener('click',()=>{
        editorPoints=[{x:0,y:320},{x:960,y:320}];
        drawEditorRoute();
        ui.editorMessage.textContent='Route teruggezet naar een rechte lijn.';
      });

      document.getElementById('editor-save').addEventListener('click',saveCustomMap);
      ui.editorBiome.addEventListener('change',drawEditorRoute);
      document.getElementById('next-mission').addEventListener('click',()=>{
        if (activeMission&&activeMission.id<campaignMissions.length) {
          startMission(activeMission.id+1);
        }
      });

      ui.difficulty.addEventListener('change',()=>{
        difficulty=ui.difficulty.value;
        ui.difficultyDescription.textContent=difficultySettings[difficulty].description;
        ui.difficultyName.textContent=difficultySettings[difficulty].name.toUpperCase();
        drawEditorRoute();
        renderCampaign();
        renderCodex();
        renderRecords();
      });

      ui.weather.addEventListener('change',()=>setWeather(ui.weather.value));
      ui.quality.addEventListener('change',()=>{
        document.getElementById('game-ui').dataset.quality=ui.quality.value;
        setQuality(ui.quality.value);
      });

      document.querySelectorAll('.tower-button').forEach(button=>{
        button.addEventListener('click',()=>{
          buildType=button.dataset.type;
          selectedTower=null;
          updateUI();
          say(types[buildType].name+' geselecteerd','#ffd166');
        });
      });

      ui.start.addEventListener('click',startWave);
      ui.pause.addEventListener('click',togglePause);
      ui.speed.addEventListener('click',()=>{
        const speeds=[1,1.5,2];
        gameSpeed=speeds[(speeds.indexOf(gameSpeed)+1)%speeds.length];
        updateUI();
      });
      ui.airstrike.addEventListener('click',toggleAirstrike);
      ui.targetMode.addEventListener('click',()=>{
        if (!selectedTower) return;
        const index=targetingModes.indexOf(selectedTower.targetMode);
        selectedTower.targetMode=targetingModes[(index+1)%targetingModes.length];
        updateUI();
        say('Doelprioriteit: '+targetingLabels[selectedTower.targetMode]);
      });

      ui.upgrade.addEventListener('click',()=>{
        if (!selectedTower || selectedTower.level>=5) return;
        const cost=upgradeCost(selectedTower);
        if (cash<cost) return say('Niet genoeg cash','#ff7681');

        cash-=cost;
        selectedTower.level++;
        selectedTower.damage*=1.45;
        selectedTower.range*=1.08;
        selectedTower.rate*=.9;
        selectedTower.sellValue+=Math.floor(cost*.45);
        updateUI();
        say('Toren geupgrade','#84e58b');
      });

      ui.sell.addEventListener('click',()=>{
        if (!selectedTower) return;
        cash+=selectedTower.sellValue;
        towers.splice(towers.indexOf(selectedTower),1);
        selectedTower=null;
        updateUI();
        say('Toren verkocht','#84e58b');
      });

      document.getElementById('back-menu').addEventListener('click',openMenu);
      document.getElementById('overlay-menu').addEventListener('click',openMenu);
      document.getElementById('restart-map').addEventListener('click',()=>{
        const mapId=Object.keys(maps).find(key=>maps[key]===currentMap);
        if (mapId) startMap(mapId);
      });

      window.addEventListener('keydown',event=>{
        const inGame=document.getElementById('game-ui').style.display==='block';
        if (inGame&&event.code==='Space') {
          event.preventDefault();
          startWave();
        }
        if (inGame&&event.code==='KeyP') togglePause();
        if (inGame&&event.code==='KeyA') toggleAirstrike();
        if (event.code==='Escape') {
          if (airstrikeTargeting) toggleAirstrike();
          else if (inGame) openMenu();
        }
        if (inGame&&event.code==='Digit1') gameSpeed=1;
        if (inGame&&event.code==='Digit2') gameSpeed=1.5;
        if (inGame&&event.code==='Digit3') gameSpeed=2;
        if (inGame&&['Digit1','Digit2','Digit3'].includes(event.code)) updateUI();
      });

      window.addEventListener('resize',resize);

      loadCustomMaps();
      renderRecords();
      function frame(time) {
        const dt=Math.min((time-lastTime)/1000||0,.04)*gameSpeed;
        lastTime=time;
        visualTime=time*.001;
        if (currentMap && !gameOver && !paused) update(dt);
        if (currentMap) draw();
        requestAnimationFrame(frame);
      }

      resize();
      requestAnimationFrame(frame);
    })();
  </script>
</body>
</html>

