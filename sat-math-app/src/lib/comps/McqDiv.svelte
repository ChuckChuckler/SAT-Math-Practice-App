<script lang="ts">
    import { onMount } from "svelte";

    let a:string=$state("");
    let b:string=$state("");
    let c:string=$state("");
    let d:string=$state("");
    let correct:string=$state("");
    let selected:string=$state("");

    let optionA:any=$state();
    let optionB:any=$state();
    let optionC:any=$state();
    let optionD:any=$state();

    let visible:boolean=$state(false);

    onMount(()=>{
        optionA.onclick=function(){
            optionA.style="background-color: #B1DBFA";
            optionB.style="background-color: #5B9CCF";
            optionC.style="background-color: #5B9CCF";
            optionD.style="background-color: #5B9CCF";
            selected=a;
        };

        optionB.onclick=function(){
            optionA.style="background-color: #5B9CCF";
            optionB.style="background-color: #B1DBFA";
            optionC.style="background-color: #5B9CCF";
            optionD.style="background-color: #5B9CCF";
            selected=b;
        };

        optionC.onclick=function(){
            optionA.style="background-color: #5B9CCF";
            optionB.style="background-color: #5B9CCF";
            optionC.style="background-color: #B1DBFA";
            optionD.style="background-color: #5B9CCF";
            selected=c;
        };

        optionD.onclick=function(){
            optionA.style="background-color: #5B9CCF";
            optionB.style="background-color: #5B9CCF";
            optionC.style="background-color: #5B9CCF";
            optionD.style="background-color: #B1DBFA";
            selected=d;
        };
    });

    export function updateOptions(newA:string, newB:string, newC:string, newD:string, newCorrect:string){
        a=newA;
        b=newB;
        c=newC;
        d=newD;
        correct=newCorrect;
    }

    export function makeVisible(newVisible:boolean){
        visible=newVisible;
        optionA.style="background-color: #5B9CCF";
        optionB.style="background-color: #5B9CCF";
        optionC.style="background-color: #5B9CCF";
        optionD.style="background-color: #5B9CCF";
    }

    export function getVisible():boolean{
        return visible;
    }

    export function checkAnswer(){
        return selected==correct;
    }
</script>

<div class="bg-blue-200 box-border p-[15px] w-[50vw]" style={visible?`display:block`:`display:none`}>
    <div class="grid grid-rows-4 h-[300px] gap-3">
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]" bind:this={optionA}>
            <div class="w-[18%] bg-white">A</div>
            <div class="w-[79%] bg-white">{a}</div>
        </div>
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]" bind:this={optionB}>
            <div class="w-[18%] bg-white">B</div>
            <div class="w-[79%] bg-white">{b}</div>
        </div>
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]" bind:this={optionC}>
            <div class="w-[18%] bg-white">C</div>
            <div class="w-[79%] bg-white">{c}</div>
        </div>
        <div class="bg-blue-400 rounded-[15px] flex justify-around box-border p-[5px]" bind:this={optionD}>
            <div class="w-[18%] bg-white">D</div>
            <div class="w-[79%] bg-white">{d}</div>
        </div>
    </div>
</div>