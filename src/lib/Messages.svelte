<script lang="ts">
	import type { Record } from 'pocketbase';
	import { onDestroy, onMount } from 'svelte';
	import { pb, currentUser } from './auth/pocketbase';

	let newMessage: string;
	let messages: Record[] = [];
	let unsubscribe: () => void;

	onMount(async () => {
		const resultList = await pb.collection('messages').getList(1, 50, {
			sort: 'created',
			expand: 'user'
		});
		messages = resultList.items;

		pb.collection('messages').subscribe('*', async ({ action, record }) => {
			if (action === 'create') {
				const user = await pb.collection('users').getOne(record.user);
				record.expand = { user };
				messages = [...messages, record];
			}
			if (action === 'delete') {
				messages = messages.filter((msg) => msg.id !== record.id);
			}
		});
	});

	onDestroy(() => {
		unsubscribe?.();
	});

	async function sendMessage() {
		const data = {
			text: newMessage,
			user: $currentUser?.id
		};
		await pb.collection('messages').create(data);
	}
</script>

{#if $currentUser}
	<div>
		{#each messages as message (message.id)}
			<div>
				<img
					src={`https://avatars.dicebear.com/api/identicon/${message.expand?.user?.username}.svg`}
					alt="avatar"
					width="40px"
				/>
				<small>
					Sent by @{message.expand?.user?.username}
				</small>
				<p>
					{message.text}
				</p>
			</div>
		{/each}
	</div>

	<form on:submit|preventDefault={sendMessage}>
		<input type="text" placeholder="Enter your message ..." bind:value={newMessage} />
		<button type="submit">Send</button>
	</form>
{/if}
