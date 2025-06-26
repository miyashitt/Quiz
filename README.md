<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>縣陵クイズ完全版</title>
  <style>
    body { font-family: sans-serif; background: #f8f8f8; text-align: center; padding: 20px; }
    .question-box { font-size: 1.5em; min-height: 100px; margin: 20px auto; width: 80%; }
    .choices { display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; }
    .choice { padding: 10px 20px; background: #ddd; border-radius: 10px; cursor: pointer; font-size: 1.3em; transition: background 0.3s ease; }
    .choice:hover { background: #bbb; }
    #startBtn, #nextBtn, #buzzBtn { padding: 10px 20px; font-size: 1.2em; margin-top: 10px; cursor: pointer; }
    #timer, #answerBox, #scoreBox, #bestScoreBox, #feedbackBox { margin-top: 20px; font-size: 1.2em; }
    #feedbackBox { transition: opacity 0.6s ease; white-space: pre-wrap; }
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
      {q:"伝説の文化祭OPは？○○○○サマー", a:"アーリー", comment:"幾度となく塗り替えようとされてきたがいまだにこれを超えるクオリティーの曲は発表されていない。音源は生徒会が管理しており、一般生徒が触れることはできない"},
      {q:"専門は家族社会学、ジェンダー論、女性学である、日本のフェミニスト・社会学者は？", a:"うえのちづこ", comment:""},
      {q:"高校生向け化学の動画を投稿し大学入試センターと戦うチャンネルは？\nOnline Chemistry by ○○○○○", a:"ヒガシマキ", comment:"https://youtu.be/ZvE1JMkcj3A?feature=shared"},
      {q:"縣陵生になると体育の時間に覚えさせられるものは?", a:"けんりょうたいそう", comment:"「準備体操とは元々軍隊などで訓練のために行われていたものである。」という真偽不明の由来故になかなかハードで準備体操にしては長めな運動を体育の前にやらされる。先生によっては少し喋っただけで最初からやり直しとなる可能性もあり、注意が必要である。なお2年生以降ではその存在は突然無くなり覚えている人間は、強力な洗脳に耐えた1部の者だけであり秘密裏にその存在は語り継がれている。「やる意味が無い」というような発言をした者は1人残らず消されている。"},
      {q:"質実剛健であれ　大道を闊歩せよ　あとひとつは？", a:"よわねをはくな", comment:"3つから成る我が校に古くから伝わる三大精神である。ほとんどの縣陵生は弱音を吐くなしか知らない。お昼の放送の曲で軽くあしらわれているが、実は在学3年間にこの精神の下、高校生活を遂行したものは殿堂入りを果たすことができる。しかし未だ達成したものはいない。"},
      {q:"地球の会←なんて読む？", a:"そらのかい", comment:"難読漢字の一種。ただの初見殺し。部活は月1"},
      {q:"縣陵応援団の言うPTAのAとは?", a:"アルコール", comment:"パチンコ、タバコ、アルコールの略であり縣陵生の誰もが知っている。応援団の「いいかお前らPTAには手を出すなよ」というフレーズは去年流行語大賞に選ばれた。"},
      {q:"小体育館の下に存在している場所は？", a:"ピロティ", comment:"最初に言われたときはどこのことか全くわからない。特に一年生の物販委販売の時に迷子が目立つ。その利用方法は多岐にわたり、普段はダンス部が利用しているが時には応援団の練習場所としてもつかわれる。大した場所ではないがここまで使わないとあの狭い校舎には人が入りきらない。"},
      {q:"第76th縣陵祭テーマソングは？", a:"ひゃっぽ", comment:"神曲。もうすぐでYouTube1万回再生される、第76回縣陵祭のテーマソングである。下級生から神と称えられるメンバーにより作詞作曲された。ほんと神曲。好き。\nhttps://youtu.be/9EJMJH15_Go?feature=shared"},
      {q:"焼肉きんぐあがた店はかつてなんだった？", a:"おこほん", comment:"広丘駅に大きな店舗があり、昔はすべての縣陵生徒がそこでパーティーをしていた。ただしおこほん運営も縣陵生が消費量の大半を占めていることを知り、松本県店を進出させた。今では焼肉きんぐに。"}
    ];

    const HIRAGANA = [..."あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわをんっょゃぽだゅがづ"];
    const KATAKANA = [..."アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲンーガィピォゴド"];
    const ALPHABET = [..."ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"];

    let quizData = [], current = 0, score = 0, bestScore = 0, currentAnswer = "", answerProgress = "", revealTimer = null, startTime = 0;

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

    let timer = null;

    startBtn.onclick = () => {
      startBtn.style.display = 'none';
      score = 0;
      current = 0;
      quizData = [...fullData].sort(() => Math.random() - 0.5).slice(0, 5);
      nextBtn.style.display = 'none';
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
      let index = answerProgress.length;
      if (index >= currentAnswer.length) {
        stopTimer();
        let elapsed = (Date.now() - startTime) / 1000;
        let displayLength = document.getElementById('question').innerText.length;
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
      timer = setInterval(() => {
        time--;
        timerEl.innerText = time;
        if (time <= 0) {
          stopTimer();
          wrongSound.play();
          feedbackBox.innerText = `時間切れ…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
          feedbackBox.style.opacity = 1;
          nextBtn.style.display = 'inline';
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


   
