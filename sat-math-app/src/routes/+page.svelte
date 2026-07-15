<script lang="ts">
    import Fraction from "$lib/comps/Fraction.svelte";
    import McqDiv from "$lib/comps/McqDiv.svelte";
    import OpenResponse from "$lib/comps/OpenResponse.svelte";

    let mcqdiv:any=$state();
    let openResponse:any=$state();

    let checkAnswerVisible:boolean=$state(true);
    let makeQuestionVisible:boolean=$state(true);

    let problem:string=$state("");
    let solutions:number[]=[];
    let pos:boolean=$state(true)
    let userAnswer:string=$state("");

    let equation1:string=$state("");
    let equation2:string=$state("");

    let feedback:string=$state("");

    function randint(min:number, max:number):number{
        let x:number=Math.floor(Math.random()*(max-min+1)+min);
        while(x==0){
            x=Math.floor(Math.random()*(max-min+1)+min);
        }
        return x;
    }

    function makeQuestion(){
        makeQuestionVisible=false;
        checkAnswerVisible=true;
        feedback="";
        userAnswer="";
        equation1="";
        equation2="";

        //let choice:number=randint(1,8);
        let choice:number=8;
        if(choice==1){
            typeA();
            openResponse.makeVisible(true);
            mcqdiv.makeVisible(false);
        }else if(choice==2){
            typeB();
            openResponse.makeVisible(true);
            mcqdiv.makeVisible(false);
        }else if(choice==3){
            typeC();
            openResponse.makeVisible(true);
            mcqdiv.makeVisible(false);
        }else if(choice==4){
            typeD();
            openResponse.makeVisible(true);
            mcqdiv.makeVisible(false);
        }else if(choice==5){
            typeE();
            openResponse.makeVisible(true);
            mcqdiv.makeVisible(false);
        }else if(choice==6){
            typeF();
            openResponse.makeVisible(true);
            mcqdiv.makeVisible(false);
        }else if(choice==7){
            typeG();
            openResponse.makeVisible(true);
            mcqdiv.makeVisible(false);
        }else if(choice==8){
            typeH();
            openResponse.makeVisible(false);
            mcqdiv.makeVisible(true);
        }
    }

    function submitAnswer(){
        if(mcqdiv.getVisible()){
            if(mcqdiv.checkAnswer()){
                feedback="Correct!";
                solutions=[];
                makeQuestionVisible=true;
                checkAnswerVisible=false;
            }else{
                feedback="Incorrect...Try again!!!"
            }
        }
    }

    function typeA(){
        problem="The ";

        var exponent1:string;
        var exponent2:string;

        let a:number=randint(1,5);
        let b:number=randint(-10,10);

        let c:number=randint(1,5);
        let d:number=randint(-10,10);

        let k:number=randint(1,5);

        var quadOrQuart=randint(1,2);

        if(quadOrQuart==1){ //quadratic
            problem+="quadratic ";
            exponent1="²";
            exponent2="";
        }else{ //quartic
            problem+="quartic ";
            exponent1="⁴";
            exponent2="²";
        }

        if(k==1){
            if(a*c<0){
                problem+=`${a*c*-1}x${exponent1} `;
                ((a*d + c*b)*-1 < 0)?problem+=`- ${(a*d + c*b)}x${exponent2}`:problem+=`+ ${(a*d + c*b)*-1}x${exponent2} `;
                (b*d*-1 < 0)?problem+=`- ${b*d}`:problem+=`+ ${b*d*-1}`
            }else{
                problem+=`${a*c}x${exponent1} `;
                ((a*d + c*b) < 0)?problem+=`- ${(a*d + c*b)*-1}x${exponent2}`:problem+=`+ ${(a*d + c*b)}x${exponent2} `;
                (b*d < 0)?problem+=`- ${b*d*-1}`:problem+=`+ ${b*d}`;
            }
            problem+=` can be factored as (ax${exponent2} + b)(cx${exponent2} + d), where a, b, c, and d are integers. `;
        }else{
            if(a*c<0){
                problem+=`${a*c*-1*k}x${exponent1} `;
                ((a*d + c*b)*-1 < 0)?problem+=`- ${(a*d + c*b)*k}x${exponent2} `:problem+=`+ ${(a*d + c*b)*k*-1}x${exponent2} `;
                (b*d*-1 < 0)?problem+=`- ${b*d*k}`:problem+=`+ ${b*d*k*-1}`;
            }else{
                problem+=`${a*c}x${exponent1} `;
                ((a*d + c*b) < 0)?problem+=`- ${(a*d + c*b)*-1}x${exponent2} `:problem+=`+ ${(a*d + c*b)}x${exponent2} `;
                (b*d < 0)?problem+=`- ${b*d*-1}`:problem+=`+ ${b*d}`
            }
            problem+=` can be factored as (k)(ax${exponent2} + b)(cx${exponent2} + d), where a, b, c, d, and k are integers. `;
        }

        var smallest:boolean=false;
        let smallOrLarge:number=randint(1,2);
        if(smallOrLarge==1){
            problem+=`What is the smallest possible value of `;
            smallest=true;
        }else{
            problem+=`What is the largest possible value of `;
            smallest=false;
        }

        /*
        possibilities:
        ab or cd
        a+b, a-b, a*b, or a/b
        c+d, c-d, c*d, or c/d
        */

        let abOrCd:number=randint(1,2);

        console.log(a, b, c, d);

        if(abOrCd==1){
            let operation:number=randint(1,4);
            if(operation==1){
                problem+=`a + b?`;
                if(smallest){
                    if(a+b<c+d){
                        solutions.push(a+b);
                    }else{
                        solutions.push(c+d);
                    }
                }else{
                    if(a+b>c+d){
                        solutions.push(a+b);
                    }else{
                        solutions.push(c+d);
                    }
                }
            }else if(operation==2){
                problem+=`a - b?`;
                if(smallest){
                    if(a-b<c-d){
                        solutions.push(a-b);
                    }else{
                        solutions.push(c-d);
                    }
                }else{
                    if(a-b>c-d){
                        solutions.push(a-b);
                    }else{
                        solutions.push(c-d);
                    }
                }
            }else if(operation==3){
                problem+=`ab?`;
                if(smallest){
                    if(a*b<c*d){
                        solutions.push(a*b);
                    }else{
                        solutions.push(c*d);
                    }
                }else{
                    if(a*b>c*d){
                        solutions.push(a*b);
                    }else{
                        solutions.push(c*d);
                    }
                }
            }else if(operation==4){
                problem+=`a/b?`;
                if(smallest){
                    if(a/b<c/d){
                        solutions.push(a/b);
                    }else{
                        solutions.push(c/d);
                    }
                }else{
                    if(a/b>c/d){
                        solutions.push(a/b);
                    }else{
                        solutions.push(c/d);
                    }
                }
            }
        }else{
            let operation:number=randint(1,4);
            if(operation==1){
                problem+=`c + d?`;
                if(smallest){
                    if(a+b<c+d){
                        solutions.push(a+b);
                    }else{
                        solutions.push(c+d);
                    }
                }else{
                    if(a+b>c+d){
                        solutions.push(a+b);
                    }else{
                        solutions.push(c+d);
                    }
                }
            }else if(operation==2){
                problem+=`c - d?`;
                if(smallest){
                    if(a-b<c-d){
                        solutions.push(a-b);
                    }else{
                        solutions.push(c-d);
                    }
                }else{
                    if(a-b>c-d){
                        solutions.push(a-b);
                    }else{
                        solutions.push(c-d);
                    }
                }
            }else if(operation==3){
                problem+=`cd?`;
                if(smallest){
                    if(a*b<c*d){
                        solutions.push(a*b);
                    }else{
                        solutions.push(c*d);
                    }
                }else{
                    if(a*b>c*d){
                        solutions.push(a*b);
                    }else{
                        solutions.push(c*d);
                    }
                }
            }else if(operation==4){
                problem+=`c/d?`;
                if(smallest){
                    if(a/b<c/d){
                        solutions.push(a/b);
                    }else{
                        solutions.push(c/d);
                    }
                }else{
                    if(a/b>c/d){
                        solutions.push(a/b);
                    }else{
                        solutions.push(c/d);
                    }
                }
            }
        }

        solutions[0]=Math.round(solutions[0]*1000)/1000;

        console.log(solutions[0]);
    }

    function typeB(){
        /*
            - find factors and coefficients. name them?
            - (r1x + a)(r2x + b)
            - a and b are the integers
            - solve out a system of equations --> get d, plug back in for c
        */

        /*
        Requirements for c and d to be nonintegers:
        assuming ax^2 + bx + c
        - b%r1 must = 0 and b%r2 must = 0!!!
        - so we will do a while loop when deciding r1, r2, a, b; make sure r1!=r2 and that (r1b+r2a)%r1 or r2 does not equal 0
        */

        let r1:number=randint(2,5);
        let r2:number=randint(2,5);
        let a:number=randint(-10,10);
        let b:number=randint(-10,10);
        while(r1==r2 || (r1*b+r2*a)%r1==0 || (r1*b+r2*a)%r2==0){
            r1=randint(2,5);
            r2=randint(2,5);
            a=randint(-10,10);
            b=randint(-10,10);
        }

        let c:number=0;
        let d:number=0;

        let ya:number=((r1*b)+(r2*a) + Math.sqrt(Math.pow((r1*b)+(r2*a),2)-(4*r1*(a*b*r2))))/(2*r1);
        let yb:number=((r1*b)+(r2*a) - Math.sqrt(Math.pow((r1*b)+(r2*a),2)-(4*r1*(a*b*r2))))/(2*r1);

        if(ya.toString().includes(".")){ //find the solution that is noninteger; that is d
            d=ya;
            c=(a*b)/ya;
        }else if(yb.toString().includes(".")){
            d=yb;
            c=(a*b)/yb;
        }

        problem=`The expression ${r1*r2}x² `;
        ((r1*b)+(r2*a)<0)?problem+=`- ${-((r1*b)+(r2*a))}x `:problem+=`+ ${(r1*b)+(r2*a)}x `;
        (a*b<0)?problem+=`- ${a*b*-1}`:problem+=`+ ${a*b}`;
       
        problem+=` can be rewritten as (${r1}x + a)(${r2}x + b), where a and b are integers, or as (${r1}x + c)(${r2}x + d), where c and d are nonintegers. What is the value of `;
       /*
       Things we can try to find:
       a+c
       a+d
       b+c
       b+d
       i fear the rest is a hugeee timesink
       */
        let whatToFind:number=randint(1,4);
        if(whatToFind==1){ //a+c
            problem+=`a + c?`;
            solutions.push(a+c);
        }else if(whatToFind==2){ //a+d
            problem+=`a + d?`;
            solutions.push(a+d);
        }else if(whatToFind==3){ //b+c
            problem+=`b + c?`;
            solutions.push(b+c);
        }else{
            problem+=`b + d?`;
            solutions.push(b+d);
        }

        solutions[0]=Math.round(solutions[0]*1000)/1000;
    }

    function typeC(){
        //the following (w,x,y,z) will be used to calculate a, b, and c in a way they can actually be factored into ints
        let w:number=randint(1,5);
        let x:number=randint(-10,10);
        let y:number=randint(1,5);
        let z:number=randint(-10,10);

        let a:number=w*y;
        let b:number=(w*z)+(x*y);
        let c:number=x*z;

        let m:number=randint(-5,5);
        let e:number=-((Math.pow(b-m,2)/(4*a))-c);
        while(e.toString().length>6){
            a=w*y;
            b=(w*z)+(x*y);
            c=x*z;
            m=randint(-5,5);

            e=-((Math.pow(b-m,2)/(4*a))-c);
        }

        //solution = (-(b-m))/(2*a);

        /*
        Possible unknowns (assuming y = ax^2 + bx + c; y = mx + e):
        - a
        - b
        - c
        - m
        - e
        */
        let whichUnknown:number=randint(1,5);
        let chosenUnknownName:string=``;
        if(whichUnknown==1){ //a
            chosenUnknownName=`a`;
            equation1=`y = ax² `;
            (b<0)?equation1+=`- ${-b}x `:equation1+=`+ ${b}x `;
            (c<0)?equation1+=`- ${-c}x `:equation1+=` + ${c}x`;
            equation2=`y = ${m}x `;
            (e<0)?equation2+=`- ${-e}`:equation2+=`+ ${e}`;
        }else if(whichUnknown==2){ //b
            chosenUnknownName=`b`;
            equation1=`y = ${a}x² + bx `;
            (c<0)?equation1+=`- ${-c}`:equation1+=`+ ${c};`
            equation2=`y = ${m}x `;
            equation2+=(e<0)?`- ${-e}`:`+ ${e}`;
        }else if(whichUnknown==3){ //c
            chosenUnknownName=`c`;
            equation1=`y = ${a}x² `;
            equation1+=(b<0)?`- ${-b}x + c`:`+ ${b}x + c`;
            equation2=`y = ${m}x `;
            equation2+=(e<0)?`- ${-e}`:`+ ${e}`;
        }else if(whichUnknown==4){ //d
            chosenUnknownName=`d`;
            equation1=`y = ${a}x² `;
            equation1+=(b<0)?`- ${-b}x `:`+ ${b}x `;
            equation1+=(c<0)?`- ${-c}`:`+ ${c}`;
            equation2=`y = mx `;
            equation2+=(e<0)?`- ${-e}`:`+ ${e}`;
        }else if(whichUnknown==5){ //e
            chosenUnknownName=`e`;
            equation1=`y = ${a}x² `;
            equation1+=(b<0)?`- ${-b}x `:`+ ${b}x `;
            equation1+=(c<0)?`- ${-c}`:`+ ${c}`;
            equation2=`y = ${m}x + e`;
        }

        let xOrY:number=randint(1,2);
        let chosenForSolution:string=``;

        if(xOrY==1){ //x
            solutions.push((Math.round(-(b-m))/(2*a)*1000)/1000);
            if(chosenUnknownName=="b"){
                solutions.push(-((Math.round(-(b-m))/(2*a)*1000)/1000));
            }
            chosenForSolution="x";
        }else{
            x = (-(b-m))/(2*a);
            solutions.push(Math.round((m*x + e)*1000)/1000);
            if(chosenUnknownName=="b"){
                x = -(-(b-m))/(2*a);
                solutions.push(Math.round((m*x + e)*1000)/1000);
            }
            chosenForSolution="y";
        }

        (chosenUnknownName=="b")?problem=`In the given system of equations, ${chosenUnknownName} is a constant. The graphs of the equations in the given system interact at exactly one point, (x, y), in the xy-plane. What is one possible value of ${chosenForSolution}?`:problem=`In the given system of equations, ${chosenUnknownName} is a constant. The graphs of the equations in the given system interact at exactly one point, (x, y), in the xy-plane. What is the value of ${chosenForSolution}?`;
    }

    function typeD(){
        console.log("if jx+k factor, what is ac")
        let alphabet:string[]=["a","b","c","d","e","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let b:number=randint(1,150)*2;
        equation1=`ax² + ${b}x + c`;
        let letter1=randint(0,alphabet.length-1);
        let letter2=randint(0,alphabet.length-1);
        while(letter1==letter2){
            letter2=randint(0,alphabet.length-1);
        }
        problem=`In the given expression, a and c are positive integer constants. If (${alphabet[letter1]}x + ${alphabet[letter2]}) is a factor of the expression, where j and k are positive constants, what is a possible value of ac?`
        solutions.push(Math.pow(b,2)/4);
    }

    function typeE(){
        //very similar to type C/choice 3
        let a:number=randint(-5,5);
        let b:number=randint(1,20);
        let c:number=Math.pow(b,2)/(4*a);
        if((a<0 && c<0) || (a>0 && c>0)){ //b will be undefined if -4ac ends up positive
            let x:number=randint(1,2);
            (x==1)?a*=-1:c*=-1;
        }
        /*options:
        - b is unknown (parabola)
        - c is unknown (horizontal line)
        */
        problem=`In the xy-plane, a line with equation `;
        let bOrC:number=randint(1,2);
        if(bOrC==1){ //b is unknown
            let coeffY=randint(1,3)*2;
            problem+=`${coeffY}y = ${Math.round(c*coeffY*10000)/10000} intersects a parabola at exactly one point. If the parabola has equation y = ${a}x² + bx, where b is a `;
            let posOrNeg:number=randint(1,2);
            if(posOrNeg==1){ //pos
                problem+=`positive constant, what is the value of b?`;
                solutions.push(b);
            }else{
                problem+=`negative constant, what is the value of b?`;
                solutions.push(-b);
            }
        }else{ //c is unknown
            let coeffY:number=randint(1,3)*2;
            problem+=`${coeffY}y = c for some constant c intersects a parabola at exactly one point. If the parabola has equation y = ${a}x² + ${b}x, what is the value of c?`;
            solutions.push(c*coeffY);
        }
    }

    function typeF(){
        //what is the value of g/k?
        let a:number=randint(-10,10);
        let b:number=randint(-10,10);
        while(a==b){
            b=randint(-10,10);
        }
        let c:number=randint(-30,30);
        let d:number=randint(-30,30);
        while(c==d){
            d=randint(-30,30);
        }

        let g:number=(d/c)*a;
        let k:number=(d/c)*b;

        let alphabet:string[]=["a","b","c","d","e","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let letter1=randint(0,alphabet.length-1);
        let letter2=randint(0,alphabet.length-1);
        while(letter1==letter2){
            letter2=randint(0,alphabet.length-1);
        } 

        equation1=`${a}x `;
        equation1+=(b<0)?`- ${-b}y = ${c}`:`+ ${b}y = ${c}`;
        equation2=`${alphabet[letter1]}x + ${alphabet[letter2]}y = ${d}`;
        problem=`In the given system of equations, ${alphabet[letter1]} and ${alphabet[letter2]} are constants. The system has infinitely many solutions. What is `;

        let solutionFormat = randint(1,4);
        if(solutionFormat==1){ //g+k
            problem+=`${alphabet[letter1]} + ${alphabet[letter2]}?`;
            solutions.push(Math.round((g+k)*1000)/1000);
        }else if(solutionFormat==2){ //g-k
            problem+=`${alphabet[letter1]} - ${alphabet[letter2]}?`;
            solutions.push(Math.round((g-k)*1000)/1000);
        }else if(solutionFormat==3){ //gk
            problem+=`${alphabet[letter1]}${alphabet[letter2]}?`;
            solutions.push(Math.round((g*k)*1000)/1000);
        }else{ //g/k
            problem+=`${alphabet[letter1]}/${alphabet[letter2]}?`;
            solutions.push(Math.round((g/k)*1000)/1000);
        }
    }

    function typeG(){
        //no solutions...what is r?
        /*
            x + y1 = y2 + c
            a + b = d
        */
        let x:number=randint(1,20);
        let y1:number=randint(-20,20);
        let y2:number=randint(-20,20);
        while(y1-y2==0){
            y2=randint(-20,20);
        }

        let c:number=randint(-30,30);
        let d:number=randint(-30,30);
        while(c==d){
            d=randint(-30,30);
        }

        let scale:number=randint(2,7);
        let a:number=0;
        let b:number=0;
        if(x%scale==0){
            a=x/scale;
            b=(y1-y2)/scale;
        }else{
            a=x*scale;
            b=(y1-y2)*scale;
        }

        let alphabet:string[]=["a","b","c","d","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];

        let equation1Format:number=randint(1,5); //5 different formats for equation 1, just because it's cool to have multiple formats
        if(equation1Format==1){
            equation1=`${x}x `;
            equation1+=(y1<0)?`- ${-y1}y = ${y2}y `:`+ ${y1}y = ${y2}y `;
            equation1+=(c<0)?`- ${-c}`:`+ ${c}`;
        }else if(equation1Format==2){
            equation1=`${x}x = ${y2-y1}y `;
            equation1+=(c<0)?`- ${-c}`:`+ ${c}`;
        }else if(equation1Format==3){
            equation1=`${y1-y2}y = ${-x}x `;
            equation1+=(c<0)?`- ${-c}`:`+ ${c}`;
        }else if(equation1Format==4){
            equation1=`${x}x `;
            equation1+=((y1-y2)<0)?`- ${-(y1-y2)}y = ${c}`:`+ ${y1-y2}y = ${c}`;
        }else if(equation1Format==5){
            equation1=`${x}x `;
            equation1+=((y1-y2)<0)?`- ${-(y1-y2)}y `:`+ ${y1-y2}y `;
            equation1+=(-c<0)?`- ${c} = 0`:`+ ${-c} = 0`;
        }

        let chosenLetter:number=randint(0,alphabet.length-1);
        equation2=`${a}x + ${alphabet[chosenLetter]}y = ${d}`;
        problem=`In the given system of equations, ${alphabet[chosenLetter]} is a constant. If the system has no solution, what is the value of ${alphabet[chosenLetter]}?`;
        solutions.push(b);
    }

    function typeH(){
        //factor of x + 2b
        let a:number=1;
        let bCoefficient:number=randint(2,5);
        let c:number=randint(2,5);
        let d:number=randint(1,15)
        while(bCoefficient%d==0 || d%bCoefficient==0){
            d=randint(1,15);
        }
        let options:number[]=[0,0,0,0];
        for(let i in options){
            let decidedValue:number=d*randint(3,10);
            while((decidedValue-d)%bCoefficient==0 || (decidedValue==options[0] || decidedValue==options[1] || decidedValue==options[2] || decidedValue==options[3])){
                decidedValue=d*randint(3,10);
            }
            options[i]=decidedValue;
        }
        let chosenCorrect=randint(0,4);
        options[chosenCorrect]=(bCoefficient*randint(3,10))+d;
        let alphabet:string[]=["a","b","c","d","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let chosenLetter:number=randint(0,alphabet.length-1);
        problem=`Which of the following has a factor of (x + ${bCoefficient}${alphabet[chosenLetter]}), where b is a positive integer constant?`;
        mcqdiv.updateOptions(`${a*c}x² + ${options[0]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[1]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[2]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[3]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[chosenCorrect]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`);
    }
</script>

<h1>SAT MATHHH AHH AHH AHH</h1>
<h3>{equation1}</h3>
<h3>{equation2}</h3>
<h3>{problem}</h3>

<McqDiv bind:this={mcqdiv}></McqDiv>
<OpenResponse bind:this={openResponse}></OpenResponse>
<button style={checkAnswerVisible?`display:block`:`display:none`} onclick={submitAnswer}>Check answer</button>
<h1>{feedback}</h1>
<br>
<!--<h1><Fraction n=4 d=5></Fraction><div class="inline-block relative bottom-2 left-1">x + 3</div></h1>-->
<button onclick={makeQuestion} style={makeQuestionVisible?`display:block`:`display:none`}>make question</button>