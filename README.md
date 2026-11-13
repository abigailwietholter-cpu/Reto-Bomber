# Reto-Bomber
Preguntas variadas sobre temas bomberiles, con un examen final
```html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Reto Bombero</title>
<style>
  :root { --red:#9e1b1b; --dark:#681313; --bg:#f2f4f7; }
  * { box-sizing:border-box; }
  body { margin:0; background:var(--bg); color:#222; font:16px/1.5 Arial,sans-serif; }
  header { padding:18px 5%; color:white; background:var(--dark); }
  header h1, header p { margin:0; }
  main { width:min(900px,94%); margin:22px auto; }
  .card { background:white; border-radius:14px; padding:20px; margin-bottom:16px; box-shadow:0 3px 14px #0001; }
  button, textarea { font:inherit; border-radius:9px; padding:11px 13px; }
  button { margin:4px; border:0; color:white; background:var(--red); cursor:pointer; font-weight:bold; }
  button:hover { background:var(--dark); }
  button.secondary { color:#222; background:#e8eaed; }
  button.answer { display:block; width:100%; text-align:left; color:#222; background:white; border:2px solid #ddd; }
  button.answer:hover { border-color:var(--red); }
  button:disabled { cursor:default; }
  .correct { background:#e6f5ea!important; border-color:#278442!important; }
  .wrong { background:#ffebeb!important; border-color:#b3261e!important; }
  .hidden { display:none!important; }
  .bar { height:12px; margin:10px 0; overflow:hidden; background:#e5e7eb; border-radius:99px; }
  .bar span { display:block; width:0; height:100%; background:#e0a322; transition:width .2s; }
  .grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(180px,1fr)); gap:9px; }
  .topic { padding:10px; background:#f6f6f6; border-radius:9px; }
  .row { display:flex; flex-wrap:wrap; gap:8px; align-items:center; }
  .row button { flex:1; }
  .feedback { min-height:28px; font-weight:bold; }
  .muted { color:#5d6268; }
  textarea { width:100%; min-height:110px; border:1px solid #aaa; }
  .notice { border-left:5px solid #e0a322; background:#fff8e5; }
  .placard { width:min(260px,90%); margin:14px auto; padding:10px; text-align:center; font-weight:bold; color:#111; border:3px solid #222; background:#f28c28; }
  .placard hr { border:1px solid #222; }
  .diamond { position:relative; width:132px; height:132px; margin:25px auto; transform:rotate(45deg); }
  .diamond div { position:absolute; display:grid; width:50%; height:50%; place-items:center; border:1px solid white; font-size:1.4rem; font-weight:bold; }
  .diamond span { transform:rotate(-45deg); }
  .health { top:0; left:0; color:white; background:#1769aa; }
  .fire { top:0; right:0; color:white; background:#c62828; }
  .instability { right:0; bottom:0; color:#222; background:#e6bd28; }
  .special { bottom:0; left:0; color:#222; background:white; }
</style>
</head>
<body>
<header>
  <h1>🚒 Reto Bombero</h1>
  <p>Práctica por temas, niveles y simulacro</p>
</header>

<main>
  <section id="home" class="card">
    <h2>¿Cuánto sabemos?</h2>
    <p id="overall">Todavía no hay respuestas registradas.</p>
    <div class="bar"><span id="overallBar"></span></div>
    <p><b id="rank">Rango: Aspirante</b> · <b id="points">0 puntos</b></p>
    <h3>Progreso por tema</h3>
    <div id="topicStats" class="grid"></div>
    <div class="row" style="margin-top:16px">
      <button onclick="startPractice()">Practicar por niveles</button>
      <button onclick="startHazmat()">Practicar cartelería</button>
      <button onclick="startExam()">Simulacro final · 70 preguntas</button>
    </div>
    <button class="secondary" onclick="resetProgress()">Borrar progreso</button>
  </section>

  <section id="game" class="card hidden">
    <div class="row"><b id="modeLabel"></b><span id="counter"></span></div>
    <div class="bar"><span id="questionBar"></span></div>
    <h2 id="questionText"></h2>
    <div id="visual"></div>
    <div id="choices"></div>
    <p id="feedback" class="feedback" aria-live="polite"></p>
    <div id="developArea" class="hidden">
      <textarea id="developAnswer" placeholder="Escribí tu respuesta..."></textarea>
      <p class="muted">La respuesta abierta se guarda para esta sesión, pero no se califica automáticamente.</p>
      <button onclick="submitDevelop()">Ver pauta de repaso</button>
    </div>
    <div class="row">
      <button id="nextButton" class="hidden" onclick="nextQuestion()">Siguiente</button>
      <button class="secondary" onclick="goHome()">Volver al inicio</button>
    </div>
  </section>

  <section id="result" class="card hidden">
    <h2 id="resultTitle"></h2>
    <p id="resultText"></p>
    <div id="examReview"></div>
    <button onclick="goHome()">Volver al inicio</button>
  </section>

  <section class="card notice">
    <b>Importante:</b> esta app es material de práctica, no reemplaza capacitación ni protocolos oficiales.
    Para materiales peligrosos, consultá la edición vigente de la GRE. La app no inventa distancias ni
    debe usarse para actuar ante una emergencia real.
  </section>
</main>

<script>
const BANK = [
  {topic:"Socorrismo",level:1,q:"Antes de acercarte a una persona lesionada, ¿qué corresponde hacer?",options:["Comprobar que la escena sea segura","Moverla de inmediato","Darle agua"],answer:0,why:"La seguridad de la escena se evalúa antes de intervenir."},
  {topic:"Socorrismo",level:1,q:"¿Cuál es la diferencia general entre signo y síntoma?",options:["El signo puede observarse o medirse; el síntoma lo refiere la persona","Son exactamente lo mismo","El síntoma siempre se mide con un equipo"],answer:0,why:"Un signo es observable o medible; un síntoma es lo que la persona siente y comunica."},
  {topic:"Socorrismo",level:1,q:"¿Qué describe una hemorragia?",options:["Pérdida de sangre de los vasos sanguíneos","Una lesión de la piel sin sangrado","Una dificultad para respirar"],answer:0,why:"Una hemorragia es la salida de sangre de los vasos sanguíneos."},
  {topic:"Socorrismo",level:2,q:"Ante una emergencia médica, ¿qué guía la atención inicial?",options:["El protocolo y la capacitación recibida","La improvisación","Una única respuesta para todos los casos"],answer:0,why:"Hay que actuar según la formación y los protocolos aplicables."},
  {topic:"Socorrismo",level:2,type:"develop",q:"Explicá por qué es importante evaluar la escena antes de atender a una persona.",reference:"La respuesta debería mencionar seguridad del rescatista, de la persona y de terceros, además de pedir ayuda según el protocolo."},
  {topic:"Socorrismo",level:3,q:"¿Qué debe hacerse si la situación supera tu nivel de capacitación?",options:["Continuar igualmente","Actuar dentro de tu formación y solicitar apoyo adecuado","Retirarte sin avisar"],answer:1,why:"Se debe trabajar dentro de la capacitación recibida y pedir apoyo."},

  {topic:"Organización bomberil",level:1,q:"Ante una emergencia, ¿qué principio general debe priorizar el personal?",options:["Actuar sin evaluar","Seguir la capacitación y los procedimientos establecidos","Separarse del equipo"],answer:1,why:"La actuación debe seguir procedimientos, capacitación y supervisión."},
  {topic:"Organización bomberil",level:1,q:"¿Por qué es importante la comunicación durante una intervención?",options:["Para coordinar tareas y transmitir información relevante","Para reemplazar los procedimientos","Para evitar toda supervisión"],answer:0,why:"La comunicación ayuda a coordinar y compartir información relevante."},
  {topic:"Organización bomberil",level:2,q:"¿Qué debe hacerse si una instrucción no se entiende con claridad?",options:["Adivinar qué significa","Solicitar aclaración por el medio establecido","Ignorarla"],answer:1,why:"La aclaración evita malentendidos y permite actuar coordinadamente."},
  {topic:"Organización bomberil",level:2,type:"develop",q:"Mencioná dos razones por las que el trabajo coordinado es importante en una emergencia.",reference:"Puede incluir seguridad, reparto de tareas, comunicación, control de recursos y seguimiento de la organización estable
```
