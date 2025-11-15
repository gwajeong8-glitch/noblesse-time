<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>noblesse-time</title>

<style>
    body {
        margin: 0;
        padding: 0;
        background: #000;
        font-family: "KoPub Batang", "Nanum Myeongjo", serif;
        color: #F6EAD2;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    /* 전체 레이아웃 컨테이너 */
    .wrapper {
        position: relative;
        width: 1024px;
        height: 1024px;
        background-image: url('이미지-주소를-여기에-넣기.png');
        background-size: cover;
        background-position: center;
    }

    /* 시·도 선택 */
    .selector-area {
        position: absolute;
        top: 360px;
        width: 100%;
        text-align: center;
    }

    select {
        font-size: 22px;
        padding: 6px 14px;
        margin: 0 10px;
        border-radius: 8px;
        border: 1px solid #F6EAD2;
        background: rgba(0,0,0,0.4);
        color: #F6EAD2;
    }

    /* 하트 아래 시간 텍스트 */
    .time-area {
        position: absolute;
        top: 610px; /* 하트 바로 아래 정확한 위치 */
        width: 100%;
        text-align: center;
        font-size: 28.4px;
        font-weight: 600;
        color: #F6EAD2;
        text-shadow: 0px 0px 10px rgba(0,0,0,0.6);
    }
</style>
</head>
<body>

<div class="wrapper">

    <!-- 지역 선택 UI -->
    <div class="selector-area">
        <select id="sido"></select>
        <select id="sigungu"></select>
    </div>

    <!-- 시간 표시 영역 -->
    <div class="time-area" id="timeDisplay">시간 로딩중...</div>

</div>

<script>
/* ---------------------------
   17개 시·도 데이터
-------------------------------*/
const regionData = {
    "서울특별시": ["종로구","중구","용산구","성동구","광진구","동대문구","중랑구","성북구","강북구","도봉구","노원구","은평구","서대문구","마포구","양천구","강서구","구로구","금천구","영등포구","동작구","관악구","서초구","강남구","송파구","강동구"],

    "부산광역시": ["중구","서구","동구","영도구","부산진구","동래구","남구","북구","해운대구","사하구","금정구","강서구","연제구","수영구","사상구","기장군"],

    "대구광역시": ["중구","동구","서구","남구","북구","수성구","달서구","달성군"],

    "인천광역시": ["중구","동구","미추홀구","연수구","남동구","부평구","계양구","서구","강화군","옹진군"],

    "광주광역시": ["동구","서구","남구","북구","광산구"],

    "대전광역시": ["동구","중구","서구","유성구","대덕구"],

    "울산광역시": ["중구","남구","동구","북구","울주군"],

    "세종특별자치시": ["세종시"],

    "경기도": ["수원시","성남시","의정부시","안양시","부천시","광명시","평택시","동두천시","안산시","고양시","과천시","구리시","남양주시","오산시","시흥시","군포시","의왕시","하남시","용인시","파주시","이천시","안성시","김포시","화성시","광주시","양주시","포천시","여주시"],

    "강원도": ["춘천시","원주시","강릉시","동해시","태백시","속초시","삼척시","홍천군","횡성군","영월군","평창군","정선군","철원군","화천군","양구군","인제군","고성군","양양군"],

    "충청북도": ["청주시","충주시","제천시","보은군","옥천군","영동군","증평군","진천군","괴산군","음성군","단양군"],

    "충청남도": ["천안시","공주시","보령시","아산시","서산시","논산시","계룡시","당진시","금산군","부여군","서천군","청양군","홍성군","예산군","태안군"],

    "전라북도": ["전주시","군산시","익산시","정읍시","남원시","김제시","완주군","진안군","무주군","장수군","임실군","순창군","고창군","부안군"],

    "전라남도": ["목포시","여수시","순천시","나주시","광양시","담양군","곡성군","구례군","고흥군","보성군","화순군","장흥군","강진군","해남군","영암군","무안군","함평군","영광군","장성군","완도군","진도군","신안군"],

    "경상북도": ["포항시","경주시","김천시","안동시","구미시","영주시","영천시","상주시","문경시","경산시","의성군","청송군","영양군","영덕군","청도군","고령군","성주군","칠곡군","예천군","봉화군","울진군","울릉군"],

    "경상남도": ["창원시","진주시","통영시","사천시","김해시","밀양시","거제시","양산시","의령군","함안군","창녕군","고성군","남해군","하동군","산청군","함양군","거창군","합천군"],

    "제주특별자치도": ["제주시","서귀포시"]
};

/* ---------------------------
   시·도/시·군·구 선택 출력
---------------------------*/
const sidoSelect = document.getElementById("sido");
const sigunguSelect = document.getElementById("sigungu");

for (let sido in regionData) {
    let option = document.createElement("option");
    option.value = sido;
    option.textContent = sido;
    sidoSelect.appendChild(option);
}

function updateSigungu() {
    sigunguSelect.innerHTML = "";
    const list = regionData[sidoSelect.value];
    list.forEach(name => {
        let opt = document.createElement("option");
        opt.value = name;
        opt.textContent = name;
        sigunguSelect.appendChild(opt);
    });
}
sidoSelect.addEventListener("change", updateSigungu);
updateSigungu();

/* ---------------------------
   시간 자동 표시
---------------------------*/
function updateTime() {
    const now = new Date();
    const year = now.getFullYear();
    const month = now.getMonth() + 1;
    const date = now.getDate();
    const hour = now.getHours();
    const minute = now.getMinutes().toString().padStart(2, "0");

    document.getElementById("timeDisplay").innerText =
        `${year}년 ${month}월 ${date}일 ${hour}시 ${minute}분`;
}
setInterval(updateTime, 1000);
updateTime();
</script>

</body>
</html>
