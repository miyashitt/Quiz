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
    const fullData = [
      { q: "伝説の文化祭OPは？○○○○サマー", a: "アーリー", comment: "幾度となく塗り替えようとされてきたがいまだにこれを超えるクオリティーの曲は発表されていない。音源は生徒会が管理しており、一般生徒が触れることはできない" },
      { q: "専門は家族社会学、ジェンダー論、女性学である、日本のフェミニスト・社会学者は？", a: "うえのちづこ", comment: "" },
      { q: "高校生向け化学の動画を投稿し大学入試センターと戦うチャンネルは？\nOnline Chemistry by ○○○○○", a: "ヒガシマキ", comment: "https://youtu.be/ZvE1JMkcj3A?feature=shared" },
      { q: "縣陵生になると体育の時間に覚えさせられるものは?", a: "けんりょうたいそう", comment: "真偽不明の由来故にハードで長め。2年生以降ではその存在は突然無くなる。" },
      { q: "質実剛健であれ　大道を闊歩せよ　あとひとつは？", a: "よわねをはくな", comment: "三大精神の一つ。ほとんどの縣陵生は弱音を吐くなしか知らない。" },
      { q: "地球の会←なんて読む？", a: "そらのかい", comment: "難読。部活は月1。" },
      { q: "縣陵応援団の言うPTAのAとは?", a: "アルコール", comment: "パチンコ・タバコ・アルコールの略。応援団の掛け声でも使用される。" },
      { q: "小体育館の下に存在している場所は？", a: "ピロティ", comment: "特に1年生の物販委販売時に迷子が続出。" },
      { q: "第76th縣陵祭テーマソングは？", a: "ひゃっぽ", comment: "神曲。もうすぐYouTube1万回再生。" },
      { q: "焼肉きんぐあがた店はかつてなんだった？", a: "おこほん", comment: "昔は全縣陵生がパーティーをしていた。" },
      { q: "2学年が探究成果を発表する大会とは？", a: "KRGP", comment: "優秀賞6名から1名が大賞に選ばれる。" },
      { q: "お昼に流れる校内放送の名称は？", a: "けんりょうオンエア", comment: "実質、縣陵版のN〇K。" },
      { q: "県ケ丘高校の文化祭の名称は？", a: "けんりょうさい", comment: "来場者5000人。山形村の人口くらい。" },
      { q: "長野県松本市に本部を置く国立大学は？", a: "しんしゅうだいがく", comment: "縣陵生が実質支配しているとの噂。" },
      { q: "学校で生徒が主体的に活動する組織は？", a: "せいとかい", comment: "事実上の独裁と思われがち。" },
      { q: "松本市中心部の周遊バスは？", a: "タウンスニーカー", comment: "雨の日はバス停が混雑。" },
      { q: "松本県ヶ丘高校の学科は？", a: "たんきゅうか", comment: "英語科を前身とする。変人:変態:常人＝5:4:1" },
      { q: "学校で行われる定期的な試験は？", a: "ていきこうさ", comment: "英気を養う7日間の前に行われる。" },
      { q: "書籍資料を保存し提供する施設は？", a: "としょかん", comment: "授業をサボっても罪に問われない。" },
      { q: "応援練習で発声する語は？", a: "け", comment: "" },
      { q: "学校教育法で定められた春の休みは？", a: "はるやすみ", comment: "2週間以下。課題は鬼。" },
      { q: "啓林館の網羅系数学参考書は？", a: "フォーカスゴールド", comment: "聖書・鈍器・漬物石など多用途。" },
      { q: "応援歌で唯一毛色の違う三文字は？", a: "ラララ", comment: "「ラーラーラ　そーれ」の掛け声が特徴。" },
      { q: "現役合格できず翌年も受験する人は？", a: "ろうにん", comment: "高校生活をもう1年行えるスポーツ。" }
    ];

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

    let revealInterval;

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
      choicesEl.innerHTML = '';
      answerBox.innerText = '';
      feedbackBox.innerText = '';
      feedbackBox.style.opacity = 0;
      answerProgress = '';
      currentAnswer = q.a;
      buzzBtn.style.display = 'inline';
      startTime = Date.now();
      let i = 0;
      revealInterval = setInterval(() => {
        if (i < q.q.length) qEl.innerHTML += q.q[i++];
        else clearInterval(revealInterval);
      }, 100);

      buzzBtn.onclick = () => {
        buzzBtn.style.display = 'none';
        clearInterval(revealInterval);
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
      const pool = (/^[ァ-ンー]+$/.test(currentAnswer) ? KATAKANA : HIRAGANA);
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
            score -= 50;
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
