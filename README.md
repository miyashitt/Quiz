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

  const HIRAGANA = [...'あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわをん'];
  const KATAKANA = [...'アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲンー'];
  const ALPHABET = [...'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz'];

  let quizData = [], current = 0, score = 0, timer = null;
  let revealCount = 0;
  let currentAnswer = '', answerProgress = '';
  let bestScore = 0;
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
    bestBox.innerHTML = bestScore ? `ベストスコア: ${bestScore}` : '';
    quizData = [...fullData].sort(() => Math.random() - 0.5).slice(0, 5);
    bgm.volume = 0.3;
    bgm.play();
    showQuestion();
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

  buzzBtn.onclick = () => {
    if(questionRevealingInterval){
      clearInterval(questionRevealingInterval);
      questionRevealingInterval = null;
      qEl.textContent = quizData[current].q;
    }
    buzzBtn.style.display = 'none';
    startTimer();
    revealCount = qEl.textContent.length; 
    answerProgress = '';
    startTime = Date.now();
    showNextChar();
  };

  function showQuestion() {
    clearInterval(timer);
    if(questionRevealingInterval){
      clearInterval(questionRevealingInterval);
      questionRevealingInterval = null;
    }
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
      if(i < q.q.length){
        qEl.textContent += q.q[i];
        i++;
        revealCount = i;
      } else {
        clearInterval(questionRevealingInterval);
        questionRevealingInterval = null;
      }
    }, 200); // 表示速度を100msから200msに変更（半分の速度）

    buzzBtn.style.display = 'inline-block';
    nextBtn.style.display = 'none';
    choicesEl.innerHTML = '';
  }

  function showNextChar() {
    const q = quizData[current];
    const index = answerProgress.length;

    let answerType = 'hiragana';
    if(/^[ァ-ヶー　]+$/.test(currentAnswer)) answerType = 'katakana';
    else if(/^[A-Za-z]+$/.test(currentAnswer)) answerType = 'alphabet';

    if(index >= currentAnswer.length){
      stopTimer();
      correctSound.currentTime = 0;
      correctSound.play();

      const totalLen = quizData[current].q.length;
      const pressedLen = revealCount;
      let basePoint = 50;
      let bonusPoint = 0;
      if(pressedLen <= totalLen / 3){
        bonusPoint = 50;
      } else if(pressedLen <= totalLen * 2 / 3){
        bonusPoint = 25;
      } else {
        bonusPoint = 10;
      }
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
    while(choices.length < 6){
      let rand = pool[Math.floor(Math.random() * pool.length)];
      if(!choices.includes(rand)) choices.push(rand);
    }
    choices.sort(() => Math.random() - 0.5);

    choicesEl.innerHTML = '';
    answerBox.textContent = answerProgress;

    choices.forEach(char => {
      const btn = document.createElement('div');
      btn.className = 'choice';
      btn.textContent = char;
      btn.onclick = () => {
        if(char === currentAnswer[index]){
          answerProgress += char;
          showNextChar();
        } else {
          stopTimer();
          wrongSound.currentTime = 0;
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

  function startTimer(){
    let time = 15;
    timeEl.textContent = time;
    timer = setInterval(() => {
      time--;
      timeEl.textContent = time;
      if(time <= 0){
        clearInterval(timer);
        wrongSound.currentTime = 0;
        wrongSound.play();

        feedbackBox.textContent = `時間切れ…\n【正解】${quizData[current].a}\n${quizData[current].comment || ''}`;
        feedbackBox.style.opacity = 1;
        nextBtn.style.display = 'inline-block';
        choicesEl.innerHTML = '';
        answerBox.textContent = quizData[current].a;
      }
    }, 1000);
  }

  function stopTimer(){
    clearInterval(timer);
  }

  function showScore(){
    qEl.textContent = '';
    choicesEl.innerHTML = '';
    timeEl.textContent = '0';
    answerBox.textContent = '';
    feedbackBox.textContent = '';
    feedbackBox.style.opacity = 0;

    if(score > bestScore) bestScore = score;

    let rank;
    if(score >= 450) rank = '神ランク ✨';
    else if(score >= 350) rank = 'マスターランク 💪';
    else if(score >= 250) rank = 'ノーマルランク 👍';
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
