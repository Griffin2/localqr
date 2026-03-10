<script>
  import TrustBar from "$lib/TrustBar.svelte";
  import ToolHeader from "$lib/ToolHeader.svelte";
  import ToolFooter from "$lib/ToolFooter.svelte";
  import QRCode from "qrcode";

  let input = $state("");
  let qrDataUrl = $state("");
  let copied = $state(false);
  let errorLevel = $state("M");
  let size = $state(256);
  let hasError = $state(false);

  const sizeOptions = [
    { label: "Small", value: 128 },
    { label: "Medium", value: 256 },
    { label: "Large", value: 512 },
    { label: "XL", value: 1024 },
  ];

  async function generate() {
    if (input.trim() === "") {
      qrDataUrl = "";
      hasError = false;
      return;
    }
    try {
      qrDataUrl = await QRCode.toDataURL(input, {
        width: size,
        margin: 2,
        errorCorrectionLevel: errorLevel,
        color: { dark: "#1a1a1a", light: "#ffffff" },
      });
      hasError = false;
    } catch (e) {
      qrDataUrl = "";
      hasError = true;
    }
  }

  function handleInput() {
    generate();
  }

  function clear() {
    input = "";
    qrDataUrl = "";
    hasError = false;
    copied = false;
  }

  function download() {
    if (!qrDataUrl) return;
    const a = document.createElement("a");
    a.href = qrDataUrl;
    a.download = "qrcode.png";
    a.click();
  }

  async function copyImage() {
    if (!qrDataUrl) return;
    try {
      const res = await fetch(qrDataUrl);
      const blob = await res.blob();
      await navigator.clipboard.write([
        new ClipboardItem({ "image/png": blob }),
      ]);
      copied = true;
      setTimeout(() => (copied = false), 1500);
    } catch {
      /* fallback: copy the text input instead */
      await navigator.clipboard.writeText(input);
      copied = true;
      setTimeout(() => (copied = false), 1500);
    }
  }

  function loadSample() {
    input = "https://example.com";
    generate();
  }

  function loadWifi() {
    input = "WIFI:T:WPA;S:MyNetwork;P:MyPassword;;";
    generate();
  }

  let charCount = $derived(new Blob([input]).size);
</script>

<TrustBar sourceUrl="https://github.com/yourusername/qr-generator" />

<div class="page-shell">
  <ToolHeader
    title="QR Code Generator"
    description="Generate QR codes instantly — nothing leaves your browser."
  />

  <div class="tool-area qr-layout">
    <div class="panel">
      <label class="tool-label" for="input">content</label>
      <textarea
        id="input"
        class="tool-textarea"
        bind:value={input}
        oninput={handleInput}
        placeholder="Paste a URL, text, WiFi string, or anything..."
        spellcheck="false"
      ></textarea>
    </div>

    <div class="panel qr-preview-panel">
      <label class="tool-label">preview</label>
      <div class="qr-preview">
        {#if qrDataUrl}
          <img src={qrDataUrl} alt="Generated QR code" class="qr-image" />
        {:else if hasError}
          <div class="qr-placeholder qr-error">
            <span>Too much data for QR code</span>
          </div>
        {:else}
          <div class="qr-placeholder">
            <span>Your QR code appears here</span>
          </div>
        {/if}
      </div>
    </div>
  </div>

  <div class="controls">
    <button class="btn btn-primary" onclick={generate}>Generate</button>
    <button class="btn" onclick={download} disabled={!qrDataUrl}
      >Download PNG</button
    >
    <button class="btn" onclick={copyImage} disabled={!qrDataUrl}>
      {copied ? "✓ Copied" : "Copy"}
    </button>
    <button class="btn" onclick={clear}>Clear</button>
    <button class="btn btn-sample" onclick={loadSample}>URL sample</button>
    <button class="btn btn-sample" onclick={loadWifi}>WiFi sample</button>

    <div class="options">
      <label class="option-label">
        <span class="option-text">Size</span>
        <select bind:value={size} onchange={generate} class="option-select">
          {#each sizeOptions as opt}
            <option value={opt.value}>{opt.label}</option>
          {/each}
        </select>
      </label>
      <label class="option-label">
        <span class="option-text">EC</span>
        <select
          bind:value={errorLevel}
          onchange={generate}
          class="option-select"
        >
          <option value="L">Low</option>
          <option value="M">Medium</option>
          <option value="Q">Quartile</option>
          <option value="H">High</option>
        </select>
      </label>
    </div>

    <div class="spacer"></div>

    {#if qrDataUrl}
      <span class="pill pill-ok">
        <span class="pill-dot"></span>
        {size}×{size}px
      </span>
    {:else if hasError}
      <span class="pill pill-err">
        <span class="pill-dot"></span>
        overflow
      </span>
    {/if}

    {#if charCount > 0}
      <span class="byte-tag">{charCount.toLocaleString()} B</span>
    {/if}
  </div>

  <ToolFooter />
</div>

<style>
  .qr-layout {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
  }

  .panel {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  .qr-preview-panel {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  .qr-preview {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid var(--border-light);
    border-radius: var(--radius-md);
    background: var(--bg-surface);
    min-height: 200px;
  }

  .qr-image {
    image-rendering: pixelated;
    max-width: 100%;
    max-height: 100%;
    border-radius: var(--radius-sm);
  }

  .qr-placeholder {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    width: 100%;
    font-family: var(--font-mono);
    font-size: 12px;
    color: var(--text-muted);
    padding: 24px;
    text-align: center;
  }

  .qr-error {
    color: #dc2626;
  }

  .btn-sample {
    font-size: 12px;
    padding: 8px 12px;
    color: var(--text-muted);
  }

  .options {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-left: 8px;
  }

  .option-label {
    display: flex;
    align-items: center;
    gap: 5px;
    font-family: var(--font-mono);
    font-size: 12px;
    color: var(--text-muted);
    cursor: pointer;
  }

  .option-text {
    color: var(--text-muted);
  }

  .option-select {
    font-family: var(--font-mono);
    font-size: 12px;
    padding: 3px 6px;
    border-radius: var(--radius-sm);
    border: 1px solid var(--border-light);
    background: var(--bg-input);
    color: var(--text-secondary);
    cursor: pointer;
    outline: none;
  }

  .option-select:focus {
    border-color: var(--accent-border);
  }

  .spacer {
    margin-left: auto;
  }

  .byte-tag {
    font-family: var(--font-mono);
    font-size: 11px;
    color: var(--text-muted);
    background: var(--bg-input);
    padding: 3px 8px;
    border-radius: var(--radius-sm);
    border: 1px solid var(--border-light);
    margin-left: 6px;
  }

  button:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  @media (max-width: 640px) {
    .qr-layout {
      grid-template-columns: 1fr;
    }

    .options {
      display: none;
    }

    .controls {
      flex-wrap: wrap;
    }
  }
</style>
