<script lang="ts">
    import Number from "./Number.svelte";

    //let numbers:any[][]=[[1,[1,2]],[3,["x"]],[3,["+"]],[2,[5]],[3,["="]],[1,[7,8]]];
    let numbers:any[][]=$state([]);
    let amountOfRefresh:number=$state(0);

    export function updateEquation(newNumbers:any[][]){
        numbers=newNumbers;
    }

    export function getNumbers():any[][]{
        return numbers;
    }
</script>

<div class="flex gap-2">
    {#each numbers as i}
        <Number type={i[0]} args={i[1]}></Number>
    {/each}
</div>
