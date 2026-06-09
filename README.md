<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HKDSE Economics Minimum Wage Simulator</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background-color: #f5f7fa; color: #333; padding: 20px; display: flex; justify-content: center; }
        .container { max-width: 800px; width: 100%; background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
        h1 { color: #1e293b; margin-top: 0; }
        .control-panel { background: #f8fafc; border: 1px solid #e2e8f0; padding: 20px; border-radius: 8px; margin-bottom: 20px; display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        label { font-weight: bold; display: block; margin-bottom: 8px; color: #475569; }
        input[type="range"] { width: 100%; }
        select { width: 100%; padding: 8px; border-radius: 4px; border: 1px solid #cbd5e1; font-size: 16px; }
        .badge { display: inline-block; padding: 6px 12px; border-radius: 20px; font-weight: bold; margin-bottom: 15px; }
        .effective { background-color: #fee2e2; color: #991b1b; }
        .ineffective { background-color: #f1f5f9; color: #475569; }
        .stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; margin-bottom: 20px; }
        .stat-card { background: #f1f5f9; padding: 15px; border-radius: 8px; text-align: center; border-left: 4px solid #cbd5e1; }
        .stat-card.gain { border-left-color: #10b981; }
        .stat-card.loss { border-left-color: #ef4444; }
        .stat-num { font-size: 24px; font-weight: bold; margin-top: 5px; color: #0f172a; }
        canvas { background: #ffffff; border: 1px solid #e2e8f0; width: 100%; height: auto; display: block; border-radius: 8px; }
    </style>
</head>
<body>

<div class="container">
    <h1>HKDSE Economics Minimum Wage Simulator</h1>
    <p>Adjust variables to see how a price floor affects labor markets based on HKDSE syllabus conditions.</p>
    
    <div class="control-panel">
        <div>
            <label for="wageSlider">Minimum Wage Rate: $<span id="wageVal">46</span></label>
            <input type="range" id="wageSlider" min="30" max="60" value="46" step="0.5">
        </div>
        <div>
            <label for="elasticitySelect">Elasticity of Demand for Labor</label>
            <select id="elasticitySelect">
                <option value="inelastic" selected>Inelastic Demand (Firms don't cut many jobs)</option>
                <option value="elastic">Elastic Demand (Firms cut many jobs)</option>
            </select>
        </div>
    </div>

    <div id="statusBadge" class="badge effective">Effective Minimum Wage (Price Floor)</div>

    <div class="stats-grid">
        <div class="stat-card">
            <div>Actual Employment</div>
            <div class="stat-num" id="statEmp">92.8</div>
        </div>
        <div class="stat-card">
            <div>Unemployment / Surplus</div>
            <div class="stat-num" id="statUnemp">19.2</div>
        </div>
        <div class="stat-card" id="earningsCard">
            <div>Total Wage Earnings</div>
            <div class="stat-num" id="statEarnings">$4,268</div>
        </div>
    </div>

    <canvas id="marketGraph" width="600" height="400"></canvas>
</div>

<script>
    const canvas = document.getElementById('marketGraph');
    const ctx = canvas.getContext('2d');
    const wageSlider = document.getElementById('wageSlider');
    const wageVal = document.getElementById('wageVal');
    const elasticitySelect = document.getElementById('elasticitySelect');
    const statusBadge = document.getElementById('statusBadge');
    const statEmp = document.getElementById('statEmp');
    const statUnemp = document.getElementById('statUnemp');
    const statEarnings = document.getElementById('statEarnings');
    const earningsCard = document.getElementById('earningsCard');

    function drawMarket() {
        const W_floor = parseFloat(wageSlider.value);
        const elasticity = elasticitySelect.value;
        wageVal.innerText = W_floor;

        const isEffective = W_floor > 40;
        
        let Qd = 100;
        let Qs = 100;
        
        if (isEffective) {
            const slope = elasticity === 'inelastic' ? 1.2 : 3.5;
            Qd = 100 - slope * (W_floor - 40);
            Qs = 100 + 2 * (W_floor - 40);
        }

        const actualEmp = isEffective ? Qd : 100;
        const surplus = isEffective ? (Qs - Qd) : 0;
        const totalEarnings = (isEffective ? W_floor : 40) * actualEmp;

        statEmp.innerText = actualEmp.toFixed(1);
        statUnemp.innerText = surplus.toFixed(1);
        statEarnings.innerText = '$' + Math.round(totalEarnings);
        
        if (isEffective) {
            statusBadge.innerText = "Effective Minimum Wage (Price Floor)";
            statusBadge.className = "badge effective";
            if (totalEarnings > 4000) {
                earningsCard.className = "stat-card gain";
            } else {
                earningsCard.className = "stat-card loss";
            }
        } else {
            statusBadge.innerText = "Ineffective Minimum Wage (No Market Impact)";
            statusBadge.className = "badge ineffective";
            earningsCard.className = "stat-card";
        }

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        const padding = 60;
        const graphWidth = canvas.width - padding * 2;
        const graphHeight = canvas.height - padding * 2;

        function getX(q) { return padding + (q / 150) * graphWidth; }
        function getY(w) { return canvas.height - padding - (w / 80) * graphHeight; }

        // Axes
        ctx.beginPath();
        ctx.strokeStyle = '#334155';
        ctx.lineWidth = 2;
        ctx.moveTo(padding, padding);
        ctx.lineTo(padding, canvas.height - padding);
        ctx.lineTo(canvas.width - padding, canvas.height - padding);
        ctx.stroke();

        // Labels
        ctx.fillStyle = '#334155';
        ctx.font = '14px sans-serif';
        ctx.fillText('Wage Rate ($)', padding - 50, padding - 10);
        ctx.fillText('Quantity of Labor (Q)', canvas.width - padding - 80, canvas.height - padding + 40);
        ctx.fillText('0', padding - 15, canvas.height - padding + 15);

        // Revenue Change Shading
        if (isEffective) {
            // Gain Rectangle
            ctx.fillStyle = 'rgba(16, 185, 129, 0.15)';
            ctx.fillRect(getX(0), getY(W_floor), getX(Qd) - getX(0), getY(40) - getY(W_floor));
            
            // Loss Rectangle
            ctx.fillStyle = 'rgba(239, 68, 68, 0.15)';
            ctx.fillRect(getX(Qd), getY(40), getX(100) - getX(Qd), getY(0) - getY(40));
        }

        // Demand Curve (D)
        ctx.beginPath();
        ctx.strokeStyle = '#2563eb';
        ctx.lineWidth = 3;
        if (elasticity === 'inelastic') {
            ctx.moveTo(getX(52), getY(80));
            ctx.lineTo(getX(148), getY(0));
        } else {
            ctx.moveTo(getX(0), getY(51.4));
            ctx.lineTo(getX(150), getY(22.8));
        }
        ctx.stroke();
        ctx.fillText('D', canvas.width - padding - 20, elasticity === 'inelastic' ? getY(10) : getY(20));

        // Supply Curve (S)
        ctx.beginPath();
        ctx.strokeStyle = '#ea580c';
        ctx.lineWidth = 3;
        ctx.moveTo(getX(60), getY(20));
        ctx.lineTo(getX(140), getY(60));
        ctx.stroke();
        ctx.fillText('S', getX(140) + 5, getY(60));

        // Baseline Equilibrium lines
        ctx.setLineDash([4, 4]);
        ctx.strokeStyle = '#94a3b8';
        ctx.lineWidth = 1;
        ctx.beginPath(); ctx.moveTo(getX(0), getY(40)); ctx.lineTo(getX(100), getY(40)); ctx.stroke();
        ctx.beginPath(); ctx.moveTo(getX(100), getY(0)); ctx.lineTo(getX(100), getY(40)); ctx.stroke();
        ctx.setLineDash([]);
        ctx.fillStyle = '#475569';
        ctx.fillText('W_e ($40)', padding - 55, getY(40) + 4);
        ctx.fillText('Q_e (100)', getX(100) - 25, canvas.height - padding + 20);

        // Floor Policy Line
        ctx.beginPath();
        ctx.strokeStyle = isEffective ? '#dc2626' : '#94a3b8';
        ctx.lineWidth = isEffective ? 2.5 : 1.5;
        ctx.moveTo(getX(0), getY(W_floor));
        ctx.lineTo(getX(145), getY(W_floor));
        ctx.stroke();
        ctx.fillStyle = isEffective ? '#dc2626' : '#64748b';
        ctx.fillText('W_min ($' + W_floor + ')', getX(145) - 80, getY(W_floor) - 8);

        if (isEffective) {
            ctx.setLineDash([2, 2]);
            ctx.strokeStyle = '#dc2626';
            ctx.beginPath(); ctx.moveTo(getX(Qd), getY(W_floor)); ctx.lineTo(getX(Qd), getY(0)); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(getX(Qs), getY(W_floor)); ctx.lineTo(getX(Qs), getY(0)); ctx.stroke();
            ctx.setLineDash([]);
            
            ctx.fillStyle = '#dc2626';
            ctx.fillText('Q_d', getX(Qd) - 10, canvas.height - padding + 20);
            ctx.fillText('Q_s', getX(Qs) - 10, canvas.height - padding + 20);

            ctx.beginPath();
            ctx.fillStyle = '#ef4444';
            ctx.arc(getX(Qd), getY(W_floor), 5, 0, 2 * Math.PI);
            ctx.arc(getX(Qs), getY(W_floor), 5, 0, 2 * Math.PI);
            ctx.fill();
        }
    }

    wageSlider.addEventListener('input', drawMarket);
    elasticitySelect.addEventListener('change', drawMarket);
    drawMarket();
</script>

</body>
</html>
