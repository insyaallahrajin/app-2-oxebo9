<script lang="ts">
  import { supabase } from '$lib/supabase';

  let email = $state('');
  let password = $state('');
  let isLogin = $state(true);
  let loading = $state(false);
  let errorMessage = $state('');

  async function handleSubmit(e: Event) {
    e.preventDefault();
    loading = true;
    errorMessage = '';

    try {
      if (isLogin) {
        const { error } = await supabase.auth.signInWithPassword({ email, password });
        if (error) throw error;
      } else {
        const { error } = await supabase.auth.signUp({ email, password });
        if (error) throw error;
        errorMessage = 'Check your email for confirmation!';
      }
    } catch (err: unknown) {
      if (err instanceof Error) errorMessage = err.message;
      else errorMessage = 'An unexpected error occurred';
    } finally {
      loading = false;
    }
  }
</script>

<div class="flex min-h-full flex-col justify-center py-12 sm:px-6 lg:px-8">
  <div class="sm:mx-auto sm:w-full sm:max-w-md">
    <h2 class="text-center text-3xl font-bold tracking-tight text-gray-900">
      {isLogin ? 'Sign in to your account' : 'Create a new account'}
    </h2>
    <p class="mt-2 text-center text-sm text-gray-600">
      {#if isLogin}
        Don't have an account?
        <button type="button" class="font-medium text-indigo-600 hover:text-indigo-500" onclick={() => { isLogin = false; errorMessage = ''; }}>
          Sign up
        </button>
      {:else}
        Already have an account?
        <button type="button" class="font-medium text-indigo-600 hover:text-indigo-500" onclick={() => { isLogin = true; errorMessage = ''; }}>
          Sign in
        </button>
      {/if}
    </p>
  </div>

  <div class="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
    <div class="bg-white px-6 py-8 shadow sm:rounded-lg">
      <form class="space-y-6" onsubmit={handleSubmit}>
        <div>
          <label for="email" class="block text-sm font-medium text-gray-700">Email address</label>
          <input
            id="email"
            type="email"
            required
            bind:value={email}
            class="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 text-gray-900 shadow-sm placeholder:text-gray-400 focus:border-indigo-500 focus:outline-none focus:ring-1 focus:ring-indigo-500 sm:text-sm"
            placeholder="you@example.com"
          />
        </div>

        <div>
          <label for="password" class="block text-sm font-medium text-gray-700">Password</label>
          <input
            id="password"
            type="password"
            required
            minlength={6}
            bind:value={password}
            class="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2 text-gray-900 shadow-sm placeholder:text-gray-400 focus:border-indigo-500 focus:outline-none focus:ring-1 focus:ring-indigo-500 sm:text-sm"
            placeholder="••••••••"
          />
        </div>

        {#if errorMessage}
          <div class="rounded-md bg-red-50 p-3 text-sm text-red-700" role="alert">
            {errorMessage}
          </div>
        {/if}

        <button
          type="submit"
          disabled={loading}
          class="flex w-full justify-center rounded-md bg-indigo-600 px-4 py-2 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {#if loading}
            <svg class="animate-spin -ml-1 mr-2 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            Processing...
          {:else}
            {isLogin ? 'Sign in' : 'Sign up'}
          {/if}
        </button>
      </form>
    </div>
  </div>
</div>