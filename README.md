<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SSIS G4-G12 주간 급식 메뉴</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Noto Sans KR", sans-serif;
      background: #f5f7fa;
      color: #222;
      line-height: 1.5;
      padding: 20px;
    }
    .container { max-width: 1100px; margin: 0 auto; }
    header {
      background: linear-gradient(135deg, #1a5f7a, #159895);
      color: white;
      padding: 28px 24px;
      border-radius: 16px;
      margin-bottom: 24px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.1);
      text-align: center;
    }
    header h1 { font-size: 1.5rem; margin-bottom: 6px; }
    header p { opacity: 0.9; font-size: 0.95rem; margin-bottom: 18px; }

    .refresh-btn {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      background: #e67e22;
      color: white;
      border: none;
      padding: 14px 26px;
      border-radius: 12px;
      font-size: 1.05rem;
      font-weight: 700;
      cursor: pointer;
      box-shadow: 0 4px 12px rgba(230, 126, 34, 0.35);
      transition: transform 0.2s, background 0.2s;
    }
    .refresh-btn:hover {
      background: #d35400;
      transform: translateY(-2px);
    }
    .refresh-btn:active {
      transform: translateY(0);
    }

    .favorites-box {
      background: white;
      border-radius: 14px;
      padding: 20px 24px;
      margin-bottom: 24px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.06);
      border-left: 5px solid #e67e22;
    }
    .favorites-box h2 { font-size: 1.05rem; margin-bottom: 4px; color: #1a5f7a; }
    .favorites-box .hint { font-size: 0.8rem; color: #888; margin-bottom: 12px; }
    .fav-input-row {
      display: flex;
      gap: 8px;
      margin-bottom: 14px;
      flex-wrap: wrap;
    }
    .fav-input-row input {
      flex: 1;
      min-width: 160px;
      padding: 10px 12px;
      border: 1px solid #ddd;
      border-radius: 8px;
      font-size: 0.9rem;
    }
    .fav-input-row button {
      background: #e67e22;
      color: white;
      border: none;
      padding: 10px 18px;
      border-radius: 8px;
      font-weight: 600;
      cursor: pointer;
    }
    .fav-input-row button:hover { background: #d35400; }
    #fav-list { display: flex; flex-wrap: wrap; gap: 8px; }
    .fav-chip {
      background: #fff4e5;
      border: 1px solid #ffcf8a;
      color: #7a5c00;
      padding: 6px 10px;
      border-radius: 20px;
      font-size: 0.85rem;
      display: inline-flex;
      align-items: center;
      gap: 6px;
    }
    .fav-chip button {
      background: none;
      border: none;
      color: #a05a00;
      cursor: pointer;
      font-size: 1rem;
      line-height: 1;
      padding: 0;
    }
    .star-badge { color: #e67e22; }

    /* 오늘 카드 강조 */
    .day-card.today { border: 3px solid #e67e22; }
    .today-badge {
      background: #e67e22;
      color: white;
      font-size: 0.68rem;
      font-weight: 700;
      padding: 2px 8px;
      border-radius: 10px;
      margin-left: 8px;
      vertical-align: middle;
    }

    /* 지출 계산기 */
    .calc-checkbox {
      margin-right: 6px;
      transform: scale(1.05);
      vertical-align: middle;
      cursor: pointer;
    }
    .station.calc-bundle .station-name {
      display: flex;
      align-items: center;
      cursor: pointer;
    }
    .dish.calc-item {
      cursor: pointer;
      display: flex;
      align-items: center;
    }
    .day-total {
      margin-top: 12px;
      padding-top: 10px;
      border-top: 1px dashed #ccc;
      font-weight: 700;
      color: #1a5f7a;
      font-size: 0.9rem;
      text-align: right;
    }
    .day-total.today-total {
      color: #e67e22;
    }
    .reset-calc-btn {
      background: none;
      border: 1px solid #ccc;
      color: #888;
      font-size: 0.7rem;
      padding: 2px 8px;
      border-radius: 8px;
      margin-left: 8px;
      cursor: pointer;
    }
    .reset-calc-btn:hover { background: #f0f0f0; }

    .days {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 16px;
      margin-bottom: 28px;
    }
    .day-card {
      background: white;
      border-radius: 14px;
      overflow: hidden;
      box-shadow: 0 2px 10px rgba(0,0,0,0.06);
    }
    .day-header {
      background: #1a5f7a;
      color: white;
      padding: 12px 16px;
      font-weight: 600;
      font-size: 1rem;
    }
    .day-body { padding: 14px 16px; font-size: 0.9rem; }
    .station {
      margin-bottom: 12px;
      padding-bottom: 10px;
      border-bottom: 1px dashed #e0e0e0;
    }
    .station:last-child { border-bottom: none; margin-bottom: 0; }
    .station-name {
      font-size: 0.75rem;
      font-weight: 700;
      color: #159895;
      letter-spacing: 0.3px;
      margin-bottom: 4px;
    }
    .dish { margin: 2px 0; }
    .dish.no-menu { color: #aaa; font-style: italic; }
    .price { color: #e67e22; font-weight: 600; font-size: 0.85rem; }

    footer {
      text-align: center;
      margin-top: 30px;
      font-size: 0.85rem;
      color: #888;
    }
    a { color: #1a5f7a; }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>SSIS G4–G12 주간 급식 메뉴</h1>
      <p>Suzhou Singapore International School · 2층<br>
      <!-- AUTO:DATE_RANGE:START -->2026년 9월 14일 (월) ~ 9월 18일 (금)<!-- AUTO:DATE_RANGE:END --></p>
      <p id="last-updated" style="font-size:0.75rem;opacity:0.75;margin-top:4px;">
        <!-- AUTO:LAST_UPDATED:START -->마지막 자동 업데이트: 2026년 9월 13일 (SSIS 공식 메뉴 PDF 기준)<!-- AUTO:LAST_UPDATED:END -->
      </p>

      <!-- 새로고침 버튼 -->
      <button class="refresh-btn" onclick="openLatestMenu()">
        🔄 최신 메뉴 새로고침 (PDF 열기)
      </button>
    </header>

    <div class="favorites-box">
      <h2>⭐ 즐겨찾는 메뉴</h2>
      <p class="hint">좋아하는 요리 이름(일부만 입력해도 OK, 예: "라자냐")을 등록하면, 그 요리가 나오는 날 메뉴 옆에 ⭐가 자동으로 붙어요. 이 기기에만 저장돼요.</p>
      <div class="fav-input-row">
        <input type="text" id="fav-input" placeholder="예: 탄두리, 라자냐, 딤섬...">
        <button id="fav-add-btn">추가</button>
      </div>
      <div id="fav-list"></div>
    </div>

    <div class="days">
      <!-- AUTO:DAYS:START -->
      <div class="day-card">
        <div class="day-header">월요일 (9/14)</div>
        <div class="day-body">
          <div class="station">
            <div class="station-name">셰프 스페셜</div>
            <div class="dish">한국식 소고기 카레</div>
            <div class="dish">일본식 볶음밥</div>
            <div class="dish">베이컨 강낭콩볶음</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">채식</div>
            <div class="dish">감자튀김</div>
            <div class="dish">마사만소스 계란튀김</div>
            <div class="dish">빵(선택)</div>
            <div class="dish">믹스 샐러드</div>
            <div class="dish">토마토 수프</div>
            <div class="price">¥32</div>
          </div>
          <div class="station">
            <div class="station-name">뚝배기</div>
            <div class="dish">대만식 루로우판(돼지고기 덮밥)</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">싸이드</div>
            <div class="dish">메뉴없음</div>
          </div>
          <div class="station">
            <div class="station-name">가성비</div>
            <div class="dish">매콤 소고기찜(수이주뉴러우) ¥15</div>
            <div class="dish">차슈소스 구운 오리다리 ¥12</div>
            <div class="dish">가정식 두부 돼지고기볶음 ¥9</div>
            <div class="dish">셀러리 돼지고기채볶음 ¥9</div>
            <div class="dish">간장조림 꽈리고추(호피고추) ¥5</div>
            <div class="dish">항저우 배추 ¥5</div>
          </div>
          <div class="station">
            <div class="station-name">면 & 딤섬</div>
            <div class="dish">토마토계란면(닭날개 토핑) ¥33</div>
            <div class="dish">샤오롱바오 ¥10</div>
            <div class="dish">과일컵 ¥9</div>
          </div>
          <div class="station">
            <div class="station-name">샌드위치</div>
            <div class="dish">메뉴없음</div>
          </div>
        </div>
      </div>

      <div class="day-card">
        <div class="day-header">화요일 (9/15)</div>
        <div class="day-body">
          <div class="station">
            <div class="station-name">셰프 스페셜</div>
            <div class="dish">한국식 프라이드치킨</div>
            <div class="dish">한국식 떡볶이</div>
            <div class="dish">마늘 완두콩 옥수수볶음</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">채식</div>
            <div class="dish">볶음 쌀국수(미펀)</div>
            <div class="dish">후무스 피타빵</div>
            <div class="dish">믹스 샐러드</div>
            <div class="dish">미네스트로네 수프</div>
            <div class="price">¥32</div>
          </div>
          <div class="station">
            <div class="station-name">뚝배기</div>
            <div class="dish">소고기 케사디아</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">싸이드</div>
            <div class="dish">구운 치킨윙 ¥12</div>
            <div class="dish">딸기무스컵 ¥12</div>
          </div>
          <div class="station">
            <div class="station-name">가성비</div>
            <div class="dish">건두부 돼지고기조림 ¥15</div>
            <div class="dish">후추소금 생선튀김 ¥11</div>
            <div class="dish">당근 목이버섯 아스파라거스상추 돼지고기채볶음 ¥9</div>
            <div class="dish">나물(地衣) 계란볶음 ¥9</div>
            <div class="dish">무말랭이 완두콩볶음 ¥5</div>
            <div class="dish">공심채(모닝글로리) ¥5</div>
          </div>
          <div class="station">
            <div class="station-name">면 & 딤섬</div>
            <div class="dish">태국식 돼지고기 쌀국수(미셴) ¥34</div>
            <div class="dish">과일컵 ¥9</div>
          </div>
          <div class="station">
            <div class="station-name">샌드위치</div>
            <div class="dish">메뉴없음</div>
          </div>
        </div>
      </div>

      <div class="day-card">
        <div class="day-header">수요일 (9/16)</div>
        <div class="day-body">
          <div class="station">
            <div class="station-name">셰프 스페셜</div>
            <div class="dish">코코넛 해산물찜</div>
            <div class="dish">파스타</div>
            <div class="dish">미니 구운 감자</div>
            <div class="dish">브로콜리&당근</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">채식</div>
            <div class="dish">콩 캐서롤</div>
            <div class="dish">구운 애호박&가지</div>
            <div class="dish">빵(선택)</div>
            <div class="dish">믹스 샐러드</div>
            <div class="dish">호박 수프</div>
            <div class="price">¥32</div>
          </div>
          <div class="station">
            <div class="station-name">뚝배기</div>
            <div class="dish">일본식 카레 돈카츠덮밥</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">싸이드</div>
            <div class="dish">메뉴없음</div>
          </div>
          <div class="station">
            <div class="station-name">가성비</div>
            <div class="dish">후추소금 새우튀김 ¥14</div>
            <div class="dish">대만식 삼배계(산베이지) ¥12</div>
            <div class="dish">건과 콜리플라워 돼지고기볶음 ¥9</div>
            <div class="dish">차수고버섯 청피망 돼지고기채볶음 ¥9</div>
            <div class="dish">숙주나물볶음 ¥5</div>
            <div class="dish">상추(잎채소) ¥5</div>
          </div>
          <div class="station">
            <div class="station-name">면 & 딤섬</div>
            <div class="dish">수제 완탕 ¥35</div>
            <div class="dish">과일컵 ¥9</div>
          </div>
          <div class="station">
            <div class="station-name">샌드위치</div>
            <div class="dish">메뉴없음</div>
          </div>
        </div>
      </div>

      <div class="day-card">
        <div class="day-header">목요일 (9/17)</div>
        <div class="day-body">
          <div class="station">
            <div class="station-name">셰프 스페셜</div>
            <div class="dish">구운 폭립</div>
            <div class="dish">치즈 구운 단호박</div>
            <div class="dish">혼합 채소</div>
            <div class="dish">또르띠아 튀김</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">채식</div>
            <div class="dish">계란 채운 구운 토마토</div>
            <div class="dish">채소 볶음밥</div>
            <div class="dish">빵(선택)</div>
            <div class="dish">믹스 샐러드</div>
            <div class="dish">감자 수프</div>
            <div class="price">¥32</div>
          </div>
          <div class="station">
            <div class="station-name">뚝배기</div>
            <div class="dish">수타 탄두리치킨 피자</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">싸이드</div>
            <div class="dish">도넛 ¥8</div>
            <div class="dish">해쉬브라운 ¥9</div>
          </div>
          <div class="station">
            <div class="station-name">가성비</div>
            <div class="dish">바삭 튀긴 돼지등심 ¥14</div>
            <div class="dish">토란 오리조림 ¥12</div>
            <div class="dish">피망 죽순 닭고기볶음 ¥9</div>
            <div class="dish">목수육(계란 목이버섯 돼지고기볶음) ¥9</div>
            <div class="dish">카레 감자 ¥5</div>
            <div class="dish">광둥 유채심(차이신) ¥5</div>
          </div>
          <div class="station">
            <div class="station-name">면 & 딤섬</div>
            <div class="dish">홍샤오 소고기면 ¥35</div>
            <div class="dish">과일컵 ¥9</div>
          </div>
          <div class="station">
            <div class="station-name">샌드위치</div>
            <div class="dish">메뉴없음</div>
          </div>
        </div>
      </div>

      <div class="day-card">
        <div class="day-header">금요일 (9/18)</div>
        <div class="day-body">
          <div class="station">
            <div class="station-name">셰프 스페셜</div>
            <div class="dish">치킨 코르동블루</div>
            <div class="dish">토마토크림소스 파스타</div>
            <div class="dish">브로콜리&콜리플라워</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">채식</div>
            <div class="dish">치즈 고구마 매쉬</div>
            <div class="dish">인도식 콩카레(투르달)</div>
            <div class="dish">빵(선택)</div>
            <div class="dish">믹스 샐러드</div>
            <div class="dish">시금치 수프</div>
            <div class="price">¥32</div>
          </div>
          <div class="station">
            <div class="station-name">뚝배기</div>
            <div class="dish">매콤 소고기 당면탕</div>
            <div class="dish">흑미밥</div>
            <div class="price">¥35</div>
          </div>
          <div class="station">
            <div class="station-name">싸이드</div>
            <div class="dish">메뉴없음</div>
          </div>
          <div class="station">
            <div class="station-name">가성비</div>
            <div class="dish">동충하화 닭찜 ¥13</div>
            <div class="dish">홍샤오 고기완자 ¥15</div>
            <div class="dish">흑후추 굴소스 오리고기볶음 ¥9</div>
            <div class="dish">감자 돼지고기채볶음 ¥9</div>
            <div class="dish">연근 모둠채소볶음(허탕샤오차오) ¥5</div>
            <div class="dish">청경채 ¥5</div>
          </div>
          <div class="station">
            <div class="station-name">면 & 딤섬</div>
            <div class="dish">파기름 간장 비빔면</div>
            <div class="dish">표고버섯죽순 돼지고기볶음</div>
            <div class="price">¥33</div>
            <div class="dish">마리네이드 계란 & 샤오마이 ¥10</div>
            <div class="dish">과일컵 ¥9</div>
          </div>
          <div class="station">
            <div class="station-name">샌드위치</div>
            <div class="dish">모듬 샌드위치 ¥33</div>
          </div>
        </div>
      </div>

      <!-- AUTO:DAYS:END -->
    </div>

    <footer>
      데이터 출처: SSIS Official Menu PDF · <!-- AUTO:SOURCE_LABEL:START -->Week 1 (14 Sep – 18 Sep 2026)<!-- AUTO:SOURCE_LABEL:END -->
    </footer>
  </div>

  <script>
    function openLatestMenu() {
      // 학교 공식 최신 메뉴 PDF 주소 (매주 같은 주소로 업데이트됨)
      const pdfUrl = "https://www.suzhousinternationalschool.com/uploaded/file/Menu/SSIS_G4-G12_menu.pdf";
      
      // 새 탭에서 열기
      window.open(pdfUrl, "_blank");
    }

    // ---- 즐겨찾기 기능 ----
    const FAV_KEY = "ssis_menu_favorites";

    function loadFavorites() {
      try {
        const raw = localStorage.getItem(FAV_KEY);
        return raw ? JSON.parse(raw) : [];
      } catch (e) {
        return [];
      }
    }

    function saveFavorites(list) {
      localStorage.setItem(FAV_KEY, JSON.stringify(list));
    }

    function escapeHtml(str) {
      return String(str)
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;");
    }

    function renderFavList() {
      const list = loadFavorites();
      const container = document.getElementById("fav-list");
      container.innerHTML = "";

      if (list.length === 0) {
        container.innerHTML = '<p style="font-size:0.85rem;color:#888;">아직 등록한 즐겨찾기가 없어요.</p>';
        return;
      }

      list.forEach((name, idx) => {
        const chip = document.createElement("span");
        chip.className = "fav-chip";
        chip.innerHTML = `⭐ ${escapeHtml(name)} <button aria-label="삭제" data-idx="${idx}">×</button>`;
        container.appendChild(chip);
      });

      container.querySelectorAll("button[data-idx]").forEach((btn) => {
        btn.addEventListener("click", () => {
          const idx = parseInt(btn.getAttribute("data-idx"), 10);
          const list = loadFavorites();
          list.splice(idx, 1);
          saveFavorites(list);
          renderFavList();
          applyFavoriteStars();
        });
      });
    }

    function addFavorite() {
      const input = document.getElementById("fav-input");
      const val = input.value.trim();
      if (!val) return;

      const list = loadFavorites();
      const alreadyExists = list.some(
        (f) => f.toLowerCase() === val.toLowerCase()
      );
      if (!alreadyExists) {
        list.push(val);
        saveFavorites(list);
      }
      input.value = "";
      renderFavList();
      applyFavoriteStars();
    }

    function applyFavoriteStars() {
      const favorites = loadFavorites()
        .map((f) => f.toLowerCase())
        .filter(Boolean);

      document.querySelectorAll(".dish").forEach((dishEl) => {
        // 기존에 붙어있던 별표 제거 (중복 방지)
        const existing = dishEl.querySelector(".star-badge");
        if (existing) existing.remove();

        const text = dishEl.textContent.toLowerCase();
        const isFav = favorites.some((f) => text.includes(f));

        if (isFav) {
          const star = document.createElement("span");
          star.className = "star-badge";
          star.textContent = " ⭐";
          dishEl.appendChild(star);
        }
      });
    }

    document.addEventListener("DOMContentLoaded", () => {
      renderFavList();
      applyFavoriteStars();

      // "메뉴없음" 텍스트에 회색/이탤릭 스타일 적용
      document.querySelectorAll(".dish").forEach((dishEl) => {
        if (dishEl.textContent.trim() === "메뉴없음") {
          dishEl.classList.add("no-menu");
        }
      });

      setupTodayHighlight();
      setupSpendingCalculator();

      document.getElementById("fav-add-btn").addEventListener("click", addFavorite);
      document.getElementById("fav-input").addEventListener("keydown", (e) => {
        // 한글/일본어/중국어 등 IME로 글자를 조합하는 중에 Enter를 누르면
        // 아직 완성되지 않은 글자가 먼저 제출되어 버리는 문제를 방지
        if (e.isComposing || e.keyCode === 229) return;

        if (e.key === "Enter") {
          e.preventDefault();
          addFavorite();
        }
      });
    });

    // ---- 오늘 카드 자동 강조 ----
    function setupTodayHighlight() {
      const jsDay = new Date().getDay(); // 0=일 1=월 ... 6=토
      const dayIndex = jsDay - 1; // 월=0, 화=1, 수=2, 목=3, 금=4
      const cards = document.querySelectorAll(".day-card");
      if (dayIndex < 0 || dayIndex > 4) return; // 주말이면 강조 없음

      const todayCard = cards[dayIndex];
      if (!todayCard) return;

      todayCard.classList.add("today");
      const header = todayCard.querySelector(".day-header");
      if (header && !header.querySelector(".today-badge")) {
        const badge = document.createElement("span");
        badge.className = "today-badge";
        badge.textContent = "오늘";
        header.appendChild(badge);
      }
    }

    // ---- 오늘 총 예상 지출 계산기 ----
    function extractPrice(text) {
      const match = text.match(/¥\s?(\d+(?:\.\d+)?)/);
      return match ? parseFloat(match[1]) : null;
    }

    function setupSpendingCalculator() {
      document.querySelectorAll(".day-card").forEach((card) => {
        const stations = card.querySelectorAll(".station");

        stations.forEach((station) => {
          const priceEl = station.querySelector(".price");

          if (priceEl) {
            // 세트(콤보) 가격: 코너 전체가 한 묶음
            const price = extractPrice(priceEl.textContent);
            if (price === null) return;

            station.classList.add("calc-bundle");
            const nameEl = station.querySelector(".station-name");

            const checkbox = document.createElement("input");
            checkbox.type = "checkbox";
            checkbox.className = "calc-checkbox";
            checkbox.dataset.price = price;

            nameEl.prepend(checkbox);

            checkbox.addEventListener("change", () => updateDayTotal(card));
            nameEl.addEventListener("click", (e) => {
              if (e.target !== checkbox) {
                checkbox.checked = !checkbox.checked;
                updateDayTotal(card);
              }
            });
          } else {
            // 개별 가격: 요리마다 따로 체크
            const dishes = station.querySelectorAll(".dish");
            dishes.forEach((dish) => {
              if (dish.classList.contains("no-menu")) return;
              const price = extractPrice(dish.textContent);
              if (price === null) return;

              dish.classList.add("calc-item");
              const checkbox = document.createElement("input");
              checkbox.type = "checkbox";
              checkbox.className = "calc-checkbox";
              checkbox.dataset.price = price;

              dish.prepend(checkbox);

              checkbox.addEventListener("change", () => updateDayTotal(card));
              dish.addEventListener("click", (e) => {
                if (e.target !== checkbox) {
                  checkbox.checked = !checkbox.checked;
                  updateDayTotal(card);
                }
              });
            });
          }
        });

        // 합계 표시 영역 + 초기화 버튼
        const dayBody = card.querySelector(".day-body");
        const totalRow = document.createElement("div");
        totalRow.className = "day-total";
        if (card.classList.contains("today")) totalRow.classList.add("today-total");
        totalRow.innerHTML =
          '<span class="total-label">합계: ¥0</span>' +
          '<button type="button" class="reset-calc-btn">초기화</button>';
        dayBody.appendChild(totalRow);

        totalRow.querySelector(".reset-calc-btn").addEventListener("click", () => {
          card.querySelectorAll(".calc-checkbox").forEach((cb) => (cb.checked = false));
          updateDayTotal(card);
        });

        updateDayTotal(card); // 초기 라벨(오늘/합계) 표시
      });
    }

    function updateDayTotal(card) {
      let sum = 0;
      card.querySelectorAll(".calc-checkbox:checked").forEach((cb) => {
        sum += parseFloat(cb.dataset.price) || 0;
      });
      const label = card.querySelector(".total-label");
      const prefix = card.classList.contains("today") ? "오늘 예상 지출: " : "합계: ";
      if (label) label.textContent = `${prefix}¥${sum}`;
    }
  </script>
</body>
</html>
