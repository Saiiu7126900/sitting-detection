<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sitting Detection</title>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image"></script>
</head>
<body>
  <h1>Sitting Detection</h1>
  
  <!-- ส่วนที่ใช้แสดงผล -->
  <p id="predictionResult">กำลังตรวจสอบ...</p>

  <!-- ส่วนที่ใช้แสดงผลวีดีโอจากกล้อง -->
  <video id="webcam" width="640" height="480" autoplay></video>

  <script>
    // ใส่โค้ด JavaScript ของคุณที่นี่
    let model, webcam, predictionResult;

    async function loadModel() {
      const URL = 'https://drive.google.com/uc?export=download&id=1ng7ojVNtClfwu0S_1-3f5lqH5JkFi17v'; // URL ที่ถูกต้องจาก Google Drive
      model = await tmImage.load(URL); // โหลดโมเดล
      console.log('โมเดลโหลดเสร็จแล้ว');
      startWebcam();
    }

    function startWebcam() {
      navigator.mediaDevices.getUserMedia({
        video: true
      }).then((stream) => {
        webcam = document.getElementById('webcam');
        webcam.srcObject = stream;
        detectSitting();
      });
    }

    async function detectSitting() {
      const predictions = await model.predict(webcam); // ถ่ายภาพจาก webcam แล้วทำนาย
      predictionResult = predictions[0].className; // รับค่าผลลัพธ์จากการทำนาย

      // แสดงผลการทำนาย
      const resultElement = document.getElementById('predictionResult');
      if (predictionResult === 'Correct Sitting') {
        resultElement.innerText = 'การนั่งถูกต้อง!';
        resultElement.style.color = 'green';
      } else if (predictionResult === 'Incorrect Sitting') {
        resultElement.innerText = 'การนั่งผิดพลาด!';
        resultElement.style.color = 'red';
      }

      // ทำการตรวจจับใหม่ทุก 1000ms (1 วินาที)
      setTimeout(detectSitting, 1000);
    }

    // โหลดโมเดลเมื่อหน้าทเว็บโหลดเสร็จ
    loadModel();
  </script>
</body>
</html>
