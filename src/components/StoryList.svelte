<script>
	import { counters, maxStories } from '../preferences';
	import { fetchStoryIDs, fetchItem } from '../hn-api';
	import Loader from './Loader.svelte';
	import Story from './Story.svelte';

	export let list;

	const pageNum = Number.parseInt(new URLSearchParams(window.location.search).get('page'), 10) || 1;
	const pageLength = Number.parseInt($maxStories);

	// Indices of first and last items on the current page
	const first = pageLength * (pageNum - 1);
	const last = first + pageLength;

	const stories = fetchStoryIDs(list)
		.then(ids => ids.slice(first, last)) // Pagination
		.then(ids => ids.map(id => fetchItem(id)))
		.then(stories => Promise.all(stories))
		.then(stories => stories.filter(story => story != null));
	// TODO: prevent null/undefined stories from messing up story numbers
</script>

{#await stories}
	<Loader />
{:then stories}
	<ol start={first + 1} class:counters={$counters} style="counter-reset: story-count {first}">
		{#each stories as story (story.id)}
			<li><Story story={story} /></li>
		{/each}
	</ol>
	<div class="pagination-buttons">
		{#if pageNum > 1}
			<a href="?page={pageNum - 1}">Previous Page</a>
		{/if}
		<a href="?page={pageNum + 1}">Next Page</a>
	</div>
{:catch error}
	<code>{error}</code>
{/await}

<style>
	ol {
		margin: 0;
		padding: 0;
		list-style: none;
	}

	li {
		counter-increment: story-count;
	}
	
	.pagination-buttons {
		text-align: center;
		margin-top: 1.5em;
		color: var(--color-textlight);
	}
	
	.pagination-buttons a {
		text-decoration: none;
		margin: 0 0.25em;
	}
</style>
