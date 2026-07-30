<script lang="ts">
    let {type, args}=$props();

    //inner text
    let n:string=$state("");
    let d:string=$state("");
    let num:number=$state(0);
    let x:string=$state("");

    //visibilities
    let fracVis:boolean=$state(false);
    let numVis:boolean=$state(false);
    let stringVis:boolean=$state(false);

    $effect(()=>{
        if(type==1){
            createFraction(args);
        }else if(type==2){
            createNonfraction(args);
        }else if(type==3){
            createString(args);
        }
    });

    function createFraction(args:any[]){
        fracVis=true;
        numVis=false;
        stringVis=false;
        n=args[0];
        d=args[1];
    }

    function createNonfraction(args:any[]){
        fracVis=false;
        numVis=true;
        stringVis=false;
        num=args[0];
    }

    function createString(args:any[]){ //can be variable or symbol
        fracVis=false;
        numVis=false;
        stringVis=true;
        x=args[0];
    }
</script>

<div id="fraction" class="mr-[-5px]" style={fracVis?`display:inline-block`:`display:none`}>
    <h1 class="text-center">{n}</h1>
    <hr>
    <h1>{d}</h1>
</div>

<div id="nonFraction" class="relative" style={numVis?`display:inline-block`:`display:none`}>
    <h1 class="absolute top-[50%] left-[50%] translate-x-[-50%] translate-y-[-50%]">{num}</h1>
</div>

<div id="variable" class="relative" style="{stringVis?`display:inline-block`:`display:none`}">
    <h1 class="absolute top-[50%] left-[50%] translate-x-[-50%] translate-y-[-50%]">{x}</h1>
</div>