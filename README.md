<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quiz Gateway</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
        .fade-in {
            animation: fadeIn 0.3s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .glass-card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }
    </style>
</head>
<body class="text-gray-800 antialiased min-h-screen flex flex-col">

    <!-- ================= LOG MASUK ================= -->
    <main id="login-page" class="flex-grow flex items-center justify-center p-4 transition-all duration-300">
        <div class="glass-card w-full max-w-md p-8 rounded-2xl fade-in">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-bold text-indigo-600 mb-2">Quiz Gateway</h1>
                <p class="text-gray-500">Sila log masuk untuk meneruskan</p>
            </div>

            <form id="login-form" class="space-y-5">
                <div>
                    <label for="nama" class="block text-sm font-medium text-gray-700 mb-1">Nama Penuh</label>
                    <input type="text" id="nama" required placeholder="Cth: Ali Bin Abu"
                        class="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none">
                </div>
                
                <div>
                    <label for="ic" class="block text-sm font-medium text-gray-700 mb-1">No. Kad Pengenalan</label>
                    <input type="text" id="ic" required placeholder="Cth: 010101-14-1234"
                        class="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none">
                </div>

                <button type="submit" 
                    class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-4 rounded-lg transition-colors duration-200 transform hover:scale-[1.02]">
                    Log Masuk
                </button>
            </form>
        </div>
    </main>

    <!-- ================= DASHBOARD ================= -->
    <main id="dashboard-page" class="hidden flex-grow p-4 md:p-8 fade-in">
        <div class="max-w-6xl mx-auto">
            <header class="flex flex-col md:flex-row justify-between items-center bg-white p-6 rounded-2xl shadow-sm mb-8">
                <div>
                    <h2 class="text-2xl font-bold text-gray-800">Selamat Datang, <span id="display-nama" class="text-indigo-600">User</span>!</h2>
                    <p class="text-sm text-gray-500 mt-1">IC: <span id="display-ic"></span></p>
                </div>
                <div class="flex flex-col md:flex-row gap-2 mt-4 md:mt-0">
                    <button onclick="downloadCSV()" 
                        class="px-5 py-2 bg-green-50 hover:bg-green-100 text-green-700 font-medium rounded-lg transition-colors border border-green-200">
                        Muat Turun CSV
                    </button>
                    <button onclick="logout()" 
                        class="px-5 py-2 bg-red-50 hover:bg-red-100 text-red-600 font-medium rounded-lg transition-colors border border-red-200">
                        Log Keluar
                    </button>
                </div>
            </header>

            <div class="mb-6">
                <h3 class="text-xl font-bold text-gray-800">Senarai Kuiz</h3>
                <p class="text-gray-500 text-sm">Sila selesaikan kuiz-kuiz di bawah.</p>
            </div>

            <div id="quiz-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- Card kuiz akan di-generate oleh JS -->
            </div>
        </div>
    </main>

    <!-- ================= JAVASCRIPT ================= -->
    <script>
        // 1. Data Kuiz
        const quizzes = [
            { id: "quiz1", tajuk: "Quiz Ibadah Set 1", link: "https://kahoot.it/solo?quizId=1e00ac9e-46ca-47c7-87e3-06fad8788fd4" },
            { id: "quiz2", tajuk: "Quiz Ibadah Set 2", link: "https://create.kahoot.it/solo?quizId=dfded675-6924-4433-9293-bd5f91ddf934&gameMode=normal" },
            { id: "quiz3", tajuk: "Sejarah Tingkatan 5 Bab 3", link: "https://quizizz.com/join/placeholder-3" },
            { id: "quiz4", tajuk: "Fizik Kuantum Asas", link: "https://quizizz.com/join/placeholder-4" }
        ];

        let timerInterval = null;

        document.addEventListener('DOMContentLoaded', () => {
            checkAuthStatus();
            document.getElementById('login-form').addEventListener('submit', (e) => {
                e.preventDefault();
                login();
            });
        });

        function checkAuthStatus() {
            const userDataStr = localStorage.getItem('quizUserData');
            if (userDataStr) {
                showDashboard(JSON.parse(userDataStr));
            } else {
                showLogin();
            }
        }

        function login() {
            const namaInput = document.getElementById('nama').value.trim();
            const icInput = document.getElementById('ic').value.trim();

            if (!namaInput || !icInput) return alert("Sila isikan nama dan nombor IC.");

            const userData = {
                nama: namaInput,
                ic: icInput,
                quizStatus: {}
            };

            localStorage.setItem('quizUserData', JSON.stringify(userData));
            document.getElementById('login-form').reset();
            showDashboard(userData);
        }

        function logout() {
            if(confirm("Adakah anda pasti mahu log keluar?")) {
                localStorage.removeItem('quizUserData');
                if(timerInterval) clearInterval(timerInterval);
                showLogin();
            }
        }

        function showLogin() {
            document.getElementById('dashboard-page').classList.add('hidden');
            document.getElementById('login-page').classList.remove('hidden');
        }

        function showDashboard(userData) {
            document.getElementById('login-page').classList.add('hidden');
            document.getElementById('dashboard-page').classList.remove('hidden');
            
            document.getElementById('display-nama').textContent = userData.nama;
            document.getElementById('display-ic').textContent = userData.ic;

            renderQuizzes(userData);
            
            if(timerInterval) clearInterval(timerInterval);
            timerInterval = setInterval(() => updateTimers(), 1000);
        }

        function renderQuizzes(userData) {
            const grid = document.getElementById('quiz-grid');
            grid.innerHTML = ''; 

            quizzes.forEach(quiz => {
                const statusData = userData.quizStatus[quiz.id] || { status: 'not_attempted' };
                let badgeHTML = '', buttonHTML = '', timerHTML = '';

                if (statusData.status === 'not_attempted') {
                    badgeHTML = `<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-gray-100 text-gray-800">❌ Not Attempted</span>`;
                    buttonHTML = `<button onclick="startQuiz('${quiz.id}', '${quiz.link}')" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-lg transition">Start Quiz</button>`;
                } else if (statusData.status === 'attempted') {
                    badgeHTML = `<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-yellow-100 text-yellow-800">🟡 In Progress</span>`;
                    buttonHTML = `
                        <div class="grid grid-cols-2 gap-2 w-full">
                            <button onclick="startQuiz('${quiz.id}', '${quiz.link}')" class="bg-gray-100 hover:bg-gray-200 text-gray-700 font-medium py-2 px-3 rounded-lg text-sm transition">Resume</button>
                            <button onclick="markAsDone('${quiz.id}')" class="bg-emerald-500 hover:bg-emerald-600 text-white font-medium py-2 px-3 rounded-lg text-sm transition shadow-sm">Mark as Done</button>
                        </div>`;
                    timerHTML = `<div class="mt-3 text-sm font-mono text-orange-600 bg-orange-50 py-1.5 px-3 rounded inline-block">Masa: <span id="timer-${quiz.id}">00:00</span></div>`;
                } else if (statusData.status === 'done') {
                    badgeHTML = `<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-emerald-100 text-emerald-800">✅ Done</span>`;
                    buttonHTML = `<button disabled class="w-full bg-gray-100 text-gray-400 font-medium py-2 px-4 rounded-lg cursor-not-allowed">Selesai</button>`;
                    timerHTML = `<div class="mt-3 text-sm text-emerald-600 font-medium">Disiapkan dalam: ${calculateTimeTaken(statusData.startTime, statusData.endTime)}<br><span class="text-indigo-600">Markah: ${statusData.markah || 'N/A'}</span></div>`;
                }

                const card = document.createElement('div');
                card.className = "bg-white rounded-xl p-6 shadow-sm border border-gray-100 hover:shadow-md transition-shadow duration-200 flex flex-col justify-between";
                card.innerHTML = `
                    <div>
                        <h4 class="text-lg font-bold text-gray-800 mb-4 leading-tight">${quiz.tajuk}</h4>
                        <div class="mb-4">${badgeHTML}</div>
                        ${timerHTML}
                    </div>
                    <div class="mt-6">${buttonHTML}</div>
                `;
                grid.appendChild(card);
            });
            updateTimers();
        }

        function startQuiz(quizId, link) {
            const userData = JSON.parse(localStorage.getItem('quizUserData'));
            if (!userData.quizStatus[quizId] || userData.quizStatus[quizId].status !== 'attempted') {
                userData.quizStatus[quizId] = { status: 'attempted', startTime: Date.now() };
                localStorage.setItem('quizUserData', JSON.stringify(userData));
                renderQuizzes(userData);
            }
            window.open(link, '_blank');
        }

        function markAsDone(quizId) {
            if(confirm("Adakah anda pasti sudah selesai menjawab kuiz ini?")) {
                const userData = JSON.parse(localStorage.getItem('quizUserData'));
                if (userData.quizStatus[quizId]) {
                    userData.quizStatus[quizId].status = 'done';
                    userData.quizStatus[quizId].endTime = Date.now();
                    
                    // Markah disetkan secara automatik sebagai notis rujukan 
                    // kerana data dari Kahoot tidak boleh ditarik secara langsung
                    userData.quizStatus[quizId].markah = "Semak di Dashboard Kuiz"; 
                    
                    localStorage.setItem('quizUserData', JSON.stringify(userData));
                    renderQuizzes(userData);
                    
                    // Panggil fungsi hantar data selepas UI di-update
                    sendDataToSpreadsheet(userData, quizId);
                }
            }
        }

        // Google Sheets ge data kaluhisuvudu
        function sendDataToSpreadsheet(userData, quizId) {
            const googleAppScriptURL = "https://script.google.com/macros/s/AKfycbwc3ubJHH3yZdjy--keSp3xGU7w6_VGk4MCfYEIf7M4U0SLBxdf1WVa9UV7nEyAQn4/exec"; 

            const qData = userData.quizStatus[quizId];
            const date = new Date(qData.endTime);
            const tarikh = `${String(date.getMonth() + 1).padStart(2, '0')}/${String(date.getDate()).padStart(2, '0')}`;
            
            // Dapatkan tajuk kuiz berdasarkan ID
            const kuizInfo = quizzes.find(q => q.id === quizId);
            const namaKuiz = kuizInfo ? kuizInfo.tajuk : quizId;
            
            // Format data baru untuk memastikan Google Sheets menerimanya dengan baik
            const formData = new URLSearchParams();
            formData.append('tarikh', tarikh);
            formData.append('tahun', date.getFullYear());
            formData.append('ic', userData.ic);
            formData.append('nama', userData.nama);
            formData.append('masaMula', new Date(qData.startTime).toLocaleTimeString('en-US', {hour12: false}));
            formData.append('masaSelesai', new Date(qData.endTime).toLocaleTimeString('en-US', {hour12: false}));
            formData.append('kuizID', namaKuiz);
            formData.append('masaDiambil', calculateTimeTaken(qData.startTime, qData.endTime));
            formData.append('markah', qData.markah || 'N/A'); // Hantar rekod markah
            formData.append('peranti', navigator.platform);

            // Beritahu pengguna data sedang dihantar
            const toast = document.createElement('div');
            toast.className = "fixed bottom-5 right-5 bg-indigo-600 text-white px-4 py-2 rounded shadow-lg transition-opacity duration-300";
            toast.innerText = "Menghantar data ke server...";
            document.body.appendChild(toast);

            fetch(googleAppScriptURL, {
                method: 'POST',
                mode: 'no-cors', // Penting untuk elak CORS block dari Google
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                },
                body: formData.toString()
            }).then(() => {
                toast.innerText = "✅ Rekod berjaya disimpan!";
                toast.classList.replace('bg-indigo-600', 'bg-emerald-500');
                setTimeout(() => document.body.removeChild(toast), 3000);
            }).catch(err => {
                toast.innerText = "❌ Ralat menghantar data.";
                toast.classList.replace('bg-indigo-600', 'bg-red-500');
                setTimeout(() => document.body.removeChild(toast), 3000);
            });
        }

        // Muat Turun CSV
        function downloadCSV() {
            const userData = JSON.parse(localStorage.getItem('quizUserData'));
            if (!userData) return alert("Tiada data untuk dimuat turun.");

            const date = new Date();
            const tarikh = `${String(date.getMonth() + 1).padStart(2, '0')}/${String(date.getDate()).padStart(2, '0')}`;
            const tahun = date.getFullYear();

            let csvContent = "Tarikh Jawab (MM/DD),Tahun,IC (ID),Nama,Quiz Ibadah Set 1,Quiz Ibadah Set 2,Peranti\n";
            
            const getQStatus = (qId) => {
                const q = userData.quizStatus[qId];
                if (!q) return "Belum Dijawab";
                if (q.status === 'attempted') return "Sedang Dijawab";
                return `Selesai (${calculateTimeTaken(q.startTime, q.endTime)}) | Markah: ${q.markah || 'N/A'}`;
            };

            const row = `"${tarikh}","${tahun}","${userData.ic}","${userData.nama}","${getQStatus('quiz1')}","${getQStatus('quiz2')}","${navigator.platform}"`;
            csvContent += row;

            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.download = `Quiz_Data_${userData.ic}.csv`;
            link.click();
        }

        function updateTimers() {
            const rawData = localStorage.getItem('quizUserData');
            if(!rawData) return;
            const userData = JSON.parse(rawData);
            const now = Date.now();

            quizzes.forEach(quiz => {
                const statusData = userData.quizStatus[quiz.id];
                if (statusData && statusData.status === 'attempted') {
                    const timerElement = document.getElementById(`timer-${quiz.id}`);
                    if (timerElement) {
                        const elapsedMs = now - statusData.startTime;
                        timerElement.textContent = formatTime(elapsedMs);
                    }
                }
            });
        }

        function formatTime(ms) {
            const totalSeconds = Math.floor(ms / 1000);
            const minutes = Math.floor(totalSeconds / 60);
            const seconds = totalSeconds % 60;
            return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
        }

        function calculateTimeTaken(start, end) {
            if(!start || !end) return "N/A";
            const elapsedMs = end - start;
            const totalSeconds = Math.floor(elapsedMs / 1000);
            const minutes = Math.floor(totalSeconds / 60);
            const seconds = totalSeconds % 60;
            
            if (minutes > 0) return `${minutes} min ${seconds} saat`;
            return `${seconds} saat`;
        }
    </script>
</body>
</html>
