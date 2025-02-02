<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Collaborative Drawing</title>
    <style>
        body {
            font-family: 'Comic Sans MS', cursive, sans-serif;
            text-align: center;
            background-color: #f0f0f0;
            color: #333;
        }
        #canvas {
            border: 2px solid #333;
            cursor: crosshair;
            background-color: #fff;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .toolbar {
            margin: 10px;
            padding: 10px;
            background-color: #fff;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .toolbar button, .toolbar select, .toolbar input {
            margin: 5px;
            padding: 5px 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            background-color: #f9f9f9;
            cursor: pointer;
        }
        .toolbar button:hover, .toolbar select:hover, .toolbar input:hover {
            background-color: #e9e9e9;
        }
    </style>
</head>
<body>
    <h1>Collaborative Drawing</h1>
    <div class="toolbar">
        <label for="colorPicker">Color:</label>
        <input type="color" id="colorPicker">
        <label for="brushSize">Brush Size:</label>
        <select id="brushSize">
            <option value="2">Fine</option>
            <option value="5">Medium</option>
            <option value="10">Thick</option>
        </select>
        <button onclick="setTool('pencil')">✏️ Pencil</button>
        <button onclick="setTool('eraser')">🧽 Eraser</button>
        <button onclick="setTool('text')">📝 Text</button>
        <button onclick="clearCanvas()">Clear</button>
        <button onclick="saveDrawing()">Save</button>
        <label for="fontSelect">Font:</label>
        <select id="fontSelect">
            <option value="Comic Sans MS">Comic Sans MS</option>
            <option value="Arial">Arial</option>
            <option value="Times New Roman">Times New Roman</option>
            <option value="Courier New">Courier New</option>
        </select>
    </div>
    <canvas id="canvas" width="800" height="600"></canvas>
    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        let drawing = false;
        let brushSize = 2;
        let tool = 'pencil';
        let drawings = JSON.parse(localStorage.getItem('drawings')) || [];

        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mouseup', stopDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('click', addText);
        document.getElementById('colorPicker').addEventListener('input', changeColor);
        document.getElementById('brushSize').addEventListener('change', changeBrushSize);

        function setTool(selectedTool) {
            tool = selectedTool;
            switch (tool) {
                case 'pencil':
                    canvas.style.cursor = 'url(https://twemoji.maxcdn.com/v/latest/72x72/270f.png), auto';
                    break;
                case 'eraser':
                    canvas.style.cursor = 'url(https://twemoji.maxcdn.com/v/latest/72x72/1f9f9.png), auto';
                    break;
                case 'text':
                    canvas.style.cursor = 'text';
                    break;
            }
        }

        function startDrawing(e) {
            if (tool === 'pencil' || tool === 'eraser') {
                drawing = true;
                draw(e);
            }
        }

        function stopDrawing() {
            drawing = false;
            ctx.beginPath();
        }

        function draw(e) {
            if (!drawing) return;
            const rect = canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            if (x < 0 || x > canvas.width || y < 0 || y > canvas.height) {
                alert('You cannot draw outside the canvas.');
                return;
            }
            if (tool === 'pencil') {
                ctx.lineWidth = brushSize;
                ctx.lineCap = 'round';
                ctx.strokeStyle = document.getElementById('colorPicker').value;
                ctx.lineTo(x, y);
                ctx.stroke();
                ctx.beginPath();
                ctx.moveTo(x, y);
            } else if (tool === 'eraser') {
                ctx.clearRect(x - brushSize / 2, y - brushSize / 2, brushSize, brushSize);
            }
        }

        function addText(e) {
            if (tool === 'text') {
                const text = prompt('Enter your text:');
                if (text) {
                    const rect = canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    const font = document.getElementById('fontSelect').value;
                    const color = document.getElementById('colorPicker').value;
                    ctx.font = `20px ${font}`;
                    ctx.fillStyle = color;
                    ctx.fillText(text, x, y);
                }
            }
        }

        function changeColor() {
            ctx.strokeStyle = document.getElementById('colorPicker').value;
        }

        function changeBrushSize() {
            brushSize = document.getElementById('brushSize').value;
        }

        function clearCanvas() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }

        function saveDrawing() {
            const author = prompt('Enter your name:');
            if (author) {
                const drawingData = { author: author, image: canvas.toDataURL() };
                drawings.push(drawingData);
                localStorage.setItem('drawings', JSON.stringify(drawings));
                alert('Drawing saved!');
            }
        }

        function loadDrawings() {
            drawings.forEach(drawing => {
                const img = new Image();
                img.src = drawing.image;
                img.onload = () => {
                    ctx.drawImage(img, 0, 0);
                };
            });
        }

        window.onload = loadDrawings;
    </script>
</body>
</html>


