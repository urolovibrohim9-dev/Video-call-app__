# Video-call-app__
<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>Til tanlash</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h1>Tilni tanlang / Select Language</h1>
    <button onclick="setLanguage('uz')">O‘zbekcha</button>
    <button onclick="setLanguage('en')">English</button>
    <br><br>
    <button id="toggleTheme">Qora/Oq fon</button>
  </div>
  <script src="script.js"></script>
</body>
</html><!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>Ro‘yxatdan o‘tish</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h1>Ro‘yxatdan o‘ting</h1>
    <form id="registerForm">
      <input type="text" id="name" placeholder="Ismingiz" required><br><br>
      <input type="text" id="phone" placeholder="Telefon raqamingiz" required><br><br>
      <button type="submit">Ro‘yxatdan o‘tish</button>
    </form>
  </div>
  <script src="firebase-config.js"></script>
  <script src="script.js"></script>
</body>
</html><!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>Video qo‘ng‘iroq</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="container">
    <h1>Video Qo‘ng‘iroq</h1>
    <video id="remoteVideo" autoplay playsinline></video>
    <video id="localVideo" autoplay muted playsinline></video>
    <br>
    <button id="switchCamera">Kamerani teskari qilish</button>
    <input type="range" id="volumeSlider" min="0" max="1" step="0.01">
  </div>
  <audio id="ringtone" src="ringtone.wav" loop></audio>
  <script src="script.js"></script>
</body>
</html>body.light {
  background-color: #ffffff;
  color: #000000;
}

body.dark {
  background-color: #000000;
  color: #ffffff;
}

.container {
  max-width: 600px;
  margin: auto;
  text-align: center;
}

video {
  width: 80%;
  max-width: 400px;
  border: 2px solid #333;
  margin: 10px;
}// Til tanlash
function setLanguage(lang) {
    alert("Til tanlandi: " + lang);
    window.location.href = "register.html";
}

// Dark/Light mode
const toggleBtn = document.getElementById('toggleTheme');
if(toggleBtn){
  toggleBtn.addEventListener('click', () => {
      document.body.classList.toggle('dark');
      document.body.classList.toggle('light');
      localStorage.setItem('theme', document.body.className);
  });

  const savedTheme = localStorage.getItem('theme');
  if (savedTheme) document.body.className = savedTheme;
  else document.body.className = 'light';
}

// Ro'yxatdan o'tish
const registerForm = document.getElementById('registerForm');
if(registerForm){
  registerForm.addEventListener('submit', (e)=>{
    e.preventDefault();
    const name = document.getElementById('name').value;
    const phone = document.getElementById('phone').value;
    firebase.firestore().collection('users').add({name, phone})
      .then(()=>{ window.location.href = "call.html"; });
  });
}

// Video qo'ng'iroq (WebRTC + Socket.IO minimal)
let localStream, remoteStream;
const localVideo = document.getElementById('localVideo');
const remoteVideo = document.getElementById('remoteVideo');
const switchCameraBtn = document.getElementById('switchCamera');
const volumeSlider = document.getElementById('volumeSlider');
const ringtone = document.getElementById('ringtone');

async function startVideo() {
    localStream = await navigator.mediaDevices.getUserMedia({video:true, audio:true});
    localVideo.srcObject = localStream;
    remoteStream = new MediaStream();
    remoteVideo.srcObject = remoteStream;
}

if(switchCameraBtn){
  switchCameraBtn.addEventListener('click', async () => {
      const videoTrack = localStream.getVideoTracks()[0];
      const constraints = videoTrack.getConstraints();
      constraints.facingMode = (constraints.facingMode === "user") ? "environment" : "user";
      const newStream = await navigator.mediaDevices.getUserMedia({video: constraints, audio: true});
      localVideo.srcObject = newStream;
      localStream = newStream;
  });
}

if(volumeSlider){
  volumeSlider.addEventListener('input', (e)=>{
      remoteVideo.volume = e.target.value;
  });
}

startVideo();// Firebase konfiguratsiyasi
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_BUCKET",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
};
firebase.initializeApp(firebaseConfig);const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

io.on('connection', (socket) => {
    console.log('User connected: ' + socket.id);

    socket.on('offer', (data) => {
        socket.broadcast.emit('offer', data);
    });

    socket.on('answer', (data) => {
        socket.broadcast.emit('answer', data);
    });

    socket.on('ice-candidate', (data) => {
        socket.broadcast.emit('ice-candidate', data);
    });

    socket.on('disconnect', () => {
        console.log('User disconnected: ' + socket.id);
    });
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});
