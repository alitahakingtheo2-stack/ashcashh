<!DOCTYPE html>
<html>
<head>
  <title>RLC Circuit Simulator</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    body {
      font-family: Arial;
      background: #eef2f3;
      padding: 20px;
    }

    .box {
      background: white;
      padding: 20px;
      border-radius: 12px;
      max-width: 1000px;
    }

    input[type=range] {
      width: 300px;
    }

    .row { margin: 10px 0; }

    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
    }

    #resonance {
      font-weight: bold;
      color: green;
    }

    .pulse {
      animation: pulse 1s infinite;
      color: red;
    }

    @keyframes pulse {
      0% { transform: scale(1); }
      50% { transform: scale(1.05); }
      100% { transform: scale(1); }
    }
  </style>
</head>

<body>

<div class="box">

<h2>⚡ RLC Circuit Simulator</h2>

<label>Mode:</label>
<select id="mode" onchange="update()">
  <option value="learn">Learning</option>
  <option value="exam">Exam</option>
</select>

<hr>

<div class="row">R (Ω):
<input type="range" id="R" min="1" max="200" value="50" oninput="update()">
<span id="Rv"></span>
</div>

<div class="row">L (H):
<input type="range" id="L" min="0.01" max="1" step="0.01" value="0.3" oninput="update()">
<span id="Lv"></span>
</div>

<div class="row">C (F):
<input type="range" id="C" min="0.0001" max="0.01" step="0.0001" value="0.0004" oninput="update()">
<span id="Cv"></span>
</div>

<div class="row">Frequency (Hz):
<input type="range" id="f" min="1" max="200" value="60" oninput="update()">
<span id="fv"></span>
</div>

<div class="row">Voltage (V):
<input type="range" id="V" min="1" max="240" value="230" oninput="update()">
<span id="Vv"></span>
</div>

<hr>

<div id="resonance"></div>

<div class="grid">
  <div>
    <h3>📊 Impedance vs Frequency</h3>
    <canvas id="chart"></canvas>
  </div>

  <div>
    <h3>📐 Phasor Diagram</h3>
    <canvas id="phasor" width="300" height="300"></canvas>
  </div>
</div>

<div id="results"></div>

</div>

<script>

let chart;

function calc(R,L,C,f,V){
  let w = 2*Math.PI*f;
  let XL = w*L;
  let XC = 1/(w*C);
  let Z = Math.sqrt(R*R + (XL-XC)*(XL-XC));
  let I = V/Z;
  let f0 = 1/(2*Math.PI*Math.sqrt(L*C));
  return {Z,XL,XC,I,f0};
}

function graph(R,L,C){
  let labels=[], data=[];
  for(let f=1; f<=200; f+=2){
    let w=2*Math.PI*f;
    let XL=w*L;
    let XC=1/(w*C);
    let Z=Math.sqrt(R*R+(XL-XC)*(XL-XC));
    labels.push(f);
    data.push(Z);
  }
  return {labels,data};
}

function phasor(R,L,C,f){
  let ctx=document.getElementById("phasor").getContext("2d");
  ctx.clearRect(0,0,300,300);

  let w=2*Math.PI*f;
  let XL=w*L;
  let XC=1/(w*C);

  ctx.save();
  ctx.translate(150,150);

  ctx.beginPath();
  ctx.moveTo(-120,0);
  ctx.lineTo(120,0);
  ctx.moveTo(0,-120);
  ctx.lineTo(0,120);
  ctx.strokeStyle="#ddd";
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(0,0);
  ctx.lineTo(100,0);
  ctx.strokeStyle="black";
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(0,0);
  ctx.lineTo(0,-XL*0.2);
  ctx.strokeStyle="blue";
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(0,0);
  ctx.lineTo(0,XC*0.2);
  ctx.strokeStyle="red";
  ctx.stroke();

  ctx.restore();
}

function update(){

  let R=+Rrange.value,
      L=+Lrange.value,
      C=+Crange.value,
      f=+Frange.value,
      V=+Vrange.value;

  Rv.innerText=R;
  Lv.innerText=L;
  Cv.innerText=C;
  fv.innerText=f;
  Vv.innerText=V;

  let o=calc(R,L,C,f,V);
  let phi=Math.atan((o.XL-o.XC)/R)*180/Math.PI;

  let mode=document.getElementById("mode").value;

  results.innerHTML = mode==="learn"
  ? `Z=${o.Z.toFixed(2)} Ω | I=${o.I.toFixed(2)} A | φ=${phi.toFixed(2)}° | f₀=${o.f0.toFixed(2)} Hz`
  : `Z=${o.Z.toFixed(2)} Ω | I=${o.I.toFixed(2)} A`;

  let res=document.getElementById("resonance");

  if(Math.abs(o.XL-o.XC)<1){
    res.innerText="⚡ RESONANCE";
    res.classList.add("pulse");
  } else {
    res.innerText="";
    res.classList.remove("pulse");
  }

  let g=graph(R,L,C);
  if(chart) chart.destroy();

  chart=new Chart(document.getElementById("chart"),{
    type:"line",
    data:{labels:g.labels,datasets:[{data:g.data,label:"Z"}]}
  });

  phasor(R,L,C,f);
}

// init refs
let Rrange=document.getElementById("R");
let Lrange=document.getElementById("L");
let Crange=document.getElementById("C");
let Frange=document.getElementById("f");
let Vrange=document.getElementById("V");

let Rv=document.getElementById("Rv");
let Lv=document.getElementById("Lv");
let Cv=document.getElementById("Cv");
let fv=document.getElementById("fv");
let Vv=document.getElementById("Vv");

let results=document.getElementById("results");

update();

</script>

</body>
</html>
