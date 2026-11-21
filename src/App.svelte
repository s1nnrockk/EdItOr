<script lang="ts">
  import { fade, fly } from "svelte/transition";
  import {
    ArrowRight, Monitor, Smartphone, Tablet, Globe2, Blocks, Command,
    FolderOpen, Settings2, Sparkles, Check, Layers3, ChevronLeft,
    Code2, Palette, Search, Play, Save, Zap, PanelLeftClose
  } from "lucide-svelte";
  import BlockEditor from "./BlockEditor.svelte";

  type Platform = "desktop" | "phone" | "tablet" | "web";
  type Page = "setup" | "editor" | "themes";
  type ThemeId = "material" | "fluent" | "ios" | "carbon";

  let selected = $state<Platform>("desktop");
  let projectName = $state("");
  let page = $state<Page>("setup");
  let selectedTheme = $state<ThemeId>("material");
  let search = $state("");
  let previewOpen = $state(false);

  const platforms = [
    { id: "desktop" as const, title: "Компьютер", subtitle: "Windows, macOS и Linux", detail: "Десктопные приложения", icon: Monitor, tag: "ПК" },
    { id: "phone" as const, title: "Телефон", subtitle: "Android и iOS", detail: "Мобильные приложения", icon: Smartphone, tag: "MOBILE" },
    { id: "tablet" as const, title: "Планшет", subtitle: "iPadOS и Android", detail: "Интерфейсы для большого экрана", icon: Tablet, tag: "TABLET" },
    { id: "web" as const, title: "Веб-приложение", subtitle: "Любой современный браузер", detail: "Сайты и веб-сервисы", icon: Globe2, tag: "WEB" }
  ];

  const themes = [
    { id: "material" as const, name: "Material 3", maker: "Google", description: "Адаптивные поверхности, выразительные цвета и динамические формы.", colors: ["#6750A4", "#D0BCFF", "#F7F2FA", "#1D1B20"], label: "Рекомендуем", type: "Компонентная система" },
    { id: "fluent" as const, name: "Fluent 2", maker: "Microsoft", description: "Чёткая типографика, спокойные поверхности и понятные состояния.", colors: ["#0F6CBD", "#479EF5", "#F5F5F5", "#242424"], label: "Для Windows", type: "Компонентная система" },
    { id: "ios" as const, name: "iOS 26", maker: "Apple", description: "Лёгкие слои, прозрачность и аккуратные системные элементы.", colors: ["#087AFF", "#70B7FF", "#F5F7FA", "#17191D"], label: "Для Apple", type: "Визуальный стиль" },
    { id: "carbon" as const, name: "Carbon", maker: "IBM", description: "Строгая сетка, функциональные компоненты и доступность.", colors: ["#0F62FE", "#A6C8FF", "#F4F4F4", "#161616"], label: "Для продуктов", type: "Компонентная система" }
  ];

  const selectedPlatform = $derived(platforms.find((item) => item.id === selected)!);
  const filteredThemes = $derived(themes.filter((theme) => (theme.name + " " + theme.maker).toLowerCase().includes(search.toLowerCase())));
  const activeTheme = $derived(themes.find((theme) => theme.id === selectedTheme)!);

  function beginProject() {
    page = "editor";
  }

  function navTo(next: Page) {
    page = next;
  }
</script>

<svelte:head>
  <title>EdItOr Studio — визуальная среда разработки</title>
  <meta name="description" content="Создавай приложения с помощью визуального блочного редактора и библиотек интерфейсов." />
</svelte:head>

<div class="app-shell">
  <aside class="sidebar">
    <a class="brand" href="/" aria-label="EdItOr Studio">
      <span class="brand-mark"><Blocks size={19} strokeWidth={2.2} /></span>
      <span class="brand-name">EdItOr<span class="brand-period">.</span></span>
    </a>

    <div class="sidebar-section">
      <div class="section-label">РАБОЧЕЕ ПРОСТРАНСТВО</div>
      <button class:active={page === "setup"} class="side-link" onclick={() => navTo("setup")}><span class="side-icon"><Command size={17}/></span> Новый проект</button>
      <button class:active={page === "editor"} class="side-link" onclick={() => page === "setup" ? beginProject() : navTo("editor")}><span class="side-icon"><Blocks size={17}/></span> Блочный редактор</button>
      <button class:active={page === "themes"} class="side-link" onclick={() => navTo("themes")}><span class="side-icon"><Layers3 size={17}/></span> Библиотека тем</button>
      <button class="side-link" disabled><span class="side-icon"><FolderOpen size={17}/></span> Мои проекты <span class="soon">Скоро</span></button>
    </div>

    <div class="sidebar-bottom">
      <button class="side-link" disabled><span class="side-icon"><Settings2 size={17}/></span> Настройки <span class="soon">Скоро</span></button>
      <div class="sidebar-version"><span class="status-dot"></span><span>EdItOr Studio</span><span class="version">0.2.0</span></div>
    </div>
  </aside>

  <main class="main-area">
    <header class="topbar">
      <div class="breadcrumb"><span>Рабочее пространство</span><span class="crumb-separator">/</span><strong>{page === "setup" ? "Новый проект" : page === "editor" ? "Блочный редактор" : "Библиотека тем"}</strong></div>
      <div class="topbar-right"><span class="local-status"><span class="status-dot"></span> Локальный проект</span><div class="avatar">E</div></div>
    </header>

    {#if page === "setup"}
      <div class="content" in:fly={{ y: 8, duration: 240 }}>
        <div class="intro">
          <div class="eyebrow"><span class="eyebrow-line"></span> ТВОРЧЕСКАЯ СРЕДА</div>
          <h1>С чего начнём?</h1>
          <p class="intro-copy">Выбери платформу, для которой хочешь создать приложение.<br class="desktop-break"/> Остальное настроим на следующем шаге.</p>
        </div>
        <section class="platform-section">
          <div class="section-heading">
            <div><h2>Целевая платформа</h2><p>Можно будет изменить настройки позже.</p></div>
            <span class="step-indicator"><span>01</span> / 03</span>
          </div>
          <div class="platform-grid">
            {#each platforms as platform}
              <button class:chosen={selected === platform.id} class="platform-card" onclick={() => selected = platform.id} aria-pressed={selected === platform.id}>
                <div class="card-top">
                  <div class="platform-icon"><svelte:component this={platform.icon} size={23} strokeWidth={1.7}/></div>
                  <span class="platform-tag">{platform.tag}</span>
                </div>
                <div class="card-copy"><h3>{platform.title}</h3><p>{platform.subtitle}</p></div>
                <div class="card-footer"><span>{platform.detail}</span><span class="selection-indicator">{#if selected === platform.id}<Check size={14} strokeWidth={2.5}/>{/if}</span></div>
              </button>
            {/each}
          </div>
        </section>
        <section class="project-section">
          <div class="section-heading"><div><h2>Название проекта <span class="optional">НЕОБЯЗАТЕЛЬНО</span></h2><p>Дай проекту имя — его можно поменять позже.</p></div></div>
          <div class="name-field"><input bind:value={projectName} maxlength="40" placeholder="Например, MyFirstApp" aria-label="Название проекта" /><span class="field-hint">.editor</span></div>
        </section>
        <div class="bottom-actions">
          <div class="selection-summary"><span class="summary-icon"><svelte:component this={selectedPlatform.icon} size={17}/></span><span><strong>{selectedPlatform.title}</strong><small>Выбранная платформа</small></span></div>
          <button class="primary-button" onclick={beginProject}>Открыть редактор <ArrowRight size={17}/></button>
        </div>
        <footer class="page-footer"><span>Создавай без ограничений.</span><span>EdItOr Studio <span class="footer-divider">·</span> Ранний доступ</span></footer>
      </div>
    {:else if page === "editor"}
      <div class="editor-page" in:fade={{ duration: 180 }}>
        <div class="editor-titlebar">
          <div class="editor-project">
            <button class="back-icon" onclick={() => navTo("setup")} title="К выбору платформы"><ChevronLeft size={17}/></button>
            <div><strong>{projectName.trim() || "Моё приложение"}</strong><span>{selectedPlatform.title} <i>·</i> {activeTheme.name}</span></div>
            <span class="saved-pill"><span></span> Черновик</span>
          </div>
          <div class="editor-title-actions">
            <button class="quiet-button" onclick={() => navTo("themes")}><Palette size={15}/> Темы</button>
            <button class="quiet-button" onclick={() => previewOpen = !previewOpen}><Play size={14}/> Предпросмотр</button>
            <button class="save-button" onclick={() => { try { localStorage.setItem("editor-project-name", projectName.trim() || "Моё приложение"); } catch {} }}><Save size={14}/> Сохранить</button>
          </div>
        </div>
        <div class="editor-welcome">
          <div><span class="eyebrow"><span class="eyebrow-line"></span> ВИЗУАЛЬНОЕ ПРОГРАММИРОВАНИЕ</span><h1>Собирай логику <span>из блоков.</span></h1><p>Перетаскивай блоки из категорий слева и соединяй их на рабочем поле.</p></div>
          <div class="editor-tip"><Zap size={16}/><span>Изменения кода<br/><strong>обновляются автоматически</strong></span></div>
        </div>
        <BlockEditor />
        {#if previewOpen}
          <div class="preview-toast" transition:fly={{ y: 12, duration: 180 }}><div><Play size={15}/><strong>Предпросмотр</strong></div><p>Сейчас доступен просмотр сгенерированного JavaScript в панели справа. Полноценный запуск UI-компонентов добавим следующим этапом.</p><button onclick={() => previewOpen = false}>Понятно</button></div>
        {/if}
      </div>
    {:else}
      <div class="themes-page" in:fly={{ y: 8, duration: 240 }}>
        <div class="themes-heading">
          <div><span class="eyebrow"><span class="eyebrow-line"></span> DESIGN SYSTEMS</span><h1>Библиотека <span>стилей.</span></h1><p>Выбирай основу интерфейса. Компоненты и токены подключим к предпросмотру на следующем этапе.</p></div>
          <div class="theme-count"><strong>{themes.length.toString().padStart(2, "0")}</strong><span>БИБЛИОТЕКИ</span></div>
        </div>
        <label class="theme-search"><Search size={16}/><input bind:value={search} placeholder="Найти библиотеку..." /><kbd>⌕</kbd></label>
        <div class="theme-grid">
          {#each filteredThemes as theme, i}
            <button class:theme-selected={selectedTheme === theme.id} class="theme-card" onclick={() => selectedTheme = theme.id} in:fly={{ y: 12, delay: i * 45, duration: 220 }}>
              <div class="theme-preview" style="--c1:{theme.colors[0]};--c2:{theme.colors[1]};--c3:{theme.colors[2]};--c4:{theme.colors[3]}">
                <div class="preview-mini-window"><div class="mini-sidebar"></div><div class="mini-content"><span></span><span></span><div class="mini-button"></div><div class="mini-card"></div></div></div>
                <span class="theme-type">{theme.type}</span>
              </div>
              <div class="theme-card-body"><div class="theme-name-row"><div><h3>{theme.name}</h3><span>{theme.maker}</span></div><span class="theme-radio">{#if selectedTheme === theme.id}<Check size={13}/>{/if}</span></div><p>{theme.description}</p><div class="swatches">{#each theme.colors as color}<i style="background:{color}"></i>{/each}<span>{theme.label}</span></div></div>
            </button>
          {/each}
        </div>
        {#if filteredThemes.length === 0}<div class="empty-themes"><Search size={20}/><strong>Ничего не найдено</strong><span>Попробуй другое название.</span></div>{/if}
        <div class="theme-bottom"><div><span class="active-theme-dot"></span><span>Выбрана библиотека <strong>{activeTheme.name}</strong></span></div><button class="primary-button" onclick={() => navTo("editor")}>Применить стиль <ArrowRight size={16}/></button></div>
      </div>
    {/if}
  </main>
</div>