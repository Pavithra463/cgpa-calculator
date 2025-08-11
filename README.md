# cgpa-calculator

//html

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>CGPA Calculator</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div class="container">
    <h1>🎓 Interactive CGPA Calculator</h1>
    <button id="themeToggle">Toggle Theme</button>
    <form id="gpaForm">
      <div id="semesters"></div>
      <button type="button" id="addSemester">+ Add Semester</button>
      <button type="submit">Calculate CGPA</button>
    </form>
    <h2 id="result">CGPA: --</h2>
    <p id="feedback"></p>
    <canvas id="cgpaChart" width="600" height="300"></canvas>
    <button onclick="downloadReport()">Download Report</button>
    <button onclick="resetAll()">Reset</button>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="script.js"></script>
</body>
</html>





//style.css

body {
  font-family: Arial, sans-serif;
  background-color: var(--bg);
  color: var(--text);
  transition: background 0.3s, color 0.3s;
  padding: 20px;
}

.container {
  max-width: 800px;
  margin: auto;
  text-align: center;
}

form input {
  padding: 5px;
  margin: 5px;
  width: 80px;
}

button {
  padding: 10px;
  margin: 10px;
  cursor: pointer;
}

/* Theme variables */
:root {
  --bg: #f0f0f0;
  --text: #333;
}

body.dark {
  --bg: #121212;
  --text: #f0f0f0;
}

body.neon {
  --bg: #0f0f33;
  --text: #39ff14;
}



//script


let semesterCount = 0;
let cgpaChart;
let chartData = [];

const semestersDiv = document.getElementById("semesters");
const form = document.getElementById("gpaForm");
const result = document.getElementById("result");
const feedback = document.getElementById("feedback");

function addSemester(gpa = "", credits = "") {
  semesterCount++;
  const div = document.createElement("div");
  div.innerHTML = `
    <label>Semester ${semesterCount}: GPA <input type="number" step="0.01" min="0" max="10" value="${gpa}" class="gpa"> Credits <input type="number" min="1" value="${credits}" class="credits"></label>
  `;
  semestersDiv.appendChild(div);
}

document.getElementById("addSemester").addEventListener("click", () => {
  addSemester();
});

form.addEventListener("submit", (e) => {
  e.preventDefault();
  const gpas = document.querySelectorAll(".gpa");
  const credits = document.querySelectorAll(".credits");

  let totalPoints = 0;
  let totalCredits = 0;
  chartData = [];

  for (let i = 0; i < gpas.length; i++) {
    const g = parseFloat(gpas[i].value);
    const c = parseFloat(credits[i].value);

    if (!isNaN(g) && !isNaN(c)) {
      totalPoints += g * c;
      totalCredits += c;
      chartData.push({ x: i + 1, y: totalPoints / totalCredits });
    }
  }

  const cgpa = (totalPoints / totalCredits).toFixed(2);
  result.innerText = `CGPA: ${isNaN(cgpa) ? "--" : cgpa}`;

  giveFeedback(cgpa);
  updateChart();
  saveData();
});

function giveFeedback(cgpa) {
  if (cgpa >= 9) feedback.innerText = "🌟 Excellent work!";
  else if (cgpa >= 8) feedback.innerText = "👍 Great! Keep it up.";
  else if (cgpa >= 7) feedback.innerText = "🙂 Good, aim higher!";
  else feedback.innerText = "📘 Time to focus and improve!";
}

function updateChart() {
  const ctx = document.getElementById("cgpaChart").getContext("2d");
  if (cgpaChart) cgpaChart.destroy();

  cgpaChart = new Chart(ctx, {
    type: "line",
    data: {
      datasets: [{
        label: 'CGPA Progress',
        data: chartData,
        borderColor: 'blue',
        backgroundColor: 'lightblue',
        fill: false,
      }]
    },
    options: {
      scales: {
        x: { title: { display: true, text: 'Semester' } },
        y: { title: { display: true, text: 'CGPA' }, min: 0, max: 10 }
      }
    }
  });
}

function resetAll() {
  semestersDiv.innerHTML = '';
  semesterCount = 0;
  addSemester();
  result.innerText = "CGPA: --";
  feedback.innerText = "";
  if (cgpaChart) cgpaChart.destroy();
  localStorage.clear();
}

function downloadReport() {
  const text = `CGPA Report\n-----------------\n${result.innerText}\n${feedback.innerText}`;
  const blob = new Blob([text], { type: "text/plain" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "cgpa_report.txt";
  a.click();
}

function toggleTheme() {
  const body = document.body;
  if (body.classList.contains("dark")) {
    body.classList.remove("dark");
    body.classList.add("neon");
  } else if (body.classList.contains("neon")) {
    body.classList.remove("neon");
  } else {
    body.classList.add("dark");
  }
}

document.getElementById("themeToggle").addEventListener("click", toggleTheme);

function saveData() {
  const data = [];
  document.querySelectorAll("#semesters div").forEach((div) => {
    const gpa = div.querySelector(".gpa").value;
    const credits = div.querySelector(".credits").value;
    data.push({ gpa, credits });
  });
  localStorage.setItem("semesters", JSON.stringify(data));
}

function loadData() {
  const saved = JSON.parse(localStorage.getItem("semesters"));
  if (saved && saved.length) {
    saved.forEach((s) => addSemester(s.gpa, s.credits));
  } else {
    addSemester();
  }
}

loadData();
