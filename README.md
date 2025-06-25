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
    #startBtn, #nextBtn { padding: 10px 20px; font-size: 1.2em; cursor: pointer; }
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
    <div class="choices" id="choices"></div>
    <div id="feedbackBox"></div>
    <button id="nextBtn" style="display:none;">次の問題</button>
  </div>
  <div id="scoreBox"></div>
  <button id="startBtn">ゲームスタート</button>

  <script>
    const quizData = [
      { q: '伝説の文化祭OPは？○○○○サマー', a: 'アーリー', type: 'katakana', comment: '幾度となく塗り替えようとされたが、今なお最強の文化祭OP。' },
      { q: '専門は家族社会学、ジェンダー論、女性学である、日本のフェミニスト・社会学者は？', a: 'うえのちづこ', type: 'hiragana', comment: '上野千鶴子氏。現代の社会学とフェミニズムに多大な影響を与えた人物。' },
      { q: '高校生向け化学の動画を投稿し大学入試センターと戦うチャンネルは？Online Chemistry by ○○○○○', a: 'ヒガシマキ', type: 'katakana', comment: 'YouTubeで話題の化学系YouTuber。教育への貢献が光る。' },
      { q: '縣陵生になると体育の時間に覚えさせられるものは?', a: 'けんりょうたいそう', type: 'hiragana', comment: '準備体操とは思えないハードな内容。1年生の通過儀礼。' },
      { q: '質実剛健であれ　大道を闊歩せよ　あとひとつは？', a: 'よわねをはくな', type: 'hiragana', comment: '縣陵三大精神のひとつ。「弱音を吐くな」は有名。' },
      { q: '地球の会←なんて読む？', a: 'そらのかい', type: 'hiragana', comment: 'ただの初見殺し。読めない人多数。' },
      { q: 'PTAのAとは?', a: 'アルコール', type: 'katakana', comment: 'パチンコ・タバコ・アルコールの略。応援団のネタで有名。' },
      { q: '小体育館の下に存在している場所は？', a: 'ピロティ', type: 'katakana', comment: '名前だけ聞いても場所が想像しづらい。迷子続出。' },
      { q: '第76th縣陵祭テーマソングは？', a: 'ひゃっぽ', type: 'hiragana', comment: '再生回数1万目前の神曲。76thを代表する名作。' },
    ];

    const HIRAGANA = 'あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわをん'.split('');
    const KATAKANA = 'アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲンー'.split('');

    let current = 0, score = 0, timer, startTime, currentAnswer = '', answerProgress = '';

    const qEl = document.getElementById('question');
    const choicesEl = document.getElementById('choices');
    const nextBtn = document.getElementById('nextBtn');
    const scoreBox = document.getElementById('scoreBox');
    const timeEl = document.getElementById('time');
    const startBtn = document.getElementById('startBtn');
    const answerBox = document.getElementById('answerBox');
    const feedbackBox = document.getElementById('feedbackBox');

    startBtn.onclick = () => {
      startBtn.style.display = 'none';
      nextBtn.style.display = 'none';
      scoreBox.innerHTML = '';
      current = 0;
      score = 0;
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
      startTime = Date.now();

      let reveal = setInterval(() => {
        if (i < q.q.length) {
          qEl.innerHTML += q.q[i++];
        } else {
          clearInterval(reveal);
          startTimer();
          showNextChar();
        }
      }, 100); // ← 表示速度を遅くした（以前は50）
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
      const targetChar = currentAnswer[index];
      const pool = (q.type === 'katakana' ? KATAKANA : HIRAGANA);
      let choices = [targetChar];
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
          if (char === targetChar) {
            answerProgress += char;
            showNextChar();
          } else {
            stopTimer();
            answerBox.innerText += ` ✕（間違い）`;
            feedbackBox.innerText = `不正解…\n【正解】${q.a}\n${q.comment || ''}`;
            feedbackBox.style.opacity = 1;
            nextBtn.style.display = 'inline';
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
          answerBox.innerText += ` ⏰（時間切れ）`;
          const q = quizData[current];
          feedbackBox.innerText = `時間切れ…\n【正解】${q.a}\n${q.comment || ''}`;
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

      let rank = '';
      if (score >= 1200) rank = '神ランク ✨';
      else if (score >= 900) rank = 'マスターランク 💪';
      else if (score >= 600) rank = 'ノーマルランク 👍';
      else rank = 'ビギナー 🔰';

      scoreBox.innerHTML = `あなたのスコア：${score} / 1350<br>ランク：${rank}`;
      startBtn.style.display = 'inline';
    }
  </script>
</body>
</html>
