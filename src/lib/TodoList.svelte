<script>
	import Button from './Button.svelte';
	import { createEventDispatcher, afterUpdate } from 'svelte';
	import FaRegTrashAlt from 'svelte-icons/fa/FaRegTrashAlt.svelte';
	import { scale, crossfade } from 'svelte/transition';
	import { flip } from 'svelte/animate';

	afterUpdate(() => {
		if (scrollOnAdd) {
			let pos;
			if (scrollOnAdd === 'top') pos = 0;

			if (scrollOnAdd === 'bottom') pos = listDivScrollHeight;

			if (autoScroll) listDiv.scrollTo(0, pos); //Se ejecuta despues de actualizar el componente
			autoScroll = false;
		}
	});

	const [send, receive] = crossfade({
		duration: 400,
		fallback(node) {
			return scale(node, { start: 0.5, duration: 300 });
		}
	});

	export let todos = null;
	export let error = null;
	export let isLoading = false;
	export let disableAdding = false;
	export let disabledItems = [];
	export let scrollOnAdd = undefined;

	$: done = todos ? todos.filter((todo) => todo.completed) : [];
	$: todo = todos ? todos.filter((todo) => !todo.completed) : [];

	let prevTodos = todos;
	let inputText = '';
	let input, listDiv, autoScroll, listDivScrollHeight;

	const dispatch = createEventDispatcher();

	$: {
		autoScroll = todos && prevTodos && todos.length > prevTodos.length;
		prevTodos = todos; // Se ejecuta cuando cambia la propiedad todos
	}

	export function clearInput() {
		inputText = '';
	}
	export function focusInput() {
		input.focus();
	}

	function handleAddTodo() {
		const isNotCancelled = dispatch('addTodo', { title: inputText }, { cancelable: true });
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
	{#if isLoading}
		<p class="state-text">Loading...</p>
	{:else if error}
		<p class="state-text">{error}</p>
	{:else if todos}
		<div class="todo-list" bind:this={listDiv}>
			<div bind:offsetHeight={listDivScrollHeight}>
				{#if todos.length === 0}
					<p class="state-text">No todos Yet</p>
				{:else}
					<div style:display="flex">
						{#each [todo, done] as list, index}
							<div class="list-wrapper">
								<h2>{index === 0 ? 'Todos' : 'Done'}</h2>
								<ul>
									{#each list as todo (todo.id)}
										{@const { id, title, completed } = todo}
										<li animate:flip={{ duration: 300 }}>
											<slot {todo}>
												<div
													in:receive|local={{ key: id }}
													out:send|local={{ key: id }}
													class:completed
												>
													<label>
														<input
															disabled={disabledItems.includes(id)}
															on:input={(e) => {
																e.currentTarget.checked = completed; //Forzando a que no cambie el estado
																handleToggleTodo(id, !completed); //Dispatch de evento
															}}
															type="checkbox"
															checked={completed}
														/>
														{title}
													</label>
													<button
														class="remove-todo-button"
														aria-label="Remove Todo: {title}"
														disabled={disabledItems.includes(id)}
														on:click={() => handleRemoveTodo(id)}
													>
														<span
															style:width="10px"
															style:display="inline-block"
															><FaRegTrashAlt /></span
														>
													</button>
												</div>
											</slot>
										</li>
									{/each}
								</ul>
							</div>
						{/each}
					</div>
				{/if}
			</div>
		</div>
	{/if}
	<form class="add-todo-form" on:submit|preventDefault={handleAddTodo}>
		<input
			disabled={disableAdding || !todos}
			bind:this={input}
			bind:value={inputText}
			type="text"
			placeholder="New Todo"
		/>
		<Button
			class="add-todo-button"
			type="submit"
			disabled={!inputText || disableAdding || !todos}
		>
			Add
		</Button>
	</form>
</div>

<style lang="scss">
	.todo-list-wrapper {
		background-color: #424242;
		border: 1px solid #4b4b4b;
		.state-text {
			margin: 0;
			padding: 15px;
			text-align: center;
			color: var(--buttonTextColor);
		}
		.todo-list {
			max-height: 400px;
			overflow: auto;
			color: var(--buttonTextColor);
			.list-wrapper {
				padding: 10px;
				flex: 1;
				h2 {
					margin: 0 0 10px 0;
				}
			}
			ul {
				margin: 0;
				padding: 0;
				list-style: none;
				li > div {
					margin-bottom: 5px;
					display: flex;
					align-items: center;
					background-color: #303030;
					border-radius: 5px;
					padding: 10px;
					position: relative;
					label {
						cursor: pointer;
						font-size: 18px;
						display: flex;
						align-items: baseline;
						padding-right: 20px;
						input[type='checkbox'] {
							margin-right: 10px;
							cursor: pointer;
							&:disabled {
								cursor: not-allowed;
							}
						}
					}
					&.completed {
						opacity: 0.5;
						text-decoration: line-through;
					}
					.remove-todo-button {
						position: absolute;
						right: 10px;
						padding: 5px;
						background: none;
						border: none;
						cursor: pointer;
						display: none;
						&:disabled {
							cursor: not-allowed;
						}
						:global(svg) {
							fill: #bd1414;
						}
					}
					&:hover {
						.remove-todo-button {
							display: block;
						}
					}
				}
			}
		}
		.add-todo-form {
			display: flex;
			background-color: #303030;
			align-items: center;
			padding: 15px;
			flex-wrap: wrap;
			border-top: 2px solid #4b4b4b;
			// :global(.add-todo-button) {
			// 	background-color: aqua;
			// }
			input {
				flex: 1;
				background-color: #424242;
				padding: 10px;
				margin-right: 10px;
				border: 1px solid #4b4b4b;
				border-radius: 5px;
				font-size: 18px;
				color: var(--buttonTextColor);
			}
		}
	}
</style>
