<!DOCTYPE html>
<html lang="zh-HK">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>香港生物的士 3D - LEGO World Edition</title>

  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>

  <!-- Three.js CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

  <style>
    @import url('https://fonts.googleapis.com/css2?family=Fredoka+One&display=swap');

    body, html { margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden; font-family: 'Fredoka One', 'PingFang HK', 'Microsoft JhengHei', cursive, sans-serif; background: #38bdf8; user-select: none; }
    #webgl-container { width: 100%; height: 100%; position: absolute; top: 0; left: 0; z-index: 1; }
    
    .lego-title { font-family: 'Fredoka One', cursive; text-shadow: 3px 3px 0px #ef4444, -3px -3px 0px #facc15; }
    .hud-font { font-family: 'Fredoka One', monospace; text-shadow: 2px 2px 0px #000; }
    .lego-panel { background: rgba(255, 255, 255, 0.92); border: 4px solid #ef4444; border-radius: 20px; box-shadow: 0 10px 0px #b91c1c; color: #0f172a; }
    .digital-meter { font-family: 'Courier New', monospace; background: #000000; color: #facc15; text-shadow: 0 0 10px #facc15; border: 3px solid #ef4444; }

    .touch-btn { touch-action: manipulation; background: rgba(255, 255, 255, 0.4); border: 2px solid #ffffff; backdrop-filter: blur(8px); border-radius: 16px; box-shadow: 0 4px 0 rgba(0,0,0,0.2); }
    .touch-btn:active { background: rgba(255, 255, 255, 0.8); transform: translateY(2px); }
  </style>
</head>
<body class="select-none text-slate-900">

  <!-- 3D WebGL Canvas Container -->
  <div id="webgl-container"></div>

  <!-- UI OVERLAY LAYER -->
  <div id="ui-layer" class="relative z-10 w-full h-full pointer-events-none">

    <!-- 1. AUTH & LEADERBOARD MODAL -->
    <div id="auth-modal" class="pointer-events-auto absolute inset-0 bg-sky-950/80 backdrop-blur-md flex justify-center items-center p-4">
      <div class="lego-panel max-w-4xl w-full p-8 flex flex-col md:flex-row gap-8">
        <!-- Auth Form -->
        <div class="flex-1 border-b md:border-b-0 md:border-r border-slate-300 pr-0 md:pr-8 pb-6 md:pb-0">
          <div class="flex items-center gap-3 mb-2">
            <span class="bg-red-600 text-yellow-300 px-3 py-1 font-black rounded-lg text-xl italic shadow-md">樂高的士 LEGO</span>
            <span class="text-xs text-red-600 font-mono tracking-widest uppercase font-bold">100% LEGO WORLD</span>
          </div>
          <h1 class="text-3xl font-black text-slate-900 tracking-tight mb-2 lego-title text-red-600">香港生物科 3D 樂高的士競速</h1>
          <p class="text-slate-600 text-xs mb-6">請輸入學校電郵登入，控制 LEGO 小人登上樂高清潔能源的士，在積木香港街道上切換軌道答題跳表！</p>
          
          <form id="auth-form" onsubmit="handleLogin(event)" class="space-y-4">
            <div>
              <label class="block text-xs font-bold text-slate-700 uppercase mb-1">學校電郵 (School Email)</label>
              <input type="email" id="input-email" required placeholder="student@xxx.edu.hk" class="w-full bg-white border-2 border-slate-300 rounded-xl px-4 py-3 text-slate-900 focus:outline-none focus:border-red-500 text-sm shadow-inner">
              <span id="email-error" class="text-xs text-red-600 font-semibold hidden mt-1">電郵格式需包含 @ 與 .edu.hk</span>
            </div>
            <div>
              <label class="block text-xs font-bold text-slate-700 uppercase mb-1">學生姓名 (Student Name)</label>
              <input type="text" id="input-name" required placeholder="陳大文" class="w-full bg-white border-2 border-slate-300 rounded-xl px-4 py-3 text-slate-900 focus:outline-none focus:border-red-500 text-sm shadow-inner">
            </div>
            <button type="submit" class="w-full bg-gradient-to-r from-red-600 to-amber-500 hover:from-red-500 hover:to-amber-400 text-white font-black py-3.5 rounded-xl shadow-lg transition transform active:scale-95">
              開始 LEGO 駕駛任務 🚀
            </button>
          </form>
        </div>

        <!-- Leaderboard -->
        <div class="flex-1 flex flex-col">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">🏆 樂高車資榮譽榜 (Top 10)</h2>
            <span class="text-xs text-slate-500 font-mono">GAS API</span>
          </div>
          <div class="flex-1 bg-amber-50/80 rounded-2xl p-4 border-2 border-amber-200 overflow-y-auto max-h-64 md:max-h-80 shadow-inner">
            <table class="w-full text-left text-xs">
              <thead class="text-slate-500 border-b border-amber-200">
                <tr>
                  <th class="pb-2">排名</th>
                  <th class="pb-2">學生姓名</th>
                  <th class="pb-2 text-right">車資總額</th>
                </tr>
              </thead>
              <tbody id="leaderboard-body" class="divide-y divide-amber-100">
                <tr><td colspan="3" class="py-6 text-center text-slate-400">載入排行榜數據中...</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- 2. CHAPTER SELECTION MODAL -->
    <div id="chapter-modal" class="pointer-events-auto hidden absolute inset-0 bg-sky-950/80 backdrop-blur-md flex justify-center items-center p-4">
      <div class="lego-panel max-w-md w-full p-8 text-center shadow-2xl">
        <h2 class="text-2xl font-black text-red-600 mb-2 lego-title">選擇生物科單元</h2>
        <p class="text-slate-600 text-xs mb-6">選擇考核主題以展開積木軌道競速</p>
        <div id="chapter-list" class="space-y-3"></div>
      </div>
    </div>

    <!-- 3. ARCADE HUD OVERLAY -->
    <div id="hud-container" class="hidden absolute inset-0 pointer-events-none p-6 flex flex-col justify-between">
      <!-- Top HUD Header -->
      <div class="flex justify-between items-start">
        <div class="flex flex-col gap-2">
          <div class="bg-red-600 border-4 border-yellow-400 rounded-2xl p-3 shadow-2xl backdrop-blur text-white">
            <div class="flex justify-between items-center gap-4 mb-1">
              <span class="text-[10px] text-yellow-300 font-black tracking-widest uppercase">TAXI METER 跳表器</span>
              <span id="turbo-badge" class="hidden text-[9px] bg-yellow-400 text-black px-1.5 py-0.5 rounded font-black animate-pulse">LEGO NITRO</span>
            </div>
            <div id="meter-display" class="digital-meter text-2xl font-bold px-3 py-1 rounded-lg">
              HK$ 29.00
            </div>
          </div>
          <div class="bg-slate-900/90 border-2 border-slate-700 rounded-xl p-2 px-3 backdrop-blur flex items-center justify-between text-xs text-white">
            <span class="text-slate-400">Combo 連答</span>
            <span id="combo-display" class="font-bold text-green-400">0 題</span>
          </div>
        </div>

        <div class="bg-slate-900/90 border-2 border-slate-700 rounded-2xl p-4 text-right backdrop-blur shadow-xl text-white">
          <div class="text-xs text-slate-400 font-mono uppercase">Remaining Time</div>
          <div id="timer-display" class="text-2xl font-mono font-black text-amber-400">03:00</div>
          <div id="phase-badge" class="mt-1 text-xs px-2.5 py-1 bg-blue-600/90 rounded-lg text-white inline-block font-semibold shadow-md">階段一：控制 LEGO 小人走到的士旁</div>
        </div>
      </div>

      <!-- Action Prompts -->
      <div id="action-hint" class="self-center bg-yellow-400 text-red-900 border-4 border-red-600 px-8 py-3 rounded-full font-bold text-sm shadow-xl animate-bounce hidden">
        [ 按 E 鍵 或 點擊上車 ] 登上 LEGO 的士
      </div>

      <!-- Fork Indicator Banner -->
      <div id="fork-indicator" class="self-center bg-blue-600 text-yellow-300 border-4 border-yellow-400 px-6 py-2 rounded-2xl font-bold text-xs shadow-xl hidden animate-pulse">
        🔀 前方分岔路口：按 [A] 走左軌道 / 按 [D] 走右軌道
      </div>

      <!-- Bottom HUD Footer -->
      <div class="flex justify-between items-end">
        <div></div>
        <div class="flex items-end gap-4">
          <!-- Speedometer -->
          <div class="bg-slate-900/90 border-2 border-slate-700 rounded-2xl p-3 text-center backdrop-blur shadow-xl text-white">
            <div id="speed-display" class="hud-font text-3xl text-cyan-400">0</div>
            <div class="text-[9px] text-slate-400 font-mono uppercase">KM/H</div>
          </div>

          <!-- Minimap -->
          <div class="bg-slate-900/90 border-2 border-slate-700 rounded-2xl p-1.5 backdrop-blur shadow-xl">
            <canvas id="minimap" width="90" height="90" class="rounded-xl bg-slate-950"></canvas>
          </div>
        </div>
      </div>

      <!-- Mobile Touch Controls -->
      <div class="pointer-events-auto flex justify-between items-end md:hidden pb-2 px-2">
        <div class="grid grid-cols-3 gap-1.5 w-36 h-36">
          <div></div>
          <button id="btn-up" class="touch-btn text-white font-bold text-xl">▲</button>
          <div></div>
          <button id="btn-left" class="touch-btn text-white font-bold text-xl">◀</button>
          <button id="btn-down" class="touch-btn text-white font-bold text-xl">▼</button>
          <button id="btn-right" class="touch-btn text-white font-bold text-xl">▶</button>
        </div>
        <div class="flex flex-col gap-3">
          <button id="btn-action" class="touch-btn w-16 h-16 rounded-full text-yellow-300 font-bold text-sm flex items-center justify-center border-yellow-400 hidden">上車</button>
          <button id="btn-brake" class="touch-btn w-16 h-16 rounded-full text-red-400 font-bold text-xs flex items-center justify-center border-red-500">煞車</button>
        </div>
      </div>
    </div>

    <!-- 4. QUIZ MODAL -->
    <div id="quiz-modal" class="pointer-events-auto hidden absolute inset-0 bg-sky-950/80 backdrop-blur-xl flex justify-center items-center p-4">
      <div class="lego-panel border-4 border-yellow-400 rounded-3xl max-w-lg w-full p-8 shadow-2xl">
        <div class="flex justify-between items-center mb-4">
          <span id="quiz-chapter-title" class="text-xs bg-red-100 text-red-600 border border-red-300 px-3 py-1 rounded-full font-bold">樂高生物問答跳表</span>
          <span class="text-xs text-slate-500 font-mono">答對即增車資與 Nitro</span>
        </div>
        <h3 id="quiz-question" class="text-lg font-bold text-slate-900 mb-6 leading-relaxed">載入中...</h3>
        <div id="quiz-options" class="space-y-3"></div>
      </div>
    </div>

  </div>

  <script>
    /* =========================================================================
       1. GLOBAL CONFIG & QUESTION BANK
       ========================================================================= */
    const GAS_WEB_APP_URL = "YOUR_GAS_URL_HERE";

    const defaultQuestionBank = {
      "chapter_1": {
        "title": "單元一：細胞與生命基礎",
        "questions": [
          { "id": 1, "question": "下列哪一種細胞器是植物細胞進行光合作用的主要場所？", "options": ["線粒體", "葉綠體", "高爾基體", "細胞核"], "answer": 1, "reward": 25 },
          { "id": 2, "question": "細胞膜的主要脂質成分是什麼？", "options": ["甘油三酯", "磷脂", "膽固醇", "脂肪酸"], "answer": 1, "reward": 30 },
          { "id": 3, "question": "被動運輸（如擴散作用）的主要特徵是什麼？", "options": ["需要消耗 ATP", "逆濃度梯度進行", "順濃度梯度進行", "需要內吞作用"], "answer": 2, "reward": 20 }
        ]
      },
      "chapter_2": {
        "title": "單元二：酶與新陳代謝",
        "questions": [
          { "id": 4, "question": "當極端高溫作用於人體內的酶時，會引致什麼變化？", "options": ["活性無限增加", "變性並失去活性", "轉化為底物", "分子量變大"], "answer": 1, "reward": 35 },
          { "id": 5, "question": "酶在生物化學反應中的主要功能是？", "options": ["提供反應所需的能量", "降低反應所需活化能", "改變化學反應的平衡點", "增加生成物的總產量"], "answer": 1, "reward": 40 }
        ]
      }
    };

    let currentStudent = { email: "", name: "" };
    let selectedChapterKey = "chapter_1";
    let gameScore = 29.00;
    let correctCount = 0;
    let totalAnswered = 0;
    let gameTimer = 180;
    let timerInterval = null, quizInterval = null;
    let gamePhase = 1; 
    let isGamePaused = false;
    let turboEndTime = 0;
    let spinOutEndTime = 0;

    /* =========================================================================
       2. WEB AUDIO SYNTHESIZER ENGINE (GTA THEME SYNTHWAVE BGM)
       ========================================================================= */
    let audioCtx = null;
    let engineOsc = null, engineGain = null;
    let bgmInterval = null;
    let bgmStep = 0;

    function initWebAudio() {
      if (audioCtx) return;
      const AudioContext = window.AudioContext || window.webkitAudioContext;
      audioCtx = new AudioContext();

      engineOsc = audioCtx.createOscillator();
      engineGain = audioCtx.createGain();

      engineOsc.type = 'sawtooth';
      engineOsc.frequency.setValueAtTime(45, audioCtx.currentTime);
      engineGain.gain.setValueAtTime(0.01, audioCtx.currentTime);

      engineOsc.connect(engineGain);
      engineGain.connect(audioCtx.destination);
      engineOsc.start();

      startGTASynthwaveBGM();
    }

    // GTA Theme Synthwave / 80s Retrowave Web Audio BGM Synthesizer
    function startGTASynthwaveBGM() {
      if (bgmInterval) return;

      // 經典 GTA 主題旋律音階 (C, Eb, F, G, Bb Synthwave Lead notes)
      const gtaLeadNotes = [
        261.63, 0, 311.13, 349.23, 392.00, 0, 466.16, 392.00,
        349.23, 311.13, 261.63, 0, 392.00, 0, 349.23, 0
      ];

      // 80s Synthwave 八分音符 Pulse Bass Line (C minor vibe)
      const bassNotes = [
        65.41, 65.41, 77.78, 65.41, 87.31, 65.41, 98.00, 65.41,
        65.41, 65.41, 77.78, 65.41, 116.54, 98.00, 87.31, 77.78
      ];

      bgmInterval = setInterval(() => {
        if (!audioCtx || isGamePaused) return;

        const now = audioCtx.currentTime;
        const step = bgmStep % 16;

        // 1. GTA Lead Synth (Brass/Sawtooth Style Lead)
        const leadFreq = gtaLeadNotes[step];
        if (leadFreq > 0) {
          const leadOsc = audioCtx.createOscillator();
          const leadGain = audioCtx.createGain();
          const leadFilter = audioCtx.createBiquadFilter();

          leadOsc.type = 'sawtooth';
          leadOsc.frequency.setValueAtTime(leadFreq, now);

          leadFilter.type = 'lowpass';
          leadFilter.frequency.setValueAtTime(1600, now);
          leadFilter.frequency.exponentialRampToValueAtTime(600, now + 0.2);

          leadGain.gain.setValueAtTime(0.12, now);
          leadGain.gain.exponentialRampToValueAtTime(0.001, now + 0.22);

          leadOsc.connect(leadFilter);
          leadFilter.connect(leadGain);
          leadGain.connect(audioCtx.destination);

          leadOsc.start(now);
          leadOsc.stop(now + 0.22);
        }

        // 2. 80s Retrowave Bass (Octave Sawtooth Bass)
        const bassFreq = bassNotes[step];
        if (bassFreq > 0) {
          const bassOsc = audioCtx.createOscillator();
          const bassGain = audioCtx.createGain();
          const bassFilter = audioCtx.createBiquadFilter();

          bassOsc.type = 'sawtooth';
          bassOsc.frequency.setValueAtTime(bassFreq, now);

          bassFilter.type = 'lowpass';
          bassFilter.frequency.setValueAtTime(500, now);

          bassGain.gain.setValueAtTime(0.18, now);
          bassGain.gain.exponentialRampToValueAtTime(0.01, now + 0.14);

          bassOsc.connect(bassFilter);
          bassFilter.connect(bassGain);
          bassGain.connect(audioCtx.destination);

          bassOsc.start(now);
          bassOsc.stop(now + 0.14);
        }

        // 3. Synthwave Drum Beat (Punchy Kick & Electro Snare)
        if (step % 4 === 0) {
          // Synth Kick Drum
          const kickOsc = audioCtx.createOscillator();
          const kickGain = audioCtx.createGain();
          kickOsc.frequency.setValueAtTime(140, now);
          kickOsc.frequency.exponentialRampToValueAtTime(35, now + 0.1);
          kickGain.gain.setValueAtTime(0.3, now);
          kickGain.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
          kickOsc.connect(kickGain);
          kickGain.connect(audioCtx.destination);
          kickOsc.start(now);
          kickOsc.stop(now + 0.1);
        }

        if (step === 4 || step === 12) {
          // Retro Gated Snare
          const bufSize = audioCtx.sampleRate * 0.12;
          const noiseBuf = audioCtx.createBuffer(1, bufSize, audioCtx.sampleRate);
          const output = noiseBuf.getChannelData(0);
          for (let i = 0; i < bufSize; i++) output[i] = Math.random() * 2 - 1;

          const snare = audioCtx.createBufferSource();
          snare.buffer = noiseBuf;
          const snareFilter = audioCtx.createBiquadFilter();
          snareFilter.type = 'highpass';
          snareFilter.frequency.setValueAtTime(1200, now);

          const snareGain = audioCtx.createGain();
          snareGain.gain.setValueAtTime(0.22, now);
          snareGain.gain.exponentialRampToValueAtTime(0.01, now + 0.12);

          snare.connect(snareFilter);
          snareFilter.connect(snareGain);
          snareGain.connect(audioCtx.destination);
          snare.start(now);
        }

        bgmStep++;
      }, 125); // 120 BPM Synthwave Groove
    }

    // 經典 GTA West Coast Hip-Hop / Funk 樂曲音效合成器 (Web Audio Synthesizer BGM)
    function startGTABGM() {
      if (bgmInterval) return;

      // 經典 G-Funk Bassline 頻率 (C, Eb, F, F#, G, Bb)
      const bassNotes = [65.41, 65.41, 0, 77.78, 87.31, 92.50, 98.00, 0, 116.54, 98.00, 87.31, 77.78, 65.41, 0, 98.00, 0];

      bgmInterval = setInterval(() => {
        if (!audioCtx || isGamePaused) return;

        const now = audioCtx.currentTime;
        const step = bgmStep % 16;

        // 1. G-Funk 復古低音 (Bassline Synthesizer)
        const note = bassNotes[step];
        if (note > 0) {
          const bassOsc = audioCtx.createOscillator();
          const bassGain = audioCtx.createGain();
          const bassFilter = audioCtx.createBiquadFilter();

          bassOsc.type = 'sawtooth';
          bassOsc.frequency.setValueAtTime(note, now);

          bassFilter.type = 'lowpass';
          bassFilter.frequency.setValueAtTime(450, now);
          bassFilter.frequency.exponentialRampToValueAtTime(120, now + 0.15);

          bassGain.gain.setValueAtTime(0.18, now);
          bassGain.gain.exponentialRampToValueAtTime(0.001, now + 0.18);

          bassOsc.connect(bassFilter);
          bassFilter.connect(bassGain);
          bassGain.connect(audioCtx.destination);

          bassOsc.start(now);
          bassOsc.stop(now + 0.18);
        }

        // 2. 爵士小鼓與 Hi-Hat 節奏 (Drums & Vinyl Groove)
        if (step === 4 || step === 12) {
          // Snare Drum (White Noise + Bandpass)
          const bufSize = audioCtx.sampleRate * 0.12;
          const noiseBuf = audioCtx.createBuffer(1, bufSize, audioCtx.sampleRate);
          const output = noiseBuf.getChannelData(0);
          for (let i = 0; i < bufSize; i++) output[i] = Math.random() * 2 - 1;

          const snare = audioCtx.createBufferSource();
          snare.buffer = noiseBuf;

          const snareFilter = audioCtx.createBiquadFilter();
          snareFilter.type = 'highpass';
          snareFilter.frequency.setValueAtTime(1000, now);

          const snareGain = audioCtx.createGain();
          snareGain.gain.setValueAtTime(0.2, now);
          snareGain.gain.exponentialRampToValueAtTime(0.01, now + 0.12);

          snare.connect(snareFilter);
          snareFilter.connect(snareGain);
          snareGain.connect(audioCtx.destination);

          snare.start(now);
        }

        // Hi-Hat 敲擊 (Steps 0, 2, 4, 6, 8, 10, 12, 14)
        if (step % 2 === 0) {
          const hatOsc = audioCtx.createOscillator();
          const hatGain = audioCtx.createGain();
          hatOsc.type = 'square';
          hatOsc.frequency.setValueAtTime(6000 + Math.random() * 2000, now);

          hatGain.gain.setValueAtTime(0.04, now);
          hatGain.gain.exponentialRampToValueAtTime(0.001, now + 0.04);

          hatOsc.connect(hatGain);
          hatGain.connect(audioCtx.destination);
          hatOsc.start(now);
          hatOsc.stop(now + 0.04);
        }

        bgmStep++;
      }, 125); // 120 BPM Hip-Hop 節奏
    }

    function updateEngineSound(speed) {
      if (!audioCtx || !engineOsc) return;
      const pitch = 45 + Math.abs(speed) * 160;
      engineOsc.frequency.setTargetAtTime(pitch, audioCtx.currentTime, 0.08);
      const gainVal = gamePhase === 2 ? Math.min(0.12, 0.02 + Math.abs(speed) * 0.1) : 0.01;
      engineGain.gain.setTargetAtTime(gainVal, audioCtx.currentTime, 0.08);
    }

    function playSwitchLaneSound() {
      if (!audioCtx) return;
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = 'square';
      osc.frequency.setValueAtTime(400, audioCtx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(800, audioCtx.currentTime + 0.05);
      gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.05);
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start();
      osc.stop(audioCtx.currentTime + 0.05);
    }

    function playCrashSound() {
      if (!audioCtx) return;
      const bufferSize = audioCtx.sampleRate * 0.3;
      const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
      const data = buffer.getChannelData(0);
      for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;

      const noise = audioCtx.createBufferSource();
      noise.buffer = buffer;
      const filter = audioCtx.createBiquadFilter();
      filter.type = 'lowpass';
      filter.frequency.setValueAtTime(800, audioCtx.currentTime);
      filter.frequency.exponentialRampToValueAtTime(100, audioCtx.currentTime + 0.3);
      const gain = audioCtx.createGain();
      gain.gain.setValueAtTime(0.4, audioCtx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
      noise.connect(filter);
      filter.connect(gain);
      gain.connect(audioCtx.destination);
      noise.start();
    }

    function playItemBoxSound() {
      if (!audioCtx) return;
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(587.33, audioCtx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(1174.66, audioCtx.currentTime + 0.15);
      gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.15);
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start();
      osc.stop(audioCtx.currentTime + 0.15);
    }

    function playSuccessSound() {
      if (!audioCtx) return;
      const now = audioCtx.currentTime;
      [523.25, 659.25, 783.99, 1046.50].forEach((freq, idx) => {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'sine';
        osc.frequency.setValueAtTime(freq, now + idx * 0.08);
        gain.gain.setValueAtTime(0.2, now + idx * 0.08);
        gain.gain.exponentialRampToValueAtTime(0.01, now + idx * 0.08 + 0.2);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start(now + idx * 0.08);
        osc.stop(now + idx * 0.08 + 0.2);
      });
    }

    function playSpinoutSound() {
      if (!audioCtx) return;
      const now = audioCtx.currentTime;
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = 'sawtooth';
      osc.frequency.setValueAtTime(300, now);
      osc.frequency.linearRampToValueAtTime(80, now + 0.5);
      gain.gain.setValueAtTime(0.3, now);
      gain.gain.exponentialRampToValueAtTime(0.01, now + 0.5);
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start(now);
      osc.stop(now + 0.5);
    }

    /* =========================================================================
       3. THREE.JS ENGINE - FULL LEGO WORLD (FIXED LANES & FORKS)
       ========================================================================= */
    let scene, camera, renderer;
    let playerGroup, playerLeftLeg, playerRightLeg, playerLeftArm, playerRightArm;
    let legoTaxiGroup, frontWheels = [], brakeLights = [];
    let particleSystem, particleGeo;
    let minimapCtx;

    // 樂高軌道系統 (Fixed Lanes System: Left: -4.0, Middle: 0.0, Right: 4.0)
    const LANES = [-4.0, 0.0, 4.0];
    let currentLaneIndex = 1; // 預設中間軌道
    let targetX = LANES[1];

    let npcVehicles = [];
    let legoPedestrians = [];

    const CHUNK_SIZE = 50;
    const POOL_SIZE = 8;
    let chunkPool = [];
    let buildingInstancedMesh;

    let cameraShake = 0;
    let walkAnimTime = 0;

    let taxiPhysics = {
      x: 0.0, z: -25,
      speed: 0,
      heading: 0,
      wheelbase: 2.8,
      bodyRoll: 0
    };

    let playerPos = { x: 9.0, y: 0, z: -25 };
    const keys = { w: false, a: false, s: false, d: false, space: false };

    // 樂高材料顏色常數
    const LEGO_RED = 0xef4444;
    const LEGO_YELLOW = 0xfacc15;
    const LEGO_BLUE = 0x2563eb;
    const LEGO_GREEN = 0x16a34a;
    const LEGO_WHITE = 0xf8fafc;
    const LEGO_BLACK = 0x1e293b;

    function init3DWorld() {
      const container = document.getElementById('webgl-container');

      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x38bdf8);
      scene.fog = new THREE.FogExp2(0x38bdf8, 0.007);

      camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);

      renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1.5));
      renderer.shadowMap.enabled = true;
      renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      renderer.toneMappingExposure = 1.2;
      container.appendChild(renderer.domElement);

      setupLighting();
      initLegoBuildings();
      initChunkPool();
      createLegoTaxiModel();
      createLegoPlayerModel();
      createNitroParticles();

      initNPCVehicles();
      initNPCPedestrians();

      const minimapCanvas = document.getElementById('minimap');
      minimapCtx = minimapCanvas.getContext('2d');

      window.addEventListener('resize', onWindowResize);
      setupControls();

      animate();
    }

    function setupLighting() {
      const ambient = new THREE.AmbientLight(0xffffff, 0.9);
      scene.add(ambient);

      const sun = new THREE.DirectionalLight(0xfffbe8, 1.3);
      sun.position.set(50, 100, 40);
      sun.castShadow = true;
      sun.shadow.mapSize.width = 1024;
      sun.shadow.mapSize.height = 1024;
      scene.add(sun);
    }

    // 輔助函式：建立樂高積木（帶有頂部圓形顆粒 Studs）
    function createLegoBrick(width, height, depth, color, studRows = 2, studCols = 2) {
      const group = new THREE.Group();
      const mat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.2, metalness: 0.1 });

      // 主積木體
      const box = new THREE.Mesh(new THREE.BoxGeometry(width, height, depth), mat);
      box.position.y = height / 2;
      box.castShadow = true;
      box.receiveShadow = true;
      group.add(box);

      // 頂部顆粒 (Studs)
      const studGeo = new THREE.CylinderGeometry(0.14, 0.14, 0.08, 12);
      const startX = -width / 2 + width / (studCols * 2);
      const startZ = -depth / 2 + depth / (studRows * 2);

      for (let r = 0; r < studRows; r++) {
        for (let c = 0; c < studCols; c++) {
          const stud = new THREE.Mesh(studGeo, mat);
          stud.position.set(
            startX + (c * (width / studCols)),
            height + 0.04,
            startZ + (r * (depth / studRows))
          );
          group.add(stud);
        }
      }
      return group;
    }

    function initLegoBuildings() {
      const maxCount = 32;
      const geo = new THREE.BoxGeometry(12, 38, 18);
      const mat = new THREE.MeshStandardMaterial({ color: LEGO_BLUE, roughness: 0.3 });
      buildingInstancedMesh = new THREE.InstancedMesh(geo, mat, maxCount);
      buildingInstancedMesh.instanceMatrix.setUsage(THREE.DynamicDrawUsage);

      const dummy = new THREE.Object3D();
      let idx = 0;
      for (let i = 0; i < POOL_SIZE; i++) {
        const chunkZ = 50 - (i * CHUNK_SIZE);
        dummy.position.set(-20, 19, chunkZ);
        dummy.updateMatrix();
        buildingInstancedMesh.setMatrixAt(idx++, dummy.matrix);

        dummy.position.set(20, 19, chunkZ);
        dummy.updateMatrix();
        buildingInstancedMesh.setMatrixAt(idx++, dummy.matrix);
      }
      buildingInstancedMesh.instanceMatrix.needsUpdate = true;
      scene.add(buildingInstancedMesh);
    }

    function initChunkPool() {
      const roadGeo = new THREE.PlaneGeometry(14, CHUNK_SIZE);
      const roadMat = new THREE.MeshStandardMaterial({ color: LEGO_BLACK, roughness: 0.3 });
      const walkGeo = new THREE.BoxGeometry(4, 0.4, CHUNK_SIZE);
      const walkMat = new THREE.MeshStandardMaterial({ color: 0x94a3b8, roughness: 0.4 });

      for (let i = 0; i < POOL_SIZE; i++) {
        const chunkGroup = new THREE.Group();
        const zPos = 50 - (i * CHUNK_SIZE);
        chunkGroup.position.z = zPos;

        // 馬路底板
        const road = new THREE.Mesh(roadGeo, roadMat);
        road.rotation.x = -Math.PI / 2;
        road.receiveShadow = true;
        chunkGroup.add(road);

        // 樂高三軌道白色分隔線 (3 Fixed Lanes: -4.0, 0.0, 4.0)
        [-2.0, 2.0].forEach(xOffset => {
          for (let d = -CHUNK_SIZE / 2; d < CHUNK_SIZE / 2; d += 6) {
            const dash = new THREE.Mesh(new THREE.PlaneGeometry(0.3, 3), new THREE.MeshStandardMaterial({ color: LEGO_WHITE }));
            dash.rotation.x = -Math.PI / 2;
            dash.position.set(xOffset, 0.02, d);
            chunkGroup.add(dash);
          }
        });

        // 樂高兩側路緣石與行人路
        [-8.5, 8.5].forEach((x) => {
          const walk = new THREE.Mesh(walkGeo, walkMat);
          walk.position.set(x, 0.2, 0);
          walk.receiveShadow = true;
          chunkGroup.add(walk);

          // 樂高積木護欄
          const fence = createLegoBrick(0.5, 0.6, CHUNK_SIZE, LEGO_YELLOW, 16, 1);
          fence.position.set(x > 0 ? -6.8 : 6.8, 0.4, 0);
          chunkGroup.add(fence);
        });

        // 樂高問號問答箱
        const qBox = createLegoBrick(1.4, 1.4, 1.4, LEGO_YELLOW, 2, 2);
        qBox.position.set(LANES[i % 3], 0.2, 0);
        chunkGroup.add(qBox);

        scene.add(chunkGroup);
        chunkPool.push({ group: chunkGroup, z: zPos, box: qBox, activeBox: true });
      }
    }

    /* --- 全樂高風香港紅的 (LEGO Hong Kong Red Crown Comfort Taxi) --- */
    function createLegoTaxiModel() {
      legoTaxiGroup = new THREE.Group();

      // 車底盤
      const chassis = createLegoBrick(2.2, 0.4, 4.4, LEGO_BLACK, 8, 4);
      chassis.position.y = 0.2;
      legoTaxiGroup.add(chassis);

      // 車頭引擎蓋
      const hood = createLegoBrick(2.1, 0.5, 1.6, LEGO_RED, 3, 4);
      hood.position.set(0, 0.6, -1.3);
      legoTaxiGroup.add(hood);

      // 車頂 (白色樂高蓋)
      const roof = createLegoBrick(1.9, 0.6, 2.2, LEGO_WHITE, 4, 4);
      roof.position.set(0, 1.1, 0.2);
      legoTaxiGroup.add(roof);

      // 樂高 TAXI 頂燈
      const sign = createLegoBrick(0.8, 0.3, 0.3, LEGO_YELLOW, 1, 2);
      sign.position.set(0, 1.7, 0.2);
      legoTaxiGroup.add(sign);

      // 煞車尾燈
      const brakeMat = new THREE.MeshStandardMaterial({ color: LEGO_RED, emissive: 0xff0000, emissiveIntensity: 0.3 });
      [-0.8, 0.8].forEach(x => {
        const light = new THREE.Mesh(new THREE.BoxGeometry(0.35, 0.2, 0.1), brakeMat);
        light.position.set(x, 0.7, 2.22);
        brakeLights.push(light);
        legoTaxiGroup.add(light);
      });

      // 樂高輪胎
      const wheelGeo = new THREE.CylinderGeometry(0.4, 0.4, 0.3, 16);
      const wheelMat = new THREE.MeshStandardMaterial({ color: LEGO_BLACK, roughness: 0.8 });
      const rimMat = new THREE.MeshStandardMaterial({ color: LEGO_WHITE });

      [[-1.15, 0.4, -1.2], [1.15, 0.4, -1.2]].forEach(pos => {
        const pivot = new THREE.Group();
        pivot.position.set(...pos);
        const w = new THREE.Mesh(wheelGeo, wheelMat);
        w.rotation.z = Math.PI / 2;
        const rim = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 0.32, 12), rimMat);
        rim.rotation.z = Math.PI / 2;
        pivot.add(w);
        pivot.add(rim);
        frontWheels.push(pivot);
        legoTaxiGroup.add(pivot);
      });

      [[-1.15, 0.4, 1.2], [1.15, 0.4, 1.2]].forEach(pos => {
        const w = new THREE.Mesh(wheelGeo, wheelMat);
        w.rotation.z = Math.PI / 2;
        w.position.set(...pos);
        legoTaxiGroup.add(w);
      });

      legoTaxiGroup.position.set(taxiPhysics.x, 0, taxiPhysics.z);
      scene.add(legoTaxiGroup);
    }

    /* --- 全樂高風主角人偶 (LEGO Minifigure) --- */
    function createLegoPlayerModel() {
      playerGroup = new THREE.Group();

      const yellowMat = new THREE.MeshStandardMaterial({ color: LEGO_YELLOW, roughness: 0.2 });
      const blueMat = new THREE.MeshStandardMaterial({ color: LEGO_BLUE, roughness: 0.3 });
      const redMat = new THREE.MeshStandardMaterial({ color: LEGO_RED, roughness: 0.3 });

      // 1. LEGO 頭部 + 頂部 Stud
      const headGroup = new THREE.Group();
      const head = new THREE.Mesh(new THREE.CylinderGeometry(0.24, 0.24, 0.36, 16), yellowMat);
      head.position.y = 1.32;
      headGroup.add(head);

      const stud = new THREE.Mesh(new THREE.CylinderGeometry(0.12, 0.12, 0.08, 12), yellowMat);
      stud.position.y = 1.54;
      headGroup.add(stud);

      // 五官眼睛與微笑
      const eyeMat = new THREE.MeshBasicMaterial({ color: 0x000000 });
      [-0.08, 0.08].forEach(ex => {
        const eye = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.06, 0.02), eyeMat);
        eye.position.set(ex, 1.35, 0.24);
        headGroup.add(eye);
      });
      playerGroup.add(headGroup);

      // 2. 梯形積木軀幹 (Torso)
      const torso = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.55, 0.3), blueMat);
      torso.position.y = 0.88;
      torso.castShadow = true;
      playerGroup.add(torso);

      // 3. 雙臂與 C 型樂高手掌
      const armGeo = new THREE.CylinderGeometry(0.07, 0.07, 0.42, 12);
      
      const leftArmPivot = new THREE.Group();
      leftArmPivot.position.set(-0.32, 1.08, 0);
      const lArm = new THREE.Mesh(armGeo, blueMat);
      lArm.position.y = -0.2;
      leftArmPivot.add(lArm);
      playerLeftArm = leftArmPivot;
      playerGroup.add(leftArmPivot);

      const rightArmPivot = new THREE.Group();
      rightArmPivot.position.set(0.32, 1.08, 0);
      const rArm = new THREE.Mesh(armGeo, blueMat);
      rArm.position.y = -0.2;
      rightArmPivot.add(rArm);
      playerRightArm = rightArmPivot;
      playerGroup.add(rightArmPivot);

      // 4. 雙腳 (Legs)
      const legGeo = new THREE.BoxGeometry(0.22, 0.5, 0.26);

      const leftLegPivot = new THREE.Group();
      leftLegPivot.position.set(-0.13, 0.5, 0);
      const lLeg = new THREE.Mesh(legGeo, redMat);
      lLeg.position.y = -0.25;
      lLeg.castShadow = true;
      leftLegPivot.add(lLeg);
      playerLeftLeg = leftLegPivot;
      playerGroup.add(leftLegPivot);

      const rightLegPivot = new THREE.Group();
      rightLegPivot.position.set(0.13, 0.5, 0);
      const rLeg = new THREE.Mesh(legGeo, redMat);
      rLeg.position.y = -0.25;
      rLeg.castShadow = true;
      rightLegPivot.add(rLeg);
      playerRightLeg = rightLegPivot;
      playerGroup.add(rightLegPivot);

      playerGroup.position.set(playerPos.x, playerPos.y, playerPos.z);
      scene.add(playerGroup);
    }

    function createNitroParticles() {
      const particleCount = 60;
      particleGeo = new THREE.BufferGeometry();
      const positions = new Float32Array(particleCount * 3);
      for (let i = 0; i < particleCount * 3; i++) positions[i] = 0;
      particleGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));

      const particleMat = new THREE.PointsMaterial({
        color: LEGO_YELLOW,
        size: 0.35,
        transparent: true,
        opacity: 0.9,
        blending: THREE.AdditiveBlending
      });
      particleSystem = new THREE.Points(particleGeo, particleMat);
      particleSystem.visible = false;
      scene.add(particleSystem);
    }

    /* --- 單向無對頭車的 NPC 樂高車輛系統 --- */
    function initNPCVehicles() {
      const colors = [LEGO_GREEN, LEGO_BLUE, LEGO_YELLOW];
      for (let i = 0; i < 5; i++) {
        const car = createLegoBrick(2.1, 0.9, 4.0, colors[i % colors.length], 4, 2);
        const laneX = LANES[i % 3];
        const z = -40 - i * 45;
        car.position.set(laneX, 0, z);

        scene.add(car);
        // 所有車輛方向一致 (No Oncoming Traffic!)
        npcVehicles.push({ mesh: car, speed: 0.22, lane: i % 3 });
      }
    }

    function initNPCPedestrians() {
      for (let i = 0; i < 8; i++) {
        const ped = createLegoBrick(0.5, 1.2, 0.5, LEGO_GREEN, 1, 1);
        const x = Math.random() > 0.5 ? 8.5 : -8.5;
        const z = -Math.random() * 180;
        ped.position.set(x, 0.2, z);

        scene.add(ped);
        legoPedestrians.push({ mesh: ped, speed: (Math.random() * 0.02 + 0.01) * (Math.random() > 0.5 ? 1 : -1) });
      }
    }

    /* =========================================================================
       4. FIXED RAIL LANES & FORK BRANCHING LOGIC
       ========================================================================= */
    function switchLane(direction) {
      if (gamePhase !== 2) return;
      initWebAudio();

      if (direction === 'left' && currentLaneIndex > 0) {
        currentLaneIndex--;
        targetX = LANES[currentLaneIndex];
        playSwitchLaneSound();
      } else if (direction === 'right' && currentLaneIndex < LANES.length - 1) {
        currentLaneIndex++;
        targetX = LANES[currentLaneIndex];
        playSwitchLaneSound();
      }
    }

    function checkNPCOvertakeAndCollisions() {
      const taxiBox = new THREE.Box3().setFromObject(legoTaxiGroup);
      const npcBox = new THREE.Box3();

      npcVehicles.forEach(v => {
        npcBox.setFromObject(v.mesh);
        if (taxiBox.intersectsBox(npcBox)) {
          taxiPhysics.speed = 0; // 車速停下
          taxiPhysics.z += 1.0; // 後退彈開
          cameraShake = 0.5;
          playCrashSound();
        }
      });
    }

    function updateNPCs() {
      npcVehicles.forEach(v => {
        v.mesh.position.z -= (taxiPhysics.speed - v.speed); // 同向相對運動
        if (v.mesh.position.z > taxiPhysics.z + 45) v.mesh.position.z = taxiPhysics.z - 160;
        if (v.mesh.position.z < taxiPhysics.z - 160) v.mesh.position.z = taxiPhysics.z + 45;
      });

      legoPedestrians.forEach(p => {
        p.mesh.position.z += p.speed;
        if (p.mesh.position.z > taxiPhysics.z + 30) p.mesh.position.z = taxiPhysics.z - 150;
        if (p.mesh.position.z < taxiPhysics.z - 150) p.mesh.position.z = taxiPhysics.z + 30;
      });
    }

    /* =========================================================================
       5. CONTROLS & ANIMATION LOOP
       ========================================================================= */
    function setupControls() {
      window.addEventListener('keydown', (e) => {
        initWebAudio();
        const k = e.key.toLowerCase();
        if (k === 'w' || k === 'arrowup') keys.w = true;
        if (k === 's' || k === 'arrowdown') keys.s = true;

        if (k === 'a' || k === 'arrowleft') {
          keys.a = true;
          if (gamePhase === 2) switchLane('left');
        }
        if (k === 'd' || k === 'arrowright') {
          keys.d = true;
          if (gamePhase === 2) switchLane('right');
        }
        if (k === ' ') keys.space = true;
        if (k === 'e') triggerAction();
      });

      window.addEventListener('keyup', (e) => {
        const k = e.key.toLowerCase();
        if (k === 'w' || k === 'arrowup') keys.w = false;
        if (k === 'a' || k === 'arrowleft') keys.a = false;
        if (k === 's' || k === 'arrowdown') keys.s = false;
        if (k === 'd' || k === 'arrowright') keys.d = false;
        if (k === ' ') keys.space = false;
      });

      document.getElementById('btn-left').addEventListener('click', () => switchLane('left'));
      document.getElementById('btn-right').addEventListener('click', () => switchLane('right'));

      const bindTouch = (id, keyName) => {
        const btn = document.getElementById(id);
        btn.addEventListener('touchstart', (e) => { initWebAudio(); e.preventDefault(); keys[keyName] = true; });
        btn.addEventListener('touchend', (e) => { e.preventDefault(); keys[keyName] = false; });
      };
      bindTouch('btn-up', 'w');
      bindTouch('btn-down', 's');
      bindTouch('btn-brake', 'space');
      document.getElementById('btn-action').addEventListener('click', triggerAction);
    }

    function triggerAction() {
      initWebAudio();
      if (gamePhase === 1) {
        const dist = playerGroup.position.distanceTo(legoTaxiGroup.position);
        if (dist < 4.5) {
          gamePhase = 2;
          playerGroup.visible = false;
          document.getElementById('action-hint').classList.add('hidden');
          document.getElementById('btn-action').classList.add('hidden');
          const badge = document.getElementById('phase-badge');
          badge.innerText = "階段二：LEGO 的士切換軌道競速";
          badge.className = "mt-1 text-xs px-2.5 py-1 bg-red-600/90 rounded-lg text-white inline-block font-semibold shadow-md";
        }
      }
    }

    function animate() {
      requestAnimationFrame(animate);

      if (!isGamePaused) {
        if (gamePhase === 1) {
          updatePlayerPhysicsAndLegAnimation();
        } else if (gamePhase === 2) {
          updateLegoTaxiRailPhysics();
          recycleChunkPoolZeroAlloc();
          checkNPCOvertakeAndCollisions();
          updateItemBoxesCollision();
          updateHUDMinimapAndSpeed();
          updateNitroParticles();
        }
        updateNPCs();
      }

      renderer.render(scene, camera);
    }

    function updatePlayerPhysicsAndLegAnimation() {
      const speed = 0.16;
      let isMoving = false;

      if (keys.w) { playerPos.z -= speed; isMoving = true; }
      if (keys.s) { playerPos.z += speed; isMoving = true; }
      if (keys.a) { playerPos.x -= speed; isMoving = true; }
      if (keys.d) { playerPos.x += speed; isMoving = true; }

      playerPos.x = THREE.MathUtils.clamp(playerPos.x, 0.0, 11.0);
      playerPos.z = THREE.MathUtils.clamp(playerPos.z, -35.0, 5.0);
      playerGroup.position.set(playerPos.x, playerPos.y, playerPos.z);

      if (isMoving) {
        walkAnimTime += 0.2;
        const angle = Math.sin(walkAnimTime) * 0.6;
        playerLeftLeg.rotation.x = angle;
        playerRightLeg.rotation.x = -angle;
        playerLeftArm.rotation.x = -angle;
        playerRightArm.rotation.x = angle;
      } else {
        playerLeftLeg.rotation.x = 0;
        playerRightLeg.rotation.x = 0;
        playerLeftArm.rotation.x = 0;
        playerRightArm.rotation.x = 0;
      }

      camera.position.set(playerPos.x, playerPos.y + 4.5, playerPos.z + 8.5);
      camera.lookAt(playerPos.x, playerPos.y + 1.2, playerPos.z);

      const dist = playerGroup.position.distanceTo(legoTaxiGroup.position);
      const hint = document.getElementById('action-hint');
      const mobileBtn = document.getElementById('btn-action');
      if (dist < 4.5) {
        hint.classList.remove('hidden');
        mobileBtn.classList.remove('hidden');
      } else {
        hint.classList.add('hidden');
        mobileBtn.classList.add('hidden');
      }
    }

    /* --- 軌道切換與阿克曼平滑移動 (Smooth Lane Switching) --- */
    function updateLegoTaxiRailPhysics() {
      let isTurbo = Date.now() < turboEndTime;
      let isSpinning = Date.now() < spinOutEndTime;

      document.getElementById('turbo-badge').classList.toggle('hidden', !isTurbo);
      particleSystem.visible = isTurbo;

      let maxSpeed = isTurbo ? 1.15 : 0.75;
      let accel = 0.025;
      let friction = 0.97;

      if (isSpinning) {
        legoTaxiGroup.rotation.y += 0.28;
        taxiPhysics.speed *= 0.9;
      } else {
        if (keys.w) taxiPhysics.speed += accel;
        if (keys.s) taxiPhysics.speed -= accel * 0.7;
        if (keys.space) {
          taxiPhysics.speed *= 0.85;
          brakeLights.forEach(l => l.material.emissiveIntensity = 1.0);
        } else {
          brakeLights.forEach(l => l.material.emissiveIntensity = 0.2);
        }

        taxiPhysics.speed *= friction;
        if (Math.abs(taxiPhysics.speed) > maxSpeed) taxiPhysics.speed = Math.sign(taxiPhysics.speed) * maxSpeed;

        // 平滑移至目標線道 targetX
        taxiPhysics.x = THREE.MathUtils.lerp(taxiPhysics.x, targetX, 0.15);
        frontWheels.forEach(w => w.rotation.y = (targetX - taxiPhysics.x) * 0.3);
        legoTaxiGroup.rotation.y = (targetX - taxiPhysics.x) * -0.1;
      }

      taxiPhysics.z -= taxiPhysics.speed;
      legoTaxiGroup.position.set(taxiPhysics.x, 0, taxiPhysics.z);

      updateEngineSound(taxiPhysics.speed);

      let shakeX = (Math.random() - 0.5) * cameraShake;
      let shakeY = (Math.random() - 0.5) * cameraShake;
      cameraShake *= 0.88;

      camera.position.x = taxiPhysics.x + shakeX;
      camera.position.y = 4.5 + shakeY;
      camera.position.z = taxiPhysics.z + 9.5;

      camera.fov = isTurbo ? 75 : 60;
      camera.updateProjectionMatrix();

      camera.lookAt(taxiPhysics.x, 1.2, taxiPhysics.z - 4);
    }

    function updateNitroParticles() {
      if (!particleSystem.visible) return;
      const positions = particleGeo.attributes.position.array;
      for (let i = 0; i < positions.length; i += 3) {
        positions[i] = taxiPhysics.x + (Math.random() - 0.5) * 0.6;
        positions[i + 1] = 0.5 + Math.random() * 0.4;
        positions[i + 2] = taxiPhysics.z + 2.2 + Math.random() * 1.5;
      }
      particleGeo.attributes.position.needsUpdate = true;
    }

    function recycleChunkPoolZeroAlloc() {
      let minZ = 0;
      for (let i = 0; i < POOL_SIZE; i++) {
        if (chunkPool[i].z < minZ) minZ = chunkPool[i].z;
      }

      const dummy = new THREE.Object3D();
      let instIdx = 0;

      for (let i = 0; i < POOL_SIZE; i++) {
        const item = chunkPool[i];
        if (item.z > taxiPhysics.z + 60) {
          item.z = minZ - CHUNK_SIZE;
          item.group.position.z = item.z;
          item.activeBox = true;
          item.box.visible = true;
          item.box.position.x = LANES[Math.floor(Math.random() * 3)];
        }

        dummy.position.set(-20, 19, item.z);
        dummy.updateMatrix();
        buildingInstancedMesh.setMatrixAt(instIdx++, dummy.matrix);

        dummy.position.set(20, 19, item.z);
        dummy.updateMatrix();
        buildingInstancedMesh.setMatrixAt(instIdx++, dummy.matrix);
      }
      buildingInstancedMesh.instanceMatrix.needsUpdate = true;
    }

    function updateItemBoxesCollision() {
      const worldPos = new THREE.Vector3();
      chunkPool.forEach(item => {
        if (item.activeBox) {
          item.box.rotation.y += 0.04;
          item.box.getWorldPosition(worldPos);
          const dist = legoTaxiGroup.position.distanceTo(worldPos);
          if (dist < 2.2) {
            item.activeBox = false;
            item.box.visible = false;
            playItemBoxSound();
            triggerQuizModal();
          }
        }
      });
    }

    function updateHUDMinimapAndSpeed() {
      const kmh = Math.round(Math.abs(taxiPhysics.speed) * 140);
      document.getElementById('speed-display').innerText = kmh;

      if (minimapCtx) {
        minimapCtx.clearRect(0, 0, 90, 90);
        minimapCtx.fillStyle = "#334155";
        minimapCtx.fillRect(25, 0, 40, 90);

        const mx = 45 + (taxiPhysics.x / 4) * 12;
        minimapCtx.fillStyle = "#ef4444";
        minimapCtx.beginPath();
        minimapCtx.arc(mx, 45, 4, 0, Math.PI * 2);
        minimapCtx.fill();
      }
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    /* =========================================================================
       6. AUTH, BIO QUIZ & LEADERBOARD FLOW
       ========================================================================= */
    function handleLogin(e) {
      e.preventDefault();
      initWebAudio();
      const email = document.getElementById('input-email').value;
      const name = document.getElementById('input-name').value;
      const emailError = document.getElementById('email-error');

      if (!email.includes('@') || !email.includes('.edu.hk')) {
        emailError.classList.remove('hidden');
        return;
      }
      emailError.classList.add('hidden');

      currentStudent = { email, name };
      document.getElementById('auth-modal').classList.add('hidden');

      renderChapterSelection();
    }

    function renderChapterSelection() {
      const modal = document.getElementById('chapter-modal');
      const list = document.getElementById('chapter-list');
      list.innerHTML = "";

      Object.keys(defaultQuestionBank).forEach(key => {
        const ch = defaultQuestionBank[key];
        const btn = document.createElement('button');
        btn.className = "w-full bg-white hover:bg-amber-100 text-slate-900 font-bold py-3.5 px-5 rounded-xl border-2 border-amber-300 transition flex justify-between items-center text-sm shadow-md";
        btn.innerHTML = `<span>${ch.title}</span><span class="text-xs font-mono text-slate-500">${ch.questions.length} 題</span>`;
        btn.onclick = () => startGame(key);
        list.appendChild(btn);
      });

      modal.classList.remove('hidden');
    }

    function startGame(chapterKey) {
      selectedChapterKey = chapterKey;
      document.getElementById('chapter-modal').classList.add('hidden');
      document.getElementById('hud-container').classList.remove('hidden');

      startTimer();
      quizInterval = setInterval(triggerQuizModal, 18000);
    }

    function startTimer() {
      timerInterval = setInterval(() => {
        if (!isGamePaused) {
          gameTimer--;
          const mins = String(Math.floor(gameTimer / 60)).padStart(2, '0');
          const secs = String(gameTimer % 60).padStart(2, '0');
          document.getElementById('timer-display').innerText = `${mins}:${secs}`;

          if (gameTimer <= 0) endGame();
        }
      }, 1000);
    }

    function triggerQuizModal() {
      if (isGamePaused || gamePhase !== 2) return;
      isGamePaused = true;

      const ch = defaultQuestionBank[selectedChapterKey];
      const q = ch.questions[Math.floor(Math.random() * ch.questions.length)];

      document.getElementById('quiz-chapter-title').innerText = ch.title;
      document.getElementById('quiz-question').innerText = q.question;

      const optsContainer = document.getElementById('quiz-options');
      optsContainer.innerHTML = "";

      q.options.forEach((optText, index) => {
        const btn = document.createElement('button');
        btn.className = "w-full text-left bg-white hover:bg-slate-100 border-2 border-slate-300 rounded-xl p-4 text-sm transition font-medium flex items-center justify-between shadow-sm";
        btn.innerHTML = `<span>${index + 1}. ${optText}</span>`;
        btn.onclick = () => handleAnswer(index, q.answer, q.reward);
        optsContainer.appendChild(btn);
      });

      document.getElementById('quiz-modal').classList.remove('hidden');
    }

    function handleAnswer(selectedIndex, correctIndex, reward) {
      totalAnswered++;
      document.getElementById('quiz-modal').classList.add('hidden');
      isGamePaused = false;

      const meterEl = document.getElementById('meter-display');

      if (selectedIndex === correctIndex) {
        correctCount++;
        gameScore += reward;
        playSuccessSound();

        document.getElementById('combo-display').innerText = `${correctCount} 題`;
        meterEl.innerText = `HK$ ${gameScore.toFixed(2)}`;
        meterEl.className = "digital-meter text-2xl font-bold px-3 py-1 rounded-lg border-2 border-green-500 text-green-400 animate-pulse";
        turboEndTime = Date.now() + 5000;
      } else {
        gameScore = Math.max(0, gameScore - 15.00);
        playSpinoutSound();

        meterEl.innerText = `HK$ ${gameScore.toFixed(2)}`;
        meterEl.className = "digital-meter text-2xl font-bold px-3 py-1 rounded-lg border-2 border-red-500 text-red-500 animate-bounce";
        spinOutEndTime = Date.now() + 1500;
      }

      setTimeout(() => {
        meterEl.className = "digital-meter text-2xl font-bold px-3 py-1 rounded-lg border-2 border-red-900/50";
      }, 1500);
    }

    function endGame() {
      clearInterval(timerInterval);
      clearInterval(quizInterval);
      isGamePaused = true;

      const accuracy = totalAnswered > 0 ? Math.round((correctCount / totalAnswered) * 100) : 0;
      alert(`🎉 樂高的士任務完成！\n累積跳表總金額：HK$ ${gameScore.toFixed(2)}\n答對率：${accuracy}%`);

      uploadScoreToGAS(accuracy);
    }

    function uploadScoreToGAS(accuracy) {
      if (GAS_WEB_APP_URL === "YOUR_GAS_URL_HERE") return;

      fetch(GAS_WEB_APP_URL, {
        method: 'POST',
        mode: 'cors',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: currentStudent.email,
          name: currentStudent.name,
          chapterTitle: defaultQuestionBank[selectedChapterKey].title,
          score: gameScore,
          accuracy: accuracy
        })
      })
      .then(res => res.json())
      .then(data => console.log("Score uploaded:", data))
      .catch(err => console.error("Score error:", err));
    }

    function fetchLeaderboard() {
      if (GAS_WEB_APP_URL === "YOUR_GAS_URL_HERE") {
        renderLeaderboard([
          { name: "張小明", score: 235.00 },
          { name: "李小玲", score: 195.00 },
          { name: "黃偉傑", score: 160.00 }
        ]);
        return;
      }

      fetch(GAS_WEB_APP_URL)
        .then(res => res.json())
        .then(data => renderLeaderboard(data))
        .catch(() => {
          document.getElementById('leaderboard-body').innerHTML = `<tr><td colspan="3" class="py-4 text-center text-red-400">未能讀取排行榜</td></tr>`;
        });
    }

    function renderLeaderboard(data) {
      const tbody = document.getElementById('leaderboard-body');
      tbody.innerHTML = "";
      data.forEach((row, idx) => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td class="py-2.5 font-bold ${idx < 3 ? 'text-amber-500' : 'text-slate-400'}">#${idx + 1}</td>
          <td class="py-2.5 font-medium text-slate-800">${row.name}</td>
          <td class="py-2.5 text-right font-mono text-green-600 font-bold">HK$ ${Number(row.score).toFixed(2)}</td>
        `;
        tbody.appendChild(tr);
      });
    }

    window.onload = () => {
      init3DWorld();
      fetchLeaderboard();
    };
  </script>
</body>
</html>
