<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Rastreamento de Objeto em Vídeo</title>
<style>
    #video-container {
        position: relative;
    }
    #canvas-overlay {
        position: absolute;
        top: 0;
        left: 0;
    }
</style>
</head>
<body>
    <h1>Rastreamento de Objeto em Vídeo</h1>
    <input type="file" accept=".mp4" id="video-input">
    <div id="video-container">
        <video id="video" width="640" height="480" controls></video>
        <canvas id="canvas-overlay" width="640" height="480"></canvas>
    </div>
    <p id="distancia-percorrida"></p>

    <script>
        let video, canvas, ctx, cap, tracker;

        async function iniciarRastreamento() {
            video = document.getElementById('video');
            canvas = document.getElementById('canvas-overlay');
            ctx = canvas.getContext('2d');

            document.getElementById('video-input').addEventListener('change', async function(e) {
                const file = e.target.files[0];
                const videoUrl = URL.createObjectURL(file);
                video.src = videoUrl;
                video.play();
                await iniciarOpenCv();
            });

            video.addEventListener('play', async function() {
                const fps = 30;
                const timeInterval = 1000 / fps;
                setInterval(processarFrame, timeInterval);
            });

            canvas.addEventListener('click', function(e) {
                const x = e.offsetX;
                const y = e.offsetY;
                iniciarRastreamentoObjeto(x, y);
            });
        }

        async function iniciarOpenCv() {
            await new Promise(resolve => {
                const script = document.createElement('script');
                script.src = 'https://docs.opencv.org/master/opencv.js';
                script.async = true;
                script.onload = resolve;
                document.head.appendChild(script);
            });

            cap = new cv.VideoCapture(video);
        }

        function iniciarRastreamentoObjeto(x, y) {
            if (tracker) {
                tracker.delete();
            }

            const bbox = new cv.Rect(x - 25, y - 25, 50, 50); // Ajuste o tamanho conforme necessário
            tracker = new cv.TrackerMedianFlow();
            tracker.init(video, bbox);
        }

        function processarFrame() {
            cap.read(video);
            ctx.drawImage(video, 0, 0, video.width, video.height);
            if (tracker !== undefined) {
                const bbox = new cv.Rect();
                const success = tracker.update(video, bbox);
                if (success) {
                    ctx.beginPath();
                    ctx.moveTo(bbox.x, bbox.y);
                    ctx.lineTo(bbox.x + bbox.width, bbox.y + bbox.height);
                    ctx.strokeStyle = '#00FF00';
                    ctx.lineWidth = 2;
                    ctx.stroke();
                }
            }
        }

        iniciarRastreamento();
    </script>
</body>
</html>
