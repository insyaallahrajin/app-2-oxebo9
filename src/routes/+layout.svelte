<script module>
  import type { User } from '@supabase/supabase-js';
}
  export type LayoutChildren = (data: { user: User | null }) => import('svelte').Snippet;

  export function load() {
    return {};
  }
</script>

<script lang="ts">
  import { supabase } from '$lib/supabase';
  import type { User } from '@supabase/supabase-js';
  import { browser } from '$app/environment';
  import '../app.css';

  let user = $state<User | null>(null);
  let loading = $state(true);

  $effect(() => {
    if (!browser) return;

    supabase.auth.getSession().then(({ data: { session } }) => {
      user = session?.user ?? null;
      loading = false;
    });

    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      user = session?.user ?? null;
      loading = false;
    });

    return () => {
      subscription.unsubscribe();
    };
  });

  function handleSignOut() {
    supabase.auth.signOut();
  }
</script>

{#if loading}
  <div class="flex min-h-screen items-center justify-center bg-gray-50">
    <div class="flex flex-col items-center gap-3">
      <div class="h-10 w-10 animate-spin rounded-full border-4 border-indigo-200 border-t-indigo-600"></div>
      <p class="text-sm text-gray-500">Loading...</p>
    </div>
  </div>
{:else}
  <div class="min-h-screen bg-gray-50">
    {#if user}
      <header class="border-b border-gray-200 bg-white shadow-sm">
        <div class="mx-auto flex max-w-3xl items-center justify-between px-4 py-3 sm:px-6">
          <div class="flex items-center gap-3">
            <div class="flex h-8 w-8 items-center justify-center rounded-full bg-indigo-100 text-sm font-semibold text-indigo-600">
              {user.email?.charAt(0).toUpperCase() ?? '?'}
            </div>
            <span class="text-sm text-gray-600 truncate max-w-[180px] sm:max-w-xs">{user.email}</span>
          </div>
          <button
            type="button"
            onclick={handleSignOut}
            class="rounded-md bg-white px-3 py-1.5 text-sm font-medium text-gray-700 ring-1 ring-gray-300 hover:bg-gray-50 hover:text-gray-900"
          >
            Sign out
          </button>
        </div>
      </header>
    {/if}
    <main class="mx-auto max-w-3xl px-4 py-8 sm:px-6">
      {#key user?.id}{@render children?.({ user })}{/key}
    </main>
  </div>
{/if}