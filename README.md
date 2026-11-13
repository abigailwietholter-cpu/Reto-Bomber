# Reto-Bomber
Preguntas variadas sobre temas bomberiles, con un examen final
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Reto Bombero</title>
<style>
  * { box-sizing: border-box; }
  body { margin: 0; background: #f2f4f7; color: #222; font: 16px/1.5 Arial, sans-serif; }
  header { padding: 18px 5%; background: #681313; color: white; }
  header h1, header p { margin: 0; }
  main { width: min(900px, 94%); margin: 22px auto; }
  .card { padding: 20px; margin-bottom: 16px; background: white; border-radius: 14px; box-shadow: 0 3px 14px #0001; }
  button, textarea { font: inherit; border-radius: 9px; padding: 11px 13px; }
  button { margin: 4px; border: 0; color: white; background: #9e1b1b; cursor: pointer; font-weight: bold; }
  button:hover { background: #681313; }
  button.secondary { color: #222; background: #e8eaed; }
  button.answer { display: block; width: 100%; text-align: left; color: #222; background: white; border: 2px solid #ddd; }
  button.answer:hover { border-color: #9e1b1b; }
  .correct { background: #e6f5ea !important; border-color: #278442 !important; }
  .wrong { background: #ffebeb !important; border-color: #b3261e !important; }
  .hidden { display: none !important; }
  .bar { height: 12px; margin: 10px 0; overflow: hidden; background: #e5e7eb; border-radius: 99px; }
  .bar span { display: block; width: 0; height: 100%; background: #e0a322; transition: width .2s; }
  .grid { display: grid; grid-template-columns: repeat(auto-fit,minmax(180px,1fr)); gap: 9px; }
  .topic { padding: 10px; background: #f6f6f6; border-radius: 9px; }
  .row { display: flex; flex-wrap: wrap; gap: 8px; align-items: center; }
  .row button { flex: 1; }
  .feedback { min-height: 28px; font-weight: bold; }
  .muted { color: #5d6268; }
  textarea { width: 100%; min-height: 110px; border: 1px solid #aaa; }
  .notice { border-left: 5px solid #e0a322; background: #fff8e5; }
  .placard { width: min(260px,90%); margin: 14px auto; padding: 10px; text-align: center; font-weight: bold; color: #111; border: 3px solid #222; background: #f28c28; }
  .placard hr { border: 1px solid #222; }
  .diamond { position: relative; width: 132px; height: 132px; margin: 25px auto; transform: rotate(45deg); }
  .diamond div { position: absolute; display: grid; width: 50%; height: 50%; place-items: center; border: 1px solid white; font-size: 1.4rem; font-weight: bold; }
  .diamond span { transform: rotate(-45deg); }
  .health { top: 0; left: 0; color: white; background: #1769aa; }
  .fire { top: 0; right: 0; color: white; background: #c62828; }
  .instability { right: 0; bottom: 0; color: #222; background: #e6bd28; }
  .special { bottom: 0; left: 0; color: #222; background: white; }
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
      <p class="muted">Las respuestas abiertas no se califican automáticamente.</p>
      <button onclick="submitDevelop()">Guardar y ver pauta</button>
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
    <b>Importante:</b> material de práctica; no reemplaza capacitación ni protocolos oficiales.
    Consultá la edición vigente de la GRE para identificar sus secciones y las distancias aplicables.
    Esta app no calcula ni inventa distancias y no debe usarse para actuar ante una emergencia real.
  </section>
</main>

<script>
/* Banco de práctica: verificá y ampliá las preguntas con los manuales.
   En opciones múltiples, answer es el índice correcto empezando desde 0.
   Las de desarrollo llevan type:"develop" y una pauta en reference. */
const BANK = [
  {topic:"Socorrismo",level:1,q:"Antes de acercarte a una persona lesionada, ¿qué corresponde hacer?",options:["Comprobar que la escena sea segura","Moverla de inmediato","Darle agua"],answer:0,why:"La seguridad de la escena se evalúa antes de intervenir."},
  {topic:"Socorrismo",level:1,q:"¿Cuál es la diferencia general entre signo y síntoma?",options:["El signo puede observarse o medirse; el síntoma lo refiere la persona","Son exactamente lo mismo","El síntoma siempre se mide con un equipo"],answer:0,why:"Un signo es observable o medible; un síntoma es lo que la persona siente y comunica."},
  {topic:"Socorrismo",level:1,q:"¿Qué describe una hemorragia?",options:["Pérdida de sangre de los vasos sanguíneos","Una lesión sin sangrado","Una dificultad para respirar"],answer:0,why:"Una hemorragia es la salida de sangre de los vasos sanguíneos."},
  {topic:"Socorrismo",level:2,q:"Ante una emergencia médica, ¿qué guía la atención inicial?",options:["El protocolo y la capacitación recibida","La improvisación","Una única respuesta para todos los casos"],answer:0,why:"Hay que actuar según la formación y los protocolos aplicables."},
  {topic:"Socorrismo",level:2,type:"develop",q:"Explicá por qué es importante evaluar la escena antes de atender a una persona.",reference:"La respuesta debería mencionar seguridad del rescatista, de la persona y de terceros, además de pedir ayuda según el protocolo."},
  {topic:"Socorrismo",level:3,q:"¿Qué debe hacerse si la situación supera tu nivel de capacitación?",options:["Continuar igualmente","Actuar dentro de tu formación y solicitar apoyo adecuado","Retirarte sin avisar"],answer:1,why:"Se debe trabajar dentro de la capacitación recibida y pedir apoyo."},

  {topic:"Organización bomberil",level:1,q:"Ante una emergencia, ¿qué principio general debe priorizar el personal?",options:["Actuar sin evaluar","Seguir la capacitación y los procedimientos establecidos","Separarse del equipo"],answer:1,why:"La actuación debe seguir procedimientos, capacitación y supervisión."},
  {topic:"Organización bomberil",level:1,q:"¿Por qué es importante la comunicación durante una intervención?",options:["Para coordinar tareas y transmitir información relevante","Para reemplazar los procedimientos","Para evitar toda supervisión"],answer:0,why:"La comunicación ayuda a coordinar y compartir información relevante."}
  {topic:"Organización bomberil",level:2,q:"¿Qué debe hacerse si una instrucción no se entiende con claridad?",options:["Adivinar qué significa","Solicitar aclaración por el medio establecido","Ignorarla"],answer:1,why:"Pedir aclaración ayuda a evitar malentendidos."},
{topic:"Organización bomberil",level:2,type:"develop",q:"Mencioná dos razones por las que el trabajo coordinado es importante en una emergencia.",reference:"La respuesta puede incluir seguridad, reparto de tareas, comunicación y coordinación."},
{topic:"Organización bomberil",level:3,q:"Si una tarea excede tu capacitación, ¿qué corresponde hacer?",options:["Intentarla igual","Informar la situación y pedir apoyo según el procedimiento","Dejar el puesto sin avisar"],answer:1,why:"Se debe actuar dentro de la capacitación y comunicar la necesidad de apoyo."},

{topic:"Incendios forestales",level:1,q:"¿Qué factor puede cambiar rápidamente el comportamiento de un incendio forestal?",options:["El viento","El color del uniforme","La matrícula del vehículo"],answer:0,why:"El viento puede cambiar la dirección y el comportamiento del fuego."},
{topic:"Incendios forestales",level:1,q:"¿Qué elementos forman el triángulo del fuego?",options:["Combustible, calor y oxígeno","Agua, humo y ceniza","Viento, lluvia y suelo"],answer:0,why:"El triángulo del fuego incluye combustible, calor y oxígeno."},
{topic:"Incendios forestales",level:2,q:"¿Por qué se debe observar continuamente el entorno durante un incendio forestal?",options:["Porque las condiciones pueden cambiar","Para evitar usar comunicaciones","Para identificar el color de los vehículos"],answer:0,why:"El viento, el terreno y otros factores pueden cambiar las condiciones."},
{topic:"Incendios forestales",level:2,type:"develop",q:"Mencioná factores del entorno que pueden influir en el comportamiento de un incendio forestal.",reference:"La respuesta puede mencionar viento, combustible, terreno y condiciones ambientales, según el manual."},
{topic:"Incendios forestales",level:3,q:"¿Qué debe guiar la elección de una acción durante una intervención?",options:["La evaluación del riesgo y los procedimientos establecidos","La rapidez por sí sola","La preferencia personal"],answer:0,why:"La seguridad y los procedimientos deben orientar la decisión."},

{topic:"Rescate con cuerdas",level:1,q:"¿Qué debe hacerse antes de usar un sistema de cuerdas?",options:["Revisar el equipo y seguir el procedimiento","Usarlo si parece firme","Usar cualquier cuerda disponible"],answer:0,why:"El sistema y el equipo deben revisarse según el procedimiento y la capacitación."},
{topic:"Rescate con cuerdas",level:1,q:"¿Por qué es importante revisar el equipo antes y después del uso?",options:["Para detectar daños o problemas","Para cambiar su color","Para evitar registrar su uso"],answer:0,why:"La revisión ayuda a identificar daños y seguir las reglas de mantenimiento."},
{topic:"Rescate con cuerdas",level:2,q:"¿Qué se debe hacer si se detecta un daño en un equipo?",options:["Retirarlo del servicio y seguir el procedimiento correspondiente","Usarlo con más cuidado","Ocultar el daño"],answer:0,why:"Un equipo dañado debe manejarse según el protocolo."},
{topic:"Rescate con cuerdas",level:2,type:"develop",q:"Explicá por qué el rescate con cuerdas debe realizarse con capacitación y supervisión adecuadas.",reference:"La respuesta debería mencionar los riesgos, la revisión del sistema, los roles y el cumplimiento de procedimientos."},
{topic:"Rescate con cuerdas",level:3,q:"¿Qué corresponde hacer si no se puede confirmar que el sistema esté correctamente preparado?",options:["Continuar para ahorrar tiempo","Detenerse y solicitar revisión de personal competente","Añadir cualquier equipo disponible"],answer:1,why:"No se debe continuar hasta resolver la duda según los procedimientos y la supervisión."},

{topic:"Materiales peligrosos",level:1,q:"¿Qué dato del cartel naranja puede ayudar a identificar un material en la GRE?",options:["El número ONU","La matrícula del vehículo","El nombre de la empresa"],answer:0,why:"El número ONU puede servir para buscar la entrada correspondiente en la guía."},
{topic:"Materiales peligrosos",level:1,q:"En el rombo NFPA, ¿qué representa el color rojo?",options:["Inflamabilidad","Salud","Riesgo especial"],answer:0,why:"El rojo representa inflamabilidad."},
{topic:"Materiales peligrosos",level:1,q:"En el rombo NFPA, ¿qué representa el color azul?",options:["Salud","Inflamabilidad","Inestabilidad"],answer:0,why:"El azul representa el riesgo para la salud."},
{topic:"Materiales peligrosos",level:1,q:"En el rombo NFPA, ¿qué representa el color amarillo?",options:["Inestabilidad o reactividad","Salud","Inflamabilidad"],answer:0,why:"El amarillo representa inestabilidad o reactividad."},
{topic:"Materiales peligrosos",level:1,q:"¿Qué representa el espacio blanco del rombo NFPA?",options:["Información o riesgo especial","El número ONU","La distancia de evacuación"],answer:0,why:"El espacio blanco se utiliza para indicar información o riesgos especiales."},
{topic:"Materiales peligrosos",level:2,q:"¿En qué sección de la GRE se busca inicialmente un material por número ONU?",options:["Índice amarillo","Índice azul","Sección naranja"],answer:0,why:"El índice amarillo permite buscar por número ONU; después se consulta la guía indicada."},
{topic:"Materiales peligrosos",level:2,q:"¿En qué sección de la GRE se busca inicialmente un material por nombre?",options:["Índice azul","Índice amarillo","Sección naranja"],answer:0,why:"El índice azul permite buscar por nombre del material."},
{topic:"Materiales peligrosos",level:2,q:"¿Qué contiene principalmente la sección naranja de la GRE?",options:["Guías de respuesta","Nombres de empresas","Puntajes NFPA"],answer:0,why:"La sección naranja contiene guías de respuesta que deben leerse para el caso correspondiente."},
{topic:"Materiales peligrosos",level:2,q:"¿Para qué se consulta la sección verde de la GRE?",options:["Para revisar distancias de aislamiento y protección de ciertos materiales","Para buscar el nombre del conductor","Para conocer el puntaje del simulacro"],answer:0,why:"La sección verde contiene tablas de distancias para determinados materiales; hay que consultar la entrada aplicable."},
{topic:"Materiales peligrosos",level:2,q:"¿El rombo NFPA indica por sí solo las distancias de aislamiento de la GRE?",options:["Sí, siempre","No; las distancias se consultan en la GRE correspondiente","Sí, si el rombo es naranja"],answer:1,why:"El rombo NFPA y las tablas de distancia de la GRE tienen usos distintos."},
{topic:"Materiales peligrosos",level:2,type:"develop",q:"Describí cómo buscarías un producto en la GRE usando el número ONU, sin inventar una distancia.",reference:"La respuesta debería indicar buscar el número en el índice amarillo, seguir la referencia a la guía correspondiente y revisar las secciones indicadas, incluida la verde si aplica. En una emergencia real se siguen los protocolos y las instrucciones del personal especializado."},
{topic:"Materiales peligrosos",level:3,q:"Si el número ONU no aparece en el cartel o no se puede leer con seguridad, ¿qué corresponde hacer?",options:["Adivinar el producto","Seguir el procedimiento de identificación y solicitar apoyo especializado","Elegir cualquier guía"],answer:1,why:"No se debe adivinar: hay que seguir los procedimientos y pedir apoyo competente."}
];
  <script>
const STORAGE_KEY = "retoBomberoProgresoV1";
let stats = loadStats();
let questions = [];
let current = 0;
let mode = "";
let examCorrect = 0;
let examAnswered = 0;
let examMisses = [];

function loadStats() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || {
      points: 0, answered: 0, correct: 0, topics: {}
    };
  } catch {
    return { points: 0, answered: 0, correct: 0, topics: {} };
  }
}

function saveStats() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(stats));
  } catch (error) {
    console.warn("No se pudo guardar el progreso.", error);
  }
}

function escapeHTML(value) {
  return String(value).replace(/[&<>"']/g, char => ({
    "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;"
  })[char]);
}

function rankName(points) {
  if (points >= 500) return "Rango: Bombero Máximo";
  if (points >= 300) return "Rango: Bombero Experto";
  if (points >= 150) return "Rango: Bombero Avanzado";
  if (points >= 50) return "Rango: Bombero en formación";
  return "Rango: Aspirante";
}

function updateHome() {
  const percent = stats.answered
    ? Math.round(stats.correct / stats.answered * 100)
    : 0;

  document.getElementById("overall").textContent = stats.answered
    ? `Respuestas correctas: ${stats.correct} de ${stats.answered} (${percent}%).`
    : "Todavía no hay respuestas registradas.";

  document.getElementById("overallBar").style.width = percent + "%";
  document.getElementById("rank").textContent = rankName(stats.points);
  document.getElementById("points").textContent = `${stats.points} puntos`;

  const topics = [...new Set(BANK.map(question => question.topic))];

  document.getElementById("topicStats").innerHTML = topics.map(topic => {
    const data = stats.topics[topic] || { answered: 0, correct: 0 };
    const topicPercent = data.answered
      ? Math.round(data.correct / data.answered * 100)
      : 0;

    return `<div class="topic">
      <b>${escapeHTML(topic)}</b><br>
      ${data.correct}/${data.answered} correctas (${topicPercent}%)
    </div>`;
  }).join("");
}

function showSection(sectionId) {
  ["home", "game", "result"].forEach(id => {
    document.getElementById(id).classList.toggle("hidden", id !== sectionId);
  });
}

function shuffle(items) {
  const copy = [...items];

  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }

  return copy;
}

function startPractice() {
  mode = "practice";
  const level = Math.min(3, 1 + Math.floor(stats.points / 50));
  questions = shuffle(BANK.filter(question => question.level <= level));
  current = 0;
  examCorrect = 0;
  examAnswered = 0;
  examMisses = [];
  showSection("game");
  renderQuestion();
}

function startHazmat() {
  mode = "hazmat";
  questions = shuffle(
    BANK.filter(question => question.topic === "Materiales peligrosos")
  );
  current = 0;
  examCorrect = 0;
  examAnswered = 0;
  examMisses = [];
  showSection("game");
  renderQuestion();
}

function startExam() {
  mode = "exam";
  const pool = shuffle(BANK);
  questions = [];

  while (questions.length < 70) {
    questions.push(pool[questions.length % pool.length]);
  }

  questions = shuffle(questions);
  current = 0;
  examCorrect = 0;
  examAnswered = 0;
  examMisses = [];
  showSection("game");
  renderQuestion();
}

function renderVisual(question) {
  const visual = document.getElementById("visual");
  visual.innerHTML = "";

  if (question.topic !== "Materiales peligrosos") return;

  const text = question.q.toLowerCase();

  if (text.includes("rombo") || text.includes("color")) {
    visual.innerHTML = `
      <div class="diamond" aria-label="Rombo NFPA de ejemplo">
        <div class="health"><span>2</span></div>
        <div class="fire"><span>3</span></div>
        <div class="instability"><span>1</span></div>
        <div class="special"><span>—</span></div>
      </div>
      <p class="muted" style="text-align:center">
        Ejemplo didáctico: estos valores no identifican un incidente real.
      </p>`;
  } else if (text.includes("cartel naranja") || text.includes("número onu")) {
    visual.innerHTML = `
      <div class="placard" aria-label="Cartel naranja de ejemplo">
        <div>30</div><hr><div>1202</div>
      </div>
      <p class="muted" style="text-align:center">
        Ejemplo didáctico. Consultá el número y la guía en la edición de GRE que estés usando.
      </p>`;
  }
}

function renderQuestion() {
  if (current >= questions.length) {
    finishSession();
    return;
  }

  const question = questions[current];
  const modeNames = {
    practice: "Práctica por niveles",
    hazmat: "Práctica de materiales peligrosos",
    exam: "Simulacro final"
  };

  document.getElementById("modeLabel").textContent = modeNames[mode];
  document.getElementById("counter").textContent =
    `Pregunta ${current + 1} de ${questions.length}`;
  document.getElementById("questionBar").style.width =
    `${((current + 1) / questions.length) * 100}%`;
  document.getElementById("questionText").textContent = question.q;
  document.getElementById("feedback").textContent = "";
  document.getElementById("choices").innerHTML = "";
  document.getElementById("developAnswer").value = "";
  document.getElementById("developArea")
    .classList.toggle("hidden", question.type !== "develop");
  document.getElementById("nextButton").classList.add("hidden");

  renderVisual(question);

  if (question.type === "develop") return;

  question.options.forEach((option, index) => {
    const button = document.createElement("button");
    button.className = "answer";
    button.textContent = option;
    button.onclick = () => chooseAnswer(index);
    document.getElementById("choices").appendChild(button);
  });
}

function recordAnswer(question, isCorrect) {
  stats.answered++;

  if (isCorrect) {
    stats.correct++;
    stats.points += 10;
  }

  if (!stats.topics[question.topic]) {
    stats.topics[question.topic] = { answered: 0, correct: 0 };
  }

  stats.topics[question.topic].answered++;

  if (isCorrect) {
    stats.topics[question.topic].correct++;
  }

  saveStats();
}

function chooseAnswer(selectedIndex) {
  const question = questions[current];
  const buttons = [...document.querySelectorAll("#choices button")];
  buttons.forEach(button => button.disabled = true);

  const isCorrect = selectedIndex === question.answer;

  if (buttons[question.answer]) {
    buttons[question.answer].classList.add("correct");
  }
  if (!isCorrect && buttons[selectedIndex]) {
    buttons[selectedIndex].classList.add("wrong");
  }

  recordAnswer(question, isCorrect);
  examAnswered++;

  if (isCorrect) {
    examCorrect++;
  } else {
    examMisses.push(question);
  }

  document.getElementById("feedback").textContent =
    (isCorrect ? "¡Correcto! " : "No era esa. ") + question.why;
  document.getElementById("nextButton").classList.remove("hidden");
}

function submitDevelop() {
  const question = questions[current];
  const answer = document.getElementById("developAnswer").value.trim();

  if (!answer) {
    document.getElementById("feedback").textContent =
      "Escribí una respuesta antes de continuar.";
    return;
  }

  recordAnswer(question, false);
  examAnswered++;
  examMisses.push(question);

  document.getElementById("feedback").textContent =
    "Pauta de repaso: " + question.reference;
  document.getElementById("developArea").classList.add("hidden");
  document.getElementById("nextButton").classList.remove("hidden");
}

function nextQuestion() {
  current++;
  renderQuestion();
}

function finishSession() {
  showSection("result");

  const isExam = mode === "exam";
  document.getElementById("resultTitle").textContent =
    isExam ? "Resultado del simulacro" : "Práctica terminada";

  document.getElementById("resultText").textContent = isExam
    ? `Respuestas de opción múltiple correctas: ${examCorrect}. ` +
      `Preguntas abiertas realizadas: ${examAnswered - examCorrect}. ` +
      `Puntos ganados: ${examCorrect * 10}.`
    : `Terminaste ${questions.length} preguntas. ` +
      `Sumaste ${examCorrect * 10} puntos en esta sesión.`;

  const review = document.getElementById("examReview");
  review.innerHTML = "";

  if (examMisses.length > 0) {
    const heading = document.createElement("h3");
    heading.textContent = "Temas para repasar";
    review.appendChild(heading);

    const list = document.createElement("ul");
    [...new Set(examMisses.map(question => question.topic))].forEach(topic => {
      const item = document.createElement("li");
      item.textContent = topic;
      list.appendChild(item);
    });
    review.appendChild(list);
  }

  updateHome();
}

function goHome() {
  updateHome();
  showSection("home");
}

function resetProgress() {
  if (!confirm("¿Querés borrar el progreso guardado en este navegador?")) return;

  stats = { points: 0, answered: 0, correct: 0, topics: {} };
  saveStats();
  updateHome();
}

updateHome();
</script>
</body>
</html>
