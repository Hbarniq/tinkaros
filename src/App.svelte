<script async script lang="ts">
  import { invoke } from "@tauri-apps/api/tauri";
  import { onMount } from "svelte";
  import { config } from "./stores/config";
  import { BootstrapToast, ToastContainer } from "svelte-toasts"
  import { open } from '@tauri-apps/api/shell';
  import Main from "./lib/Main.svelte";
  import Welcome from "./lib/Welcome.svelte";
  import newToast from "./scripts/toasts";
  import Loading from "./components/Loading.svelte";
  import { state } from "./stores/state";
  import { fade } from "svelte/transition";

  onMount(async () => {
    try { await invoke("check_online"); } catch (err) {
      return newToast("error", "internet not connected", err);
    }

    await invoke("get_config").then(async (c) => {
      config.set(c);
    }).catch(err => { newToast("error", "unable to load configs", err) });
    if ($config.check_tinkaros_update) {
      invoke("check_tinkaros_update").then(async (r: boolean) => {
        if (r) {
          newToast("info", "update available!", "Tinkaros has a new update available it is recommended you update!", 15000, () => { open("https://github.com/Hbarniq/tinkaros/releases/latest") })
        }
      }).catch(err => { newToast("error", "unable to find tinkaros updates", err) })
    }
    $state.loading = false
  });
</script>

<main>
  <div id="background" class="full"></div>
  <ToastContainer let:data={data}>
    <BootstrapToast theme="dark" {data} />
  </ToastContainer>

  {#if $state.loading}
    <div class="blur-background" transition:fade="{{duration: 200}}">
      <Loading />
    </div>
  {/if}

  {#if $config && $config.init}
    <Main />
  {:else}
    <Welcome />
  {/if}
</main>

<style>
  #background {
    position: absolute;
    inset: 0;
    z-index: -1;

    background-image: url("https://github.com/Hbarniq/ahms/raw/main/assets/backgrounds/3.png");
    background-size: cover;
    filter: blur(3px) brightness(80%);
  }
</style>
