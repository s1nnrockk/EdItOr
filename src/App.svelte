<script lang="ts">
  import { fade, fly } from "svelte/transition";
  import {
    ArrowRight, Monitor, Smartphone, Tablet, Globe2, Blocks, Command,
    FolderOpen, Settings2, Sparkles, Check, Layers3, ChevronLeft,
    Code2, Palette, Search, Play, Save, Zap, PanelLeftClose
  } from "lucide-svelte";
  import BlockEditor from "./BlockEditor.svelte";
  import CodeEditor from "./CodeEditor.svelte";
  import Repository from "./Repository.svelte";

  type Platform = "desktop" | "phone" | "tablet" | "web";
  type Page = "setup" | "editor" | "themes" | "repo";
  type ThemeId = string;

  let selected = $state<Platform>("desktop");
  let projectName = $state("");
  let page = $state<Page>("setup");
  let selectedTheme = $state<ThemeId>("material");
  let search = $state("");
  let previewOpen = $state(false);
  let editorMode = $state<"blocks" | "code">("blocks");
  let sourceCode = $state("const app = document.querySelector(\"#app\");\n\napp.innerHTML = \"<h1>Hello from EdItOr</h1>\";");

  const platforms = [
    { id: "desktop" as const, title: "Компьютер", subtitle: "Windows, macOS и Linux", detail: "Десктопные приложения", icon: Monitor, tag: "ПК" },
    { id: "phone" as const, title: "Телефон", subtitle: "Android и iOS", detail: "Мобильные приложения", icon: Smartphone, tag: "MOBILE" },
    { id: "tablet" as const, title: "Планшет", subtitle: "iPadOS и Android", detail: "Интерфейсы для большого экрана", icon: Tablet, tag: "TABLET" },
    { id: "web" as const, title: "Веб-приложение", subtitle: "Любой современный браузер", detail: "Сайты и веб-сервисы", icon: Globe2, tag: "WEB" }
  ];

  const themes = [
    { id: "material", name: "Material 3", maker: "Google", description: "Material You, динамические цвета, адаптивные компоненты и motion.", colors: ["#6750A4","#D0BCFF","#F7F2FA","#1D1B20"], label: "Android · Web", type: "Design system", url: "https://m3.material.io/" },
    { id: "fluent", name: "Fluent 2", maker: "Microsoft", description: "Система Microsoft для Windows и кроссплатформенных продуктов.", colors: ["#0F6CBD","#479EF5","#F5F5F5","#242424"], label: "Windows", type: "Design system", url: "https://fluent2.microsoft.design/" },
    { id: "apple-hig", name: "Apple HIG", maker: "Apple", description: "Официальные принципы интерфейсов iOS, iPadOS и macOS.", colors: ["#087AFF","#70B7FF","#F5F7FA","#17191D"], label: "Apple platforms", type: "Guidelines", url: "https://developer.apple.com/design/human-interface-guidelines/" },
    { id: "carbon", name: "Carbon", maker: "IBM", description: "Открытая система IBM: компоненты, сетка, токены и доступность.", colors: ["#0F62FE","#A6C8FF","#F4F4F4","#161616"], label: "Enterprise", type: "Design system", url: "https://carbondesignsystem.com/" },
    { id: "ant", name: "Ant Design", maker: "Ant Group", description: "Большой набор компонентов для админок и бизнес-приложений.", colors: ["#1677FF","#91CAFF","#F5F8FF","#1F1F1F"], label: "React", type: "Component library", url: "https://ant.design/" },
    { id: "mui", name: "MUI", maker: "MUI", description: "React-компоненты с реализацией принципов Material Design.", colors: ["#1976D2","#90CAF9","#F5F7FA","#17212B"], label: "React", type: "Component library", url: "https://mui.com/" },
    { id: "chakra", name: "Chakra UI", maker: "Chakra UI", description: "Доступные composable-компоненты и удобная система темизации.", colors: ["#319795","#81E6D9","#F0FFF4","#1A202C"], label: "React", type: "Component library", url: "https://chakra-ui.com/" },
    { id: "mantine", name: "Mantine", maker: "Mantine", description: "React-компоненты, хуки, формы и готовые паттерны.", colors: ["#228BE6","#A5D8FF","#F1F8FF","#20242B"], label: "React", type: "Component library", url: "https://mantine.dev/" },
    { id: "shadcn", name: "shadcn/ui", maker: "shadcn", description: "Компоненты и исходный код прямо внутри твоего проекта.", colors: ["#18181B","#A1A1AA","#FAFAFA","#18181B"], label: "React · Tailwind", type: "Open source components", url: "https://ui.shadcn.com/" },
    { id: "radix", name: "Radix UI", maker: "WorkOS", description: "Низкоуровневые доступные примитивы для собственной дизайн-системы.", colors: ["#6E56CF","#D4C8FF","#FAF9FF","#1C1B22"], label: "React", type: "Primitives", url: "https://www.radix-ui.com/" },
    { id: "bootstrap", name: "Bootstrap", maker: "Bootstrap team", description: "Адаптивная сетка, утилиты и готовые веб-компоненты.", colors: ["#7952B3","#C5B3E6","#F8F7FC","#212529"], label: "Web", type: "CSS framework", url: "https://getbootstrap.com/" },
    { id: "tailwind", name: "Tailwind CSS", maker: "Tailwind Labs", description: "Utility-first подход для сборки собственных интерфейсов.", colors: ["#06B6D4","#A5F3FC","#ECFEFF","#164E63"], label: "Web", type: "CSS framework", url: "https://tailwindcss.com/" },
    { id: "bulma", name: "Bulma", maker: "Bulma", description: "Лёгкий CSS-фреймворк с понятными классами и flexbox-сеткой.", colors: ["#00D1B2","#9FFFEA","#F2FFFC","#25333C"], label: "Web", type: "CSS framework", url: "https://bulma.io/" },
    { id: "vuetify", name: "Vuetify", maker: "Vuetify", description: "Material-компоненты и инструменты для приложений на Vue.", colors: ["#1867C0","#A3C8F0","#F5F9FF","#20252B"], label: "Vue", type: "Component framework", url: "https://vuetifyjs.com/" },
    { id: "quasar", name: "Quasar", maker: "Quasar", description: "Vue-фреймворк для SPA, мобильных и desktop-приложений.", colors: ["#1976D2","#90CAF9","#F5F8FC","#263238"], label: "Vue · Multi-platform", type: "App framework", url: "https://quasar.dev/" },
    { id: "ionic", name: "Ionic", maker: "Ionic", description: "Мобильные UI-компоненты для web, iOS и Android.", colors: ["#3880FF","#B5D0FF","#F4F8FF","#222428"], label: "Mobile · Web", type: "Component library", url: "https://ionicframework.com/" },
    { id: "paper", name: "React Native Paper", maker: "Callstack", description: "Material-компоненты для нативных приложений React Native.", colors: ["#6200EE","#BB86FC","#F7F2FA","#1C1B1F"], label: "React Native", type: "Component library", url: "https://reactnativepaper.com/" },
    { id: "nativewind", name: "NativeWind", maker: "NativeWind", description: "Tailwind-подобная стилизация React Native.", colors: ["#38BDF8","#BAE6FD","#F0F9FF","#0C4A6E"], label: "React Native", type: "Styling system", url: "https://www.nativewind.dev/" },
    { id: "spectrum", name: "Adobe Spectrum", maker: "Adobe", description: "Дизайн-система Adobe с акцентом на согласованность и доступность.", colors: ["#1473E6","#B8D8FF","#F8F8F8","#292929"], label: "Creative tools", type: "Design system", url: "https://spectrum.adobe.com/" },
    { id: "polaris", name: "Shopify Polaris", maker: "Shopify", description: "Компоненты и паттерны для коммерческих продуктов.", colors: ["#008060","#AEE9D1","#F6F6F7","#202223"], label: "Commerce", type: "Design system", url: "https://polaris.shopify.com/" }
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
      <button class:active={page === "themes"} class="side-link" onclick={() => navTo("themes")}><span class="side-icon"><Layers3 size={17}/></span> Библиотека</button>
      <button class:active={page === "repo"} class="side-link" onclick={() => navTo("repo")}><span class="side-icon"><FolderOpen size={17}/></span> Репозиторий</button>
      <button class="side-link" disabled><span class="side-icon"><FolderOpen size={17}/></span> Мои проекты <span class="soon">Скоро</span></button>
    </div>

    <div class="sidebar-bottom">
      <button class="side-link" disabled><span class="side-icon"><Settings2 size={17}/></span> Настройки <span class="soon">Скоро</span></button>
      <div class="sidebar-version"><span class="status-dot"></span><span>EdItOr Studio</span><span class="version">0.2.0</span></div>
    </div>
  </aside>

  <main class="main-area">
    <header class="topbar">
      <div class="breadcrumb"><span>Рабочее пространство</span><span class="crumb-separator">/</span><strong>{page === "setup" ? "Новый проект" : page === "editor" ? "Редактор" : page === "repo" ? "Репозиторий проекта" : "Библиотека тем"}</strong></div>
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
                  <div class="platform-icon"><platform.icon size={23} strokeWidth={1.7}/></div>
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
          <div class="selection-summary"><span class="summary-icon"><selectedPlatform.icon size={17}/></span><span><strong>{selectedPlatform.title}</strong><small>Выбранная платформа</small></span></div>
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
            <button class="quiet-button" onclick={() => navTo("repo")}><FolderOpen size={15}/> Репозиторий</button>
            <button class="quiet-button" onclick={() => navTo("themes")}><Palette size={15}/> Темы</button>
            <div class="mode-switch"><button class:mode-active={editorMode === "blocks"} onclick={() => editorMode = "blocks"}><Blocks size={13}/> Блоки</button><button class:mode-active={editorMode === "code"} onclick={() => editorMode = "code"}><Code2 size={13}/> Код</button></div>
            <button class="quiet-button" onclick={() => previewOpen = !previewOpen}><Play size={14}/> Предпросмотр</button>
            <button class="save-button" onclick={() => { try { localStorage.setItem("editor-project-name", projectName.trim() || "Моё приложение"); } catch {} }}><Save size={14}/> Сохранить</button>
          </div>
        </div>
        <div class="editor-welcome">
          <div><span class="eyebrow"><span class="eyebrow-line"></span> ВИЗУАЛЬНОЕ ПРОГРАММИРОВАНИЕ</span><h1>Собирай логику <span>из блоков.</span></h1><p>Перетаскивай блоки из категорий слева и соединяй их на рабочем поле.</p></div>
          <div class="editor-tip"><Zap size={16}/><span>Изменения кода<br/><strong>обновляются автоматически</strong></span></div>
        </div>
        {#if editorMode === "blocks"}
          <BlockEditor />
        {:else}
          <div class="source-workspace"><div class="source-head"><div><Code2 size={15}/><strong>main.js</strong><span>JavaScript · редактирование</span></div><button class="source-copy" onclick={() => navigator.clipboard.writeText(sourceCode)}>Копировать код</button></div><div class="source-body"><CodeEditor value={sourceCode} language="javascript" onChange={(value) => sourceCode = value}/></div></div>
        {/if}
        {#if previewOpen}
          <div class="preview-toast" transition:fly={{ y: 12, duration: 180 }}><div><Play size={15}/><strong>Предпросмотр</strong></div><p>Сейчас доступен просмотр сгенерированного JavaScript в панели справа. Полноценный запуск UI-компонентов добавим следующим этапом.</p><button onclick={() => previewOpen = false}>Понятно</button></div>
        {/if}
      </div>
    {:else if page === "repo"}
      <div class="repo-page" in:fly={{ y: 8, duration: 220 }}><div class="repo-page-intro"><span class="eyebrow"><span class="eyebrow-line"></span> PROJECT SOURCE CONTROL</span><h1>Твой проект. <span>Твоя история.</span></h1><p>Файлы, исходный код и локальные коммиты в одном месте.</p></div><Repository /></div>
    {:else}
      <div class="themes-page" in:fly={{ y: 8, duration: 240 }}>
        <div class="themes-heading">
          <div><span class="eyebrow"><span class="eyebrow-line"></span> DESIGN SYSTEMS</span><h1>Библиотека <span>стилей.</span></h1><p>Реальные дизайн-системы и UI-библиотеки. Изучай назначение и открывай официальную документацию.</p></div>
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
              <div class="theme-card-body"><div class="theme-name-row"><div><h3>{theme.name}</h3><span>{theme.maker}</span></div><span class="theme-radio">{#if selectedTheme === theme.id}<Check size={13}/>{/if}</span></div><p>{theme.description}</p><div class="swatches">{#each theme.colors as color}<i style="background:{color}"></i>{/each}<span>{theme.label}</span></div><a class="theme-doc-link" href={theme.url} target="_blank" rel="noreferrer" onclick={(event) => event.stopPropagation()}>Документация ↗</a></div>
            </button>
          {/each}
        </div>
        {#if filteredThemes.length === 0}<div class="empty-themes"><Search size={20}/><strong>Ничего не найдено</strong><span>Попробуй другое название.</span></div>{/if}
        <div class="theme-bottom"><div><span class="active-theme-dot"></span><span>Выбрана библиотека <strong>{activeTheme.name}</strong></span></div><button class="primary-button" onclick={() => navTo("editor")}>Применить стиль <ArrowRight size={16}/></button></div>
      </div>
    {/if}
  </main>
</div>