# namurulru.github.io

Study
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>일본어 단어 학습 프로그램</title>
    <style>
        body {
            font-family: 'Malgun Gothic', sans-serif;
            background-color: #f0f2f5;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .card {
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            width: 400px;
            text-align: center;
            position: relative;
            overflow: hidden;
        }
        h2 {
            margin-top: 0;
            color: #333;
        }
        .word-display {
            font-size: 24px;
            font-weight: bold;
            margin: 20px 0;
            color: #4a90e2;
        }
        .pronunciation {
            font-size: 16px;
            color: #777;
            margin-bottom: 20px;
        }
        input[type="text"] {
            width: 80%;
            padding: 10px;
            font-size: 16px;
            border: 2px solid #ddd;
            border-radius: 5px;
            outline: none;
            transition: border-color 0.3s;
        }
        input[type="text"]:focus {
            border-color: #4a90e2;
        }
        .btn-group {
            margin-top: 15px;
        }
        button {
            padding: 10px 15px;
            font-size: 14px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
            background-color: #4a90e2;
            color: white;
            transition: background 0.2s;
        }
        button:hover {
            background-color: #357abd;
        }
        .hint-btn {
            background-color: #f0ad4e;
        }
        .hint-btn:hover {
            background-color: #ec971f;
        }
        /* 위에서 아래로 내려오는 애니메이션 안내 문구 */
        .message-box {
            background-color: #e2f0d9;
            color: #385723;
            padding: 10px;
            border-radius: 5px;
            margin-top: 15px;
            font-size: 14px;
            font-weight: bold;
            animation: slideDown 0.4s ease-out forwards;
            opacity: 0;
            transform: translateY(-20px);
        }
        .message-box.error {
            background-color: #fce4d6;
            color: #c65911;
        }
        .hint-box {
            background-color: #fff2cc;
            color: #7f6000;
            padding: 8px;
            border-radius: 5px;
            margin-top: 8px;
            font-size: 13px;
            text-align: left;
            animation: slideDown 0.3s ease-out forwards;
        }
        @keyframes slideDown {
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
    </style>
</head>
<body>

<div class="card">
    <h2>일본어 단어 퀴즈</h2>
    <hr>
    <div class="word-display" id="jpWord">日本語</div>
    <div class="pronunciation" id="pronunciation">(발음)</div>
    
    <p>뜻을 입력하세요!</p>
    <input type="text" id="answerInput" placeholder="정답 입력" onkeypress="handleKeyPress(event)">
    
    <div class="btn-group">
        <button onclick="checkAnswer()">정답 확인</button>
        <button class="hint-btn" onclick="showHint(1)">힌트 1</button>
        <button class="hint-btn" onclick="showHint(2)">힌트 2</button>
        <button onclick="nextQuestion()" style="background-color: #6c757d;">넘어가기</button>
    </div>

    <div id="messageContainer"></div>
    <div id="hintContainer"></div>
</div>

<script>
// 단어 데이터 목록 (일본어 표기 추가 및 유사어 배열 처리)
const wordList = [
    { jp: "海鮮丼", pr: "카이센돈", meaning: ["해산물덮밥"] },
    { jp: "味噌汁", pr: "미소시루", meaning: ["미소", "미소된장국"] },
    { jp: "上", pr: "우에", meaning: ["위"] },
    { jp: "下", pr: "시타", meaning: ["아래"] },
    { jp: "中", pr: "나카", meaning: ["안", "속"] },
    { jp: "隣", pr: "토나리", meaning: ["옆"] },
    { jp: "左", pr: "히다리", meaning: ["왼쪽"] },
    { jp: "右", pr: "미기", meaning: ["오른쪽"] },
    { jp: "後ろ", pr: "우시로", meaning: ["뒤"] },
    { jp: "図書館", pr: "토쇼칸", meaning: ["도서관"] },
    { jp: "どこですか", pr: "도코데스카", meaning: ["어디입니까", "어디에요"] },
    { jp: "学校", pr: "캌코ㅡ", meaning: ["학교"] },
    { jp: "公園", pr: "코ㅡ엔", meaning: ["공원", "공연"] },
    { jp: "犬", pr: "이누", meaning: ["개"] },
    { jp: "そば", pr: "소바", meaning: ["국수", "메밀국수"] },
    { jp: "牛丼", pr: "규ㅡ돈", meaning: ["소고기덮밥"] },
    { jp: "うどん", pr: "우돈", meaning: ["우동"] },
    { jp: "安い", pr: "야스이", meaning: ["싸다"] },
    { jp: "美味しい", pr: "오이시ㅡ", meaning: ["맛있다"] },
    { jp: "大きい", pr: "오ㅡ키ㅡ", meaning: ["크다"] },
    { jp: "白い", pr: "시로이", meaning: ["하얗다", "하얀색"] },
    { jp: "可愛い", pr: "카와이ㅡ", meaning: ["귀엽다"] },
    { jp: "辛い", pr: "카라이", meaning: ["맵다"] },
    { jp: "あまり", pr: "아마리", meaning: ["별로"] },
    { jp: "静かだ", pr: "시즈카다", meaning: ["조용하다"] },
    { jp: "綺麗다", pr: "키레ㅡ다", meaning: ["깨끗하다", "예쁘다"] },
    { jp: "親切だ", pr: "신세츠다", meaning: ["친절하다"] },
    { jp: "簡単다", pr: "칸탄다", meaning: ["간단하다"] },
    { jp: "店", pr: "미세", meaning: ["가게"] },
    { jp: "得意다", pr: "토쿠이다", meaning: ["특기다", "잘하다"] },
    { jp: "夏", pr: "나츠", meaning: ["여름"] },
    { jp: "色々", pr: "이로이로", meaning: ["여러가지", "여러 가지"] },
    { jp: "お待たせしました", pr: "오마타세시마시타", meaning: ["오래기다리렸습니다", "오래 기다렸습니다"] },
    { jp: "全然", pr: "젠전", meaning: ["전혀"] },
    { jp: "お腹", pr: "오나카", meaning: ["배"] },
    { jp: "広い", pr: "히로이", meaning: ["넓다"] },
    { jp: "値段", pr: "네단", meaning: ["가격"] },
    { jp: "ごちそうさまでした", pr: "고치소ㅡ사마데시타", meaning: ["잘먹었습니다", "잘 먹었습니다"] },
    { jp: "親子丼", pr: "오야코돈", meaning: ["부모자식덮밥"] },
    { jp: "しゃぶしゃぶ", pr: "샤부샤부", meaning: ["샤브샤브"] },
    { jp: "すき焼き", pr: "스키야키", meaning: ["스키야키"] },
    { jp: "おでん", pr: "오덴", meaning: ["오뎅", "어묵"] },
    { jp: "寿司", pr: "스시", meaning: ["초밥"] },
    { jp: "チラ시寿司", pr: "치라시즈시", meaning: ["치라시즈시"] },
    { jp: "手巻き寿司", pr: "테마키즈시", meaning: ["테마키즈시"] },
    { jp: "四時", pr: "요지", meaning: ["네시", "4시"] },
    { jp: "七時", pr: "시치지", meaning: ["일곱시", "7시"] },
    { jp: "九時", pr: "쿠지", meaning: ["아홉시", "9시"] },
    { jp: "月曜日", pr: "게츠요ㅡ비", meaning: ["월요일"] },
    { jp: "火曜日", pr: "카요ㅡ비", meaning: ["화요일"] },
    { jp: "水曜日", pr: "스이요ㅡ비", meaning: ["수요일"] },
    { jp: "木曜日", pr: "모쿠요ㅡ비", meaning: ["목요일"] },
    { jp: "金曜日", pr: "킨요ㅡ비", meaning: ["금요일"] },
    { jp: "土曜日", pr: "도요ㅡ비", meaning: ["토요일"] },
    { jp: "일요일", pr: "니치요ㅡ비", meaning: ["일요일"] },
    { jp: "どうですか", pr: "도ㅡ데스카", meaning: ["어때요"] },
    { jp: "焼きそば", pr: "야키소바", meaning: ["볶음국수", "야키소바"] },
    { jp: "自転車", pr: "지텐샤", meaning: ["자전거"] },
    { jp: "どうしたの", pr: "도ㅡ시타노", meaning: ["무슨일이야", "무슨 일이야"] },
    { jp: "頭", pr: "아타마", meaning: ["머리"] },
    { jp: "寒い", pr: "사무이", meaning: ["춥다"] },
    { jp: "会う", pr: "아우", meaning: ["만나다"] },
    { jp: "遊ぶ", pr: "아소부", meaning: ["놀다"] },
    { jp: "座る", pr: "스와루", meaning: ["앉다"] },
    { jp: "話す", pr: "하나스", meaning: ["이야기하다", "말하다"] },
    { jp: "読む", pr: "요무", meaning: ["읽다"] },
    { jp: "新聞", pr: "신분", meaning: ["신문"] },
    { jp: "映画", pr: "에ㅡ가", meaning: ["영화"] },
    { jp: "英語", pr: "에ㅡ고", meaning: ["영어"] },
    { jp: "勉強", pr: "벤쿄ㅡ", meaning: ["공부"] },
    { jp: "昨日", pr: "키노ㅡ", meaning: ["어제"] },
    { jp: "今日", pr: "쿄ㅡ", meaning: ["오늘"] },
    { jp: "明日", pr: "아시타", meaning: ["내일"] },
    { jp: "ワクワク", pr: "와쿠와쿠", meaning: ["두근두근"] },
    { jp: "是非", pr: "제히", meaning: ["부디", "꼭"] },
    { jp: "毎日", pr: "마이니치", meaning: ["매일"] },
    { jp: "ニコニコ", pr: "니코니코", meaning: ["웃음", "싱글벙글"] },
    { jp: "キラキラ", pr: "키라키라", meaning: ["반짝반짝"] },
    { jp: "ペコペコ", pr: "페코페코", meaning: ["꼬륵꼬륵", "배고픈모양"] },
    { jp: "그리고", pr: "소시테", meaning: ["그리고"] },
    { jp: "一緒に", pr: "잇쇼니", meaning: ["함께", "같이"] },
    { jp: "まだまだ", pr: "마다마다", meaning: ["아직", "아직멀었습니다"] },
    { jp: "絵", pr: "에", meaning: ["그림"] },
    { jp: "やった", pr: "얕타", meaning: ["해냈다"] },
    { jp: "気をつけて", pr: "키오 츠케테", meaning: ["조심해", "조심하세요"] },
    { jp: "残念だね", pr: "잔넨다네", meaning: ["유감이다", "아쉽네"] },
    { jp: "聞く", pr: "키쿠", meaning: ["듣다"] },
    { jp: "撮る", pr: "토루", meaning: ["찍다"] },
    { jp: "買い物", pr: "카이모노", meaning: ["쇼핑", "물건사기"] },
    { jp: "もうすぐ", pr: "모ㅡ스구", meaning: ["이제곧", "이제 곧"] },
    { jp: "早く", pr: "하야쿠", meaning: ["빠르게", "빨리"] },
    { jp: "相槌", pr: "아이즈치", meaning: ["맞장구"] },
    { jp: "四月", pr: "시가츠", meaning: ["4월", "네월"] },
    { jp: "七月", pr: "시치가츠", meaning: ["7월", "일곱월"] },
    { jp: "九月", pr: "쿠가츠", meaning: ["9월", "아홉월"] },
    { jp: "入学式", pr: "뉴가쿠시기", meaning: ["입학식"] },
    { jp: "文化財", pr: "분카사이", meaning: ["문화재"] },
    { jp: "夏休み", pr: "나츠야스미", meaning: ["여름방학"] },
    { jp: "卒業式", pr: "소츠교ㅡ시키", meaning: ["졸업식"] },
    { jp: "いつ", pr: "이츠", meaning: ["언제"] },
    { jp: "から", pr: "카라", meaning: ["부터"] },
    { jp: "まで", pr: "마데", meaning: ["까지"] },
    { jp: "何にしますか", pr: "나니니 시마스카", meaning: ["무엇으로할까요", "무엇으로 하시겠습니까"] },
    { jp: "いただきます", pr: "이타다키마스", meaning: ["잘먹겠습니다", "잘 먹겠습니다"] },
    { jp: "高い", pr: "타카이", meaning: ["비싸다", "높다"] },
    { jp: "好きだ", pr: "스키다", meaning: ["좋아하다"] },
    { jp: "ちょっと", pr: "춑토", meaning: ["조금"] },
    { jp: "行く", pr: "이쿠", meaning: ["가다"] },
    { jp: "見る", pr: "미루", meaning: ["보다"] },
    { jp: "する", pr: "스루", meaning: ["하다"] },
    { jp: "来る", pr: "쿠루", meaning: ["오다"] },
    { jp: "そろそろ", pr: "소로소로", meaning: ["슬슬"] },
    { jp: "今から", pr: "이마카라", meaning: ["지금부터"] },
    { jp: "友達", pr: "토모다치", meaning: ["친구"] },
    { jp: "一人", pr: "히토리", meaning: ["한사람", "한명", "1명"] },
    { jp: "二人", pr: "후타리", meaning: ["두사람", "두명", "2명"] },
    { jp: "三人", pr: "산닌", meaning: ["세사람", "세명", "3명"] },
    { jp: "餅", pr: "모치", meaning: ["떡"] },
    { jp: "大丈夫だよ", pr: "다이죠ㅡ부다요", meaning: ["괜찮아", "괜찮아요"] },
    { jp: "相撲", pr: "스모", meaning: ["스모"] },
    { jp: "買う", pr: "카우", meaning: ["사다"] },
    { jp: "電話", pr: "덴화", meaning: ["전화"] },
    { jp: "ご飯", pr: "고한", meaning: ["밥"] },
    { jp: "掃除", pr: "소ㅡ지", meaning: ["청소"] },
    { jp: "書く", pr: "카쿠", meaning: ["쓰다"] },
    { jp: "泳ぐ", pr: "오요구", meaning: ["헤엄치다", "수영하다"] },
    { jp: "待つ", pr: "마츠", meaning: ["기다리다"] },
    { jp: "作る", pr: "츠쿠루", meaning: ["만들다"] },
    { jp: "貸す", pr: "카스", meaning: ["빌려주다"] },
    { jp: "窓", pr: "마도", meaning: ["창문"] },
    { jp: "開ける", pr: "아케루", meaning: ["열다"] }
];

let currentIndex = 0;
let hint1Shown = false;
let hint2Shown = false;

// 특정 단어가 속한 범주(힌트1용)를 찾는 함수
function getCategory(word) {
    const meanings = word.meaning;
    const isMatched = (keywords) => keywords.some(keyword => meanings.some(m => m.includes(keyword)));

    if (isMatched(["요일"])) return "요일과 관련이 있어요!";
    if (isMatched(["시", "월", "언제", "어제", "오늘", "내일", "매일"])) return "시간이나 날짜와 관련이 있어요!";
    if (isMatched(["덮밥", "미소", "국수", "우동", "초밥", "밥", "떡", "오뎅", "스키야키", "샤브샤브"])) return "음식과 관련이 있어요!";
    if (isMatched(["위", "아래", "안", "옆", "쪽", "뒤"])) return "위치나 방향과 관련이 있어요!";
    if (isMatched(["식", "방학", "문화재", "학교", "도서관", "공원", "영어", "공부", "신문"])) return "학교나 공공생활과 관련이 있어요!";
    if (isMatched(["다", "이"]) && !isMatched(["기다리다", "오래기다리렸습니다"])) return "상태나 성질을 나타내는 말이에요!";
    if (isMatched(["다", "가다", "보다", "하다", "오다", "사다", "쓰다", "만들다", "열다", "놀다", "앉다", "읽다", "듣다", "찍다"])) return "움직임(동사)과 관련이 있어요!";
    if (isMatched(["두근두근", "반짝반짝", "꼬륵꼬륵", "싱글벙글", "웃음"])) return "모양이나 소리를 묘사하는 말이에요!";
    return "일상 대화나 표현과 관련이 있어요!";
}

function loadQuestion() {
    // 셔플 기능을 원할 경우를 대비해 순차 진행 혹은 무작위 설정 가능 (현재는 순차)
    if (currentIndex >= wordList.length) {
        currentIndex = 0; // 한 바퀴 다 돌면 처음부터
    }
    
    const currentWord = wordList[currentIndex];
    document.getElementById("jpWord").innerText = currentWord.jp;
    document.getElementById("pronunciation").innerText = `(${currentWord.pr})`;
    document.getElementById("answerInput").value = "";
    
    // 힌트 및 메시지 초기화
    document.getElementById("messageContainer").innerHTML = "";
    document.getElementById("hintContainer").innerHTML = "";
    hint1Shown = false;
    hint2Shown = false;
    document.getElementById("answerInput").focus();
}

function showHint(type) {
    const currentWord = wordList[currentIndex];
    const hintContainer = document.getElementById("hintContainer");
    
    // 기존에 출력된 힌트 박스 엘리먼트 가져오기
    let h1Box = document.getElementById("hint1-box");
    let h2Box = document.getElementById("hint2-box");

    if (type === 1 && !hint1Shown) {
        const category = getCategory(currentWord);
        hintContainer.insertAdjacentHTML('beforeend', `<div class="hint-box" id="hint1-box">💡 힌트 1: 이 문제는 ${category}</div>`);
        hint1Shown = true;
    } else if (type === 2 && !hint2Shown) {
        // 대표 뜻(첫 번째 뜻) 기준 글자 수 판별
        const primaryMeaning = currentWord.meaning[0];
        if (primaryMeaning.length <= 2) {
            hintContainer.insertAdjacentHTML('beforeend', `<div class="hint-box" id="hint2-box">⚠️ 힌트 2: 단어가 너무 짧아 힌트를 표출할 수 없어요.</div>`);
        } else {
            const firstChar = primaryMeaning.charAt(0);
            hintContainer.insertAdjacentHTML('beforeend', `<div class="hint-box" id="hint2-box">💡 힌트 2: 첫 글자는 '${firstChar}'입니다.</div>`);
        }
        hint2Shown = true;
    }
}

function checkAnswer() {
    const userInput = document.getElementById("answerInput").value.trim().replace(/\s+/g, ''); // 공백 제거 처리
    const currentWord = wordList[currentIndex];
    const messageContainer = document.getElementById("messageContainer");
    
    // 복수 정답 검증 (제시된 의미 리스트 중 하나라도 일치하는지 확인)
    const isCorrect = currentWord.meaning.some(m => m.replace(/\s+/g, '') === userInput);

    if (isCorrect) {
        messageContainer.innerHTML = `<div class="message-box">🎉 정답입니다! 다음 문제로 넘어갑니다.</div>`;
        setTimeout(() => {
            currentIndex++;
            loadQuestion();
        }, 1500);
    } else {
        messageContainer.innerHTML = `<div class="message-box error">❌ 틀렸습니다! 다시 시도해보세요.</div>`;
    }
}

function nextQuestion() {
    currentIndex++;
    loadQuestion();
}

function handleKeyPress(event) {
    if (event.key === "Enter") {
        checkAnswer();
    }
}

// 첫 문제 실행
window.onload = loadQuestion;
</script>

</body>
</html>
