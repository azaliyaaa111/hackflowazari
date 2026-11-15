<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HackFlow — Платформа хакатонов</title>
  <!-- Подключение шрифта Inter -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
  <!-- Подключение Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Подключение иконок Lucide -->
  <script src="https://unpkg.com/lucide@latest"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'sans-serif'],
          }
        }
      }
    }
  </script>
  <style>
    /* Плавное появление элементов */
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(6px); }
      to { opacity: 1; transform: translateY(0); }
    }
    .animate-fade-in {
      animation: fadeIn 0.25s ease-out forwards;
    }
    /* Кастомный скроллбар для темной темы */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #141414;
    }
    ::-webkit-scrollbar-thumb {
      background: #2a2a2a;
      border-radius: 3px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #b07d62;
    }
  </style>
</head>
<body class="bg-[#0f0f0f] text-[#e0e0e0] font-sans selection:bg-[#b07d62] selection:text-white min-h-screen">

  <!-- Корневой контейнер приложения -->
  <div id="app"></div>

  <!-- Основная логика приложения на чистом JS -->
  <script>
    // --- КОНФИГУРАЦИЯ ТЕМЫ ---
    const theme = {
      bg: 'bg-[#0f0f0f]',
      card: 'bg-[#1a1a1a]',
      cardHover: 'hover:bg-[#222222]',
      border: 'border-[#2a2a2a]',
      accent: 'bg-[#b07d62]',
      accentHover: 'hover:bg-[#966850]',
      textAccent: 'text-[#b07d62]',
      textMain: 'text-[#e0e0e0]',
      textMuted: 'text-[#808080]',
    };

    // --- ИСХОДНЫЕ ДАННЫЕ ---
    const HACKATHONS = [
      { id: 1, title: 'EdTech Challenge 2026', dates: '15-17 Ноября', prize: '500 000 ₽', tags: ['EdTech', 'Web', 'UX/UI'] },
      { id: 2, title: 'FinTech Cyberthon', dates: '1-3 Декабря', prize: '1 000 000 ₽', tags: ['FinTech', 'Mobile', 'AI'] },
      { id: 3, title: 'Urban Smart City', dates: '10-12 Декабря', prize: '300 000 ₽', tags: ['Urban', 'IoT', 'Data'] }
    ];

    const DEMO_USER = {
      name: 'Алексей Смирнов',
      role: 'Капитан',
      team: 'CyberFoxes'
    };

    const SCHEDULE = [
      { id: 1, date: '15 Ноября, 10:00', title: 'Церемония открытия', type: 'Трансляция', link: '#' },
      { id: 2, date: '15 Ноября, 12:00', title: 'Q&A сессия с экспертами', type: 'Zoom', link: '#' },
      { id: 3, date: '16 Ноября, 18:00', title: 'Чек-поинт #1', type: 'Дедлайн', link: '' },
      { id: 4, date: '17 Ноября, 23:59', title: 'Сдача финального решения', type: 'Дедлайн', link: '' },
    ];

    const EDUCATION = [
      { id: 1, title: 'Мастер-класс: Архитектура MVP', duration: '45 мин', type: 'Видео' },
      { id: 2, title: 'Как подготовить питч для жюри', duration: '30 мин', type: 'Видео' },
      { id: 3, title: 'Требования к UI/UX', duration: '15 стр', type: 'Документ' },
    ];

    // --- ГЛОБАЛЬНОЕ СОСТОЯНИЕ ПРИЛОЖЕНИЯ (STATE) ---
    const state = {
      viewState: 'feed', // 'feed' | 'registration' | 'platform'
      selectedHackathon: null,
      teamData: { teamName: '', captain: '', members: '' },
      activeTab: 'dashboard', // 'dashboard' | 'team' | 'education' | 'schedule' | 'submission'
      isMobileMenuOpen: false,
      tasks: [],
      messages: [],
      newTaskText: '',
      newTaskAssignee: 'Не назначен',
      newMessageText: '',
      isSubmitted: false,
      submissionForm: {
        title: '',
        repoUrl: '',
        comment: '',
        fileName: ''
      }
    };

    // Безопасное экранирование текста для предотвращения XSS при вводе пользователя
    function escapeHTML(str) {
      if (!str) return '';
      return String(str)
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#039;');
    }

    // Вспомогательная функция получения данных пользователя и списка команды
    function getTeamContext() {
      const currentUser = {
        name: state.teamData.captain || DEMO_USER.name,
        role: 'Капитан',
        team: state.teamData.teamName || DEMO_USER.team
      };
      const teamMembers = [
        state.teamData.captain || 'Алексей',
        ...(state.teamData.members || '').split('\n').map(m => m.trim()).filter(Boolean)
      ];
      return { currentUser, teamMembers };
    }

    // Инициализация задач и чата после регистрации команды
    function initPlatformData() {
      const { teamMembers } = getTeamContext();
      const getMember = (idx) => teamMembers[idx] || teamMembers[0] || 'Участник';

      state.tasks = [
        { id: 1, title: 'Анализ бизнес-требований', status: 'done', assignee: getMember(0) },
        { id: 2, title: 'Проектирование архитектуры', status: 'in_progress', assignee: getMember(1) },
        { id: 3, title: 'Верстка интерфейса', status: 'in_progress', assignee: getMember(2) },
        { id: 4, title: 'Подготовка презентации', status: 'todo', assignee: getMember(0) },
      ];

      state.messages = [
        { id: 1, author: 'Система', text: 'Добро пожаловать в командный чат!', time: '10:00' },
        { id: 2, author: getMember(1), text: 'Привет! Я начинаю собирать материалы. Скину ссылку через час.', time: '10:15' },
        { id: 3, author: getMember(2), text: 'Супер, жду. Я пока накидаю компоненты интерфейса.', time: '10:17' },
      ];
    }

    // --- ОБРАБОТЧИКИ СОБЫТИЙ И ДЕЙСТВИЙ ---
    window.selectHackathon = function(id) {
      const hack = HACKATHONS.find(h => h.id === id);
      if (hack) {
        state.selectedHackathon = hack;
        state.viewState = 'registration';
        render();
      }
    };

    window.goBackToFeed = function() {
      state.viewState = 'feed';
      render();
    };

    window.handleRegistrationSubmit = function(e) {
      e.preventDefault();
      const teamName = document.getElementById('reg-teamName').value.trim();
      const captain = document.getElementById('reg-captain').value.trim();
      const members = document.getElementById('reg-members').value.trim();

      state.teamData = { teamName, captain, members };
      initPlatformData();
      state.viewState = 'platform';
      state.activeTab = 'dashboard';
      render();
    };

    window.setActiveTab = function(tabId) {
      state.activeTab = tabId;
      state.isMobileMenuOpen = false;
      render();
    };

    window.toggleMobileMenu = function() {
      state.isMobileMenuOpen = !state.isMobileMenuOpen;
      render();
    };

    window.handleAddTask = function(e) {
      e.preventDefault();
      if (!state.newTaskText.trim()) return;
      state.tasks.push({
        id: Date.now(),
        title: state.newTaskText.trim(),
        status: 'todo',
        assignee: state.newTaskAssignee
      });
      state.newTaskText = '';
      state.newTaskAssignee = 'Не назначен';
      render();
    };

    window.claimTask = function(taskId, e) {
      if (e) e.stopPropagation();
      const { currentUser } = getTeamContext();
      state.tasks = state.tasks.map(t => t.id === taskId ? { ...t, assignee: currentUser.name } : t);
      render();
    };

    window.moveTask = function(taskId, newStatus, e) {
      if (e) e.stopPropagation();
      state.tasks = state.tasks.map(t => t.id === taskId ? { ...t, status: newStatus } : t);
      render();
    };

    window.handleSendMessage = function(e) {
      e.preventDefault();
      if (!state.newMessageText.trim()) return;
      const { currentUser } = getTeamContext();
      state.messages.push({
        id: Date.now(),
        author: currentUser.name,
        text: state.newMessageText.trim(),
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      });
      state.newMessageText = '';
      render();

      // Прокрутка чата вниз после отправки сообщения
      const chatContainer = document.getElementById('chat-messages');
      if (chatContainer) {
        chatContainer.scrollTop = chatContainer.scrollHeight;
      }
    };

    window.handleSubmission = function(e) {
      e.preventDefault();
      state.isSubmitted = true;
      render();
    };

    window.resetSubmission = function() {
      state.isSubmitted = false;
      render();
    };

    window.triggerFileUpload = function() {
      const fileInput = document.getElementById('sub-file-input');
      if (fileInput) fileInput.click();
    };

    window.handleFileChange = function(e) {
      const file = e.target.files[0];
      if (file) {
        state.submissionForm.fileName = file.name;
        render();
      }
    };

    // --- ШАБЛОНЫ ЭКРАНОВ (VIEWS) ---

    // 1. Витрина хакатонов
    function renderFeedView() {
      const cardsHtml = HACKATHONS.map(hack => {
        const tagsHtml = hack.tags.map(tag => `
          <span class="text-[10px] uppercase tracking-wider bg-[#2a2a2a] text-gray-300 px-2 py-1 rounded">
            ${escapeHTML(tag)}
          </span>
        `).join('');

        return `
          <div class="${theme.card} border ${theme.border} p-6 rounded-2xl flex flex-col transition-all hover:border-[#b07d62] group">
            <div class="flex justify-between items-start mb-4">
              <div class="flex gap-2 flex-wrap">
                ${tagsHtml}
              </div>
              <i data-lucide="trophy" class="w-[18px] h-[18px] text-[#555] group-hover:text-[#b07d62] transition-colors shrink-0"></i>
            </div>
            <h3 class="text-xl font-medium mb-2">${escapeHTML(hack.title)}</h3>
            <div class="text-sm text-gray-400 space-y-1 mb-6 flex-1">
              <p>Даты: <span class="text-gray-200">${escapeHTML(hack.dates)}</span></p>
              <p>Призовой фонд: <span class="text-[#b07d62] font-medium">${escapeHTML(hack.prize)}</span></p>
            </div>
            <button 
              onclick="selectHackathon(${hack.id})"
              class="w-full py-3 border border-[#333] group-hover:border-[#b07d62] group-hover:bg-[#b07d62] group-hover:text-white text-xs tracking-widest uppercase rounded-lg transition-all font-bold"
            >
              Выбрать
            </button>
          </div>
        `;
      }).join('');

      return `
        <div class="min-h-screen ${theme.bg} ${theme.textMain} font-sans p-6 md:p-12 animate-fade-in">
          <div class="max-w-6xl mx-auto">
            <header class="mb-12 text-center">
              <h1 class="text-4xl tracking-[0.2em] font-light uppercase mb-3 text-white">HackFlow</h1>
              <p class="${theme.textMuted}">Платформа отбора и проведения хакатонов</p>
            </header>
            
            <h2 class="text-xl font-light uppercase tracking-widest mb-8 border-b border-[#2a2a2a] pb-4">Актуальные хакатоны</h2>
            
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              ${cardsHtml}
            </div>
          </div>
        </div>
      `;
    }

    // 2. Экран регистрации команды
    function renderRegistrationView() {
      const hackTitle = state.selectedHackathon ? state.selectedHackathon.title : '';
      return `
        <div class="min-h-screen ${theme.bg} ${theme.textMain} font-sans flex items-center justify-center p-6 animate-fade-in">
          <div class="${theme.card} border ${theme.border} p-8 rounded-3xl max-w-md w-full relative">
            <button onclick="goBackToFeed()" class="absolute top-8 left-8 text-gray-500 hover:text-white transition-colors" title="Назад">
              <i data-lucide="arrow-left" class="w-6 h-6"></i>
            </button>
            <div class="text-center mb-8 mt-2">
              <h2 class="text-2xl font-light uppercase tracking-widest mb-2">Регистрация</h2>
              <p class="text-sm ${theme.textAccent}">${escapeHTML(hackTitle)}</p>
            </div>

            <form onsubmit="handleRegistrationSubmit(event)" class="space-y-5">
              <div>
                <label class="block text-xs uppercase tracking-widest text-[#808080] mb-2">Название команды</label>
                <input 
                  id="reg-teamName" 
                  required 
                  type="text" 
                  value="${escapeHTML(state.teamData.teamName)}" 
                  oninput="state.teamData.teamName = this.value"
                  class="w-full bg-[#111] border border-[#333] rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white" 
                  placeholder="АйтиШники" 
                />
              </div>
              <div>
                <label class="block text-xs uppercase tracking-widest text-[#808080] mb-2">ФИО Капитана</label>
                <input 
                  id="reg-captain" 
                  required 
                  type="text" 
                  value="${escapeHTML(state.teamData.captain)}" 
                  oninput="state.teamData.captain = this.value"
                  class="w-full bg-[#111] border border-[#333] rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white" 
                  placeholder="Иванов Иван Иванович" 
                />
              </div>
              <div>
                <label class="block text-xs uppercase tracking-widest text-[#808080] mb-2">
                  ФИО остальных участников<br/>
                  <span class="text-[10px] lowercase text-[#555]">(каждый с новой строки)</span>
                </label>
                <textarea 
                  id="reg-members" 
                  rows="3" 
                  required 
                  oninput="state.teamData.members = this.value"
                  class="w-full bg-[#111] border border-[#333] rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white resize-none" 
                  placeholder="Петров Петр Петрович&#10;Сидорова Анна Ивановна"
                >${escapeHTML(state.teamData.members)}</textarea>
              </div>
              <button type="submit" class="w-full py-4 ${theme.accent} ${theme.accentHover} text-white text-sm tracking-widest uppercase rounded-xl font-bold transition-all shadow-lg shadow-[#b07d62]/20 mt-4">
                Зарегистрировать команду
              </button>
            </form>
          </div>
        </div>
      `;
    }

    // 3. Основная платформа и её вкладки
    function renderDashboardTab(currentUser, teamMembersCount) {
      const title = state.selectedHackathon?.title || 'HackFlow Challenge';
      const doneTasksCount = state.tasks.filter(t => t.status === 'done').length;
      const totalTasksCount = state.tasks.length || 1;
      const progressPercent = Math.round((doneTasksCount / totalTasksCount) * 100);

      return `
        <div class="space-y-8 animate-fade-in">
          <header class="mb-10">
            <h2 class="text-3xl font-light uppercase tracking-widest mb-2">${escapeHTML(title)}</h2>
            <p class="${theme.textMuted}">Основной этап соревнования. Создайте цифровое решение будущего.</p>
          </header>

          <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
            <!-- Карточка статуса -->
            <div class="${theme.card} border ${theme.border} p-6 rounded-2xl relative overflow-hidden group">
              <div class="absolute top-0 right-0 p-4 ${theme.textAccent} opacity-20 group-hover:opacity-100 transition-opacity">
                <i data-lucide="clock" class="w-12 h-12"></i>
              </div>
              <h3 class="text-xs uppercase tracking-widest text-[#808080] mb-2">Статус этапа</h3>
              <div class="text-2xl font-semibold mb-1">В процессе</div>
              <p class="text-sm text-[#b07d62]">До дедлайна: 2 дня 14 часов</p>
            </div>

            <!-- Карточка команды -->
            <div class="${theme.card} border ${theme.border} p-6 rounded-2xl relative overflow-hidden group">
              <div class="absolute top-0 right-0 p-4 ${theme.textAccent} opacity-20 group-hover:opacity-100 transition-opacity">
                <i data-lucide="users" class="w-12 h-12"></i>
              </div>
              <h3 class="text-xs uppercase tracking-widest text-[#808080] mb-2">Ваша команда</h3>
              <div class="text-2xl font-semibold mb-1 truncate">${escapeHTML(currentUser.team)}</div>
              <p class="text-sm text-gray-400">Участников: ${teamMembersCount || 4} / Ментор: Назначен</p>
            </div>

            <!-- Карточка прогресса -->
            <div class="${theme.card} border ${theme.border} p-6 rounded-2xl relative overflow-hidden group">
              <div class="absolute top-0 right-0 p-4 ${theme.textAccent} opacity-20 group-hover:opacity-100 transition-opacity">
                <i data-lucide="check-square" class="w-12 h-12"></i>
              </div>
              <h3 class="text-xs uppercase tracking-widest text-[#808080] mb-2">Прогресс</h3>
              <div class="text-2xl font-semibold mb-1">${progressPercent}% готово</div>
              <div class="w-full bg-[#2a2a2a] h-1.5 rounded-full mt-3 overflow-hidden">
                <div class="${theme.accent} h-1.5 rounded-full transition-all duration-300" style="width: ${progressPercent}%"></div>
              </div>
            </div>
          </div>

          <div class="${theme.card} border ${theme.border} p-8 rounded-2xl">
            <h3 class="text-sm uppercase tracking-widest text-[#808080] mb-6">Условие кейса</h3>
            <div class="max-w-none text-gray-300 text-sm leading-relaxed space-y-4">
              <p>
                После завершения отборочного этапа организаторы хакатона сталкиваются с задачей — обеспечить эффективную работу участников. Командная работа, обучение и коммуникация часто распределены между разными сервисами: мессенджерами, онлайн-досками, видеоконференциями.
              </p>
              <p>
                <strong>Ваша задача:</strong> Разработать MVP цифровой среды, которая обеспечивает проведение основного этапа студенческого хакатона. Платформа должна объединять командную работу, расписание, образовательный трек и сдачу решений в едином пространстве.
              </p>
              <button onclick="setActiveTab('education')" class="mt-6 px-6 py-3 ${theme.accent} ${theme.accentHover} text-white text-xs tracking-widest uppercase rounded-lg transition-colors">
                Скачать полные материалы (PDF)
              </button>
            </div>
          </div>
        </div>
      `;
    }

    function renderTeamTab(currentUser, teamMembers) {
      const columns = [
        { id: 'todo', title: 'К выполнению' },
        { id: 'in_progress', title: 'В работе' },
        { id: 'done', title: 'Готово' }
      ];

      const memberOptionsHtml = [
        `<option value="Не назначен" ${state.newTaskAssignee === 'Не назначен' ? 'selected' : ''}>Свободная задача</option>`,
        ...teamMembers.map(member => `
          <option value="${escapeHTML(member)}" ${state.newTaskAssignee === member ? 'selected' : ''}>
            ${escapeHTML(member)}
          </option>
        `)
      ].join('');

      const columnsHtml = columns.map(col => {
        const colTasks = state.tasks.filter(t => t.status === col.id);
        const tasksHtml = colTasks.map(task => {
          const assigneeHtml = task.assignee === 'Не назначен'
            ? `<button
                onclick="claimTask(${task.id}, event)"
                class="text-[10px] uppercase tracking-wider bg-[#b07d62]/20 text-[#b07d62] hover:bg-[#b07d62] hover:text-white transition-colors px-3 py-1.5 rounded font-bold"
              >
                Взять себе
              </button>`
            : `<span class="text-[10px] uppercase tracking-wider text-[#808080] bg-[#111] px-2 py-1 rounded">
                ${escapeHTML(task.assignee)}
              </span>`;

          const prevBtn = col.id !== 'todo'
            ? `<button onclick="moveTask(${task.id}, '${col.id === 'done' ? 'in_progress' : 'todo'}', event)" class="text-gray-500 hover:text-white p-1" title="Назад">
                <i data-lucide="chevron-right" class="w-3.5 h-3.5 rotate-180"></i>
              </button>`
            : '';

          const nextBtn = col.id !== 'done'
            ? `<button onclick="moveTask(${task.id}, '${col.id === 'todo' ? 'in_progress' : 'done'}', event)" class="text-gray-500 hover:text-white p-1" title="Вперед">
                <i data-lucide="chevron-right" class="w-3.5 h-3.5"></i>
              </button>`
            : '';

          return `
            <div class="bg-[#222222] border border-[#333] p-4 rounded-xl group hover:border-[#b07d62] transition-colors cursor-pointer flex flex-col">
              <p class="text-sm mb-3 flex-1">${escapeHTML(task.title)}</p>
              <div class="flex justify-between items-center mt-1">
                ${assigneeHtml}
                <div class="opacity-100 md:opacity-0 group-hover:opacity-100 transition-opacity flex gap-1">
                  ${prevBtn}
                  ${nextBtn}
                </div>
              </div>
            </div>
          `;
        }).join('');

        return `
          <div class="${theme.card} border ${theme.border} rounded-2xl p-4 flex flex-col">
            <div class="flex items-center justify-between mb-4">
              <h3 class="text-xs uppercase tracking-widest text-gray-400 font-semibold">${col.title}</h3>
              <span class="bg-[#2a2a2a] text-xs py-0.5 px-2 rounded-full text-gray-400">
                ${colTasks.length}
              </span>
            </div>
            <div class="flex-1 space-y-3 overflow-y-auto pr-1">
              ${tasksHtml}
            </div>
          </div>
        `;
      }).join('');

      const messagesHtml = state.messages.map(msg => {
        const isMe = msg.author === currentUser.name;
        return `
          <div class="flex flex-col ${isMe ? 'items-end' : 'items-start'}">
            <span class="text-[10px] text-gray-500 mb-1">${escapeHTML(msg.author)} • ${escapeHTML(msg.time)}</span>
            <div class="p-3 rounded-2xl max-w-[85%] text-sm ${
              isMe 
                ? `${theme.accent} text-white rounded-br-sm shadow-md` 
                : 'bg-[#2a2a2a] text-gray-200 rounded-bl-sm'
            }">
              ${escapeHTML(msg.text)}
            </div>
          </div>
        `;
      }).join('');

      return `
        <div class="h-full flex flex-col animate-fade-in">
          <header class="mb-6 flex justify-between items-end">
            <div>
              <h2 class="text-3xl font-light uppercase tracking-widest mb-2">Рабочее пространство</h2>
              <p class="${theme.textMuted}">Команда: ${escapeHTML(currentUser.team)}</p>
            </div>
          </header>

          <div class="grid grid-cols-1 lg:grid-cols-3 gap-6 flex-1 min-h-[600px]">
            <!-- Kanban Доска -->
            <div class="lg:col-span-2 flex flex-col space-y-4">
              <form onsubmit="handleAddTask(event)" class="flex gap-2 flex-wrap sm:flex-nowrap">
                <input 
                  type="text" 
                  value="${escapeHTML(state.newTaskText)}"
                  oninput="state.newTaskText = this.value"
                  placeholder="Новая задача..." 
                  class="flex-1 bg-[#1a1a1a] border border-[#2a2a2a] rounded-lg px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white placeholder-gray-600 min-w-[200px]"
                />
                <select
                  onchange="state.newTaskAssignee = this.value"
                  class="bg-[#1a1a1a] border border-[#2a2a2a] rounded-lg px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white outline-none cursor-pointer"
                >
                  ${memberOptionsHtml}
                </select>
                <button type="submit" class="px-4 py-2 ${theme.accent} ${theme.accentHover} text-white rounded-lg transition-colors flex items-center justify-center">
                  <i data-lucide="plus" class="w-5 h-5"></i>
                </button>
              </form>

              <div class="grid grid-cols-1 md:grid-cols-3 gap-4 flex-1">
                ${columnsHtml}
              </div>
            </div>

            <!-- Командный Чат -->
            <div class="${theme.card} border ${theme.border} rounded-2xl flex flex-col lg:h-full max-h-[600px]">
              <div class="p-4 border-b border-[#2a2a2a]">
                <h3 class="text-xs uppercase tracking-widest text-[#808080] font-semibold flex items-center gap-2">
                  <i data-lucide="message-square" class="w-3.5 h-3.5"></i> Чат команды
                </h3>
              </div>
              
              <div id="chat-messages" class="flex-1 p-4 overflow-y-auto space-y-4">
                ${messagesHtml}
              </div>

              <form onsubmit="handleSendMessage(event)" class="p-4 border-t border-[#2a2a2a] flex gap-2">
                <input 
                  type="text" 
                  value="${escapeHTML(state.newMessageText)}"
                  oninput="state.newMessageText = this.value"
                  placeholder="Сообщение..." 
                  class="flex-1 bg-[#111] border border-[#333] rounded-full px-4 py-2 text-sm focus:outline-none focus:border-[#b07d62] text-white placeholder-gray-600"
                />
                <button type="submit" class="p-2.5 rounded-full ${theme.accent} ${theme.accentHover} text-white transition-colors flex items-center justify-center">
                  <i data-lucide="send" class="w-[18px] h-[18px] ml-0.5"></i>
                </button>
              </form>
            </div>
          </div>
        </div>
      `;
    }

    function renderEducationTab() {
      const itemsHtml = EDUCATION.map(item => {
        const iconName = item.type === 'Видео' ? 'play-circle' : 'file-text';
        return `
          <div class="${theme.card} border ${theme.border} rounded-2xl overflow-hidden group cursor-pointer hover:border-[#b07d62] transition-colors">
            <div class="h-40 bg-[#111] relative flex items-center justify-center">
              <i data-lucide="${iconName}" class="w-12 h-12 text-[#333] group-hover:text-[#b07d62] transition-colors"></i>
              <div class="absolute bottom-2 right-2 bg-black/80 text-[10px] uppercase tracking-wider px-2 py-1 rounded text-gray-400">
                ${escapeHTML(item.duration)}
              </div>
            </div>
            <div class="p-5 border-t border-[#2a2a2a]">
              <span class="text-[10px] uppercase tracking-widest ${theme.textAccent} font-bold block mb-2">${escapeHTML(item.type)}</span>
              <h3 class="text-sm font-medium text-gray-200">${escapeHTML(item.title)}</h3>
            </div>
          </div>
        `;
      }).join('');

      return `
        <div class="space-y-8 animate-fade-in">
          <header class="mb-10">
            <h2 class="text-3xl font-light uppercase tracking-widest mb-2">Образовательный трек</h2>
            <p class="${theme.textMuted}">Материалы для успешного решения кейса</p>
          </header>

          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            ${itemsHtml}
          </div>
        </div>
      `;
    }

    function renderScheduleTab() {
      const eventsHtml = SCHEDULE.map((event, index) => {
        const isLast = index === SCHEDULE.length - 1;
        const isDeadline = event.type === 'Дедлайн';
        const hasPlayIcon = event.type === 'Zoom' || event.type === 'Трансляция';

        return `
          <div class="flex gap-6 ${!isLast ? 'relative' : ''}">
            ${!isLast ? `<div class="absolute left-[19px] top-10 bottom-[-24px] w-px bg-[#2a2a2a]"></div>` : ''}
            
            <div class="mt-1 w-10 h-10 rounded-full flex items-center justify-center shrink-0 z-10 ${
              isDeadline ? 'bg-[#b07d62]/20 text-[#b07d62]' : 'bg-[#1a1a1a] border border-[#333] text-gray-400'
            }">
              <i data-lucide="${isDeadline ? 'clock' : 'calendar'}" class="w-[18px] h-[18px]"></i>
            </div>

            <div class="${theme.card} border ${theme.border} p-6 rounded-2xl flex-1 hover:border-[#444] transition-colors">
              <div class="flex justify-between items-start flex-wrap gap-4">
                <div>
                  <span class="text-[10px] uppercase tracking-widest text-[#808080] font-bold block mb-1">${escapeHTML(event.date)}</span>
                  <h3 class="text-lg font-medium text-gray-200">${escapeHTML(event.title)}</h3>
                </div>
                
                ${event.link ? `
                  <button class="px-4 py-2 bg-[#111] border border-[#333] hover:border-[#b07d62] text-xs uppercase tracking-widest rounded-lg transition-all flex items-center gap-2 text-gray-300">
                    ${hasPlayIcon ? `<i data-lucide="play-circle" class="w-3.5 h-3.5"></i>` : ''}
                    ${escapeHTML(event.type)}
                  </button>
                ` : ''}
              </div>
            </div>
          </div>
        `;
      }).join('');

      return `
        <div class="space-y-8 animate-fade-in max-w-3xl">
          <header class="mb-10">
            <h2 class="text-3xl font-light uppercase tracking-widest mb-2">Расписание</h2>
            <p class="${theme.textMuted}">Ключевые события и чек-поинты</p>
          </header>

          <div class="space-y-6">
            ${eventsHtml}
          </div>
        </div>
      `;
    }

    function renderSubmissionTab() {
      if (state.isSubmitted) {
        return `
          <div class="h-[60vh] flex flex-col items-center justify-center animate-fade-in text-center">
            <div class="w-24 h-24 bg-[#b07d62]/20 rounded-full flex items-center justify-center mb-6 text-[#b07d62]">
              <i data-lucide="check-square" class="w-12 h-12"></i>
            </div>
            <h2 class="text-3xl font-light uppercase tracking-widest mb-4">Проект отправлен</h2>
            <p class="text-gray-400 max-w-md">
              Решение вашей команды успешно сохранено в системе. Вы можете обновить файлы до наступления дедлайна.
            </p>
            <button 
              onclick="resetSubmission()"
              class="mt-8 text-sm text-[#b07d62] hover:text-white underline decoration-[#b07d62]/50 underline-offset-4 transition-colors"
            >
              Обновить решение
            </button>
          </div>
        `;
      }

      return `
        <div class="space-y-8 animate-fade-in max-w-2xl mx-auto">
          <header class="mb-10 text-center">
            <h2 class="text-3xl font-light uppercase tracking-widest mb-2">Сдача проекта</h2>
            <p class="${theme.textMuted}">Загрузите презентацию и укажите ссылку на репозиторий</p>
          </header>

          <form 
            onsubmit="handleSubmission(event)"
            class="${theme.card} border ${theme.border} p-8 rounded-3xl space-y-6"
          >
            <div>
              <label class="block text-xs uppercase tracking-widest text-[#808080] mb-3">Название решения</label>
              <input 
                required
                type="text" 
                value="${escapeHTML(state.submissionForm.title)}"
                oninput="state.submissionForm.title = this.value"
                placeholder="Например: Платформа HackNova"
                class="w-full bg-[#111] border border-[#333] rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white"
              />
            </div>

            <div>
              <label class="block text-xs uppercase tracking-widest text-[#808080] mb-3">Ссылка на продукт / Репозиторий</label>
              <input 
                required
                type="url" 
                value="${escapeHTML(state.submissionForm.repoUrl)}"
                oninput="state.submissionForm.repoUrl = this.value"
                placeholder="https://github.com/..."
                class="w-full bg-[#111] border border-[#333] rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white"
              />
            </div>

            <div>
              <label class="block text-xs uppercase tracking-widest text-[#808080] mb-3">Презентация (PDF)</label>
              <input type="file" id="sub-file-input" accept=".pdf,.pptx" class="hidden" onchange="handleFileChange(event)" />
              <div onclick="triggerFileUpload()" class="border-2 border-dashed border-[#333] rounded-xl p-8 text-center hover:border-[#b07d62] transition-colors cursor-pointer bg-[#111]">
                <i data-lucide="upload" class="w-8 h-8 mx-auto text-gray-500 mb-3"></i>
                <p class="text-sm text-gray-400">
                  ${state.submissionForm.fileName 
                    ? `<span class="text-[#b07d62] font-medium">Выбран файл: ${escapeHTML(state.submissionForm.fileName)}</span>` 
                    : 'Нажмите для загрузки файла или перетащите его сюда'}
                </p>
                <p class="text-xs text-gray-600 mt-2">Максимум 50MB</p>
              </div>
            </div>

            <div>
              <label class="block text-xs uppercase tracking-widest text-[#808080] mb-3">Комментарий для жюри (опционально)</label>
              <textarea 
                rows="3"
                oninput="state.submissionForm.comment = this.value"
                placeholder="Краткое описание ключевых фич..."
                class="w-full bg-[#111] border border-[#333] rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-[#b07d62] text-white resize-none"
              >${escapeHTML(state.submissionForm.comment)}</textarea>
            </div>

            <button 
              type="submit"
              class="w-full py-4 ${theme.accent} ${theme.accentHover} text-white text-sm tracking-widest uppercase rounded-xl font-bold transition-all shadow-lg shadow-[#b07d62]/20 mt-4"
            >
              Отправить решение
            </button>
          </form>
        </div>
      `;
    }

    // Сборка основного экрана платформы с боковым меню
    function renderPlatformView() {
      const { currentUser, teamMembers } = getTeamContext();

      const navItems = [
        { id: 'dashboard', icon: 'layout-dashboard', label: 'Главная' },
        { id: 'team', icon: 'users', label: 'Команда' },
        { id: 'education', icon: 'book-open', label: 'Обучение' },
        { id: 'schedule', icon: 'calendar', label: 'Расписание' },
        { id: 'submission', icon: 'upload', label: 'Сдача проекта' },
      ];

      const navButtonsHtml = navItems.map(item => {
        const isActive = state.activeTab === item.id;
        return `
          <button
            onclick="setActiveTab('${item.id}')"
            class="w-full flex items-center gap-3 px-4 py-3 rounded-lg transition-all duration-200 uppercase tracking-wider text-xs font-semibold ${
              isActive 
                ? `${theme.accent} text-white shadow-lg` 
                : `text-[#808080] ${theme.cardHover} hover:text-white`
            }"
          >
            <i data-lucide="${item.icon}" class="w-[18px] h-[18px]"></i>
            ${item.label}
          </button>
        `;
      }).join('');

      let activeTabContent = '';
      switch (state.activeTab) {
        case 'dashboard':
          activeTabContent = renderDashboardTab(currentUser, teamMembers.length);
          break;
        case 'team':
          activeTabContent = renderTeamTab(currentUser, teamMembers);
          break;
        case 'education':
          activeTabContent = renderEducationTab();
          break;
        case 'schedule':
          activeTabContent = renderScheduleTab();
          break;
        case 'submission':
          activeTabContent = renderSubmissionTab();
          break;
        default:
          activeTabContent = renderDashboardTab(currentUser, teamMembers.length);
      }

      const firstLetter = currentUser.name ? currentUser.name.charAt(0).toUpperCase() : 'А';

      return `
        <div class="min-h-screen ${theme.bg} ${theme.textMain} font-sans flex flex-col md:flex-row">
          
          <!-- Мобильная шапка -->
          <div class="md:hidden flex items-center justify-between p-4 ${theme.card} border-b ${theme.border}">
            <div class="text-xl tracking-widest font-bold uppercase">HackFlow</div>
            <button onclick="toggleMobileMenu()" class="text-gray-300 hover:text-white">
              <i data-lucide="${state.isMobileMenuOpen ? 'x' : 'menu'}" class="w-6 h-6"></i>
            </button>
          </div>

          <!-- Боковая панель (Сайдбар) -->
          <aside class="${state.isMobileMenuOpen ? 'block' : 'hidden'} md:flex w-full md:w-64 ${theme.card} border-r ${theme.border} flex-col transition-all duration-300 shrink-0">
            <div class="p-6 hidden md:block border-b border-[#2a2a2a]">
              <h1 class="text-2xl tracking-[0.2em] font-light uppercase cursor-pointer" onclick="goBackToFeed()" title="К списку хакатонов">HackFlow</h1>
              <p class="text-[10px] ${theme.textAccent} tracking-widest mt-1 font-bold uppercase">Платформа хакатонов</p>
            </div>
            
            <div class="p-6 border-b border-[#2a2a2a]">
              <div class="flex items-center gap-3">
                <div class="w-10 h-10 rounded-full ${theme.accent} flex items-center justify-center font-bold text-white shrink-0">
                  ${escapeHTML(firstLetter)}
                </div>
                <div class="overflow-hidden">
                  <div class="font-medium text-sm truncate">${escapeHTML(currentUser.name)}</div>
                  <div class="text-xs ${theme.textMuted} truncate">${escapeHTML(currentUser.team)} • ${escapeHTML(currentUser.role)}</div>
                </div>
              </div>
            </div>

            <nav class="flex-1 p-4 space-y-2">
              ${navButtonsHtml}
            </nav>

            <div class="p-4 border-t border-[#2a2a2a]">
              <button onclick="goBackToFeed()" class="w-full flex items-center gap-2 px-4 py-2 text-xs text-gray-500 hover:text-gray-300 transition-colors uppercase tracking-wider">
                <i data-lucide="arrow-left" class="w-4 h-4"></i> Сменить хакатон
              </button>
            </div>
          </aside>

          <!-- Основной контент -->
          <main class="flex-1 p-4 md:p-8 lg:p-12 overflow-y-auto md:h-screen">
            <div class="max-w-6xl mx-auto">
              ${activeTabContent}
            </div>
          </main>
        </div>
      `;
    }

    // --- ГЛАВНАЯ ФУНКЦИЯ ОТРИСОВКИ (RENDER) ---
    function render() {
      const appContainer = document.getElementById('app');
      if (!appContainer) return;

      if (state.viewState === 'feed') {
        appContainer.innerHTML = renderFeedView();
      } else if (state.viewState === 'registration') {
        appContainer.innerHTML = renderRegistrationView();
      } else {
        appContainer.innerHTML = renderPlatformView();
      }

      // Инициализация всех SVG-иконок Lucide после обновления DOM
      if (window.lucide) {
        lucide.createIcons();
      }
    }

    // Запуск при загрузке страницы
    window.addEventListener('DOMContentLoaded', () => {
      render();
    });
  </script>
</body>
</html>
