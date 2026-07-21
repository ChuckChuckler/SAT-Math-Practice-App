<script lang="ts">
    let n:string=$state("");
    let d:string=$state("");
    let vis:boolean=$state(false);

    export function updateFrac(newN:string, newD:string){
        n=newN;
        d=newD;
    }

    export function makeVisible(visible:boolean){
        vis=visible;
    }
</script>

<div class="text-center" style={vis?`display:inline-block`:`display:none`}>
    <h1>{n}</h1>
    <hr>
    <h1>{d}</h1>
</div>