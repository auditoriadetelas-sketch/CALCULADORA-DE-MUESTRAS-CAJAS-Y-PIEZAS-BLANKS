<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Calculadora de Muestras de Auditoría</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <style>
        body { font-family: 'Segoe UI', Arial, sans-serif; background: #f4f7f6; padding: 20px; }
        .container { max-width: 900px; margin: auto; background: white; padding: 30px; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        #reporte-area { padding: 10px; }
        h1 { color: #1e293b; text-align: center; margin-bottom: 5px; }
        .subtitle { text-align: center; color: #64748b; margin-bottom: 30px; }
        
        .resumen-muestreo { 
            display: flex; 
            justify-content: space-around; 
            background: #1e293b; 
            color: white; 
            padding: 20px; 
            border-radius: 8px; 
            margin-bottom: 25px;
        }
        .dato-muestreo { text-align: center; }
        .dato-muestreo span { display: block; font-size: 2.2em; font-weight: bold; color: #fbbf24; }
        .dato-muestreo label { font-size: 0.8em; text-transform: uppercase; opacity: 0.8; }

        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #e2e8f0; padding: 12px; text-align: center; }
        th { background: #f8fafc; color: #475569; font-size: 0.85em; text-transform: uppercase; }
        
        .muestra-valor { font-weight: bold; color: #2563eb; font-size: 1.1em; }
        input { width: 90%; padding: 8px; border: 1px solid #cbd5e1; border-radius: 4px; text-align: center; }
        
        .no-print { margin-top: 25px; display: flex; gap: 10px; }
        .btn { padding: 12px 24px; cursor: pointer; border: none; border-radius: 6px; font-weight: bold; flex: 1; transition: 0.2s; }
        .btn-add { background: #10b981; color: white; }
        .btn-pdf { background: #ef4444; color: white; }
        .btn-calc { background: #2563eb; color: white; }
        .btn-remove { background: #fee2e2; color: #ef4444; padding: 5px 10px; border-radius: 4px; }
    </style>
</head>
<body>

<div class="container">
    <div id="reporte-area">
        <h1>Muestra de Auditoría</h1>
        <p class="subtitle">Planificación basada en tabla de muestreo estándar</p>
        
        <div class="resumen-muestreo">
            <div class="dato-muestreo">
                <label>Total Muestra Piezas</label>
                <span id="mTotalPiezas">0</span>
            </div>
            <div class="dato-muestreo">
                <label>Total Muestra Cajas</label>
                <span id="mTotalCajas">0</span>
            </div>
        </div>

        <table id="tablaColores">
            <thead>
                <tr>
                    <th>Color / Estilo</th>
                    <th>Cant. Cajas</th>
                    <th>Cant. Piezas</th>
                    <th>Muestra Cajas</th>
                    <th>Muestra Piezas</th>
                    <th class="accion-col"></th>
                </tr>
            </thead>
            <tbody></tbody>
            <tfoot>
                <tr style="background: #f1f5f9; font-weight: bold;">
                    <td>TOTALES</td>
                    <td id="tCajas">0</td>
                    <td id="tPiezas">0</td>
                    <td colspan="3"></td>
                </tr>
            </tfoot>
        </table>
    </div>

    <div class="no-print">
        <button class="btn btn-add" onclick="agregarFila()">+ Agregar Color</button>
        <button class="btn btn-calc" onclick="calcular()">Calcular Muestra</button>
        <button class="btn btn-pdf" onclick="descargarPDF()">Descargar PDF</button>
    </div>
</div>

<script>
    const tablaMuestreo = [
        { min: 10001, max: 35000, muestra: 315 },
        { min: 3201, max: 10000, muestra: 200 },
        { min: 1201, max: 3200, muestra: 125 },
        { min: 501, max: 1200, muestra: 80 },
        { min: 281, max: 500, muestra: 50 },
        { min: 151, max: 280, muestra: 32 },
        { min: 91, max: 150, muestra: 20 },
        { min: 51, max: 90, muestra: 13 },
        { min: 26, max: 50, muestra: 8 },
        { min: 16, max: 25, muestra: 5 },
        { min: 9, max: 15, muestra: 3 },
        { min: 2, max: 8, muestra: 2 }
    ];

    function obtenerMuestra(n) {
        const rango = tablaMuestreo.find(r => n >= r.min && n <= r.max);
        return rango ? rango.muestra : (n > 35000 ? 500 : 0);
    }

    function agregarFila() {
        const tbody = document.querySelector("#tablaColores tbody");
        const tr = document.createElement("tr");
        tr.innerHTML = `
            <td><input type="text" placeholder="..." class="c-nom"></td>
            <td><input type="number" value="0" class="c-caj" oninput="sumar()"></td>
            <td><input type="number" value="0" class="c-pie" oninput="sumar()"></td>
            <td class="muestra-valor res-c">0</td>
            <td class="muestra-valor res-p">0</td>
            <td class="accion-col"><button class="btn-remove" onclick="this.parentElement.parentElement.remove(); sumar();">X</button></td>
        `;
        tbody.appendChild(tr);
    }

    function sumar() {
        let tc = 0, tp = 0;
        document.querySelectorAll('.c-caj').forEach(i => tc += parseInt(i.value || 0));
        document.querySelectorAll('.c-pie').forEach(i => tp += parseInt(i.value || 0));
        document.getElementById('tCajas').innerText = tc;
        document.getElementById('tPiezas').innerText = tp;
    }

    function calcular() {
        const tc = parseInt(document.getElementById('tCajas').innerText);
        const tp = parseInt(document.getElementById('tPiezas').innerText);
        
        const mTC = obtenerMuestra(tc);
        const mTP = obtenerMuestra(tp);

        document.getElementById('mTotalCajas').innerText = mTC;
        document.getElementById('mTotalPiezas').innerText = mTP;

        document.querySelectorAll("#tablaColores tbody tr").forEach(fila => {
            const c = parseInt(fila.querySelector('.c-caj').value || 0);
            const p = parseInt(fila.querySelector('.c-pie').value || 0);
            fila.querySelector('.res-c').innerText = tc > 0 ? Math.round((c/tc)*mTC) : 0;
            fila.querySelector('.res-p').innerText = tp > 0 ? Math.round((p/tp)*mTP) : 0;
        });
    }

    function descargarPDF() {
        document.querySelectorAll('.accion-col').forEach(el => el.style.display = 'none');
        const element = document.getElementById('reporte-area');
        html2pdf().set({
            margin: 10,
            filename: 'Muestra_Auditoria.pdf',
            html2canvas: { scale: 2 },
            jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
        }).from(element).save().then(() => {
            document.querySelectorAll('.accion-col').forEach(el => el.style.display = 'table-cell');
        });
    }

    agregarFila();
</script>
</body>
</html>
