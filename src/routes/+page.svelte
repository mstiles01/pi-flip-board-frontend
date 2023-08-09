<script lang="ts">
    import { onMount } from "svelte";
    let firstRow = '';
    let secondRow = '';
    let thirdRow = '';
    let alignment = "left";  // default alignment

    interface Cell {
        id: number;
        value: string;
    }

    let cellData = [
        "", "A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z",
        "1", "2", "3", "4", "5", "6", "7", "8", "9", "0",
        "#FF0000", "#0000FF", "#FFFF00", "#008000", "#800080", "#FFA500"
    ];

    let cells: Cell[][] = [];

    onMount(() => {
        let newCells: Cell[][] = [];
        for(let i = 0; i < 5; i++) {
            let row: Cell[] = [];
            for(let j = 0; j < 21; j++) {
                row.push({id: i*21+j, value: cellData[0]});
            }
            newCells.push(row);
        }
        cells = newCells;  // this will trigger Svelte's reactivity
    });

    function handleClick(cell: Cell) {
        let index = cellData.indexOf(cell.value);
        cell.value = cellData[(index + 1) % cellData.length];
        cells = [...cells];
    }

    const sleep = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

    const animateCell = async (cell: Cell, targetValue: string, delay: number) => {
        targetValue = targetValue.toUpperCase();
        const targetIndex = cellData.indexOf(targetValue);
        if (targetIndex === -1) {
            console.log(`Invalid target value: ${targetValue}`);
            return;
        }
        for (let i = 0; i < 3; i++) {
            for (let j = 0; j < cellData.length; j++) {
                cell.value = cellData[j];
                cells = [...cells];
                await sleep(50);
                if (i === 2 && j === targetIndex) {
                    await sleep(delay);
                }
            }
        }
        cell.value = cellData[targetIndex];
        cells = [...cells];
    }


    const submit = () => {
        let rowIndex = 1;
        let promises = [];

        for (let rowInput of [firstRow, secondRow, thirdRow]) {
            let start = 9;

            // Alignment calculations
            if (alignment === "left") {
                start = 0;
            } else if (alignment === "center") {
                start = Math.max(0, Math.floor((21 - rowInput.length) / 2));
            } // For right alignment, start will be (21 - rowInput.length).

            for (let i = 0; i < rowInput.length; i++) {
                const delay = i * 200;  // adjust the multiplier as needed
                promises.push(animateCell(cells[rowIndex][start + i], rowInput.charAt(i), delay));
            }
            rowIndex++;
        }
        Promise.all(promises);
    }

    async function save() {
        let flatBoard = flattenBoard(cells);

        try {
            const response = await fetch("http://localhost:8081/save", {
                method: "POST",
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ message: flatBoard })
            });

            if (!response.ok) {
                console.error("Failed to send board:", response.statusText);
                return;
            }

            console.log("Board sent successfully!");
        } catch (error) {
            console.error("Error sending board:", error);
        }
    }

    function flattenBoard(board) {
        let flatBoard = [];
        for (let row of board) {
            for (let cell of row) {
                flatBoard.push(cell.value);
            }
        }
        return flatBoard;
    }


</script>

<style>
    .grid {
        display: grid;
        grid-template-columns: repeat(21, 1fr);
        gap: 5px;
    }
    .cell {
        width: 40px;
        height: 40px;
        border: 1px solid black;
        text-align: center;
        line-height: 40px;
        cursor: pointer;
        color: white;
    }

</style>

<div class="grid">
    {#each cells as row, i}
        {#each row as cell, j}
            <div class="cell" style="background-color: {cell.value.startsWith('#') ? cell.value : 'black'}" on:click={() => handleClick(cell)}>
                {cell.value.startsWith('#') ? '' : cell.value}
            </div>
        {/each}
    {/each}
</div>

<div class="flex flex-col max-w-[300px] mt-4">
    <label class="mb-2 flex justify-center">Alignment</label>
    <select bind:value={alignment}>
        <option value="left">Left</option>
        <option value="center">Center</option>
        <option value="right">Right</option>
    </select>
</div>


<div class="flex justify-center mt-14 flex-col w-full items-center">
    <div class="flex flex-col max-w-[300px]">
        <label class="mb-2 flex justify-center">First Row</label>
        <input type="text" bind:value={firstRow} pattern="[A-Za-z0-9 ]*" on:input="{e => e.target.value = e.target.value.toUpperCase().replace(/[^A-Z0-9 ]/g, '')}" />
    </div>

    <div class="flex flex-col max-w-[300px] mt-4">
        <label class="mb-2 flex justify-center">Second Row</label>
        <input type="text" bind:value={secondRow} pattern="[A-Za-z0-9 ]*" on:input="{e => e.target.value = e.target.value.toUpperCase().replace(/[^A-Z0-9 ]/g, '')}" />
    </div>

    <div class="flex flex-col max-w-[300px] mt-4">
        <label class="mb-2 flex justify-center">Third Row</label>
        <input type="text" bind:value={thirdRow} pattern="[A-Za-z0-9 ]*" on:input="{e => e.target.value = e.target.value.toUpperCase().replace(/[^A-Z0-9 ]/g, '')}" />
    </div>

    <button class="bg-[#1da1f2] border-2 border-black mt-4 " on:click={submit}>Submit</button>
    <button on:click={save}>Save</button>

</div>

