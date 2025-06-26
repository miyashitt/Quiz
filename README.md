<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>縣陵クイズ完全版</title>
  <style>
    body {
      font-family: sans-serif;
      background: #f8f8f8;
      text-align: center;
      padding: 20px;
    }
    .question-box {
      font-size: 1.5em;
      min-height: 100px;
      margin: 20px auto;
      width: 80%;
      white-space: pre-wrap;
    }
    .choices {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 10px;
    }
    .choice {
      padding: 10px 20px;
      background: #ddd;
      border-radius: 10px;
      cursor: pointer;
      font-size: 1.3em;
      transition: background 0.3s ease;
      user-select: none;
    }
    .choice:hover {
      background: #bbb;
    }
    #startBtn,
    #nextBtn,
    #buzzBtn {
      padding: 10px 20px;
      font-size: 1.2em;
      margin-top: 10px;
      cursor: pointer;
    }
    #timer,
    #answerBox,
    #scoreBox,
    #bestScoreBox,
    #feedbackBox {
      margin-top: 20px;
      font-size: 1.2em;
    }
    #feedbackBox {
      transition: opacity 0.6s ease;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <h1>縣陵百科クイズ</h1>
  <audio id="bgm" src="https://cdn.pixabay.com/download/audio/2023/03/14/audio_ef16bc184b.mp3?filename=fun-pop-music-14550.mp3" loop autoplay></audio>
  <audio id="correctSound" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_e4c2590b01.mp3?filename=correct-answer-2-109766.mp3"></audio>
  <audio id="wrongSound" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_f1fc1bb3f7.mp3?filename=wrong-answer-3-204254.mp3"></audio>

  <div class="question-box" id="question"></div>
  <div id="timer">制限時間: <span id="time">15</span>秒</div>
  <div id="answerBox"></div>
  <button id="buzzBtn" style="display:none;">早押し！</button>
  <div class="choices" id="choices"></div>
  <div id="feedbackBox"></div>
  <button id="nextBtn" style="display:none;">次の問題</button>
  <div id="scoreBox"></div>
  <div id="bestScoreBox"></div>
  <button id="startBtn">ゲームスタート</button>

<script>
  const fullData = [
    { q:"伝説の文化祭OPは？○○○○サマー", a:"アーリー", comment:"幾度となく塗り替えようとされてきたがいまだにこれを超えるクオリティーの曲は発表されていない。音源は生徒会が管理しており、一般生徒が触れることはできない" },
    { q:"専門は家族社会学、ジェンダー論、女性学である、日本のフェミニスト・社会学者は？", a:"うえのちづこ", comment:"" },
    { q:"高校生向け化学の動画を投稿し大学入試センターと戦うチャンネルは？\nOnline Chemistry by ○○○○○", a:"ヒガシマキ", comment:"https://youtu.be/ZvE1JMkcj3A?feature=shared" },
    { q:"縣陵生になると体育の時間に覚えさせられるものは?", a:"けんりょうたいそう", comment:"「準備体操とは...」" },
    { q:"質実剛健であれ　大道を闊歩せよ　あとひとつは？", a:"よわねをはくな", comment:"..." },
    { q:"地球の会←なんて読む？", a:"そらのかい", comment:"..." },
    { q:"縣陵応援団の言うPTAのAとは?", a:"アルコール", comment:"..." },
    { q:"小体育館の下に存在している場所は？", a:"ピロティ", comment:"..." },
    { q:"第76th縣陵祭テーマソングは？", a:"ひゃっぽ", comment:"..." },
    { q:"焼肉きんぐあがた店はかつてなんだった？", a:"おこほん", comment:"..." },
    { q:"2学年が探究成果を発表する大会とは？", a:"KRGP", comment:"..." },
    { q:"お昼に流れる校内放送の名称は？", a:"けんりょうオンエア", comment:"..." },
    { q:"県ケ丘高校の文化祭の名称は？", a:"けんりょうさい", comment:"..." },
    { q:"長野県松本市に本部を置く国立大学は？", a:"しんしゅうだいがく", comment:"..." },
    { q:"中学校や高等学校において、生徒が主体的に学校生活の改善や向上を目指すための組織は？", a:"せいとかい", comment:"..." },
    { q:"松本市の中心部を走る城下町まつもとの有名観光スポットを巡る周遊バスは？", a:"タウンスニーカー", comment:"..." },
    { q:"探究活動を行うことを重視した松本県ヶ丘高校の学科は？", a:"たんきゅうか", comment:"..." },
    { q:"学校で行われる定期的な試験は？", a:"ていきこうさ", comment:"..." },
    { q:"書籍や記録などを保存・提供する施設は？", a:"としょかん", comment:"..." },
    { q:"応援練習における、発声練習で発する語は？", a:"け", comment:"..." },
    { q:"学校教育法で定められた春季休暇は？", a:"はるやすみ", comment:"..." },
    { q:"数研出版の青チャートと並ぶ参考書（啓林館）は？", a:"フォーカスゴールド", comment:"..." },
    { q:"応援歌で唯一カタカナ三文字の歌の名前は？", a:"ラララ", comment:"..." },
    { q:"大学受験で現役合格できなかった人は？", a:"ろうにん", comment:"..." }
  ];

  const HIRAGANA = [..."あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわをん"];
  const KATAKANA = [..."アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲンー"];
  const ALPHABET = [..."ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"];

  let quizData = [], current = 0, score = 0, bestScore = 0, currentAnswer = "", answerProgress = "", revealTimer = null, startTime = 0;
  let timer = null, isTimeUp = false;

  const qEl = document.getElementById('question');
  const choicesEl = document.getElementById('choices');
  const timerEl = document.getElementById('time');
  const feedbackBox = document.getElementById('feedbackBox');
  const scoreBox = document.getElementById('scoreBox');
  const bestBox = document.getElementById('bestScoreBox');
  const answerBox = document.getElementById('answerBox');
  const buzzBtn = document.getElementById('buzzBtn');
  const nextBtn = document.getElementById('nextBtn');
  const startBtn = document.getElementById('startBtn');
  const correctSound = document.getElementById('correctSound');
  const wrongSound = document.getElementById('wrongSound');

  startBtn.onclick = () => {
    startBtn.style.display = 'none';
    score = 0;
    current = 0;
    quizData = [...fullData].sort(() => Math.random() - 0.5).slice(0, 24);
    nextBtn.style.display = 'none';
    isTimeUp = false;
    showQuestion();
  };

  function showQuestion() {
    qEl.innerText = '';
    choicesEl.innerHTML = '';
    feedbackBox.innerText = '';
    feedbackBox.style.opacity = 0;
    answerBox.innerText = '';
    buzzBtn.style.display = 'inline';
    nextBtn.style.display = 'none';
    currentAnswer = quizData[current].a;
    answerProgress = '';
    isTimeUp = false;
    let q = quizData[current];
    let i = 0;
    startTime = Date.now();
    revealTimer = setInterval(() => {
      if (i < q.q.length) {
        qEl.innerText += q.q[i++];
      } else {
        clearInterval(revealTimer);
      }
    }, 150);
  }

  buzzBtn.onclick = () => {
    clearInterval(revealTimer);
    buzzBtn.style.display = 'none';
    startTimer();
    showNextChar();
  };

  function showNextChar() {
    if (isTimeUp) return; // 時間切れ後は表示停止
    let index = answerProgress.length;
    if (index >= currentAnswer.length) {
      stopTimer();
      let elapsed = (Date.now() - startTime) / 1000;
      let displayLength = qEl.innerText.length;
      let fullLength = quizData[current].q.length;
      let bonus = displayLength <= fullLength / 3 ? 50 : displayLength <= (fullLength * 2 / 3) ? 25 : 10;
      score += 50 + bonus;
      correctSound.play();
      feedbackBox.innerText = `正解！\n【答え】${quizData[current].a}\n${quizData[current].comment || ''}`;
      feedbackBox.style.opacity = 1;
      nextBtn.style.display = 'inline';
      return;
    }
    const pool = getCharPool(currentAnswer[index]);
    let choices = [currentAnswer[index]];
    while (choices.length < 6) {
      let r = pool[Math.floor(Math.random() * pool.length)];
      if (!choices.includes(r)) choices.push(r);
    }
    choices = choices.sort(() => Math.random() - 0.5);
    choicesEl.innerHTML = '';
    answerBox.innerText = answerProgress;
    choices.forEach(c => {
      let div = document.createElement('div');
      div.className = 'choice';
      div.innerText = c;
      div.onclick = () => {
        if (isTimeUp) return; // クリック無効化
        if (c === currentAnswer[index]) {
          answerProgress += c;
          showNextChar();
        } else {
          stopTimer();
          wrongSound.play();
          feedbackBox.innerText = `不正解…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
          feedbackBox.style.opacity = 1;
          score -= 20;
          nextBtn.style.display = 'inline';
          // 選択肢操作不可に
          choicesEl.querySelectorAll('.choice').forEach(el => el.onclick = null);
        }
      };
      choicesEl.appendChild(div);
    });
  }

  function getCharPool(char) {
    if (HIRAGANA.includes(char)) return HIRAGANA;
    if (KATAKANA.includes(char)) return KATAKANA;
    return ALPHABET;
  }

  function startTimer() {
    let time = 15;
    timerEl.innerText = time;
    isTimeUp = false;
    timer = setInterval(() => {
      time--;
      timerEl.innerText = time;
      if (time <= 0) {
        stopTimer();
        isTimeUp = true;
        wrongSound.play();
        feedbackBox.innerText = `時間切れ…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
        feedbackBox.style.opacity = 1;
        nextBtn.style.display = 'inline';
        answerBox.innerText = currentAnswer;
        choicesEl.innerHTML = ''; // 選択肢非表示
      }
    }, 1000);
  }

  function stopTimer() {
    clearInterval(timer);
  }

  nextBtn.onclick = () => {
    current++;
    if (current < quizData.length) {
      showQuestion();
    } else {
      showScore();
    }
  };

  function showScore() {
    qEl.innerText = '';
    choicesEl.innerHTML = '';
    timerEl.innerText = '0';
    answerBox.innerText = '';
    feedbackBox.innerText = '';
    nextBtn.style.display = 'none';
    scoreBox.innerHTML = `今回のスコア：${score} / 500`;
    if (score > bestScore) bestScore = score;
    bestBox.innerHTML = `ベストスコア：${bestScore} / 500`;
    startBtn.style.display = 'inline';
  }
</script>
</body>
</html>
