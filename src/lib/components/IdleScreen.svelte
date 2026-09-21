<script lang="ts">
  import { open } from "@tauri-apps/plugin-dialog";
  import { invoke } from "@tauri-apps/api/core";

  export let binaryPath = "";
  export let metadataPath = "";
  export let outputDir = "";
  export let onConfigClick: () => void = () => {};
  export let onDump: () => void = () => {};
  export let onBinaryPicked: (path: string, format: string, unity_version: string) => void = () => {};

  // Converts Android content:// URIs to real filesystem paths
  function androidUriToPath(uri: string): string {
    if (!uri.startsWith("content://")) return uri;
    const decoded = decodeURIComponent(uri);
    let match = decoded.match(/com\.android\.externalstorage\.documents\/document\/primary(?:%3A|:|\/)(.*)/);
    if (match && match[1]) return "/storage/emulated/0/" + match[1].replace(/%2F/g, '/');
    match = decoded.match(/(\/storage\/emulated\/0\/.*)/);
    if (match && match[1]) return match[1];
    return uri;
  }

  async function pickFile(type: "binary" | "metadata") {
    const raw_file = await open({
      multiple: false,
    });

    if (raw_file) {
      let path = androidUriToPath(raw_file as string);

      if (type === "binary") {
        binaryPath = path;
        try {
          const info = await invoke<{ format: string; unity_version: string }>("detect_binary", { path });
          onBinaryPicked(path, info.format, info.unity_version);
        } catch (e) {
          alert(`Failed to detect binary: ${e}`);
        }
      } else {
        metadataPath = path;
      }
    }
  }
</script>

<div class="p-5 space-y-4">
  <div class="space-y-2">
    <p class="text-sm font-medium m3-secondary">IL2CPP Binary</p>
    <div class="flex gap-2">
      <input 
        class="m3-text-field flex-1" 
        type="text" 
        bind:value={binaryPath} 
        placeholder="libil2cpp.so" 
      />
      <button class="m3-button m3-button-tonal" on:click={() => pickFile("binary")}>
        Pick
      </button>
    </div>
  </div>

  <div class="space-y-2">
    <p class="text-sm font-medium m3-secondary">Metadata</p>
    <div class="flex gap-2">
      <input 
        class="m3-text-field flex-1" 
        type="text" 
        bind:value={metadataPath} 
        placeholder="global-metadata.dat" 
      />
      <button class="m3-button m3-button-tonal" on:click={() => pickFile("metadata")}>
        Pick
      </button>
    </div>
  </div>

  <div class="space-y-2">
    <p class="text-sm font-medium m3-secondary">Output Directory</p>
    <div class="flex gap-2">
      <input 
        class="m3-text-field flex-1" 
        type="text" 
        bind:value={outputDir} 
        placeholder="IL2CppDumper" 
      />
    </div>
  </div>

  <button class="m3-button m3-button-filled w-full" on:click={onDump}>
    Start Dump
  </button>
  <button class="m3-button m3-button-text w-full" on:click={onConfigClick}>
    Dump Options
  </button>
</div>
