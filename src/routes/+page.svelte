<script>
	// @ts-nocheck

	import { initializeApp } from 'firebase/app';
	import {
		getFirestore,
		collection,
		onSnapshot,
		doc,
		addDoc,
		updateDoc,
		deleteDoc
	} from 'firebase/firestore';
	import { firebaseConfig } from '$lib/firebaseConfig';

	const firebaseApp = initializeApp(firebaseConfig);
	const db = getFirestore(firebaseApp);
	const collectionRef = collection(db, 'todos');

	let todos = [];
	let task = '';
	let editingTaskId = null;
	let editingTaskText = '';

	const unsubscribe = onSnapshot(collectionRef, (querySnapshot) => {
		let firebaseTodos = [];

		querySnapshot.forEach((doc) => {
			let todo = { ...doc.data(), id: doc.id };
			firebaseTodos = [todo, ...firebaseTodos];
		});
		todos = firebaseTodos;
	});

	const addTodo = async () => {
		if (task !== '') {
			const docRef = await addDoc(collection(db, 'todos'), {
				task: task,
				isComplete: false,
				createdAt: new Date()
			});
		}
		task = '';
	};

	const handleKeyPress = (event) => {
		if (event.key === 'Enter') {
			addTodo();
		}
	};

	const markTodoComplete = async (item) => {
		await updateDoc(doc(db, 'todos', item.id), {
			isComplete: !item.isComplete
		});
	};

	const deleteTodo = async (id) => {
		await deleteDoc(doc(db, 'todos', id));
	};

	const startEditingTask = (item) => {
		editingTaskId = item.id;
		editingTaskText = item.task;
	};

	const cancelEditing = () => {
		editingTaskId = null;
		editingTaskText = '';
	};

	const updateTask = async (id) => {
		if (editingTaskText.trim() !== '') {
			await updateDoc(doc(db, 'todos', id), {
				task: editingTaskText
			});
			cancelEditing();
		}
	};

	const handleEditingKeyPress = (event, id) => {
		if (event.key === 'Enter') {
			updateTask(id);
		}
	};
</script>

<div class="container">
	<h1>Todo List</h1>
	<div class="input-group">
		<input type="text" on:keypress={handleKeyPress} placeholder="Add Task" bind:value={task} />
		<button on:click={addTodo}>Add Task</button>
	</div>
	<ol>
		{#each todos as item}
			<li class:complete={item.isComplete}>
				{#if editingTaskId === item.id}
					<input
						type="text"
						bind:value={editingTaskText}
						on:keypress={(e) => handleEditingKeyPress(e, item.id)}
					/>
					<button on:click={() => updateTask(item.id)}>Update</button>
					<button on:click={cancelEditing}>Cancel</button>
				{:else}
					<span class="task">{item.task}</span>
					<div class="actions">
						<button class="complete-btn" on:click={() => markTodoComplete(item)}>✔</button>
						<button class="edit-btn" on:click={() => startEditingTask(item)}>✏️</button>
						<button class="delete-btn" on:click={() => deleteTodo(item.id)}>🗑</button>
					</div>
				{/if}
			</li>
		{:else}
			<h3>No Task(s) right now</h3>
		{/each}
	</ol>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

	:global(body) {
		font-family: 'Roboto', sans-serif;
		background: #e5e5e6;
		margin: 0;
		padding: 0;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100vh;
	}

	.container {
		background: rgb(202, 193, 193);
		padding: 2rem;
		border-radius: 10px;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
		max-width: 400px;
		width: 100%;
		text-align: center;
	}

	h1 {
		margin-bottom: 1rem;
		color: #272727;
	}

	.input-group {
		display: flex;
		margin-bottom: 1rem;
	}

	input[type='text'] {
		flex: 1;
		padding: 0.5rem;
		border: 1px solid #9c7676;
		border-radius: 5px 0 0 5px;
		outline: none;
	}

	button {
		padding: 0.5rem 1rem;
		border: none;
		background: #007bff;
		color: white;
		cursor: pointer;
		border-radius: 0 5px 5px 0;
	}

	button:hover {
		background: #325b86;
	}

	ol {
		list-style: none;
		padding: 0;
		margin: 0;
	}

	li {
		background: #f9f9f9;
		margin-bottom: 0.5rem;
		padding: 0.5rem;
		border-radius: 5px;
		display: flex;
		justify-content: space-between;
		align-items: center;
		transition: background 0.3s;
	}

	li:hover {
		background: #f1f1f1;
	}

	li.complete {
		background: #d1e7dd;
		text-decoration: line-through;
		color: #155724;
	}

	.task {
		flex: 1;
	}

	.actions {
		display: flex;
		gap: 0.5rem;
	}

	.complete-btn {
		background: #28a745;
		color: white;
		border: none;
		border-radius: 5px;
		padding: 0.3rem 0.5rem;
		cursor: pointer;
	}

	.complete-btn:hover {
		background: #218838;
	}

	.delete-btn {
		background: #dc3545;
		color: white;
		border: none;
		border-radius: 5px;
		padding: 0.3rem 0.5rem;
		cursor: pointer;
	}

	.delete-btn:hover {
		background: #c82333;
	}

	.edit-btn {
		background: #ffc107;
		color: white;
		border: none;
		border-radius: 5px;
		padding: 0.3rem 0.5rem;
		cursor: pointer;
	}

	.edit-btn:hover {
		background: #e0a800;
	}
</style>
