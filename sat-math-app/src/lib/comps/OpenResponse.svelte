<script lang="ts">
    let answerInterfaceVisible:boolean=$state(false);
    let userAnswer:string=$state("");
    let pos:boolean=$state(false);

    function checkIfNegative(){
        (userAnswer.includes("-"))?pos=false:pos=true;
    }

    export function checkAnswer(solutions:number[]):boolean{
        let x:number=0;
        userAnswer.includes("/")?x=(Math.round(parseFloat(userAnswer.split("/")[0])/parseFloat(userAnswer.split("/")[1])*1000))/1000:x=parseFloat(userAnswer);
        for(let i of solutions){
            if(x==i){
                return true;
            }
        }
        return false;
    }

    export function makeVisible(visible:boolean){
        answerInterfaceVisible=visible;
    }

    export function reset(){
        userAnswer="";
    }
</script>

<div id="answerInputContainer" style={answerInterfaceVisible?`display:block`:`display:none`}>
    <label for="answer">Answer:</label>
    <input type="text" autocomplete="off" name="answer" id="answer" maxlength={pos?5:6} bind:value={userAnswer} oninput={checkIfNegative}>
    <br>
    <p>Enter your answer as an integer, an improper fraction, or a rounded decimal. See <a href="https://satsuite.collegeboard.org/media/pdf/english-sat-test-directions-bb.pdf" target="_blank" aria-label="SAT decimal rounding guide">SAT decimal rounding guide (p.7)</a>.</p>
    <br>
</div>

