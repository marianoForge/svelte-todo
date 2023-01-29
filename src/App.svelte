<script>
	import TodoList from './lib/TodoList.svelte';
	import { v4 as uuid } from 'uuid';

	let todos = [
		{ id: '1', title: 'Learn Svelte', completed: true },
		{ id: '2', title: 'Learn Svelte 2', completed: false },
		{ id: '3', title: 'Learn Svelte 3', completed: false }
	];

	function handleAddTodo(event) {
		event.preventDefault();
		todos = [...todos, { id: uuid(), title: event.detail.title, completed: false }];
	}

	function handleRemoveTodo(event) {
		todos = todos.filter((todo) => todo.id !== event.detail.id);
	}

	function handleToggleTodo(event) {
		todos = todos.map((todo) => {
			if (todo.id === event.detail.id) {
				return { ...todo, completed: event.detail.value };
			}
			console.log(event.detail);
			return todo;
		});
	}
</script>

<TodoList
	{todos}
	on:addTodo={handleAddTodo}
	on:removeTodo={handleRemoveTodo}
	on:toggleTodo={handleToggleTodo}
/>

<style></style>
