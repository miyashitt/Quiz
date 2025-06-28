<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8" />
    <title>文化祭まとめサイト</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="icon" href="favicon.ico" type="image/x-icon">
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@300;400;500;700&display=swap" rel="stylesheet">
    <style>
        /* CSS Variables for a cohesive design */
        :root {
            --primary-color: #4CAF50; /* Greenish */
            --primary-hover-color: #45a049;
            --secondary-color: #007bff; /* Blueish */
            --secondary-hover-color: #0056b3;
            --text-dark: #333;
            --text-light: #f8f8f8;
            --background-gradient-start: #e0f7fa;
            --background-gradient-end: #c8e6c9;
            --shadow-light: rgba(0, 0, 0, 0.1);
            --shadow-medium: rgba(0, 0, 0, 0.2);
            --accent-color: #ff9800; /* Orange for highlights */
        }

        body {
            margin: 0;
            font-family: 'Noto Sans JP', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, var(--background-gradient-start) 0%, var(--background-gradient-end) 100%);
            color: var(--text-dark);
            text-align: center;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            user-select: none;
            overflow: hidden; /* Prevent all scrolling for full-screen effect */
            transition: opacity 0.5s ease-in-out;
            opacity: 0; /* JavaScriptでloadedクラスが付与されたら表示 */
        }

        body.loaded {
            opacity: 1;
        }

        body.virus-active {
            background-color: #000 !important;
            margin: 0 !important;
            padding: 0 !important;
            overflow: hidden !important;
            opacity: 1 !important; /* Force display during virus screen */
        }

        .hidden {
            display: none !important;
        }

        /* Initial Screen styles */
        #initial-screen {
            background: linear-gradient(135deg, var(--background-gradient-start) 0%, var(--background-gradient-end) 100%);
            color: var(--text-dark);
            height: 100vh;
            width: 100vw;
            position: fixed;
            top: 0;
            left: 0;
            z-index: 9997;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 1.5rem;
            box-sizing: border-box;
            animation: fadeInScale 0.8s ease-out forwards;
        }

        #initial-screen h1 {
            font-size: 2.8rem;
            color: var(--primary-color);
            margin-bottom: 2.5rem;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
            line-height: 1.3;
        }

        #initial-screen p {
            font-size: 1.2rem;
            margin-bottom: 3rem;
            max-width: 600px;
            line-height: 1.6;
        }

        /* Virus Screen styles */
        #virus-screen {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            width: 100vw;
            height: 100vh;
            background-color: #000;
            color: #ff3300;
            z-index: 999999;
            font-family: 'Courier New', Courier, monospace;
            font-size: 1.2rem;
            overflow: hidden;
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            box-sizing: border-box;
            pointer-events: none;
            animation: noise 0.1s infinite, flicker 0.2s infinite alternate;
            text-shadow: 0 0 5px #ff0000, 0 0 15px #ff0000;
        }

        #virus-screen::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(255, 0, 0, 0.2);
            background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAQAAAAECAYAAACp8da+AAAAAXNSR0IArs4c6QAAABJJREFUGFdjYGBgYGBgYGAAAgABAAyW/71mAAAAAElFTkSuQmCC');
            background-size: 2px 2px;
            opacity: 0.7;
            pointer-events: none;
            z-index: -1;
        }

        #virus-screen h2 {
            font-size: 2.8rem;
            margin-bottom: 2rem;
            animation: blink-fast 0.3s infinite alternate, glitch 0.5s infinite alternate;
            width: 90%;
            text-align: center;
            box-sizing: border-box;
            text-shadow: 0 0 15px #f00, 0 0 30px #f00; /* Stronger shadow */
        }

        #countdown {
            font-size: 7rem; /* Larger countdown */
            font-weight: bold;
            color: #00ff00;
            text-shadow: 0 0 8px #00ff00, 0 0 25px #00ff00; /* Stronger shadow */
            margin-bottom: 1.5rem;
            width: 90%;
            text-align: center;
            box-sizing: border-box;
            animation: glitch 0.3s infinite alternate, scanlines 0.1s infinite;
        }

        #reveal-message {
            font-size: 1.5rem; /* Larger reveal message */
            color: #f0f0f0;
            background-color: rgba(0, 0, 0, 0.85); /* Slightly darker background */
            padding: 1.5rem;
            border-radius: 8px; /* More rounded */
            display: none;
            margin-top: 2rem;
            animation: fade-in 0.8s forwards;
            max-width: 85%;
            box-sizing: border-box;
            text-align: center;
            line-height: 1.6;
            border: 1px solid rgba(255, 255, 255, 0.2);
        }

        @keyframes fade-in {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        @keyframes blink-fast {
            0% { opacity: 1; }
            49% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }

        @keyframes glitch {
            0% { transform: translate(0); }
            20% { transform: translate(2px, -2px); }
            40% { transform: translate(-1px, 1px); }
            60% { transform: translate(-2px, -1px); }
            80% { transform: translate(1px, 2px); }
            100% { transform: translate(0); }
        }

        @keyframes noise {
            0% { opacity: 0.05; }
            10% { opacity: 0.1; }
            20% { opacity: 0.08; }
            30% { opacity: 0.15; }
            40% { opacity: 0.03; }
            50% { opacity: 0.12; }
            60% { opacity: 0.07; }
            70% { opacity: 0.18; }
            80% { opacity: 0.06; }
            90% { opacity: 0.11; }
            100% { opacity: 0.04; }
        }

        @keyframes flicker {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.8; }
        }

        @keyframes scanlines {
            0% { background-position: 0 0; }
            100% { background-position: 0 20px; }
        }

        /* Relief Screen Styles */
        #relief-screen {
            background: linear-gradient(45deg, #e0f7fa, #b2ebf2); /* Brighter gradient */
            color: #006064;
            height: 100vh;
            width: 100vw;
            position: fixed;
            top: 0;
            left: 0;
            z-index: 9998;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            font-size: 2rem; /* Larger font */
            font-weight: 500;
            display: none;
            user-select: text;
            animation: fade-in 0.8s ease-in-out;
            text-shadow: 1px 1px 3px rgba(0,0,0,0.1); /* Subtle text shadow */
        }

        #relief-screen p {
            margin: 0.8rem 0; /* More spacing */
        }

        /* Main Content Styles */
        #main {
            padding: 2.5rem;
            background: transparent;
            min-height: 100vh;
            width: 100vw;
            position: fixed;
            top: 0;
            left: 0;
            color: var(--text-dark);
            user-select: none;
            display: none; /* Hidden by default */
            border-radius: 16px;
            box-shadow: none;
            margin-top: 0;
            box-sizing: border-box;
            animation: fadeInScale 0.8s ease-out forwards;
            justify-content: center;
            align-items: center;
            flex-direction: column;
        }

        #main.visible {
            display: flex;
        }

        /* Navigation Button Styles */
        .nav-buttons {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            gap: 1.5rem;
            width: 100%;
            max-width: 400px;
        }

        .nav-buttons button {
            width: 90%;
            max-width: 300px;
            background-color: var(--secondary-color);
            border: none;
            padding: 1.4rem 2rem;
            font-size: 1.6rem;
            font-weight: 600;
            border-radius: 10px;
            color: var(--text-light);
            cursor: pointer;
            box-shadow: 0 6px 12px var(--shadow-medium);
            transition: background-color 0.3s ease, transform 0.2s ease, box-shadow 0.3s ease;
            letter-spacing: 0.5px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 0.8rem;
            white-space: nowrap;
            box-sizing: border-box;
        }

        .nav-buttons button:hover {
            background-color: var(--secondary-hover-color);
            transform: translateY(-3px) scale(1.02);
            box-shadow: 0 8px 16px var(--shadow-medium);
        }

        .share-icon {
            font-size: 1.6rem;
        }

        /* Responsive Design */
        @media (max-width: 768px) {
            .nav-buttons button {
                padding: 1.2rem 1.8rem;
                font-size: 1.4rem;
            }
            .share-icon {
                font-size: 1.4rem;
            }
            #virus-screen h2 {
                font-size: 2.2rem;
            }
            #countdown {
                font-size: 5rem;
            }
            #reveal-message {
                font-size: 1.2rem;
                padding: 1.2rem;
            }
            #initial-screen h1 {
                font-size: 2.2rem;
                margin-bottom: 2rem;
            }
            #initial-screen p {
                font-size: 1.1rem;
                margin-bottom: 2rem;
            }
        }

        @media (max-width: 480px) {
            .nav-buttons {
                gap: 1rem;
            }
            .nav-buttons button {
                width: 95%;
                padding: 1rem 1.5rem;
                font-size: 1.3rem;
                max-width: none;
            }
            .share-icon {
                font-size: 1.3rem;
            }
            #virus-screen h2 {
                font-size: 1.8rem;
                margin-bottom: 0.8rem;
            }
            #countdown {
                font-size: 4rem;
            }
            #reveal-message {
                font-size: 1rem;
                padding: 0.8rem;
            }
            #initial-screen h1 {
                font-size: 1.8rem;
                margin-bottom: 1.5rem;
            }
            #initial-screen p {
                font-size: 1.0rem;
                margin-bottom: 1.5rem;
            }
        }
    </style>
</head>
<body>
    <div id="virus-screen" role="alert" aria-live="assertive" aria-atomic="true" class="hidden">
        </div>

    <div id="relief-screen" class="hidden" role="region" aria-live="polite" aria-atomic="true">
        <p>冗談だよ😊</p>
        <p>文化祭を楽しんで</p>
    </div>

    <div id="initial-screen" role="region" aria-label="文化祭特設サイトへようこそ">
        <h1>ようこそ！文化祭特設サイトへ</h1>
        <p>文化祭の情報を公開しています。</p>
        <div class="nav-buttons">
            <button id="trigger-virus-btn" type="button">
                <span class="share-icon"></span> 文化祭情報を見る
            </button>
        </div>
    </div>

    <div id="main" role="main" aria-label="文化祭まとめコンテンツ" class="hidden">
        <div class="nav-buttons">
            <button id="repeat-virus-btn" type="button">
                <span class="share-icon"></span> もう一度ウイルス演出を見る
            </button>
            <button id="show-bunkasai-info-btn" type="button">
                <span class="share-icon"></span> 文化祭情報
            </button>
            <button id="show-quiz-minigame-btn" type="button">
                <span class="share-icon"></span> クイズ
            </button>
            <button class="share-btn share-x" role="button" tabindex="0" data-sns="x" title="Xで共有" aria-label="Xで共有">
                <span class="share-icon"></span> 𝕏で共有
            </button>
            <button class="share-btn share-line" role="button" tabindex="0" data-sns="line" title="LINEで共有" aria-label="LINEで共有">
                <span class="share-icon"></span> LINEで共有
            </button>
        </div>
    </div>

    <script defer>
        document.addEventListener("DOMContentLoaded", () => {
            // --- Constants ---
            const LOCAL_STORAGE_KEY = "bunkasai_visited";
            const VIRUS_COUNTDOWN_SECONDS = 5;
            const QUIZ_SITE_URL = "https://miyashitt.github.io/Quiz/"; // 変更されたURL
            const BUNKASAI_INSTAGRAM_URL = "https://www.instagram.com/kenryo_fes_78th?utm_source=ig_web_button_share_sheet&igsh=MWkyZDRrbjRuYnl6ag==";

            // --- DOM Elements ---
            const body = document.body;
            const initialScreen = document.getElementById("initial-screen");
            const virusScreen = document.getElementById("virus-screen");
            const reliefScreen = document.getElementById("relief-screen");
            const mainScreen = document.getElementById("main");

            const triggerVirusBtn = document.getElementById("trigger-virus-btn");
            const repeatVirusBtn = document.getElementById("repeat-virus-btn");
            const showBunkasaiInfoBtn = document.getElementById("show-bunkasai-info-btn");
            const showQuizMinigameBtn = document.getElementById("show-quiz-minigame-btn");
            const shareButtons = document.querySelectorAll('.share-btn');

            // --- Global Variables for State Management ---
            let speechSynth = window.speechSynthesis;
            let currentUtterance = null;
            let alarmAudio = null;
            let isVoiceLoopActive = false;
            let countdownTimerId = null; // Stores the setTimeout ID for the countdown
            const isLineBrowserDetected = navigator.userAgent.includes("Line");

            // --- Audio and Speech Functions ---

            /**
             * Initializes and plays an alarm sound.
             */
            function playAlarmSound() {
                if (alarmAudio) {
                    alarmAudio.pause();
                    alarmAudio.currentTime = 0;
                }
                alarmAudio = new Audio('alarm.mp3');
                alarmAudio.loop = true;
                alarmAudio.volume = 0.5;
                alarmAudio.play().catch(e => console.error("Error playing audio:", e));
            }

            /**
             * Speaks the given text using the Web Speech API.
             * @param {string} text - The text to be spoken.
             */
            function speakText(text) {
                if (speechSynth.speaking) {
                    speechSynth.cancel();
                }
                currentUtterance = new SpeechSynthesisUtterance(text);
                currentUtterance.lang = 'ja-JP';
                currentUtterance.rate = 1.0;
                currentUtterance.pitch = 1.0;
                currentUtterance.volume = 0.7;
                speechSynth.speak(currentUtterance);
            }

            /**
             * Starts a continuous voice loop for the virus message.
             */
            function startVoiceLoop() {
                isVoiceLoopActive = true;
                function loop() {
                    if (!isVoiceLoopActive) return;
                    speakText("デバイスはウイルスに感染しました");
                    currentUtterance.onend = () => {
                        if (isVoiceLoopActive) {
                            setTimeout(loop, 3000); // Repeat after 3 seconds
                        }
                    };
                }
                loop(); // Start the first loop
            }

            /**
             * Stops the voice loop and alarm sound.
             */
            function stopVoiceAndAlarm() {
                isVoiceLoopActive = false;
                if (speechSynth.speaking) {
                    speechSynth.cancel();
                }
                if (alarmAudio) {
                    alarmAudio.pause();
                    alarmAudio.currentTime = 0;
                }
            }

            // --- Screen Management Functions ---

            /**
             * Resets and initiates the virus simulation.
             */
            function startVirusSimulation() {
                // Clear any ongoing countdown or audio
                if (countdownTimerId) {
                    clearTimeout(countdownTimerId);
                }
                stopVoiceAndAlarm();

                // Set up virus screen content
                virusScreen.innerHTML = `
                    <h2><span style="color:red;">警告!!!</span> デバイスはウイルスに侵害されました。</h2>
                    <div id="countdown" style="color: #00FF00;">${VIRUS_COUNTDOWN_SECONDS}</div>
                    <div id="reveal-message" class="hidden">
                        <h3>これは文化祭の告知です！</h3>
                        <p><strong>文化祭の日程や見どころはInstagram公式アカウントをチェック！</strong></p>
                        <p>最新情報をGETして文化祭を楽しもう！</p>
                        <p>※偽のウイルス演出はこれで終了です。</p>
                        <a href="${BUNKASAI_INSTAGRAM_URL}" target="_blank" style="color: #66ccff; text-decoration: underline; font-weight: bold;">Instagram公式アカウントへ</a>
                    </div>
                `;

                // Show virus screen and hide others
                body.classList.add("virus-active");
                initialScreen.classList.add("hidden");
                mainScreen.classList.add("hidden");
                reliefScreen.classList.add("hidden");
                virusScreen.classList.remove("hidden");
                virusScreen.style.pointerEvents = 'none'; // Disable interaction during countdown

                // Play audio only if not in LINE browser
                if (!isLineBrowserDetected) {
                    speakText("デバイスはウイルスに感染しました");
                    startVoiceLoop();
                    playAlarmSound();
                }

                let countdown = VIRUS_COUNTDOWN_SECONDS;
                const countdownElement = document.getElementById("countdown");
                const revealMessage = document.getElementById("reveal-message");

                function runCountdown() {
                    if (countdownElement) { // Ensure element exists before updating
                        countdownElement.textContent = countdown;
                    }

                    if (countdown <= 0) {
                        if (countdownTimerId) {
                            clearTimeout(countdownTimerId);
                        }
                        countdownElement?.classList.add("hidden"); // Use optional chaining for safety
                        revealMessage?.classList.remove("hidden");
                        stopVoiceAndAlarm();
                        virusScreen.style.pointerEvents = 'auto'; // Enable interaction after reveal

                        setTimeout(() => {
                            virusScreen.classList.add("hidden");
                            reliefScreen.classList.remove("hidden");
                            reliefScreen.style.display = 'flex'; // Explicitly set display for relief screen fade-in

                            setTimeout(() => {
                                reliefScreen.classList.add("hidden");
                                reliefScreen.style.display = 'none'; // Hide relief screen
                                mainScreen.classList.remove("hidden");
                                mainScreen.classList.add("visible");
                                body.classList.remove("virus-active");
                                localStorage.setItem(LOCAL_STORAGE_KEY, "true"); // Set flag that virus has played
                            }, 2000); // Hide relief, show main after 2 seconds
                        }, 3000); // Show reveal message for 3 seconds
                    } else {
                        countdown--;
                        countdownTimerId = setTimeout(runCountdown, 1000); // Schedule next tick
                    }
                }

                // Start the countdown
                countdownTimerId = setTimeout(runCountdown, 1000); // Initial delay before first decrement
            }

            // --- Event Listeners ---

            // Initial fade-in for the body
            body.classList.add("loaded");

            // Determine which screen to show on initial load based on local storage
            if (localStorage.getItem(LOCAL_STORAGE_KEY) === "true") {
                initialScreen.classList.add("hidden");
                mainScreen.classList.remove("hidden");
                mainScreen.classList.add("visible");
            } else {
                initialScreen.classList.remove("hidden");
            }

            // Initial Screen: "文化祭情報を見る" button
            triggerVirusBtn.addEventListener("click", startVirusSimulation);

            // Main Screen: "もう一度ウイルス演出を見る" button
            repeatVirusBtn.addEventListener("click", () => {
                localStorage.removeItem(LOCAL_STORAGE_KEY); // Allow re-play
                startVirusSimulation();
            });

            // Main Screen: "文化祭情報" button
            showBunkasaiInfoBtn.addEventListener("click", () => {
                window.open(BUNKASAI_INSTAGRAM_URL, "_blank");
            });

            // Main Screen: "クイズ" button
            showQuizMinigameBtn.addEventListener("click", () => {
                window.open(QUIZ_SITE_URL, "_blank");
            });

            // Share buttons logic
            shareButtons.forEach(button => {
                button.addEventListener('click', () => {
                    const sns = button.dataset.sns;
                    const url = encodeURIComponent(window.location.href);
                    const text = encodeURIComponent("私たちの文化祭の特設サイトを見てね！\n#文化祭 #高校生活");
                    let shareUrl = '';

                    switch (sns) {
                        case 'x':
                            shareUrl = `https://twitter.com/intent/tweet?url=${url}&text=${text}`;
                            break;
                        case 'line':
                            shareUrl = `https://social-plugins.line.me/lineit/share?url=${url}&text=${text}`;
                            break;
                    }

                    if (shareUrl) {
                        window.open(shareUrl, '_blank');
                    }
                });
            });
        });
    </script>
</body>
</html>
