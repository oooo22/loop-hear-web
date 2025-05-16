<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Feedback Loop Demo</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      text-align: center;
      padding: 50px;
      background: #E6DFF3; /* สีม่วงพาสเทล */
      color: #5A3E85; /* ม่วงเข้ม */
    }
    button {
      font-size: 18px;
      padding: 12px 24px;
      margin: 12px;
      cursor: pointer;
      border: none;
      border-radius: 8px;
      box-shadow: 0 4px 6px rgba(90, 62, 133, 0.3);
      transition: background-color 0.3s ease, color 0.3s ease;
      font-weight: 600;
    }
    #startBtn {
      background-color: #F6E27F; /* เหลืองพาสเทล */
      color: #5A3E85;
    }
    #startBtn:hover {
      background-color: #E5D46B;
    }
    #stopBtn {
      background-color: #B59ED9; /* ม่วงพาสเทลเข้ม */
      color: #FFF;
    }
    #stopBtn:hover {
      background-color: #977AC6;
    }
    #goBtn {
      background-color: #F6E27F; /* เหลืองพาสเทล */
      color: #5A3E85;
    }
    #goBtn:hover {
      background-color: #E5D46B;
    }
    h2 {
      font-weight: 700;
      margin-bottom: 5px;
    }
    p {
      font-size: 16px;
      margin-bottom: 30px;
      font-weight: 500;
    }
  </style>
</head>
<body>
  <h2>Feedback Echo Demo</h2>
  <p>กดค้างเพื่อพูด → ปล่อยเพื่อให้เสียงก้องค้างอยู่</p>

  <!-- ปุ่มควบคุม -->
  <button id="startBtn">🎤 กดค้างเพื่อเริ่มจับเสียง</button>
  <button id="stopBtn">🛑 หยุดเสียงทั้งหมด</button>
  <br />
  <button id="goBtn">🔗 ไปเว็บที่คุณต้องการ</button>

  <script>
    let audioCtx;
    let micStream;
    let micSource;
    let delayNode;
    let feedbackGain;
    let loopInterval;

    const startBtn = document.getElementById('startBtn');
    const stopBtn = document.getElementById('stopBtn');
    const goBtn = document.getElementById('goBtn');

    async function initAudio() {
      if (!audioCtx) {
        audioCtx = new AudioContext();
        delayNode = audioCtx.createDelay(1.0);
        delayNode.delayTime.value = 0.25;

        feedbackGain = audioCtx.createGain();
        feedbackGain.gain.value = 0.3;

        delayNode.connect(feedbackGain);
        feedbackGain.connect(delayNode);
        delayNode.connect(audioCtx.destination);

        // เพิ่ม feedback ทีละนิดเรื่อยๆ
        loopInterval = setInterval(() => {
          if (feedbackGain.gain.value < 0.95) {
            feedbackGain.gain.value += 0.01;
            console.log("เพิ่มความก้อง:", feedbackGain.gain.value.toFixed(2));
          }
        }, 5000);
      }
    }

    // กดค้างไว้เพื่อเริ่มจับเสียง
    startBtn.addEventListener('mousedown', async () => {
      await initAudio();

      try {
        micStream = await navigator.mediaDevices.getUserMedia({ audio: true });
        micSource = audioCtx.createMediaStreamSource(micStream);
        micSource.connect(delayNode);
        console.log("🎤 เริ่มจับเสียง");
      } catch (err) {
        alert("ไม่สามารถเปิดไมโครโฟนได้: " + err.message);
      }
    });

    // ปล่อยปุ่มเพื่อหยุดจับเสียง
    startBtn.addEventListener('mouseup', () => {
      if (micStream) {
        micStream.getTracks().forEach(track => track.stop());
        micStream = null;
        micSource = null;
        console.log("📴 ปิดไมโครโฟนแล้ว เสียงก้องยังอยู่");
      }
    });

    // รองรับการใช้งานบนมือถือ (touch screen)
    startBtn.addEventListener('touchstart', async (e) => {
      e.preventDefault();
      await initAudio();

      try {
        micStream = await navigator.mediaDevices.getUserMedia({ audio: true });
        micSource = audioCtx.createMediaStreamSource(micStream);
        micSource.connect(delayNode);
        console.log("🎤 เริ่มจับเสียง (touch)");
      } catch (err) {
        alert("ไม่สามารถเปิดไมโครโฟนได้: " + err.message);
      }
    });

    startBtn.addEventListener('touchend', () => {
      if (micStream) {
        micStream.getTracks().forEach(track => track.stop());
        micStream = null;
        micSource = null;
        console.log("📴 ปิดไมโครโฟนแล้ว (touch)");
      }
    });

    // หยุดเสียงทั้งหมด
    stopBtn.onclick = () => {
      if (micStream) {
        micStream.getTracks().forEach(track => track.stop());
        micStream = null;
      }

      if (loopInterval) clearInterval(loopInterval);
      if (audioCtx) {
        audioCtx.close();
        audioCtx = null;
      }

      console.log("🛑 หยุดเสียงทั้งหมดและปิดระบบ");

      // รีเซ็ตทุกอย่าง
      micSource = null;
      delayNode = null;
      feedbackGain = null;
    };

    // ปุ่มไปเว็บที่คุณต้องการ
    goBtn.addEventListener('click', () => {
      window.location.href = 'https://your-destination-website.com'; // เปลี่ยน URL ตรงนี้เป็นเว็บที่ต้องการ
    });
  </script>
</body>
</html>
