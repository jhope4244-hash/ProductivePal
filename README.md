# ProductivePal
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ProductivePal</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@700&display=swap" rel="stylesheet">
<script src="https://cdn.tailwindcss.com"></script>
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#3b82f6">
<style>#progress-ring{transform:rotate(-90deg);transform-origin:50% 50%;transition:stroke-dashoffset 1s linear}</style>
</head>
<body class="bg-gray-50 font-inter">
<header class="sticky top-0 bg-white shadow p-4 flex justify-between items-center z-10"><h1 class="text-xl font-semibold">ProductivePal</h1><nav class="flex space-x-4"><button id="timer-tab" class="font-medium underline">Timer</button><button id="tracker-tab" class="font-medium">Tracker</button><button id="settings-tab" class="font-medium">Settings</button></nav></header>
<main class="max-w-6xl mx-auto p-6 flex flex-col items-center gap-8">
<div class="w-full flex flex-col items-center gap-4"><input id="work-input" type="text" placeholder="What are you working on?" maxlength="50" class="p-4 border rounded-lg w-full md:w-1/2 focus:ring-2 focus:ring-blue-500"></div>
<div class="relative flex flex-col items-center justify-center">
<svg class="w-72 h-72 md:w-96 md:h-96" viewBox="0 0 200 200"><circle cx="100" cy="100" r="90" stroke="#e5e7eb" stroke-width="10" fill="none"/><circle id="progress-ring" cx="100" cy="100" r="90" stroke="#3b82f6" stroke-width="10" fill="none" stroke-dasharray="565.48" stroke-dashoffset="565.48"/></svg>
<div class="absolute inset-0 flex flex-col items-center justify-center">
<div id="timer-display" class="text-8xl font-mono font-bold">25:00</div>
<div class="flex space-x-4 mt-6"><button id="start-btn" class="px-8 py-4 bg-blue-500 text-white rounded-full">Start</button><button id="pause-btn" class="px-8 py-4 bg-gray-300 rounded-full">Pause</button><button id="reset-btn" class="px-8 py-4 bg-red-500 text-white rounded-full">Reset</button></div>
<div class="mt-4 text-lg font-medium" id="session-type">Focus</div>
</div></div>
<section id="stats" class="w-full grid grid-cols-1 md:grid-cols-3 gap-6">
<div class="bg-white rounded-xl p-6 shadow"><div class="text-5xl font-bold" id="sessions-completed">0</div><div class="text-sm uppercase tracking-wide">Sessions Completed</div></div>
<div class="bg-white rounded-xl p-6 shadow"><div class="text-5xl font-bold" id="focus-hours">0</div><div class="text-sm uppercase tracking-wide">Focus Hours</div></div>
<div class="bg-white rounded-xl p-6 shadow"><div class="text-5xl font-bold" id="breaks-taken">0</div><div class="text-sm uppercase tracking-wide">Breaks Taken</div></div>
</section>
<div id="toasts" class="fixed top-4 right-4 flex flex-col gap-2 z-50"></div>
</main>
<div id="settings-modal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
<div class="bg-white rounded-xl p-8 w-96"><h2 class="text-2xl font-semibold mb-4">Settings</h2>
<div class="flex flex-col gap-4">
<label>Pomodoro Duration (minutes)<input id="pomodoro-duration" type="number" min="1" value="25" class="w-full border rounded p-2"></label>
<label>Short Break (minutes)<input id="short-break-duration" type="number" min="1" value="5" class="w-full border rounded p-2"></label>
<label>Long Break (minutes)<input id="long-break-duration" type="number" min="1" value="15" class="w-full border rounded p-2"></label>
<label>Auto-start Breaks<input id="auto-break-toggle" type="checkbox" checked class="ml-2"></label>
<button id="close-settings" class="mt-4 px-6 py-3 bg-blue-500 text-white rounded-full">Close</button>
</div></div></div>
<script>
const timerDisplay=document.getElementById("timer-display");const startBtn=document.getElementById("start-btn");const pauseBtn=document.getElementById("pause-btn");const resetBtn=document.getElementById("reset-btn");const progressRing=document.getElementById("progress-ring");const sessionTypeDisplay=document.getElementById("session-type");const sessionsCompletedDisplay=document.getElementById("sessions-completed");const breaksTakenDisplay=document.getElementById("breaks-taken");const settingsModal=document.getElementById("settings-modal");const closeSettingsBtn=document.getElementById("close-settings");const pomodoroInput=document.getElementById("pomodoro-duration");const shortBreakInput=document.getElementById("short-break-duration");const longBreakInput=document.getElementById("long-break-duration");const autoBreakToggle=document.getElementById("auto-break-toggle");let pomodoroDuration=parseInt(localStorage.getItem("pomodoroDuration")||25)*60;let shortBreakDuration=parseInt(localStorage.getItem("shortBreakDuration")||5)*60;let longBreakDuration=parseInt(localStorage.getItem("longBreakDuration")||15)*60;let timerInterval=null;let remainingSeconds=parseInt(localStorage.getItem("remainingSeconds")||pomodoroDuration);let sessionType=localStorage.getItem("sessionType")||"Focus";let sessionsCompleted=parseInt(localStorage.getItem("sessionsCompleted")||0);let breaksTaken=parseInt(localStorage.getItem("breaksTaken")||0);sessionsCompletedDisplay.textContent=sessionsCompleted;breaksTakenDisplay.textContent=breaksTaken;function updateDisplay(){const minutes=String(Math.floor(remainingSeconds/60)).padStart(2,"0");const seconds=String(remainingSeconds%60).padStart(2,"0");timerDisplay.textContent=`${minutes}:${seconds}`;const circumference=2*Math.PI*90;const offset=circumference-(remainingSeconds/(sessionType==="Focus"?pomodoroDuration:sessionType==="Short Break"?shortBreakDuration:longBreakDuration))*circumference;progressRing.style.strokeDashoffset=offset;sessionTypeDisplay.textContent=sessionType;}function saveState(){localStorage.setItem("pomodoroDuration",pomodoroDuration/60);localStorage.setItem("shortBreakDuration",shortBreakDuration/60);localStorage.setItem("longBreakDuration",longBreakDuration/60);localStorage.setItem("remainingSeconds",remainingSeconds);localStorage.setItem("sessionType",sessionType);localStorage.setItem("sessionsCompleted",sessionsCompleted);localStorage.setItem("breaksTaken",breaksTaken);}function startTimer(){if(timerInterval)return;timerInterval=setInterval(()=>{remainingSeconds--;updateDisplay();saveState();if(remainingSeconds<=0){clearInterval(timerInterval);timerInterval=null;showToast(`${sessionType} session complete!`);nextSession();}},1000);}function pauseTimer(){clearInterval(timerInterval);timerInterval=null;}function resetTimer(){remainingSeconds=sessionType==="Focus"?pomodoroDuration:sessionType==="Short Break"?shortBreakDuration:longBreakDuration;updateDisplay();clearInterval(timerInterval);timerInterval=null;saveState();}function nextSession(){if(sessionType==="Focus"){sessionsCompleted++;sessionsCompletedDisplay.textContent=sessionsCompleted;sessionType=sessionsCompleted%4===0?"Long Break":"Short Break";breaksTaken++;breaksTakenDisplay.textContent=breaksTaken;remainingSeconds=sessionType==="Short Break"?shortBreakDuration:longBreakDuration;if(autoBreakToggle.checked)startTimer();}else{sessionType="Focus";remainingSeconds=pomodoroDuration;}updateDisplay();saveState();}function showToast(message){const toastContainer=document.getElementById("toasts");const toast=document.createElement("div");toast.className="bg-blue-500 text-white p-4 rounded shadow";toast.textContent=message;toastContainer.appendChild(toast);setTimeout(()=>{toast.remove();},5000);}startBtn.addEventListener("click",startTimer);pauseBtn.addEventListener("click",pauseTimer);resetBtn.addEventListener("click",resetTimer);document.getElementById("settings-tab").addEventListener("click",()=>{settingsModal.classList.remove("hidden");});closeSettingsBtn.addEventListener("click",()=>{pomodoroDuration=parseInt(pomodoroInput.value)*60;shortBreakDuration=parseInt(shortBreakInput.value)*60;longBreakDuration=parseInt(longBreakInput.value)*60;resetTimer();settingsModal.classList.add("hidden");saveState();});updateDisplay();if('serviceWorker'in navigator){navigator.serviceWorker.register('service-worker.js');}
</script>
</body>
</html>
{
  "name": "ProductivePal",
  "short_name": "ProductivePal",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#f9fafb",
  "theme_color": "#3b82f6",
  "icons": [
    {
      "src": "https://via.placeholder.com/192",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "https://via.placeholder.com/512",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
self.addEventListener('install', e => { console.log('Service Worker installed'); });
self.addEventListener('fetch', e => { e.respondWith(fetch(e.request)); });


