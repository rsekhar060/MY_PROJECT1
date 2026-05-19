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
      margin-bottom:25px;
      flex-wrap:wrap;
      gap:10px;
    }

    .top-bar h1{
      font-size:32px;
    }

    input[type=file]{
      background:#101522;
      color:white;
      border:1px solid #2c3447;
      padding:12px;
      border-radius:10px;
    }

    .stats{
      display:grid;
      grid-template-columns:repeat(auto-fit,minmax(220px,1fr));
      gap:15px;
      margin-bottom:30px;
    }

    .stat-card{
      background:#0f131d;
      border:1px solid #232a3b;
      border-radius:16px;
      padding:22px;
    }

    .stat-card h4{
      color:#7f8aa3;
      font-size:12px;
      letter-spacing:1px;
      margin-bottom:10px;
    }

    .stat-card h1{
      font-size:42px;
    }

    .green{
      color:#27e97c;
    }

    .orange{
      color:#ff9d1c;
    }

    .blue{
      color:#4f83ff;
    }

    .po-card{
      background:#08101f;
      border-radius:20px;
      padding:30px;
      margin-bottom:25px;
      border:1px solid #2d3447;
    }

    .clean{
      border:1px solid #00ff84;
    }

    .review{
      border:1px solid #ff9d1c;
    }

    .po-header{
      display:flex;
      justify-content:space-between;
      align-items:center;
      margin-bottom:20px;
      flex-wrap:wrap;
      gap:15px;
    }

    .po-title{
      font-size:42px;
      font-weight:bold;
    }

    .badge{
      padding:10px 18px;
      border-radius:10px;
      font-size:13px;
      font-weight:bold;
      letter-spacing:1px;
    }

    .badge-clean{
      background:#013d20;
      color:#52ff9c;
    }

    .badge-review{
      background:#332000;
      color:#ffbf5e;
    }

    .warning-box{
      background:#2d2100;
      border:1px solid #7f5b00;
      color:#ffd56e;
      padding:15px;
      border-radius:10px;
      margin-top:18px;
      line-height:1.7;
    }

    .values{
      display:grid;
      grid-template-columns:repeat(auto-fit,minmax(150px,1fr));
      gap:25px;
      margin-top:30px;
    }

    .value-box h5{
      color:#8190af;
      margin-bottom:10px;
      font-size:13px;
      letter-spacing:1px;
    }

    .value-box p{
      font-size:24px;
      font-weight:bold;
    }

    .buttons{
      display:flex;
      gap:15px;
      margin-top:35px;
      flex-wrap:wrap;
    }

    button{
      border:none;
      padding:16px 22px;
      border-radius:12px;
      font-size:18px;
      font-weight:bold;
      cursor:pointer;
    }

    .approve{
      background:#4c83ff;
      color:white;
    }

    .flag{
      background:#ffc107;
      color:black;
    }

    .refresh{
      background:#293142;
      color:white;
    }

    .section-title{
      font-size:28px;
      margin-bottom:20px;
    }

  </style>
</head>

<body>

<div class="top-bar">

  <h1>PO Validation Dashboard</h1>

  <input type="file" id="csvFile" accept=".csv"/>

</div>

<div class="stats">

  <div class="stat-card">
    <h4>TOTAL ROWS</h4>
    <h1 id="totalRows">0</h1>
  </div>

  <div class="stat-card">
    <h4>CLEAN</h4>
    <h1 class="green" id="cleanCount">0</h1>
  </div>

  <div class="stat-card">
    <h4>NEEDS REVIEW</h4>
    <h1 class="orange" id="reviewCount">0</h1>
  </div>

</div>

<h2 class="section-title">Validation Results</h2>

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

    const tpMatch =
      poTP.toFixed(2) === ucTP.toFixed(2);

    const mrpMatch =
      poMRP.toFixed(2) === ucMRP.toFixed(2);

    const gstMatch =
      portalGST.toFixed(2) === ucGST.toFixed(2);

    const isClean =
      tpMatch && mrpMatch && gstMatch;

    if(isClean){
      cleanCount++;
    }else{
      reviewCount++;
    }

    const poID =
      row.PO_ID ||
      row.PO ||
      row.PO_NUMBER ||
      row.NO_PO ||
      `PO_${index+1}`;

    const card = document.createElement('div');

    card.className =
      `po-card ${isClean ? 'clean' : 'review'}`;

    card.innerHTML = `

      <div class="po-header">

        <div class="po-title">
          ${poID}
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

          ${!tpMatch ? 'PO_TP mismatch detected.<br>' : ''}
          ${!mrpMatch ? 'PO_MRP mismatch detected.<br>' : ''}
          ${!gstMatch ? 'GST mismatch detected.' : ''}

        </div>
        `
        :
        ''
      }

      <div class="values">

        <div class="value-box">
          <h5>PO TP</h5>
          <p>₹${poTP.toFixed(2)}</p>
        </div>

        <div class="value-box">
          <h5>UC TP</h5>
          <p>₹${ucTP.toFixed(2)}</p>
        </div>

        <div class="value-box">
          <h5>PO MRP</h5>
          <p>₹${poMRP.toFixed(2)}</p>
        </div>

        <div class="value-box">
          <h5>UC MRP</h5>
          <p>₹${ucMRP.toFixed(2)}</p>
        </div>

        <div class="value-box">
          <h5>PORTAL GST</h5>
          <p>${portalGST}%</p>
        </div>

        <div class="value-box">
          <h5>UC GST</h5>
          <p>${ucGST}%</p>
        </div>

      </div>

      <div class="buttons">

        ${
          isClean
          ?
          `
          <button class="approve">
            Approve — Invoice at ₹${ucTP.toFixed(2)}
          </button>
          `
          :
          `
          <button class="flag">
            NEEDS REVIEW
          </button>
          `
        }

        <button class="refresh">
          TP Refresh
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
