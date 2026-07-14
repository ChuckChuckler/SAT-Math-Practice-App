<script lang="ts">
    let checkAnswerVisible:boolean=$state(true);
    let answerInterfaceVisible:boolean=$state(false);
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

    function checkIfNegative(){
        (userAnswer.includes("-"))?pos=false:pos=true;
    }

    function makeQuestion(){
        makeQuestionVisible=false;
        checkAnswerVisible=true;
        answerInterfaceVisible=true;
        feedback="";
        userAnswer="";
        equation1="";
        equation2="";

        let choice:number=randint(1,5);
        //let choice:number=3;
        if(choice==1){
            typeA();
        }else if(choice==2){
            typeB()
        }else if(choice==3){
            typeC();
        }else if(choice==4){
            typeD();
        }else if(choice==5){
            typeE();
        }
    }

    function checkAnswer():boolean{
        let x:number=0;
        userAnswer.includes("/")?x=(Math.round(parseFloat(userAnswer.split("/")[0])/parseFloat(userAnswer.split("/")[1])*1000))/1000:x=parseFloat(userAnswer);
        for(let i of solutions){
            //console.log(i);
            if(x==i){
                feedback="Correct!";
                solutions=[];
                makeQuestionVisible=true;
                checkAnswerVisible=false;
                return true;
            }
        }
        feedback="Incorrect...try again!";
        return false;
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
        problem=`In the given expression, a and c are positive integer constants. If (${alphabet[randint(0,alphabet.length-1)]}x + ${alphabet[randint(0,alphabet.length-1)]}) is a factor of the expression, where j and k are positive constants, what is a possible value of ac?`
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
    /*List of parabola questions:
    - smallest or largest values of ab
    - find a+c, a+d, b+c, b+d...int and non-int
    - a is constant, intersect at point (x,y), where do they intersect
    - f jx+k factor, what is ac
    - line intersects at one point
    */
</script>

<h1>SAT MATHHH AHH AHH AHH</h1>
<h3>{equation1}</h3>
<h3>{equation2}</h3>
<h3>{problem}</h3>
<div id="answerInputContainer" style={answerInterfaceVisible?`display:block`:`display:none`}>
    <label for="answer">Answer:</label>
    <input type="text" autocomplete="off" name="answer" id="answer" maxlength={pos?5:6} bind:value={userAnswer} oninput={checkIfNegative}>
    <br>
    <button onclick={checkAnswer} style={checkAnswerVisible?`display:block`:`display:none`}>Check answer</button>
    <p>Enter your answer as an integer, an improper fraction, or a rounded decimal. See <a href="_blank" target="_blank" aria-label="SAT decimal rounding guide">SAT decimal rounding guide</a>.</p>
    <br>
</div>
<h1>{feedback}</h1>
<br>
<button onclick={makeQuestion} style={makeQuestionVisible?`display:block`:`display:none`}>make question</button>