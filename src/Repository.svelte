<script lang="ts">
  import { onMount } from "svelte";
  import CodeEditor from "./CodeEditor.svelte";
  import { FileCode2, FileJson, FileText, Plus, GitBranch, GitCommit, Check, Trash2, Download, Upload, X } from "lucide-svelte";

  type RepoFile = { path: string; content: string; language: string };
  type Commit = { id: string; message: string; time: string; files: number; snapshot?: RepoFile[] };
  const seedFiles: RepoFile[] = [
    { path: "index.html", language: "html", content: "<!doctype html>\n<html lang=\"ru\">\n  <head>\n    <meta charset=\"UTF-8\" />\n    <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\" />\n    <title>My App</title>\n    <link rel=\"stylesheet\" href=\"src/style.css\" />\n  </head>\n  <body>\n    <main id=\"app\"></main>\n    <script type=\"module\" src=\"src/main.js\"><\\/script>\n  </body>\n</html>" },
    { path: "src/main.js", language: "javascript", content: "const app = document.querySelector('#app');\n\napp.innerHTML = '<section class=\"welcome\"><h1>Привет, EdItOr.</h1><p>Начни создавать своё приложение.</p><button id=\"action\">Нажми меня</button></section>';\n\ndocument.querySelector('#action').addEventListener('click', () => {\n  alert('Работает!');\n});" },
    { path: "src/style.css", language: "css", content: ":root {\n  font-family: system-ui, sans-serif;\n  color: #202923;\n  background: #f4f6f2;\n}\n\nbody { margin: 0; }\n\n.welcome { max-width: 640px; margin: 12vh auto; padding: 32px; border-radius: 16px; background: white; }\n\nbutton { padding: 12px 18px; border: 0; border-radius: 8px; color: white; background: #315b42; cursor: pointer; }" },
    { path: "README.md", language: "markdown", content: "# My App\n\nПроект, созданный в EdItOr Studio.\n\n## Запуск\n\nОткройте index.html в браузере или запустите локальный dev-сервер." }
  ];

  let files = $state<RepoFile[]>(seedFiles);
  let selectedPath = $state("src/main.js");
  let activeTab = $state<"files" | "history">("files");
  let branch = $state("main");
  let commits = $state<Commit[]>([]);
  let commitMessage = $state("");
  let showCommit = $state(false);
  let newFileName = $state("");
  let showNewFile = $state(false);
  let dirty = $state(false);
  let notice = $state("");
  const currentFile = $derived(files.find((file) => file.path === selectedPath) ?? files[0]);
  const fileIcon = (path: string) => path.endsWith(".css") ? "css" : path.endsWith(".json") ? "json" : path.endsWith(".md") ? "md" : path.endsWith(".html") ? "html" : "code";

  onMount(() => {
    try {
      const saved = localStorage.getItem("editor-repository-v1");
      if (saved) {
        const parsed = JSON.parse(saved);
        if (Array.isArray(parsed.files) && parsed.files.length) files = parsed.files;
        if (Array.isArray(parsed.commits)) commits = parsed.commits;
        if (typeof parsed.branch === "string") branch = parsed.branch;
      }
    } catch { /* Invalid local state: keep starter files. */ }
  });
  function persist() {
    try { localStorage.setItem("editor-repository-v1", JSON.stringify({ files, commits, branch })); } catch {}
  }
  function updateContent(content: string) {
    files = files.map((file) => file.path === selectedPath ? { ...file, content } : file);
    dirty = true; persist();
  }
  function createFile() {
    const path = newFileName.trim().replace(/^\/+/, "");
    if (!path || files.some((file) => file.path === path)) return;
    const extension = path.split(".").pop() ?? "";
    const language = ({ js: "javascript", ts: "typescript", html: "html", css: "css", json: "json" } as Record<string,string>)[extension] ?? "javascript";
    files = [...files, { path, language, content: "" }];
    selectedPath = path; newFileName = ""; showNewFile = false; dirty = true; persist();
  }
  function removeFile(path: string) {
    if (files.length <= 1) return;
    files = files.filter((file) => file.path !== path);
    selectedPath = files[0].path; dirty = true; persist();
  }
  function commit() {
    const message = commitMessage.trim();
    if (!message) return;
    commits = [{ id: crypto.randomUUID().slice(0, 7), message, time: new Date().toLocaleString("ru-RU"), files: files.length, snapshot: files.map((file) => ({ ...file })) }, ...commits].slice(0, 40);
    commitMessage = ""; dirty = false; showCommit = false; persist();
  }
  function restoreCommit(commit: Commit) {
    if (!commit.snapshot?.length) return;
    files = commit.snapshot.map((file) => ({ ...file }));
    selectedPath = files[0]?.path ?? "";
    dirty = true;
    persist();
  }  function exportRepository() {
    const payload = { name: "editor-project", branch, files, commits };
    const blob = new Blob([JSON.stringify(payload, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const anchor = document.createElement("a"); anchor.href = url; anchor.download = "editor-repository.json"; anchor.click();
    URL.revokeObjectURL(url);
  }
  async function importRepository(event: Event) {
    const input = event.currentTarget as HTMLInputElement;
    const file = input.files?.[0]; if (!file) return;
    try {
      const data = JSON.parse(await file.text());
      if (!Array.isArray(data.files) || !data.files.every((f: any) => typeof f.path === "string" && typeof f.content === "string")) throw new Error("invalid");
      files = data.files; branch = typeof data.branch === "string" ? data.branch : "main";
      commits = Array.isArray(data.commits) ? data.commits : []; selectedPath = files[0]?.path ?? "";
      dirty = false; persist(); notice = "Репозиторий импортирован";
    } catch { notice = "Не удалось прочитать файл репозитория"; }
    input.value = "";
  }
</script>

<div class="repository-shell">
  <header class="repo-header">
    <div class="repo-title"><span class="repo-mark"><GitBranch size={17}/></span><div><strong>Репозиторий проекта</strong><small>Локальная история и файлы</small></div></div>
    <div class="repo-actions"><button class="repo-outline" onclick={exportRepository}><Download size={14}/> Экспорт</button><label class="repo-outline import-button"><Upload size={14}/> Импорт<input type="file" accept=".json,application/json" onchange={importRepository}/></label><button class="repo-commit" onclick={() => showCommit = true}><GitCommit size={14}/> Commit changes</button></div>
  </header>
  <div class="repo-toolbar"><span class="branch-select"><GitBranch size={14}/>{branch}</span><span class="repo-path">{files.length} файлов <i>·</i> {commits.length} коммитов</span><span class:dirty class="repo-state"><span></span>{dirty ? "Есть изменения" : "Чистое состояние"}</span></div>
  <div class="repo-layout">
    <aside class="repo-sidebar">
      <div class="repo-sidebar-tabs"><button class:repo-tab-active={activeTab === "files"} onclick={() => activeTab = "files"}>Файлы</button><button class:repo-tab-active={activeTab === "history"} onclick={() => activeTab = "history"}>История</button></div>
      {#if activeTab === "files"}
        <div class="repo-file-head"><span>ФАЙЛЫ ПРОЕКТА</span><button onclick={() => showNewFile = !showNewFile} title="Создать файл"><Plus size={15}/></button></div>
        {#if showNewFile}
          <form class="new-file-form" onsubmit={(e) => { e.preventDefault(); createFile(); }}><input bind:value={newFileName} placeholder="src/new-file.js"/><button type="submit"><Check size={13}/></button><button type="button" onclick={() => showNewFile = false}><X size={13}/></button></form>
        {/if}
        <div class="repo-file-list">
          {#each files as file}
            <div class:repo-file-active={selectedPath === file.path} class="repo-file-row"><button class="repo-file" onclick={() => selectedPath = file.path}><span class="file-glyph">{#if fileIcon(file.path) === "md"}<FileText size={14}/>{:else if fileIcon(file.path) === "json"}<FileJson size={14}/>{:else}<FileCode2 size={14}/>{/if}</span>{file.path}</button><button class="file-remove" onclick={() => removeFile(file.path)} title="Удалить файл"><Trash2 size={12}/></button></div>
          {/each}
        </div>
      {:else}
        <div class="commit-history">
          {#if commits.length === 0}<div class="history-empty"><GitCommit size={20}/><strong>Пока нет коммитов</strong><span>Сохрани первую версию проекта.</span></div>{:else}
            {#each commits as commit}<div class="history-item"><span class="history-node"></span><strong>{commit.message}</strong><code>{commit.id}</code><small>{commit.time} · {commit.files} файлов</small>{#if commit.snapshot?.length}<button class="restore-commit" onclick={() => restoreCommit(commit)}>Восстановить эту версию</button>{/if}</div>{/each}
          {/if}
        </div>
      {/if}
      <div class="repo-sidebar-foot"><span class="repo-online-dot"></span> Локальное хранилище</div>
    </aside>
    <section class="repo-editor">
      {#if currentFile}
        <div class="repo-editor-head"><div><span>{currentFile.path}</span><small>{currentFile.language.toUpperCase()}</small></div><button class="repo-outline" onclick={() => removeFile(currentFile.path)} disabled={files.length <= 1}><Trash2 size={13}/> Удалить</button></div>
        <div class="repo-code-area"><CodeEditor value={currentFile.content} language={currentFile.language} onChange={updateContent}/></div>
        <div class="repo-editor-foot"><span>{currentFile.content.split("\n").length} строк</span><span>UTF-8 <i>·</i> {currentFile.language}</span><span class:dirty>{dirty ? "Изменено" : "Сохранено локально"}</span></div>
      {/if}
    </section>
  </div>
  {#if showCommit}
    <div class="commit-overlay" role="presentation" onclick={(e) => { if (e.target === e.currentTarget) showCommit = false; }}>
      <form class="commit-dialog" onsubmit={(e) => { e.preventDefault(); commit(); }}>
        <div class="commit-dialog-head"><div><GitCommit size={17}/><strong>Зафиксировать изменения</strong></div><button type="button" onclick={() => showCommit = false}><X size={16}/></button></div>
        <p>Создай локальный снимок текущего состояния проекта.</p>
        <label>Сообщение коммита<input bind:value={commitMessage} placeholder="Например: Добавил экран авторизации" autofocus required maxlength="100"/></label>
        <div class="commit-dialog-actions"><button type="button" class="repo-outline" onclick={() => showCommit = false}>Отмена</button><button class="repo-commit" type="submit"><GitCommit size={14}/> Commit</button></div>
      </form>
    </div>
  {/if}
  {#if notice}<div class="repo-notice">{notice}<button onclick={() => notice = ""}><X size={13}/></button></div>{/if}
</div>

<style>
  .repository-shell{display:flex;flex-direction:column;min-height:560px;height:calc(100vh - 126px);max-height:900px;border:1px solid #e0e7df;border-radius:9px;background:#fff;overflow:hidden;box-shadow:0 5px 22px #243b2b08}
  .repo-header{height:58px;flex:0 0 58px;padding:0 16px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #e8ede7;gap:10px}
  .repo-title{display:flex;align-items:center;gap:10px}.repo-mark{width:30px;height:30px;display:grid;place-items:center;border-radius:7px;background:#edf4ee;color:#50745a}.repo-title>div{display:flex;flex-direction:column;gap:3px}.repo-title strong{font:700 11px 'Manrope';color:#354239}.repo-title small{font-size:9px;color:#929e94}
  .repo-actions{display:flex;align-items:center;gap:6px}.repo-outline,.repo-commit{height:31px;padding:0 10px;border:1px solid #e1e8e1;border-radius:5px;background:#fff;color:#66756a;display:inline-flex;align-items:center;justify-content:center;gap:6px;font:500 9px 'DM Sans';cursor:pointer;transition:background .18s,border-color .18s,transform .18s}.repo-outline:hover{background:#f4f7f3;border-color:#cbd9cc;transform:translateY(-1px)}.repo-commit{background:#315b42;color:white;border-color:#315b42}.repo-commit:hover{background:#3c6b4e;transform:translateY(-1px)}.repo-outline:disabled{opacity:.45;cursor:default;transform:none}.import-button input{display:none}
  .repo-toolbar{height:39px;flex:0 0 39px;padding:0 14px;display:flex;align-items:center;gap:13px;border-bottom:1px solid #edf0ec;background:#fbfcfb}.branch-select{display:flex;align-items:center;gap:6px;border:0;background:transparent;color:#56675a;font-size:9px;font-weight:600}.repo-path{color:#8c988f;font-size:9px}.repo-path i{font-style:normal;padding:0 4px;color:#c3cbc3}.repo-state{margin-left:auto;display:flex;align-items:center;gap:6px;color:#89958b;font-size:9px}.repo-state span{width:6px;height:6px;border-radius:50%;background:#83b18d}.repo-state.dirty span{background:#d7a65d}
  .repo-layout{display:grid;grid-template-columns:220px minmax(0,1fr);flex:1;min-height:0}.repo-sidebar{display:flex;flex-direction:column;min-width:0;border-right:1px solid #e8ede7;background:#fbfcfb}.repo-sidebar-tabs{display:flex;gap:15px;padding:12px 14px 0;border-bottom:1px solid #edf0ec}.repo-sidebar-tabs button{padding:0 0 10px;border:0;border-bottom:2px solid transparent;background:transparent;color:#8a968c;font-size:10px;cursor:pointer}.repo-sidebar-tabs button.repo-tab-active{border-color:#5e8968;color:#415d47;font-weight:600}.repo-file-head{height:39px;padding:0 12px 0 14px;display:flex;align-items:center;justify-content:space-between;color:#96a198;font-size:8px;font-weight:700;letter-spacing:.8px}.repo-file-head button{border:0;background:transparent;color:#738276;display:grid;place-items:center;cursor:pointer}.new-file-form{display:flex;gap:3px;padding:0 8px 8px}.new-file-form input{min-width:0;width:100%;height:27px;border:1px solid #dfe7df;border-radius:4px;padding:0 6px;font-size:9px}.new-file-form button{width:25px;border:0;border-radius:4px;background:#edf4ee;color:#52745a;display:grid;place-items:center}.repo-file-list{display:flex;flex-direction:column;gap:1px;padding:0 6px}.repo-file-row{display:flex;align-items:center;border-radius:4px}.repo-file-row.repo-file-active{background:#eaf2eb}.repo-file{height:31px;min-width:0;flex:1;display:flex;align-items:center;gap:8px;padding:0 8px;border:0;background:transparent;text-align:left;color:#59675d;font-size:9px;cursor:pointer;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}.file-glyph{display:grid;place-items:center;color:#8b9b8e}.file-remove{width:25px;height:25px;border:0;background:transparent;color:#a2aca3;display:none;place-items:center;cursor:pointer}.repo-file-row:hover .file-remove{display:grid}.repo-file-row:hover{background:#f0f4ef}.repo-sidebar-foot{margin-top:auto;padding:12px 13px;border-top:1px solid #edf0ec;color:#89968c;font-size:8px;display:flex;align-items:center;gap:7px}.repo-online-dot{width:6px;height:6px;border-radius:50%;background:#7cae87}
  .repo-editor{min-width:0;display:flex;flex-direction:column;background:#202a24}.repo-editor-head{height:39px;flex:0 0 39px;padding:0 12px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #334039;background:#27332c}.repo-editor-head>div{display:flex;align-items:center;gap:9px;color:#e0e8e1;font-size:10px}.repo-editor-head small{font-size:8px;color:#839287}.repo-editor-head .repo-outline{height:25px;background:#2e3b33;border-color:#3a493e;color:#aab8ad}.repo-code-area{flex:1;min-height:0}.repo-editor-foot{height:26px;flex:0 0 26px;display:flex;align-items:center;gap:15px;padding:0 11px;background:#29352e;border-top:1px solid #354239;color:#8e9c91;font-size:8px}.repo-editor-foot span:last-child{margin-left:auto}.repo-editor-foot .dirty{color:#d6b178}
  .restore-commit{align-self:flex-start;margin-top:3px;padding:5px 7px;border:1px solid #e1e9e1;border-radius:4px;background:#f4f8f4;color:#53745a;font-size:8px;cursor:pointer;transition:background .18s,border-color .18s}.restore-commit:hover{background:#e9f2ea;border-color:#cbdcca}\n  .history-empty{display:flex;flex-direction:column;align-items:center;text-align:center;gap:8px;padding:35px 13px;color:#9aa69c}.history-empty strong{font-size:10px;color:#637166}.history-empty span{font-size:8px;line-height:1.5}.commit-history{padding:12px 14px}.history-item{position:relative;padding:0 0 17px 13px;border-left:1px solid #dce5dc;display:flex;flex-direction:column;gap:5px}.history-node{position:absolute;width:7px;height:7px;border-radius:50%;background:#75a080;left:-4px;top:3px}.history-item strong{font-size:9px;color:#526156;line-height:1.4}.history-item code{font-size:8px;color:#6b896f}.history-item small{font-size:8px;color:#9aa49c}
  .commit-overlay{position:fixed;inset:0;z-index:20;background:#18231c66;display:grid;place-items:center;padding:18px;backdrop-filter:blur(4px);animation:repo-fade .15s ease}.commit-dialog{width:min(410px,100%);padding:19px;border:1px solid #e1e8e1;border-radius:10px;background:#fff;box-shadow:0 20px 65px #14201933;animation:repo-rise .2s cubic-bezier(.2,.75,.25,1)}.commit-dialog-head{display:flex;justify-content:space-between;align-items:center}.commit-dialog-head>div{display:flex;align-items:center;gap:8px;color:#4e7357}.commit-dialog-head strong{font:700 12px 'Manrope';color:#344138}.commit-dialog-head button{border:0;background:transparent;color:#89968c;cursor:pointer}.commit-dialog>p{margin:9px 0 17px;color:#89958b;font-size:10px}.commit-dialog label{display:flex;flex-direction:column;gap:7px;color:#56665a;font-size:10px}.commit-dialog label input{height:37px;padding:0 10px;border:1px solid #dfe7df;border-radius:5px;font-size:10px}.commit-dialog-actions{display:flex;justify-content:flex-end;gap:7px;margin-top:18px}.repo-notice{position:fixed;right:22px;bottom:22px;z-index:22;padding:11px 13px;border:1px solid #dfe8df;border-radius:7px;background:#fff;box-shadow:0 8px 25px #1f34251c;color:#526c57;font-size:10px;display:flex;gap:12px;align-items:center}.repo-notice button{border:0;background:transparent;color:#8b988d;display:grid;place-items:center}
  @keyframes repo-fade{from{opacity:0}to{opacity:1}}@keyframes repo-rise{from{opacity:0;transform:translateY(8px) scale(.99)}to{opacity:1;transform:none}}
  @media(max-width:680px){.repository-shell{height:calc(100vh - 110px);min-height:520px}.repo-header{padding:0 9px}.repo-actions{gap:4px}.repo-outline,.repo-commit{padding:0 7px;font-size:8px}.repo-layout{grid-template-columns:145px minmax(0,1fr)}.repo-file{font-size:8px}.repo-sidebar-tabs{gap:10px;padding-left:10px}.repo-toolbar{padding:0 8px;gap:7px}.repo-state{font-size:8px}.repo-editor-foot{gap:7px;font-size:7px}}
</style>