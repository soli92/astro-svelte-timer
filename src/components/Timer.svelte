<script>
  import { onDestroy } from 'svelte';

  // --- State ---
  let seconds = $state(0);
  let running = $state(false);
  let interval = $state(null);
  let laps = $state([]);
  let mood = $state('😴');
  let confetti = $state([]);

  // --- Moods in base al tempo ---
  const getMood = (s) => {
    if (s === 0)   return '😴';
    if (s < 10)    return '🙂';
    if (s < 30)    return '😊';
    if (s < 60)    return '🤩';
    if (s < 120)   return '🔥';
    if (s < 300)   return '🚀';
    return '🤯';
  };

  const getMoodLabel = (s) => {
    if (s === 0)   return 'Pronto quando vuoi!';
    if (s < 10)    return 'Appena iniziato...';
    if (s < 30)    return 'Ci siamo!';
    if (s < 60)    return 'Stai andando forte!';
    if (s < 120)   return 'Inarrestabile!';
    if (s < 300)   return 'Sei un razzo! 🚀';
    return 'Sei una leggenda! 🏆';
  };

  // --- Formato tempo ---
  const format = (s) => {
    const h = Math.floor(s / 3600);
    const m = Math.floor((s % 3600) / 60);
    const sec = s % 60;
    if (h > 0) return `${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`;
    return `${String(m).padStart(2,'0')}:${String(sec).padStart(2,'0')}`;
  };

  // --- Confetti ---
  const spawnConfetti = () => {
    confetti = Array.from({ length: 18 }, (_, i) => ({
      id: Date.now() + i,
      x: Math.random() * 100,
      color: ['#ff6b6b','#ffd93d','#6bcb77','#4d96ff','#ff922b','#cc5de8'][Math.floor(Math.random()*6)],
      delay: Math.random() * 0.5,
      size: 8 + Math.random() * 10,
    }));
    setTimeout(() => confetti = [], 1800);
  };

  // --- Controlli ---
  const start = () => {
    if (running) return;
    running = true;
    interval = setInterval(() => {
      seconds += 1;
      mood = getMood(seconds);
    }, 1000);
  };

  const pause = () => {
    running = false;
    clearInterval(interval);
    interval = null;
  };

  const reset = () => {
    pause();
    seconds = 0;
    laps = [];
    mood = getMood(0);
    confetti = [];
  };

  const lap = () => {
    if (!running) return;
    spawnConfetti();
    laps = [{ time: format(seconds), raw: seconds }, ...laps];
  };

  onDestroy(() => clearInterval(interval));
</script>

<div class="wrapper">

  <!-- Confetti -->
  {#each confetti as c (c.id)}
    <div
      class="confetto"
      style="left:{c.x}%; background:{c.color}; width:{c.size}px; height:{c.size}px; animation-delay:{c.delay}s;"
    ></div>
  {/each}

  <div class="card">

    <!-- Header mood -->
    <div class="mood-row">
      <span class="mood-emoji">{mood}</span>
      <span class="mood-label">{getMoodLabel(seconds)}</span>
    </div>

    <!-- Display tempo -->
    <div class="timer-display" class:running>
      {format(seconds)}
    </div>

    <!-- Barra di progresso circolare SVG -->
    <div class="progress-ring-wrap">
      <svg viewBox="0 0 120 120" class="ring">
        <circle cx="60" cy="60" r="54" class="ring-bg"/>
        <circle
          cx="60" cy="60" r="54"
          class="ring-fill"
          stroke-dasharray="339.29"
          stroke-dashoffset={339.29 - (339.29 * ((seconds % 60) / 60))}
        />
      </svg>
    </div>

    <!-- Bottoni -->
    <div class="controls">
      {#if !running}
        <button class="btn btn-start" onclick={start}>
          ▶ {seconds === 0 ? 'Start' : 'Riprendi'}
        </button>
      {:else}
        <button class="btn btn-pause" onclick={pause}>⏸ Pausa</button>
      {/if}
      <button class="btn btn-lap" onclick={lap} disabled={!running}>🏁 Lap</button>
      <button class="btn btn-reset" onclick={reset}>🔄 Reset</button>
    </div>

    <!-- Lap list -->
    {#if laps.length > 0}
      <div class="laps">
        <h3>🏆 Lap times</h3>
        <ul>
          {#each laps as lap, i}
            <li>
              <span class="lap-num">#{laps.length - i}</span>
              <span class="lap-time">{lap.time}</span>
            </li>
          {/each}
        </ul>
      </div>
    {/if}

  </div>
</div>

<style>
  .wrapper {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100vw;
    min-height: 100vh;
    overflow: hidden;
  }

  /* ---- Card ---- */
  .card {
    background: rgba(255,255,255,0.07);
    backdrop-filter: blur(18px);
    border: 1px solid rgba(255,255,255,0.15);
    border-radius: 2rem;
    padding: 2.5rem 2rem;
    width: min(420px, 92vw);
    box-shadow: 0 8px 40px rgba(0,0,0,0.4);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.4rem;
    position: relative;
    z-index: 1;
  }

  /* ---- Mood ---- */
  .mood-row {
    display: flex;
    align-items: center;
    gap: .6rem;
  }
  .mood-emoji {
    font-size: 2.2rem;
    animation: bounce 1s infinite alternate;
  }
  .mood-label {
    color: rgba(255,255,255,0.75);
    font-size: 1rem;
    font-weight: 700;
  }

  @keyframes bounce {
    from { transform: translateY(0); }
    to   { transform: translateY(-6px); }
  }

  /* ---- Timer display ---- */
  .timer-display {
    font-size: clamp(3rem, 14vw, 5rem);
    font-weight: 900;
    color: #fff;
    letter-spacing: .05em;
    text-shadow: 0 0 30px rgba(255,255,255,0.3);
    transition: color 0.4s;
  }
  .timer-display.running {
    color: #ffd93d;
    text-shadow: 0 0 40px rgba(255,217,61,0.5);
    animation: pulse 1s infinite;
  }
  @keyframes pulse {
    0%, 100% { transform: scale(1); }
    50%       { transform: scale(1.03); }
  }

  /* ---- Ring ---- */
  .progress-ring-wrap {
    width: 130px;
    height: 130px;
  }
  .ring { width: 100%; height: 100%; transform: rotate(-90deg); }
  .ring-bg {
    fill: none;
    stroke: rgba(255,255,255,0.1);
    stroke-width: 8;
  }
  .ring-fill {
    fill: none;
    stroke: #ffd93d;
    stroke-width: 8;
    stroke-linecap: round;
    transition: stroke-dashoffset 0.9s linear;
    filter: drop-shadow(0 0 6px #ffd93d);
  }

  /* ---- Buttons ---- */
  .controls {
    display: flex;
    gap: .8rem;
    flex-wrap: wrap;
    justify-content: center;
  }
  .btn {
    padding: .65rem 1.3rem;
    border: none;
    border-radius: 999px;
    font-family: 'Nunito', sans-serif;
    font-size: .95rem;
    font-weight: 700;
    cursor: pointer;
    transition: transform 0.15s, box-shadow 0.15s, opacity 0.2s;
  }
  .btn:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(0,0,0,0.3);
  }
  .btn:active:not(:disabled) { transform: scale(0.96); }
  .btn:disabled { opacity: 0.35; cursor: not-allowed; }

  .btn-start  { background: #6bcb77; color: #1a1a2e; }
  .btn-pause  { background: #ffd93d; color: #1a1a2e; }
  .btn-lap    { background: #4d96ff; color: #fff; }
  .btn-reset  { background: #ff6b6b; color: #fff; }

  /* ---- Laps ---- */
  .laps {
    width: 100%;
    background: rgba(255,255,255,0.05);
    border-radius: 1rem;
    padding: 1rem;
    max-height: 180px;
    overflow-y: auto;
  }
  .laps h3 {
    color: #ffd93d;
    font-size: .9rem;
    margin-bottom: .6rem;
    text-align: center;
  }
  .laps ul {
    list-style: none;
    display: flex;
    flex-direction: column;
    gap: .4rem;
  }
  .laps li {
    display: flex;
    justify-content: space-between;
    color: rgba(255,255,255,0.85);
    font-size: .9rem;
    background: rgba(255,255,255,0.07);
    border-radius: .5rem;
    padding: .3rem .7rem;
  }
  .lap-num { color: #4d96ff; font-weight: 700; }
  .lap-time { font-weight: 900; }

  /* ---- Confetti ---- */
  .confetto {
    position: fixed;
    top: -20px;
    border-radius: 3px;
    animation: fall 1.6s ease-in forwards;
    z-index: 999;
  }
  @keyframes fall {
    0%   { transform: translateY(0) rotate(0deg); opacity: 1; }
    100% { transform: translateY(100vh) rotate(720deg); opacity: 0; }
  }

  /* ---- Scrollbar ---- */
  .laps::-webkit-scrollbar { width: 4px; }
  .laps::-webkit-scrollbar-track { background: transparent; }
  .laps::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.2); border-radius: 2px; }
</style>
