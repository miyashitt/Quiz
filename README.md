<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>縣陵百科クイズ</title>
  <style>
    body { font-family: 'Arial', sans-serif; background: #f5f5f5; text-align: center; padding: 20px; }
    .question-box { font-size: 1.5em; margin: 20px; min-height: 100px; white-space: pre-line; }
    .choices { display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin: 20px; }
    .choice { padding: 10px 20px; background: #eee; border-radius: 10px; cursor: pointer; font-size: 1.2em; transition: background 0.3s ease; user-select:none; }
    .choice:hover { background: #ccc; }
    #startBtn, #nextBtn, #buzzBtn { padding: 10px 20px; font-size: 1.2em; cursor: pointer; }
    #scoreBox { font-size: 1.5em; margin-top: 30px; }
    #bestScoreBox { font-size: 1.2em; margin-top: 10px; color:#555; }
    #timer { font-size: 1.2em; margin-top: 10px; }
    #answerBox { font-size: 1.5em; margin: 10px; color: #333; min-height: 40px; }
    #feedbackBox { font-size: 1.2em; margin: 10px auto; max-width: 80%; color: #444; line-height: 1.6; opacity: 0; transition: opacity 0.8s ease; white-space: pre-line; }
  </style>
</head>
<body>
  <h1>縣陵百科クイズ</h1>
  <audio id="bgm" loop autoplay volume="0.3" preload="auto">
    <source src="https://cdn.pixabay.com/download/audio/2023/03/14/audio_ef16bc184b.mp3?filename=fun-pop-music-14550.mp3" type="audio/mp3" />
  </audio>
  <div id="quiz">
    <div class="question-box" id="question"></div>
    <div id="timer">制限時間: <span id="time">15</span>秒</div>
    <div id="answerBox"></div>
    <button id="buzzBtn" style="display:none;">早押し！</button>
    <div class="choices" id="choices"></div>
    <div id="feedbackBox"></div>
    <button id="nextBtn" style="display:none;">次の問題</button>
  </div>
  <div id="scoreBox"></div>
  <div id="bestScoreBox"></div>
  <button id="startBtn">ゲームスタート</button>

  <!-- 効果音 -->
  <audio id="correctSound" preload="auto">
    <source src="https://cdn.pixabay.com/download/audio/2021/09/17/audio_bfa04e31de.mp3?filename=correct-2-46134.mp3" type="audio/mp3" />
  </audio>
  <audio id="wrongSound" preload="auto">
    <source src="https://cdn.pixabay.com/download/audio/2021/09/17/audio_c553d19755.mp3?filename=wrong-2-46049.mp3" type="audio/mp3" />
  </audio>

<script>
  // 省略せず fullData を入れてください（長いためここでは省略）

  const HIRAGANA = [...'あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわをん'];
  const KATAKANA = [...'アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲンー'];
  const ALPHABET = [...'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz'];

  let quizData = [], current = 0, score = 0, timer = null;
  let revealCount = 0;
  let currentAnswer = '', answerProgress = '';
  let bestScore = Number(localStorage.getItem('kenryoBestScore') || 0);
  let questionRevealingInterval = null;

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
  const bgm = document.getElementById('bgm');
  const correctSound = document.getElementById('correctSound');
  const wrongSound = document.getElementById('wrongSound');

  startBtn.onclick = () => {
    startBtn.style.display = 'none';
    nextBtn.style.display = 'none';
    buzzBtn.style.display = 'inline-block';
    scoreBox.innerHTML = '';
    feedbackBox.style.opacity = 0;
    current = 0;
    score = 0;
    bestBox.textContent = `ベストスコア: ${bestScore}`;
    quizData = [...fullData].sort(() => Math.random() - 0.5).slice(0, 5);
    bgm.volume = 0.3;
    bgm.play();
    showQuestion();
  };

  buzzBtn.onclick = () => {
    if (questionRevealingInterval) {
      clearInterval(questionRevealingInterval);
      questionRevealingInterval = null;
    }
    buzzBtn.style.display = 'none';
    startTimer();
    answerProgress = '';
    showNextChar();
  };

  nextBtn.onclick = () => {
    nextBtn.style.display = 'none';
    feedbackBox.style.opacity = 0;
    current++;
    if (current < quizData.length) {
      showQuestion();
    } else {
      showScore();
    }
  };

  function showQuestion() {
    clearInterval(timer);
    if (questionRevealingInterval) clearInterval(questionRevealingInterval);

    qEl.textContent = '';
    answerBox.textContent = '';
    feedbackBox.textContent = '';
    feedbackBox.style.opacity = 0;
    choicesEl.innerHTML = '';
    timeEl.textContent = '15';
    answerProgress = '';
    revealCount = 0;

    const q = quizData[current];
    currentAnswer = q.a;

    let i = 0;
    questionRevealingInterval = setInterval(() => {
      if (i < q.q.length) {
        qEl.textContent += q.q[i++];
        revealCount = i;
      } else {
        clearInterval(questionRevealingInterval);
        questionRevealingInterval = null;
      }
    }, 200);

    buzzBtn.style.display = 'inline-block';
    nextBtn.style.display = 'none';
    choicesEl.innerHTML = '';
  }

  function showNextChar() {
    const q = quizData[current];
    const index = answerProgress.length;

    let answerType = 'hiragana';
    if (/^[ァ-ヶー　]+$/.test(currentAnswer)) answerType = 'katakana';
    else if (/^[A-Za-z]+$/.test(currentAnswer)) answerType = 'alphabet';

    if (index >= currentAnswer.length) {
      stopTimer();
      correctSound.play();

      const totalLen = quizData[current].q.length;
      const pressedLen = revealCount;
      let basePoint = 50;
      let bonusPoint = 0;
      if (pressedLen <= totalLen / 3) bonusPoint = 50;
      else if (pressedLen <= totalLen * 2 / 3) bonusPoint = 25;
      else bonusPoint = 10;

      score += basePoint + bonusPoint;

      feedbackBox.textContent = `正解！\n【答え】${q.a}\n${q.comment || ''}\n(基本点 50点 + ボーナス ${bonusPoint}点)`;
      feedbackBox.style.opacity = 1;
      nextBtn.style.display = 'inline-block';
      answerBox.textContent = currentAnswer;
      choicesEl.innerHTML = '';
      return;
    }

    const pool = answerType === 'katakana' ? KATAKANA : answerType === 'alphabet' ? ALPHABET : HIRAGANA;
    let choices = [currentAnswer[index]];
    while (choices.length < 6) {
      let rand = pool[Math.floor(Math.random() * pool.length)];
      if (!choices.includes(rand)) choices.push(rand);
    }
    choices.sort(() => Math.random() - 0.5);

    choicesEl.innerHTML = '';
    answerBox.textContent = answerProgress;

    choices.forEach(char => {
      const btn = document.createElement('div');
      btn.className = 'choice';
      btn.textContent = char;
      btn.onclick = () => {
        if (char === currentAnswer[index]) {
          answerProgress += char;
          showNextChar();
        } else {
          stopTimer();
          wrongSound.play();
          feedbackBox.textContent = `不正解…\n【正解】${q.a}\n${q.comment || ''}`;
          feedbackBox.style.opacity = 1;
          nextBtn.style.display = 'inline-block';
          answerBox.textContent = currentAnswer;
          choicesEl.innerHTML = '';
        }
      };
      choicesEl.appendChild(btn);
    });
  }

  function startTimer() {
    let time = 15;
    timeEl.textContent = time;
    timer = setInterval(() => {
      time--;
      timeEl.textContent = time;
      if (time <= 0) {
        clearInterval(timer);
        wrongSound.play();
        feedbackBox.textContent = `時間切れ…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
        feedbackBox.style.opacity = 1;
        nextBtn.style.display = 'inline-block';
        choicesEl.innerHTML = '';
        answerBox.textContent = quizData[current].a;
      }
    }, 1000);
  }

  function stopTimer() {
    clearInterval(timer);
  }

  function showScore() {
    qEl.textContent = '';
    choicesEl.innerHTML = '';
    timeEl.textContent = '0';
    answerBox.textContent = '';
    feedbackBox.textContent = '';
    feedbackBox.style.opacity = 0;

    if (score > bestScore) {
      bestScore = score;
      localStorage.setItem('kenryoBestScore', bestScore);
    }

    let rank;
    if (score >= 450) rank = '神ランク ✨';
    else if (score >= 350) rank = 'マスターランク 💪';
    else if (score >= 250) rank = 'ノーマルランク 👍';
    else rank = 'ビギナー 🔰';

    scoreBox.innerHTML = `今回のスコア：${score} / 500<br>ランク：${rank}`;
    bestBox.textContent = `ベストスコア：${bestScore} / 500`;
    startBtn.style.display = 'inline-block';
    buzzBtn.style.display = 'none';
    nextBtn.style.display = 'none';
  }
</script>
</body>
</html>
