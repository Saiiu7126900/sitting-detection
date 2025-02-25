<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ตรวจจับการนั่ง</title>
 
    <style>
        body {
            font-family: 'Arial', sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
 
        h1 {
            color: #333;
            font-size: 2.5rem;
            margin-bottom: 20px;
        }
 
        canvas {
            border: 2px solid #ff6f61;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }
 
        .button-container {
            margin: 20px 0;
        }
 
        button {
            margin: 10px;
            padding: 12px 24px;
            font-size: 1rem;
            color: #fff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease, transform 0.2s ease;
        }
 
        button:hover {
            transform: scale(1.05);
        }
 
        button:active {
            transform: scale(0.95);
        }
 
        button:nth-child(1) {
            background-color: #28a745;
        }
 
        button:nth-child(1):hover {
            background-color: #218838;
        }
 
        button:nth-child(2) {
            background-color: #dc3545;
        }
 
        button:nth-child(2):hover {
            background-color: #c82333;
        }
 
        button:nth-child(3) {
            background-color: #fd7e14;
        }
 
        button:nth-child(3):hover {
            background-color: #e86b10;
        }
 
        #label-container {
            margin-top: 20px;
            font-size: 1.2rem;
            color: #333;
        }
    </style>
</head>
 
<body>
    <h1>ตรวจจับการนั่ง</h1>
    <div class="button-container">
        <button type="button" onclick="init()">Start</button>
        <button type="button" onclick="stop()">Stop</button>
        <button type="button" onclick="reset()">Reset</button>
    </div>
    <div><canvas id="canvas" width="640" height="480"></canvas></div>
    <div id="label-container"></div>
 
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/pose@0.8/dist/teachablemachine-pose.min.js"></script>
 
    <script type="text/javascript">
        const URL = "https://teachablemachine.withgoogle.com/models/xtgDT3JxI/";
        let model, webcam, ctx, labelContainer, maxPredictions;
        let isRunning = false;
 
        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";
 
            model = await tmPose.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();
 
            const size = 400;
            const flip = true;
            webcam = new tmPose.Webcam(size, size, flip);
            await webcam.setup();
            await webcam.play();
            isRunning = true;
            window.requestAnimationFrame(loop);
 
            const canvas = document.getElementById("canvas");
            canvas.width = size; canvas.height = size;
            ctx = canvas.getContext("2d");
            labelContainer = document.getElementById("label-container");
        }
 
        async function loop(timestamp) {
            if (!isRunning) return;
            webcam.update();
            await predict();
            window.requestAnimationFrame(loop);
        }
 
        async function predict() {
    const { pose, posenetOutput } = await model.estimatePose(webcam.canvas);
    const prediction = await model.predict(posenetOutput);
 
    let outputHTML = "";
    for (let i = 0; i < maxPredictions; i++) {
        let className = prediction[i].className;
        const probability = (prediction[i].probability * 100).toFixed(0);

        let backgroundColor = "";  // Default color
        if (className === "Class 1") {
            className = "นั่งถูกหลักการยศาสตร์";
            backgroundColor = "#28a745";  // สีเขียวสำหรับการนั่งถูกหลักการยศาสตร์
        } else if (className === "Class 2") {
            className = "นั่งผิดหลักการยศาสตร์";
            backgroundColor = "#dc3545";  // สีแดงสำหรับการนั่งผิดหลักการยศาสตร์
        }
 
        outputHTML += `
                    <div class="class-label" style="background-color: ${backgroundColor}; color: #fff; padding: 10px; border-radius: 5px; margin: 5px 0;">
                        ${className}: ${probability}%
                    </div>
                `;
            }
 
    labelContainer.innerHTML = outputHTML;
    drawPose(pose);
}
 
 
 
        function drawPose(pose) {
            if (webcam.canvas) {
                ctx.drawImage(webcam.canvas, 0, 0);
                if (pose) {
                    const minPartConfidence = 0.5;
                    tmPose.drawKeypoints(pose.keypoints, minPartConfidence, ctx);
                    tmPose.drawSkeleton(pose.keypoints, minPartConfidence, ctx);
                }
            }
        }
 
        function stop() {
            isRunning = false;
            if (webcam) {
                webcam.stop();
            }
            console.log("Stopped");
        }
 
        function reset() {
            window.location.reload();
        }
    </script>
</body>
 
</html>
