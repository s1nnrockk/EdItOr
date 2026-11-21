<script lang="ts">
  import { onMount } from "svelte";
  import * as Blockly from "blockly";
  import "blockly/blocks";
  import { javascriptGenerator } from "blockly/javascript";
  import { Copy, Check, RotateCcw, Code2, Blocks, ChevronDown } from "lucide-svelte";

  let host: HTMLDivElement;
  let workspace: Blockly.WorkspaceSvg | null = null;
  let generated = $state("");
  let copied = $state(false);
  let showCode = $state(true);
  let blockCount = $state(0);

  const toolbox = {
    kind: "categoryToolbox",
    contents: [
      { kind: "category", name: "Логика", categorystyle: "logic_category", contents: [
        { kind: "block", type: "controls_if" },
        { kind: "block", type: "logic_compare" },
        { kind: "block", type: "logic_operation" },
        { kind: "block", type: "logic_boolean" },
        { kind: "block", type: "logic_negate" }
      ]},
      { kind: "category", name: "Циклы", categorystyle: "loop_category", contents: [
        { kind: "block", type: "controls_repeat_ext" },
        { kind: "block", type: "controls_whileUntil" },
        { kind: "block", type: "controls_for" }
      ]},
      { kind: "category", name: "Математика", categorystyle: "math_category", contents: [
        { kind: "block", type: "math_number" },
        { kind: "block", type: "math_arithmetic" },
        { kind: "block", type: "math_round" },
        { kind: "block", type: "math_random_int" }
      ]},
      { kind: "category", name: "Текст", categorystyle: "text_category", contents: [
        { kind: "block", type: "text" },
        { kind: "block", type: "text_print" },
        { kind: "block", type: "text_join" },
        { kind: "block", type: "text_length" }
      ]},
      { kind: "category", name: "Переменные", custom: "VARIABLE", categorystyle: "variable_category" },
      { kind: "category", name: "Функции", custom: "PROCEDURE", categorystyle: "procedure_category" },
      { kind: "sep" },
      { kind: "category", name: "Интерфейс", categorystyle: "ui_category", contents: [
        { kind: "block", type: "editor_ui_text" },
        { kind: "block", type: "editor_ui_button" },
        { kind: "block", type: "editor_ui_input" }
      ]}
    ]
  };

  function registerAppBlocks() {
    Blockly.Blocks["editor_ui_text"] = {
      init() {
        this.appendDummyInput().appendField("Показать текст").appendField(new Blockly.FieldTextInput("Привет, мир!"), "VALUE");
        this.setPreviousStatement(true);
        this.setNextStatement(true);
        this.setStyle("text_blocks");
        this.setTooltip("Добавляет текстовый элемент в интерфейс.");
      }
    };
    Blockly.Blocks["editor_ui_button"] = {
      init() {
        this.appendDummyInput().appendField("Создать кнопку").appendField(new Blockly.FieldTextInput("Нажми меня"), "LABEL");
        this.setPreviousStatement(true);
        this.setNextStatement(true);
        this.setStyle("ui_blocks");
        this.setTooltip("Создаёт кнопку с указанной надписью.");
      }
    };
    Blockly.Blocks["editor_ui_input"] = {
      init() {
        this.appendDummyInput().appendField("Поле ввода").appendField(new Blockly.FieldTextInput("Введите текст"), "PLACEHOLDER");
        this.setPreviousStatement(true);
        this.setNextStatement(true);
        this.setStyle("ui_blocks");
        this.setTooltip("Добавляет поле для ввода текста.");
      }
    };
    javascriptGenerator.forBlock["editor_ui_text"] = (block) => {
      const value = block.getFieldValue("VALUE");
      return `editor.addText(${JSON.stringify(value)});\n`;
    };
    javascriptGenerator.forBlock["editor_ui_button"] = (block) => {
      const label = block.getFieldValue("LABEL");
      return `editor.addButton(${JSON.stringify(label)});\n`;
    };
    javascriptGenerator.forBlock["editor_ui_input"] = (block) => {
      const placeholder = block.getFieldValue("PLACEHOLDER");
      return `editor.addInput(${JSON.stringify(placeholder)});\n`;
    };
  }

  function updateCode() {
    if (!workspace) return;
    generated = javascriptGenerator.workspaceToCode(workspace) || "// Добавь блоки, чтобы увидеть код здесь.";
    blockCount = workspace.getAllBlocks(false).length;
    try {
      localStorage.setItem("editor-block-workspace", JSON.stringify(Blockly.serialization.workspaces.save(workspace)));
    } catch { /* Local storage may be unavailable in private mode. */ }
  }

  async function copyCode() {
    try {
      await navigator.clipboard.writeText(generated);
      copied = true;
      window.setTimeout(() => copied = false, 1500);
    } catch {
      copied = false;
    }
  }

  function clearWorkspace() {
    workspace?.clear();
    updateCode();
  }

  onMount(() => {
    registerAppBlocks();
    const theme = Blockly.Theme.defineTheme("editorTheme", {
      name: "editorTheme",
      base: Blockly.Themes.Classic,
      componentStyles: {
        workspaceBackgroundColour: "#f8faf8",
        toolboxBackgroundColour: "#f1f4f0",
        toolboxForegroundColour: "#45564a",
        flyoutBackgroundColour: "#ffffff",
        flyoutForegroundColour: "#344238",
        flyoutOpacity: 1,
        scrollbarColour: "#cbd5cc",
        insertionMarkerColour: "#5d9870",
        insertionMarkerOpacity: 0.35,
        cursorColour: "#426c56"
      },
      fontStyle: { family: "DM Sans, sans-serif", weight: "500", size: 12 },
      startHats: false,
      blockStyles: {
        logic_blocks: { colourPrimary: "#789b83", colourSecondary: "#6b8c76", colourTertiary: "#55745f" },
        loop_blocks: { colourPrimary: "#c39a68", colourSecondary: "#b18a5b", colourTertiary: "#947149" },
        math_blocks: { colourPrimary: "#6f91b1", colourSecondary: "#5f80a0", colourTertiary: "#4b6c8b" },
        text_blocks: { colourPrimary: "#a28ab7", colourSecondary: "#9076a6", colourTertiary: "#765c8c" },
        variable_blocks: { colourPrimary: "#c18c75", colourSecondary: "#ae7962", colourTertiary: "#91634f" },
        procedure_blocks: { colourPrimary: "#c18c75", colourSecondary: "#ae7962", colourTertiary: "#91634f" },
        ui_blocks: { colourPrimary: "#5e9472", colourSecondary: "#4c805f", colourTertiary: "#3d6a4e" }
      },
      categoryStyles: {
        logic_category: { colour: "#789b83" },
        loop_category: { colour: "#c39a68" },
        math_category: { colour: "#6f91b1" },
        text_category: { colour: "#a28ab7" },
        variable_category: { colour: "#c18c75" },
        procedure_category: { colour: "#c18c75" },
        ui_category: { colour: "#5e9472" }
      }
    });
    workspace = Blockly.inject(host, {
      toolbox,
      theme,
      renderer: "zelos",
      grid: { spacing: 24, length: 3, colour: "#e2e9e2", snap: true },
      zoom: { controls: true, wheel: true, startScale: 0.92, maxScale: 1.5, minScale: 0.55, scaleSpeed: 1.08 },
      trashcan: true,
      sounds: false,
      move: { scrollbars: true, drag: true, wheel: true }
    });
    workspace.addChangeListener((event) => {
      if (event.isUiEvent) return;
      updateCode();
    });
    try {
      const saved = localStorage.getItem("editor-block-workspace");
      if (saved) Blockly.serialization.workspaces.load(JSON.parse(saved), workspace);
    } catch { /* Start with an empty workspace if saved data is invalid. */ }
    updateCode();
    const resize = () => workspace?.resize();
    window.addEventListener("resize", resize);
    return () => {
      window.removeEventListener("resize", resize);
      workspace?.dispose();
      workspace = null;
    };
  });
</script>

<div class="block-editor">
  <div class="workspace-toolbar">
    <div class="workspace-heading">
      <span class="workspace-mark"><Blocks size={16}/></span>
      <div><strong>Рабочая область</strong><small>{blockCount} {blockCount === 1 ? "блок" : blockCount > 1 && blockCount < 5 ? "блока" : "блоков"} на сцене</small></div>
    </div>
    <div class="workspace-actions">
      <button class="tool-button" onclick={clearWorkspace} title="Очистить рабочую область"><RotateCcw size={15}/><span>Сбросить</span></button>
      <button class:code-active={showCode} class="tool-button code-toggle" onclick={() => showCode = !showCode}><Code2 size={15}/><span>Код</span><ChevronDown size={13}/></button>
    </div>
  </div>
  <div class:code-hidden={!showCode} class="workspace-content">
    <div class="blockly-host" bind:this={host}></div>
    {#if showCode}
      <aside class="code-panel">
        <div class="code-panel-head"><div><span class="code-dot"></span><strong>JavaScript</strong></div><button class="copy-button" onclick={copyCode} title="Скопировать код">{#if copied}<Check size={14}/>{:else}<Copy size={14}/>{/if}<span>{copied ? "Скопировано" : "Копировать"}</span></button></div>
        <pre><code>{generated}</code></pre>
        <div class="code-panel-foot">Код генерируется автоматически из блоков</div>
      </aside>
    {/if}
  </div>
</div>