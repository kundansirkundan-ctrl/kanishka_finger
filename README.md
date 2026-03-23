<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cosmic Bubble Universe - Mouse + Touch Drawing</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            overflow: hidden;
        }

        body {
            height: 100vh;
            background: #000;
            font-family: Arial, sans-serif;
            cursor: none;
            touch-action: none;
        }

        #canvas {
            display: block;
        }

        .cursor-bubble {
            position: fixed;
            width: 50px;
            height: 50px;
            border: 2px solid rgba(255,255,255,0.4);
            border-radius: 50%;
            pointer-events: none;
            z-index: 1000;
            background: rgba(255,255,255,0.1);
            backdrop-filter: blur(15px);
            box-shadow: 0 0 20px rgba(255,255,255,0.3);
            transition: all 0.1s ease-out;
        }

        .glow {
            position: fixed;
            pointer-events: none;
            border-radius: 50%;
            background: radial-gradient(circle, rgba(255,200,100,0.8) 0%, transparent 70%);
            z-index: 999;
            animation: glowPulse 0.8s ease-out forwards;
        }

        @keyframes glowPulse {
            0% { transform: scale(1); opacity: 1; }
            100% { transform: scale(4); opacity: 0; }
        }
    </style>
</head>
<body>
    <canvas id="canvas"></canvas>
    
    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // Stars
        const stars = [];
        for(let i = 0; i < 250; i++) {
            stars.push({
                x: Math.random() * canvas.width,
                y: Math.random() * canvas.height,
                size: Math.random() * 2.5 + 0.5,
                twinkle: Math.random() * Math.PI * 2,
                speed: Math.random() * 0.03 + 0.01
            });
        }

        // DRAWING SYSTEM - MOUSE + TOUCH
        let isDrawing = false;
        let strokePoints = [];
        let drawingTimer = 0;
        let currentLetter = '';

        // Letter patterns
        const letterPatterns = {
            'A': [[0.2,0.8],[0.5,0.2],[0.8,0.8],[0.35,0.5],[0.65,0.5]],
            'B': [[0.2,0.2],[0.2,0.8],[0.6,0.3],[0.6,0.6],[0.2,0.5]],
            'C': [[0.8,0.3],[0.2,0.3],[0.2,0.7],[0.8,0.7]],
            'D': [[0.2,0.2],[0.2,0.8],[0.8,0.5]],
            'E': [[0.8,0.2],[0.2,0.2],[0.2,0.8],[0.8,0.8],[0.2,0.5]],
            'F': [[0.2,0.2],[0.2,0.8],[0.8,0.2],[0.2,0.5]],
            'H': [[0.2,0.2],[0.2,0.8],[0.8,0.2],[0.8,0.8],[0.5,0.5]],
            'I': [[0.5,0.1],[0.5,0.9],[0.2,0.5],[0.8,0.5]],
            'L': [[0.2,0.2],[0.2,0.9],[0.8,0.9]],
            'M': [[0.2,0.8],[0.5,0.2],[0.8,0.8],[0.2,0.5],[0.8,0.5]],
            'N': [[0.2,0.8],[0.2,0.2],[0.8,0.8],[0.8,0.2]],
            'O': [[0.2,0.3],[0.8,0.3],[0.8,0.7],[0.2,0.7]],
            'P': [[0.2,0.2],[0.2,0.7],[0.6,0.4],[0.6,0.2]],
            'R': [[0.2,0.2],[0.2,0.7],[0.6,0.4],[0.6,0.2],[0.8,0.7]],
            'S': [[0.8,0.3],[0.2,0.3],[0.2,0.6],[0.8,0.6],[0.8,0.9]],
            'T': [[0.5,0.1],[0.5,0.9],[0.2,0.5],[0.8,0.5]],
            'U': [[0.2,0.9],[0.2,0.3],[0.8,0.3],[0.8,0.9]],
            'W': [[0.2,0.3],[0.2,0.9],[0.5,0.5],[0.8,0.9],[0.8,0.3]],
            'X': [[0.2,0.2],[0.8,0.8],[0.2,0.8],[0.8,0.2]],
            'Y': [[0.5,0.2],[0.2,0.5],[0.8,0.5],[0.5,0.8]],
            '1': [[0.5,0.1],[0.5,0.9]],
            '2': [[0.2,0.2],[0.8,0.2],[0.8,0.6],[0.2,0.6],[0.2,0.9]],
            '3': [[0.2,0.2],[0.8,0.2],[0.8,0.5],[0.2,0.5],[0.8,0.9]],
            '4': [[0.2,0.9],[0.2,0.5],[0.5,0.5],[0.5,0.2]],
            '5': [[0.8,0.2],[0.2,0.2],[0.2,0.5],[0.8,0.5],[0.8,0.9]],
            '6': [[0.8,0.3],[0.2,0.3],[0.2,0.7],[0.8,0.7],[0.5,0.5]],
            '7': [[0.2,0.2],[0.8,0.2],[0.5,0.9]],
            '8': [[0.5,0.5],[0.2,0.3],[0.8,0.3],[0.8,0.7],[0.2,0.7]],
            '9': [[0.2,0.7],[0.8,0.7],[0.8,0.3],[0.5,0.5]],
            '0': [[0.2,0.2],[0.8,0.2],[0.8,0.8],[0.2,0.8]]
        };

        function recognizeLetter(points) {
            if (points.length < 8) return '';
            
            const bbox = {
                minX: Math.min(...points.map(p => p.x)),
                maxX: Math.max(...points.map(p => p.x)),
                minY: Math.min(...points.map(p => p.y)),
                maxY: Math.max(...points.map(p => p.y))
            };
            
            const width = bbox.maxX - bbox.minX;
            const height = bbox.maxY - bbox.minY;
            if (width === 0 || height === 0) return '';
            
            const normalizedPoints = points.map(p => ({
                x: (p.x - bbox.minX) / width,
                y: (p.y - bbox.minY) / height
            }));

            let bestMatch = '';
            let bestScore = 0;
            
            for (let letter in letterPatterns) {
                let score = 0;
                const pattern = letterPatterns[letter];
                
                normalizedPoints.forEach(point => {
                    let minDist = Infinity;
                    pattern.forEach(patternPoint => {
                        const dist = Math.hypot(point.x - patternPoint[0], point.y - patternPoint[1]);
                        minDist = Math.min(minDist, dist);
                    });
                    score += (1 - minDist);
                });
                
                if (score > bestScore) {
                    bestScore = score;
                    bestMatch = letter;
                }
            }
            
            return bestScore > 6 ? bestMatch : '';
        }

        // FloatingText - 4 SECONDS
        class FloatingText {
            constructor(char) {
                this.x = Math.random() * (canvas.width - 200) + 100;
                this.y = Math.random() * (canvas.height - 200) + 100;
                this.char = char;
                this.size = 0;
                this.targetSize = 140;
                this.growthSpeed = 5;
                this.life = 1.0;
                this.totalLife = 240;  // 4 SECONDS
                this.currentFrame = 0;
                this.colorIndex = Math.floor(Math.random() * colors.length);
                this.glowIntensity = 0;
            }

            update() {
                this.currentFrame++;
                if (this.currentFrame >= this.totalLife) return false;
                
                if (this.currentFrame < 60) {
                    this.size = Math.min(this.size + this.growthSpeed, 140);
                    this.glowIntensity = this.currentFrame / 60;
                } 
                else if (this.currentFrame < 180) {
                    this.size = 140;
                    this.glowIntensity = 1.0;
                } 
                else {
                    const fadeProgress = (this.currentFrame - 180) / 60;
                    this.size = 140 * (1 - fadeProgress);
                    this.glowIntensity = 1 - fadeProgress;
                    this.life = 1 - fadeProgress;
                }
                return true;
            }

            draw() {
                ctx.save();
                ctx.globalAlpha = this.life * this.glowIntensity;
                
                const gradient = ctx.createLinearGradient(
                    this.x - 70, this.y, this.x + 70, this.y
                );
                gradient.addColorStop(0, colors[this.colorIndex % colors.length]);
                gradient.addColorStop(0.25, colors[(this.colorIndex + 1) % colors.length]);
                gradient.addColorStop(0.5, colors[(this.colorIndex + 2) % colors.length]);
                gradient.addColorStop(0.75, colors[(this.colorIndex + 3) % colors.length]);
                gradient.addColorStop(1, colors[(this.colorIndex + 4) % colors.length]);

                ctx.shadowColor = colors[(this.colorIndex + 2) % colors.length];
                ctx.shadowBlur = 50 * this.glowIntensity;
                
                ctx.fillStyle = gradient;
                ctx.font = `${this.size}px Arial Black`;
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(this.char, this.x, this.y);
                
                ctx.lineWidth = 6;
                ctx.strokeStyle = 'rgba(255,255,255,0.9)';
                ctx.strokeText(this.char, this.x, this.y);
                ctx.shadowBlur = 0;
                ctx.restore();
            }
        }

        const colors = [
            '#FF6B6B', '#4ECDC4', '#45B7D1', '#F9CA24', '#F0932B',
            '#EB4D4B', '#6C5CE7', '#A29BFE', '#FD79A8', '#00B894',
            '#00CEC9', '#55A3FF', '#FFEAA7', '#D63031', '#00B894'
        ];

        class Bubble {
            constructor() {
                this.x = Math.random() * canvas.width;
                this.y = -120;
                this.size = Math.random() * 140 + 90;
                this.speedY = Math.random() * 1.2 + 0.4;
                this.speedX = (Math.random() - 0.5) * 0.7;
                this.opacity = Math.random() * 0.35 + 0.25;
                this.life = 1.0;
                this.sparkling = Math.random() > 0.5;
            }

            update() {
                this.y += this.speedY;
                this.x += this.speedX;
                this.life -= 0.003;
                return this.y < canvas.height + 150 && this.life > 0;
            }

            draw() {
                ctx.save();
                ctx.globalAlpha = this.opacity * this.life;
                
                const gradient = ctx.createRadialGradient(
                    this.x, this.y - this.size/2.5, 0,
                    this.x, this.y, this.size
                );
                gradient.addColorStop(0, `rgba(100,200,255,${0.9 * this.life})`);
                gradient.addColorStop(0.4, `rgba(150,220,255,${0.6 * this.life})`);
                gradient.addColorStop(0.8, `rgba(50,150,255,${0.2 * this.life})`);
                gradient.addColorStop(1, `rgba(0,50,150,${0.05 * this.life})`);

                ctx.fillStyle = gradient;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fill();

                ctx.fillStyle = `rgba(255,255,255,${0.7 * this.life})`;
                ctx.beginPath();
                ctx.arc(this.x - this.size/4, this.y - this.size/4, this.size/5, 0, Math.PI * 2);
                ctx.fill();

                if (this.sparkling) {
                    for(let i = 0; i < 4; i++) {
                        if (Math.random() > 0.6) {
                            const angle = (i / 4) * Math.PI * 2;
                            const sx = this.x + Math.cos(angle) * this.size / 2;
                            const sy = this.y + Math.sin(angle) * this.size / 2;
                            ctx.fillStyle = `rgba(255,255,255,${0.8 * this.life})`;
                            ctx.beginPath();
                            ctx.arc(sx, sy, 2.5, 0, Math.PI * 2);
                            ctx.fill();
                        }
                    }
                }
                ctx.restore();
            }

            createGlowRing() {
                const glow = document.createElement('div');
                glow.className = 'glow';
                glow.style.left = (this.x - 120) + 'px';
                glow.style.top = (this.y - 120) + 'px';
                glow.style.width = '240px';
                glow.style.height = '240px';
                document.body.appendChild(glow);
                setTimeout(() => glow.remove(), 800);
            }
        }

        let bubbles = [];
        let floatingTexts = [];
        let cursorBubble = null;
        let mouseX = 0, mouseY = 0;

        // 🔥 MOUSE DRAWING - LEFT BUTTON HOLD!
        canvas.addEventListener('mousedown', (e) => {
            if (e.button === 0) { // Left button
                isDrawing = true;
                strokePoints = [];
                drawingTimer = 60;
                mouseX = e.clientX;
                mouseY = e.clientY;
            }
        });

        canvas.addEventListener('mousemove', (e) => {
            mouseX = e.clientX;
            mouseY = e.clientY;
            
            if (!cursorBubble) {
                cursorBubble = document.createElement('div');
                cursorBubble.className = 'cursor-bubble';
                document.body.appendChild(cursorBubble);
            }
            cursorBubble.style.left = (mouseX - 25) + 'px';
            cursorBubble.style.top = (mouseY - 25) + 'px';
            
            if (isDrawing) {
                const rect = canvas.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                strokePoints.push({x, y});
                
                // DRAWING TRAIL
                ctx.strokeStyle = 'rgba(255,255,255,0.7)';
                ctx.lineWidth = 12;
                ctx.lineCap = 'round';
                ctx.globalAlpha = 0.8;
                ctx.lineJoin = 'round';
                ctx.beginPath();
                ctx.moveTo(mouseX - rect.left || x, mouseY - rect.top || y);
                ctx.lineTo(x, y);
                ctx.stroke();
                ctx.globalAlpha = 1;
            } else if (Math.random() > 0.7) {
                const trail = new Bubble();
                trail.x = mouseX;
                trail.y = mouseY;
                trail.size *= 0.35;
                trail.speedY = Math.random() * 1 + 0.5;
                trail.opacity = 0.6;
                trail.life = 0.5;
                bubbles.push(trail);
            }
        });

        canvas.addEventListener('mouseup', (e) => {
            if (e.button === 0) {
                isDrawing = false;
                if (strokePoints.length > 8) {
                    currentLetter = recognizeLetter(strokePoints);
                    if (currentLetter) {
                        floatingTexts.push(new FloatingText(currentLetter));
                        if (bubbles.length > 0) {
                            const randomBubble = bubbles[Math.floor(Math.random() * bubbles.length)];
                            randomBubble.createGlowRing();
                        }
                    }
                }
            }
        });

        // TOUCH EVENTS (unchanged)
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            isDrawing = true;
            strokePoints = [];
            drawingTimer = 60;
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            mouseX = touch.clientX;
            mouseY = touch.clientY;
        });

        canvas.addEventListener('touchmove', (e) => {
            e.preventDefault();
            if (!isDrawing) return;
            
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            const x = touch.clientX - rect.left;
            const y = touch.clientY - rect.top;
            
            strokePoints.push({x, y});
            
            ctx.strokeStyle = 'rgba(255,255,255,0.7)';
            ctx.lineWidth = 12;
            ctx.lineCap = 'round';
            ctx.globalAlpha = 0.8;
            ctx.lineJoin = 'round';
            ctx.beginPath();
            ctx.moveTo(mouseX - rect.left || x, mouseY - rect.top || y);
            ctx.lineTo(x, y);
            ctx.stroke();
            ctx.globalAlpha = 1;
            mouseX = touch.clientX;
            mouseY = touch.clientY;
        });

        canvas.addEventListener('touchend', (e) => {
            e.preventDefault();
            isDrawing = false;
            if (strokePoints.length > 8) {
                currentLetter = recognizeLetter(strokePoints);
                if (currentLetter) {
                    floatingTexts.push(new FloatingText(currentLetter));
                    if (bubbles.length > 0) {
                        const randomBubble = bubbles[Math.floor(Math.random() * bubbles.length)];
                        randomBubble.createGlowRing();
                    }
                }
            }
        });

        // Keyboard (unchanged)
        document.addEventListener('keydown', (e) => {
            e.preventDefault();
            let char = '';
            if (e.code === 'Space') {
                const emojis = ['🌟','✨','⭐','💫','🪐','🌌','💎','🔥','⚡','🌈','🎆','🎇','🌙','☄️'];
                char = emojis[Math.floor(Math.random() * emojis.length)];
            } else if (/Digit[0-9]/.test(e.code)) {
                char = e.code.replace('Digit', '');
            } else if (/Key[A-Z]/.test(e.code)) {
                char = e.code.replace('Key', '').toUpperCase();
            }
            if (char) {
                floatingTexts.push(new FloatingText(char));
                if (bubbles.length > 0) {
                    const randomBubble = bubbles[Math.floor(Math.random() * bubbles.length)];
                    randomBubble.createGlowRing();
                }
            }
        });

        function drawStars() {
            stars.forEach(star => {
                star.twinkle += star.speed;
                const alpha = (Math.sin(star.twinkle) * 0.5 + 0.5) * 0.9 + 0.1;
                ctx.save();
                ctx.globalAlpha = alpha;
                ctx.fillStyle = '#ffffff';
                ctx.beginPath();
                ctx.arc(star.x, star.y, star.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            });
        }

        function animate() {
            ctx.fillStyle = 'rgba(0,0,8,0.12)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            drawStars();
            
            if (Math.random() > 0.9) {
                bubbles.push(new Bubble());
            }
            
            if (drawingTimer > 0) {
                drawingTimer--;
            }
            
            for (let i = floatingTexts.length - 1; i >= 0; i--) {
                if (!floatingTexts[i].update()) {
                    floatingTexts.splice(i, 1);
                } else {
                    floatingTexts[i].draw();
                }
            }
            
            for (let i = bubbles.length - 1; i >= 0; i--) {
                if (!bubbles[i].update()) {
                    bubbles.splice(i, 1);
                } else {
                    bubbles[i].draw();
                }
            }
            
            requestAnimationFrame(animate);
        }

        animate();
    </script>
</body>
</html>
