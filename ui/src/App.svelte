<script>
	export let baseURI;

	import { onMount } from 'svelte';
	import FilePreviewer from "./FilePreviewer.svelte"
	import EmptyState from "./EmptyState.svelte"

	let filePreviewer

	let targets = []
	let dispPage = 1
	let dispTargetID

	// Functions
	const refreshTargetList = async () => {
		const resp = await axios.get(`${baseURI}/api/targets`, {
			headers: { "accept": "application/json" }
		})

		targets = resp.data || []
	}

	const openPage = (pageNumber) => {
		dispPage = pageNumber
		dispTargetID = targets[dispPage - 1].id

		refreshPage()
	}

	const markCurrent = async (value) => {
		await axios.post(`${baseURI}/api/targets/${dispTargetID}/mark`, { value }, {
			headers: { "accept": "application/json" }
		})
		
		await refreshPage()
	}
	
	const runFunc = async (funcName) => {
		await axios.post(`${baseURI}/api/funcs/${funcName}`, {
			headers: { "accept": "application/json" }
		})
		
		openPage(1)
	}

	const refreshPage = async () => {
		console.log('Refreshing...');
		
		await refreshTargetList()
		dispPage = dispPage
		dispTargetID = targets[dispPage - 1].id

		filePreviewer.refresh()
	}

	onMount(async () => {
		await refreshTargetList()

		if (targets.length > 0){
			dispPage = 1
			dispTargetID = targets[dispPage - 1].id
		}

		// Mousetrap Hotkey Setup
		Mousetrap.bind('right', function() { dispPage < targets.length ? openPage(dispPage + 1) : null; }, 'keydown');
		Mousetrap.bind('left', function() { dispPage > 1 ? openPage(dispPage - 1) : null; }, 'keydown');
		Mousetrap.bind('r', function() { markCurrent('remove'); }, 'keyup');

		refreshPage()

		var rc = setInterval(refreshPage, 10000);
	})
</script>

<div class="container grid-lg">
	<div style="margin-top: 40px;">
		<h1>FileTinder <small class="label">UI</small></h1>
		<p class="text-gray" style="margin-top: -15px;"><em>Tinderify your files. Keep what you liked, delete what you hate (with caution)</em></p>
	</div>

	{#if targets.length <= 0}
	<!-- Empty State -->
	<div class="columns">
		<EmptyState/>
	</div>
	{:else}
	<!-- Non Empty State -->
	<div class="columns">
		<div class="column col-9">
			<!--  Previewer -->
			<FilePreviewer targetID={dispTargetID} bind:this={filePreviewer}/>
			<!-- {()=>openPage()} -->
			<!-- Pagination -->
			<div style="margin-top: 20px; text-align: center;">
				<div class="d-inline-block">
					<ul class="pagination">
						<li on:click={()=>{if(dispPage <= 1)return;openPage(dispPage - 1)}} class="page-item {dispPage <= 1 ? "disabled" : ""}">
							<a href="#{dispPage - 1}" tabindex="-1">Previous</a>
						</li>
						{#if dispPage > 2}
						<li on:click={()=>openPage(1)} class="page-item">
							<a href="#">1</a>
						</li>
						{/if}
						{#if dispPage > 2}
						<li class="page-item">
							<span>...</span>
						</li>
						{/if}
						{#if dispPage > 1}
						<li on:click={()=>openPage(dispPage - 1)} class="page-item">
							<a href="#">{dispPage - 1}</a>
						</li>
						{/if}
						<li class="page-item active">
							<a href="#">{dispPage}</a>
						</li>
						{#if targets.length - dispPage >= 1}
						<li on:click={()=>openPage(dispPage + 1)} class="page-item">
							<a href="#">{dispPage + 1}</a>
						</li>
						{/if}
						{#if targets.length - dispPage > 1}
						<li class="page-item">
							<span>...</span>
						</li>
						{/if}
						{#if targets.length - dispPage > 1}
						<li on:click={()=>openPage(targets.length)} class="page-item">
							<a href="#">{targets.length}</a>
						</li>
						{/if}
						<li on:click={()=>{if(dispPage == targets.length)return; openPage(dispPage + 1)}} class="page-item {dispPage == targets.length ? "disabled" : ""}">
							<a href="#">Next</a>
						</li>
					</ul>
				</div>
			</div>
		</div>
		
		<!--  Menu -->
		<div class="column col-3">
			<ul class="menu">
				<li class="divider" data-content="TAG FILE"></li>
				
				<!-- Menu Entries -->
				<li on:click={()=>markCurrent("remove")} class="menu-item tooltip tooltip-right" data-tooltip="shortcut: R">
					<a href="#">
						<i class="icon icon-bookmark"></i> Remove
					</a>
				</li>
			</ul>

			<br>

			<ul class="menu">
				<li class="divider" data-content="RUN FUNCTIONS"></li>

				<!-- Menu Entries -->
				<li on:click={()=>runFunc("delete-all")} class="menu-item tooltip tooltip-bottom" data-tooltip="Delete all files you have marked as 'remove'">
					<a href="#">
						<i class="icon icon-link"></i> Delete All Marked
					</a>
					
				</li>
			</ul>
		</div>
	</div>
	{/if}
</div>
