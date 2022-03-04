<script lang="ts">
	import ListItem from "./ListItem.svelte"
	import Comparison from "./Comparison.svelte"
    import { fly, scale } from "svelte/transition"
	import { flip } from "svelte/animate"

	type Item = {
		name: string
		uid: string
	} 

	let mainArray: Item[] = []

	let addItemTextValue: string = ""
	let canAddItem: boolean = true

	const addItem = (e: Event) => {
		e.preventDefault()

		if(addItemTextValue === "" || !canAddItem) return

		setTimeout(() => canAddItem = true, 500)
		canAddItem = false
			
		let uid: string = generateUid()
		mainArray = [{name: addItemTextValue, uid: uid}, ...mainArray]

		addItemTextValue = ""
	}

	const generateUid = (): string => {
		return Math.random().toString(36).substr(2, 3)
	}

	const removeItem = (e): void => {
		mainArray = mainArray.filter(item => item.uid !== e.detail)
	}

	let placeholderText: string = ""
	let finishedSorting: boolean = false
	$: getPlaceholderText = () => {
		if (finishedSorting) return placeholderText
		else if (mainArray.length === 0) placeholderText = "Add an item"
		else if (mainArray.length === 1) placeholderText = "Add another item"
		else placeholderText = mainArray.length + " items added, estimated sorting time is " + getEstimatedTime()
		return placeholderText
	}

	const getEstimatedTime = (array: any[] = mainArray, secondsPerOperation: number = 5): string => {
		let sortingTime: number = array.length * Math.log10(array.length) * secondsPerOperation
		return getTimeAndUnitFromSeconds(sortingTime)
	}

	const getTimeAndUnitFromSeconds = (timeInSeconds: number): string => {
		let timeUnit = " seconds"
		let time = Math.round(timeInSeconds)
		if (timeInSeconds > 120) {
			time = Math.round(timeInSeconds / 60)
			timeUnit = " minutes"
		}
		return time + timeUnit
	}

	let comparisonLeft
	let comparisonRight
	let comparisonVisible = false
	let promise: (choseleft: boolean) => void

	const mergeSort = async (array = mainArray, half = array.length/2) => {

		if (array.length < 2) {
			return array
		}

		const left = array.splice(0, half)

		return merger(await mergeSort(left), await mergeSort(array))
	}

	const merger = async (left: any[], right: any[]) => {

		const arr = []

		while (left.length && right.length) {
			comparisonLeft = left[0].name
			comparisonRight = right[0].name

			const choice: boolean = await new Promise((res, rej) => {
				comparisonVisible = true
				promise = res
			})

			if (choice) {
				arr.push(left.shift())
			}
			else {
				arr.push(right.shift())
			}
		}

		return [...arr,...left,...right]
	}

	const choseLeft = (e) => {
		promise(e.detail)
	}

	const sort = async () => {
		let estimatedTime: string = getEstimatedTime()
		let startTime = new Date().getTime()

		mainArray = await mergeSort()
		comparisonVisible = false

		let endTime = new Date().getTime()
		let totalTime = getTimeAndUnitFromSeconds( (endTime - startTime)/1000 )

		finishedSorting = true
		placeholderText = "Sorting time was " + totalTime + ", estimated " + estimatedTime
	}
	
	;['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
    	document.addEventListener(eventName, preventDefaults, false)
	})


	function preventDefaults(e){e.preventDefault();e.stopPropagation();}
	document.addEventListener('drop', handleDrop, false)
	function handleDrop(e) {
		let files = e.dataTransfer.files

		let file = files[0]
		if(file.type == "text/plain"){
			if(file.size < 100000){
				let reader = new FileReader()

				reader.onload = async function () {
					let tempArray: string[] = (<string>reader.result).split("\n")
					tempArray.forEach(name => {
						mainArray = [{name: name, uid: generateUid()}, ...mainArray]
					});
				}

				reader.readAsText(file)
			}
			else{
				alert("Din fil är på tok för stor")
			}
		}
		else{
			alert("Endast .txt tillåts")
		}
	}

</script>

<main>
	{#if comparisonVisible}
		<Comparison optionLeft={comparisonLeft} optionRight={comparisonRight} on:choseleft={choseLeft} />
	{/if}
	<ul>
		<li>
			<form id="addItemForm" autocomplete="off" on:submit={addItem}>
				<input type="text" name="itemName" id="addItemText" placeholder={getPlaceholderText()} bind:value={addItemTextValue} />
			</form>
		</li>
		{#each mainArray as item (item.uid)}
			<div in:fly={{ x: -1000, duration: 500 }} out:scale={{ duration: 200, opacity: 1 }} animate:flip>
				<ListItem name={item.name} uid={item.uid} on:removeitem={removeItem} />
			</div>
		{/each}
	</ul>
	<div id="sort" on:click={sort}>Sort!</div>
</main>

<style lang="scss">
	main {
		display: grid;
		grid-template-columns: [pageleft] 4rem [listleft] 50rem [listright] 4rem [sortleft] 8rem [sortright];
		grid-template-rows: [pagetop] 4rem [listtop] var(--listItemHeight) [listbottom];
		width: 100%;
		height: 100%;
		color: var(--textcolor);
	}

	#addItemText {
		position: relative;
		color: inherit;
		font-family: inherit;
		font-size: inherit;
		font-weight: inherit;

		background: none;
		border: none;
		outline-style: none;

		width: 100%;

		&::placeholder {
			color: rgba(white, 0.6);
		}
	}

	ul {
		margin: 0;
		padding: 0;
		grid-column: listleft / listright;
		grid-row-start: listtop;
		list-style: decimal;
	}

	#sort {
		display: grid;
		place-items: center;

		grid-column: sortleft / sortright;
		grid-row: listtop / listbottom;
		box-sizing: border-box;
		width: 100%;
		height: 100%;
		border: 0.5rem solid var(--textcolor);
		border-radius: 1rem;
		text-align: center;

		font-weight: 700;
		font-size: 2.5rem;

		transition: 500ms;

		user-select: none;

		&:hover {
			background-color: var(--textcolor);
			color: #111;
			transition: 200ms;
		}
	}
</style>