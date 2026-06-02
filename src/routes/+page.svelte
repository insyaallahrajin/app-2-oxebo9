<script lang="ts">
  import { supabase } from '$lib/supabase';
  import type { Todo } from '$lib/types';
  import type { User } from '@supabase/supabase-js';
  import Auth from '$lib/components/Auth.svelte';
  import TodoItem from '$lib/components/TodoItem.svelte';

  let { data }: { data: { user: User | null } } = $props();

  let todos = $state<Todo[]>([]);
  let loadingTodos = $state(true);
  let newTitle = $state('');
  let adding = $state(false);
  let filter = $state<'all' | 'active' | 'completed'>('all');

  let filteredTodos = $derived(() => {
    if (filter === 'active') return todos.filter(t => !t.completed);
    if (filter === 'completed') return todos.filter(t => t.completed);
    return todos;
  });

  let activeCount = $derived(todos.filter(t => !t.completed).length);
  let completedCount = $derived(todos.filter(t => t.completed).length);

  $effect(() => {
    if (!data.user) return;

    loadTodos();

    const channel = supabase.channel(`todos_${data.user.id}`)
      .on('postgres_changes',
        { event: '*', schema: 'public', table: 'todos', filter: `user_id=eq.${data.user.id}` },
        () => { loadTodos(); }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  });

  async function loadTodos() {
    loadingTodos = true;
    const { data: todosData, error } = await supabase
      .from('todos')
      .select('*')
      .order('created_at', { ascending: false });

    if (!error && todosData) {
      todos = todosData as Todo[];
    }
    loadingTodos = false;
  }

  async function addTodo() {
    const title = newTitle.trim();
    if (!title || !data.user) return;

    adding = true;
    const { error } = await supabase
      .from('todos')
      .insert({ title, user_id: data.user.id, completed: false });

    if (!error) {
      newTitle = '';
      await loadTodos();
    }
    adding = false;
  }

  async function updateTodo(id: string, updates: Partial<Todo>) {
    const { error } = await supabase
      .from('todos')
      .update(updates)
      .eq('id', id);

    if (!error) {
      todos = todos.map(t => t.id === id ? { ...t, ...updates } : t);
    }
  }

  async function deleteTodo(id: string) {
    const { error } = await supabase
      .from('todos')
      .delete()
      .eq('id', id);

    if (!error) {
      todos = todos.filter(t => t.id !== id);
    }
  }

  async function clearCompleted() {
    const completedIds = todos.filter(t => t.completed).map(t => t.id);
    if (completedIds.length === 0) return;

    const { error } = await supabase
      .from('todos')
      .delete()
      .in('id', completedIds);

    if (!error) {
      todos = todos.filter(t => !t.completed);
    }
  }

  function handleKeydown(e: KeyboardEvent) {
    if (e.key === 'Enter') addTodo();
  }
</script>

{#if !data.user}
  <Auth />
{:else}
  <div class="space-y-6">
    <!-- Header -->
    <div class="text-center">
      <h1 class="text-4xl font-bold tracking-tight text-gray-900">todos</h1>
      <p class="mt-1 text-sm text-gray-500">Stay organized, get things done</p>
    </div>

    <!-- Add Todo Form -->
    <div class="flex gap-2">
      <div class="relative flex-1">
        <input
          type="text"
          bind:value={newTitle}
          onkeydown={handleKeydown}
          placeholder="What needs to be done?"
          disabled={adding}
          class="block w-full rounded-lg border border-gray-300 bg-white px-4 py-3 pr-10 text-sm text-gray-900 shadow-sm placeholder:text-gray-400 focus:border-indigo-500 focus:outline-none focus:ring-1 focus:ring-indigo-500 disabled:opacity-50"
        />
        {#if newTitle.trim()}
          <button
            type="button"
            onclick={() => { newTitle = ''; }}
            class="absolute right-2 top-1/2 -translate-y-1/2 rounded p-1 text-gray-400 hover:text-gray-600"
            aria-label="Clear input"
          >
            <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        {/if}
      </div>
      <button
        type="button"
        onclick={addTodo}
        disabled={!newTitle.trim() || adding}
        class="flex items-center gap-2 rounded-lg bg-indigo-600 px-5 py-3 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed transition"
      >
        {#if adding}
          <svg class="h-4 w-4 animate-spin" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
        {:else}
          <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 4v16m8-8H4" />
          </svg>
        {/if}
        Add
      </button>
    </div>

    <!-- Filters & Stats -->
    <div class="flex flex-wrap items-center justify-between gap-3 text-sm text-gray-500">
      <span class="text-xs sm:text-sm">{activeCount} item{activeCount !== 1 ? 's' : ''} left</span>
      <div class="flex gap-1 rounded-lg bg-gray-100 p-0.5">
        {#each ['all', 'active', 'completed'] as f}
          <button
            type="button"
            onclick={() => filter = f as typeof filter}
            class="rounded-md px-3 py-1 text-xs font-medium transition sm:text-sm"
            class:bg-white={filter === f}
            class:text-gray-900={filter === f}
            class:text-gray-500={filter !== f}
            class:shadow-sm={filter === f}
          >
            {f.charAt(0).toUpperCase() + f.slice(1)}
          </button>
        {/each}
      </div>
      {#if completedCount > 0}
        <button
          type="button"
          onclick={clearCompleted}
          class="text-xs text-gray-400 hover:text-red-500 sm:text-sm"
        >
          Clear completed ({completedCount})
        </button>
      {/if}
    </div>

    <!-- Todo List -->
    {#if loadingTodos}
      <div class="flex justify-center py-12">
        <div class="h-8 w-8 animate-spin rounded-full border-3 border-indigo-200 border-t-indigo-600"></div>
      </div>
    {:else if filteredTodos().length === 0}
      <div class="flex flex-col items-center justify-center rounded-lg border-2 border-dashed border-gray-200 py-16 text-center">
        {#if filter === 'all' && todos.length === 0}
          <svg class="h-12 w-12 text-gray-300" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.5">
            <path stroke-linecap="round" stroke-linejoin="round" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4" />
          </svg>
          <p class="mt-3 text-sm font-medium text-gray-900">No todos yet</p>
          <p class="mt-1 text-xs text-gray-500">Add one above to get started!</p>
        {:else if filter === 'active'}
          <p class="text-sm text-gray-500">🎉 All done! No active items.</p>
        {:else}
          <p class="text-sm text-gray-500">No completed items yet.</p>
        {/if}
      </div>
    {:else}
      <ul class="space-y-2" role="list">
        {#each filteredTodos() as todo (todo.id)}
          <TodoItem {todo} {onUpdate} {onDelete} />
        {/each}
      </ul>
    {/if}
  </div>
{/if}