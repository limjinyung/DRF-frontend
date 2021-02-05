<script>
	import { onMount} from "svelte";
	import Cookies from 'js-cookie'

	export let name;

	let username="limjinyung";
	let password="password";
	let csrfToken;
  	let isAuthenticated = false;

	let task_list;
	let jwt;
	let files;
	let dataFile = null;
	let value;
	let todoUpload;

	const tokenUrl = 'http://127.0.0.1:8000/api/token';
	const apiUrl = 'http://127.0.0.1:8000/jwt'

	onMount(() => {
		fetch("http://127.0.0.1:8000/api/session/", {
			mode: 'cors',
			credentials: "include",
			method: 'GET',
		})
		.then((res) => res.json())
		.then((data) => {
			console.log(data);
		if (data.isAuthenticated == true) {
			isAuthenticated = true;
		} else {
			isAuthenticated = false;
			csrf();
		}
		})
		.catch((err) => {
			console.log(err);
		});
	});

	const csrf = () => {
		fetch("http://localhost:8000/api/csrf/", {
			credentials: "include",
			method: "GET"
		})
		.then((res) => {
			// console.log(res);
			// csrfToken = res.headers.get(["Set-Cookie"]);
			// console.log(csrfToken);
			csrfToken = Cookies.get('csrftoken');
			console.log(csrfToken);
		})
		.catch((err) => {
			console.log(err);
		});
	}


	const login = () => {
		fetch("http://localhost:8000/api/login/", {
			method: "POST",
			headers: {
				// 'Accept': 'application/json, text/javascript, */*; q=0.01',
				// "Content-Type": "application/json",
				'Accept': 'application/json',
				'Content-Type': 'application/json',
				"X-CSRFToken": csrfToken
			},
			credentials: "include",
			body: JSON.stringify({ 
				username: username,
				password: password 
			}),
		})
		.then((res) => res.json())
		.then((data) => {
			console.log(data);
			if (data.login == true) {
				isAuthenticated = true;
			}
			csrfToken = Cookies.get('csrftoken');
			console.log(csrfToken);
		})
		.catch((err) => {
			console.log(err);
		});
	}

	const logout = () => {
		fetch("http://localhost:8000/api/logout/", {
			credentials: "include",
		})
		.then(() => {
			isAuthenticated = false;
		})
		.catch((err) => {
			console.log(err);
		});
	};
		
	// onMount(() => { fetch('http://127.0.0.1:8000/api/token/', {
	// 		mode: 'cors',
	// 		credentials: 'include',
	// 		method: 'POST',
	// 		headers: {
    //             'Accept': 'application/json',
    //             'Content-Type': 'application/json'
    //         },
	// 		body: JSON.stringify({
	// 			username:username,
	// 			password:password
	// 		})
	// 	})
	// 	.then((res) => res.json())
	// 	.then((data) => {
	// 		jwt=data.refresh;
	// 		console.log(jwt);
	// 	})
	// })

	const taskList = () => {
		fetch("http://127.0.0.1:8000/api/task_list/", {
			method: "GET",
			headers: {
				'Accept': '*/*',
				'Content-Type': "application/json;",
				"X-CSRFToken": csrfToken
			},
			credentials: "include",
		})
		.then((res) => res.json())
		.then((data) => {
			task_list=data;
			console.log(task_list);
		})
		.catch((err) => {
			console.log(err);
		});
	}

	function getFileExtension(filename){
		console.log(filename.location);
		return filename.split('.').pop();
	}

	function getUploadURL(extension){
		((extension.equals("pdf") || extension.equals("txt")) ? 'http://localhost:8000/api/document_upload/' :  ((extension.equals("jpg") || extension.equals("jpeg") || extension.equals("png")) ? 'http://localhost:8000/api/image_upload/' : 'http://localhost:8000/api/error/'))  

	}
	
	function uploadDocument() {
        const formData = new FormData();
		formData.append('todo',todoUpload);
		formData.append('document', files[0]);
        const upload = fetch('http://localhost:8000/api/document_upload/', {
			method: 'POST',
			headers: {
				"X-CSRFToken": csrfToken
			},
			credentials: "include",
            body: formData
		})
		.then((response) => response.json()).then((result) => {
            console.log(result);
        })
		.catch((error) => {
			console.log(error);
		});
	}
	
	function uploadImage() {
        const formData = new FormData();
		formData.append('todo',todoUpload);
		formData.append('image', files[0]);
        const upload = fetch('http://localhost:8000/api/image_upload/', {
			method: 'POST',
			headers: {
				"X-CSRFToken": csrfToken
			},
			credentials: "include",
            body: formData
		})
		.then((response) => response.json()).then((result) => {
            console.log(result);
        })
		.catch((error) => {
			console.log(error);
		});
    }

</script>

<main>
	<!-- <h1>Hello {name}!</h1> -->

	<center>
		<div class="container">
			{#if isAuthenticated}
			<h1>You are authenticated!</h1>
			<button type="button" on:click={logout}>logout</button>
			{:else}
			<h1>Log in</h1>
			<form id="form">
				username:
				<input type="text" bind:value={username} />
				<br /><br />
				password:
				<input type="password" bind:value={password} /><br /><br />
				<button type="button" on:click={login}>login</button>
			</form>
			{/if}
		</div>
	</center>

	<hr/>

	<h1>Upload Image</h1>

	todo: <input bind:value={todoUpload}><br/>
	<input id="fileUpload" type="file" bind:files>

	{#if todoUpload && files[0]}
		<p>
			{files[0].name}
		</p>
	{/if}

	{#if todoUpload && files}
		<button on:click={uploadImage}>Submit</button>
	{:else}
		<button on:click={uploadImage} disabled>Submit</button>
	{/if}

	<hr/>

	<h1>Upload Document</h1>

	todo: <input bind:value={todoUpload}><br/>
	<input id="fileUpload" type="file" bind:files>

	{#if todoUpload && files[0]}
		<p>
			{files[0].name}
		</p>
	{/if}

	{#if todoUpload && files}
		<button on:click={uploadDocument}>Submit</button>
	{:else}
		<button on:click={uploadDocument} disabled>Submit</button>
	{/if}

	<hr/>

	<h1>Check Task</h1>

	<p>
		Todo List:
	</p>
	<button type="button" on:click={taskList}>
		Check tasks
	</button>
	<section>
	{#if task_list}
		{#each task_list.all_task as {todo,description}}
			<!-- {task_list.all_task} -->
			<p>todo: {todo}</p>
			<p>description: {description}</p>
		{/each}
	{:else}
	Loading task...
	{/if}
	</section>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>