<script lang="ts">
    let i:string=$state("");
    let ii:string=$state("");
    let iii:string=$state("");
    let visible:boolean=$state(false);

    export function makeVisible(vis:boolean){
        visible=vis;
    }

    export function set(ni:string, nii:string, niii:string){
        i=ni;
        ii=nii;
        iii=niii;
    }
</script>

<div style={visible?`display:block`:`display:none`}>
    <h3>I. {i}</h3>
    <h3>II. {ii}</h3>
    <h3>III. {iii}</h3>
</div>
