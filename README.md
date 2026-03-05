# The-Master-and-Miss-Reading
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>读记打卡 - 我的每日阅读记录</title>
  <style>
    :root {
      --primary: #4f46e5;
      --primary-dark: #4338ca;
      --bg: #f8fafc;
      --card: #ffffff;
      --text: #1e293b;
      --text-light: #64748b;
      --danger: #ef4444;
    }

    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: system-ui, -apple-system, sans-serif;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
      padding: 16px;
      line-height: 1.5;
    }

    .container {
      max-width: 600px;
      margin: 0 auto;
    }

    h1 {
      text-align: center;
      margin: 24px 0 32px;
      color: var(--primary);
      font-size: 2rem;
    }

    .card {
      background: var(--card);
      border-radius: 12px;
      box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -1px rgba(0,0,0,0.06);
      padding: 20px;
      margin-bottom: 24px;
    }

    .form-group {
      margin-bottom: 16px;
    }

    label {
      display: block;
      margin-bottom: 6px;
      font-weight: 500;
      color: var(--text-light);
    }

    input, textarea {
      width: 100%;
      padding: 10px 12px;
      border: 1px solid #e2e8f0;
      border-radius: 6px;
      font-size: 1rem;
    }

    textarea {
      min-height: 90px;
      resize: vertical;
    }

    button {
      background: var(--primary);
      color: white;
      border: none;
      padding: 12px 24px;
      border-radius: 6px;
      font-size: 1rem;
      cursor: pointer;
      transition: background 0.2s;
    }

    button:hover {
      background: var(--primary-dark);
    }

    .btn-delete {
      background: var(--danger);
      padding: 8px 16px;
      font-size: 0.9rem;
    }

    .btn-delete:hover {
      background: #dc2626;
    }

    .record {
      border-bottom: 1px solid #e2e8f0;
      padding: 16px 0;
    }

    .record:last-child {
      border-bottom: none;
    }

    .record-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 8px;
    }

    .record-title {
      font-weight: 600;
      font-size: 1.1rem;
    }

    .record-date {
      color: var(--text-light);
      font-size: 0.9rem;
    }

    .record-page {
      color: var(--primary);
      font-weight: 600;
    }

    .stats {
      display: flex;
      gap: 32px;
      justify-content: center;
      margin: 32px 0;
      text-align: center;
    }

    .stat-item strong {
      display: block;
      font-size: 1.8rem;
      color: var(--primary);
    }

    .empty {
      text-align: center;
      color: var(--text-light);
      padding: 40px 0;
      font-size: 1.1rem;
    }

    @media (max-width: 500px) {
      .stats { flex-direction: column; gap: 16px; }
    }
  </style>
</head>
<body>

<div class="container">
  <h1>读记打卡</h1>

  <!-- 统计区域 -->
  <div class="stats">
    <div class="stat-item">
      <strong id="totalPages">0</strong>
      <div>总页数</div>
    </div>
    <div class="stat-item">
      <strong id="totalRecords">0</strong>
      <div>打卡次数</div>
    </div>
  </div>

  <!-- 添加记录表单 -->
  <div class="card">
    <form id="readingForm">
      <div class="form-group">
        <label for="bookTitle">书名</label>
        <input type="text" id="bookTitle" required placeholder="例如：《原子习惯》"/>
      </div>
      <div class="form-group">
        <label for="date">阅读日期</label>
        <input type="date" id="date" required value="" />
      </div>
      <div class="form-group">
        <label for="pages">今天读了多少页</label>
        <input type="number" id="pages" min="1" required placeholder="页数"/>
      </div>
      <div class="form-group">
        <label for="note">感悟 / 笔记</label>
        <textarea id="note" placeholder="今天读到了什么？有什么触动？（可选）"></textarea>
      </div>
      <button type="submit">记录今天阅读</button>
    </form>
  </div>

  <!-- 记录列表 -->
  <div class="card">
    <h2 style="margin-bottom:16px;">我的阅读足迹</h2>
    <div id="recordsList"></div>
    <div id="emptyMsg" class="empty" style="display:none;">还没有任何阅读记录～<br>今天开始打卡吧！</div>
  </div>
</div>

<script>
  const form = document.getElementById('readingForm');
  const recordsList = document.getElementById('recordsList');
  const emptyMsg = document.getElementById('emptyMsg');
  const totalPagesEl = document.getElementById('totalPages');
  const totalRecordsEl = document.getElementById('totalRecords');

  let records = JSON.parse(localStorage.getItem('readingRecords')) || [];

  // 设置默认日期为今天
  document.getElementById('date').value = new Date().toISOString().split('T')[0];

  function saveRecords() {
    localStorage.setItem('readingRecords', JSON.stringify(records));
    renderRecords();
    updateStats();
  }

  function renderRecords() {
    if (records.length === 0) {
      recordsList.innerHTML = '';
      emptyMsg.style.display = 'block';
      return;
    }

    emptyMsg.style.display = 'none';

    // 按日期倒序
    const sorted = [...records].sort((a,b) => new Date(b.date) - new Date(a.date));

    recordsList.innerHTML = sorted.map((r, index) => `
      <div class="record">
        <div class="record-header">
          <div class="record-title">${r.bookTitle}</div>
          <button class="btn-delete" onclick="deleteRecord(${index})">删除</button>
        </div>
        <div class="record-date">${new Date(r.date).toLocaleDateString('zh-CN')}</div>
        <div style="margin:8px 0;">
          阅读了 <span class="record-page">${r.pages} 页</span>
        </div>
        ${r.note ? `<div style="color:#475569;">“${r.note}”</div>` : ''}
      </div>
    `).join('');
  }

  function updateStats() {
    const totalPages = records.reduce((sum, r) => sum + Number(r.pages), 0);
    totalPagesEl.textContent = totalPages;
    totalRecordsEl.textContent = records.length;
  }

  function deleteRecord(index) {
    if (confirm('确定要删除这条阅读记录吗？')) {
      records.splice(index, 1);
      saveRecords();
    }
  }

  form.addEventListener('submit', e => {
    e.preventDefault();

    const newRecord = {
      bookTitle: document.getElementById('bookTitle').value.trim(),
      date: document.getElementById('date').value,
      pages: document.getElementById('pages').value,
      note: document.getElementById('note').value.trim()
    };

    if (!newRecord.bookTitle || !newRecord.date || !newRecord.pages) return;

    records.push(newRecord);
    saveRecords();

    form.reset();
    // 恢复今天日期
    document.getElementById('date').value = new Date().toISOString().split('T')[0];
  });

  // 首次加载
  renderRecords();
  updateStats();
</script>

</body>
</html>
