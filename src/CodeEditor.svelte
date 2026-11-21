<script lang="ts">
  import { onMount } from "svelte";
  import { basicSetup } from "codemirror";
  import { EditorView, keymap } from "@codemirror/view";
  import { defaultKeymap, indentWithTab } from "@codemirror/commands";
  import { javascript } from "@codemirror/lang-javascript";
  import { html } from "@codemirror/lang-html";
  import { css } from "@codemirror/lang-css";
  import { json } from "@codemirror/lang-json";
  import { oneDark } from "@codemirror/theme-one-dark";

  type Props = { value?: string; language?: string; onChange?: (value: string) => void; readOnly?: boolean };
  let { value = "", language = "javascript", onChange = (_value: string) => {}, readOnly = false }: Props = $props();
  let host: HTMLDivElement;
  let view: EditorView | undefined;
  const languages: Record<string, () => any> = {
    javascript, typescript: () => javascript({ typescript: true }), html, css, json
  };

  onMount(() => {
    view = new EditorView({
      doc: value,
      parent: host,
      extensions: [
        basicSetup,
        keymap.of([...defaultKeymap, indentWithTab]),
        (languages[language] ?? javascript)(),
        oneDark,
        EditorView.editable.of(!readOnly),
        EditorView.updateListener.of((update) => {
          if (update.docChanged) onChange(update.state.doc.toString());
        }),
        EditorView.theme({
          "&": { height: "100%", fontSize: "12px", backgroundColor: "#202a24" },
          ".cm-scroller": { overflow: "auto", fontFamily: "ui-monospace, SFMono-Regular, Consolas, monospace", lineHeight: "1.75" },
          ".cm-gutters": { backgroundColor: "#202a24", border: "none", color: "#68776d" },
          ".cm-activeLine": { backgroundColor: "#ffffff08" },
          ".cm-activeLineGutter": { backgroundColor: "#ffffff08" },
          "&.cm-focused": { outline: "none" }
        })
      ]
    });
    return () => view?.destroy();
  });

  $effect(() => {
    if (!view) return;
    const current = view.state.doc.toString();
    if (value !== current) view.dispatch({ changes: { from: 0, to: current.length, insert: value } });
  });
</script>

<div class="code-editor-host" bind:this={host}></div>
<style>
  .code-editor-host{height:100%;min-height:260px;overflow:hidden;background:#202a24}
  .code-editor-host :global(.cm-editor){height:100%}
</style>