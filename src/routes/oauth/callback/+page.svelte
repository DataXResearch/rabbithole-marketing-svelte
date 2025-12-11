<script>
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';

  let status = 'Authenticating...';
  let error = '';

  onMount(() => {
    if (!browser) return;

    // Get the full URL including hash and query params
    const fullUrl = window.location.href;
    const params = new URLSearchParams(window.location.search);
    const hashParams = new URLSearchParams(window.location.hash.slice(1));

    // Check for errors in both query and hash
    const errorParam = params.get('error') || hashParams.get('error');
    const errorDescription = params.get('error_description') || hashParams.get('error_description');

    if (errorParam) {
      error = errorDescription || errorParam;
      status = 'Authentication failed';

      if (window.opener) {
        window.opener.postMessage({
          type: 'oauth-callback',
          error: errorParam,
          errorDescription
        }, '*');
      }
      return;
    }

    // Send the full callback URL to the opener so BrowserOAuthClient can process it
    if (window.opener) {
      window.opener.postMessage({
        type: 'oauth-callback',
        callbackUrl: fullUrl
      }, '*');
      status = 'Authentication successful! This window will close automatically.';
      setTimeout(() => window.close(), 1500);
    } else {
      // Try BroadcastChannel as fallback
      const channel = new BroadcastChannel('rabbithole-oauth');
      channel.postMessage({
        type: 'oauth-callback',
        callbackUrl: fullUrl
      });
      channel.close();
      status = 'Authentication successful! You can close this window and return to the extension.';
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
