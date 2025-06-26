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
    #feedbackBox { transition: opacity 0.6s ease; }
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
      {q:"焼肉きんぐあがた店はかつてなんだった？", a:"おこほん", comment:"広丘駅に大きな店舗があり、昔はすべての縣陵生徒がそこでパーティーをしていた。ただしおこほん運営も縣陵生が消費量の大半を占めていることを知り、松本県店を進出させた。今では焼肉きんぐに。"},
      {q:"2学年が探究成果を発表する大会とは？", a:"KRGP", comment:"優秀賞が普通科から3名、探究科から3名、計6名選出され、その中から大賞が1名大学の教授や校長によって選ばれる。"},
      {q:"お昼に流れる校内放送の名称は？", a:"けんりょうオンエア", comment:"独立したメディアかと思えばコンテンツ内容に関しては生徒会が一枚かんでいる。要するに縣陵版のN〇Kである。"},
      {q:"県ケ丘高校の文化祭の名称は？", a:"けんりょうさい", comment:"入場者は5000人程で山形村の人口くらい。毎年この準備のため生徒会役員は官僚レベルの重労働が強制される。"},
      {q:"8つの学部と6つの大学院を持つ総合大学で、学部には人文学部、教育学部、経法学部、理学部、医学部、工学部、農学部、繊維学部がある、長野県松本市に本部を置く国立大学は？", a:"しんしゅうだいがく", comment:"縣陵生が実質支配しているといっても過言ではない大学。松本にもキャンパスがあり、一年生は全員松本で過ごすので松本のことをよく知っている縣陵生は少し優位に立てるといわれいている。しかも実家通いのためお金の心配をしなくてもよい。"},
      {q:"中学校や高等学校において、生徒が主体的に学校生活の改善や向上を目指すための組織は？", a:"せいとかい", comment:"事実上の独裁体制を敷いていると思われがちだが、アニメやラノベ小説の世界ほど権力は高くない。"},
      {q:"松本市の中心部を走る城下町まつもとの有名観光スポットを巡るのにぴったりな周遊バスは？", a:"タウンスニーカー", comment:"学校から松本駅までの間を送迎してくれるサービス。雨の日は特に使用率が高く学校前のバス停には長蛇の列ができる。\n7月豪雨の際にはバス待ちの生徒の列が松本駅まで続いたという。"},
      {q:"基礎的な学力に加え、知識を総合的に活用する能力や、課題解決力、創造力、表現力を養うことを目的とする、生徒が自ら課題を設定し、解決に向けて探究活動を行うことを重視した松本県ヶ丘高校の学科は？", a:"たんきゅうか", comment:"英語科を前身として生み出された精鋭部隊。週2回の探究の授業を組み込むために授業進度はかなり無理をして頑張っている。倍率は年々異なるがだいたい2倍近くあり小論文がどんどん難しくなっている。変人:変態:常人＝5:4:1"},
      {q:"学校で、各教科の学習成果を評価するために、定期的に行われる試験は？", a:"ていきこうさ", comment:"全校強制参加のエクストリームスポーツ。年5回ほど行われていて、英気を養うための7日間というものがテスト前に設けられている。生徒たちはこの期間に日々の生活時間の乱れを回復するために睡眠時間を多くとる"},
      {q:"書籍や記録などの資料を収集・整理・保存し、一般の人々が利用できるように提供する施設は？", a:"としょかん", comment:"本を読むまたは借りる目的で使う人は存在しない。治外法権が適応されている場所であり、授業をサボることが罪に問われない。ただし文系科目の自習の時間に使われることがまれにあるため、容易に授業中逃げ込むことは避けた方が良い。"},
      {q:"応援練習における、発声練習で発する語は？", a:"け", comment:""},
      {q:"学校教育法で定められた春季休暇は？", a:"はるやすみ", comment:"春休みと呼ばれる期間はたった2週間以下だが、３月はなんだかんだ休みがたくさんあるので困ることはない。春が来たからと言って君の彼女いない歴＝年齢の呪いが終わるわけではないのでせいぜい頑張ってほしい。お決まり通り課題は鬼。"},
      {q:"数研出版から出版されている高校数学の網羅系参考書で、一般に上から二番目の難易度とされる問題集は青チャートであるが、それと比較されがちな啓林館が出版しているマスター編、チャレンジ編、実践編で構成される参考書は？", a:"フォーカスゴールド", comment:"世間一般でも使われている標準的な数学参考書。その使い方は実に多様であり漬物石、トレーニング、鈍器、聖書…etc.　"},
      {q:"松本県ヶ丘高等学校の応援歌において1つだけ毛色の異なる応援歌はカタカナ三文字で？", a:"ラララ", comment:"応援歌の一つ。その歌い出しと、途中の掛け声が特徴である。1年生のうちは応援団の恐怖により触れることを畏れるが、2・3年生になると仲間内でふざけている最中に「ラーラーラ　そーーれ」という掛け声とともに歌い出す。"},
      {q:"主に大学受験において、現役で志望校に合格できず、もう一年受験勉強に励む人のことをなんと言う？", a:"ろうにん", comment:"楽しい高校生活をもう一年行えるエクストリームスポーツ。"}
    ];

    const HIRAGANA = [..."あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわをん"];
    const KATAKANA = [..."アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲンー"];
    const ALPHABET = [..."ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"];
    const NUMBERS = [..."0123456789"];
    const SYMBOLS = [..."！？、。ー・"]; 

    let quizData = [], current = 0, score = 0, bestScore = 0,
        currentAnswer = "", answerProgress = "",
        revealTimer = null, answerTimer = null,
        startTime = 0;

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

    const QUESTION_REVEAL_INTERVAL = 150;
    const ANSWER_TIME_LIMIT = 15;

    startBtn.onclick = () => {
      startBtn.style.display = 'none';
      score = 0;
      current = 0;
      quizData = [...fullData].sort(() => Math.random() - 0.5).slice(0, 5); // 5問に限定
      nextBtn.style.display = 'none';
      nextBtn.innerText = '次の問題'; // ボタンのテキストをリセット
      nextBtn.onclick = handleNextQuestion; // クリックイベントをリセット
      showQuestion();
    };

    // nextBtnのデフォルトのクリックハンドラ
    function handleNextQuestion() {
      current++;
      if (current < quizData.length) {
        showQuestion();
      } else {
        // 最終問題の場合はボタンを「結果を見る」に切り替える
        nextBtn.innerText = '結果を見る';
        nextBtn.onclick = showScore; // クリックイベントをshowScoreに設定
      }
    }

    function showQuestion() {
      clearTimers();

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
      }, QUESTION_REVEAL_INTERVAL);
    }

    buzzBtn.onclick = () => {
      clearInterval(revealTimer);
      buzzBtn.style.display = 'none';
      
      startAnswerTimer();
      showChoicesForNextChar();
    };

    function startAnswerTimer() {
      let time = ANSWER_TIME_LIMIT;
      timerEl.innerText = time;
      answerTimer = setInterval(() => {
        time--;
        timerEl.innerText = time;
        if (time <= 0) {
          stopTimer();
          wrongSound.play();
          choicesEl.innerHTML = '';
          answerBox.innerText = answerProgress + '...';
          feedbackBox.innerText = `時間切れ…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
          feedbackBox.style.opacity = 1;
          nextBtn.style.display = 'inline';
          score -= 20;
          // 最終問題の場合はボタンを「結果を見る」に切り替える
          if (current === quizData.length - 1) {
            nextBtn.innerText = '結果を見る';
            nextBtn.onclick = showScore;
          }
        }
      }, 1000);
    }

    function showChoicesForNextChar() {
      let index = answerProgress.length;

      if (index >= currentAnswer.length) {
        stopTimer();
        let elapsed = (Date.now() - startTime) / 1000;
        let displayLength = document.getElementById('question').innerText.length;
        let fullLength = quizData[current].q.length;
        
        let bonus = 0;
        if (displayLength <= fullLength / 3) {
          bonus = 50;
        } else if (displayLength <= (fullLength * 2 / 3)) {
          bonus = 25;
        } else {
          bonus = 10;
        }
        score += 50 + bonus;

        correctSound.play();
        choicesEl.innerHTML = '';
        answerBox.innerText = currentAnswer;
        feedbackBox.innerText = `正解！\n【答え】${quizData[current].a}\n${quizData[current].comment || ''}`;
        feedbackBox.style.opacity = 1;
        nextBtn.style.display = 'inline';
        
        // 最終問題の場合はボタンを「結果を見る」に切り替える
        if (current === quizData.length - 1) {
          nextBtn.innerText = '結果を見る';
          nextBtn.onclick = showScore;
        }
        return;
      }

      const currentChar = currentAnswer[index];
      const charType = getCharType(currentChar);

      choicesEl.innerHTML = '';
      answerBox.innerText = answerProgress;

      if (charType === 'other') {
        answerProgress += currentChar;
        showChoicesForNextChar();
        return;
      }

      const pool = getCharPool(charType);
      let choices = [currentChar];

      while (choices.length < 6) {
        let r = pool[Math.floor(Math.random() * pool.length)];
        if (!choices.includes(r)) choices.push(r);
      }
      choices = choices.sort(() => Math.random() - 0.5);

      choices.forEach(c => {
        let div = document.createElement('div');
        div.className = 'choice';
        div.innerText = c;
        div.onclick = () => {
          if (c === currentChar) {
            answerProgress += c;
            showChoicesForNextChar();
          } else {
            stopTimer();
            wrongSound.play();
            choicesEl.innerHTML = '';
            answerBox.innerText = answerProgress + '...';
            feedbackBox.innerText = `不正解…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
            feedbackBox.style.opacity = 1;
            score -= 20;
            nextBtn.style.display = 'inline';
            // 最終問題の場合はボタンを「結果を見る」に切り替える
            if (current === quizData.length - 1) {
              nextBtn.innerText = '結果を見る';
              nextBtn.onclick = showScore;
            }
          }
        };
        choicesEl.appendChild(div);
      });
    }

    function getCharType(char) {
      if (/[ぁ-ん]/.test(char)) return 'hiragana';
      if (/[ァ-ン]/.test(char)) return 'katakana';
      if (/[a-zA-Z]/.test(char)) return 'alphabet';
      if (/[0-9]/.test(char)) return 'number';
      return 'other'; 
    }

    function getCharPool(type) {
      switch (type) {
        case 'hiragana': return HIRAGANA;
        case 'katakana': return KATAKANA;
        case 'alphabet': return ALPHABET;
        case 'number': return NUMBERS;
        default: return [];
      }
    }

    function clearTimers() {
      clearInterval(revealTimer);
      clearInterval(answerTimer);
    }

    function stopTimer() {
      clearInterval(answerTimer);
    }

    function showScore() {
      clearTimers();

      qEl.innerText = '';
      choicesEl.innerHTML = '';
      timerEl.innerText = '0';
      answerBox.innerText = '';
      feedbackBox.innerText = '';
      nextBtn.style.display = 'none';
      scoreBox.innerHTML = `今回のスコア：${score} / ${quizData.length * 100}`;
      if (score > bestScore) bestScore = score;
      bestBox.innerHTML = `ベストスコア：${bestScore} / ${quizData.length * 100}`;
      startBtn.style.display = 'inline';
      // 結果表示後、次のゲーム開始に備えてボタンテキストをリセット
      nextBtn.innerText = '次の問題'; 
      nextBtn.onclick = handleNextQuestion;
    }
  </script>
</body>
</html>
