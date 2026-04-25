
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Portal de Resultados Académicos I.E. La Esperanza</title>
    <script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <style>
        :root {
            --primario: #0d47a1; /* Azul Marino */
            --acento: #2979ff; /* Azul Brillante */
            --fondo: #f0f4f8;
            --sobresaliente: #d4edda; --sobresaliente-t: #155724;
            --normal: #fff3cd; --normal-t: #856404;
            --bajo: #ffe0b3; --bajo-t: #d35400;
            --critico: #f8d7da; --critico-t: #721c24;
        }

        body { font-family: 'Segoe UI', sans-serif; background: var(--fondo); margin: 0; padding: 0; color: #333; }
        
        /* ==============================
           PANTALLA DE CARGA (INTERFAZ LLAMATIVA) 
           ============================== */
        .admin-screen {
            height: 100vh; display: flex; flex-direction: column; justify-content: center; align-items: center;
            background: linear-gradient(135deg, #0d47a1 0%, #1976d2 100%); color: white;
        }
        
        .upload-box { 
            background: white; padding: 50px; border-radius: 25px; 
            color: #333; text-align: center; 
            box-shadow: 0 15px 35px rgba(0,0,0,0.3);
            border: 4px solid rgba(255,255,255,0.3);
            transition: transform 0.3s ease;
        }
        .upload-box:hover { transform: translateY(-5px); }
        
        .upload-icon {
            font-size: 60px; color: var(--acento); margin-bottom: 20px;
            display: inline-block; animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.1); opacity: 0.8; }
            100% { transform: scale(1); opacity: 1; }
        }

        .custom-file-input {
            display: inline-block; padding: 15px 35px; background: var(--acento); 
            color: white; border-radius: 50px; cursor: pointer; 
            font-weight: bold; font-size: 1.1em; margin-top: 25px; 
            transition: background 0.3s, box-shadow 0.3s;
            box-shadow: 0 4px 15px rgba(41, 121, 255, 0.4);
        }
        .custom-file-input:hover { background: #1565c0; box-shadow: 0 6px 20px rgba(41, 121, 255, 0.6); }

        /* ==============================
           DASHBOARD PRINCIPAL 
           ============================== */
        .dashboard { display: none; padding: 20px; }
        .container { max-width: 1300px; margin: auto; }
        
        .header-nav { 
            display: flex; justify-content: space-between; align-items: center; 
            background: white; padding: 20px 30px; border-radius: 15px; margin-bottom: 25px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.05);
        }

        .btn { padding: 12px 24px; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; color: white; transition: 0.3s; display: flex; align-items: center; gap: 8px; }
        .btn-view { background: #673ab7; }
        .btn-view:hover { background: #512da8; }
        .btn-pdf { background: #c62828; }
        .btn-pdf:hover { background: #b71c1c; }

        .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(190px, 1fr)); gap: 15px; margin-bottom: 25px; }
        .stat-card { background: white; padding: 25px; border-radius: 15px; text-align: center; border-bottom: 6px solid var(--primario); box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
        .stat-card h3 { margin: 0; font-size: 0.85em; color: #666; text-transform: uppercase; letter-spacing: 1px; }
        .stat-card .valor { font-size: 2.2em; font-weight: bold; color: var(--primario); margin: 10px 0 5px; }
        .stat-card .desv { font-size: 0.9em; color: #d32f2f; background: #fff1f0; padding: 3px 10px; border-radius: 10px; font-weight: 600; display: inline-block; }

        .charts-row { display: grid; grid-template-columns: 1fr 1.3fr; gap: 20px; margin-bottom: 25px; }
        .chart-box { background: white; padding: 25px; border-radius: 15px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }

        /* BUSCADOR INDIVIDUAL */
        .search-area { background: var(--primario); color: white; padding: 40px; border-radius: 20px; margin-bottom: 25px; text-align: center; box-shadow: 0 4px 15px rgba(0,0,0,0.2); }
        #searchBox { width: 85%; padding: 15px; border-radius: 10px; border: none; font-size: 1.2em; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .res-indiv { display: none; background: white; color: #333; padding: 25px; border-radius: 15px; margin-top: 25px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }

        /* TABLA RANKING */
        .table-card { background: white; padding: 25px; border-radius: 15px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; font-size: 0.95em; }
        th { background: #f1f3f5; padding: 15px; text-align: center; font-size: 0.85em; text-transform: uppercase; }
        td { padding: 15px; text-align: center; border-bottom: 1px solid #eee; }

        .badge { padding: 6px 14px; border-radius: 20px; font-weight: bold; font-size: 0.8em; text-transform: uppercase; border: 1px solid currentColor; }

        /* MODO SOLO VISUALIZACIÓN */
        body.view-only .admin-controls { display: none !important; }
        body.view-only .header-nav { background: #eee; }
    </style>
</head>
<body>

    <!-- PÁGINA DE CARGA LLAMATIVA -->
    <div id="adminPanel" class="admin-screen">
        <div class="upload-box">
            <div class="upload-icon">📄</div>
            <h1 style="margin:0 0 10px;">Portal de Simulacros</h1>
            <p style="color:#555;">I.E. La Esperanza - Resultados Finales 2026</p>
            <label class="custom-file-input">
                <input type="file" id="fileIn" accept=".xlsx" style="display:none;" />
                🔗 Adjuntar Resultados (Excel)
            </label>
        </div>
    </div>

    <!-- DASHBOARD DE VISUALIZACIÓN -->
    <div id="dashboard" class="dashboard">
        <div class="container">
            <div class="header-nav">
                <div>
                    <h2 style="margin:0; color:var(--primario)">Dashboard de Rendimiento Académico</h2>
                    <small id="fechaActual"></small>
                </div>
                <div class="admin-controls" style="display:flex; gap:10px;">
                    <button onclick="activarVisualizacion()" class="btn btn-view">👁️ Modo Lectura</button>
                    <button onclick="descargarPDF()" class="btn btn-pdf">📕 PDF</button>
                </div>
            </div>

            <div class="stats-grid" id="statsGrid"></div>

            <div class="charts-row">
                <div class="chart-box"><canvas id="pieChart"></canvas></div>
                <div class="chart-box"><canvas id="barChart"></canvas></div>
            </div>

            <div class="search-area">
                <h3>🔍 Consulta de Resultados Individuales</h3>
                <p style="margin-bottom:20px;">Escribe tu nombre completo tal como aparece en la lista</p>
                <input type="text" id="searchBox" placeholder="Ej: LUIS CARLOS MENDOZA..." oninput="buscarEstudiante()">
                <div id="cardIndiv" class="res-indiv">
                    <h2 id="nomIndiv" style="color:var(--primario);"></h2>
                    <div id="scoresIndiv" style="display:flex; justify-content:space-around; font-size:1.3em; margin:20px 0; font-weight:500;"></div>
                    <h3 id="globalIndiv" style="background:var(--fondo); padding:10px; border-radius:10px; display:inline-block;"></h3>
                </div>
            </div>

            <div class="table-card">
                <h3>🏆 Ranking General (Mayor a Menor)</h3>
                <table>
                    <thead>
                        <tr><th>Puesto</th><th>Estudiante</th><th>Lectura</th><th>Mates</th><th>Soc.</th><th>Cie.</th><th>Ing.</th><th>Global</th><th>Nivel</th></tr>
                    </thead>
                    <tbody id="tableBody"></tbody>
                </table>
            </div>
        </div>
    </div>

<script>
    let dataSets = [];
    let charts = {};

    document.getElementById('fileIn').addEventListener('change', function(e) {
        const reader = new FileReader();
        reader.onload = (evt) => {
            const wb = XLSX.read(evt.target.result, {type:'array'});
            const json = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]], {header:1});
            procesar(json);
        };
        reader.readAsArrayBuffer(e.target.files[0]);
    });

    function clasificar(p) {
        if (p >= 300) return { n: "Sobresaliente", c: "row-sobresaliente", cl: "--sobresaliente", tl: "--sobresaliente-t" };
        if (p >= 250) return { n: "Normal", c: "row-normal", cl: "--normal", tl: "--normal-t" };
        if (p >= 200) return { n: "Bajo", c: "row-bajo", cl: "--bajo", tl: "--bajo-t" };
        return { n: "Crítico", c: "row-critico", cl: "--critico", tl: "--critico-t" };
    }

    function procesar(rows) {
        let list = [];
        rows.forEach(r => {
            // Estructura: Nombre (1), Lec (2), Mat (5), Soc (8), Cie (11), Ing (12), Global (13)
            if(r.length >= 14 && !isNaN(parseFloat(r[13]))) {
                const g = parseFloat(r[13]);
                list.push({n:r[1], l:parseFloat(r[2])||0, m:parseFloat(r[5])||0, s:parseFloat(r[8])||0, c:parseFloat(r[11])||0, i:parseFloat(r[12])||0, g:g, info:clasificar(g)});
            }
        });
        list.sort((a,b) => b.g - a.g);
        dataSets = list;

        renderStats();
        renderCharts();
        renderTable();

        document.getElementById('adminPanel').style.display = 'none';
        document.getElementById('dashboard').style.display = 'block';
        document.getElementById('fechaActual').innerText = "Generado el: " + new Date().toLocaleDateString();
    }

    function renderStats() {
        const keys = ['l','m','s','c','i','g'];
        const labs = ['Lectura','Matemáticas','Sociales','Ciencias','Inglés','Puntaje Global'];
        let html = '';
        keys.forEach((k, i) => {
            const vals = dataSets.map(e => e[k]);
            const avg = (vals.reduce((a,b)=>a+b,0)/vals.length).toFixed(1);
            const sd = Math.sqrt(vals.map(x => Math.pow(x-avg,2)).reduce((a,b)=>a+b,0)/vals.length).toFixed(2);
            html += `<div class="stat-card"><h3>${labs[i]}</h3><div class="valor">${avg}</div><span class="desv">&sigma;: ${sd}</span></div>`;
        });
        document.getElementById('statsGrid').innerHTML = html;
    }

    function renderCharts() {
        const nivs = {"Sobresaliente":0, "Normal":0, "Bajo":0, "Crítico":0};
        dataSets.forEach(e => nivs[e.info.n]++);
        
        charts.p = new Chart(document.getElementById('pieChart'), {
            type:'doughnut', 
            data:{labels:Object.keys(nivs), datasets:[{data:Object.values(nivs), backgroundColor:['#2e7d32','#fbc02d','#ef6c00','#c62828']}]},
            options:{plugins:{title:{display:true, text:'Distribución de Desempeño'}}}
        });

        const avgs = ['l','m','s','c','i'].map(k => (dataSets.reduce((a,b)=>a+b[k],0)/dataSets.length).toFixed(1));
        charts.b = new Chart(document.getElementById('barChart'), {
            type:'bar',
            data:{
                labels:['Lectura','Mates','Sociales','Ciencias','Inglés'], 
                datasets:[{
                    label: 'Media por Área',
                    data:avgs, 
                    // COLORES DIFERENCIADOS PARA CADA BARRA
                    backgroundColor: [
                        '#42a5f5', // Azul Lec
                        '#66bb6a', // Verde Mat
                        '#ffa726', // Naranja Soc
                        '#26a69a', // Turquesa Cie
                        '#ab47bc'  // Púrpura Ing
                    ],
                    borderRadius: 8
                }]
            },
            options:{ scales: { y: { beginAtZero:true, max:100 } }, plugins:{legend:{display:false}, title:{display:true, text:'Comparativo de Áreas'}}}
        });
    }

    function renderTable() {
        document.getElementById('tableBody').innerHTML = dataSets.map((e, i) => `
            <tr class="${e.info.c}">
                <td><strong>#${i+1}</strong></td>
                <td style="text-align:left">${e.n}</td>
                <td>${e.l}</td><td>${e.m}</td><td>${e.s}</td><td>${e.c}</td><td>${e.i}</td>
                <td><strong>${e.g.toFixed(1)}</strong></td>
                <td><span class="badge" style="background:var(${e.info.cl}); color:var(${e.info.tl})">${e.info.n}</span></td>
            </tr>
        `).join('');
    }

    function buscarEstudiante() {
        const q = document.getElementById('searchBox').value.toUpperCase();
        const res = dataSets.find(e => e.n.toUpperCase().includes(q));
        const card = document.getElementById('cardIndiv');
        if(q.length > 2 && res) {
            card.style.display = 'block';
            document.getElementById('nomIndiv').innerText = res.n;
            document.getElementById('scoresIndiv').innerHTML = `<span>Lec: ${res.l}</span><span>Mat: ${res.m}</span><span>Soc: ${res.s}</span><span>Cie: ${res.c}</span><span>Ing: ${res.i}</span>`;
            document.getElementById('globalIndiv').innerText = `Global: ${res.g.toFixed(1)} (${res.info.n})`;
        } else { card.style.display = 'none'; }
    }

    function activarVisualizacion() {
        if(confirm("¿Activar modo de solo lectura? Se ocultarán los controles de administración para presentar o compartir.")) {
            document.body.classList.add('view-only');
        }
    }

    function descargarPDF() {
        const el = document.getElementById('dashboard');
        html2pdf().from(el).set({margin:0.2, filename:'Reporte_I.E._Esperanza.pdf', html2canvas:{scale:2}, jsPDF:{orientation:'landscape'}}).save();
    }
</script>
</body>
</html>
