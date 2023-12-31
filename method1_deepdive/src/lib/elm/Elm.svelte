<script context="module" lang="ts">
  declare let Elm: ElmInstance;

  type Callback = () => void;

  const scriptsLoaded = new Set<string>();
  const loadingPromises: Record<string, Promise<void>> = {};

  const loadScript = (src: string, callback: Callback): void => {
    if (scriptsLoaded.has(src)) {
      callback();
      return;
    }

    if (!loadingPromises[src]) {
      loadingPromises[src] = new Promise((resolve, reject) => {
        const script = document.createElement("script");
        script.src = src;
        script.async = true;

        script.onload = () => {
          scriptsLoaded.add(src);
          resolve();
        };

        script.onerror = (event) => {
          console.error(`Error loading script ${src}:`, event);
          reject(new Error(`Script load error: ${src}`));
        };

        document.head.appendChild(script);
      });
    }

    loadingPromises[src]?.then(callback).catch(() => {
      console.error(`Failed to load script: ${src}`);
    });
  };
</script>

<script lang="ts">
  import { onMount } from "svelte";
  import { assets } from "$app/paths";

  export let elmJsFilename: `${string}.js`;
  export let moduleName: string;

  const elmAssetsDirectory: string = `${assets}/elm`;
  const elmJsFilePath: string = `${elmAssetsDirectory}/${elmJsFilename}`;

  let elmRoot: Node;
  const handleLoad: Callback = () => {
    if (Elm && Elm[moduleName]) {
      Elm[moduleName].init({ node: elmRoot });
    } else {
      console.error("Elm module not found or not loaded: ", moduleName);
    }
  };

  onMount(() => {
    loadScript(elmJsFilePath, handleLoad);
  });
</script>

<div bind:this={elmRoot} />
