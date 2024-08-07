<script lang="ts">
  import { onMount } from 'svelte';
  import type { LlmProvider } from './types';
  import { LlmProviderList } from './types';
  import Icon from '@iconify/svelte';
  import { ListBox, ListBoxItem } from '@skeletonlabs/skeleton';
  import { fade } from 'svelte/transition';
  import { themeStore } from './store';
  import { LightSwitch } from '@skeletonlabs/skeleton';
  import { autoModeWatcher } from '@skeletonlabs/skeleton';
  import { setInitialClassState } from '@skeletonlabs/skeleton';
  import { SlideToggle } from '@skeletonlabs/skeleton';
  //import * as webllm from "@mlc-ai/web-llm";
  import ConfigModal from './ConfigModal.svelte';  // Import the modal component

    //import { Modal } from '@skeletonlabs/skeleton';

  //export let open = false;


  export let selectedItem: LlmProvider | null = null;

  const themes = [
    'skeleton',
    'wintry',
    'modern',
    'hamlindigo',
    'rocket',
    'sahara',
    'gold-nouveau',
    'vintage',
    'seafoam',
    'crimson'
  ];

  let isConfigModalOpen: boolean = false;  // State to control modal visibility


  let selectedTheme: string;

  themeStore.subscribe(value => {
    selectedTheme = value;
    if (typeof document !== 'undefined') {
      document.body.setAttribute('data-theme', value);
    }
  });

  function handleSelectItem(event: CustomEvent<LlmProvider> | { detail: LlmProvider }): void {
    selectedItem = event.detail;
    console.log("AppBarContent: selectedItem:", selectedItem);
    isListBoxVisible = false;
  }

  function handleSelectTheme(theme: string): void {
    themeStore.setTheme(theme);
    isThemeListBoxVisible = false;
  }

  $: if (!selectedItem) {
    const desiredSelector = 'gpt-4o-mini';
    const matchedItem = $LlmProviderList.find(item => item.model === desiredSelector);
    if (matchedItem) {
      selectedItem = matchedItem;
    } else {
      console.log(`No item with selector "${desiredSelector}" was found.`);
    }
  }

  let isListBoxVisible = false;
  let isThemeListBoxVisible = false;
  let listBoxContainer: HTMLElement;
  let themeListBoxContainer: HTMLElement;

  function handleClickOutside(event: MouseEvent) {
    if (listBoxContainer && !listBoxContainer.contains(event.target as Node)) {
      isListBoxVisible = false;
    }
    if (themeListBoxContainer && !themeListBoxContainer.contains(event.target as Node)) {
      isThemeListBoxVisible = false;
    }
  }

  onMount(() => {
    themeStore.init();
    document.addEventListener('click', handleClickOutside);

    // not a good plan
    // Load WebLLM models
    // const loadWebLLMModels = async () => {
    //   const availableModels = webllm.prebuiltAppConfig.model_list.map(
    //     (m) => m.model_id
    //   );
    //
    //   availableModels.forEach((model) => {
    //     LlmProviderList.update(list => [
    //       ...list,
    //       {
    //         provider: "webllm",
    //         model: model,
    //         title: `WebLLM - ${model}`,
    //         icon: "material-symbols:skull",
    //         subtitle: "Local WebLLM model",
    //         systemMessage: "You are a helpful AI assistant.",
    //         apiKeyName: "",
    //         local: true
    //       }
    //     ]);
    //   });
    // };
    //
    // loadWebLLMModels();

    return () => {
      document.removeEventListener('click', handleClickOutside);
    };
  });

  let localwebLlm: boolean = false;
</script>

<svelte:head>{@html '<script>(' + setInitialClassState.toString() + autoModeWatcher.toString() + ')();</script>'}</svelte:head>
<div class="relative" bind:this={listBoxContainer}>
  <button type="button" class="btn btn-sm variant-ghost-surface rounded-md"
          on:click|stopPropagation={() => isListBoxVisible = !isListBoxVisible}>
    <span>
      <Icon
        icon={selectedItem ? selectedItem.icon : "material-symbols:skull"}
        class="w-6 h-5"
      />
    </span>
    <span class="font-nunito text-sm bg-gradient-to-br from-pink-500 to-violet-500
    bg-clip-text text-transparent box-decoration-clone"
    >{selectedItem ? selectedItem.title : 'Select Model'}</span>
    <SlideToggle name="slide" bind:checked={localwebLlm}
                 active="" size="sm"
    >{localwebLlm ? 'local' : 'remote'}</SlideToggle>
  </button>
  {#if isListBoxVisible}
    <div transition:fade class="absolute top-full right-0 mt-2 z-50 min-w-[200px]
    w-max rounded-md p-3 bg-surface-500/80">
      <!-- config button -->
      <div class="flex justify-end">
        <button
          type="button"
          class="btn variant-filled"
          on:click={() => isConfigModalOpen = true}
        >
          <span>
            <Icon
              icon="majesticons:settings-cog-line"
              class="w-6 h-5"
            />
          </span>
          <span>Configure</span>
        </button>
      </div>

      <ListBox class="w-full">
        {#each $LlmProviderList as item}
          {#if item.local === localwebLlm}
            <ListBoxItem
              on:click={() => handleSelectItem({ detail: item })}
              active={selectedItem?.model === item.model}
              value={item.model}
              class="whitespace-nowrap"
              group="llmSelector"
              name="llmSelector"
            >
              <svelte:fragment slot="lead">
                <Icon icon="{item.icon}" class="text-white-800/90 w-6 h-6" />
              </svelte:fragment>
              {item.title}
            </ListBoxItem>
          {/if}
        {/each}
      </ListBox>
    </div>
  {/if}
</div>
<strong class="font-nunito text-xl bg-gradient-to-br from-pink-500 to-violet-500 bg-clip-text
text-transparent box-decoration-clone">MultiShot.AI</strong>
<div class="relative" bind:this={themeListBoxContainer}>
  <button type="button" class="btn btn-sm variant-ghost-surface rounded-md"
          on:click|stopPropagation={() => isThemeListBoxVisible = !isThemeListBoxVisible}>
    <span class="font-nunito">{selectedTheme}</span>
    <LightSwitch />
  </button>
  {#if isThemeListBoxVisible}
    <div
          transition:fade
          class="absolute top-full right-0 mt-2 z-50 min-w-[200px] w-max rounded-md p-3 bg-surface-500/80 max-h-[80vh] overflow-y-auto"
        >
        <ListBox class="w-full">
          {#each themes as theme}
            <ListBoxItem
              on:click={() => handleSelectTheme(theme)}
              active={selectedTheme === theme}
              value={theme}
              class="whitespace-nowrap"
              group="themeSelector"
              name="themeSelector"
            >
              {theme}
            </ListBoxItem>
          {/each}
        </ListBox>
    </div>
    {/if}
</div>
  {#if isConfigModalOpen}
    <ConfigModal bind:open={isConfigModalOpen} />
  {/if}

<a class="font-nunito btn btn-sm variant-ghost-surface rounded-md"
   href="https://twitter.com/cronuser" target="_blank" rel="noreferrer">
  <Icon icon="simple-icons:x" class="w-6 h-5" /></a>
<a class="font-nunito btn btn-sm variant-ghost-surface rounded-md"
   href="https://github.com/randomtask2000/MultiShot.AI" target="_blank" rel="noreferrer">
  <Icon icon="simple-icons:github" class="w-6 h-5 pr-2" /> GitHub </a>
