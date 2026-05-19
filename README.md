<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>PO Validation Dashboard</title>

  <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>

  <style>
    *{
      margin:0;
      padding:0;
      box-sizing:border-box;
      font-family:Arial, sans-serif;
    }

    body{
      background:#05070d;
      color:white;
      padding:20px;
    }

    .top-bar{
      display:flex;
      justify-content:space-between;
      align-items:center;
      margin-bottom:20px;
      flex-wrap:wrap;
      gap:10px;
    }

    .upload-box{
      display:flex;
      gap:10px;
      align-items:center;
    }

    input[type=file]{
      color:white;
    }

    .stats{
      display:grid;
      grid-template-columns:repeat(auto-fit,minmax(220px,1fr));
      gap:15px;
      margin-bottom:25px;
    }

    .card{
      background:#0f131d;
      border:1px solid #232a3b;
      border-radius:14px;
      padding:20px;
    }

    .card h3{
      color:#8d98b3;
      font-size:13px;
      margin-bottom:10px;
      letter-spacing:1px;
    }

    .card h1{
      font-size:40px;
    }

    .green{
      color:#24e36c;
    }

    .orange{
      color:#ff8c1a;
    }

    .blue{
      color:#4c7dff;
    }

    .po-card{
      background:#0d111b;
      border:1px solid #2c3447;
      border-radius:18px;
      padding:25px;
      margin-bottom:20px;
    }

    .review{
      border:1px solid #ff8c1a;
    }

    .clean{
      border:1px solid #24e36c;
    }

    .po-header{
      display:flex;
      justify-content:space-between;
      align-items:center;
      margin-bottom:20px;
      flex-wrap:wrap;
      gap:10px;
    }

    .badge{
      padding:8px 14px;
      border-radius:8px;
      font-size:12px;
      font-weight:bold;
      letter-spacing:1px;
    }

    .badge-review{
      background:#2a1a00;
      color:#ffb347;
    }

    .badge-clean{
      background:#032e14;
      color:#4dff9a;
    }

    .warning-box{
      background:#2c1d00;
      border:1px solid #6f4700;
      padding:12px;
      border-radius:8px;
      margin-top:15px;
      color:#ffcc66;
      font-size:14px;
    }

    .values{
      display:flex;
      gap:50px;
      margin-top:20px;
      flex-wrap:wrap;
    }

    .value-block h4{
      color:#7f8aa3;
      margin-bottom:8px;
      font-size:12px;
    }

    .value-block p{
      font-size:28px;
      font-weight:bold;
    }

    .buttons{
      display:flex;
      gap:10px;
      margin-top:25px;
      flex-wrap:wrap;
    }

    button{
      border:none;
      padding:14px 20px;
      border-radius:10px;
      cursor:pointer;
      font-weight:bold;
    }

    .approve{
      background:#3f7cff;
      color:white;
    }

    .flag{
      background:#ffc107;
      color:black;
    }

    .refresh{
      background:#2a2f3a;
      color:white;
    }

    .section-title{
      margin-bottom:20px;
      font-size:28px;
    }
  </style>
</head>

<body>

<div class="top-bar">
  <h1>PO Validation Dashboard</h1>

  <div class="upload-box">
    <input type="file" id="csvFile" accept=".csv"/>
  </div>
</div>

<div class="stats">

  <div class="card">
    <h3>TOTAL ROWS</h3>
    <h1 id="totalRows">0</h1>
  </div>

  <div class="card">
    <h3>CLEAN</h3>
    <h1 class="green" id="cleanCount">0</h1>
  </div>

  <div class="card">
    <h3>NEEDS REVIEW</h3>
    <h1 class="orange" id="reviewCount">0</h1>
  </div>

</div>

<h2 class="section-title">PO Validation Results</h2>

<div id="dashboard"></div>

<script>

document.getElementById('csvFile').addEventListener('change', handleFile);

function handleFile(event){

  const file = event.target.files[0];

  Papa.parse(file,{
    header:true,
    skipEmptyLines:true,

    complete:function(results){

      generateDashboard(results.data);

    }
  });
}

function generateDashboard(data){

  const dashboard = document.getElementById('dashboard');

  dashboard.innerHTML = "";

  let cleanCount = 0;
  let reviewCount = 0;

  data.forEach((row,index)=>{

    const poTP = Number(row.PO_TP || 0);
    const ucTP = Number(row.UC_TP || 0);

    const poMRP = Number(row.PO_MRP || 0);
    const ucMRP = Number(row.UC_MRP || 0);

    const portalGST = Number(row.Portal_Gst || 0);
    const ucGST = Number(row.UC_GST || 0);

    const tpMatch = poTP.toFixed(2) === ucTP.toFixed(2);
    const mrpMatch = poMRP.toFixed(2) === ucMRP.toFixed(2);
    const gstMatch = portalGST.toFixed(2) === ucGST.toFixed(2);

    const isClean = tpMatch && mrpMatch && gstMatch;

    if(isClean){
      cleanCount++;
    }else{
      reviewCount++;
    }

    const card = document.createElement('div');

    card.className = `po-card ${isClean ? 'clean' : 'review'}`;

    card.innerHTML = `

      <div class="po-header">

        <div>
          <h2>${row.PO_ID || 'NO_PO'}</h2>
        </div>

        <div class="badge ${isClean ? 'badge-clean' : 'badge-review'}">
          ${isClean ? 'CLEAN' : 'NEEDS REVIEW'}
        </div>

      </div>

      ${
        !isClean
        ?
        `
        <div class="warning-box">
          PO_TP vs UC_TP OR PO_MRP vs UC_MRP OR GST mismatch detected.
        </div>
        `
        :
        ''
      }

      <div class="values">

        <div class="value-block">
          <h4>PO TP</h4>
          <p>₹${poTP}</p>
        </div>

        <div class="value-block">
          <h4>UC TP</h4>
          <p>₹${ucTP}</p>
        </div>

        <div class="value-block">
          <h4>PO MRP</h4>
          <p>₹${poMRP}</p>
        </div>

        <div class="value-block">
          <h4>UC MRP</h4>
          <p>₹${ucMRP}</p>
        </div>

        <div class="value-block">
          <h4>PORTAL GST</h4>
          <p>${portalGST}%</p>
        </div>

        <div class="value-block">
          <h4>UC GST</h4>
          <p>${ucGST}%</p>
        </div>

      </div>

      <div class="buttons">

        <button class="approve">
          Approve
        </button>

        <button class="flag">
          Flag Platform
        </button>

        <button class="refresh">
          Refresh
        </button>

      </div>

    `;

    dashboard.appendChild(card);

  });

  document.getElementById('totalRows').innerText = data.length;
  document.getElementById('cleanCount').innerText = cleanCount;
  document.getElementById('reviewCount').innerText = reviewCount;

}

</script>

</body>
</html>
