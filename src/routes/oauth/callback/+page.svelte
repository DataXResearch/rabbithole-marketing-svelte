<script>
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';
  import { page } from '$app/stores';

  let status = 'Authenticating...';
  let error = '';

  onMount(() => {
    if (!browser) return;

    const params = $page.url.searchParams;
    const code = params.get('code');
    const state = params.get('state');
    const errorParam = params.get('error');
    const errorDescription = params.get('error_description');

    if (errorParam) {
      error = errorDescription || errorParam;
      status = 'Authentication failed';
      return;
    }

    if (code) {
      // Send message to opener window (the extension popup or tab that initiated OAuth)
      if (window.opener) {
        window.opener.postMessage({
          type: 'oauth-callback',
          code,
          state
        }, '*');
        status = 'Authentication successful! You can close this window.';
        setTimeout(() => window.close(), 1000);
      } else {
        // Try to communicate with extension via BroadcastChannel
        const channel = new BroadcastChannel('rabbithole-oauth');
        channel.postMessage({
          type: 'oauth-callback',
          code,
          state
        });
        channel.close();
        status = 'Authentication successful! You can close this window and return to the extension.';
      }
    } else {
      error = 'No authorization code received';
      status = 'Authentication failed';
    }
  });
</script>

<svelte:head>
  <title>Rabbithole - OAuth Callback</title>
</svelte:head>

<div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-purple-900 to-indigo-900">
  <div class="text-center p-8 bg-white/10 rounded-2xl backdrop-blur-sm">
    <h1 class="text-2xl font-bold text-white mb-4">Rabbithole</h1>
    
    {#if error}
      <div class="text-red-300 mb-4">
        <p class="font-medium">Error</p>
        <p class="text-sm">{error}</p>
      </div>
    {:else}
      <p class="text-white/80">{status}</p>
    {/if}
    
    <button
      on:click={() => window.close()}
      class="mt-6 px-6 py-2 bg-white/20 hover:bg-white/30 text-white rounded-full transition-colors"
    >
      Close Window
    </button>
  </div>
</div>
