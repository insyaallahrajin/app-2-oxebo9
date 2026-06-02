<script lang="ts">
  import type { Todo } from '$lib/types';

  let { todo, onUpdate, onDelete }: {
    todo: Todo;
    onUpdate: (id: string, updates: Partial<Todo>) => void;
    onDelete: (id: string) => void;
  } = $props();

  let editing = $state(false);
  let editTitle = $state(todo.title);

  function handleToggle() {
    onUpdate(todo.id, { completed: !todo.completed });
  }

  function handleEdit() {
    if (editTitle.trim() && editTitle.trim() !== todo.title) {
      onUpdate(todo.id, { title: editTitle.trim() });
    }
    editing = false;
  }

  function handleCancel() {
    editTitle = todo.title;
    editing = false;
  }

  function handleKeydown(e: KeyboardEvent) {
    if (e.key === 'Enter') handleEdit();
    if (e.key === 'Escape') handleCancel();
  }
</script>

<li class="group flex items-center gap-3 rounded-lg border border-gray-200 bg-white px-4 py-3 shadow-sm transition hover:shadow-md">
  <button
    type="button"
    onclick={handleToggle}
    class="flex h-5 w-5 shrink-0 items-center justify-center rounded-full border-2 transition"
    class:border-indigo-500={!todo.completed}
    class:bg-indigo-500={todo.completed}
    class:border-transparent={todo.completed}
    aria-label={todo.completed ? 'Mark as incomplete' : 'Mark as complete'}
  >
    {#if todo.completed}
      <svg class="h-3 w-3 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="3">
        <path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
      </svg>
    {/if}
  </button>

  {#if editing}
    <input
      type="text"
      bind:value={editTitle}
      onkeydown={handleKeydown}
      onblur={handleEdit}
      autofocus
      class="flex-1 rounded border border-gray-300 px-2 py-1 text-sm text-gray-900 focus:border-indigo-500 focus:outline-none focus:ring-1 focus:ring-indigo-500"
    />
  {:else}
    <span
      class="flex-1 cursor-pointer text-sm select-none"
      class:line-through={todo.completed}
      class:text-gray-400={todo.completed}
      class:text-gray-900={!todo.completed}
      ondblclick={() => { editTitle = todo.title; editing = true; }}
    >
      {todo.title}
    </span>
  {/if}

  <div class="flex shrink-0 gap-1 opacity-0 transition-opacity group-hover:opacity-100">
    <button
      type="button"
      onclick={() => { editTitle = todo.title; editing = true; }}
      class="rounded p-1 text-gray-400 hover:bg-gray-100 hover:text-gray-600"
      aria-label="Edit todo"
    >
      <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
        <path stroke-linecap="round" stroke-linejoin="round" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
      </svg>
    </button>
    <button
      type="button"
      onclick={() => onDelete(todo.id)}
      class="rounded p-1 text-gray-400 hover:bg-red-50 hover:text-red-500"
      aria-label="Delete todo"
    >
      <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
        <path stroke-linecap="round" stroke-linejoin="round" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
      </svg>
    </button>
  </div>
</li>