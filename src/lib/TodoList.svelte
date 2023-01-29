<script>
	import Button from './Button.svelte';
	import { createEventDispatcher } from 'svelte';

	export let todos = [];
	let inputText = '';

	const dispatch = createEventDispatcher();

	function handleAddTodo() {
		const isNotCancelled = dispatch('addTodo', { title: inputText }, { cancelable: false });
		if (isNotCancelled) {
			inputText = '';
		}
	}

	function handleRemoveTodo(id) {
		dispatch('removeTodo', { id });
	}

	function handleToggleTodo(id, value) {
		dispatch('toggleTodo', { id, value });
	}
</script>

<div class="todo-list-wrapper">
	<ul>
		{#each todos as { id, title, completed } (id)}
			<li>
				<label
					><input
						on:input={(e) => {
							e.currentTarget.checked = completed; //Forzando a que no cambie el estado
							handleToggleTodo(id, !completed); //Dispatch de evento
						}}
						type="checkbox"
						checked={completed}
					/>{title}</label
				>
				<button on:click={() => handleRemoveTodo(id)}>Remove</button>
			</li>
		{/each}
	</ul>
	<form class="add-todo-form" on:submit|preventDefault={handleAddTodo}>
		<input bind:value={inputText} type="text" />
		<Button type="submit" disabled={!inputText}>Add</Button>
	</form>
</div>

<style></style>
