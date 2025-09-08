<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>สร้างแผนผังแบบกำหนดเอง</title>
    <script src="https://unpkg.com/pptxgenjs@3.11.0/dist/pptxgen.bundle.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #f5f5f5;
            color: #333;
            line-height: 1.6;
        }
        .container {
            display: flex;
            flex-direction: column;
            height: 100vh;
        }
        header {
            background-color: #2c3e50;
            color: white;
            padding: 1rem;
            text-align: center;
        }
        .main-content {
            display: flex;
            flex: 1;
            overflow: hidden;
        }
        .toolbar {
            width: 250px;
            background-color: #34495e;
            color: white;
            padding: 1rem;
            overflow-y: auto;
        }
        .toolbar-section {
            margin-bottom: 1.5rem;
        }
        .toolbar h3 {
            margin-bottom: 0.5rem;
            font-size: 1rem;
            border-bottom: 1px solid #4a6278;
            padding-bottom: 0.5rem;
        }
        .shape-option {
            display: inline-block;
            width: 50px;
            height: 50px;
            margin: 5px;
            background-color: white;
            cursor: pointer;
            text-align: center;
            line-height: 50px;
            border-radius: 4px;
            color: #333;
        }
        .canvas-container {
            flex: 1;
            position: relative;
            overflow: auto;
            background-color: #ecf0f1;
            background-image: linear-gradient(45deg, #f0f0f0 25%, transparent 25%), 
                              linear-gradient(-45deg, #f0f0f0 25%, transparent 25%), 
                              linear-gradient(45deg, transparent 75%, #f0f0f0 75%), 
                              linear-gradient(-45deg, transparent 75%, #f0f0f0 75%);
            background-size: 20px 20px;
            background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
        }
        #drawing-canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 2000px;
            height: 2000px;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 0.5rem 1rem;
            margin: 0.25rem;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #2980b9;
        }
        .export-btn {
            background-color: #27ae60;
        }
        .export-btn:hover {
            background-color: #219653;
        }
        .text-input {
            width: 100%;
            padding: 0.5rem;
            margin-bottom: 0.5rem;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        .color-picker {
            display: flex;
            flex-wrap: wrap;
            gap: 5px;
            margin-bottom: 1rem;
        }
        .color-option {
            width: 30px;
            height: 30px;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid transparent;
        }
        .color-option.selected {
            border-color: white;
            box-shadow: 0 0 5px rgba(0,0,0,0.5);
        }
        footer {
            background-color: #2c3e50;
            color: white;
            padding: 0.5rem;
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>สร้างแผนผังแบบกำหนดเอง</h1>
            <p>เพิ่มรูปร่างและข้อความได้ตามต้องการ และบันทึกเป็นไฟล์ PowerPoint</p>
        </header>
        
        <div class="main-content">
            <div class="toolbar">
                <div class="toolbar-section">
                    <h3>รูปร่าง</h3>
                    <div class="shape-option" data-shape="rectangle">□</div>
                    <div class="shape-option" data-shape="circle">○</div>
                    <div class="shape-option" data-shape="triangle">△</div>
                    <div class="shape-option" data-shape="arrow">→</div>
                </div>
                
                <div class="toolbar-section">
                    <h3>ข้อความ</h3>
                    <input type="text" id="text-input" class="text-input" placeholder="พิมพ์ข้อความที่นี่...">
                    <button id="add-text-btn">เพิ่มข้อความ</button>
                </div>
                
                <div class="toolbar-section">
                    <h3>สีเติม</h3>
                    <div class="color-picker">
                        <div class="color-option selected" style="background-color: #ffffff;" data-color="#ffffff"></div>
                        <div class="color-option" style="background-color: #ff9999;" data-color="#ff9999"></div>
                        <div class="color-option" style="background-color: #99ff99;" data-color="#99ff99"></div>
                        <div class="color-option" style="background-color: #9999ff;" data-color="#9999ff"></div>
                        <div class="color-option" style="background-color: #ffff99;" data-color="#ffff99"></div>
                        <div class="color-option" style="background-color: #ff99ff;" data-color="#ff99ff"></div>
                    </div>
                </div>
                
                <div class="toolbar-section">
                    <h3>การดำเนินการ</h3>
                    <button id="clear-btn">ล้างทั้งหมด</button>
                    <button id="export-btn" class="export-btn">บันทึกเป็น PPT</button>
                </div>
            </div>
            
            <div class="canvas-container">
                <canvas id="drawing-canvas"></canvas>
            </div>
        </div>
        
        <footer>
            <p>ลากและวางเพื่อสร้างแผนผัง - คลิกขวาเพื่อแก้ไขหรือลบ</p>
        </footer>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const canvas = document.getElementById('drawing-canvas');
            const ctx = canvas.getContext('2d');
            const textInput = document.getElementById('text-input');
            const addTextBtn = document.getElementById('add-text-btn');
            const clearBtn = document.getElementById('clear-btn');
            const exportBtn = document.getElementById('export-btn');
            const colorOptions = document.querySelectorAll('.color-option');
            const shapeOptions = document.querySelectorAll('.shape-option');
            
            let isDrawing = false;
            let currentShape = 'rectangle';
            let fillColor = '#ffffff';
            let startX, startY;
            let shapes = [];
            let selectedShapeIndex = -1;
            let isDragging = false;
            let dragOffsetX, dragOffsetY;
            
            function resizeCanvas() {
                canvas.width = 2000;
                canvas.height = 2000;
            }
            resizeCanvas();
            
            function drawShape(shape) {
                ctx.save();
                ctx.fillStyle = shape.color;
                ctx.strokeStyle = '#333';
                ctx.lineWidth = 2;
                switch(shape.type) {
                    case 'rectangle':
                        ctx.fillRect(shape.x, shape.y, shape.width, shape.height);
                        ctx.strokeRect(shape.x, shape.y, shape.width, shape.height);
                        break;
                    case 'circle':
                        const radius = Math.min(shape.width, shape.height) / 2;
                        ctx.beginPath();
                        ctx.arc(shape.x + shape.width/2, shape.y + shape.height/2, radius, 0, Math.PI * 2);
                        ctx.fill();
                        ctx.stroke();
                        break;
                    case 'triangle':
                        ctx.beginPath();
                        ctx.moveTo(shape.x + shape.width/2, shape.y);
                        ctx.lineTo(shape.x + shape.width, shape.y + shape.height);
                        ctx.lineTo(shape.x, shape.y + shape.height);
                        ctx.closePath();
                        ctx.fill();
                        ctx.stroke();
                        break;
                    case 'arrow':
                        ctx.beginPath();
                        ctx.moveTo(shape.x, shape.y + shape.height/2);
                        ctx.lineTo(shape.x + shape.width, shape.y + shape.height/2);
                        ctx.lineTo(shape.x + shape.width - 10, shape.y);
                        ctx.moveTo(shape.x + shape.width, shape.y + shape.height/2);
                        ctx.lineTo(shape.x + shape.width - 10, shape.y + shape.height);
                        ctx.stroke();
                        break;
                    case 'text':
                        ctx.font = '16px Arial';
                        ctx.fillStyle = '#000';
                        ctx.fillText(shape.text, shape.x, shape.y + 16);
                        break;
                }
                if (shape.text && shape.type !== 'text') {
                    ctx.font = '14px Arial';
                    ctx.fillStyle = '#000';
                    ctx.textAlign = 'center';
                    ctx.fillText(shape.text, shape.x + shape.width/2, shape.y + shape.height/2 + 5);
                }
                ctx.restore();
            }
            
            function redrawCanvas() {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                shapes.forEach((shape, index) => {
                    drawShape(shape);
                    if (index === selectedShapeIndex) {
                        ctx.strokeStyle = '#3498db';
                        ctx.lineWidth = 2;
                        ctx.setLineDash([5, 5]);
                        ctx.strokeRect(shape.x - 5, shape.y - 5, shape.width + 10, shape.height + 10);
                        ctx.setLineDash([]);
                    }
                });
            }
            
            canvas.addEventListener('mousedown', function(e) {
                const rect = canvas.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                for (let i = shapes.length - 1; i >= 0; i--) {
                    const shape = shapes[i];
                    if (x >= shape.x && x <= shape.x + shape.width &&
                        y >= shape.y && y <= shape.y + shape.height) {
                        selectedShapeIndex = i;
                        isDragging = true;
                        dragOffsetX = x - shape.x;
                        dragOffsetY = y - shape.y;
                        redrawCanvas();
                        return;
                    }
                }
                selectedShapeIndex = -1;
                isDrawing = true;
                startX = x;
                startY = y;
            });
            
            canvas.addEventListener('mousemove', function(e) {
                if (!isDrawing && !isDragging) return;
                const rect = canvas.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                if (isDragging && selectedShapeIndex >= 0) {
                    shapes[selectedShapeIndex].x = x - dragOffsetX;
                    shapes[selectedShapeIndex].y = y - dragOffsetY;
                } else if (isDrawing) {
                    redrawCanvas();
                    ctx.save();
                    ctx.setLineDash([5, 5]);
                    drawShape({ type: currentShape, x: startX, y: startY, width: x - startX, height: y - startY, color: fillColor });
                    ctx.restore();
                }
                redrawCanvas();
            });
            
            canvas.addEventListener('mouseup', function(e) {
                if (isDrawing) {
                    const rect = canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    const width = x - startX;
                    const height = y - startY;
                    if (Math.abs(width) > 5 && Math.abs(height) > 5) {
                        shapes.push({ type: currentShape, x: startX, y: startY, width: width, height: height, color: fillColor, text: '' });
                        selectedShapeIndex = shapes.length - 1;
                    }
                }
                isDrawing = false;
                isDragging = false;
                redrawCanvas();
            });
            
            canvas.addEventListener('contextmenu', function(e) {
                e.preventDefault();
                const rect = canvas.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                for (let i = shapes.length - 1; i >= 0; i--) {
                    const shape = shapes[i];
                    if (x >= shape.x && x <= shape.x + shape.width &&
                        y >= shape.y && y <= shape.y + shape.height) {
                        selectedShapeIndex = i;
                        const text = prompt('แก้ไขข้อความ (เว้นว่างเพื่อลบ):', shape.text || '');
                        if (text === null) return;
                        if (text === '') {
                            shapes.splice(i, 1);
                            selectedShapeIndex = -1;
                        } else {
                            shapes[i].text = text;
                        }
                        redrawCanvas();
                        return;
                    }
                }
            });
            
            addTextBtn.addEventListener('click', function() {
                const text = textInput.value.trim();
                if (text) {
                    shapes.push({ type: 'text', x: 100, y: 100, width: 0, height: 0, color: '#000', text: text });
                    textInput.value = '';
                    selectedShapeIndex = shapes.length - 1;
                    redrawCanvas();
                }
            });
            
            clearBtn.addEventListener('click', function() {
                if (confirm('ต้องการล้างแผนผังทั้งหมดใช่หรือไม่?')) {
                    shapes = [];
                    selectedShapeIndex = -1;
                    redrawCanvas();
                }
            });
            
            colorOptions.forEach(option => {
                option.addEventListener('click', function() {
                    colorOptions.forEach(opt => opt.classList.remove('selected'));
                    this.classList.add('selected');
                    fillColor = this.getAttribute('data-color');
                    if (selectedShapeIndex >= 0) {
                        shapes[selectedShapeIndex].color = fillColor;
                        redrawCanvas();
                    }
                });
            });
            
            shapeOptions.forEach(option => {
                option.addEventListener('click', function() {
                    shapeOptions.forEach(opt => opt.style.backgroundColor = '');
                    this.style.backgroundColor = '#3498db';
                    this.style.color = 'white';
                    currentShape = this.getAttribute('data-shape');
                });
            });
            
            exportBtn.addEventListener('click', function() {
                const pptx = new PptxGenJS();
                const slide = pptx.addSlide();
                slide.addText('แผนผังที่สร้าง', { x: 0.5, y: 0.2, w: 9, h: 0.5, fontSize: 24, align: 'center' });
                canvas.toBlob(function(blob) {
                    const reader = new FileReader();
                    reader.onloadend = function() {
                        slide.addImage({ data: reader.result, x: 1, y: 1, w: 8, h: 5 });
                        pptx.writeFile({ fileName: 'แผนผังของฉัน.pptx' });
                    };
                    reader.readAsDataURL(blob);
                });
            });
            redrawCanvas();
        });
    </script>
</body>
</html>
