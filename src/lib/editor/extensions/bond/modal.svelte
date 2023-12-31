<script lang="ts">
	import { Dialog, DialogOverlay, DialogTitle } from '@rgossiaux/svelte-headlessui';
	import { bondModalState } from './store';
	import { localStorage } from '$lib/localStorage';
	import { trpc } from '$lib/trpc/client';
	import type { RouterOutput } from '$lib/trpc/routers';
	import { attitudes } from '$lib/editor/types';
	import type { Attitude, NodeWithMeta } from '$lib/editor/types';
	import RenderNode from '$lib/editor/render/Node.svelte';

	let { recentReadArticles } = localStorage;
	type Article = RouterOutput['article']['get'];
	let selectedArticle: Article | null = null;
	let selectedNodes: NodeWithMeta[] = [];
	let attitude: Attitude = '中立';
</script>

<Dialog
	class="bondModal"
	open={$bondModalState.isOpen}
	on:close={() => {
		$bondModalState.isOpen = false;
		selectedArticle = null;
	}}
>
	<DialogOverlay class="modalOverlay" />

	<DialogTitle>引用</DialogTitle>

	<!-- <div>
		<input placeholder="文章標題/代碼" bind:value={$bondArticleId} />
		<button>查詢</button>
	</div> -->
	<div>
		{#if selectedArticle}
			<div>
				{selectedArticle.title}
				<button
					on:click={() => {
						selectedArticle = null;
					}}>選擇其他文章</button
				>
				<div class="nodes">
					{#each selectedArticle.nodes as node}
						<label>
							<div class="node">
								<input type="checkbox" value={node} bind:group={selectedNodes} />
								<RenderNode node={node.value} />
							</div>
						</label>
					{/each}
				</div>
				<div>
					態度
					<select bind:value={attitude}>
						{#each attitudes as attitude}
							<option value={attitude}>
								{attitude}
							</option>
						{/each}
					</select>
				</div>
			</div>
		{:else}
			<div>
				最近閱讀
				<ul>
					{#each $recentReadArticles as articleMeta}
						<li>
							<button
								on:click={() => {
									trpc()
										.article.get.query({ id: articleMeta.id })
										.then((article) => {
											selectedArticle = article;
										});
								}}>{articleMeta.title}</button
							>
						</li>
					{/each}
				</ul>
			</div>
		{/if}
	</div>

	<button
		on:click={() => {
			if ($bondModalState.setBond) {
				if (selectedArticle == null) {
					console.error('未選擇文章');
					return;
				}
				selectedNodes.sort((p1, p2) => {
					return p1.order - p2.order;
				});
				const article = (({ id, title }) => ({ id, title }))(selectedArticle);
				$bondModalState.setBond({
					article,
					quotedNodes: selectedNodes,
					attitude
				});
			} else {
				console.error('編輯器未提供 setBond 函式');
			}
			$bondModalState.isOpen = false;
			selectedArticle = null;
			selectedNodes = [];
		}}>確定</button
	>
	<button on:click={() => ($bondModalState.isOpen = false)}>取消</button>
</Dialog>

<style>
	:global(.bondModal) {
		position: fixed;
		width: 100%;
		max-height: var(--doc-height);
		z-index: 200;
		background-color: white;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
		border: 1px solid black;
		@media (min-width: 600px) {
			width: 600px;
		}

		& :global(.modalOverlay) {
			position: fixed;
			top: 0;
			left: 0;
			background-color: rgb(0 0 0);
			opacity: 0.3;
		}
		& .nodes {
			margin: 10px;
			max-height: 400px;
			overflow-y: scroll;
			/* NOTE: 火狐到 110 版都還沒支援 has 關鍵字
			https://stackoverflow.com/questions/73936048/how-do-you-enable-has-selector-on-firefox */
			& div:has(input:checked) {
				background-color: antiquewhite;
			}
			& .node {
				display: flex;
				& p {
					margin: 4px;
				}
			}
		}
	}
</style>
