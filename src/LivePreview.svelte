<script lang="ts">
  type Platform = "desktop" | "phone" | "tablet" | "web";
  let { code = "", projectName = "Моё приложение", platform = "desktop" }: { code?: string; projectName?: string; platform?: Platform } = $props();
  let refreshTick = $state(0);
  const encodedCode = $derived(btoa(unescape(encodeURIComponent(code))));
  function escapeHtml(value: string) { return value.replace(/[&<>"]/g, (char) => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;" }[char] ?? char)); }
  const srcdoc = $derived(
    '<!doctype html><html lang="ru"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">' +
    '<meta http-equiv="Content-Security-Policy" content="default-src \'none\'; img-src data: blob:; style-src \'unsafe-inline\'; script-src \'unsafe-inline\' \'unsafe-eval\'; font-src data:">' +
    '<style>*{box-sizing:border-box}body{margin:0;padding:26px;font:14px -apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;color:#243128;background:#fff;line-height:1.5}#app{max-width:560px;margin:0 auto}h1{font-size:25px;letter-spacing:-.7px;margin:0 0 12px}p{color:#657269}button{border:0;border-radius:8px;background:#416b50;color:white;padding:10px 15px;font-weight:600;cursor:pointer}input{display:block;width:100%;max-width:320px;margin:10px 0;padding:10px 12px;border:1px solid #d6ded7;border-radius:7px;font:inherit}#error{display:none;margin:0 0 14px;padding:9px 11px;background:#fff1ef;color:#a43c31;border-radius:6px;font-size:12px}</style></head><body><div id="error"></div><main id="app"><h1>' +
    escapeHtml(projectName) +
    '</h1></main><script>window.onerror=function(message){var e=document.getElementById("error");e.style.display="block";e.textContent="Ошибка: "+String(message)};try{(0,eval)(decodeURIComponent(escape(atob("' +
    encodedCode +
    '"))))}catch(error){window.onerror(error.message)}<\/script></body></html>'
  );
</script>

<section class="live-preview">
  <header class="live-preview-head">
    <div class="live-preview-label"><span class="preview-live-dot"></span><div><strong>Живой предпросмотр</strong><small>Изолированный запуск · обновляется при изменениях</small></div></div>
    <div class="preview-tools">
      <span class="preview-device">{#if platform === "phone"}Телефон{:else if platform === "tablet"}Планшет{:else if platform === "web"}Браузер{:else}Компьютер{/if}</span>
      <button type="button" class="preview-refresh" onclick={() => refreshTick++} title="Перезагрузить">↻</button>
    </div>
  </header>
  <div class="preview-canvas">
    <div class:device-phone={platform === "phone"} class:device-tablet={platform === "tablet"} class:device-desktop={platform === "desktop" || platform === "web"} class="preview-device-shell">
      {#key refreshTick}<iframe title="Предпросмотр приложения" sandbox="allow-scripts" srcdoc={srcdoc}></iframe>{/key}
    </div>
  </div>
  <footer class="preview-foot"><span>Sandbox iframe · без доступа к файлам приложения и сети</span><span>LIVE</span></footer>
</section>

<style>
.live-preview{border:1px solid #dfe7df;border-radius:9px;overflow:hidden;background:#fff;box-shadow:0 5px 22px #263b2b08}
.live-preview-head{height:55px;padding:0 15px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #e8ede8}
.live-preview-label,.preview-tools{display:flex;align-items:center;gap:10px}.live-preview-label strong{display:block;font-size:11px;color:#2b392f}.live-preview-label small{display:block;margin-top:3px;color:#8a968c;font-size:9px}.preview-live-dot{width:7px;height:7px;border-radius:50%;background:#5ca575;box-shadow:0 0 0 3px #5ca57520}
.preview-tools{gap:7px}.preview-device{color:#718075;font-size:9px;padding:6px 8px;background:#f3f6f2;border-radius:5px}
.preview-refresh{width:28px;height:28px;display:grid;place-items:center;border:1px solid #e4eae4;border-radius:5px;background:white;color:#647469;cursor:pointer;font-size:19px;transition:background .18s}.preview-refresh:hover{background:#f0f5f0}
.preview-canvas{min-height:320px;padding:22px;display:grid;place-items:center;background-color:#f3f5f2;background-image:radial-gradient(#d9e0d9 .8px,transparent .8px);background-size:16px 16px}
.preview-device-shell{width:100%;height:330px;background:#fff;overflow:hidden;box-shadow:0 8px 28px #1d30221a;border:1px solid #dce3dc;transition:width .25s,height .25s,border-radius .25s}
.preview-device-shell iframe{width:100%;height:100%;border:0;background:#fff}
.preview-phone{width:210px;height:370px;border:6px solid #26332b;border-radius:27px}
.preview-tablet{width:min(520px,100%);height:350px;border:7px solid #26332b;border-radius:17px}
.preview-desktop{width:100%;max-width:700px;border-radius:7px}
.preview-foot{height:32px;display:flex;align-items:center;justify-content:space-between;padding:0 13px;border-top:1px solid #e8ede8;color:#8a968c;font-size:9px}.preview-foot span:last-child{font-weight:700;letter-spacing:1px;color:#5c9870}
@media(max-width:650px){.preview-canvas{padding:14px;min-height:300px}.preview-device-shell{height:300px}.preview-phone{width:178px;height:320px}.preview-foot{font-size:8px}}
</style>