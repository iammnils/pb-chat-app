<script lang="ts">
	import { currentUser, pb } from './pocketbase';

	let username: string;
	let password: string;

	async function login() {
		await pb.collection('users').authWithPassword(username, password);
	}

	async function signUp() {
		try {
			const data = {
				username,
				password,
				passwordConfirm: password
			};

			const createdUser = await pb.collection('users').create(data);
			await login();
		} catch (err) {
			console.error(err);
		}
	}

	function signOut() {
		pb.authStore.clear();
	}
</script>

{#if $currentUser}
	<p>Signed in as {$currentUser.username}</p>
	<button on:click={signOut}>Sign Out</button>
{:else}
	<form on:submit|preventDefault>
		<label>
			Username
			<input
				type="text"
				placeholder="What's your name?"
				minlength="2"
				maxlength="20"
				required
				bind:value={username}
			/>
		</label>
		<label>
			Password
			<input
				type="password"
				placeholder="Keep it a secret"
				minlength="8"
				required
				bind:value={password}
			/>
		</label>

		<button on:click={signUp}>Sign Up</button>
		<button on:click={login}>Login</button>
	</form>
{/if}
