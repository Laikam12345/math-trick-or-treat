<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>แบบทดสอบเรื่องจำนวนเต็ม</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutrals (Stone, Slate, Emerald, Red) -->
    <!-- Application Structure Plan: A sequential, single-page application with three distinct views: Start (now with name input), Quiz (one question at a time with immediate feedback), and Results (score, visual chart, and a review section for incorrect answers, with score saving). This structure turns a static test into an interactive learning tool by focusing the user, providing instant reinforcement, enabling targeted review of mistakes, and now allowing score tracking. -->
    <!-- Visualization & Content Choices: Questions are presented individually for focus. Multiple-choice options are interactive buttons for clear user action. Feedback is given via color-coded buttons (HTML/CSS) for instant recognition. The final score is visualized with a Chart.js Donut Chart to give a quick, graphical sense of performance (Correct vs. Incorrect). An HTML list is used to present the incorrect answers for review, which is a key learning feature. All choices are designed to maximize engagement and learning effectiveness, now with added score persistence. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
        }
        .correct-answer {
            background-color: #10B981 !important;
            color: white !important;
            border-color: #059669 !important;
        }
        .incorrect-answer {
            background-color: #EF4444 !important;
            color: white !important;
            border-color: #DC2626 !important;
        }
        .disabled-option {
            pointer-events: none;
            opacity: 0.7;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 300px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 300px;
        }
    </style>
</head>
<body class="bg-stone-100 text-slate-800 flex items-center justify-center min-h-screen p-4">

    <div id="app" class="w-full max-w-2xl mx-auto">

        <!-- Start Screen -->
        <div id="start-screen" class="text-center bg-white p-8 rounded-2xl shadow-lg">
            <h1 class="text-3xl md:text-4xl font-bold text-slate-700 mb-4">ทดสอบความรู้เรื่องจำนวนเต็ม</h1>
            <p class="text-slate-600 mb-8">มาทบทวนความเข้าใจเรื่องการบวก ลบ คูณ และหารจำนวนเต็มกันเถอะ! แบบทดสอบนี้มีทั้งหมด 20 ข้อ</p>
            <div class="mb-6">
                <label for="student-name" class="block text-slate-700 text-lg font-bold mb-2">ชื่อนักเรียน:</label>
                <input type="text" id="student-name" class="shadow appearance-none border rounded-lg w-full py-3 px-4 text-slate-700 leading-tight focus:outline-none focus:shadow-outline" placeholder="กรอกชื่อของคุณ" required>
                <div id="name-input-feedback" class="text-red-500 text-sm mt-2 hidden">กรุณากรอกชื่อนักเรียน</div>
            </div>
            <button onclick="startQuiz()" class="bg-slate-700 text-white font-bold py-3 px-8 rounded-lg hover:bg-slate-800 transition-colors duration-300 text-lg">
                เริ่มทำแบบทดสอบ
            </button>
        </div>

        <!-- Quiz Screen -->
        <div id="quiz-screen" class="hidden">
            <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-lg">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-bold text-slate-700">คำถามที่ <span id="question-number"></span>/20</h2>
                    <div id="score-display" class="text-lg font-bold text-emerald-600">คะแนน: <span id="score">0</span></div>
                </div>
                <p id="question-text" class="text-2xl md:text-3xl font-bold text-center my-8 min-h-[100px]"></p>
                <div id="options-container" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <!-- Options will be injected here -->
                </div>
                <div id="feedback-container" class="mt-6 text-center hidden">
                     <p id="feedback-text" class="text-lg font-medium p-3 rounded-lg"></p>
                </div>
                <div class="mt-8 text-center">
                    <button id="next-button" onclick="nextQuestion()" class="bg-slate-700 text-white font-bold py-3 px-12 rounded-lg hover:bg-slate-800 transition-colors duration-300 hidden text-lg">
                        ข้อต่อไป
                    </button>
                </div>
            </div>
        </div>

        <!-- Results Screen -->
        <div id="results-screen" class="hidden text-center bg-white p-8 rounded-2xl shadow-lg">
            <h1 class="text-3xl md:text-4xl font-bold text-slate-700 mb-4">สรุปผลคะแนน</h1>
            <p class="text-slate-600 mb-4 text-2xl">คุณทำได้ <span id="final-score" class="font-bold"></span> คะแนน</p>
            <p id="final-message" class="text-lg mb-6 font-medium"></p>
            <div class="chart-container mb-8">
                <canvas id="results-chart"></canvas>
            </div>
            <div id="review-section" class="mt-8 text-left hidden">
                <h3 class="text-2xl font-bold mb-4 text-center">ทบทวนข้อที่ตอบผิด</h3>
                <ul id="review-list" class="space-y-4">
                    <!-- Incorrect answers will be listed here -->
                </ul>
            </div>
            <button onclick="restartQuiz()" class="bg-slate-700 text-white font-bold py-3 px-8 rounded-lg hover:bg-slate-800 transition-colors duration-300 mt-8 text-lg">
                ทำแบบทดสอบอีกครั้ง
            </button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const quizData = [
            { question: "(-5) + 8 = ?", options: ["3", "-3", "13", "-13"], answer: "3", rationale: "เมื่อบวกจำนวนเต็มต่างเครื่องหมาย ให้นำค่าสัมบูรณ์มาลบกัน แล้วใส่เครื่องหมายตามจำนวนที่มีค่าสัมบูรณ์มากกว่า" },
            { question: "12 + (-7) = ?", options: ["5", "-5", "19", "-19"], answer: "5", rationale: "เมื่อบวกจำนวนเต็มต่างเครื่องหมาย ให้นำค่าสัมบูรณ์มาลบกัน แล้วใส่เครื่องหมายตามจำนวนที่มีค่าสัมบูรณ์มากกว่า" },
            { question: "(-10) + (-4) = ?", options: ["-14", "14", "-6", "6"], answer: "-14", rationale: "เมื่อบวกจำนวนเต็มที่มีเครื่องหมายเหมือนกัน ให้นำค่าสัมบูรณ์มาบวกกัน แล้วใส่เครื่องหมายเดิม" },
            { question: "9 - (-3) = ?", options: ["12", "6", "-12", "-6"], answer: "12", rationale: "การลบจำนวนเต็มคือการบวกด้วยจำนวนตรงข้าม ดังนั้น 9 - (-3) = 9 + 3" },
            { question: "(-15) - 6 = ?", options: ["-21", "-9", "9", "21"], answer: "-21", rationale: "การลบจำนวนเต็มคือการบวกด้วยจำนวนตรงข้าม ดังนั้น (-15) - 6 = (-15) + (-6)" },
            { question: "7 × (-4) = ?", options: ["-28", "28", "11", "3"], answer: "-28", rationale: "การคูณจำนวนเต็มต่างเครื่องหมาย ผลลัพธ์จะเป็นจำนวนลบ" },
            { question: "(-6) × (-5) = ?", options: ["30", "-30", "-11", "1"], answer: "30", rationale: "การคูณจำนวนเต็มที่มีเครื่องหมายเหมือนกัน (ลบคูณลบ) ผลลัพธ์จะเป็นจำนวนบวก" },
            { question: "20 ÷ (-5) = ?", options: ["-4", "4", "15", "25"], answer: "-4", rationale: "การหารจำนวนเต็มต่างเครื่องหมาย ผลลัพธ์จะเป็นจำนวนลบ" },
            { question: "(-48) ÷ 6 = ?", options: ["-8", "8", "-42", "-54"], answer: "-8", rationale: "การหารจำนวนเต็มต่างเครื่องหมาย ผลลัพธ์จะเป็นจำนวนลบ" },
            { question: "(-30) ÷ (-10) = ?", options: ["3", "-3", "40", "20"], answer: "3", rationale: "การหารจำนวนเต็มที่มีเครื่องหมายเหมือนกัน (ลบหารลบ) ผลลัพธ์จะเป็นจำนวนบวก" },
            { question: "5 + (-12) - 3 = ?", options: ["-10", "10", "-4", "4"], answer: "-10", rationale: "5 + (-12) = -7,  จากนั้น -7 - 3 = -10" },
            { question: "(-8) - (-2) + 10 = ?", options: ["4", "-4", "0", "-16"], answer: "4", rationale: "(-8) + 2 = -6, จากนั้น -6 + 10 = 4" },
            { question: "(-3) × 7 + 5 = ?", options: ["-16", "26", "-26", "-10"], answer: "-16", rationale: "ต้องคูณก่อนบวก: (-3) × 7 = -21, จากนั้น -21 + 5 = -16" },
            { question: "40 ÷ (-8) - 2 = ?", options: ["-7", "3", "-3", "7"], answer: "-7", rationale: "ต้องหารก่อนลบ: 40 ÷ (-8) = -5, จากนั้น -5 - 2 = -7" },
            { question: "(-2) × (-9) - 10 = ?", options: ["8", "-28", "28", "-8"], answer: "8", rationale: "ลบคูณลบได้บวก: (-2) × (-9) = 18, จากนั้น 18 - 10 = 8" },
            { question: "18 ÷ 3 + (-5) = ?", options: ["1", "-1", "-11", "11"], answer: "1", rationale: "ต้องหารก่อนบวก: 18 ÷ 3 = 6, จากนั้น 6 + (-5) = 1" },
            { question: "(6 - 10) × (-2) = ?", options: ["8", "-8", "-2", "2"], answer: "8", rationale: "ทำในวงเล็บก่อน: 6 - 10 = -4, จากนั้น (-4) × (-2) = 8" },
            { question: "(-25) ÷ (1 - 6) = ?", options: ["5", "-5", "2", "-2"], answer: "5", rationale: "ทำในวงเล็บก่อน: 1 - 6 = -5, จากนั้น (-25) ÷ (-5) = 5" },
            { question: "10 - ((-3) × 4) = ?", options: ["22", "-2", "-22", "2"], answer: "22", rationale: "ทำในวงเล็บก่อน: (-3) × 4 = -12, จากนั้น 10 - (-12) = 10 + 12 = 22" },
            { question: "5 × (-2) + (-15) ÷ 3 = ?", options: ["-15", "-5", "-25", "5"], answer: "-15", rationale: "ทำคูณและหารก่อน: 5 × (-2) = -10 และ (-15) ÷ 3 = -5, จากนั้น -10 + (-5) = -15" },
        ];

        let currentQuestionIndex = 0;
        let score = 0;
        let incorrectAnswers = [];
        let resultsChartInstance = null;
        let currentStudentName = '';

        const startScreen = document.getElementById('start-screen');
        const quizScreen = document.getElementById('quiz-screen');
        const resultsScreen = document.getElementById('results-screen');
        const questionNumberEl = document.getElementById('question-number');
        const questionTextEl = document.getElementById('question-text');
        const optionsContainer = document.getElementById('options-container');
        const scoreEl = document.getElementById('score');
        const nextButton = document.getElementById('next-button');
        const feedbackContainer = document.getElementById('feedback-container');
        const feedbackTextEl = document.getElementById('feedback-text');
        const studentNameInput = document.getElementById('student-name');
        const nameInputFeedback = document.getElementById('name-input-feedback');

        // Firebase Configuration - REPLACE WITH YOUR OWN FIREBASE PROJECT CONFIG
        // ไปที่ Firebase Console (console.firebase.google.com) สร้างโปรเจกต์ใหม่
        // จากนั้นไปที่ Project settings (รูปเฟือง) > General > Your apps > Web app (ไอคอน </>)
        // คัดลอกข้อมูล firebaseConfig มาวางที่นี่
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_AUTH_DOMAIN",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_STORAGE_BUCKET",
            messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
            appId: "YOUR_APP_ID"
        };

        // If you want to use a specific app ID for Firestore collection path (e.g., for multi-app setups)
        // Otherwise, it will default to the projectId from firebaseConfig
        const appIdForFirestore = firebaseConfig.projectId; // หรือกำหนดค่าเอง เช่น 'my-quiz-app'

        let app;
        let db;
        let auth;
        let userId;

        // Initialize Firebase and authenticate
        async function initializeFirebase() {
            try {
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // For GitHub Pages, we typically use anonymous sign-in or other methods.
                // If you need specific user authentication (e.g., Google Sign-In), you'll need to implement that here.
                await signInAnonymously(auth);
                userId = auth.currentUser?.uid || crypto.randomUUID(); // Fallback to random UUID if anonymous sign-in fails
                console.log("Firebase initialized and authenticated. User ID:", userId);
            } catch (error) {
                console.error("Error initializing Firebase or authenticating:", error);
            }
        }

        function startQuiz() {
            currentStudentName = studentNameInput.value.trim();
            if (!currentStudentName) {
                nameInputFeedback.classList.remove('hidden');
                return;
            } else {
                nameInputFeedback.classList.add('hidden');
            }

            startScreen.classList.add('hidden');
            quizScreen.classList.remove('hidden');
            showQuestion();
        }

        function showQuestion() {
            resetState();
            const currentQuestion = quizData[currentQuestionIndex];
            questionNumberEl.textContent = currentQuestionIndex + 1;
            questionTextEl.textContent = currentQuestion.question;
            
            currentQuestion.options.forEach(option => {
                const button = document.createElement('button');
                button.textContent = option;
                button.className = 'w-full bg-white border-2 border-slate-300 text-slate-700 font-bold p-4 rounded-lg hover:bg-slate-100 hover:border-slate-400 transition-colors duration-200 text-lg';
                button.onclick = () => selectAnswer(button, option, currentQuestion.answer, currentQuestion.rationale);
                optionsContainer.appendChild(button);
            });
        }

        function resetState() {
            optionsContainer.innerHTML = '';
            nextButton.classList.add('hidden');
            feedbackContainer.classList.add('hidden');
        }

        function selectAnswer(button, selectedOption, correctAnswer, rationale) {
            Array.from(optionsContainer.children).forEach(btn => {
                btn.classList.add('disabled-option');
                if (btn.textContent === correctAnswer) {
                    btn.classList.add('correct-answer');
                }
            });

            if (selectedOption === correctAnswer) {
                score++;
                scoreEl.textContent = score;
                feedbackTextEl.textContent = 'ถูกต้อง! ' + rationale;
                feedbackTextEl.className = 'text-lg font-medium p-3 rounded-lg bg-emerald-100 text-emerald-800';
            } else {
                button.classList.add('incorrect-answer');
                incorrectAnswers.push(quizData[currentQuestionIndex]);
                 feedbackTextEl.textContent = 'ยังไม่ถูกนะ คำตอบที่ถูกคือ ' + correctAnswer + ' เพราะ ' + rationale;
                 feedbackTextEl.className = 'text-lg font-medium p-3 rounded-lg bg-red-100 text-red-800';
            }
            feedbackContainer.classList.remove('hidden');
            nextButton.classList.remove('hidden');
        }

        function nextQuestion() {
            currentQuestionIndex++;
            if (currentQuestionIndex < quizData.length) {
                showQuestion();
            } else {
                showResults();
            }
        }

        async function showResults() {
            quizScreen.classList.add('hidden');
            resultsScreen.classList.remove('hidden');

            const finalScoreEl = document.getElementById('final-score');
            const finalMessageEl = document.getElementById('final-message');
            finalScoreEl.textContent = `${score} / ${quizData.length}`;
            
            if(score === 20) {
                finalMessageEl.textContent = "ยอดเยี่ยมมาก! คุณเข้าใจเรื่องจำนวนเต็มเป็นอย่างดี";
                finalMessageEl.className = "text-lg mb-6 font-medium text-emerald-600";
            } else if (score >= 15) {
                finalMessageEl.textContent = "เก่งมาก! พยายามอีกนิดเพื่อความสมบูรณ์แบบนะ";
                finalMessageEl.className = "text-lg mb-6 font-medium text-blue-600";
            } else {
                finalMessageEl.textContent = "ไม่เป็นไรนะ ลองทบทวนข้อที่ผิดแล้วมาลองใหม่อีกครั้ง!";
                finalMessageEl.className = "text-lg mb-6 font-medium text-orange-600";
            }

            await saveScore(currentStudentName, score);
            renderChart();
            renderReview();
        }
        
        function renderChart() {
            const ctx = document.getElementById('results-chart').getContext('2d');
            if (resultsChartInstance) {
                resultsChartInstance.destroy();
            }
            resultsChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: ['ตอบถูก', 'ตอบผิด'],
                    datasets: [{
                        data: [score, quizData.length - score],
                        backgroundColor: ['#10B981', '#EF4444'],
                        borderColor: ['#ffffff', '#ffffff'],
                        borderWidth: 4
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    cutout: '70%',
                    plugins: {
                        legend: {
                            position: 'bottom',
                             labels: {
                                font: {
                                    size: 14,
                                    family: "'Sarabun', sans-serif"
                                }
                            }
                        },
                        tooltip: {
                             titleFont: { family: "'Sarabun', sans-serif" },
                             bodyFont: { family: "'Sarabun', sans-serif" },
                        }
                    }
                }
            });
        }
        
        function renderReview() {
            const reviewSection = document.getElementById('review-section');
            const reviewList = document.getElementById('review-list');
            reviewList.innerHTML = '';
            
            if (incorrectAnswers.length > 0) {
                reviewSection.classList.remove('hidden');
                incorrectAnswers.forEach(item => {
                    const li = document.createElement('li');
                    li.className = "bg-stone-50 p-4 rounded-lg border border-stone-200";
                    li.innerHTML = `
                        <p class="font-bold text-slate-800 mb-2">${item.question}</p>
                        <p class="text-red-600"><strong>คำตอบที่ถูก:</strong> ${item.answer}</p>
                        <p class="text-slate-600 mt-1"><strong>คำอธิบาย:</strong> ${item.rationale}</p>
                    `;
                    reviewList.appendChild(li);
                });
            } else {
                 reviewSection.classList.add('hidden');
            }
        }

        async function saveScore(studentName, finalScore) {
            if (!db || !userId) {
                console.error("Firestore or User ID not available. Score cannot be saved.");
                return;
            }
            try {
                // The collection path for public data needs to use the appIdForFirestore
                const scoresCollectionRef = collection(db, `artifacts/${appIdForFirestore}/public/data/quiz_scores`);
                await addDoc(scoresCollectionRef, {
                    studentName: studentName,
                    score: finalScore,
                    totalQuestions: quizData.length,
                    timestamp: serverTimestamp(),
                    userId: userId
                });
                console.log("Score saved successfully!");
            } catch (error) {
                console.error("Error saving score:", error);
            }
        }

        function restartQuiz() {
            currentQuestionIndex = 0;
            score = 0;
            incorrectAnswers = [];
            scoreEl.textContent = '0';
            studentNameInput.value = ''; // Clear student name
            resultsScreen.classList.add('hidden');
            startScreen.classList.remove('hidden');
        }

        window.addEventListener('load', initializeFirebase);
    </script>
</body>
</html>
