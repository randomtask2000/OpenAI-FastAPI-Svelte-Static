<script lang="ts">
  import { onMount, createEventDispatcher } from 'svelte';
  import Icon from '@iconify/svelte';
  import { ProgressRadial } from '@skeletonlabs/skeleton';
  import * as webllm from "@mlc-ai/web-llm";
  import { LlmProviderList } from './types';
  import { get } from 'svelte/store';
  import { listStore } from './store';

  export let open = false;

  const dispatch = createEventDispatcher();
  let dialog: HTMLDialogElement;
  let availableModels: string[] = [];
  let selectedModel = "TinyLlama-1.1B-Chat-v0.4-q4f32_1-MLC-1k";
  let isDownloading = false;
  let downloadStatus = '';

  const engine = new webllm.MLCEngine();

  onMount(() => {
    availableModels = webllm.prebuiltAppConfig.model_list.map(m => m.model_id);
  });

  $: if (open) {
    if (dialog && !dialog.open) dialog.showModal();
  } else {
    if (dialog && dialog.open) dialog.close();
  }

  function closeModal() {
    open = false;
    dispatch('close');
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === 'Escape') {
      closeModal();
    }
  }

  async function initializeWebLLMEngine() {
    isDownloading = true;
    const config = {
      temperature: 1.0,
      top_p: 1
    };

    try {
      await engine.reload(selectedModel, config);
      isDownloading = false;
      downloadStatus = "Model initialized successfully.";

      // Add the downloaded model to the LlmProviderList
      LlmProviderList.update(providers => [
        ...providers,
        {
          provider: "webllm",
          model: selectedModel,
          title: selectedModel,
          icon: "mdi:brain",
          subtitle: "Local WebLLM model",
          systemMessage: "You are a helpful AI assistant.",
          apiKeyName: "",
          local: true
        }
      ]);

      // Update the store with the new model
      listStore.addModel(selectedModel);
    } catch (error) {
      console.error("Error initializing WebLLM engine:", error);
      downloadStatus = "Error: Failed to initialize the model. Please try again.";
    } finally {
      isDownloading = false;
    }
  }

  function updateEngineInitProgressCallback(report: { progress: number; text: string }) {
    console.log("initialize", report.progress);
    downloadStatus = report.text;
  }

  engine.setInitProgressCallback(updateEngineInitProgressCallback);
</script>

<dialog bind:this={dialog} on:close={closeModal} class="w-full max-w-2xl p-4 rounded-lg shadow-xl bg-surface-100/90 text-left fixed top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">

  <div class="flex flex-col h-full">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-xl font-bold">WebLLM Configuration</h2>
      <button
        on:click={closeModal}
        class="text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200"
        aria-label="Close modal"
      >
        <Icon icon="mdi:close" />
      </button>
    </div>

    <div class="space-y-4 flex-grow overflow-y-auto">
      <div>
        <label for="model-selection" class="block mb-2">Select Model</label>
        <select
          id="model-selection"
          bind:value={selectedModel}
          class="w-full p-2 border rounded bg-surface-300"
        >
          {#each availableModels as model}
            <option value={model}>{model}</option>
          {/each}
        </select>
      </div>

      <div>
        <button
          on:click={initializeWebLLMEngine}
          disabled={isDownloading}
          class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 disabled:bg-blue-300"
        >
          {isDownloading ? 'Downloading...' : 'Download'}
        </button>
        {#if isDownloading}
          <div class="mt-2 flex items-center">
            <ProgressRadial stroke={100} meter="stroke-primary-500" track="stroke-primary-500/30" />
            <p class="ml-2">{downloadStatus}</p>
          </div>
        {/if}
      </div>
    </div>

    <div class="flex justify-end mt-4">
      <button
        on:click={closeModal}
        class="px-4 py-2 bg-gray-300 text-gray-800 rounded hover:bg-gray-400"
      >
        Close
      </button>
    </div>
  </div>
</dialog>

<style>
  dialog::backdrop {
    background-color: rgba(0, 0, 0, 0.5);
  }
</style>
