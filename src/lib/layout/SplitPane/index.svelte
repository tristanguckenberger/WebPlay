<svelte:options accessors />

<script>
	import { afterUpdate, onDestroy } from 'svelte';
	import { clearSplit } from '$lib/stores/splitStore';
	import Split from 'split.js';
	import { paneMinHeightModifier } from '$lib/stores/layoutStore';
	/**
	 * @type {any[]}
	 */
	 export let panes;
	/**
	 * @type {null}
	 */
	 export let sizes = null;
	export let vertical = false;

	/**
	 * @type {Split.Instance}
	 */
	let splitInstance;

	const reloadSplit = () => {
		// Destory existing splitInstance
		if (splitInstance) splitInstance?.destroy(false, false);

		// Init a new split instance
		splitInstance = Split(panes, {
			direction: vertical ? 'vertical' : 'horizontal',
			gutterSize: 10,
			// @ts-ignore
			sizes,
			minSize: $paneMinHeightModifier
		});
	};

	/**
	 * @type {HTMLDivElement | null}
	 */
	let split;

	afterUpdate(() => {
		if ($clearSplit) {
			reloadSplit();
			setTimeout(() => {
				clearSplit?.set(false);
			}, 100);
		}
	});

	onDestroy(() => {
		split = null;
		splitInstance?.destroy();
	});

	// Replaces functionality found in onMount
	$: if (split) {
		panes?.map(async (query) => {
			await split?.querySelector(query).classList.add('pane');
		});

		// Init Split.js
		setTimeout(() => reloadSplit(), 50);
	}
</script>

<div class="split {vertical ? 'vertical' : ''}" bind:this={split}>
	<slot />
</div>

<style>
	/* TODO: REWRITE WITH SCSS */
	.split {
		display: flex;
		flex-direction: row;
		height: 100%;
		width: 100%;
		max-width: 100vw;
	}
	.split.vertical {
		display: block;
		flex-direction: unset;
	}

	:global(.gutter) {
		background-color: transparent;
		border-radius: 3px;
		background-repeat: no-repeat;
		background-position: 50%;
		z-index: 2147483647;
		transition: background-color 250ms ease;
	}
	:global(.gutter:hover) {
		background-color: #4ca5ff;
	}

	:global(.gutter.gutter-horizontal) {
		cursor: col-resize;
	}

	:global(.gutter.gutter-vertical) {
		cursor: row-resize;
	}
</style>
