<script lang="ts">
    let problem:string=$state("");
    let solution:number=0;
    let pos:boolean=$state(true)
    let userAnswer:string=$state("");

    let feedback:string=$state("");

    function randint(min:number, max:number):number{
        let x:number=Math.floor(Math.random()*(max-min+1)+min);
        while(x==0){
            x=Math.floor(Math.random()*(max-min+1)+min);
        }
        return x;
    }
    function makeQuestion(){

        //let choice:number=randint(1,5);
        let choice:number=3;
        if(choice==1){
            typeA();
        }else if(choice==2){
            
        }else if(choice==3){
            typeC();
        }else if(choice==4){
            console.log("if jx+k factor, what is ac")
        }else if(choice==5){
            console.log("horizontal line with equation intersects parabola at 1 point")
        }
        
    }

    function checkAnswer():boolean{
        let x:number=parseFloat(userAnswer);
        console.log(x, solution);
        if(x==solution){
            feedback="Correct!";
            return true;
        }else{
            feedback="Incorrect...try again!";
            return false;
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
            exponent1="^2";
            exponent2="";
        }else{ //quartic
            problem+="quartic ";
            exponent1="^4";
            exponent2="^2";
        }

        if(k==1){
            (a*c)<0?(problem+=`${a*c*-1}x${exponent1} + ${(a*d + c*b)*-1}x${exponent2} + ${b*d*-1}`):(problem+=`${a*c}x${exponent1} + ${a*d + c*b}x${exponent2} + ${b*d}`);
            problem+=` can be factored as (ax${exponent2} + b)(cx${exponent2} + d), where a, b, c, and d are integers. `;
        }else{
            (a*c)<0?(problem+=`${a*c*-1*k}x${exponent1} + ${(a*d + c*b)*-1*k}x${exponent2} + ${b*d*-1*k}`):(problem+=`${a*c*k}x${exponent1} + ${(a*d + c*b)*k}x${exponent2} + ${b*d*k}`);
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
                        solution=a+b;
                    }else{
                        solution=c+d;
                    }
                }else{
                    if(a+b>c+d){
                        solution=a+b;
                    }else{
                        solution=c+d;
                    }
                }
            }else if(operation==2){
                problem+=`a - b?`;
                if(smallest){
                    if(a-b<c-d){
                        solution=a-b;
                    }else{
                        solution=c-d;
                    }
                }else{
                    if(a-b>c-d){
                        solution=a-b;
                    }else{
                        solution=c-d;
                    }
                }
            }else if(operation==3){
                problem+=`ab?`;
                if(smallest){
                    if(a*b<c*d){
                        solution=a*b;
                    }else{
                        solution=c*d;
                    }
                }else{
                    if(a*b>c*d){
                        solution=a*b;
                    }else{
                        solution=c*d;
                    }
                }
            }else if(operation==4){
                problem+=`a/b?`;
                if(smallest){
                    if(a/b<c/d){
                        solution=a/b;
                    }else{
                        solution=c/d;
                    }
                }else{
                    if(a/b>c/d){
                        solution=a/b;
                    }else{
                        solution=c/d;
                    }
                }
            }
        }else{
            let operation:number=randint(1,4);
            if(operation==1){
                problem+=`c + d?`;
                if(smallest){
                    if(a+b<c+d){
                        solution=a+b;
                    }else{
                        solution=c+d;
                    }
                }else{
                    if(a+b>c+d){
                        solution=a+b;
                    }else{
                        solution=c+d;
                    }
                }
            }else if(operation==2){
                problem+=`c - d?`;
                if(smallest){
                    if(a-b<c-d){
                        solution=a-b;
                    }else{
                        solution=c-d;
                    }
                }else{
                    if(a-b>c-d){
                        solution=a-b;
                    }else{
                        solution=c-d;
                    }
                }
            }else if(operation==3){
                problem+=`cd?`;
                if(smallest){
                    if(a*b<c*d){
                        solution=a*b;
                    }else{
                        solution=c*d;
                    }
                }else{
                    if(a*b>c*d){
                        solution=a*b;
                    }else{
                        solution=c*d;
                    }
                }
            }else if(operation==4){
                problem+=`c/d?`;
                if(smallest){
                    if(a/b<c/d){
                        solution=a/b;
                    }else{
                        solution=c/d;
                    }
                }else{
                    if(a/b>c/d){
                        solution=a/b;
                    }else{
                        solution=c/d;
                    }
                }
            }
        }

        if(solution < 0) pos=false;
        solution=Math.round(solution*1000)/1000;

        console.log(solution);
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

        console.log(`${a}x^2 + ${b}x + ${c}`);
        console.log(`${m}x + ${e}`);

        solution = (-(b-m))/(2*a);

        console.log(`x =`, solution);
        /*
        Possible unknowns (assuming y = ax^2 + bx + c; y = mx + e):
        - a
        - b
        - c
        - m
        - e
        */
        let whichUnknown:number=randint(1,5);
        if(whichUnknown==1){ //a
            
        }else if(whichUnknown==2){ //b

        }else if(whichUnknown==3){ //c

        }else if(whichUnknown==4){ //d

        }else if(whichUnknown==5){ //e

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
<h3>{problem}</h3>
<label for="answer">Answer:</label>
<input type="text" autocomplete="off" name="answer" id="answer" maxlength={pos?5:6} bind:value={userAnswer}>
<br>
<button onclick={checkAnswer}>Check answer</button>
<p>Enter your answer as an integer, an improper fraction, or a rounded decimal. See <a href="_blank" target="_blank" aria-label="SAT decimal rounding guide">SAT decimal rounding guide</a>.</p>
<br>
<h1>{feedback}</h1>
<br>
<button onclick={makeQuestion}>make question</button>