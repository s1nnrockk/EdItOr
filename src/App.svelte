<script lang="ts">
  import { ArrowRight, Monitor, Smartphone, Tablet, Globe2, Blocks, Command, FolderOpen, Settings2, Sparkles, Check } from "lucide-svelte";

  type Platform = "desktop" | "phone" | "tablet" | "web";
  let selected = $state<Platform>("desktop");
  let projectName = $state("");
  let started = $state(false);

  const platforms = [
    { id: "desktop" as const, title: "Компьютер", subtitle: "Windows, macOS и Linux", detail: "Десктопные приложения", icon: Monitor, tag: "ПК" },
    { id: "phone" as const, title: "Телефон", subtitle: "Android и iOS", detail: "Мобильные приложения", icon: Smartphone, tag: "MOBILE" },
    { id: "tablet" as const, title: "Планшет", subtitle: "iPadOS и Android", detail: "Интерфейсы для большого экрана", icon: Tablet, tag: "TABLET" },
    { id: "web" as const, title: "Веб-приложение", subtitle: "Любой современный браузер", detail: "Сайты и веб-сервисы", icon: Globe2, tag: "WEB" }
  ];

  const selectedPlatform = $derived(platforms.find((item) => item.id === selected)!);

  function beginProject() {
    started = true;
  }
</script>

<svelte:head>
  <title>EdItOr Studio — новый проект</title>
  <meta name="description" content="Создавай приложения с помощью визуального блочного редактора." />
</svelte:head>

<div class="app-shell">
  <aside class="sidebar">
    <a class="brand" href="/" aria-label="EdItOr Studio">
      <span class="brand-mark"><Blocks size={19} strokeWidth={2.2} /></span>
      <span class="brand-name">EdItOr<span class="brand-period">.</span></span>
    </a>

    <div class="sidebar-section">
      <div class="section-label">РАБОЧЕЕ ПРОСТРАНСТВО</div>
      <button class="side-link active"><span class="side-icon"><Command size={17}/></span> Новый проект</button>
      <button class="side-link" disabled><span class="side-icon"><FolderOpen size={17}/></span> Мои проекты <span class="soon">Скоро</span></button>
      <button class="side-link" disabled><span class="side-icon"><Blocks size={17}/></span> Библиотека <span class="soon">Скоро</span></button>
    </div>

    <div class="sidebar-bottom">
      <button class="side-link" disabled><span class="side-icon"><Settings2 size={17}/></span> Настройки</button>
      <div class="sidebar-version"><span class="status-dot"></span><span>EdItOr Studio</span><span class="version">0.1.0</span></div>
    </div>
  </aside>

  <main class="main-area">
    <header class="topbar">
      <div class="breadcrumb"><span>Рабочее пространство</span><span class="crumb-separator">/</span><strong>Новый проект</strong></div>
      <div class="topbar-right"><span class="local-status"><span class="status-dot"></span> Локальный проект</span><div class="avatar">S</div></div>
    </header>

    <div class="content">
      <div class="intro">
        <div class="eyebrow"><span class="eyebrow-line"></span> ТВОРЧЕСКАЯ СРЕДА</div>
        <h1>С чего начнём?</h1>
        <p class="intro-copy">Выбери платформу, для которой хочешь создать приложение.<br class="desktop-break"/> Остальное настроим на следующем шаге.</p>
      </div>

      {#if !started}
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
                <div class="card-copy">
                  <h3>{platform.title}</h3>
                  <p>{platform.subtitle}</p>
                </div>
                <div class="card-footer">
                  <span>{platform.detail}</span>
                  <span class="selection-indicator">{#if selected === platform.id}<Check size={14} strokeWidth={2.5}/>{/if}</span>
                </div>
              </button>
            {/each}
          </div>
        </section>

        <section class="project-section">
          <div class="section-heading">
            <div><h2>Название проекта <span class="optional">НЕОБЯЗАТЕЛЬНО</span></h2><p>Дай проекту имя — его можно поменять позже.</p></div>
          </div>
          <div class="name-field">
            <input bind:value={projectName} maxlength="40" placeholder="Например, MyFirstApp" aria-label="Название проекта" />
            <span class="field-hint">.editor</span>
          </div>
        </section>

        <div class="bottom-actions">
          <div class="selection-summary"><span class="summary-icon"><svelte:component this={selectedPlatform.icon} size={17}/></span><span><strong>{selectedPlatform.title}</strong><small>Выбранная платформа</small></span></div>
          <button class="primary-button" onclick={beginProject}>Продолжить <ArrowRight size={17}/></button>
        </div>
      {:else}
        <section class="next-step">
          <div class="success-mark"><Check size={25}/></div>
          <div class="eyebrow">ПЛАТФОРМА ВЫБРАНА</div>
          <h2>Отличное начало.</h2>
          <p>Проект «{projectName.trim() || "Новый проект"}» будет создан для платформы «{selectedPlatform.title}».</p>
          <div class="next-summary"><span>Платформа</span><strong>{selectedPlatform.title}</strong><span>Название</span><strong>{projectName.trim() || "Новый проект"}</strong></div>
          <button class="secondary-button" onclick={() => started = false}>Вернуться к выбору</button>
          <p class="coming-note">Следующий этап — настройки проекта и блочный редактор.</p>
        </section>
      {/if}

      <footer class="page-footer"><span>Создавай без ограничений.</span><span>EdItOr Studio <span class="footer-divider">·</span> Ранний доступ</span></footer>
    </div>
  </main>
</div>