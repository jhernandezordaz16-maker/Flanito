<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ruleta de Premios - ¡Gana con tu Flan!</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #fff5e6 0%, #ffcc80 100%);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
            color: #5d4037;
        }

        .container {
            background: white;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            text-align: center;
            max-width: 400px;
            width: 100%;
        }

        h1 {
            margin-top: 0;
            color: #d84315;
            font-size: 24px;
        }

        p {
            font-size: 16px;
            line-height: 1.5;
            margin-bottom: 25px;
        }

        /* Contenedor de la ruleta y la flecha */
        .wheel-container {
            position: relative;
            width: 280px;
            height: 280px;
            margin: 0 auto 30px auto;
        }

        /* Flecha indicadora */
        .pointer {
            position: absolute;
            top: -15px;
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 0;
            border-left: 15px solid transparent;
            border-right: 15px solid transparent;
            border-top: 30px solid #d84315;
            z-index: 10;
            filter: drop-shadow(0px 3px 2px rgba(0,0,0,0.2));
        }

        /* Canvas de la ruleta */
        #wheel {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
            transition: transform 4s cubic-bezier(0.15, 0.85, 0.35, 1);
        }

        /* Botón para girar */
        .btn-spin {
            background: #d84315;
            color: white;
            border: none;
            padding: 12px 35px;
            font-size: 18px;
            font-weight: bold;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.15);
            transition: all 0.2s ease;
        }

        .btn-spin:hover {
            background: #bf360c;
            transform: translateY(-2px);
            box-shadow: 0 6px 8px rgba(0,0,0,0.2);
        }

        .btn-spin:disabled {
            background: #b0bec5;
            cursor: not-allowed;
            transform: none;
            box-shadow: none;
        }

        /* Modal o mensaje de resultado */
        .result-popup {
            display: none;
            margin-top: 20px;
            padding: 15px;
            border-radius: 10px;
            font-weight: bold;
            font-size: 18px;
            animation: popIn 0.5s ease;
        }

        .success {
            background-color: #e8f5e9;
            color: #2e7d32;
            border: 2px dashed #81c784;
        }

        .error {
            background-color: #ffebee;
            color: #c62828;
            border: 1px solid #ef9a9a;
            font-size: 14px;
        }

        .code-container {
            margin-top: 10px;
            font-size: 14px;
            color: #37474f;
            background: rgba(255,255,255,0.5);
            padding: 5px;
            border-radius: 5px;
            display: inline-block;
        }

        @keyframes popIn {
            0% { transform: scale(0.8); opacity: 0; }
            100% { transform: scale(1); opacity: 1; }
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🍮 ¡Raspa y Gana tu Flan! 🍮</h1>
    <p id="instruction-text">¡Escaneaste el QR! Gira la ruleta para descubrir si ganaste un premio especial con tu compra.</p>
    
    <div class="wheel-container">
        <div class="pointer"></div>
        <canvas id="wheel" width="300" height="300"></canvas>
    </div>

    <button class="btn-spin" id="spin-button" onclick="spinWheel()">¡GIRAR!</button>

    <div class="result-popup" id="result-box">
        <div id="result-text"></div>
        <div class="code-container" id="code-box" style="display:none;">
            Código de validación: <strong id="validation-code"></strong>
        </div>
    </div>
</div>

<script>
    // Configuración de los premios
    const prizes = [
        { text: "FLAN EXTRA", color: "#ffb74d" },
        { text: "15% DESC.", color: "#ffe082" },
        { text: "50% DESC.", color: "#ffd54f" },
        { text: "20% DESC.", color: "#ffca28" }
    ];

    const canvas = document.getElementById("wheel");
    const ctx = canvas.getContext("2d");
    const spinButton = document.getElementById("spin-button");
    const resultBox = document.getElementById("result-box");
    const resultText = document.getElementById("result-text");
    const codeBox = document.getElementById("code-box");
    const validationCode = document.getElementById("validation-code");
    const instructionText = document.getElementById("instruction-text");

    const numSegments = prizes.length;
    const arcSize = (2 * Math.PI) / numSegments;
    let currentRotation = 0;

    // Dibujar la ruleta
    function drawWheel() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        for (let i = 0; i < numSegments; i++) {
            const angle = i * arcSize;
            
            // Dibujar rebanada
            ctx.fillStyle = prizes[i].color;
            ctx.beginPath();
            ctx.moveTo(150, 150);
            ctx.arc(150, 150, 140, angle, angle + arcSize);
            ctx.lineTo(150, 150);
            ctx.fill();
            ctx.strokeStyle = "#ffffff";
            ctx.lineWidth = 3;
            ctx.stroke();

            // Dibujar Texto
            ctx.save();
            ctx.fillStyle = "#5d4037";
            ctx.font = "bold 14px sans-serif";
            ctx.translate(150, 150);
            ctx.rotate(angle + arcSize / 2);
            ctx.textAlign = "right";
            ctx.fillText(prizes[i].text, 130, 5);
            ctx.restore();
        }

        // Círculo central estético
        ctx.beginPath();
        ctx.arc(150, 150, 20, 0, 2 * Math.PI);
        ctx.fillStyle = "#ffffff";
        ctx.fill();
        ctx.stroke();
    }

    // Comprobación de juego diario usando localStorage
    function checkDailyLimit() {
        const lastSpin = localStorage.getItem("lastFlanSpin");
        if (lastSpin) {
            const lastSpinDate = new Date(parseInt(lastSpin));
            const today = new Date();
            
            // Compara si es el mismo día, mes y año
            if (lastSpinDate.getDate() === today.getDate() &&
                lastSpinDate.getMonth() === today.getMonth() &&
                lastSpinDate.getFullYear() === today.getFullYear()) {
                
                // Si ya jugó hoy, bloquear ruleta
                spinButton.disabled = true;
                instructionText.innerText = "¡Vaya! Parece que ya probaste tu suerte hoy.";
                resultBox.className = "result-popup error";
                resultBox.style.display = "block";
                resultText.innerHTML = "<strong>Ya utilizaste tu oportunidad por hoy.</strong><br>Vuelve mañana con tu siguiente flan.";
                return true;
            }
        }
        return false;
    }

    // Función para hacer girar la ruleta
    function spinWheel() {
        if (checkDailyLimit()) return;

        spinButton.disabled = true;
        
        // Generar un giro aleatorio (mínimo 5 vueltas completas + ángulo extra)
        const totalSpins = 5; 
        const randomSegmentIndex = Math.floor(Math.random() * numSegments);
        
        // Calcular el ángulo objetivo para que el segmento quede arriba (apuntado por la flecha a -90 grados o 1.5 * PI)
        // En canvas 0 grados es a la derecha. La flecha está arriba (-90 grados).
        const targetAngle = (1.5 * Math.PI) - (randomSegmentIndex * arcSize) - (arcSize / 2);
        
        // Ajustar rotación final sumando vueltas completas
        const finalRotation = (totalSpins * 2 * Math.PI) + targetAngle;
        
        canvas.style.transform = `rotate(${finalRotation}rad)`;
        
        // Guardar la fecha del juego actual
        localStorage.setItem("lastFlanSpin", Date.now().toString());

        // Esperar a que termine la animación (4 segundos configurados en CSS)
        setTimeout(() => {
            showWinner(randomSegmentIndex);
        }, 4000);
    }

    // Mostrar el resultado
    function showWinner(index) {
        const prize = prizes[index];
        resultBox.className = "result-popup success";
        resultBox.style.display = "block";
        
        // Generar un código aleatorio simple para evitar fraudes (capturas de pantalla repetidas)
        const randomCode = "FLAN-" + Math.floor(1000 + Math.random() * 9000);

        if (prize.text === "FLAN EXTRA") {
            resultText.innerHTML = "🎉 ¡Felicidades! ¡Te ganaste un <strong>FLAN EXTRA gratis</strong>! 🎉";
        } else {
            resultText.innerHTML = `🎉 ¡Felicidades! Obtuviste un <strong>${prize.text}</strong> en tu próxima compra. 🎉`;
        }

        validationCode.innerText = randomCode;
        codeBox.style.display = "block";
        instructionText.innerText = "Muestra esta pantalla al encargado para reclamar tu premio.";
    }

    // Inicializar
    drawWheel();
    checkDailyLimit();

</script>

</body>
</html>
