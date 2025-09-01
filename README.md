<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>봉사 공지/신청 사이트 – 프로토타입 (localStorage)</title>
  <style>
    :root {
      --bg: #f5f7fb;
      --card: #ffffff;
      --text: #111827;
      --muted: #6b7280;
      --primary: #2563eb;
      --primary-d: #1e40af;
      --danger: #dc2626;
      --success: #16a34a;
      --border: #e5e7eb;
      --shadow: 0 10px 25px rgba(0,0,0,0.07);
      --radius: 14px;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Noto Sans KR", Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji";
      color: var(--text);
      background: linear-gradient(180deg, #eef2ff, #f9fafb 40%);
      min-height: 100vh;
    }
    a { color: var(--primary); text-decoration: none; }
    .container { max-width: 1000px; margin: 32px auto; padding: 0 16px; }
    .card {
      background: var(--card);
      border: 1px solid var(--border);
      box-shadow: var(--shadow);
      border-radius: var(--radius);
    }
    header.topbar {
      position: sticky; top: 0; z-index: 30;
      backdrop-filter: saturate(1.2) blur(6px);
      background: rgba(255,255,255,0.85);
      border-bottom: 1px solid var(--border);
    }
    .topbar-inner { display:flex; align-items:center; justify-content:space-between; gap: 12px; padding: 12px 16px; }
    .brand { display:flex; align-items:center; gap: 10px; font-weight: 800; letter-spacing: -0.2px; }
    .brand .logo { width: 32px; height: 32px; border-radius: 8px; background: linear-gradient(135deg,#3b82f6,#a78bfa); box-shadow: inset 0 0 0 2px #fff; }
    .toolbar { display:flex; align-items:center; gap: 8px; }
    .btn {
      display:inline-flex; align-items:center; justify-content:center; gap:8px;
      padding: 10px 14px; border-radius: 10px; border: 1px solid var(--border);
      background: #fff; cursor:pointer; font-weight: 600; transition: .15s ease;
    }
    .btn:hover { transform: translateY(-1px); box-shadow: 0 4px 14px rgba(0,0,0,0.08); }
    .btn.primary { background: var(--primary); color: #fff; border-color: transparent; }
    .btn.primary:hover { background: var(--primary-d); }
    .btn.danger { background: #fff5f5; color: var(--danger); border-color: #fecaca; }
    .btn.ghost { background: transparent; }

    .grid { display:grid; gap:16px; }
    .grid.cols-2 { grid-template-columns: repeat(2, minmax(0,1fr)); }
    @media (max-width: 820px) { .grid.cols-2 { grid-template-columns: 1fr; } }

    .section { margin-top: 20px; }
    .section-title { padding: 16px 18px; border-bottom: 1px solid var(--border); font-weight: 800; }

    .pad { padding: 18px; }

    .muted { color: var(--muted); }

    .input, textarea {
      width: 100%; padding: 12px 14px; border: 1px solid var(--border);
      border-radius: 10px; outline: none; background: #fff; font-size: 15px;
    }
    .input:focus, textarea:focus { border-color: #c7d2fe; box-shadow: 0 0 0 3px #e0e7ff; }

    .notice-card { padding:16px; border-top:1px solid var(--border); display:grid; gap:8px; }
    .notice-header { display:flex; align-items:center; justify-content:space-between; gap:12px; }
    .badge { font-size: 12px; padding: 4px 8px; border-radius: 20px; border: 1px solid var(--border); background:#f8fafc; }
    .badge.good { color: var(--success); border-color:#bbf7d0; background:#f0fdf4; }
    .badge.warn { color: #b45309; border-color:#fed7aa; background:#fffbeb; }

    .empty { text-align:center; padding: 32px; color: var(--muted); }

    /* Auth layout */
    .auth-wrap { display:grid; grid-template-columns: 1.1fr 0.9fr; gap:16px; }
    @media (max-width: 900px) { .auth-wrap { grid-template-columns: 1fr; } }

    .tabs { display:flex; gap:6px; padding: 8px; background:#f3f4f6; border-radius: 10px; }
    .tab { flex:1; text-align:center; padding:10px; border-radius: 8px; cursor:pointer; font-weight:700; }
    .tab.active { background:#fff; border:1px solid var(--border); }

    .table { width:100%; border-collapse: collapse; }
    .table th, .table td { padding: 10px 8px; border-bottom: 1px solid var(--border); text-align:left; }
    .table th { background:#f9fafb; font-weight: 700; }

    .pill { display:inline-flex; align-items:center; gap:6px; padding:4px 10px; border-radius:999px; border:1px solid var(--border); background:#fff; font-size: 12px; }

    .kicker { font-size: 12px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); }
  </style>
</head>
<body>
  <!-- Topbar (role별로 버튼 다르게 표시) -->
  <header class="topbar">
    <div class="topbar-inner container">
      <div class="brand">
        <div class="logo" aria-hidden="true"></div>
        <div>
          <div>School Service Hub</div>
          <div class="kicker">봉사 공지 · 신청 관리</div>
        </div>
      </div>
      <div class="toolbar" id="toolbar"></div>
    </div>
  </header>

  <main class="container">
    <!-- Auth Section -->
    <section id="authSection" class="section">
      <div class="grid cols-2">
        <div class="card">
          <div class="section-title">로그인</div>
          <div class="pad">
            <div class="tabs" role="tablist">
              <div class="tab active" id="tabLoginStudent">학생</div>
              <div class="tab" id="tabLoginTeacher">선생님</div>
            </div>
            <div id="loginStudent" class="section" aria-labelledby="tabLoginStudent">
              <div class="grid">
                <input class="input" id="loginStudentId" placeholder="학번" />
                <input class="input" id="loginStudentPw" type="password" placeholder="비밀번호" />
                <button class="btn primary" id="btnLoginStudent">학생 로그인</button>
              </div>
            </div>
            <div id="loginTeacher" class="section" aria-labelledby="tabLoginTeacher" style="display:none;">
              <div class="grid">
                <input class="input" id="loginTeacherEmail" placeholder="이메일" type="email" />
                <input class="input" id="loginTeacherPw" type="password" placeholder="비밀번호" />
                <input class="input" id="loginTeacherSecret" type="password" placeholder="교사용 비밀코드" />
                <button class="btn primary" id="btnLoginTeacher">선생님 로그인</button>
              </div>
              <div class="muted" style="margin-top:8px; font-size:12px;">비밀코드는 학교에서 배포된 코드로만 로그인 가능합니다.</div>
            </div>
          </div>
        </div>

        <div class="card">
          <div class="section-title">회원가입</div>
          <div class="pad">
            <div class="tabs" role="tablist">
              <div class="tab active" id="tabJoinStudent">학생</div>
              <div class="tab" id="tabJoinTeacher">선생님</div>
            </div>
            <div id="joinStudent" class="section">
              <div class="grid">
                <input class="input" id="joinSName" placeholder="이름" />
                <input class="input" id="joinSId" placeholder="학번 (로그인 ID)" />
                <input class="input" id="joinSRoom" placeholder="자습실 번호" />
                <input class="input" id="joinSPw" type="password" placeholder="비밀번호" />
                <button class="btn primary" id="btnJoinStudent">학생 회원가입</button>
              </div>
            </div>
            <div id="joinTeacher" class="section" style="display:none;">
              <div class="grid">
                <input class="input" id="joinTName" placeholder="이름" />
                <input class="input" id="joinTEmail" placeholder="이메일 (로그인 ID)" type="email" />
                <input class="input" id="joinTPw" type="password" placeholder="비밀번호" />
                <input class="input" id="joinTSecret" type="password" placeholder="교사용 비밀코드" />
                <button class="btn primary" id="btnJoinTeacher">선생님 회원가입</button>
              </div>
              <div class="muted" style="margin-top:8px; font-size:12px;">비밀코드가 일치해야 교사용 계정이 생성됩니다.</div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Student Dashboard -->
    <section id="studentSection" class="section" style="display:none;">
      <div class="card">
        <div class="section-title">공지 목록</div>
        <div id="studentNoticeList"></div>
      </div>

      <div class="card" style="margin-top:16px;">
        <div class="section-title">내 신청 내역</div>
        <div id="studentMyApplies" class="pad"></div>
      </div>
    </section>

    <!-- Teacher Dashboard -->
    <section id="teacherSection" class="section" style="display:none;">
      <div class="card">
        <div class="section-title">공지 올리기</div>
        <div class="pad grid">
          <input class="input" id="noticeTitle" placeholder="제목" />
          <textarea class="input" id="noticeContent" placeholder="내용" rows="5"></textarea>
          <div class="grid" style="grid-template-columns: 1fr 1fr; gap:12px;">
            <input class="input" id="noticeLimit" type="number" placeholder="신청 인원 제한 (선택)" min="1" />
            <input class="input" id="noticeTag" placeholder="태그 (예: 봉사, 환경, 도서)" />
          </div>
          <button class="btn primary" id="btnPublish">공지 등록</button>
        </div>
      </div>

      <div class="card" style="margin-top:16px;">
        <div class="section-title">등록된 공지</div>
        <div id="teacherNoticeList"></div>
      </div>
    </section>

    <!-- Applicants Modal -->
    <div id="modal" style="display:none; position:fixed; inset:0; background: rgba(0,0,0,0.35); z-index:50; align-items:center; justify-content:center;">
      <div class="card" style="width: min(880px, 92vw); max-height: 82vh; overflow:auto;">
        <div class="section-title" id="modalTitle">신청자 목록</div>
        <div class="pad">
          <div id="applicantsTableWrap"></div>
          <div style="display:flex; justify-content:flex-end; margin-top:12px;">
            <button class="btn" id="btnCloseModal">닫기</button>
          </div>
        </div>
      </div>
    </div>
  </main>

  <script>
    /*************************
     * Config
     *************************/
    const TEACHER_SECRET = "teacher2025"; // ← 교사용 비밀코드 (배포 전에 변경하세요)

    /*************************
     * LocalStorage helpers
     *************************/
    const LS_KEYS = {
      USERS: 'vna_users',
      NOTICES: 'vna_notices',
      CURRENT: 'vna_currentUser'
    };

    function loadUsers() {
      const raw = localStorage.getItem(LS_KEYS.USERS);
      return raw ? JSON.parse(raw) : { students: {}, teachers: {} };
    }
    function saveUsers(users) {
      localStorage.setItem(LS_KEYS.USERS, JSON.stringify(users));
    }

    function loadNotices() {
      const raw = localStorage.getItem(LS_KEYS.NOTICES);
      return raw ? JSON.parse(raw) : [];
    }
    function saveNotices(notices) {
      localStorage.setItem(LS_KEYS.NOTICES, JSON.stringify(notices));
    }

    function loadCurrent() {
      const raw = localStorage.getItem(LS_KEYS.CURRENT);
      return raw ? JSON.parse(raw) : null;
    }
    function saveCurrent(obj) {
      localStorage.setItem(LS_KEYS.CURRENT, JSON.stringify(obj));
    }
    function clearCurrent() {
      localStorage.removeItem(LS_KEYS.CURRENT);
    }

    function uid() { return 'n_' + Math.random().toString(36).slice(2) + Date.now().toString(36); }
    function fmtDate(ts) { const d = new Date(ts); return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')} ${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}`; }

    /*************************
     * Seed (처음 사용하는 사용자 편의용)
     *************************/
    (function seed() {
      const users = loadUsers();
      const notices = loadNotices();
      if (!users.teachers['demo@school.kr']) {
        users.teachers['demo@school.kr'] = { name: '데모교사', email:'demo@school.kr', pw: '1234', createdAt: Date.now(), createdWithSecret: true };
        saveUsers(users);
      }
      if (notices.length === 0) {
        notices.push({ id: uid(), title: '도서관 정리 봉사', content: '매주 수요일 방과후 16:30~18:00. 정리 및 분류 지원.', tag: '봉사', limit: 10, createdAt: Date.now(), author: '데모교사', authorEmail: 'demo@school.kr', applicants: [] });
        saveNotices(notices);
      }
    })();

    /*************************
     * State & Rendering
     *************************/
    const el = (id) => document.getElementById(id);

    function setToolbar(role, name) {
      const bar = el('toolbar');
      bar.innerHTML = '';
      if (!role) return;
      const hello = document.createElement('div');
      hello.className = 'pill';
      hello.textContent = `${name}님`;
      bar.appendChild(hello);

      if (role === 'teacher') {
        const btnLogout = button('로그아웃', () => doLogout());
        const btnScrollNotices = button('공지 목록');
        btnScrollNotices.addEventListener('click', () => document.getElementById('teacherSection').scrollIntoView({behavior:'smooth'}));
        bar.appendChild(btnScrollNotices);
        bar.appendChild(btnLogout);
      } else if (role === 'student') {
        const btnMy = button('신청 내역');
        btnMy.addEventListener('click', () => document.getElementById('studentMyApplies').scrollIntoView({behavior:'smooth'}));
        const btnLogout = button('로그아웃', () => doLogout());
        bar.appendChild(btnMy);
        bar.appendChild(btnLogout);
      }
    }

    function button(text, onClick, cls='btn') {
      const b = document.createElement('button'); b.className = cls; b.textContent = text; if (onClick) b.addEventListener('click', onClick); return b;
    }

    function render() {
      const current = loadCurrent();
      const auth = el('authSection');
      const stu = el('studentSection');
      const tch = el('teacherSection');

      if (!current) {
        setToolbar(null);
        auth.style.display = 'block';
        stu.style.display = 'none';
        tch.style.display = 'none';
        return;
      }

      if (current.role === 'student') {
        auth.style.display = 'none';
        stu.style.display = 'block';
        tch.style.display = 'none';
        const users = loadUsers();
        const s = users.students[current.key];
        setToolbar('student', s?.name || '학생');
        renderStudent();
      } else if (current.role === 'teacher') {
        auth.style.display = 'none';
        stu.style.display = 'none';
        tch.style.display = 'block';
        const users = loadUsers();
        const t = users.teachers[current.key];
        setToolbar('teacher', t?.name || '선생님');
        renderTeacher();
      }
    }

    /*************************
     * Auth Handlers
     *************************/
    // Tabs
    function setupTabs() {
      const pairs = [
        { tab: 'tabLoginStudent', pane: 'loginStudent' },
        { tab: 'tabLoginTeacher', pane: 'loginTeacher' },
      ];
      pairs.forEach(({tab, pane}) => {
        el(tab).addEventListener('click', () => {
          pairs.forEach(({tab, pane}) => { el(tab).classList.remove('active'); el(pane).style.display = 'none'; });
          el(tab).classList.add('active'); el(pane).style.display = 'block';
        });
      });

      const pairs2 = [
        { tab: 'tabJoinStudent', pane: 'joinStudent' },
        { tab: 'tabJoinTeacher', pane: 'joinTeacher' },
      ];
      pairs2.forEach(({tab, pane}) => {
        el(tab).addEventListener('click', () => {
          pairs2.forEach(({tab, pane}) => { el(tab).classList.remove('active'); el(pane).style.display = 'none'; });
          el(tab).classList.add('active'); el(pane).style.display = 'block';
        });
      });
    }

    // Join (Student)
    el('btnJoinStudent').addEventListener('click', () => {
      const name = el('joinSName').value.trim();
      const id = el('joinSId').value.trim();
      const room = el('joinSRoom').value.trim();
      const pw = el('joinSPw').value;
      if (!name || !id || !room || !pw) return alert('모든 항목을 입력하세요.');
      const users = loadUsers();
      if (users.students[id]) return alert('이미 존재하는 학번입니다.');
      users.students[id] = { name, studentId: id, room, pw, createdAt: Date.now() };
      saveUsers(users);
      alert('학생 회원가입 완료! 이제 로그인하세요.');
      el('joinSName').value = el('joinSId').value = el('joinSRoom').value = el('joinSPw').value = '';
    });

    // Join (Teacher)
    el('btnJoinTeacher').addEventListener('click', () => {
      const name = el('joinTName').value.trim();
      const email = el('joinTEmail').value.trim();
      const pw = el('joinTPw').value;
      const secret = el('joinTSecret').value;
      if (!name || !email || !pw || !secret) return alert('모든 항목을 입력하세요.');
      if (secret !== TEACHER_SECRET) return alert('비밀코드가 일치하지 않습니다.');
      const users = loadUsers();
      if (users.teachers[email]) return alert('이미 존재하는 이메일입니다.');
      users.teachers[email] = { name, email, pw, createdAt: Date.now(), createdWithSecret: true };
      saveUsers(users);
      alert('교사 회원가입 완료! 이제 로그인하세요.');
      el('joinTName').value = el('joinTEmail').value = el('joinTPw').value = el('joinTSecret').value = '';
    });

    // Login (Student)
    el('btnLoginStudent').addEventListener('click', () => {
      const id = el('loginStudentId').value.trim();
      const pw = el('loginStudentPw').value;
      const users = loadUsers();
      const u = users.students[id];
      if (!u) return alert('존재하지 않는 학번입니다.');
      if (u.pw !== pw) return alert('비밀번호가 틀렸습니다.');
      saveCurrent({ role: 'student', key: id });
      render();
    });

    // Login (Teacher)
    el('btnLoginTeacher').addEventListener('click', () => {
      const email = el('loginTeacherEmail').value.trim();
      const pw = el('loginTeacherPw').value;
      const secret = el('loginTeacherSecret').value;
      if (secret !== TEACHER_SECRET) return alert('비밀코드가 일치하지 않습니다.');
      const users = loadUsers();
      const u = users.teachers[email];
      if (!u) return alert('존재하지 않는 교사 계정입니다.');
      if (u.pw !== pw) return alert('비밀번호가 틀렸습니다.');
      saveCurrent({ role: 'teacher', key: email });
      render();
    });

    function doLogout() { clearCurrent(); render(); }

    /*************************
     * Student UI
     *************************/
    function renderStudent() {
      const current = loadCurrent();
      const users = loadUsers();
      const me = users.students[current.key];
      const notices = loadNotices().sort((a,b)=>b.createdAt-a.createdAt);
      const list = document.createElement('div');

      if (notices.length === 0) {
        list.innerHTML = `<div class="empty">등록된 공지가 없습니다.</div>`;
      } else {
        notices.forEach(n => {
          const applied = n.applicants?.some(a => a.studentId === me.studentId);
          const isFull = n.limit && (n.applicants?.length || 0) >= n.limit;
          const card = document.createElement('div');
          card.className = 'notice-card';
          card.innerHTML = `
            <div class="notice-header">
              <div style="font-weight:800;">${escapeHTML(n.title)}</div>
              <div style="display:flex; gap:6px; align-items:center;">
                <span class="badge">${n.tag ? escapeHTML(n.tag) : '공지'}</span>
                ${n.limit ? `<span class="badge ${isFull? 'warn' : 'good'}">${(n.applicants?.length||0)}/${n.limit}</span>` : ''}
              </div>
            </div>
            <div class="muted">${escapeHTML(n.content)}</div>
            <div class="muted" style="font-size:12px;">${n.author} · ${fmtDate(n.createdAt)}</div>
          `;
          const row = document.createElement('div');
          row.style.display = 'flex';
          row.style.gap = '8px';
          row.style.justifyContent = 'flex-end';

          const btn = button(applied ? '신청 완료' : (isFull ? '마감' : '신청하기'), null, 'btn primary');
          if (applied || isFull) { btn.disabled = true; btn.style.opacity = '0.6'; }
          else {
            btn.addEventListener('click', () => {
              const notices2 = loadNotices();
              const idx = notices2.findIndex(x => x.id === n.id);
              if (idx === -1) return;
              const nn = notices2[idx];
              if (nn.limit && (nn.applicants?.length || 0) >= nn.limit) { alert('마감되었습니다.'); return; }
              if (!nn.applicants) nn.applicants = [];
              if (nn.applicants.some(a => a.studentId === me.studentId)) { alert('이미 신청했습니다.'); return; }
              nn.applicants.push({ studentId: me.studentId, name: me.name, room: me.room, appliedAt: Date.now() });
              saveNotices(notices2);
              renderStudent();
            });
          }
          row.appendChild(btn);
          card.appendChild(row);
          list.appendChild(card);
        });
      }
      el('studentNoticeList').innerHTML = '';
      el('studentNoticeList').appendChild(list);

      // 내 신청 내역
      const appliedNotices = notices.filter(n => n.applicants?.some(a => a.studentId === me.studentId));
      if (appliedNotices.length === 0) {
        el('studentMyApplies').innerHTML = '<div class="empty">신청 내역이 없습니다.</div>';
      } else {
        const wrap = document.createElement('div');
        const table = document.createElement('table');
        table.className = 'table';
        table.innerHTML = `<thead><tr><th>제목</th><th>태그</th><th>신청일시</th><th>상태</th></tr></thead>`;
        const tb = document.createElement('tbody');
        appliedNotices.forEach(n => {
          const my = n.applicants.find(a => a.studentId === me.studentId);
          const tr = document.createElement('tr');
          tr.innerHTML = `<td>${escapeHTML(n.title)}</td><td>${n.tag || ''}</td><td>${fmtDate(my.appliedAt)}</td><td>${n.limit && n.applicants.length > n.limit ? '대기' : '신청'}</td>`;
          tb.appendChild(tr);
        });
        table.appendChild(tb); wrap.appendChild(table);
        el('studentMyApplies').innerHTML = '';
        el('studentMyApplies').appendChild(wrap);
      }
    }

    /*************************
     * Teacher UI
     *************************/
    function renderTeacher() {
      const current = loadCurrent();
      const users = loadUsers();
      const me = users.teachers[current.key];
      // 공지 목록 렌더
      const notices = loadNotices().sort((a,b)=>b.createdAt-a.createdAt);
      const list = document.createElement('div');
      if (notices.length === 0) list.innerHTML = '<div class="empty">등록된 공지가 없습니다.</div>';
      else {
        notices.forEach(n => {
          const card = document.createElement('div'); card.className = 'notice-card';
          const count = n.applicants?.length || 0;
          const isFull = n.limit && count >= n.limit;
          card.innerHTML = `
            <div class="notice-header">
              <div style="font-weight:800;">${escapeHTML(n.title)}</div>
              <div style="display:flex; gap:6px; align-items:center;">
                <span class="badge">${n.tag ? escapeHTML(n.tag) : '공지'}</span>
                ${n.limit ? `<span class="badge ${isFull? 'warn' : 'good'}">${count}/${n.limit}</span>` : `<span class="badge">신청 ${count}명</span>`}
              </div>
            </div>
            <div class="muted">${escapeHTML(n.content)}</div>
            <div class="muted" style="font-size:12px;">${n.author} · ${fmtDate(n.createdAt)}</div>
          `;
          const row = document.createElement('div');
          row.style.display = 'flex'; row.style.gap = '8px'; row.style.justifyContent = 'flex-end';
          const btnApplicants = button('신청자 확인', () => openApplicants(n.id), 'btn');
          const btnDelete = button('공지 삭제', () => deleteNotice(n.id), 'btn danger');
          row.appendChild(btnApplicants);
          if (me.email === n.authorEmail) row.appendChild(btnDelete);
          card.appendChild(row);
          list.appendChild(card);
        });
      }
      el('teacherNoticeList').innerHTML = '';
      el('teacherNoticeList').appendChild(list);

      // 게시 버튼 핸들러 (중복 바인딩 방지 위해 재설정)
      const btn = el('btnPublish');
      const cloned = btn.cloneNode(true);
      btn.parentNode.replaceChild(cloned, btn);
      cloned.addEventListener('click', () => {
        const title = el('noticeTitle').value.trim();
        const content = el('noticeContent').value.trim();
        const limit = Number(el('noticeLimit').value) || null;
        const tag = el('noticeTag').value.trim();
        if (!title || !content) return alert('제목과 내용을 입력하세요.');
        const notices = loadNotices();
        notices.unshift({ id: uid(), title, content, tag: tag || null, limit, createdAt: Date.now(), author: me.name, authorEmail: me.email, applicants: [] });
        saveNotices(notices);
        el('noticeTitle').value = el('noticeContent').value = el('noticeLimit').value = el('noticeTag').value = '';
        renderTeacher();
      });
    }

    function openApplicants(noticeId) {
      const notices = loadNotices();
      const n = notices.find(x => x.id === noticeId);
      if (!n) return;
      el('modalTitle').textContent = `신청자 목록 · ${n.title}`;
      const w = document.createElement('div');
      if (!n.applicants || n.applicants.length === 0) {
        w.innerHTML = '<div class="empty">신청자가 없습니다.</div>';
      } else {
        const table = document.createElement('table'); table.className = 'table';
        table.innerHTML = '<thead><tr><th>이름</th><th>학번</th><th>자습실</th><th>신청일시</th></tr></thead>';
        const tb = document.createElement('tbody');
        n.applicants.forEach(a => {
          const tr = document.createElement('tr');
          tr.innerHTML = `<td>${escapeHTML(a.name)}</td><td>${a.studentId}</td><td>${escapeHTML(a.room)}</td><td>${fmtDate(a.appliedAt)}</td>`;
          tb.appendChild(tr);
        });
        table.appendChild(tb); w.appendChild(table);
      }
      const wrap = el('applicantsTableWrap');
      wrap.innerHTML = ''; wrap.appendChild(w);
      el('modal').style.display = 'flex';
    }

    function deleteNotice(id) {
      if (!confirm('해당 공지를 삭제하시겠습니까?')) return;
      const notices = loadNotices().filter(n => n.id !== id);
      saveNotices(notices);
      renderTeacher();
    }

    el('btnCloseModal').addEventListener('click', () => { el('modal').style.display = 'none'; });

    /*************************
     * Utils
     *************************/
    function escapeHTML(str) {
      return String(str).replace(/[&<>"](/g, function(m) {
        return ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'})[m];
      });
    }

    /*************************
     * Init
     *************************/
    setupTabs();
    render();
  </script>
</body>
</html>
