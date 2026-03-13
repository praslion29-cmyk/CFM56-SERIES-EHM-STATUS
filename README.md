# CFM56-SERIES-EHM-STATUS

<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>EHM Skywise Compact</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body { font-family: Arial, sans-serif; background-color: #f4f4f9; margin: 20px; }
h1 { color: #004080; }
table { width: 100%; border-collapse: collapse; margin-top: 20px; }
th, td { border: 1px solid #ccc; padding: 8px; text-align: center; font-size: 14px; }
th { background-color: #004080; color: #fff; }
.warning { background-color: #ffcccc; color: #b30000; font-weight: bold; }
.caution { background-color: #fff3cd; color: #856404; font-weight: bold; }
.normal { background-color: #ccffcc; color: #006600; font-weight: bold; }
.form-section { margin-bottom: 20px; }
label { display: inline-block; width: 150px; font-size: 14px; }
.print-section { margin-top: 20px; text-align: center; }
.print-section button { padding: 8px 16px; font-size: 14px; background: #004080; color: #fff; border: none; border-radius: 5px; cursor: pointer; }
.print-section button:hover { background: #0066cc; }
canvas { margin-top: 15px; background: #fff; padding: 5px; border: 1px solid #ccc; }
</style>
</head>
<body>

<h1>📊 Engine Health Monitoring (EHM) - Compact Skywise Mode</h1>

<div class="form-section">
<h2>Input Data Cycle</h2>
<form id="cycleForm">
<label>Tanggal:</label><input type="date" id="date"><br><br>
<label>Aircraft Reg:</label><input type="text" id="reg"><br><br>
<label>Engine Number:</label><input type="text" id="engine"><br><br>
<label>EGT (°C):</label><input type="number" id="egt"><br><br>
<label>N1 (%):</label><input type="number" id="n1"><br><br>
<label>N2 (%):</label><input type="number" id="n2"><br><br>
<label>Oil Pressure (psi):</label><input type="number" id="oilP"><br><br>
<label>Oil Temp (°C):</label><input type="number" id="oilT"><br><br>
<label>Vibration (IPS):</label><input type="number" id="vib"><br><br>
<label>Fuel Flow (ffph):</label><input type="number" id="fuel"><br><br>
<button type="button" onclick="addCycle()">Tambah Cycle</button>
</form>
</div>

<table id="ehmTable">
<tr>
<th>Tanggal</th><th>Aircraft Reg</th><th>Engine No</th><th>Parameter</th>
<th>Batas Atas</th><th>Nilai Aktual</th><th>Possible Cause</th><th>Maintenance Recommendation</th>
<th>Health Score</th>
</tr>
</table>

<h2>Grafik Tren Parameter</h2>
<canvas id="egtChart" width="400" height="200"></canvas>
<canvas id="oilChart" width="400" height="200"></canvas>
<canvas id="vibChart" width="400" height="200"></canvas>

<div class="print-section">
<button onclick="window.print()">🖨️ Print Laporan</button>
</div>

<script>
let cycles=[];
let egtChart, oilChart, vibChart;

function addCycle(){
  const date=document.getElementById("date").value;
  const reg=document.getElementById("reg").value;
  const engine=document.getElementById("engine").value;
  const egt=parseFloat(document.getElementById("egt").value);
  const n1=parseFloat(document.getElementById("n1").value);
  const n2=parseFloat(document.getElementById("n2").value);
  const oilP=parseFloat(document.getElementById("oilP").value);
  const oilT=parseFloat(document.getElementById("oilT").value);
  const vib=parseFloat(document.getElementById("vib").value);
  const fuel=parseFloat(document.getElementById("fuel").value);

  if(!date||!reg||!engine){ alert("Tanggal, Aircraft Reg, dan Engine wajib diisi"); return; }

  const newCycle={date,reg,engine,egt,n1,n2,oilP,oilT,vib,fuel};
  cycles.push(newCycle);

  updateTable(newCycle);
  updateCharts();
  document.getElementById("cycleForm").reset();
}

// ---------- Engine Analysis ----------
function analyzeEngine(cycle){
  let alerts=[], status="NORMAL", recommendation="-";
  let score=100;

  if(cycle.egt>=950){alerts.push("EGT Overlimit"); status="WARNING"; recommendation="Inspect turbine & fuel nozzle"; score-=30;}
  else if(cycle.egt>=900){alerts.push("EGT High"); status="CAUTION"; recommendation="Check combustion efficiency"; score-=15;}

  if(cycle.oilP<25){alerts.push("Low Oil Pressure"); status="WARNING"; recommendation="Check oil pump"; score-=25;}
  else if(cycle.oilP<30){if(status!=="WARNING"){status="CAUTION"; score-=10;}}

  if(cycle.vib>=4){alerts.push("High Engine Vibration"); status="WARNING"; recommendation="Rotor balancing"; score-=20;}
  else if(cycle.vib>=3.5){if(status!=="WARNING"){status="CAUTION"; score-=10;}}

  if(cycle.fuel>5160){alerts.push("Fuel Burn Inefficient"); if(status!=="WARNING"){status="CAUTION"; recommendation="Inspect fuel nozzle"; score-=10;}}

  if(alerts.length>0){ setTimeout(()=>{ alert(`⚠️ ALERT ${cycle.reg} - Engine ${cycle.engine}\n${alerts.join("\n")}`); },100); }

  return {alerts,status,recommendation,score};
}

// ---------- Update Table ----------
function updateTable(cycle){
  const analysis=analyzeEngine(cycle);
  const table=document.getElementById("ehmTable");

  function addRow(param, limit, value){
    const row=table.insertRow();
    row.insertCell(0).innerText=cycle.date;
    row.insertCell(1).innerText=cycle.reg;
    row.insertCell(2).innerText=cycle.engine;
    row.insertCell(3).innerText=param;
    row.insertCell(4).innerText=limit;
    const valCell=row.insertCell(5); valCell.innerText=value;

    // Only show Possible Cause & Recommendation if status caution/warning
    const causeCell=row.insertCell(6);
    const recCell=row.insertCell(7);
    if(analysis.status==="NORMAL"){ causeCell.innerText="-"; recCell.innerText="-"; }
    else{
      if(param==="EGT") { causeCell.innerText="Turbine degradation"; recCell.innerText="Inspect fuel nozzle & turbine"; }
      else if(param==="Oil Pressure"){ causeCell.innerText="Lubrication leak"; recCell.innerText="Check oil pump"; }
      else if(param==="Vibration"){ causeCell.innerText="Rotor imbalance"; recCell.innerText="Rotor balancing"; }
      else if(param==="Fuel Flow"){ causeCell.innerText="Combustion inefficiency"; recCell.innerText="Check fuel nozzle"; }
    }

    const scoreCell=row.insertCell(8); scoreCell.innerText=analysis.score;

    if(param==="EGT"){ if(value>=950) valCell.classList.add("warning"); else if(value>=900) valCell.classList.add("caution"); else valCell.classList.add("normal"); }
    if(param==="Oil Pressure"){ if(value<25) valCell.classList.add("warning"); else if(value<30) valCell.classList.add("caution"); else valCell.classList.add("normal"); }
    if(param==="Vibration"){ if(value>=4) valCell.classList.add("warning"); else if(value>=3.5) valCell.classList.add("caution"); else valCell.classList.add("normal"); }
    if(param==="Fuel Flow"){ if(value>5160) valCell.classList.add("warning"); else if(value>=4800) valCell.classList.add("caution"); else valCell.classList.add("normal"); }
  }

  addRow("EGT","950 °C",cycle.egt);
  addRow("Oil Pressure",">25 psi",cycle.oilP);
  addRow("Vibration","<4 IPS",cycle.vib);
  addRow("Fuel Flow","5160 ffph",cycle.fuel);

  // Engine Status Row
  const row=table.insertRow();
  row.insertCell(0).innerText=cycle.date;
  row.insertCell(1).innerText=cycle.reg;
  row.insertCell(2).innerText=cycle.engine;
  row.insertCell(3).innerText="ENGINE STATUS";
  row.insertCell(4).innerText="-";
  const statusCell=row.insertCell(5); statusCell.innerText=analysis.status;
  row.insertCell(6).innerText=analysis.alerts.join(", ")||"-";
  row.insertCell(7).innerText=analysis.recommendation;
  const scoreCell=row.insertCell(8); scoreCell.innerText=analysis.score;
  if(analysis.status==="WARNING") statusCell.classList.add("warning");
  else if(analysis.status==="CAUTION") statusCell.classList.add("caution");
  else statusCell.classList.add("normal");
}

// ---------- Update Charts ----------
function updateCharts(){
  const labels=cycles.map((c,i)=> c.date||("Cycle "+(i+1)));
  const egtData=cycles.map(c=>c.egt);
  const oilData=cycles.map(c=>c.oilP);
  const vibData=cycles.map(c=>c.vib);

  function createOrUpdate(chart,canvasId,label,data,color){
    if(!chart){ return new Chart(document.getElementById(canvasId),{type:"line",data:{labels,datasets:[{label,data,borderColor:color,fill:false}]},options:{plugins:{legend:{display:true}},scales:{y:{beginAtZero:true}}}});}
    chart.data.labels=labels; chart.data.datasets[0].data=data; chart.update(); return chart;
  }

  egtChart=createOrUpdate(egtChart,"egtChart","EGT (°C)",egtData,"red");
  oilChart=createOrUpdate(oilChart,"oilChart","Oil Pressure (psi)",oilData,"green");
  vibChart=createOrUpdate(vibChart,"vibChart","Vibration (IPS)",vibData,"orange");
}
</script>

