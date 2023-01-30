<script>
	import TodoList from './lib/TodoList.svelte';
	import { v4 as uuid } from 'uuid';
	import { tick } from 'svelte';
	import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';
	import spin from './lib/transitions/spin';
	import fade from './lib/transitions/fade';

	let todoList;
	let showList = true;

	let todos = null;
	let error = null;
	let isLoading = false;
	let isAdding = false;
	let disabledItems = [];

	onMount(() => {
		loadTodos();
	});

	async function loadTodos() {
		isLoading = true;
		await fetch('https://jsonplaceholder.typicode.com/todos?_limit=10').then(
			async (response) => {
				if (response.ok) {
					todos = await response.json();
				} else {
					error = 'An error has occurred.';
				}
			}
		);
		isLoading = false;
	}

	async function handleAddTodo(event) {
		event.preventDefault();
		isAdding = true;
		await fetch('https://jsonplaceholder.typicode.com/todos', {
			method: 'POST',
			body: JSON.stringify({
				title: event.detail.title,
				completed: false
			}),
			headers: {
				'Content-type': 'application/json; charset=UTF-8'
			}
		}).then(async (response) => {
			if (response.ok) {
				const todo = await response.json();
				todos = [{ ...todo, id: uuid() }, ...todos];
				todoList.clearInput();
			} else {
				throw new Error('An error has occurred.');
			}
		});
		isAdding = false;
		await tick();
		todoList && todoList.focusInput();
	}

	async function handleRemoveTodo(event) {
		const id = event.detail.id;
		if (disabledItems.includes(id)) return; // if item is already being deleted, return
		disabledItems = [...disabledItems, id]; // add item to disabledItems
		await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
			method: 'DELETE'
		}).then((response) => {
			if (response.ok) {
				todos = todos.filter((todo) => todo.id !== event.detail.id);
			} else {
				console.log('An error has occurred.');
			}
		});
		disabledItems = disabledItems.filter((itemId) => itemId !== id); // remove item from disabledItems
	}

	async function handleToggleTodo(event) {
		const id = event.detail.id;
		if (disabledItems.includes(id)) return; // if item is already being deleted, return
		disabledItems = [...disabledItems, id]; // add item to disabledItems
		await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
			method: 'DELETE'
		}).then((response) => {
			if (response.ok) {
				todos = todos.map((todo) => {
					if (todo.id === event.detail.id) {
						return { ...todo, completed: event.detail.value };
					}
					return todo;
				});
			} else {
				console.log('An error has occurred.');
			}
		});
		disabledItems = disabledItems.filter((itemId) => itemId !== id); // remove item from disabledItems
	}
</script>

<label for="">
	<input type="checkbox" bind:checked={showList} />
	Show/Hide List
</label>

{#if showList}
	<div style:max-width="800px">
		<TodoList
			{todos}
			{error}
			{isLoading}
			{disabledItems}
			disableAdding={isAdding}
			scrollOnAdd="top"
			bind:this={todoList}
			on:addTodo={handleAddTodo}
			on:removeTodo={handleRemoveTodo}
			on:toggleTodo={handleToggleTodo}
			let:todo
		/>
	</div>
	{#if todos}
		<p>
			Number of todos:{#key todos.length}<span
					style:display="inline-block"
					in:fly|local={{ y: -10 }}>{todos.length}</span
				>
			{/key}
		</p>
	{/if}
{/if}

<style></style>
