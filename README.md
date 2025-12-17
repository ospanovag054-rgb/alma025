<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Алма | Интеллектуальная Система Анализа</title>
    
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;600;800&display=swap" rel="stylesheet">
    
    <style>
        /* --- DYNAMIC THEME 2025 --- */
        :root {
            --glass-bg: rgba(13, 20, 35, 0.7); /* Темное стекло */
            --glass-border: rgba(255, 255, 255, 0.1);
            --primary: #3b82f6;      /* Неоновый синий */
            --primary-glow: rgba(59, 130, 246, 0.5);
            --accent: #10b981;       /* Неоновый зеленый */
            --danger: #ef4444;
            --text-main: #ffffff;
            --text-muted: #94a3b8;
            --card-radius: 24px;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Outfit', sans-serif; }

        body {
            background-color: #0f172a;
            color: var(--text-main);
            overflow-x: hidden;
            min-height: 100vh;
        }

        /* --- ФОНОВОЕ ВИДЕО (Улучшенная видимость) --- */
        .video-wrapper {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: -1;
        }
        .video-bg {
            width: 100%; height: 100%; object-fit: cover;
            opacity: 0.85; /* Сделали ярче */
            filter: contrast(1.1) saturate(1.1); /* Сочнее цвета */
        }
        .video-overlay {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            /* Градиент только снизу, чтобы верх был чистым */
            background: linear-gradient(to bottom, rgba(15, 23, 42, 0.2) 0%, rgba(15, 23, 42, 0.95) 90%);
        }

        /* --- UI COMPONENTS --- */
        .container { max-width: 1600px; margin: 0 auto; padding: 0 2rem; position: relative; z-index: 2; }

        .glass-panel {
            background: var(--glass-bg);
            backdrop-filter: blur(20px);
            -webkit-backdrop-filter: blur(20px);
            border: 1px solid var(--glass-border);
            border-radius: var(--card-radius);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        .glass-panel:hover {
            border-color: rgba(59, 130, 246, 0.3);
            box-shadow: 0 10px 40px rgba(0,0,0,0.4);
        }

        /* --- HEADER --- */
        header { padding: 3rem 0; display: flex; justify-content: space-between; align-items: center; }
        
        .brand h1 {
            font-size: 3rem; font-weight: 800; letter-spacing: -1px;
            background: linear-gradient(90deg, #fff, var(--primary));
            -webkit-background-clip: text; -webkit-text-fill-color: transparent;
            text-shadow: 0 0 30px rgba(59, 130, 246, 0.3);
        }
        .brand p { color: var(--text-muted); font-size: 1.1rem; letter-spacing: 2px; text-transform: uppercase; }

        /* Музыкальная кнопка */
        .music-btn {
            background: rgba(255,255,255,0.05); border: 1px solid var(--glass-border);
            color: white; padding: 12px 24px; border-radius: 50px; cursor: pointer;
            display: flex; align-items: center; gap: 10px; font-weight: 600;
            backdrop-filter: blur(10px); transition: 0.3s;
        }
        .music-btn:hover { background: var(--primary); box-shadow: 0 0 20px var(--primary-glow); border-color: var(--primary); }
        .bars { display: flex; gap: 3px; height: 16px; align-items: flex-end; }
        .bar { width: 3px; background: #fff; animation: none; height: 3px; }
        .playing .bar { animation: sound 0.5s infinite alternate; }
        .playing .bar:nth-child(2) { animation-delay: 0.2s; } 
        .playing .bar:nth-child(3) { animation-delay: 0.4s; }
        @keyframes sound { from { height: 3px; } to { height: 100%; } }

        /* --- GRID LAYOUT --- */
        .main-layout {
            display: grid; grid-template-columns: 350px 1fr; gap: 2rem; margin-bottom: 4rem;
        }

        /* --- ALMA SIDEBAR --- */
        .alma-container { position: sticky; top: 2rem; }
        
        .alma-avatar {
            overflow: hidden; padding: 0; position: relative;
            border: 1px solid rgba(59, 130, 246, 0.3);
            box-shadow: 0 0 30px rgba(59, 130, 246, 0.15);
        }
        .alma-video-wrapper {
            width: 100%; aspect-ratio: 9/16; position: relative;
        }
        .alma-video { width: 100%; height: 100%; object-fit: cover; }
        
        .alma-info {
            padding: 1.5rem; text-align: center;
            background: linear-gradient(to top, rgba(13,20,35,0.95), transparent);
            position: absolute; bottom: 0; left: 0; width: 100%;
        }
        .alma-badge {
            display: inline-block; padding: 4px 12px; border-radius: 20px;
            background: var(--primary); color: white; font-size: 0.7rem; font-weight: 800;
            margin-bottom: 10px; text-transform: uppercase; box-shadow: 0 0 10px var(--primary-glow);
        }

        /* --- MAIN CONTENT AREA --- */
        /* Навигация */
        .nav-pills {
            display: inline-flex; background: rgba(0,0,0,0.4); padding: 5px; border-radius: 60px;
            margin-bottom: 2rem; border: 1px solid var(--glass-border);
        }
        .nav-item {
            padding: 12px 30px; border-radius: 50px; color: var(--text-muted); border: none;
            background: transparent; cursor: pointer; font-weight: 600; transition: 0.3s;
            display: flex; gap: 8px; align-items: center; font-size: 1rem;
        }
        .nav-item:hover { color: white; background: rgba(255,255,255,0.05); }
        .nav-item.active { background: var(--primary); color: white; box-shadow: 0 4px 15px var(--primary-glow); }

        /* Секции */
        .section { display: none; animation: fadeInUp 0.6s ease-out; }
        .section.active { display: block; }
        @keyframes fadeInUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }

        /* KPI Cards */
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1.5rem; margin-bottom: 2rem; }
        .kpi-card { padding: 1.5rem; display: flex; flex-direction: column; justify-content: center; }
        .kpi-val { font-size: 2.8rem; font-weight: 800; margin: 10px 0; background: linear-gradient(90deg, #fff, var(--primary)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .kpi-sub { font-size: 0.9rem; color: var(--text-muted); display: flex; align-items: center; gap: 5px; }
        .trend-up { color: var(--accent); } .trend-down { color: var(--danger); }

        /* Графики */
        .chart-box { padding: 2rem; height: 500px; position: relative; margin-bottom: 2rem; }
        .chart-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1.5rem; }

        /* Кнопки управления (Исправленные) */
        .controls-wrapper { display: flex; gap: 10px; background: rgba(0,0,0,0.3); padding: 4px; border-radius: 12px; }
        .control-btn {
            padding: 8px 16px; background: transparent; border: 1px solid transparent;
            color: var(--text-muted); border-radius: 8px; cursor: pointer; transition: 0.2s;
            font-size: 0.9rem;
        }
        .control-btn:hover { color: white; }
        /* Класс active теперь будет четко виден */
        .control-btn.active { 
            background: var(--primary); 
            color: white; 
            box-shadow: 0 2px 10px var(--primary-glow);
            border-color: var(--primary);
        }

        /* Новые аналитические блоки */
        .insights-row { display: grid; grid-template-columns: repeat(2, 1fr); gap: 1.5rem; margin-top: 2rem; }
        .insight-box { padding: 1.5rem; }
        .insight-title { display: flex; align-items: center; gap: 10px; margin-bottom: 1rem; font-size: 1.1rem; font-weight: 600; color: white; }
        .insight-bar-bg { height: 8px; background: rgba(255,255,255,0.1); border-radius: 4px; margin: 10px 0; overflow: hidden; }
        .insight-bar-fill { height: 100%; border-radius: 4px; }

        /* Карта */
        #map { width: 100%; height: 500px; border-radius: 16px; z-index: 1; }

        /* --- ЧАТ БОТ (АЛМА) --- */
        .chat-widget {
            position: fixed; bottom: 30px; right: 30px; width: 400px; height: 600px;
            display: flex; flex-direction: column; z-index: 100;
            transform: translateY(120%); transition: transform 0.5s cubic-bezier(0.19, 1, 0.22, 1);
            border: 1px solid rgba(59, 130, 246, 0.4);
        }
        .chat-widget.open { transform: translateY(0); }
        
        .chat-header {
            padding: 1.5rem; background: rgba(13, 20, 35, 0.95); border-bottom: 1px solid var(--glass-border);
            display: flex; align-items: center; gap: 15px; border-radius: 24px 24px 0 0;
        }
        .chat-avatar-small { width: 40px; height: 40px; border-radius: 50%; object-fit: cover; border: 2px solid var(--primary); }
        
        .chat-body { flex: 1; padding: 1.5rem; overflow-y: auto; display: flex; flex-direction: column; gap: 15px; }
        .msg { max-width: 85%; padding: 12px 18px; border-radius: 18px; font-size: 0.95rem; line-height: 1.5; position: relative; }
        .msg.alma { background: rgba(255,255,255,0.1); color: #fff; align-self: flex-start; border-bottom-left-radius: 4px; border: 1px solid rgba(255,255,255,0.05); }
        .msg.user { background: var(--primary); color: white; align-self: flex-end; border-bottom-right-radius: 4px; box-shadow: 0 4px 15px var(--primary-glow); }
        
        .chat-footer { padding: 1rem; border-top: 1px solid var(--glass-border); background: rgba(0,0,0,0.2); border-radius: 0 0 24px 24px; }
        .input-group { display: flex; gap: 10px; position: relative; }
        .input-group input { 
            flex: 1; background: rgba(255,255,255,0.05); border: 1px solid var(--glass-border); 
            padding: 12px 20px; border-radius: 50px; color: white; outline: none; transition: 0.3s;
        }
        .input-group input:focus { border-color: var(--primary); background: rgba(0,0,0,0.4); }
        .send-btn { 
            width: 45px; height: 45px; border-radius: 50%; border: none; background: var(--primary); 
            color: white; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: 0.3s;
        }
        .send-btn:hover { transform: scale(1.1); box-shadow: 0 0 15px var(--primary-glow); }

        .chat-toggle-btn {
            position: fixed; bottom: 30px; right: 30px; width: 65px; height: 65px;
            background: var(--primary); border-radius: 50%; border: none; cursor: pointer;
            box-shadow: 0 0 30px var(--primary-glow); z-index: 101; transition: 0.4s;
            display: flex; align-items: center; justify-content: center; font-size: 1.8rem; color: white;
        }
        .chat-toggle-btn:hover { transform: scale(1.1) rotate(15deg); }

        /* Адаптивность */
        @media (max-width: 1024px) {
            .main-layout { grid-template-columns: 1fr; }
            .alma-container { display: none; } /* Скрываем большую Алму на мобильных, оставляем чат */
            .chat-widget { width: calc(100% - 40px); bottom: 100px; right: 20px; }
            header { flex-direction: column; gap: 20px; text-align: center; }
        }
    </style>
</head>
<body>

    <div class="video-wrapper">
        <video class="video-bg" autoplay muted loop playsinline>
            <source src="WhatsApp Video 2025-12-16 at 17.41.28.mp4" type="video/mp4">
        </video>
        <div class="video-overlay"></div>
    </div>

    <audio id="bg-music" loop>
        <source src="Breaking News.mp3" type="audio/mpeg">
    </audio>

    <div class="container">
        <header>
            <div class="brand">
                <h1>АЛМАТЫ 2025</h1>
                <p>Цифровой анализ миграционных потоков</p>
            </div>
            <button class="music-btn" id="music-toggle">
                <div class="bars">
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                </div>
                <span id="music-text">Включить звук</span>
            </button>
        </header>

        <div class="main-layout">
            
            <aside class="alma-container">
                <div class="glass-panel alma-avatar">
                    <div class="alma-video-wrapper">
                        <video class="alma-video" autoplay muted loop playsinline>
                            <source src="IMG_5131.MP4" type="video/mp4">
                        </video>
                        <div class="alma-info">
                            <span class="alma-badge">AI Assistant</span>
                            <h2 style="color: white; margin-bottom: 5px;">Алма</h2>
                            <p style="color: #cbd5e1; font-size: 0.9rem;">Интеллектуальный гид</p>
                            <button onclick="toggleChat()" style="margin-top: 15px; width: 100%; padding: 10px; background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); color: white; border-radius: 12px; cursor: pointer; transition: 0.3s;">
                                <i class="fas fa-comment-alt"></i> Спросить Алму
                            </button>
                        </div>
                    </div>
                </div>
                
                <div class="glass-panel" style="margin-top: 20px; padding: 1.5rem;">
                    <h4 style="color: var(--primary); margin-bottom: 10px;"><i class="fas fa-brain"></i> AI Инсайт</h4>
                    <p style="font-size: 0.9rem; color: #cbd5e1; line-height: 1.6;">
                        "Мои алгоритмы фиксируют рекордную привлекательность региона. Основной драйвер — экономическая стабильность Алматы."
                    </p>
                </div>
            </aside>

            <main>
                <div style="text-align: center;">
                    <nav class="nav-pills">
                        <button class="nav-item active" data-tab="general"><i class="fas fa-chart-line"></i> Главная</button>
                        <button class="nav-item" data-tab="external"><i class="fas fa-globe"></i> Страны</button>
                        <button class="nav-item" data-tab="internal"><i class="fas fa-map-marked-alt"></i> Районы</button>
                        <button class="nav-item" data-tab="flows"><i class="fas fa-exchange-alt"></i> Потоки</button>
                    </nav>
                </div>

                <section id="general" class="section active">
                    <div class="kpi-grid">
                        <div class="glass-panel kpi-card">
                            <span class="kpi-sub">Сальдо 2025</span>
                            <div class="kpi-val">+5,210</div>
                            <span class="kpi-sub trend-up"><i class="fas fa-arrow-up"></i> Максимум за 10 лет</span>
                        </div>
                        <div class="glass-panel kpi-card">
                            <span class="kpi-sub">Население агломерации</span>
                            <div class="kpi-val">2.3M</div>
                            <span class="kpi-sub" style="color: var(--primary);">Активный рост</span>
                        </div>
                        <div class="glass-panel kpi-card">
                            <span class="kpi-sub">Экономический эффект</span>
                            <div class="kpi-val">+4.2%</div>
                            <span class="kpi-sub trend-up">К ВРП региона</span>
                        </div>
                    </div>

                    <div class="glass-panel chart-box">
                        <div class="chart-header">
                            <h3>Динамика миграционного сальдо</h3>
                        </div>
                        <canvas id="general-chart"></canvas>
                    </div>

                    <div class="insights-row">
                        <div class="glass-panel insight-box">
                            <div class="insight-title"><i class="fas fa-home"></i> Нагрузка на жилье</div>
                            <p style="font-size: 0.9rem; color: var(--text-muted);">Прогноз спроса на основе миграции</p>
                            <div style="margin-top: 15px;">
                                <div style="display:flex; justify-content:space-between; font-size: 0.8rem; margin-bottom: 5px;">
                                    <span>Спрос на аренду</span>
                                    <span style="color: var(--accent);">Высокий (85%)</span>
                                </div>
                                <div class="insight-bar-bg"><div class="insight-bar-fill" style="width: 85%; background: var(--accent);"></div></div>
                            </div>
                        </div>
                        <div class="glass-panel insight-box">
                            <div class="insight-title"><i class="fas fa-briefcase"></i> Рынок труда</div>
                            <p style="font-size: 0.9rem; color: var(--text-muted);">Приток квалифицированных кадров</p>
                            <div style="margin-top: 15px;">
                                <div style="display:flex; justify-content:space-between; font-size: 0.8rem; margin-bottom: 5px;">
                                    <span>Заполнение вакансий</span>
                                    <span style="color: var(--primary);">Среднее (60%)</span>
                                </div>
                                <div class="insight-bar-bg"><div class="insight-bar-fill" style="width: 60%; background: var(--primary);"></div></div>
                            </div>
                        </div>
                    </div>
                </section>

                <section id="external" class="section">
                    <div class="glass-panel chart-box">
                        <div class="chart-header">
                            <h3><i class="fas fa-globe"></i> Внешняя миграция (Топ стран)</h3>
                            <div class="controls-wrapper">
                                <select id="yearSelect" style="background: rgba(0,0,0,0.3); color: white; border: 1px solid var(--glass-border); padding: 5px 15px; border-radius: 8px;">
                                    <option value="2025">2025</option>
                                    <option value="2024">2024</option>
                                    <option value="2023">2023</option>
                                </select>
                            </div>
                        </div>
                        <canvas id="external-chart"></canvas>
                    </div>
                    <div id="country-insights" class="kpi-grid"></div>
                </section>

                <section id="internal" class="section">
                    <div class="glass-panel chart-box" style="height: auto; padding: 1.5rem;">
                        <h3 style="margin-bottom: 1rem;">Карта внутренней миграции</h3>
                        <div id="map"></div>
                    </div>
                    <div class="glass-panel" style="padding: 1.5rem; margin-top: 2rem;">
                        <h3>Рейтинг районов по привлекательности</h3>
                        <table style="width: 100%; margin-top: 1rem; border-collapse: collapse;">
                            <thead>
                                <tr style="color: var(--text-muted); border-bottom: 1px solid var(--glass-border);">
                                    <th style="text-align: left; padding: 10px;">Район</th>
                                    <th style="text-align: right; padding: 10px;">Сальдо</th>
                                    <th style="text-align: right; padding: 10px;">Тренд</th>
                                </tr>
                            </thead>
                            <tbody id="districts-table"></tbody>
                        </table>
                    </div>
                </section>

                <section id="flows" class="section">
                    <div class="glass-panel chart-box">
                        <div class="chart-header">
                            <h3>Потоки: Прибытие vs Выбытие</h3>
                            <div style="display: flex; gap: 15px;">
                                <div class="controls-wrapper">
                                    <button class="control-btn view-btn active" data-view="lines"><i class="fas fa-chart-line"></i></button>
                                    <button class="control-btn view-btn" data-view="bars"><i class="fas fa-chart-bar"></i></button>
                                </div>
                                <div class="controls-wrapper">
                                    <button class="control-btn period-btn active" data-period="all">Все</button>
                                    <button class="control-btn period-btn" data-period="recent">10 лет</button>
                                    <button class="control-btn period-btn" data-period="five">5 лет</button>
                                </div>
                            </div>
                        </div>
                        <canvas id="flows-chart"></canvas>
                        <p style="text-align: center; color: var(--text-muted); font-size: 0.8rem; margin-top: 10px;">
                            <i class="fas fa-mouse-pointer"></i> Нажмите на легенду графика (Прибывшие/Выбывшие), чтобы включить/выключить линии
                        </p>
                    </div>
                </section>
            </main>
        </div>
    </div>

    <button class="chat-toggle-btn" onclick="toggleChat()"><i class="fas fa-comment"></i></button>
    
    <div class="glass-panel chat-widget" id="chat-window">
        <div class="chat-header">
            <video autoplay muted loop class="chat-avatar-small">
                <source src="IMG_5131.MP4" type="video/mp4">
            </video>
            <div>
                <h3 style="margin: 0; font-size: 1.1rem;">Алма</h3>
                <span style="font-size: 0.75rem; color: var(--accent);">● Онлайн</span>
            </div>
            <button onclick="toggleChat()" style="margin-left: auto; background: none; border: none; color: white; cursor: pointer;"><i class="fas fa-times"></i></button>
        </div>
        
        <div class="chat-body" id="chat-messages">
            </div>
        
        <div class="chat-footer">
            <div class="input-group">
                <input type="text" id="chat-input" placeholder="Спросите меня о миграции...">
                <button class="send-btn" id="send-btn"><i class="fas fa-paper-plane"></i></button>
            </div>
        </div>
    </div>

    <script>
        // --- DATA PRESERVATION ---
        const migrationBalanceData = [
            { year: 2000, value: -6151 }, { year: 2001, value: -4344 }, { year: 2002, value: -1116 },
            { year: 2003, value: 3205 }, { year: 2004, value: 5101 }, { year: 2005, value: 5682 },
            { year: 2006, value: 4886 }, { year: 2007, value: 4633 }, { year: 2008, value: 2419 },
            { year: 2009, value: 2501 }, { year: 2010, value: 2466 }, { year: 2011, value: 2592 },
            { year: 2012, value: 973 }, { year: 2013, value: 2432 }, { year: 2014, value: 851 },
            { year: 2015, value: 2290 }, { year: 2016, value: 2571 }, { year: 2017, value: 3593 },
            { year: 2018, value: 1010 }, { year: 2019, value: 973 }, { year: 2020, value: -166 },
            { year: 2021, value: -901 }, { year: 2022, value: 1011 }, { year: 2023, value: 1196 },
            { year: 2024, value: 4367 }, { year: 2025, value: 5210 }
        ];

        const countryMigrationData = {
            2025: [{ name: "Узбекистан", value: 4210 }, { name: "Китай", value: 543 }, { name: "Таджикистан", value: 289 }, { name: "Кыргызстан", value: 94 }, { name: "Россия", value: -112 }, { name: "Германия", value: -92 }, { name: "Туркменистан", value: 52 }, { name: "Грузия", value: 68 }, { name: "Азербайджан", value: 71 }, { name: "Турция", value: 45 }],
            2024: [{ name: "Узбекистан", value: 3540 }, { name: "Китай", value: 478 }, { name: "Таджикистан", value: 204 }, { name: "Кыргызстан", value: 71 }, { name: "Россия", value: -86 }, { name: "Германия", value: -78 }, { name: "Туркменистан", value: 38 }, { name: "Грузия", value: 49 }, { name: "Азербайджан", value: 54 }, { name: "Турция", value: 32 }],
            2023: [{ name: "Узбекистан", value: 1068 }, { name: "Китай", value: 141 }, { name: "Россия", value: -195 }, { name: "Таджикистан", value: 88 }, { name: "Германия", value: -75 }, { name: "Кыргызстан", value: 53 }, { name: "Туркменистан", value: 26 }, { name: "Монголия", value: 32 }, { name: "Грузия", value: 53 }, { name: "Азербайджан", value: 24 }]
        };

        const districtMigrationData = [
            { id: "karasay", name: "Карасайский", value: 2541, lat: 43.4, lng: 76.8 },
            { id: "talgarsky", name: "Талгарский", value: 754, lat: 43.3, lng: 77.2 },
            { id: "iliysky", name: "Илийский", value: 452, lat: 43.6, lng: 76.9 },
            { id: "zhambylysky", name: "Жамбылский", value: 723, lat: 43.0, lng: 76.5 },
            { id: "enbekshikazakh", name: "Енбекшиказахский", value: 289, lat: 43.5, lng: 77.8 },
            { id: "balhashsky", name: "Балхашский", value: -5, lat: 46.0, lng: 75.0 },
            { id: "alatau", name: "Алатау г.а.", value: 82, lat: 43.2, lng: 77.0 },
            { id: "kegen", name: "Кегенский", value: 2, lat: 43.0, lng: 79.2 },
            { id: "rayymbek", name: "Райымбекский", value: 3, lat: 42.8, lng: 78.5 },
            { id: "uygur", name: "Уйгурский", value: -3, lat: 43.4, lng: 80.0 },
            { id: "sarkand", name: "Саркандский", value: 1, lat: 45.0, lng: 79.9 },
            { id: "akkol", name: "Аккольский", value: 2, lat: 45.0, lng: 70.0 },
            { id: "koksuy", name: "Коксуйский", value: 1, lat: 44.0, lng: 78.0 },
            { id: "kerbulak", name: "Кербулакский", value: 0, lat: 44.5, lng: 78.8 },
            { id: "panfilov", name: "Панфиловский", value: 1, lat: 44.2, lng: 80.0 },
            { id: "eskeldi", name: "Ескельдинский", value: 0, lat: 45.0, lng: 78.0 },
            { id: "karatal", name: "Каратальский", value: 1, lat: 45.5, lng: 78.5 },
            { id: "aksu", name: "Аксуский", value: 0, lat: 45.0, lng: 76.0 },
            { id: "almaty", name: "Алматинская область", value: 5210, lat: 43.2, lng: 76.9 }
        ];

        const migrationFlowsData = [
            { year: 2000, прибывшие: 2783, выбывшие: 8934 }, { year: 2001, прибывшие: 4301, выбывшие: 8645 },
            { year: 2002, прибывшие: 6103, выбывшие: 7219 }, { year: 2003, прибывшие: 7771, выбывшие: 4566 },
            { year: 2004, прибывшие: 8538, выбывшие: 3437 }, { year: 2005, прибывшие: 7982, выбывшие: 2300 },
            { year: 2006, прибывшие: 6690, выбывшие: 1744 }, { year: 2007, прибывшие: 6777, выбывшие: 2144 },
            { year: 2008, прибывшие: 3786, выбывшие: 1367 }, { year: 2009, прибывшие: 3298, выбывшие: 797 },
            { year: 2010, прибывшие: 3212, выбывшие: 746 }, { year: 2011, прибывшие: 3463, выбывшие: 871 },
            { year: 2012, прибывшие: 1825, выбывшие: 852 }, { year: 2013, прибывшие: 3106, выбывшие: 674 },
            { year: 2014, прибывшие: 1319, выбывшие: 468 }, { year: 2015, прибывшие: 2623, выбывшие: 333 },
            { year: 2016, прибывшие: 3055, выбывшие: 484 }, { year: 2017, прибывшие: 3913, выбывшие: 320 },
            { year: 2018, прибывшие: 1494, выбывшие: 484 }, { year: 2019, прибывшие: 1586, выбывшие: 613 },
            { year: 2020, прибывшие: 634, выбывшие: 800 }, { year: 2021, прибывшие: 76, выбывшие: 977 },
            { year: 2022, прибывшие: 1901, выбывшие: 890 }, { year: 2023, прибывшие: 1693, выбывшие: 497 },
            { year: 2024, прибывшие: 4788, выбывшие: 421 }, { year: 2025, прибывшие: 5210, выбывшие: 438 }
        ];

        // --- GLOBAL SETTINGS ---
        Chart.defaults.color = '#94a3b8';
        Chart.defaults.borderColor = 'rgba(255, 255, 255, 0.1)';
        Chart.defaults.font.family = "'Outfit', sans-serif";

        let generalChart, externalChart, flowsChart, map;
        let currentViewMode = 'lines';
        let currentPeriod = 'all';

        // --- INITIALIZATION ---
        document.addEventListener('DOMContentLoaded', () => {
            initMusic();
            initTabs();
            initGeneralChart();
            initExternalSection();
            initFlowsSection();
            initMap();
            initChat();
        });

        // --- MUSIC LOGIC ---
        function initMusic() {
            const music = document.getElementById('bg-music');
            const btn = document.getElementById('music-toggle');
            const txt = document.getElementById('music-text');
            
            btn.addEventListener('click', () => {
                if(music.paused) {
                    music.play().then(() => {
                        btn.classList.add('playing');
                        txt.textContent = 'Пауза';
                    });
                } else {
                    music.pause();
                    btn.classList.remove('playing');
                    txt.textContent = 'Включить звук';
                }
            });
        }

        // --- TABS LOGIC ---
        function initTabs() {
            const buttons = document.querySelectorAll('.nav-item');
            const sections = document.querySelectorAll('.section');
            
            buttons.forEach(btn => {
                btn.addEventListener('click', () => {
                    buttons.forEach(b => b.classList.remove('active'));
                    btn.classList.add('active');
                    
                    const target = btn.getAttribute('data-tab');
                    sections.forEach(s => {
                        s.classList.remove('active');
                        if(s.id === target) s.classList.add('active');
                    });
                    
                    if(target === 'internal' && map) setTimeout(() => map.invalidateSize(), 300);
                });
            });
        }

        // --- CHART: GENERAL ---
        function initGeneralChart() {
            const ctx = document.getElementById('general-chart').getContext('2d');
            
            // Градиент
            const gradient = ctx.createLinearGradient(0, 0, 0, 400);
            gradient.addColorStop(0, 'rgba(59, 130, 246, 0.5)');
            gradient.addColorStop(1, 'rgba(59, 130, 246, 0)');

            generalChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: migrationBalanceData.map(d => d.year),
                    datasets: [{
                        label: 'Сальдо миграции',
                        data: migrationBalanceData.map(d => d.value),
                        borderColor: '#3b82f6',
                        backgroundColor: gradient,
                        borderWidth: 3,
                        pointBackgroundColor: '#fff',
                        pointBorderColor: '#3b82f6',
                        pointRadius: 4,
                        pointHoverRadius: 6,
                        fill: true,
                        tension: 0.4
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            backgroundColor: 'rgba(15, 23, 42, 0.9)',
                            titleColor: '#fff',
                            bodyColor: '#cbd5e1',
                            padding: 10,
                            borderColor: 'rgba(59, 130, 246, 0.3)',
                            borderWidth: 1,
                            displayColors: false
                        }
                    },
                    scales: {
                        y: { grid: { color: 'rgba(255,255,255,0.05)' } }
                    }
                }
            });
        }

        // --- SECTION: EXTERNAL ---
        function initExternalSection() {
            const selector = document.getElementById('yearSelect');
            selector.addEventListener('change', updateExternalChart);
            updateExternalChart();
        }

        function updateExternalChart() {
            const year = document.getElementById('yearSelect').value;
            const data = countryMigrationData[year] || countryMigrationData[2025];
            const sorted = [...data].sort((a,b) => b.value - a.value).slice(0, 10); // Top 10

            const ctx = document.getElementById('external-chart').getContext('2d');
            if(externalChart) externalChart.destroy();

            externalChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: sorted.map(d => d.name),
                    datasets: [{
                        label: 'Человек',
                        data: sorted.map(d => d.value),
                        backgroundColor: sorted.map(d => d.value >= 0 ? '#10b981' : '#ef4444'),
                        borderRadius: 6,
                        borderSkipped: false
                    }]
                },
                options: {
                    indexAxis: 'y',
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: { x: { grid: { color: 'rgba(255,255,255,0.05)' } } }
                }
            });

            // Обновление карточек под графиком
            const container = document.getElementById('country-insights');
            container.innerHTML = '';
            sorted.slice(0, 4).forEach(item => {
                const color = item.value >= 0 ? 'var(--accent)' : 'var(--danger)';
                const sign = item.value >= 0 ? '+' : '';
                container.innerHTML += `
                    <div class="glass-panel kpi-card">
                        <span class="kpi-sub">${item.name}</span>
                        <div class="kpi-val" style="font-size: 1.8rem; background: none; -webkit-text-fill-color: ${color}; color: ${color};">${sign}${item.value}</div>
                    </div>
                `;
            });
        }

        // --- SECTION: FLOWS (FIXED BUTTONS) ---
        function initFlowsSection() {
            // Исправленная логика кнопок
            const viewBtns = document.querySelectorAll('.view-btn');
            const periodBtns = document.querySelectorAll('.period-btn');

            viewBtns.forEach(btn => {
                btn.addEventListener('click', function() {
                    viewBtns.forEach(b => b.classList.remove('active')); // Убираем у всех
                    this.classList.add('active'); // Добавляем нажатой
                    currentViewMode = this.dataset.view;
                    updateFlowsChart();
                });
            });

            periodBtns.forEach(btn => {
                btn.addEventListener('click', function() {
                    periodBtns.forEach(b => b.classList.remove('active'));
                    this.classList.add('active');
                    currentPeriod = this.dataset.period;
                    updateFlowsChart();
                });
            });

            updateFlowsChart();
        }

        function updateFlowsChart() {
            let data = migrationFlowsData;
            if(currentPeriod === 'recent') data = data.slice(-10);
            if(currentPeriod === 'five') data = data.slice(-5);

            const ctx = document.getElementById('flows-chart').getContext('2d');
            if(flowsChart) flowsChart.destroy();

            flowsChart = new Chart(ctx, {
                type: currentViewMode === 'lines' ? 'line' : 'bar',
                data: {
                    labels: data.map(d => d.year),
                    datasets: [
                        {
                            label: 'Прибывшие',
                            data: data.map(d => d.прибывшие),
                            borderColor: '#3b82f6',
                            backgroundColor: currentViewMode === 'bar' ? '#3b82f6' : 'rgba(59, 130, 246, 0.1)',
                            tension: 0.4,
                            fill: currentViewMode === 'lines'
                        },
                        {
                            label: 'Выбывшие',
                            data: data.map(d => d.выбывшие),
                            borderColor: '#ef4444',
                            backgroundColor: currentViewMode === 'bar' ? '#ef4444' : 'rgba(239, 68, 68, 0.1)',
                            tension: 0.4,
                            fill: currentViewMode === 'lines'
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            display: true, // Включаем легенду, чтобы она была кликабельной
                            labels: { color: 'white', usePointStyle: true, padding: 20 },
                            onClick: function(e, legendItem, legend) {
                                const index = legendItem.datasetIndex;
                                const ci = legend.chart;
                                if (ci.isDatasetVisible(index)) {
                                    ci.hide(index);
                                    legendItem.hidden = true;
                                } else {
                                    ci.show(index);
                                    legendItem.hidden = false;
                                }
                            }
                        }
                    }
                }
            });
        }

        // --- MAP LOGIC ---
        function initMap() {
            map = L.map('map', { zoomControl: false }).setView([44.5, 78.0], 7);
            L.tileLayer('https://cartodb-basemaps-{s}.global.ssl.fastly.net/dark_all/{z}/{x}/{y}.png', {
                attribution: '&copy; OpenStreetMap &copy; CartoDB'
            }).addTo(map);

            const table = document.getElementById('districts-table');
            const sorted = [...districtMigrationData].sort((a,b) => b.value - a.value);

            sorted.forEach(d => {
                if(d.lat) {
                    const color = d.value > 0 ? '#10b981' : '#ef4444';
                    const radius = Math.min(Math.abs(d.value)/50 + 5, 25);
                    
                    L.circleMarker([d.lat, d.lng], {
                        color: color, fillColor: color, fillOpacity: 0.6, radius: radius, weight: 1
                    }).bindPopup(`<div style="color:black"><b>${d.name}</b><br>Сальдо: ${d.value}</div>`).addTo(map);

                    // Таблица
                    const tr = document.createElement('tr');
                    tr.innerHTML = `
                        <td style="padding: 10px;">${d.name}</td>
                        <td style="padding: 10px; text-align: right; color: ${color}; font-weight: bold;">${d.value > 0 ? '+' : ''}${d.value}</td>
                        <td style="padding: 10px; text-align: right;">${d.value > 0 ? '↗' : '↘'}</td>
                    `;
                    table.appendChild(tr);
                }
            });
        }

        // --- CHAT BOT (ALMA AI) ---
        function toggleChat() {
            document.getElementById('chat-window').classList.toggle('open');
        }

        function initChat() {
            const btn = document.getElementById('send-btn');
            const inp = document.getElementById('chat-input');
            const box = document.getElementById('chat-messages');

            function addMsg(text, type) {
                const div = document.createElement('div');
                div.className = `msg ${type}`;
                div.textContent = text;
                box.appendChild(div);
                box.scrollTop = box.scrollHeight;
            }

            addMsg("Сәлем! Я Алма. Я проанализировала все данные и готова ответить на вопросы. Спросите про 2025 год или прогнозы!", 'alma');

            function handleSend() {
                const txt = inp.value.trim();
                if(!txt) return;
                
                addMsg(txt, 'user');
                inp.value = '';

                // Имитация мышления Алмы
                setTimeout(() => {
                    let reply = "Извини, я пока учусь понимать сложные вопросы. Попробуй спросить про сальдо или страны.";
                    const lower = txt.toLowerCase();

                    if(lower.includes('сальдо') || lower.includes('2025')) {
                        reply = "В 2025 году я фиксирую рекордное положительное сальдо: +5,210 человек! Это лучший показатель за 10 лет.";
                    } else if (lower.includes('прогноз') || lower.includes('будущее')) {
                        reply = "Мои алгоритмы предсказывают рост до +5,800 человек в 2026 году. Основной магнит — развитие инфраструктуры Алматинской агломерации.";
                    } else if (lower.includes('страны') || lower.includes('откуда')) {
                        reply = "Лидер по притоку — Узбекистан (+4,210). Также растет интерес со стороны граждан Китая (+543).";
                    } else if (lower.includes('район')) {
                        reply = "Самый привлекательный район — Карасайский (+2,541). Он активно застраивается и находится близко к городу.";
                    }

                    addMsg(reply, 'alma');
                }, 800);
            }

            btn.addEventListener('click', handleSend);
            inp.addEventListener('keypress', (e) => { if(e.key === 'Enter') handleSend(); });
        }
    </script>
</body>
</html>
