<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>縣陵クイズゲーム</title>
  <style>
    body { font-family: 'Arial', sans-serif; background: #f5f5f5; text-align: center; padding: 20px; }
    .question-box { font-size: 1.5em; margin: 20px; min-height: 100px; }
    .choices { display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin: 20px; }
    .choice { padding: 10px 20px; background: #eee; border-radius: 10px; cursor: pointer; font-size: 1.2em; transition: background 0.3s ease; }
    .choice:hover { background: #ccc; }
    #startBtn, #nextBtn, #buzzBtn { padding: 10px 20px; font-size: 1.2em; cursor: pointer; }
    #scoreBox { font-size: 1.5em; margin-top: 30px; }
    #timer { font-size: 1.2em; margin-top: 10px; }
    #answerBox { font-size: 1.5em; margin: 10px; color: #333; }
    #feedbackBox { font-size: 1.2em; margin: 10px auto; max-width: 80%; color: #444; line-height: 1.6; opacity: 0; transition: opacity 0.8s ease; }
  </style>
</head>
<body>
  <h1>縣陵百科クイズ</h1>
  <audio id="bgm" loop autoplay volume="0.3">
    <source src="https://cdn.pixabay.com/download/audio/2023/03/14/audio_ef16bc184b.mp3?filename=fun-pop-music-14550.mp3" type="audio/mp3">
  </audio>
  <div id="quiz">
    <div class="question-box" id="question"></div>
    <div id="timer">制限時間: <span id="time">15</span>秒</div>
    <div id="answerBox"></div>
    <button id="buzzBtn" style="display:none;">押す！</button>
    <div class="choices" id="choices"></div>
    <div id="feedbackBox"></div>
    <button id="nextBtn" style="display:none;">次の問題</button>
  </div>
  <div id="scoreBox"></div>
  <div id="bestScoreBox"></div>
  <button id="startBtn">ゲームスタート</button>

  <script>
    const fullData = [...(略)...]; // 省略（省略前と同じデータ）

    let quizData = [], current = 0, score = 0, timer, startTime, currentAnswer = '', answerProgress = '', bestScore = 0;
    const HIRAGANA = [...'あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわをん'];
    const KATAKANA = [...'アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲンー'];

    const qEl = document.getElementById('question');
    const choicesEl = document.getElementById('choices');
    const nextBtn = document.getElementById('nextBtn');
    const scoreBox = document.getElementById('scoreBox');
    const bestBox = document.getElementById('bestScoreBox');
    const timeEl = document.getElementById('time');
    const startBtn = document.getElementById('startBtn');
    const buzzBtn = document.getElementById('buzzBtn');
    const answerBox = document.getElementById('answerBox');
    const feedbackBox = document.getElementById('feedbackBox');

    startBtn.onclick = () => {
      startBtn.style.display = 'none';
      nextBtn.style.display = 'none';
      buzzBtn.style.display = 'none';
      scoreBox.innerHTML = '';
      feedbackBox.style.opacity = 0;
      current = 0;
      score = 0;
      quizData = [...fullData].sort(() => Math.random() - 0.5).slice(0, 5);
      document.getElementById('bgm').play();
      showQuestion();
    }

    nextBtn.onclick = () => {
      nextBtn.style.display = 'none';
      feedbackBox.style.opacity = 0;
      current++;
      if (current < quizData.length) {
        showQuestion();
      } else {
        showScore();
      }
    }

    function showQuestion() {
      let q = quizData[current];
      qEl.innerHTML = '';
      answerBox.innerText = '';
      feedbackBox.innerText = '';
      feedbackBox.style.opacity = 0;
      answerProgress = '';
      currentAnswer = q.a;
      let i = 0;
      buzzBtn.style.display = 'inline';
      startTime = Date.now();

      let reveal = setInterval(() => {
        if (i < q.q.length) {
          qEl.innerHTML += q.q[i++];
        } else {
          clearInterval(reveal);
        }
      }, 100);

      buzzBtn.onclick = () => {
        buzzBtn.style.display = 'none';
        startTimer();
        showNextChar();
      }
    }

    function showNextChar() {
      const q = quizData[current];
      const index = answerProgress.length;
      if (index >= currentAnswer.length) {
        stopTimer();
        let elapsed = (Date.now() - startTime) / 1000;
        let scoreAdd = Math.max(0, Math.round((15 - elapsed) * 10));
        score += scoreAdd;
        feedbackBox.innerText = `正解！\n【答え】${q.a}\n${q.comment || ''}`;
        feedbackBox.style.opacity = 1;
        nextBtn.style.display = 'inline';
        return;
      }
      const pool = (q.type === 'katakana' ? KATAKANA : HIRAGANA);
      let choices = [currentAnswer[index]];
      while (choices.length < 6) {
        let rand = pool[Math.floor(Math.random() * pool.length)];
        if (!choices.includes(rand)) choices.push(rand);
      }
      choices.sort(() => Math.random() - 0.5);
      choicesEl.innerHTML = '';
      answerBox.innerText = answerProgress;
      choices.forEach(char => {
        let btn = document.createElement('div');
        btn.className = 'choice';
        btn.innerText = char;
        btn.onclick = () => {
          if (char === currentAnswer[index]) {
            answerProgress += char;
            showNextChar();
          } else {
            stopTimer();
            feedbackBox.innerText = `不正解…\n【正解】${q.a}\n${q.comment || ''}`;
            feedbackBox.style.opacity = 1;
            nextBtn.style.display = 'inline';
            score -= 50; // お手付きで減点
          }
        };
        choicesEl.appendChild(btn);
      });
    }

    function startTimer() {
      let time = 15;
      timeEl.innerText = time;
      timer = setInterval(() => {
        time--;
        timeEl.innerText = time;
        if (time <= 0) {
          clearInterval(timer);
          feedbackBox.innerText = `時間切れ…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
          feedbackBox.style.opacity = 1;
          nextBtn.style.display = 'inline';
        }
      }, 1000);
    }

    function stopTimer() {
      clearInterval(timer);
    }

    function showScore() {
      qEl.innerHTML = '';
      choicesEl.innerHTML = '';
      timeEl.innerText = '0';
      answerBox.innerHTML = '';
      feedbackBox.innerText = '';
      if (score > bestScore) bestScore = score;

      let rank = score >= 450 ? '神ランク ✨' : score >= 350 ? 'マスターランク 💪' : score >= 250 ? 'ノーマルランク 👍' : 'ビギナー 🔰';

      scoreBox.innerHTML = `今回のスコア：${score} / 750<br>ランク：${rank}`;
      bestBox.innerHTML = `ベストスコア：${bestScore} / 750`;
      startBtn.style.display = 'inline';
    }
  </script>
</body>
</html>
