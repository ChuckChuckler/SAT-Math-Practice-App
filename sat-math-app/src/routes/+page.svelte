<script lang="ts">
    import { onMount } from "svelte";

    import Equation from "$lib/comps/Equation.svelte";

    //comps
    import McqDiv from "$lib/comps/McqDiv.svelte";
    import Number from "$lib/comps/Number.svelte";
    import OpenResponse from "$lib/comps/OpenResponse.svelte";
    import Possibles from "$lib/comps/Possibles.svelte";

    //imgs
    import typemImg from "$lib/images/typeM.png";
    import typeyImg from "$lib/images/typeY.png";

    let mcqdiv:any=$state();
    let openResponse:any=$state();
    let possibles:any=$state();
    let imageVisible:boolean=$state(false);
    let fraction:any=$state();
    let imgBind:any=$state();

    let eq1Visible:boolean=$state(false);
    let eq2Visible:boolean=$state(false);
    let checkAnswerVisible:boolean=$state(true);
    let makeQuestionVisible:boolean=$state(true);

    let desmosVisible:boolean=$state(false);

    let problem:string=$state("");
    let solutions:number[]=[];

    let type:string=$state("");
    let equation1:any=$state();
    let equation2:any=$state();

    let feedback:string=$state("");

    let steps:string[]=$state([]);
    let displayedSteps:string[]=$state([]);
    let stepsIndex=0;

    function randint(min:number, max:number):number{ //for when 0 is unwanted
        let x:number=Math.floor(Math.random()*(max-min+1)+min);
        while(x==0){
            x=Math.floor(Math.random()*(max-min+1)+min);
        }
        return x;
    }

    function randint2(min:number,max:number):number{ //for when 0 is ok
        return Math.floor(Math.random()*(max-min+1)+min);
    }

    let questionsSorted:Record<string, Record<string,()=>void>>={
        "Algebra":{
            "type G":typeG,
            "type J":typeJ,
            "type F":typeF,
            "type N":typeN,
            "type O":typeO,
            "type R":typeR,
            "type X":typeX
        },
        "Advanced Math":{
            "type E":typeE,
            "type A":typeA,
            "type B":typeB,
            "type C":typeC,
            "type D":typeD,
            "type H":typeH,
            "type K":typeK,
            "type P":typeP,
            "type Q":typeQ,
            "type S":typeS,
            "type T":typeT
        },
        "Problem-Solving and Data Analysis":{
            "type I":typeI,
            "type U":typeU
        },
        "Geometry and Trigonometry":{
            "type K":typeK,
            "type L":typeL,
            "type M":typeM,
            "type V":typeV,
            "type W":typeW,
            "type Y":typeY
        }
    }

    onMount(()=>{
        makeQuestion();
    })

    function makeQuestion():void{
        makeQuestionVisible=false;
        checkAnswerVisible=true;
        imageVisible=false;
        possibles.makeVisible(false);
        openResponse.reset();
        feedback="";
        steps=[];
        displayedSteps=[];
        stepsIndex=0;

        let domain=questionsSorted[Object.keys(questionsSorted)[randint2(0,3)]];
        let index:number=randint2(0,Object.keys(domain).length-1);
        domain[Object.keys(domain)[index]]();
        type=Object.keys(domain)[index];
        //typeX();

        eq1Visible=(equation1.getNumbers().length==0)?false:true;
        eq2Visible=(equation2.getNumbers().length==0)?false:true;
    }

    function createRandomAnswers(unrandomizedPassed:any[]):any[]{
        let unrandomized=[...unrandomizedPassed];
        let randomized:any[]=[];
        for(let i=0;i<4;i++){
            let index:number=randint2(0,unrandomized.length-1);
            randomized.push(unrandomized[index]);
            unrandomized.splice(index,1);
        }
        return randomized;
    }

    function reduceFraction(numerator:number,denominator:number):number[]{
        let gcf:number=1;
        for(let i=2;i<=((Math.abs(denominator)<Math.abs(numerator))?Math.abs(denominator):Math.abs(numerator));i++){
            if(denominator%i==0&&numerator%i==0){
                gcf=i;
            }
        }
        return [numerator/gcf, denominator/gcf];
    }

    function getFactors(product:number):number[][]{
        let factors:number[][]=[];
        for(let i=1;i<=product;i++){
            if(product%i==0){
                factors.push([i,product/i]);
            }
        }
        return factors;
    }

    function getGCF(a:number,b:number):number{
        let gcf=1;
        let aFactors:number[][]=getFactors(a);
        let bFactors:number[][]=getFactors(b);
        for(let i of aFactors){
            for(let j of bFactors){
                if(i[0]==j[0]){
                    if(i[0]>gcf){
                        gcf=i[0];
                    }
                }
            }
        }
        return gcf;
    }

    function submitAnswer():void{
        let correct:boolean=false;
        if(mcqdiv.getVisible()){
            correct=mcqdiv.checkAnswer();
        }else{
            correct=openResponse.checkAnswer(solutions);
        }
        if(correct){
            feedback="Correct!";
            solutions=[];
            makeQuestionVisible=true;
            checkAnswerVisible=false;
        }else{
            feedback="Incorrect...Try again!!!"
        }
    }

    function makeEquationArr(eq1:string):any[][]{
        let equation1Arr:any[][]=[];
        if(eq1.includes("[fracfunc]")){
            eq1=eq1.replace("[fracfunc]","");
            let eq1arr:string[]=eq1.split("=");
            for(let i of eq1arr[0].split("")){
                equation1Arr.push([3,[i]]);
            }
            equation1Arr.push([3,["="]]);
            equation1Arr.push([3,[" "]]);
            equation1Arr.push([1,[eq1arr[1].split("/")[0],eq1arr[1].split("/")[1]]]);
        }else{
            for(let i of eq1.split(" ")){
                if(i.includes("/")){ //is a fraction
                    let twoParentheses=i.includes("))");
                    let fracAndVar:string[]=i.split(")");
                    if(fracAndVar[0].includes("((")){
                        equation1Arr.push([3,["("]]);
                    }
                    equation1Arr.push([1,[fracAndVar[0].split("/")[0].replaceAll("(",""),fracAndVar[0].split("/")[1]]]);
                    if(fracAndVar[1]!=""){
                        equation1Arr.push([3," "]);
                        equation1Arr.push([3,[fracAndVar[1]]]);
                    }
                    if(twoParentheses) equation1Arr.push([3,[")"]]);
                }else{ //is not a fraction
                    for(let j=0;j<i.length;j++){
                        if(isNaN(parseFloat(i[j]))){
                            equation1Arr.push([3,[i[j]]]);
                        }else{
                            if(i[j]=="1" || i[j]=="-1"){
                                if(!isNaN(parseInt(i[j+1])) || !isNaN(parseInt(i[j-1])) || j==i.length-1 || i[j+1]=="."){
                                    equation1Arr.push([2,[i[j]]]);
                                }
                            }else if(i[j]=="0" && j==0){
                                if(!(i[j+1]=="." || i[j-1]==".")){
                                    equation1Arr.splice(equation1Arr.length-4,4);
                                    break;
                                }
                            }else{
                                equation1Arr.push([2,[i[j]]]);
                            }
                        }
                    }
                }
                equation1Arr.push([3,[" "]]);
            }
        }
        return equation1Arr;
    }

    function round(number:number):number{
        return Math.round((number)*(Math.pow(10,4-(number).toString().split(".")[0].replace("-","").length)))/(Math.pow(10,4-(number).toString().split(".")[0].replace("-","").length));
    }

    function typeA():void{ //smallest possible value of ab?
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);

        problem="The ";

        var exponent1:string;
        var exponent2:string;

        let a:number=randint(1,5);
        let b:number=randint(-10,10);

        let c:number=randint(1,5);
        let d:number=randint(-10,10);

        while((a*c)+(b*d)==0){
            d=randint(-10,10);
        }

        var quadOrQuart=randint(1,2);

        if(quadOrQuart==1){ //quadratic
            problem+="quadratic ";
            exponent1="²";
            exponent2="";
            steps.push("This is a quadratic function with a, b, and c given.");
        }else{ //quartic
            problem+="quartic ";
            exponent1="⁴";
            exponent2="²";
            steps.push("The given function is a quartic, but its factored form uses x² with a and c. So we can simply factor the quartic as if it were a quadratic, using x² instead of x.")
        }

        steps.push("Let's factor the function!");

        let k:number=0;
        if(getGCF(a,b)!=1){
            k=getGCF(a,b);
        }else{
            k=getGCF(c,d);
        }
        console.log(k);
        let eq:string="";
        if(k==1){
            if(a*c<0){
                eq=`${a*c*-1}x${exponent1} `;
                ((a*d + c*b)*-1 < 0)?eq+=`- ${(a*d + c*b)}x${exponent2}`:eq+=`+ ${(a*d + c*b)*-1}x${exponent2} `;
                (b*d*-1 < 0)?eq+=`- ${b*d}`:eq+=`+ ${b*d*-1}`
            }else{
                eq=`${a*c}x${exponent1} `;
                ((a*d + c*b) < 0)?eq+=`- ${(a*d + c*b)*-1}x${exponent2}`:eq+=`+ ${(a*d + c*b)}x${exponent2} `;
                (b*d < 0)?eq+=`- ${b*d*-1}`:eq+=`+ ${b*d}`;
            }
            problem+=eq;
            problem+=` can be factored as (ax${exponent2} + b)(cx${exponent2} + d), where a, b, c, and d are integers. `;
        }else{
            if(a*c<0){
                eq=`${a*c*-1*k}x${exponent1} `;
                ((a*d + c*b)*-1 < 0)?eq+=`- ${(a*d + c*b)*k}x${exponent2} `:eq+=`+ ${(a*d + c*b)*k*-1}x${exponent2} `;
                (b*d*-1 < 0)?eq+=`- ${b*d*k}`:eq+=`+ ${b*d*k*-1}`;
            }else{
                eq=`${a*c}x${exponent1} `;
                ((a*d + c*b) < 0)?eq+=`- ${(a*d + c*b)*-1}x${exponent2} `:eq+=`+ ${(a*d + c*b)}x${exponent2} `;
                (b*d < 0)?eq+=`- ${b*d*-1}`:eq+=`+ ${b*d}`
            }
            problem+=eq;
            problem+=` can be factored as (k)(ax${exponent2} + b)(cx${exponent2} + d), where a, b, c, d, and k are integers. `;
        }
        steps.push(`Our function ${eq} can be factored as (${a}x${exponent2} + ${b})(${c}x${exponent2} + ${d}).`);
        let step:string="";

        if(k!=1){
            steps.push(`But wait! Our question asks for something more. According to the question, we should be able to factor this equation as k(ax${exponent2} + b)(cx${exponent2} + d). Right now, we just have (ax${exponent2} + b)(cx${exponent2} + d).`);
            step=`Let's look at our first factor, (${a}x${exponent2} + ${b}). `
            if(a%k==0&&b%k==0){
                step+=`It looks like ${a} and ${b}, both share a common factor. They have a gcf of ${k}!`;
                step+=`That means that we can take ${k} out of the factor and place it in front of our factored equation to get ${k}(${a/k}x${exponent2} + ${b/k})(${c}x${exponent2} + ${d})-- the form the question asks for.`;
            }else{
                step+=`Seems like ${a} and ${b} do not share any common factors. On the other hand, in our second factor (${c}x${exponent2} + ${d}), ${c} and ${d} both have a gcf of ${k}!`;
                step+=`That means that we can take ${k} out of the factor and place it in front of our factored equation to get ${k}(${a}x${exponent2} + ${b})(${c/k}x${exponent2} + ${d/k})-- the form the question asks for.`;
            }
            steps.push(step);
        }

        var smallest:boolean=false;
        let smallOrLarge:number=randint(1,2);
        if(smallOrLarge==1){
            problem+=`What is the smallest possible value of `;
            smallest=true;
            step=`We are looking for the smallest value of `;
        }else{
            problem+=`What is the largest possible value of `;
            smallest=false;
            step=`We are looking for the largest value of `;
        }

        /*
        possibilities:
        ab or cd
        a+b, a-b, a*b, or a/b
        c+d, c-d, c*d, or c/d
        */

        let abOrCd:number=randint(1,2);
        let operation:number=randint(1,4);

        if(abOrCd==1){
            if(operation==1){
                problem+=`a + b?`;
                step+=`a + b.`;
            }else if(operation==2){
                problem+=`a - b?`;
                step+=`a - b.`;
            }else if(operation==3){
                problem+=`ab?`;
                step+=`ab.`;
            }else if(operation==4){
                problem+=`a/b?`;
                step+=`a/b.`;
            }
        }else{
            if(operation==1){
                problem+=`c + d?`;
                step+=`c + d.`;
            }else if(operation==2){
                problem+=`c - d?`;
                step+=`c - d.`;
            }else if(operation==3){
                problem+=`cd?`;
                step+=`cd.`;
            }else if(operation==4){
                problem+=`c/d?`;
                step+=`c/d.`;
            }
        }

        steps.push(step);

        if(operation==1){
            if(smallest){
                if(a+b<c+d){
                    step=`Since ${a} + ${((b<0)?`(${b}) (= ${a+b}) is smaller than ${c} + `:`${b} (= ${a+b}) is smaller than ${c} + `)}${(d<0)?`${d} (= ${c+d}), we can determine that ${a} + `:`(${d}) (= ${c+d}), we can determine that ${a} + `}${(b<0)?`(${b}), or ${a+b}, is the solution to this question.`:`${b}, or ${a+b}, is the solution to this question.`}`;
                    // step+=(b<0)?`(${b}) (= ${a+b}) is smaller than ${c} + `:`${b} (= ${a+b}) is smaller than ${c} + `;
                    //step+=`${b}, or ${a+b}, is the solution to this question.`;
                    solutions.push(a+b);
                }else{
                    step=`Since ${c} + ${((d<0)?`(${d}) (= ${c+d}) is smaller than ${c} + `:`${d} (= ${c+d}) is smaller than ${a} + `)}${(b<0)?`${b} (= ${a+b}), we can determine that ${c} + `:`(${b}) (= ${a+b}), we can determine that ${c} + `}${(d<0)?`(${d}), or ${c+d}, is the solution to this question.`:`${d}, or ${c+d}, is the solution to this question.`}`;
                    solutions.push(c+d);
                }
            }else{
                if(a+b>c+d){
                    //step=`Since ${a} + ${b} (${a+b}) is larger than ${c} + ${d} (${c+d}), we can determine that ${a} + ${b}, or ${a+b}, is the solution to this question.`;
                    step=`Since ${a} + ${((b<0)?`(${b}) (= ${a+b}) is larger than ${c} + `:`${b} (= ${a+b}) is larger than ${c} + `)}${(d<0)?`${d} (= ${c+d}), we can determine that ${a} + `:`(${d}) (= ${c+d}), we can determine that ${a} + `}${(b<0)?`(${b}), or ${a+b}, is the solution to this question.`:`${b}, or ${a+b}, is the solution to this question.`}`;
                    solutions.push(a+b);
                }else{
                    //step=`Since ${c} + ${d} (${c+d}) is larger than ${a} + ${b} (${a+b}), we can determine that ${c} + ${d}, or ${c+d}, is the solution to this question.`;
                    step=`Since ${c} + ${((d<0)?`(${d}) (= ${c+d}) is larger than ${c} + `:`${d} (= ${c+d}) is larger than ${a} + `)}${(b<0)?`${b} (= ${a+b}), we can determine that ${c} + `:`(${b}) (= ${a+b}), we can determine that ${c} + `}${(d<0)?`(${d}), or ${c+d}, is the solution to this question.`:`${d}, or ${c+d}, is the solution to this question.`}`;
                    solutions.push(c+d);
                }
            }
        }else if(operation==2){
            if(smallest){
                if(a-b<c-d){
                    step=`Since ${a} - ${((b<0)?`(${b}) (= ${a-b}) is smaller than ${c} - `:`${b} (= ${a-b}) is smaller than ${c} - `)}${(d<0)?`${d} (= ${c-d}), we can determine that ${a} - `:`(${d}) (= ${c-d}), we can determine that ${a} - `}${(b<0)?`(${b}), or ${a-b}, is the solution to this question.`:`${b}, or ${a-b}, is the solution to this question.`}`;
                    solutions.push(a-b);
                }else{
                    step=`Since ${c} - ${((d<0)?`(${d}) (= ${c-d}) is smaller than ${c} - `:`${d} (= ${c-d}) is smaller than ${a} - `)}${(b<0)?`${b} (= ${a-b}), we can determine that ${c} - `:`(${b}) (= ${a-b}), we can determine that ${c} - `}${(d<0)?`(${d}), or ${c-d}, is the solution to this question.`:`${d}, or ${c-d}, is the solution to this question.`}`;
                    solutions.push(c-d);
                }
            }else{
                if(a-b>c-d){
                    step=`Since ${a} - ${((b<0)?`(${b}) (= ${a-b}) is larger than ${c} - `:`${b} (= ${a-b}) is larger than ${c} - `)}${(d<0)?`${d} (= ${c-d}), we can determine that ${a} - `:`(${d}) (= ${c-d}), we can determine that ${a} - `}${(b<0)?`(${b}), or ${a-b}, is the solution to this question.`:`${b}, or ${a-b}, is the solution to this question.`}`;
                    solutions.push(a-b);
                }else{
                    step=`Since ${c} - ${((d<0)?`(${d}) (= ${c-d}) is larger than ${c} - `:`${d} (= ${c-d}) is larger than ${a} - `)}${(b<0)?`${b} (= ${a-b}), we can determine that ${c} - `:`(${b}) (= ${a-b}), we can determine that ${c} - `}${(d<0)?`(${d}), or ${c-d}, is the solution to this question.`:`${d}, or ${c-d}, is the solution to this question.`}`;
                    solutions.push(c-d);
                }
            }
        }else if(operation==3){
            if(smallest){
                if(a*b<c*d){
                    step=`Since ${a} * ${b} (= ${a*b}) is smaller than ${c} * ${d} (= ${c*d}), we can determine that ${a} * ${b}, or ${a*b}, is the solution to this question.`;
                    solutions.push(a*b);
                }else{
                    step=`Since ${c} * ${d} (= ${c*d}) is smaller than ${a} * ${b} (= ${a*b}), we can determine that ${c} * ${d}, or ${c*d}, is the solution to this question.`;
                    solutions.push(c*d);
                }
            }else{
                if(a*b>c*d){
                    step=`Since ${a} * ${b} (= ${a*b}) is larger than ${c} * ${d} (= ${c*d}), we can determine that ${a} * ${b}, or ${a*b}, is the solution to this question.`;
                    solutions.push(a*b);
                }else{
                    step=`Since ${c} * ${d} (= ${c*d}) is larger than ${a} * ${b} (= ${a*b}), we can determine that ${c} * ${d}, or ${c*d}, is the solution to this question.`;
                    solutions.push(c*d);
                }
            }
        }else if(operation==4){
            if(smallest){
                if(a/b<c/d){
                    step=`Since ${a}/${((b<0)?`(${b}) (≈ ${Math.round(a/b*10)/10}) is smaller than ${c} / `:`${b} (≈ ${Math.round(a/b*10)/10}) is smaller than ${c} / `)}${(d<0)?`${d} (≈ ${Math.round(c/d*10)/10}), we can determine that ${a} / `:`(${d}) (≈ ${Math.round(c/d*10)/10}), we can determine that ${a} / `}${(b<0)?`(${b}) is the solution to this question.`:`${b} is the solution to this question.`}`;
                    solutions.push(a/b);
                }else{
                    step=`Since ${c}/${((d<0)?`(${d}) (≈ ${Math.round(c/d*10)/10}) is smaller than ${a} / `:`${d} (≈ ${Math.round(c/d*10)/10}) is smaller than ${a} / `)}${(b<0)?`${b} (≈ ${Math.round(a/b*10)/10}), we can determine that ${c} / `:`(${b}) (≈ ${Math.round(a/b*10)/10}), we can determine that ${c} / `}${(d<0)?`(${d}) is the solution to this question.`:`${d} is the solution to this question.`}`;
                    solutions.push(c/d);
                }
            }else{
                if(a/b>c/d){
                    step=`Since ${a}/${((b<0)?`(${b}) (≈ ${Math.round(a/b*10)/10}) is larger than ${c} / `:`${b} (≈ ${Math.round(a/b*10)/10}) is larger than ${c} / `)}${(d<0)?`${d} (≈ ${Math.round(c/d*10)/10}), we can determine that ${a} / `:`(${d}) (≈ ${Math.round(c/d*10)/10}), we can determine that ${a} / `}${(b<0)?`(${b}) is the solution to this question.`:`${b} is the solution to this question.`}`;
                    solutions.push(a/b);
                }else{
                    step=`Since ${c}/${((d<0)?`(${d}) (≈ ${Math.round(c/d*10)/10}) is larger than ${a} / `:`${d} (≈ ${Math.round(c/d*10)/10}) is larger than ${a} / `)}${(b<0)?`${b} (≈ ${Math.round(a/b*10)/10}), we can determine that ${c} / `:`(${b}) (≈ ${Math.round(a/b*10)/10}), we can determine that ${c} / `}${(d<0)?`(${d}) is the solution to this question.`:`${d} is the solution to this question.`}`;
                    solutions.push(c/d);
                }
            }
        }

        steps.push(`Recall that we know the factors, but we do not know which factor corresponds to (ax${exponent2} + b) and which corresponds to (cx${exponent2} + d). In other words, we do not know which numbers specifically correspond to a, b, c, and d.`);
        steps.push(`That means that, since we want the ${(smallest)?`smallest`:`largest`} possible value, we want the factor with the two numbers that will give us the ${(smallest)?`smaller`:`larger`} value.`);
        steps.push(step);
        solutions[0]=Math.round(solutions[0]*1000)/1000;
    }

    function typeB():void{ //ab integers cd nonintegers
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
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

        steps.push("We're given a quadratic in factored form. This means that whatever a and b, or c and d, are, they will be the x-intercepts/roots of our function.");
        steps.push("Use any method to solve the expression for y = 0 and find your roots.");

        let ya:number=((r1*b)+(r2*a) + Math.sqrt(Math.pow((r1*b)+(r2*a),2)-(4*r1*(a*b*r2))))/(2*r1);
        let yb:number=((r1*b)+(r2*a) - Math.sqrt(Math.pow((r1*b)+(r2*a),2)-(4*r1*(a*b*r2))))/(2*r1);

        if(ya.toString().includes(".")){ //find the solution that is noninteger; that is d
            d=ya;
            c=(a*b)/ya;
        }else if(yb.toString().includes(".")){
            d=yb;
            c=(a*b)/yb;
        }

        let reducedSolution1:number[]=reduceFraction((-(r1*b+r2*a) + Math.sqrt(Math.pow(r1*b+r2*a,2)-4*(r1*r2)*(a*b))),(2*r1*r2));
        let reducedSolution2:number[]=reduceFraction((-(r1*b+r2*a) - Math.sqrt(Math.pow(r1*b+r2*a,2)-4*(r1*r2)*(a*b))),(2*r1*r2));
        let reducedSolutionsStepForm:string=`${(reducedSolution1[1]==1)?`${reducedSolution1[0]}`:`${reducedSolution1[0]}/${reducedSolution1[1]}`} and ${(reducedSolution2[1]==1)?`${reducedSolution2[0]}`:`${reducedSolution2[0]}/${reducedSolution2[1]}`}`;
        steps.push(`The roots of the expression given here are ${reducedSolutionsStepForm}.`);
        steps.push(`Now, looking at our factored form, we can see that the x coefficients of each factor are ${r1} and ${r2} respectively. The question states that for one pair of factors, constants a and b will be integers; for the other pair, constants c and d will be nonintegers.`);
        steps.push(`We can get our values for a, b, c, and d by plugging in the roots we calculated for into x in the factored forms and solving for 0. Whether we find a and b or c and d depends on which coefficients we pair our roots with.`);
        steps.push(`In other words, plugging ${reducedSolutionsStepForm} into the given factored form in one order will result in our solutions being integers (a and b), while plugging them in the opposite order will result in our solutions being nonintegers (c and d).`);
        steps.push(`Multiplying ${reducedSolutionsStepForm.split(" and ")[0]} by ${r1} gives us ${Math.round((reducedSolution1[0]*r1)/reducedSolution1[1]*100)/100}, and multiplying ${reducedSolutionsStepForm.split(" and ")[1]} by ${r2} gives us ${Math.round((reducedSolution2[0]*r2)/reducedSolution2[1])*100/100}. ${(((reducedSolution2[0]*r2)/reducedSolution2[1]).toString().includes("."))?`Neither of these are integers, which means we've found c and d:`:`Both of these are integers, which means we've found a and b:`} ${-(reducedSolution1[0]*r1)/reducedSolution1[1]} and ${-(reducedSolution2[0]*r2)/reducedSolution2[1]} respectively.`);
        steps.push(`Then, ${(((reducedSolution2[0]*r2)/reducedSolution2[1]).toString().includes("."))?`a and b, the integers, `:`c and d, the nonintegers, `}can be found by plugging in the roots in the opposite order: ${reducedSolutionsStepForm.split(" and ")[1]} by ${r1} to get ${Math.round((reducedSolution2[0]*r1)/reducedSolution2[1]*100)/100} and ${reducedSolutionsStepForm.split(" and ")[0]} by ${r2} to get ${Math.round((reducedSolution1[0]*r2)/reducedSolution1[1]*100)/100}.`);
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
            steps.push(`We're looking for a + c, so we simply add the values we've calculated for a and c together.`);
            solutions.push(a+c);
        }else if(whatToFind==2){ //a+d
            problem+=`a + d?`;
            steps.push(`We're looking for a + d, so we simply add the values we've calculated for a and c together.`);
            solutions.push(a+d);
        }else if(whatToFind==3){ //b+c
            problem+=`b + c?`;
            steps.push(`We're looking for b + c, so we simply add the values we've calculated for a and c together.`);
            solutions.push(b+c);
        }else{
            problem+=`b + d?`;
            steps.push(`We're looking for b + d, so we simply add the values we've calculated for a and c together.`);
            solutions.push(b+d);
        }

        solutions[0]=round(solutions[0]);
    }

    function typeC():void{ //non-horizontal line intersects parabola at 1 point
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
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
            w=randint(1,5);
            x=randint(-10,10);
            y=randint(1,5);
            z=randint(-10,10);

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

        let eq1:string="";
        let eq2:string="";
        let chosenUnknownName:string=``;
        if(whichUnknown==1){ //a
            chosenUnknownName=`a`;
            eq1=`y = ax² `;
            (b<0)?eq1+=`- ${-b}x `:eq1+=`+ ${b}x `;
            (c<0)?eq1+=`- ${-c}`:eq1+=`+ ${c}`;
            eq2=`y = ${m}x `;
            (e<0)?eq2+=`- ${-e}`:eq2+=`+ ${e}`;
        }else if(whichUnknown==2){ //b
            chosenUnknownName=`b`;
            eq1=`y = ${a}x² + bx `;
            (c<0)?eq1+=`- ${-c}`:eq1+=`+ ${c}`;
            eq2=`y = ${m}x `;
            eq2+=(e<0)?`- ${-e}`:`+ ${e}`;
        }else if(whichUnknown==3){ //c
            chosenUnknownName=`c`;
            eq1=`y = ${a}x² `;
            eq1+=(b<0)?`- ${-b}x + c`:`+ ${b}x + c`;
            eq2=`y = ${m}x `;
            eq2+=(e<0)?`- ${-e}`:`+ ${e}`;
        }else if(whichUnknown==4){ //d
            chosenUnknownName=`m`;
            eq1=`y = ${a}x² `;
            eq1+=(b<0)?`- ${-b}x `:`+ ${b}x `;
            eq1+=(c<0)?`- ${-c}`:`+ ${c}`;
            eq2=`y = mx `;
            eq2+=(e<0)?`- ${-e}`:`+ ${e}`;
        }else if(whichUnknown==5){ //e
            chosenUnknownName=`e`;
            eq1=`y = ${a}x² `;
            eq1+=(b<0)?`- ${-b}x `:`+ ${b}x `;
            eq1+=(c<0)?`- ${-c}`:`+ ${c}`;
            eq2=`y = ${m}x + e`;
        }

        equation1.updateEquation(makeEquationArr(eq1));
        equation2.updateEquation(makeEquationArr(eq2));

        let xOrY:number=randint(1,2);
        let chosenForSolution:string=``;

        if(xOrY==1){ //x
            solutions.push(round(-(b-m))/(2*a));
            if(chosenUnknownName=="b"){
                solutions.push(round(-(-(b-m))/(2*a)));
            }
            chosenForSolution="x";
        }else{
            x = (-(b-m))/(2*a);
            solutions.push(round(m*x + e));
            if(chosenUnknownName=="b"){
                x = -(-(b-m))/(2*a);
                solutions.push(round(m*x + e));
            }
            chosenForSolution="y";
        }

        steps.push("Although we don't know one of the values of this system, we do know that the system will have exactly one solution. This is a bigger clue than you might think.");
        steps.push("Since we know the system will have exactly one solution, we can utilize the discriminant formula to help us find the unknown value in the two equations. To do that, we need to get one equation out of the two here.");
        steps.push("Since both equations equal y, we can set both equations equal to each other and move terms so that all terms are on one side with an equation equaling 0.")
        let step:string=`The equation for 0 we get after combining ${eq1} and ${eq2} is: `;
        if(chosenUnknownName=="a"){
            step+=`ax² ${(b-m<0)?`- ${-(b-m)}x`:`+ ${b-m}x`}  ${(c-e<0)?`- ${-(c-e)}`:`+ ${c-e}`} = 0.`;
        }else if(chosenUnknownName=="b"){
            step+=`${a}x² + (b - ${m})x  ${(c-e<0)?`- ${-(c-e)}`:`+ ${c-e}`} = 0.`;
        }else if(chosenUnknownName=="c"){
            step+=`${a}x² ${(b-m<0)?`- ${-(b-m)}x`:`+ ${b-m}x`} + (c - (${e})) = 0.`;
        }else if(chosenUnknownName=="m"){
            step+=`${a}x² + (${b} - m)x ${(c-e<0)?`- ${-(c-e)}`:`+ ${c-e}`} = 0.`;
        }else{
            step+=`${a}x² ${(b-m<0)?`- ${-(b-m)}x`:`+ ${b-m}x`} + (${c} - e) = 0.`;
        }
        steps.push(step);
        steps.push(`We know that this combined equation will only have one correct solution, so we can plug it into the discriminant formula and solve for 0 (as when the discriminant is 0, the quadratic has one solution).`);
        step=`In the discriminant formula, this will be `;
        if(chosenUnknownName=="a"){
            step+`(${b-m})² - 4(a)(${c-e}).`;
            steps.push(step);
            steps.push(`After solving this for a, we can determine that a = ${a}.`);
        }else if(chosenUnknownName=="b"){
            step+=`(b ${(m<0)?`+ ${-m}`:`- ${m}`})² - 4(${a})(${c-e}).`;
            steps.push(step);
            steps.push(`After solving this for b, we can determine that b = ${b}`);
        }else if(chosenUnknownName=="c"){
            step+=`(${b-m})² - 4(${a})(c ${(e<0)?`+ ${-e}`:`- ${e}`}).`;
            steps.push(step);
            steps.push(`After solving this for c, we can determine that c = ${c}`);
        }else if(chosenUnknownName=="m"){
            step+=`(${b} - m)² - 4(${a})(${c-e}).`;
            steps.push(step);
            steps.push(`After solving this for m, we can determine that m = ${m}`);
        }else{
            step+=`(${b-m})² - 4(${a})(${c} - e).`;
            steps.push(step);
            steps.push(`After solving this for e, we can determine that e = ${e}`);
        }
        steps.push(`But wait, this isn't our answer! Don't get ahead of yourself! Our question is asking for the ${chosenForSolution} coordinate of the point of intersection, NOT the value of ${chosenUnknownName}!`);
        steps.push(`Now that we know the value of ${chosenUnknownName}, we can plug it back into the original system and solve the system to find our point of intersection, aka solution.`);
        if(solutions.length==2){
            steps.push(`The ${chosenForSolution} coordinate of our solution can be ${solutions[0]} or ${solutions[1]}.`);
        }else{
            steps.push(`The ${chosenForSolution} coordinate of our solution is ${solutions[0]}.`);
        }

        (chosenUnknownName=="b")?problem=`In the given system of equations, ${chosenUnknownName} is a constant. The graphs of the equations in the given system interact at exactly one point, (x, y), in the xy-plane. What is one possible value of ${chosenForSolution}?`:problem=`In the given system of equations, ${chosenUnknownName} is a constant. The graphs of the equations in the given system interact at exactly one point, (x, y), in the xy-plane. What is the value of ${chosenForSolution}?`;
    }

    function typeD():void{ //(jx+k) is a factor
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
        console.log("if jx+k factor, what is ac")
        let alphabet:string[]=["a","b","c","d","e","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let b:number=randint(1,150)*2;
        equation1.updateEquation(makeEquationArr(`ax² + ${b}x + c`));
        let letter1=randint(0,alphabet.length-1);
        let letter2=randint(0,alphabet.length-1);
        while(letter1==letter2){
            letter2=randint(0,alphabet.length-1);
        }
        solutions.push(Math.pow(b,2)/4);
        problem=`In the given expression, a and c are positive integer constants. If (${alphabet[letter1]}x + ${alphabet[letter2]}) is a factor of the expression, where ${alphabet[letter1]} and ${alphabet[letter2]} are positive constants, what is the greatest possible value of ac?`;
        steps.push(`The fact that we are given a factor of the quadratic means that the quadratic has at least one real solution or root.`);
        steps.push(`That means we can use the discriminant formula to determine the value of ac.`);
        steps.push(`Since we want the greatest possible value, we have to find the value of ac that makes the discriminant equal 0. Anything greater would make the discriminant negative, thus signaling that the system has no solutions.`);
        steps.push(`Recall that the discriminant formula is b² - 4ac.`);
        steps.push(`Since ac is already present in the discriminant formula, we can simply plug in ${b} for b and solve for ac together.`);
        steps.push(`After solving the discriminant, we end up with ac = ${solutions[0]}.`);
    }

    function typeE():void{ //horizontal line intersects a parabola at 1 point
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
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
            steps.push(`The first thing we should do is isolate the y side of the linear equation.`);
            steps.push(`We'll rewrite ${coeffY}y = ${Math.round(c*coeffY*10000)/10000} as y = ${Math.round(c*1000)/1000}.`);
            steps.push(`Now both of the equations in our system are rewritten to be equal to y.`);
            steps.push(`Since one of the equations of the system is a quadratic, we can combine the two parts of the system to make one equation, then use the discriminant to find our unknown b. (That's why we rewrote that other equation to isolate y!)`);
            steps.push(`We'll combine our equations on one side and have 0 on the other side. We'll get ${a}x² + bx ${(c>0)?`- ${Math.round(c*1000)/1000}`:`+ ${-Math.round(c*1000)/1000}`} = 0.`);
            steps.push(`The question defines that the system has exactly one solution, so our discriminant should equal exactly 0.`);
            steps.push(`Plug a (${a}), our unknown b, and ${-Math.round(c*1000)/1000} into the discriminant to get: b² - 4(${a})(-${Math.round(c*1000)/1000}) = 0.`);
            steps.push(`After solving this, we'll get two possible values for b: ${b} and ${-b}.`); 
            let posOrNeg:number=randint(1,2);
            if(posOrNeg==1){ //pos
                problem+=`positive constant, what is the value of b?`;
                solutions.push(b);
                steps.push(`Since our question states that b is a POSITIVE constant, it follows that b must be ${b}.`);
            }else{
                problem+=`negative constant, what is the value of b?`;
                solutions.push(-b);
                steps.push(`Since our question NEGATIVE constant, it follows that b must be ${-b}.`);
            }
        }else{ //c is unknown
            let coeffY:number=randint(1,3)*2;
            problem+=`${coeffY}y = c for some constant c intersects a parabola at exactly one point. If the parabola has equation y = ${a}x² + ${b}x, what is the value of c?`;
            solutions.push(Math.round(c*coeffY*1000)/1000);
            steps.push(`The first thing we should do is rewrite the equation with our constant c for y.`);
            steps.push(`We'll rewrite ${coeffY}y = c as y = c/${coeffY}.`);
            steps.push(`Now both of the equations in our system are rewritten to be equal to y.`);
            steps.push(`Since one of the equations of the system is a quadratic, we can combine the two parts of the system to make one equation, then use the discriminant to find our unknown c. (That's why we rewrote that other equation to isolate y!)`);
            steps.push(`We'll combine our equations on one side and have 0 on the other side. We'll get ${a}x² ${(b<0)?`- ${-b}`:`+ ${b}`}x ${(coeffY<0)?`+ c/${-coeffY}`:`- c/${coeffY}`} = 0.`);
            steps.push(`The question defines that the system has exactly one solution, so our discriminant should equal exactly 0.`);
            steps.push(`Plug a (${a}), b (${b}), and our unknown ${(coeffY<0)?`-c/${coeffY}`:`c/${coeffY}`} into the discriminant to get: (${b})² - 4(${a})(${(coeffY<0)?`c/${coeffY}`:`-c/${coeffY}`}) = 0.`);
            steps.push(`Finally, we'll solve for c using the discriminant formula; c = ${Math.round(c*coeffY*1000)/1000}.`);
        }
    }

    function typeF():void{ //g/k, infinite solutions
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
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

        let eq1:string="";
        let eq2:string="";

        eq1=`${a}x `;
        eq1+=(b<0)?`- ${-b}y = ${c}`:`+ ${b}y = ${c}`;
        eq2=`${alphabet[letter1]}x + ${alphabet[letter2]}y = ${d}`;
        let eq1Arr:any[][]=makeEquationArr(eq1);
        let eq2Arr:any[][]=makeEquationArr(eq2);
        /*console.log(eq1Arr);
        console.log(eq2Arr);*/
        equation1.updateEquation(eq1Arr);
        equation2.updateEquation(eq2Arr);
        
        steps.push(`The system has infinite solutions. This means that both functions will be identical.`);
        steps.push(`In a linear system like this, one equation will usually be a scaled version of the other; e.g. all coefficients and constants of the second equation will be the same as the first equation's, but perhaps all multiplied by the exact same scale.`);
        steps.push(`As we can see, for the second equation in this system, we don't know our x and y coefficients, but we DO know our constant, ${d}.`);
        steps.push(`Since the two equations must factor out to be identical, it follows that whatever the constant of our first equation (${c}) was multiplied by to get the constant of our second equation (${d}) will be what the entire second equation was hypothetically multiplied by.`);
        steps.push(`We can set up our two constants ${c} and ${d} in an equation: ${c}x = ${d}.`);
        let scaleFraction:string=`${(d%c==0)?`${d/c}`:`${((c>0&&d<0||c<0&&d>0)?`${-Math.abs(reduceFraction(d,c)[0])}/${Math.abs(reduceFraction(d,c)[1])}`:`${Math.abs(reduceFraction(d,c)[0])}/${Math.abs(reduceFraction(d,c)[1])}`)}`}`;
        steps.push(`Solving this out gets us ${scaleFraction}.`);
        steps.push(`We now know that our first equation was multiplied by ${scaleFraction} to get our second. As such, we can multiply the x and y coefficients of the first equation by that same number to get ${alphabet[letter1]} and ${alphabet[letter2]}.`);
        let gFraction:string=`${(d*a%c==0)?`${d*a/c}`:`${((c>0&&d*a<0||c<0&&d*a>0)?`${-Math.abs(d*a)}/${Math.abs(c)}`:`${Math.abs(d*a)}/${Math.abs(c)}`)}`}`;
        let kFraction:string=`${(d*b%c==0)?`${d*b/c}`:`${((c>0&&d*b<0||c<0&&d*b>0)?`${-Math.abs(d*b)}/${Math.abs(c)}`:`${Math.abs(d*b)}/${Math.abs(c)}`)}`}`;
        steps.push(`${alphabet[letter1]} = ${a}(${scaleFraction}), or ${gFraction}.`);
        steps.push(`${alphabet[letter2]} = ${b}(${scaleFraction}), or ${kFraction}.`);
        problem=`In the given system of equations, ${alphabet[letter1]} and ${alphabet[letter2]} are constants. The system has infinitely many solutions. What is `;
        
        let solutionFormat = randint(1,3);
        solutionFormat=1;
        if(solutionFormat==1){ //g+k
            problem+=`${alphabet[letter1]} + ${alphabet[letter2]}?`;
            steps.push(`The question wants us to find the value of ${alphabet[letter1]} + ${alphabet[letter2]}.`);
            //console.log(Math.round((g+k)*(Math.pow(10,4-(g+k).toString().split(".")[0].replace("-","").length)))/(Math.pow(10,4-(g+k).toString().split(".")[0].replace("-","").length)));
            solutions.push(g+k);
            steps.push(`We'll add ${gFraction} and ${kFraction} to get ${(d*(a+b)%c==0)?`${(d*(a+b)/c)}`:`${((d*(a+b)>0&&c<0)||(d*(a+b)<0&&c>0))?`-`:``}${Math.abs(reduceFraction(d*(a+b),c)[0])}/${Math.abs(reduceFraction(d*(a+b),c)[1])}, or ~${solutions[0]}`}.`);
        }else if(solutionFormat==2){ //g-k
            problem+=`${alphabet[letter1]} - ${alphabet[letter2]}?`;
            steps.push(`The question wants us to find the value of ${alphabet[letter1]} - ${alphabet[letter2]}.`);
            solutions.push(round(g-k));
            steps.push(`We'll subtract ${gFraction} by ${kFraction} to get ${(d*(a-b)%c==0)?`${(d*(a-b)/c)}`:`${((d*(a-b)>0&&c<0)||(d*(a-b)<0&&c>0))?`-`:``}${Math.abs(reduceFraction(d*(a-b),c)[0])}/${Math.abs(reduceFraction(d*(a-b),c)[1])}, or ~${solutions[0]}`}.`);
        }else if(solutionFormat==3){ //gk
            problem+=`${alphabet[letter1]}${alphabet[letter2]}?`;
            steps.push(`The question wants us to find the value of ${alphabet[letter1]} * ${alphabet[letter2]}.`);
            solutions.push(round(g*k));
            steps.push(`We'll multiply ${gFraction} and ${kFraction} to get ${((d*d*a*b)%(c*c)==0)?`${((d*d*a*b)/(c*c))}`:`${(((d*d*a*b)>0&&(c*c)<0)||((d*d*a*b)<0&&(c*c)>0))?`-`:``}${Math.abs(reduceFraction(d*(a+b),(c*c))[0])}/${Math.abs(reduceFraction(d*d*a*b,(c*c))[1])}, or ~${solutions[0]}`}.`);
        }
    }

    function typeG():void{ //r, no solutions
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);

        let xNumerator:number=randint(2,11);
        let xDenominator:number=randint(2,11);

        while(xNumerator%xDenominator==0){
            xDenominator=randint(2,11);
        }

        let yNumerator:number=randint(2,11);
        let yDenominator:number=randint(2,11);

        while(yDenominator==xDenominator || yNumerator%yDenominator==0){
            yDenominator=randint(2,11);
        }

        let xFracs:number[]=reduceFraction(xNumerator, xDenominator);
        xNumerator=xFracs[0];
        xDenominator=xFracs[1];

        let yFracs:number[]=reduceFraction(yNumerator,yDenominator);
        yNumerator=yFracs[0];
        yDenominator=yFracs[1];

        yNumerator=reduceFraction(yNumerator,yDenominator)[0];
        yDenominator=reduceFraction(yNumerator,yDenominator)[1];

        let eq1:string=`(${xNumerator}/${xDenominator})x + (${yNumerator}/${yDenominator})y = -(${yNumerator}/${yDenominator})y + `;
        let cNumerator:number=randint(2,11);
        let cDenominator:number=randint(2,11);

        while(cDenominator%cNumerator==0){
            cDenominator=randint(2,11);
        }

        let cFracs:number[]=reduceFraction(cNumerator,cDenominator);    
        cNumerator=cFracs[0];
        cDenominator=cFracs[1];
        
        eq1+=`(${cNumerator}/${cDenominator})`;
        equation1.updateEquation(makeEquationArr(eq1));

        let xNumerator2:number=randint(1,11);
        let xDenominator2:number=randint(2,11);

        while(xNumerator2%xDenominator2==0 || xDenominator2==xDenominator){
            xDenominator2=randint(2,11);
        }

        let xFracs2:number[]=reduceFraction(xNumerator2, xDenominator2);
        while(xFracs2[1]==xDenominator){
            xDenominator2=randint(2,11);
            xFracs2=reduceFraction(xNumerator2, xDenominator2);
        }

        xNumerator2=xFracs2[0];
        xDenominator2=xFracs2[1];

        let eq2:string=`(${xNumerator2}/${xDenominator2})x + ky = `;
        
        let cNumerator2:number=randint(2,11);
        let cDenominator2:number=randint(2,11);

        while(cNumerator2%cDenominator2==0){
            cDenominator2=randint(2,11);
        }

        let xScale:number=(xNumerator2/xDenominator2)*(xDenominator/xNumerator);
        while((cNumerator/cDenominator)*xScale==(cNumerator2/cDenominator2)){
            let cNumerator2:number=randint(2,11);
            let cDenominator2:number=randint(2,11);
            while(cNumerator2%cDenominator2==0){
                cDenominator2=randint(2,11);
            }
        }

        eq2+=`${reduceFraction(cNumerator2, cDenominator2)[0]}/${reduceFraction(cNumerator2, cDenominator2)[1]}`;
        equation2.updateEquation(makeEquationArr(eq2));
        problem=`In the given system of equations, k is a constant. If the system has no solution, what is the value of k?`;

        steps.push(`Since this is a system of linear equations with no solution, it follows that the two lines must be parallel.`);
        steps.push(`Let's change the format of the second equation to make it the same as the first equation's format.`);
        steps.push(`We'll move (${-yNumerator}/${yDenominator})y to the left side of the equation so that both equations in the system are in Ax + By = C format.`);
        steps.push(`The first equation will become (${xNumerator}/${xDenominator})x + (${yNumerator*2}/${yDenominator})y = ${cNumerator}/${cDenominator}.`);
        steps.push(`Recall that the two lines must be parallel. In other words, they must have the same slope-- by extension, same coefficients-- but different constants.`);
        steps.push(`We have two different x coefficients, ${xNumerator}/${xDenominator} and ${xNumerator2}/${xDenominator2}.`);
        steps.push(`Since the two equations must have the same slope, it follows that whatever was done to one x coefficient to get the second one must have been done to the relative y coefficient.`);
        steps.push(`Therefore, our next step is figuring out what the x coefficient of our first equation was multiplied by to get the x coefficient of our second equation.`);
        steps.push(`We can set this up as (${xNumerator}/${xDenominator})a = ${xNumerator2}/${xDenominator2}, where a is what the first x coefficient was multiplied by to get the second one.`);
        let a:string=`${reduceFraction(xNumerator2*xDenominator,xNumerator*xDenominator2)[0]}/${reduceFraction(xNumerator2*xDenominator,xNumerator*xDenominator2)[1]}`;
        steps.push(`After doing this out, we can determine that a is ${a}.`);
        steps.push(`Again, we know that the slopes for both equations must be equal, and so we can find k by multiplying our first y coefficient ${yNumerator}/${yDenominator} by ${a}.`);
        steps.push(`The value of k equals ${reduceFraction(xNumerator2*xDenominator*yNumerator,xNumerator*xDenominator2*yDenominator)[0]}/${reduceFraction(xNumerator2*xDenominator*yNumerator,xNumerator*xDenominator2*yDenominator)[1]}.`);
        solutions.push(round((xNumerator2*xDenominator*yNumerator)/(xNumerator*xDenominator2*yDenominator)));
    }

    function typeH():void{ //has a factor of (x + 2b), what could be the equation...
        //factor of x + 2b
        openResponse.makeVisible(false);
        mcqdiv.makeVisible(true);
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
        let chosenCorrect=randint(0,3);
        options[chosenCorrect]=(bCoefficient*randint(3,10))+d;
        let alphabet:string[]=["d","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let chosenLetter:number=randint(0,alphabet.length-1);
        problem=`Which of the following has a factor of (x + ${bCoefficient}${alphabet[chosenLetter]}), where b is a positive integer constant?`;
        mcqdiv.updateOptions(`${a*c}x² + ${options[0]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[1]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[2]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[3]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`, `${a*c}x² + ${options[chosenCorrect]}x + ${bCoefficient*d}${alphabet[chosenLetter]}`);

        steps.push(`The first thing that we can notice is that for every option, assuming the format ax² + bx + c, a is ${a*c} and c is ${bCoefficient*d}${alphabet[chosenLetter]}.`);
        steps.push(`We know that our correct option WILL have the factor (x + ${bCoefficient}${alphabet[chosenLetter]}).`);
        steps.push(`Our next step should be finding the other factor that will give us a = ${a*c} and c = ${bCoefficient*d}${alphabet[chosenLetter]}.`);
        steps.push(`Since the coefficient before x in our given factor (x + ${bCoefficient}${alphabet[chosenLetter]}) is just one, we know that the coefficient of x in the second factor MUST be ${a*c} as ${a*c} * 1 is ${a*c}.`);
        steps.push(`Okay, so now we have (${a*c}x + an unknown constant)(x + ${bCoefficient}${alphabet[chosenLetter]}) = ${a*c}x² + (another unknown)x + ${bCoefficient*d}${alphabet[chosenLetter]}. By logic of expanding factored form, we should be able to find that unknown constant in our factor.`);
        steps.push(`What number, when multiplied by ${bCoefficient}${alphabet[chosenLetter]}, gives us ${bCoefficient*d}${alphabet[chosenLetter]}?`);
        steps.push(`The constant in our second factor must be ${d}.`);
        steps.push(`We now know that the factored form of our equation HAS to be (${a*c}x + ${d})(x + ${bCoefficient}${alphabet[chosenLetter]}).`);
        steps.push(`When we expand this, we get ${a*c}x² + (${a*bCoefficient}${alphabet[chosenLetter]} + ${d})x + ${bCoefficient*d}${alphabet[chosenLetter]}.`);
        steps.push(`Recall from the question that ${alphabet[chosenLetter]} must be an integer.`);
        steps.push(`Between the four options we have, the only thing that changes is that b value. So, we need to compare the four potential b values we are given and decide which one, when set equal to (${a*bCoefficient}${alphabet[chosenLetter]} + ${d}) to solve for ${alphabet[chosenLetter]}, will result in ${alphabet[chosenLetter]} being an integer.`);
        steps.push(`Out of the four potential b values, ${options[chosenCorrect]} is the only one where ${alphabet[chosenLetter]} would be an integer in ${a*bCoefficient}${alphabet[chosenLetter]} + ${d} = ${options[chosenCorrect]} (${alphabet[chosenLetter]} = ${(options[chosenCorrect]-d)/(a*bCoefficient)}).`)

    }

    function typeI():void{ //find the store cost of an item?
        //store cost of an item
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
        problem=`The regular cost of an item at a store is `;
        let regularCost:number=randint(500,2000)/100;
        problem+=`$${regularCost}. The sale price of the item is `;
        let salePricePercentOff:number=randint(60,90);
        problem+=`${salePricePercentOff}% less than the regular price, and the sale price is `;
        let salePricePercentGreater:number=randint(20,50);
        problem+=`${salePricePercentGreater}% greater than the store's cost for the item. What was the store's cost, in dollars, for the item? (Disregard the $ sign when entering your answer.)`;
        let s = regularCost*(1-(salePricePercentOff/100));
        let c = s*(100/(100+salePricePercentGreater));
        solutions.push(Math.round(c*100)/100);

        steps.push(`Let's ignore the given regular cost of the item so we don't confuse ourselves. For now, let's try to first set up an equation that will help us find the store cost of the item.`);
        steps.push(`We know that the sale price is ${salePricePercentOff}% LESS than the regular price, which will mean the sale price is the regular price minus ${salePricePercentOff}% of the regular price.`);
        steps.push(`In other words, if we let r = regular price and s = sale price, then s = r - ${salePricePercentOff/100}r, or, s = ${(100-salePricePercentOff)/100}r.`);
        steps.push(`We also know that the sale price is ${salePricePercentGreater}% greater than the store's cost, which means that the the sale price = the store cost + ${salePricePercentGreater}% of the store cost.`);
        steps.push(`If we define c = store cost and continue to use s = sale price, then c + ${salePricePercentGreater/100}c = s, or ${(100+salePricePercentGreater)/100}c = s.`);
        steps.push(`Since we're solving for c (store cost), we should isolate c; dividing both sides by ${(100+salePricePercentGreater)/100} gets us c = (1/${(100+salePricePercentGreater)/100})s.`);
        steps.push(`Recall that s = ${(100-salePricePercentOff)/100}r.`);
        steps.push(`In that case, we can replace s in the equation for store cost (c) with ${(100-salePricePercentOff)/100}r.`);
        steps.push(`The equation for store cost, c, now becomes c = (1/${(100+salePricePercentGreater)/100})(${(100-salePricePercentOff)/100}r).`);
        steps.push(`Since the question has provided us with our regular price, $${regularCost}, we can substitute that number into our equation for c.`);
        steps.push(`We can now solve c = (1/${(100+salePricePercentGreater)/100})(${(100-salePricePercentOff)/100}*${regularCost}).`);
        steps.push(`c ≈ ${solutions[0]}. The store cost of the item is $${solutions[0]}.`)
    }

    function typeJ():void{ //how many fewer miles? how many fewer gallons?
        openResponse.makeVisible(false);
        mcqdiv.makeVisible(true);
        let mpg:number=randint(20,40); //miles per gallon
        let mpw:number=mpg*randint(4,6); //miles per week
        let dpg:number=randint(3,6); //dollars per gallon
        let dtr:number=randint(2,5); //dollars to reduce
        let mtr:number=randint(10,30); //miles to reduce
        while(mpg==mtr){
            mtr=randint(10,30);
        }
        while(mpw==mtr){
            mtr=randint(10,30);
        }
        /*
        Potential problems? Equation to model:
        - How mnay fewer average miles
        - How many miles (NOT fewer)
        - How much money will be saved (if miles are reduced)
        - How much money he will be paying for gas (if miles are reduced)
        */

        let whatToSolve:number=randint(1,4);
        //whatToSolve=4;
        if(whatToSolve==1){
            problem=`Joe Banana drives an average of ${mpw} miles each week. His car travels an average of ${mpg} miles per gallon. Joe Banana would like to reduce his weekly expidenture on gas by $${dtr}. Assuming gas is $${dpg} per gallon, which equation models how many fewer miles, m, Joe Banana should drive each week?`;
            let optionsUnrandomized:string[]=["","","",""];
            optionsUnrandomized[0]=`(${mpg}/${dpg})* m = ${((mpw/mpg)*dpg)-dtr}`;
            optionsUnrandomized[1]=`(${mpg}/${dpg})* m = ${dtr}`;
            optionsUnrandomized[2]=`(${dpg}/${mpg})* m = ${((mpw/mpg)*dpg)-dtr}`;
            optionsUnrandomized[3]=`(${dpg}/${mpg})* m = ${dtr}`;
            let correct:string=optionsUnrandomized[3];
            let optionsRandomized:string[]=[];
            for(let i=0;i<4;i++){
                let index:number=randint2(0,optionsUnrandomized.length-1);
                optionsRandomized.push(optionsUnrandomized[index]);
                optionsUnrandomized.splice(index,1);
            }
            mcqdiv.updateOptions(optionsRandomized[0], optionsRandomized[1], optionsRandomized[2], optionsRandomized[3], correct);

            steps.push(`We want to find the number of FEWER miles Joe Banana should drive per week. In other words, we want to find how many miles yield a weekly expenditure of $${dtr}, because that's the number of miles Joe Banana should reduce.`);
            steps.push(`So our correct option will have ${dtr} on the right side of its equation. We can eliminate both options that have ${((mpw/mpg)*dpg)-dtr} on the right side, because those equations would give us how many miles Joe Banana should drive, not how many FEWER he should drive, per week.`);
            steps.push(`If we recall, we're given how far his car travels on a gallon of gasoline (${mpg}) and how much a gallon costs (${dpg}).`);
            steps.push(`m represents miles, not dollars.`);
            steps.push(`Therefore, we need to set up our ratio (aka our slope) as dollars over miles. That way, the values representing miles cancel out (as one, ${mpg}, is in a denominator, and one, m, is in the numerator), leaving only a dollar value.`);
            steps.push(`Another way to think about it is this: in ${dpg}m = ${dtr}, m would represent how many miles should be saved if Joe Banana's car could only move a mile per gallon of gas. But we know that Joe's car can actually move ${mpg} miles per gallon of gas, which is why we need to divide ${dpg}m by ${mpg}-- so we can determine how many gallons we actually need.`);
            steps.push(`The correct equation representing this scenario is (${dpg}/${mpg})m = ${dtr}.`);
        }else if(whatToSolve==2){
            problem=`Joe Banana drives an average of ${mpw} miles each week. His car travels an average of ${mpg} miles per gallon. Joe Banana would like to reduce his weekly expidenture on gas by $${dtr}. Assuming gas is $${dpg} per gallon, which equation models how many miles, m, Joe Banana should drive each week?`;
            let optionsUnrandomized:string[]=["","","",""];
            optionsUnrandomized[0]=`(${mpg}/${dpg})* m = ${((mpw/mpg)*dpg)-dtr}`;
            optionsUnrandomized[1]=`(${mpg}/${dpg})* m = ${dtr}`;
            optionsUnrandomized[2]=`(${dpg}/${mpg})* m = ${((mpw/mpg)*dpg)-dtr}`;
            optionsUnrandomized[3]=`(${dpg}/${mpg})* m = ${dtr}`;
            let correct:string=optionsUnrandomized[2];
            let optionsRandomized:string[]=[];
            optionsRandomized=createRandomAnswers(optionsUnrandomized);
            mcqdiv.updateOptions(optionsRandomized[0], optionsRandomized[1], optionsRandomized[2], optionsRandomized[3], correct);

            steps.push(`We want to find the number of miles Joe Banana should drive per week if he wants to save $${dtr}. So, we need to find the number of miles Joe Banana needs for his weekly expidenture on gas to be $${dtr} less than it is right now.`);
            steps.push(`We know how many miles he drives already (${mpw}), how many miles he can drive per gallon (${mpg}), and how much a gallon costs ($${dpg}), so we can figure out how much he pays for gas per week right now ((${mpw}/${mpg}) * ${dpg} = $${mpw/mpg*dpg}).`);
            steps.push(`What we solve for should be $${dtr} less than that: $${(mpw/mpg*dpg)-dtr}.`);
            steps.push(`And so our correct option will have ${(mpw/mpg*dpg)-dtr} on the right side of its equation. We can eliminate both options that have ${dtr} on the right side, because those equations would give us how many FEWER miles Joe Banana should drive per week, not how many he SHOULD drive.`);
            steps.push(`m represents miles, not dollars.`);
            steps.push(`Therefore, we need to set up our ratio (aka our slope) as dollars over miles. That way, the values representing miles cancel out (as one, ${mpg}, is in a denominator, and one, m, is in the numerator), leaving only a dollar value.`);
            steps.push(`Another way to think about it is this: in ${dpg}m = ${(mpw/mpg*dpg)-dtr}, m would represent how many miles Joe Banana should drive per week if Joe Banana's car could only move a mile per gallon of gas. But we know that Joe's car can actually move ${mpg} miles per gallon of gas, which is why we need to divide ${dpg}m by ${mpg}-- so we can determine how many gallons we actually need.`);
            steps.push(`The correct equation representing this scenario is (${dpg}/${mpg})m = ${(mpw/mpg*dpg)-dtr}.`);
        }else if(whatToSolve==3){
            problem=`Joe Banana drives an average of ${mpw} miles each week. His car travels an average of ${mpg} miles per gallon. Over the next week, Joe Banana will be driving ${mtr} miles less in total. Assuming gas is $${dpg} per gallon, which equation models how many dollars, d, Joe Banana will save on gas next week?`;
            let optionsUnrandomized:string[]=["","","",""];
            optionsUnrandomized[0]=`(${(mpg)}/${mpw-mtr})[${dpg}] = d`;
            optionsUnrandomized[1]=`(${(mpw-mtr)}/${mpg})[${dpg}] = d`;
            optionsUnrandomized[2]=`(${mpg}/${mtr})[${dpg}] = d`;
            optionsUnrandomized[3]=`(${mtr}/${mpg})[${dpg}] = d`;
            let correct:string=optionsUnrandomized[3];
            let optionsRandomized:string[]=[];
            for(let i=0;i<4;i++){
                let index:number=randint2(0,optionsUnrandomized.length-1);
                optionsRandomized.push(optionsUnrandomized[index]);
                optionsUnrandomized.splice(index,1);
            }
            mcqdiv.updateOptions(optionsRandomized[0], optionsRandomized[1], optionsRandomized[2], optionsRandomized[3], correct);
            
            steps.push(`We want to know how much money Joe Banana will be saving. In order to determine that, we need to use the number of fewer miles that he'll be driving per week.`);
            steps.push(`We can eliminate any option that doesn't have ${mtr} in it, because those options would be hinting towards Joe Banana's new expidenture on gas, not how much he'd be saving.`);
            steps.push(`We know that Joe Banana can drive ${mpg} miles with a gallon of gas, and that each gallon costs ${dpg} dollars.`);
            steps.push(`We need to know how many miles Joe can drive per gallon so we can accurately see how many gallons of gas he'll need for a certain amount of miles (${mtr}), and then we can use that to find out how much that many gallons of gas will cost.`);
            steps.push(`Therefore, we need to use ${mpg} miles per gallon in our equation.`);
            steps.push(`Since we're ultimately getting a price (unit: dollars only) through this equation, and our price of ${dpg} is per gallon (dollars/gallon), we should set up our equation so that our ratio cancels out the miles and leaves gallons in the numerator-- (miles)/(miles/gallons) * (dollars/gallon) --> gallons * (dollars/gallons) = dollars.`);
            steps.push(`The only equation that fits this is ${correct}.`);
        }else{
            problem=`Joe Banana drives an average of ${mpw} miles each week. His car travels an average of ${mpg} miles per gallon. Over the next week, Joe Banana will be driving ${mtr} miles less in total. Assuming gas is $${dpg} per gallon, which equation models how many dollars, d, Joe Banana will pay for gas next week?`;
            let optionsUnrandomized:string[]=["","","",""];
            optionsUnrandomized[0]=`(${(mpg)}/${mpw-mtr})[${dpg}] = d`;
            optionsUnrandomized[1]=`(${(mpw-mtr)}/${mpg})[${dpg}] = d`;
            optionsUnrandomized[2]=`(${mpg}/${mtr})[${dpg}] = d`;
            optionsUnrandomized[3]=`(${mtr}/${mpg})[${dpg}] = d`;
            let correct:string=optionsUnrandomized[1];
            let optionsRandomized:string[]=[];
            for(let i=0;i<4;i++){
                let index:number=randint2(0,optionsUnrandomized.length-1);
                optionsRandomized.push(optionsUnrandomized[index]);
                optionsUnrandomized.splice(index,1);
            }
            mcqdiv.updateOptions(optionsRandomized[0], optionsRandomized[1], optionsRandomized[2], optionsRandomized[3], correct);

            steps.push(`We want to know how much money Joe Banana will now pay weekly for gas after the reduction in weekly miles. In order to determine that, we need to find out how many miles per week he'll actually be driving now.`);
            steps.push(`We can find that by simply subtracting ${mtr}, the miles Joe will not be driving now, from ${mpw}, the previous amount of weekly miles Joe was driving.`);
            steps.push(`${mpw} - ${mtr} = ${mpw-mtr}.`);
            steps.push(`So we can eliminate any option that doesn't have ${mpw-mtr} in it, because those options would be hinting towards how much money Joe Banana would save, not how much his new expidenture on gas actually is.`);
            steps.push(`We know that Joe Banana can drive ${mpg} miles with a gallon of gas, and that each gallon costs ${dpg} dollars.`);
            steps.push(`We need to know how many miles Joe can drive per gallon so we can accurately see how many gallons of gas he'll need for a certain amount of miles (${mtr}), and then we can use that to find out how much that many gallons of gas will cost.`);
            steps.push(`Therefore, we need to use ${mpg} miles per gallon in our equation.`);
            steps.push(`Since we're ultimately getting a price (unit: dollars only) through this equation, and our price of ${dpg} is per gallon (dollars/gallon), we should set up our equation so that our ratio cancels out the miles and leaves gallons in the numerator-- (miles)/(miles/gallons) * (dollars/gallon) --> gallons * (dollars/gallons) = dollars.`);
            steps.push(`The only equation that fits this is ${correct}.`);
        }
        
    }

    function typeK():void{ //ktan33
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);
        let angleZ:number=randint(20,80);
        while(angleZ==30||angleZ==45||angleZ==60){
            angleZ=randint(20,80);
        }
        let lenYZ:number=randint(5,30);
        solutions.push(Math.pow(lenYZ,2)/2);
        problem=`In triangle XYZ, angle Y is a right angle, the measure of angle Z is ${angleZ}°, and the length of side YZ is ${lenYZ} units. If the area, in square units, of triangle XYZ can be represented by the expression ktan${angleZ}°, where k is a constant, what is the value of k?`;

        steps.push(`Recall that the tangent of a right triangle's angle is y/x, where y is the side opposite that angle and x is the side adjacent.`);
        steps.push(`We're given side YZ, which is ${lenYZ}. Side YZ is adjacent to angle Z, which means that x = side YZ and thus, x = ${lenYZ}.`);
        steps.push(`Remember that the area of a triangle is 1/2(bh). In a right triangle, b and h are the two legs-- x and y.`);
        steps.push(`Since tan = y/x, and we have both an angle measure (Z-- ${angleZ})° and its x (${lenYZ}).`);
        steps.push(`We can find side y by multiplying x(y/x) and therefore cancelling x out in the denominator.`);
        steps.push(`Since y/x = tan, it follows that x(y/x) is equivalent to xtan, or ${lenYZ}tan${angleZ}°.`);
        steps.push(`We now know our two legs, ${lenYZ} and ${lenYZ}tan${angleZ}°.`);
        steps.push(`Multiplying these gets us ${lenYZ*lenYZ}tan${angleZ}°.`);
        steps.push(`Again, the area of a triangle is 1/2(bh), so we need to divide ${lenYZ*lenYZ}tan${angleZ}° by 2.`);
        steps.push(`We get ${(lenYZ*lenYZ)/2}tan${angleZ}°.`);
        steps.push(`The question gives us the format ktan${angleZ}° and asks us for k. We have our area as ${(lenYZ*lenYZ)/2}tan${angleZ}°.`);
        steps.push(`Therefore, k = ${(lenYZ*lenYZ)/2}.`);
    }

    function typeL():void{ //sphere inside a cube
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
        let squareEdgeLength:number=randint(5,23)*2;
        let sphereRadius:number=squareEdgeLength/2;
        let sqVolume:number=Math.pow(squareEdgeLength,3);
        let sphVolume:number=(4/3)*Math.PI*Math.pow(sphereRadius,3);
        solutions.push(round(sqVolume-sphVolume));
        problem=`A cube has edge length ${squareEdgeLength} inches. A solid sphere with radius ${sphereRadius} inches is inside the cube, such that the sphere touches the center of each face of the cube. What is the volume of the space in the cube NOT taken up by the sphere?`;

        steps.push(`Let's first find the volume of the cube.`);
        steps.push(`The formula for the volume of a cube is s³, where s = the side length of the cube.`);
        steps.push(`We know the side length of the cube is ${squareEdgeLength}.`);
        steps.push(`The volume of the cube is ${squareEdgeLength}³, or ${Math.pow(squareEdgeLength,3)}.`);
        steps.push(`Next, we'll find the volume of the sphere.`);
        steps.push(`The formula for the volume of a sphere is (4/3)πr³, where r = the radius of the sphere.`);
        steps.push(`We know the radius of the sphere = ${sphereRadius}.`);
        steps.push(`The volume of the sphere is (4/3)π(${sphereRadius})³, or (4/3)(${Math.pow(sphereRadius, 3)})π.`);
        steps.push(`To find the volume of the cube not taken up by the sphere, we simply subtract the volume of the sphere from the volume of the cube to get the leftover volume.`);
        steps.push(`${Math.pow(squareEdgeLength,3)} - (4/3)(${Math.pow(sphereRadius, 3)})π ≈ ${solutions[0]}.`);
        steps.push(`The volume of the space not taken up by the sphere is ~${solutions[0]}.`);
    }

    function typeM():void{ //given a triangle and a smaller similar triangle inside, find DE, etc.
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);
        imageVisible=true;
        imgBind=typemImg;
        problem=`In the figure above, tan`;
        let ac:number=randint(3,15);
        let ba:number=randint(3,15);
        while(ac==ba || ac%ba==0 || ba%ac==0){
            ba=randint(3,8);
        }
        problem+=`B = ${ac}/${ba}. If BC = `;
        let bc:number=Math.pow(ac,2)+Math.pow(ba,2);
        let scale:number=randint(3,5);
        let bcInString:string=``;
        if(Math.sqrt(bc*Math.pow(scale,2)).toString().includes(".")){ //not a perfect square
            problem+=`√(${bc*Math.pow(scale,2)})`;
            bcInString=`√(${bc*Math.pow(scale,2)})`;
        }else{
            problem+=`${Math.sqrt(bc*Math.pow(scale,2))}`;
            bcInString=`${Math.sqrt(bc*Math.pow(scale,2))}`;
        }

        steps.push(`When a segment is drawn in a triangle that is perpendicular to one of the legs, the resulting smaller triangle within the original triangle is actually similar to the original triangle.`);
        steps.push(`DE is perpendicular to BA, which means that triangle BDE is actually similar to triangle BAC.`);
        steps.push(`That means that we can find the sides of △BDE to find those of △BAC, and vice versa.`);
        steps.push(`We're given BC, ${bcInString}, and tanB, ${ac}/${ba}. We can use tanB to find the other two sides of △BAC.`);
        steps.push(`We can't know what AC and BA are using just tanB. However, we do know that tanB will ultimately simplify to ${ac}/${ba}.`);
        steps.push(`That means that AC and BA, if not ${ac} and ${ba}, have to share some common factor that cancels out when the two are divided.`);
        steps.push(`We can use x to represent this common factor, and use pythagorean theorem to find the value of x.`);
        steps.push(`We know our hypotenuse, BC, is ${bcInString}. So we can set our equation for x up as (${ba}x)² + (${ac}x)² = ${bcInString}.`);
        steps.push(`x = ${scale}. Our scale factor is ${scale}. Side AC = ${ac} * ${scale} = ${ac*scale}, and side BA = ${ba} * ${scale} = ${ba*scale}.`)
        
        let toFind:number=randint(1,3);
        //toFind=3;
        if(toFind==1){ //de
            ba*=scale;
            let da:number=randint(3,10);
            while(ba-da<=0){
                da=randint(3,10);
            }
            problem+=` and DA = ${da}, what is the length of segment DE, rounded to the nearest hundredth?`;
            solutions.push(round((ac*scale)*((ba-da)/ba)));

            steps.push(`In the question, we are also given side DA = ${da}. Segment BA is made of segments DA and BD.`);
            steps.push(`This means we can subtract DA from BA to get BD.`);
            steps.push(`${ba*scale} - ${da} = ${(ba-da)}. BD = ${(ba-da)}.`);
            steps.push(`Because △BAC and △BDE are similar, we can now use BA and BD to find the scale factor between △BAC and △BDE.`);
            steps.push(`If we use a variable k to represent the scale factor between the two triangles, then BA(k) = BD.`);
            steps.push(`k = ${ba-da}/${ba}.`);
            steps.push(`Finally, since DE on △BDE corresponds to AC on △BAC, we can multiply AC by k (our scale factor) to get DE.`);
            steps.push(`${ac*scale}(${ba-da}/${ba}) ≈ ${round((ac*scale)*((ba-da)/ba))}. DE ≈ ${round((ac*scale)*((ba-da)/ba))}.`);
        }else if(toFind==2){ //da
            ba*=scale;
            ac*=scale;
            let de:number=randint(3,10);
            while(ac-de<=0){
                de=randint(3,10);
            }
            problem+=` and DE = ${de}, what is the length of segment DA?`;
            solutions.push(round((ba-(ba*(de/ac)))));

            steps.push(`In the question, we are also given side DE = ${de}. Remember that △BAC and △BDE are similar; side DE on △BDE corresponds to side AC on △BAD.`);
            steps.push(`This means that we can use DE and AC to find the scale factor between △BAC and △BDE.`);
            steps.push(`If we use a variable k to represent the scale factor between the two triangles, then AC(k) = DE.`);
            steps.push(`k = ${de}/${ac}.`);
            steps.push(`Since segment BA is composed of segments BD and DA together, and BD is part of △BDE, we can find BD and subtract it from BA to get DA.`);
            steps.push(`BD corresponds to BA, so BD = BA(k) (with k being our scale factor).`);
            steps.push(`DA will equal BA - BA(k).`);
            steps.push(`Substitue the length of BA and our scale factor into this, and we get DA = ${ba} - ${ba}(${de}/${ac}).`);
            steps.push(`DA ≈ ${solutions[0]}.`);
        }else if(toFind==3){ //find bd
            ba*=scale;
            ac*=scale;
            let de:number=randint(3,10);
            while(ac-de<=0){
                de=randint(3,10);
            }
            problem+=` and DE = ${de}, what is the length of segment BD?`;
            solutions.push(round(ba*(de/ac)));

            steps.push(`In the question, we are also given side DE = ${de}. Remember that △BAC and △BDE are similar; side DE on △BDE corresponds to side AC on △BAD.`);
            steps.push(`This means that we can use DE and AC to find the scale factor between △BAC and △BDE.`);
            steps.push(`If we use a variable k to represent the scale factor between the two triangles, then AC(k) = DE.`);
            steps.push(`k = ${de}/${ac}.`);
            steps.push(`Side BD in △BDE corresponds to side BA in △BAC, so we can use our newly-found scale factor between the two triangles to determine BD from BA.`);
            steps.push(`BD = BA(k) (where k represents the scale factor between the two triangles).`);
            steps.push(`BD = ${ba}(${de}/${ac}).`);
            steps.push(`BD = ${solutions[0]}.`);
        }
    }

    function typeN():void{ //find s or k idk
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);
        let alphabet:string[]=["a","b","c","d","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let whatToSolve:number=randint(1,4);
        //whatToSolve=4;
        if(whatToSolve==1){ //var outside, infinite solutions
            let toDistribute:number=randint(3,10);
            let xCoeff:number=randint(2,25);
            let constant:number=randint(2,25);
            let chosenLetter:number=randint(0,alphabet.length-1);
            let eq1:string=`${alphabet[chosenLetter]}(${xCoeff}x + ${constant}) = ${xCoeff*toDistribute}x + ${constant*toDistribute}`;
            equation1.updateEquation(makeEquationArr(eq1));
            problem=`In the given equation, ${alphabet[chosenLetter]} is a constant. The equation has infinite solutions. What is the value of ${alphabet[chosenLetter]}?`;
            solutions.push(toDistribute);

            steps.push(`If an equation has infinite solutions, that means that it's identical on both the left and right sides.`);
            steps.push(`We just need to find the value of ${alphabet[chosenLetter]} that multiplies with ${xCoeff} to get ${xCoeff*toDistribute} and multiplies with ${constant} to get ${constant*toDistribute}.`);
            steps.push(`${alphabet[chosenLetter]} = ${toDistribute}.`);
        }else if(whatToSolve==2){ //var outside, no solutions
            let toDistribute:number=randint(3,10);
            let xCoeff:number=randint(2,25);
            let constant:number=randint(2,25);
            let constantRight:number=randint(2*toDistribute, constant*toDistribute);
            while(constant*toDistribute==constantRight){
                constantRight=randint(2*toDistribute, constant*toDistribute);
            }
            let chosenLetter:number=randint(0,alphabet.length-1);
            let eq1:string=`${alphabet[chosenLetter]}(${xCoeff}x + ${constant}) = ${xCoeff*toDistribute}x + ${constantRight}`;
            equation1.updateEquation(makeEquationArr(eq1));
            problem=`In the given equation, ${alphabet[chosenLetter]} is a constant. The equation has no solutions. What is the value of ${alphabet[chosenLetter]}?`;
            solutions.push(toDistribute);

            steps.push(`If an equation has no solutions, that means that it comes out to something that's always false no matter what.`);
            steps.push(`In this case, the coefficients before the x should be the exact same so they can be cancelled out, leaving only two constants that are not equal to each other (e.g. 2 = 5 is false).`);
            steps.push(`So we need to find the value of ${alphabet[chosenLetter]} that multiplies with ${xCoeff} to get ${xCoeff*toDistribute}, but does not multiply with ${constant} to get ${constantRight}.`);
            steps.push(`${alphabet[chosenLetter]} = ${toDistribute}.`);
        }else if(whatToSolve==3){ //var inside, infinite solutions
            let toDistribute:number=randint(3,10);
            let xCoeff:number=randint(2,25);
            let constant:number=randint(2,25);
            let chosenLetter:number=randint(0,alphabet.length-1);
            let eq1:string=`${toDistribute}(${alphabet[chosenLetter]}x + ${constant}) = ${xCoeff*toDistribute}x + ${constant*toDistribute}`;
            equation1.updateEquation(makeEquationArr(eq1));
            solutions.push(xCoeff);
            problem=`In the given equation, ${alphabet[chosenLetter]} is a constant. The equation has infinite solutions. What is the value of ${alphabet[chosenLetter]}?`;

            steps.push(`If an equation has infinite solutions, that means that it's identical on both the left and right sides.`);
            steps.push(`We just need to find the value of ${alphabet[chosenLetter]} that multiplies with ${toDistribute} to get ${xCoeff*toDistribute}.`);
            steps.push(`${alphabet[chosenLetter]} = ${xCoeff}.`);
        }else if(whatToSolve==4){ //var inside, no solutions
            let toDistribute:number=randint(3,10);
            let xCoeff:number=randint(2,25);
            let constant:number=randint(2,25);
            let constantRight:number=randint(2*toDistribute, constant*toDistribute);
            while(constant==constantRight/toDistribute){
                constantRight=randint(2*toDistribute, constant*toDistribute);
            }
            let chosenLetter:number=randint(0,alphabet.length-1);
            let eq1:string=`${toDistribute}(${alphabet[chosenLetter]}x + ${constant}) = ${xCoeff*toDistribute}x + ${constantRight}`;
            equation1.updateEquation(makeEquationArr(eq1));
            problem=`In the given equation, ${alphabet[chosenLetter]} is a constant. The equation has no solutions. What is the value of ${alphabet[chosenLetter]}?`;
            solutions.push(xCoeff);

            steps.push(`If an equation has no solutions, that means that it comes out to something that's always false no matter what.`);
            steps.push(`In this case, the coefficients before the x should be the exact same so they can be cancelled out, leaving only two constants that are not equal to each other (e.g. 2 = 5 is false).`);
            steps.push(`We just need to find the value of ${alphabet[chosenLetter]} that multiplies with ${toDistribute} to get ${xCoeff*toDistribute}; we already know that ${toDistribute} * ${constant} ≠ ${constantRight}.`);
            steps.push(`${alphabet[chosenLetter]} = ${xCoeff}.`);
        }
    }

    function typeO():void{ //for each real number r, whch of the following points...
        mcqdiv.makeVisible(true);
        openResponse.makeVisible(false);
        let xCoeff:number=randint(2,10);
        let yCoeff:number=randint(2,10);
        let c:number=randint(1,10);
        let scale:number=randint(3,6);
        let eq1:string=`${xCoeff}x + ${yCoeff}y = ${c}`;
        equation1.updateEquation(makeEquationArr(eq1));
        let eq2:string=`${xCoeff*scale}x + ${yCoeff*scale}y = ${c*scale}`;
        equation2.updateEquation(makeEquationArr(eq2));
        problem=`For each real number r, which of the following points lies on the graph of each equation in the xy-plane for the given equation?`;
        let xOrY=randint(1,2);
        //xOrY=1;
        let optionsUnrandomized:string[]=["","","",""];
        let optionsRandomized:string[]=[];
        let correct:string="";
        optionsUnrandomized[0]=`((r/${scale})+ ${c}, (r/${scale})+ ${c*scale})`;

        steps.push(`If we compare the two equations in this system, we'll notice that they are actually the same equation.`);
        steps.push(`Because of that, we can just pick one of these two equations to find our points. We'll pick the first one since the numbers are smaller and more manageable.`);
        steps.push(`In the first equation, we'll plug r first into x, then into y, and solve for the other coordinate to get two possible points.`);
        steps.push(`Let's set x = r first. Our x coordinate will be r; let's solve for our y coordinate.`);
        steps.push(`${xCoeff}r + ${yCoeff}y = ${c}.`);
        steps.push(`After solving this, we get y = (${-xCoeff}/${yCoeff})r + ${c}/${yCoeff}.`);
        steps.push(`When x = r, our coordinate point is (r, (${-xCoeff}/${yCoeff})r + ${c}/${yCoeff}).`);

        if(xOrY==1){ //x = r, y = whatever whatever r
            console.log("h");
            optionsUnrandomized[1]=`(r, (${-xCoeff}/${yCoeff})r + ${c}/${yCoeff}))`;
            optionsUnrandomized[2]=`((${yCoeff}/${xCoeff})r + ${c}/${xCoeff}, r)`;
            optionsUnrandomized[3]=`((${-xCoeff}/${yCoeff})r + ${c}/${yCoeff}, r)`;
            correct=optionsUnrandomized[1];

            steps.push(`(r, ${-xCoeff}r/${yCoeff} + ${c}/${yCoeff}) is actually one of our option choices, so we can select it with no problem without needing to plug r into y.`);
        }else{
            optionsUnrandomized[1]=`((${-yCoeff}/${xCoeff})r + ${c}/${xCoeff}, r)`;
            optionsUnrandomized[2]=`(r, (${xCoeff}/${yCoeff})r + ${c}/${yCoeff}))`;
            optionsUnrandomized[3]=`(r, (${-yCoeff}/${xCoeff})r + ${c}/${xCoeff}))`;
            correct=optionsUnrandomized[1];

            steps.push(`(r, (${xCoeff}/${yCoeff})r + ${c}/${yCoeff}) is not one of our answer choices, so let's try plugging in r for y next and solving for x.`);
            steps.push(`${xCoeff}x + ${yCoeff}r = ${c}.`);
            steps.push(`After solving this, we get x = (${-yCoeff}/${xCoeff})r + ${c}/${xCoeff}.`);
            steps.push(`When y = r, our coordinate point is ((${-yCoeff}/${xCoeff})r + ${c}/${xCoeff}, r).`);
            steps.push(`This is one of our option choices, so we can now select it as our answer.`);
        }
        
        optionsRandomized=createRandomAnswers(optionsUnrandomized);

        mcqdiv.updateOptions(optionsRandomized[0], optionsRandomized[1], optionsRandomized[2], optionsRandomized[3], correct);
    }

    function typeP():void{ //the solutions are kab, what is k?
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);
        let product:number=randint(20,72);
        let factors:number[][]=getFactors(product);
        let factorsChosen:number=randint2(0,factors.length-1);
        let factor1:number=factors[factorsChosen][0];
        let factor2:number=factors[factorsChosen][1];
        let factorsNegative:number=randint(1,2);
        //factorsNegative=1;
        let eq1:string=factorsNegative==1?`${product}x² + (${factor1}a + ${factor2}b)x + ab`:`${product}x² - (${factor1}a + ${factor2}b)x + ab`;
        equation1.updateEquation(makeEquationArr(eq1));

        steps.push(`We can use the equation we're given to find the factored form, which we can then use to find the solutions.`);
        steps.push(`Since the constant of this equation is just ab, we can assume that the constants in our factored form will just be a and b, since-- obviously-- those multiply to just be ab.`);
        steps.push(`Since our linear coefficient (b in the format ax² + bx + c) is ${(factorsNegative==1)?`(${factor1}a + ${factor2}b)`:`-(${factor1}a + ${factor2}b)`}, we know that our factored equation will look something like ${(factorsNegative==1)?`([some factor]x + a)([some factor]x + b)`:`([some factor]x - a)([some factor]x - b)`}.`)
        steps.push(`Speaking of our linear coefficient, we can also use it to find the two coefficients for x in the two factors. In accordance with FOIL rules, because the linear coefficient, again, is ${(factorsNegative==1)?`(${factor1}a + ${factor2}b)`:`-(${factor1}a + ${factor2}b)`}, it follows that one coefficient of x in the factored form must be ${factor1} and one must be ${factor2}.`);
        steps.push(`Specifically, each coefficient must be in the opposite factor of the term it's multiplied by in the linear coefficient so they can actually be multiplied when FOILing. ${factor1} multiplies with a, so it must be in the factor with b; ${factor2} multiplies with b, so vice versa.`);
        steps.push(`Our factored form is ${(factorsNegative==1)?`(${factor2}x + a)(${factor1}x + b)`:`(${factor2}x - a)(${factor1}x - b)`}.`);
        steps.push(`Now let's set this equal to 0 so we can find our two solutions.`);
        if(factorsNegative==1){
            steps.push(`(${factor2}x + a) = 0. x = ${(factor2==1)?`-${factor2}a`:`-(1/${factor2})a`}.`);
            steps.push(`(${factor1}x + b) = 0. x = ${(factor1==1)?`-${factor1}b`:`-(1/${factor1})b`}`);
        }else{
            steps.push(`(${factor2}x - a) = 0. x = ${(factor2==1)?`${factor2}a`:`(1/${factor2})a`}.`);
            steps.push(`(${factor1}x - b) = 0. x = ${(factor1==1)?`${factor1}b`:`(1/${factor1})b`}.`);
        }


        let whatToFind:number=randint(1,4);
        //whatToFind=4;
        problem=`In the given equation, a and b are positive constants. The `;
        let kDenom:number=0;
        if(whatToFind==1 || whatToFind==2){ //sum or difference (they have identical setup)
            problem+=`${(whatToFind==2)?`difference`:`sum`} of the solutions to the given equation is `;
            if(factor1>factor2){
                factorsNegative==1?kDenom=-factor1:kDenom=factor1;
            }else if(factor2>factor1){
                factorsNegative==1?kDenom=-factor2:kDenom=factor2;
            }
            problem+=`k(`;
            problem+=(kDenom%factor2==0)?`${Math.abs(kDenom)/factor2}a `:`${Math.abs(kDenom)}/${factor2}a `;
            problem+=(whatToFind==2)?` - `:` + `;
            problem+=(kDenom%factor1==0)?`${Math.abs(kDenom)/factor1}b), `:`${Math.abs(kDenom)}/${factor1}b), `;
            solutions.push(round((1/kDenom)));

            if(whatToFind==1){
                steps.push(`The question asks us to find k if the sum of the solutions is k(${(kDenom%factor2==0)?`${Math.abs(kDenom)/factor2}a `:`(${Math.abs(kDenom)}/${factor2})a `} + ${(kDenom%factor1==0)?`${Math.abs(kDenom)/factor1}b`:`(${Math.abs(kDenom)}/${factor1})b`}).`);
                let step:string=`${(factorsNegative==1)?`${(factor2==1)?`-${factor2}a`:`-(1/${factor2})a`} - ${(factor1==1)?`${factor1}b`:`(1/${factor1})b`}`:`${(factor2==1)?`${factor2}a`:`(1/${factor2})a`} + ${(factor1==1)?`${factor1}b`:`(1/${factor1})b`}`}`;
                steps.push(step+` gives us the sum of the solutions.`);
                steps.push(`The form given by the question means that something-- k-- has to have been factored out of the sum. We need to find out what multiplies with (${(kDenom%factor2==0)?`${Math.abs(kDenom)/factor2}a `:`(${Math.abs(kDenom)}/${factor2})a `} + ${(kDenom%factor1==0)?`${Math.abs(kDenom)/factor1}b`:`(${Math.abs(kDenom)}/${factor1})b`}) to get (${step}), and that will be k.`);
                steps.push(`k = 1/${kDenom}, or ~${round(1/kDenom)}.`);
            }else{
                steps.push(`The question asks us to find k if the difference of the solutions is k(${(kDenom%factor2==0)?`${Math.abs(kDenom)/factor2}a `:`(${Math.abs(kDenom)}/${factor2})a `} - ${(kDenom%factor1==0)?`${Math.abs(kDenom)/factor1}b`:`(${Math.abs(kDenom)}/${factor1})b`}).`);
                let step:string=`${(factorsNegative==1)?`${(factor2==1)?`-${factor2}a`:`-(1/${factor2})a`} + ${(factor1==1)?`${factor1}b`:`(1/${factor1})b`}`:`${(factor2==1)?`${factor2}a`:`(1/${factor2})a`} - ${(factor1==1)?`${factor1}b`:`(1/${factor1})b`}`}`;
                steps.push(step+` gives us the difference of the solutions.`);
                steps.push(`The form given by the question means that something-- k-- has to have been factored out of the sum. We need to find out what multiplies with (${(kDenom%factor2==0)?`${Math.abs(kDenom)/factor2}a `:`(${Math.abs(kDenom)}/${factor2})a `} - ${(kDenom%factor1==0)?`${Math.abs(kDenom)/factor1}b`:`(${Math.abs(kDenom)}/${factor1})b`}) to get (${step}), and that will be k.`);
                steps.push(`k = ${(kDenom<0)?`-1/${-kDenom}`:`1/${kDenom}`}, or ~${round(1/kDenom)}.`);
            }
        }else if(whatToFind==3){ //product
            solutions.push(round(1/(factor1*factor2)));
            problem+=`product of the solutions to the given equation is kab, `;

            steps.push(`The question asks us to find k if the product of the solutions is kab.`);
            steps.push(`We just need to multiply the two solutions together. k = the product of the two coefficients in front of a and b.`);
            let step:string=`${(factorsNegative==1)?`${(factor2==1)?`-${factor2}a`:`-(1/${factor2})a`} * ${(factor1==1)?`${factor1}b`:`(1/${factor1})b`}`:`${(factor2==1)?`${factor2}a`:`(1/${factor2})a`} * ${(factor1==1)?`${factor1}b`:`(1/${factor1})b`}`}`;
            steps.push(step+` gives us the product of the solutions.`);
            steps.push(`k = 1/${factor1*factor2}, or ~${round(1/(factor1*factor2))}.`);
        }else{
            solutions.push(round(factor1/factor2));
            problem+=`quotient of the solutions to the given equation is k(a/b), `

            steps.push(`The question asks us to find k if the quotient of the solutions is k(a/b).`);
            steps.push(`We just need to divide the solution with a by the solution with b. k = the result of the two coefficients in front of a and b being divided (a/b).`);
            let step:string=`(${(factorsNegative==1)?`${(factor2==1)?`-${factor2}a`:`-(1/${factor2})a`})/(${(factor1==1)?`${factor1}b`:`(1/${factor1})b`})`:`${(factor2==1)?`${factor2}a`:`(1/${factor2})a`})/(${(factor1==1)?`${factor1}b`:`(1/${factor1})b`})`}`;
            steps.push(step+` gives us the quotient of the solutions.`);
            steps.push(`k = ${factor1}/${factor2}, or ~${round(factor1/factor2)}.`);
        }
        
        problem+=`where k is a constant. What is the value of k?`;
    }

    function typeQ():void{ //the function is...the graph passes through these two points...a is greater than 1...
        mcqdiv.makeVisible(true);
        openResponse.makeVisible(false);
        problem=`The function f is defined by f(x) = ax² + bx + c, where a, b, and c are constants. The graph of y = f(x) in the xy-plane passes through the points `;
        let r1:number=randint(-10,10);
        let r2:number=randint(-10,10);
        while(r1==r2){
            r2=randint(-10,10);
        }
        problem+=`(${r1}, 0) and (${r2}, 0). If a is an integer `
        let bUnscaled:number=-(r1+r2);
        let cUnscaled:number=r1*r2;
        let aGreaterOrLess:number=randint(1,2);
        //aGreaterOrLess=2;
        let aScale:number=randint(2,4);
        let optionsUnrandomized:number[]=[0,0,0,0];
        let optionsRandomized:number[]=[];
        let correct:number=0;

        steps.push(`Since we're given the x intercepts, (${r1}, 0) and (${r2}, 0), we can set them up in factored form as a(x ${(r1>0)?`- ${Math.abs(r1)}`:`+ ${r1}`})(x ${(r2>0)?`- ${Math.abs(r2)}`:`+ ${r2}`}). We don't know the value of a, so we'll put it in front as a variable.`);
        steps.push(`Expanding the factors gets us a(x² ${(bUnscaled<0)?`- ${Math.abs(bUnscaled)}`:`+ ${bUnscaled}`}x ${(r1*r2<0)?`- ${Math.abs(r1*r2)}`:`+ ${r1*r2}`}).`);
        steps.push(`Distributing a into the expanded form yields ax² ${(-(r1+r2)<0)?`- ${Math.abs(r1+r2)}`:`+ ${r1+r2}`}ax ${(r1*r2<0)?`- ${Math.abs(r1*r2)}`:`+ ${r1*r2}`}a.`);
        steps.push(`It follows that a = a, b = ${-(r1+r2)}a, and c = ${r1*r2}a.`);

        let sumDecided:string=``;

        if(aGreaterOrLess==1){ //a>1
            problem+=`greater than 1, which of the following could be the value of `;
            let whatToFind:number=randint(1,3);
            //whatToFind=3;
            if(whatToFind==1){ //a+b
                problem+=`a + b?`;
                optionsUnrandomized[0]=1+bUnscaled;
                optionsUnrandomized[1]=(1+bUnscaled)*aScale;
                optionsUnrandomized[2]=(1+bUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                while(optionsUnrandomized[2]%(1+bUnscaled)==0){
                    optionsUnrandomized[2]=(1+bUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                }
                optionsUnrandomized[3]=(1+bUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                while(optionsUnrandomized[2]==optionsUnrandomized[3] || optionsUnrandomized[3]%(1+bUnscaled)==0){
                    optionsUnrandomized[3]=(1+bUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                }

                steps.push(`Then, a + b = a + ${bUnscaled}a, or ${bUnscaled+1}a.`);
                sumDecided=`${bUnscaled+1}`;
            }else if(whatToFind==2){ //b+c
                problem+=`b + c?`;
                optionsUnrandomized[0]=(bUnscaled+cUnscaled);
                optionsUnrandomized[1]=(bUnscaled+cUnscaled)*aScale;
                optionsUnrandomized[2]=(bUnscaled+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                while(optionsUnrandomized[2]%(bUnscaled+cUnscaled)==0){
                    optionsUnrandomized[2]=(bUnscaled+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                }
                optionsUnrandomized[3]=(bUnscaled+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                while(optionsUnrandomized[2]==optionsUnrandomized[3] || optionsUnrandomized[3]%(bUnscaled+cUnscaled)==0){
                    optionsUnrandomized[3]=(bUnscaled+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                }

                steps.push(`Then, b + c = ${bUnscaled}a + ${cUnscaled}a, or ${bUnscaled+cUnscaled}a.`);
                sumDecided=`${bUnscaled+cUnscaled}`;
            }else{ //a+c
                problem+=`a + c?`;
                optionsUnrandomized[0]=(1+cUnscaled);
                optionsUnrandomized[1]=(1+cUnscaled)*aScale;
                optionsUnrandomized[2]=(1+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                while(optionsUnrandomized[2]%(1+cUnscaled)==0){
                    optionsUnrandomized[2]=(1+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                }
                optionsUnrandomized[3]=(1+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                while(optionsUnrandomized[2]==optionsUnrandomized[3] || optionsUnrandomized[3]%(1+cUnscaled)==0){
                    optionsUnrandomized[3]=(1+cUnscaled)*aScale+(randint2(-3,3)*aScale+1);
                }

                steps.push(`Then, a + c = a + ${cUnscaled}a, or ${1+cUnscaled}a.`);
                sumDecided=`${1+cUnscaled}`;
            }

            steps.push(`We are told that a is greater than 1, and that it is an integer.`);
            steps.push(`So we need to see which of the four options results in a being an integer greater than 1 when it is set equal to ${sumDecided}a. Or, which option, when divided by ${sumDecided}, yields an integer which is greater than 1.`);
            steps.push(`Out of the four options, ${optionsUnrandomized[1]} is the only one which, when divided by ${sumDecided}, is greater than 1 and an integer (${optionsUnrandomized[1]}/${sumDecided} = ${aScale}; a = ${aScale}).`);
        }else{ //a<1
            problem+=`less than 1, which of the following could be the value of `;
            let whatToFind:number=randint(1,3);
            //whatToFind=3;
            if(whatToFind==1){ //a+b
                problem+=`a + b?`;
                optionsUnrandomized[0]=1+bUnscaled;
                optionsUnrandomized[1]=(1+bUnscaled)*-aScale;
                optionsUnrandomized[2]=(1+bUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                while(optionsUnrandomized[2]%(1+bUnscaled)==0){
                    optionsUnrandomized[2]=(1+bUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                }
                optionsUnrandomized[3]=(1+bUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                while(optionsUnrandomized[2]==optionsUnrandomized[3] || optionsUnrandomized[3]%(1+bUnscaled)==0){
                    optionsUnrandomized[3]=(1+bUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                }

                steps.push(`Then, a + b = a + ${bUnscaled}a, or ${bUnscaled+1}a.`);
                sumDecided=`${bUnscaled+1}`;
            }else if(whatToFind==2){ //b+c
                problem+=`b + c?`;
                optionsUnrandomized[0]=(bUnscaled+cUnscaled);
                optionsUnrandomized[1]=(bUnscaled+cUnscaled)*-aScale;
                optionsUnrandomized[2]=(bUnscaled+cUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                while(optionsUnrandomized[2]%(bUnscaled+cUnscaled)==0){
                    optionsUnrandomized[2]=(bUnscaled+cUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                }
                optionsUnrandomized[3]=(bUnscaled+cUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                while(optionsUnrandomized[2]==optionsUnrandomized[3] || optionsUnrandomized[3]%(bUnscaled+cUnscaled)==0){
                    optionsUnrandomized[3]=(bUnscaled+cUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                }

                steps.push(`Then, b + c = ${bUnscaled}a + ${cUnscaled}a, or ${bUnscaled+cUnscaled}a.`);
                sumDecided=`${bUnscaled+cUnscaled}`;
            }else{ //a+c
                problem+=`a + c?`;
                optionsUnrandomized[0]=(1+cUnscaled);
                optionsUnrandomized[1]=(1+cUnscaled)*-aScale;
                optionsUnrandomized[2]=(1+cUnscaled)*-aScale+(randint2(-3,3)*-aScale+1);
                while(optionsUnrandomized[2]%(1+cUnscaled)==0){
                    optionsUnrandomized[2]=(1+cUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                }
                optionsUnrandomized[3]=(1+cUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                while(optionsUnrandomized[2]==optionsUnrandomized[3] || optionsUnrandomized[3]%(1+cUnscaled)==0){
                    optionsUnrandomized[3]=(1+cUnscaled)*-aScale+(randint2(-3,3)*-aScale-1);
                }

                steps.push(`Then, a + c = a + ${cUnscaled}a, or ${cUnscaled+1}a.`);
                sumDecided=`${cUnscaled+1}`;
            }

            steps.push(`We are told that a is less than 1, and that it is an integer.`);
            steps.push(`So we need to see which of the four options results in a being an integer less than 1 when it is set equal to ${sumDecided}a. Or, which option, when divided by ${sumDecided}, yields an integer which is less than 1.`);
            steps.push(`Out of the four options, ${optionsUnrandomized[1]} is the only one which, when divided by ${sumDecided}, is less than 1 and an integer (${optionsUnrandomized[1]}/${sumDecided} = ${-aScale}; a = ${-aScale}).`);
        }

        correct=optionsUnrandomized[1];

        optionsRandomized=createRandomAnswers(optionsUnrandomized);
        mcqdiv.updateOptions(optionsRandomized[0].toString(), optionsRandomized[1].toString(), optionsRandomized[2].toString(), optionsRandomized[3].toString(), correct.toString());
    }

    function typeR():void{ //the equation blah is given as blah, no solutions, what must be true..
        mcqdiv.makeVisible(true);
        openResponse.makeVisible(false);
        let unrandomized:string[]=["","",""];
        let randomized:string[]=[];
        let corrects:string[]=[];
        
        let m:number=randint(-10,10);
        let b:number=randint(-10,10);
        let eq1:string=`y = ${m}x `+((b<0)?`- ${-b}`:`+ ${b}`);
        let eq2:string=`y = a(x + b)`;
        equation1.updateEquation(makeEquationArr(eq1));
        equation2.updateEquation(makeEquationArr(eq2));
        let mustOrCould:number=randint(1,2);
        mustOrCould=1;
        if(mustOrCould==1){ //which of the following MUST be true
            problem=`If the above system has no solutions, which of the following must be true?`;
            let randomB:number=randint(-20,20);
            let randomA:number=randint(-10,10);
            while(randomA==m){
                randomA=randint(-10,10);
            }
            let possibleCorrects:string[]=[];
            //`a = ${m}`, `b ≠ ${b}/${m}`
            if(b%m==0){
                possibleCorrects=[`a = ${m}`, `b ≠ ${b/m}`];
            }else if(b<0&&m>0 || b<0&&m<0){
                possibleCorrects=[`a = ${m}`, `b ≠ ${-b}/${-m}`];
            }else{
                possibleCorrects=[`a = ${m}`, `b ≠ ${b}/${m}`];
            }
            let possibleIncorrects:string[]=[`a ≠ 0`, `a ≠ ${m}`, `b = ${b}`, `b ≠ ${randomB}`, `a = ${randomA}`];
            let howManyCorrects:number=randint(1,2);

            if(howManyCorrects==1){ //1 correct answer
                let i:number=randint2(0,possibleCorrects.length-1);
                unrandomized[0]=possibleCorrects[i];
                corrects.push(possibleCorrects[i]);
                possibleCorrects.splice(i,1);

                i=randint2(0,possibleIncorrects.length-1);
                unrandomized[1]=possibleIncorrects[i];
                possibleIncorrects.splice(i,1);

                i=randint2(0,possibleIncorrects.length-1);
                unrandomized[2]=possibleIncorrects[i];
                possibleIncorrects.splice(i,1);
            }else{
                let i:number=randint2(0,possibleCorrects.length-1);
                unrandomized[0]=possibleCorrects[i];
                corrects.push(possibleCorrects[i]);
                possibleCorrects.splice(i,1);

                i=randint2(0,possibleCorrects.length-1);
                unrandomized[1]=possibleCorrects[i];
                corrects.push(possibleCorrects[i]);
                possibleCorrects.splice(i,1);

                i=randint2(0,possibleIncorrects.length-1);
                unrandomized[2]=possibleIncorrects[i];
                possibleIncorrects.splice(i,1);
            }
        }
    
        possibles.makeVisible(true);
        for(let i=0;i<3;i++){
            let index:number=randint2(0,unrandomized.length-1);
            randomized.push(unrandomized[index]);
            unrandomized.splice(index,1);
        }
        let boolCorrects:boolean[]=[false,false,false];
        for(let i of corrects){
            boolCorrects[randomized.indexOf(i)]=true;
        }

        if(corrects.length==1){ //only 1 correct option
            if(boolCorrects[0]==true){
                let answerChoices:number=randint(1,5);
                if(answerChoices==1){
                    mcqdiv.updateOptions("None", "I only", "I and II", "I and III", "I only");
                }else if(answerChoices==2){
                    mcqdiv.updateOptions("I only", "II only", "II and III", "I and III", "I only");
                }else if(answerChoices==3){
                    mcqdiv.updateOptions("None", "I only", "II only", "I and III", "I only");
                }else if(answerChoices==4){
                    mcqdiv.updateOptions("I only", "I and II", "III only", "II and III", "I only");
                }else if(answerChoices==5){
                    mcqdiv.updateOptions("I only", "I and II", "III only", "I and III", "I only");
                }
            }else if(boolCorrects[1]==true){
                let answerChoices:number=randint(1,2);
                if(answerChoices==1){
                    mcqdiv.updateOptions("I only", "II only", "II and III", "I and III", "II only");
                }else if(answerChoices==2){
                    mcqdiv.updateOptions("None", "I only", "II only", "I and III", "II only");
                }
            }else if(boolCorrects[2]==true){
                let answerChoices:number=randint(1,2);
                if(answerChoices==1){
                    mcqdiv.updateOptions("I only", "I and II", "III only", "II and III", "III only");
                }else if(answerChoices==2){
                    mcqdiv.updateOptions("I only", "I and II", "III only", "I and III", "III only");
                }
            }
        }else{ //2 correct options
            if(boolCorrects[0]==true&&boolCorrects[1]==true){ //I and II
                let answerChoices:number=randint(1,3);
                if(answerChoices==1){
                    mcqdiv.updateOptions("None", "I only", "I and II", "I and III", "I and II");
                }else if(answerChoices==2){
                    mcqdiv.updateOptions("I only", "I and II", "III only", "II and III", "I and II");
                }else{
                    mcqdiv.updateOptions("I only", "I and II", "III only", "I and III", "I and II");
                }
            }else if(boolCorrects[0]==true&&boolCorrects[2]==true){ //I and III
                let answerChoices:number=randint(1,4);
                if(answerChoices==1){
                    mcqdiv.updateOptions("None", "I only", "I and II", "I and III", "I and III");
                }else if(answerChoices==2){
                    mcqdiv.updateOptions("I only", "I and II", "III only", "I and III", "I and III");
                }else if(answerChoices==3){
                    mcqdiv.updateOptions("I only", "II only", "II and III", "I and III", "I and III");
                }else if(answerChoices==4){
                    mcqdiv.updateOptions("None", "I only", "II only", "I and III", "I and III");
                }
            }else if(boolCorrects[1]==true&&boolCorrects[2]==true){ //II and III
                let answerChoices:number=randint(1,2);
                if(answerChoices==1){
                    mcqdiv.updateOptions("I only", "II only", "II and III", "I and III", "II and III");
                }else if(answerChoices==2){
                    mcqdiv.updateOptions("I only", "I and II", "III only", "II and III", "II and III");
                }
            }
        }
        possibles.set(randomized[0],randomized[1],randomized[2]);
    }   

    function typeS():void{ //function g defined, x and y intercept, what is b
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
        let alphabet:string[]=["a","b","c","d","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let eq1:string=`g(x) = `;
        let letter1:string=alphabet[randint2(0,alphabet.length-1)];
        let letter2:string=alphabet[randint2(0,alphabet.length-1)];

        let a:number=0;
        let b:number=0;

        while(letter2==letter1){
            letter2=alphabet[randint2(0,alphabet.length-1)];
        }
        let xIntercept:number=randint(-30,30);
        while(Math.abs(xIntercept)==1){
            xIntercept=randint(-30,30);
        }
        /*
            numerator: x^2 - x - a, x^2 + x - a, x^2 - x + a, or x^2 + x + a
        */
        let numerator:string="";
        let formatChoice1:number=randint(1,4);
        if(formatChoice1==1){ //x^2 - x - a
            numerator=`x² - x - ${letter1}`;
            a=Math.pow(xIntercept,2)-xIntercept;
        }else if(formatChoice1==2){
            numerator=`x² + x + ${letter1}`;
            a=-(Math.pow(xIntercept,2)+xIntercept);
        }else if(formatChoice1==3){
            numerator=`x² + x - ${letter1}`;
            a=Math.pow(xIntercept,2)+xIntercept;
        }else{
            numerator=`x² - x + ${letter1}`;
            a=-(Math.pow(xIntercept,2)-xIntercept);
        }

        let possibleFactors:number[]=[];
        for(let i=-Math.abs(a);i<-Math.ceil(Math.abs(a)/30);i++){
            if(a%i==0 && i!=-1 && a/i!=1){
                possibleFactors.push(i);
            }
        }
        for(let i=Math.ceil(a/30);i<Math.abs(a);i++){
            if(a%i==0 && i!=1 && a/i!=1){
                possibleFactors.push(i);
            }
        }

        b=possibleFactors[randint2(0,possibleFactors.length-1)];

        let yIntercept:number=0;
        let denominator:string="";
        let formatChoice2:number=randint(1,4);
        if(formatChoice2==1){
            denominator=`x³ - x - ${letter2}`;
            if(formatChoice1==1||formatChoice1==3){
                yIntercept=a/b;
            }else{
                yIntercept=-(a/b);
            }
        }else if(formatChoice2==2){
            denominator=`x³ + x - ${letter2}`;
            if(formatChoice1==1||formatChoice1==3){
                yIntercept=a/b;
            }else{
                yIntercept=-(a/b);
            }
        }else if(formatChoice2==3){
            denominator=`x³ - x + ${letter2}`;
            if(formatChoice1==1||formatChoice1==3){
                yIntercept=-(a/b);
            }else{
                yIntercept=a/b;
            }
        }else{
            denominator=`x³ + x + ${letter2}`;
            if(formatChoice1==1||formatChoice1==3){
                yIntercept=-(a/b);
            }else{
                yIntercept=a/b;
            }
        }

        equation1.updateEquation(makeEquationArr(`${eq1}${numerator}/${denominator}[fracfunc]`));
        problem=`The function g is defined by the given equation, where ${letter1} and ${letter2} are constants. In the xy-plane, the graph of y = g(x) passes through the point `;
        let points:string[]=(randint(1,2)==1)?[`(${xIntercept}, 0)`, `g(0) = ${yIntercept}`]:[`(0, ${yIntercept})`, `g(${xIntercept}) = 0`];
        problem+=(`${points[0]} and ${points[1]}. `);
        
        steps.push(`We are told that ${points[0]} and that ${points[1]}.`);
        steps.push(`When ${xIntercept} is plugged into the equation, the resulting value is 0 (g(${xIntercept}) = 0), meaning (${xIntercept}, 0) is the x-intercept.`);
        steps.push(`We can use the x-intercept to find the value of ${letter1}.`);
        steps.push(`We know that the denominator cannot be 0, because that would result in an undefined value. That means that when ${xIntercept} is plugged into the function, the NUMERATOR must end up as 0 (0/anything = 0).`);
        let step:string=`We'll plug ${xIntercept} into just our numerator, giving us the equation `;
        if(formatChoice1==1){
            step+=`(${xIntercept})² ${(xIntercept<0)?`+`:`-`} ${Math.abs(xIntercept)} - ${letter1} = 0.`;
        }else if(formatChoice1==2){
            step+=`(${xIntercept})² ${(xIntercept<0)?`-`:`+`} ${Math.abs(xIntercept)} + ${letter1} = 0.`;
        }else if(formatChoice1==3){
            step+=`(${xIntercept})² ${(xIntercept<0)?`-`:`+`} ${Math.abs(xIntercept)} - ${letter1} = 0.`;
        }else{
            step+=`(${xIntercept})² ${(xIntercept<0)?`+`:`-`} ${Math.abs(xIntercept)} + ${letter1} = 0.`;
        }

        steps.push(step);
        steps.push(`This makes ${letter1} = ${a}.`);
        
        let whatToFind:number=randint(1,2);
        whatToFind=2;
        if(whatToFind==1){
            problem+=`What is the value of ${letter1}?`;
            solutions.push(a);
        }else{
            problem+=`What is the value of ${letter2}?`;
            solutions.push(b);

            steps.push(`But the question asks us for the value of ${letter2}, not ${letter1}.`);
            steps.push(`To find ${letter2}, we can replace ${letter1} in the function with ${a}, since we know it now.`);
            steps.push(`Then we can use our second given point, our y-intercept (0, ${yIntercept}), to find the value of ${letter2}. When x = 0, our value should be ${yIntercept}.`);
            step=`After plugging in 0 for every x, we're left with just `;
            if(formatChoice1==1 || formatChoice1==3){
                if(formatChoice2==1 || formatChoice2==2){
                    if(a<0){
                        step+=`-${Math.abs(a)}/${letter2}`;
                    }else{
                        step+=`${Math.abs(a)}/${letter2}`;
                    }
                }else{
                    if(a<0){
                        step+=`${Math.abs(a)}/${letter2}`;
                    }else{
                        step+=`-${Math.abs(a)}/${letter2}`;
                    }
                }
            }else{
                if(formatChoice2==1 || formatChoice2==2){
                    if(a<0){
                        step+=`${Math.abs(a)}/${letter2}`;
                    }else{
                        step+=`-${Math.abs(a)}/${letter2}`;
                    }
                }else{
                    if(a<0){
                        step+=`-${Math.abs(a)}/${letter2}`;
                    }else{
                        step+=`${Math.abs(a)}/${letter2}`;
                    }
                }
            }
            steps.push(`${step} = ${yIntercept}.`);
            steps.push(`Solving this, we get ${letter2} = ${b}.`)
        }
    }

    function typeT():void{ //vertex given, what could be a + b + c OR js c?
        mcqdiv.makeVisible(true);
        openResponse.makeVisible(false);
        let h:number=randint(-10,10);
        let k:number=randint(-15,15);
        problem=`In the xy-plane, a parabola has vertex (${h}, ${k}) and `;
        steps.push(`According to the question, the vertex is (${h}, ${k}). This means that the y-intercept of the vertex = ${k}, which is ${(k<0)?`below`:`above`} the x-axis.`);

        let intersects:number=randint(1,2);
        if(intersects==1){
            problem+=`intersects the x-axis at two points. If the equation is written in the form y = ax² + bx + c, where a, b, and c are constants, which of the following could be the value of a + b + c?`;
            let unrandomized:number[]=[];
            let randomized:number[]=[];
            unrandomized[0]=k;
            if(k>0){ //k is positive
                unrandomized[1]=k-randint(2,10);
                unrandomized[2]=k+randint(2,10);
                unrandomized[3]=k+randint(2,10);
                while(unrandomized[3]==unrandomized[2]){
                    unrandomized[3]=k+randint(2,10);
                }
            }else{
                unrandomized[1]=k+randint(2,10);
                unrandomized[2]=k-randint(2,10);
                unrandomized[3]=k-randint(2,10);
                while(unrandomized[3]==unrandomized[2]){
                    unrandomized[3]=k-randint(2,10);
                }
            }
            randomized=createRandomAnswers(unrandomized);
            mcqdiv.updateOptions(randomized[0].toString(),randomized[1].toString(),randomized[2].toString(),randomized[3].toString(),unrandomized[1].toString());

            steps.push(`We are told also that the graph crosses the x-axis at two points.`);
            steps.push(`This means two things: one, that a has to be ${(k<0)?`positive`:`negative`} so that the graph can actually cross the x-axis, and two, that c-- our y-intercept-- must be ${(k<0)?`greater than`:`less than`} ${k} (since it literally can't be ${(k<0)?`less`:`greater`}).`);
            steps.push(`Out of the four option choices, there is only one that is ${(k<0)?`greater than`:`less than`} ${k}, which is ${unrandomized[1].toString()}.`);
        }else{
            problem+=`does not intersect the x-axis. If the equation is written in the form y = ax² + bx + c, where a, b, and c are constants, which of the following could be the value of a + b + c?`;
            let unrandomized:number[]=[];
            let randomized:number[]=[];
            unrandomized[0]=k;
            if(k>0){ //k is positive
                unrandomized[1]=k+randint(2,10);
                unrandomized[2]=k-randint(2,10);
                unrandomized[3]=k-randint(2,10);
                while(unrandomized[3]==unrandomized[2]){
                    unrandomized[3]=k-randint(2,10);
                }
            }else{
                unrandomized[1]=k-randint(2,10);
                unrandomized[2]=k+randint(2,10);
                unrandomized[3]=k+randint(2,10);
                while(unrandomized[3]==unrandomized[2]){
                    unrandomized[3]=k+randint(2,10);
                }
            }
            randomized=createRandomAnswers(unrandomized);
            mcqdiv.updateOptions(randomized[0].toString(),randomized[1].toString(),randomized[2].toString(),randomized[3].toString(),unrandomized[1].toString());
        
            steps.push(`We are told also that the graph does not cross the x-axis.`);
            steps.push(`This means two things: one, that a has to be ${(k<0)?`negative`:`positive`} so that the graph does not cross the x-axis, and two, that c-- our y intercept-- must be ${(k<0)?`less than`:`greater than`} ${k} (since it literally can't be ${(k<0)?`greater`:`less`}).`);
            steps.push(`Out of the four option choices, there is only one that is ${(k<0)?`less than`:`greater than`} ${k}, which is ${unrandomized[1].toString()}.`);
        }
    }

    function typeU():void{ //data set consists of integers...what is the value of the largest..
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);
        let number=randint(10,15);
        problem=`Data set A consists of ${number} integers less than `;
        let averageWithout:number=randint(20,50);
        let dataSet:number[]=[];
        let balance=0;

        for(let i=0;i<(number-1);i++){
            let increment:number=randint(-7,7);
            dataSet.push(averageWithout+increment);
            balance+=increment;
        }
        
        if(balance>0){
            while(balance!=0){
                let index:number=randint2(0,number-2);
                dataSet[index]=dataSet[index]-(balance-Math.floor(balance/2));
                balance=Math.floor(balance/2);
            }
        }else if(balance<0){
            balance=Math.abs(balance);
            while(balance!=0){
                let index:number=randint2(0,number-2);
                dataSet[index]=dataSet[index]+(balance-Math.floor(balance/2));
                balance=Math.floor(balance/2);
            }
        }

        /*
        mean of full data set must be greater than meanWithout
        */

        let baseDividend:number=averageWithout*(number-1)
        let add:number=averageWithout+1;
        while((baseDividend+add)%number!=0){
            add++;
        }

        solutions.push(add);
        let eq1:string="";
        for(let i of dataSet){
            eq1+=`${i},`;
        }
        eq1=eq1.substring(0,eq1.length-2);
        equation1.updateEquation(makeEquationArr(eq1));
        problem+=`${(Math.floor(add/10)+1)*10}. The list above shows ${number-1} of the integers from data set A. The mean of these ${number-1} integers is ${averageWithout}. If the mean of data set A is an integer greater than ${averageWithout}, what is the value of the largest integer from data set A?`;

        steps.push(`We are given ${number-1} integers part of Data Set A. We know that the mean of these ${number-1} integers is ${averageWithout}, so it's safe to assume that (${number-1}*${averageWithout})/${number-1}, or ${(number-1)*averageWithout}/${number-1}, is another way to represent this average.`);
        steps.push(`However, Data Set A has ${number} values, not ${number-1}; one value is excluded from the set above.`);
        steps.push(`If we factor that unknown value in, using x to represent it, we'll get our average as (${(number-1)*averageWithout} + x)/${number}.`);
        steps.push(`All ${number} values in Data Set A are less than ${(Math.floor(add/10)+1)*10}, and the mean of Data Set A as a whole is greater than ${(number-1)*averageWithout}. So we can set this up as an inequality: 24 < (${(number-1)*averageWithout} + x)/${number} < ${(Math.floor(add/10)+1)*10}.`);
        steps.push(`We know that Data Set A consists of only integers, so we know that the value not included above has to be an integer, too. We are additionally told that the mean is also an integer. So both x and (${(number-1)*averageWithout} + x)/${number} must end up as integers.`);
        steps.push(`Now we find the value of x that is an integer, yields an integer, and fits the inequality.`);
        steps.push(`x = ${add}. (${(number-1)*averageWithout} + ${add})/${number} = ${((number-1)*averageWithout+add)/number}. ${((number-1)*averageWithout+add)/number} is greater than ${((number-1)*averageWithout)/(number-1)} and less than ${(Math.floor(add/10)+1)*10}.`);
        steps.push(`Since the mean of Data Set A is lower when x is not present, x must be the largest integer in the data set, as the mean will increase when it is added.`);
        steps.push(`The largest integer in Data Set A is ${add}.`);
    }

    function typeV():void{ //xy plane, there are these points...what angle
        openResponse.makeVisible(false);
        mcqdiv.makeVisible(true);

        let alphabet:string[]=["a","b","c","d","f","g","h","j","k","m","n","p","q","r","u","v","w","z"];
        let startIndex:number=randint2(0,alphabet.length-3);
        problem=`Point ${alphabet[startIndex].toUpperCase()} lies on the xy-plane and has coordinates `;

        let unrandomized:string[]=[];
        let randomized:string[]=[];
        let aCoords:string="";
        let cCoords:string="";

        let scale:number=randint(15,40)*2;
        let denominator:number=randint(2,5)*2;
        unrandomized[0]=`${scale*denominator}π/${denominator}`;
        unrandomized[1]=`${scale*denominator+(denominator/2)}π/${denominator}`;
        unrandomized[2]=`${(scale+1)*denominator}π/${denominator}`;
        unrandomized[3]=`${scale*denominator+((denominator/2)*3)}π/${denominator}`;
        randomized=createRandomAnswers(unrandomized);

        let possibleCoords:string[]=["(1,0)","(0,1)","(-1,0)","(0,-1)"];
        let coordUnknowns:number=randint(1,2);
        coordUnknowns=1;

        steps.push(`Let's first find out what our angle should be.`);

        if(coordUnknowns==1){ //no coordinate unknown
            aCoords=possibleCoords[randint2(0,3)];
            cCoords=possibleCoords[randint2(0,3)];
            while(cCoords==aCoords){
                cCoords=possibleCoords[randint2(0,3)];
            }

            problem+=`${aCoords}. Point ${alphabet[startIndex+1].toUpperCase()} has coordinates (0,0), and point ${alphabet[startIndex+2].toUpperCase()} has coordinates ${cCoords}. `;
            steps.push(`We are given one coordinate as ${aCoords}, one as (0,0), and one as ${cCoords}.`);
            
            if(([aCoords,cCoords].includes("(1,0)")&&[aCoords,cCoords].includes("(0,1)")) || ([aCoords,cCoords].includes("(0,1)")&&[aCoords,cCoords].includes("(-1,0)")) || ([aCoords,cCoords].includes("(-1,0)")&&[aCoords,cCoords].includes("(0,-1)")) || ([aCoords,cCoords].includes("(0,-1)")&&[aCoords,cCoords].includes("(1,0)"))){
                mcqdiv.updateOptions(randomized[0],randomized[1],randomized[2],randomized[3],unrandomized[1]);
                steps.push(`If we sketch this out on a coordinate plane, one point would lie on the x-axis, and one would lie on the y-axis.`);
                steps.push(`That would mean that the two segments form a 90° angle.`);
                steps.push(`To convert from degrees to radians, we multiply our angle in degrees by (π/180).`);
                steps.push(`90*(π/180) = π/2 radians.`);
                steps.push(`The four answer options may look scary at first, but try simplifying them, and you'll see they end with a denominator of either 1 or 2.`);
                steps.push(`This means that the four options are coterminal-- they share the same angle measure after one or more full rotations-- with the four quadrantals (90°, 180°, 270°, and 360°).`);
                steps.push(`Let's find out which of the four options is coterminal with π/2.`);
                steps.push(`Immediately, we can eliminate ${unrandomized[0]} and ${unrandomized[2]}, since they have 1 as their denominator, not 2; that means they're either π (180°) or 2π (0°/360°).`);
                steps.push(`Now, we have to find what angles ${unrandomized[1]} and ${unrandomized[3]} are coterminal with.`);
                steps.push(`To do this, we can simplify ${unrandomized[1]} as ${(scale*denominator+(denominator/2))/(denominator/2)}π/2 and ${unrandomized[3]} as ${(scale*denominator+((denominator/2)*3))/(denominator/2)}π/2.`);
                steps.push(`Remember that a coterminal angle means that one or more full rotations have passed. So if we subtract enough full rotations from ${(scale*denominator+(denominator/2))/(denominator/2)}π/2 and ${(scale*denominator+((denominator/2)*3))/(denominator/2)}π/2, we should eventually get π/2 for one and 3π/2 for one.`);
                steps.push(`Instead of subtracting two over and over, we'll just find the number closest to ${(scale*denominator+(denominator/2))/(denominator/2)} and ${(scale*denominator+((denominator/2)*3))/(denominator/2)} that yields an even number when divided by two (as if it yields an odd number, it would be coterminal with a half rotation and not a full rotation).`);
                steps.push(`In this case, we'll subtract both angles by ${((scale*denominator+(denominator/2))/(denominator/2))-1}π/2.`);
                steps.push(`${(scale*denominator+(denominator/2))/(denominator/2)}π/2 (unsimplified: ${unrandomized[1]}) is coterminal with π/2.`);
            }else if(([aCoords,cCoords].includes("(1,0)")&&[aCoords,cCoords].includes("(-1,0)")) || ([aCoords,cCoords].includes("(0,1)")&&[aCoords,cCoords].includes("(0,-1)"))){
                mcqdiv.updateOptions(randomized[0],randomized[1],randomized[2],randomized[3],unrandomized[2]);
                steps.push(`If we sketch this out on a coordinate plane, both points would lie on the ${([aCoords,cCoords].includes("(1,0)")&&[aCoords,cCoords].includes("(-1,0)"))?`x`:`y`}-axis.`);
                steps.push(`That would mean that the two segments form a 180° angle.`);
                steps.push(`To convert from degrees to radians, we multiply our angle in degrees by (π/180).`);
                steps.push(`180*(π/180) = π radians.`);
                steps.push(`The four answer options may look scary at first, but try simplifying them, and you'll see they end with a denominator of either 1 or 2.`);
                steps.push(`This means that the four options are coterminal-- they share the same angle measure after one or more full rotations-- with four different quadrantals (90° * n).`);
                steps.push(`Let's find out which of the four options is coterminal with π.`);
                steps.push(`Immediately, we can eliminate ${unrandomized[1]} and ${unrandomized[3]}, since they have 2 as their denominator, not 1; that means they're either π/2 (90°) or 3π/2 (270°).`);
                steps.push(`From there, figuring out whether the answer is ${unrandomized[0]} or ${unrandomized[2]} is easy. Simplify both answers. If the angle is coterminal with 2π, it will be even; if it is coterminal with π, it will be odd.`);
                steps.push(`${(scale+1)*denominator}/${denominator} = ${scale+1}. The correct option is ${unrandomized[2]}.`);
                
            }
        }else if(coordUnknowns==2){ //x and y are unknown, but we know their signs (pos/neg)
            let index:number=randint2(0,3);
            let signs:number=randint(1,2);
            let aCoords:string=possibleCoords[index];
            problem+=`${aCoords}. Point ${alphabet[startIndex+1].toUpperCase()} has coordinates (0,0). Point ${alphabet[startIndex+2].toUpperCase()} has coordinates `;
            if(index==0){ //aCoords = (1,0)
                if(signs==1){ //(0,y) where y is a positive constant
                    problem+=`(0,y), where y is a positive constant. `;
                }else if(signs==2){ //(x, 0) where x is a negative constant
                    problem+=`(x,0), where x is a negative constant. `;
                }
            }else if(index==1){ //aCoords = (0,1)
                if(signs==1){
                    problem+=`(x,0), where x is a negative constant. `;
                }else if(signs==2){
                    problem+=`(0,y), where y is a negative constant. `;
                }
            }else if(index==2){ // aCoords = (0,-1)
                if(signs==1){
                    problem+=`(0,y), where y is a negative constant. `;
                }else if(signs==2){
                    problem+=`(x,0), where x is a positive constant. `;
                }
            }else{
                if(signs==1){
                    problem+=`(x,0), where x is a positive constant. `;
                }else if(signs==2){
                    problem+=`(0,y), where y is a positive constant. `;
                }
            }

            if(signs==1){
                mcqdiv.updateOptions(randomized[0],randomized[1],randomized[2],randomized[3],unrandomized[1]);
            }else{
                mcqdiv.updateOptions(randomized[0],randomized[1],randomized[2],randomized[3],unrandomized[2]);
            }
        }
        problem+=`Which of the following could be the smallest measure of angle ∠${alphabet[startIndex].toUpperCase()}${alphabet[startIndex+1].toUpperCase()}${alphabet[startIndex+2].toUpperCase()}?`;
    }

    function typeW():void{ //pyramid with square base, find surface area of triangle face
        openResponse.makeVisible(true);
        mcqdiv.makeVisible(false);
        let triples:number[][]=[[3,4,5],[5,12,13],[7,24,25],[8,15,17],[9,40,41],[11,60,61],[12,35,37],[20,21,29],[28,45,53]];
        let chosenTriple:number[]=triples[randint2(0,triples.length-1)]; //[side1, side2, hypotenuse]
        let choice:number=randint2(0,1);
        let s1:number=(choice==0)?chosenTriple[0]:chosenTriple[1];
        let s2:number=(choice==0)?chosenTriple[1]:chosenTriple[0];
        problem=`A rectangular pyramid has a square base with an area of ${Math.pow(s1*2,2)} square units. What is the surface area, in square units, of `;

        steps.push(`Firstly, since area of a square = a², we can determine that the edge length of the base = ${s1*2}.`);
        steps.push(`We are given the volume of the rectangular pyramid, ${(Math.pow(s1*2,2)*s2)/3} units³.`);
        steps.push(`The volume of a pyramid is given by V = 1/3(lwh), where l and w are the length and width of the base and h is the height of the pyramid.`);
        steps.push(`Since we know our length and width are ${s1*2} (area of the base ${Math.pow(s1*2,2)}), we can find the height of our pyramid by solving ${(Math.pow(s1*2,2)*s2)/3} = 1/3(${Math.pow(s1*2,2)})h.`);
        steps.push(`The height of our triangle = ${s2}.`);
        choice=randint(1,2);
        if(choice==1){ //find surface area of js triangular side
            problem+=`one of the triangular faces `;
            solutions.push(chosenTriple[2]*s1);
        }else if(choice==2){
            problem+=`the rectangular pyramid `;
            solutions.push(((chosenTriple[2]*s1)*4)+Math.pow(s1*2,2));
        }
        problem+=`if the rectangular pyramid has a volume of ${(Math.pow(s1*2,2)*s2)/3} cubic units?`;
        
        steps.push(`But that alone isn't enough to find the surface area of ${(choice==1)?`a triangular face!`:`the pyramid!`}`);
        steps.push(`If we want to find the surface area of one triangular face, we actually need to get the slant height, aka the height of that individual face; we only have the height of the 3D pyramid right now.`);
        steps.push(`To get the slant height, we can think of it as the hypotenuse of a right triangle, where the vertical leg is the height of the pyramid and the horizontal leg extends from where the height touches the base to the bottom edge of the triangular face. So one leg would be ${s2} and the other would be half of the base's edge length-- ${s1*2}/2, or ${s1}.`);
        steps.push(`From there, we can use pythagorean theorem to find the slant height.`);
        steps.push(`√(${s1}² + ${s2}²) = √(${Math.pow(chosenTriple[2],2)}) = ${chosenTriple[2]}. The slant height is ${chosenTriple[2]}.`);
        steps.push(`Now that we have the actual height of a triangular face of this pyramid, we can calculate the area of the triangular face with 1/2(bh).`);
        steps.push(`We'll multiply the edge length of the square (${s1*2}) by the height, ${chosenTriple[2]}, to get ${(s1*2)*chosenTriple[2]}; then, we divide it by 2 to get the area of a triangular face: ${((s1*2)*chosenTriple[2])/2}.`);
        if(choice==2){
            steps.push(`There are four triangular faces in a pyramid, so we'll multiply ${((s1*2)*chosenTriple[2])/2} * 4 to get the combined surface area, ${(((s1*2)*chosenTriple[2])/2)*4}.`);
            steps.push(`Finally, we add the surface area of our base, ${Math.pow(s1*2,2)} to this to get our total surface area: ${solutions[0]}.`);
        }
    }

    function typeX():void{ //what is the value of k or m?
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);

        let x1:number=randint(-15,15);
        let y1:number=randint(-15,15);
        let c1:number=randint(-50,50);

        let x2:number=randint(-15,15);
        let y2:number=randint(-15,15);
        let c2:number=randint(-50,50);

        while((y2/(x1/x2)==y1) || (y1/(x2/x1)==y2)){
            x2=randint(-15,15);
            y2=randint(-15,15);
            c2=randint(-50,50);
        }

        

        let eq1:string=`${x1}ax `;
        eq1+=(y1<0)?`- ${-y1}by = ${c1}`:`+ ${y1}by = ${c1}`
        equation1.updateEquation(makeEquationArr(eq1));

        let eq2:string=`${x2}ax `;
        eq2+=(y2<0)?`- ${-y2}by = ${c2}`:`+ ${y2}by = ${c2}`
        equation2.updateEquation(makeEquationArr(eq2));

        problem=`In the given system of equations, a and b are constants. The system has a solution of `;

        let aOrB:number=randint(1,2);
        aOrB=1;
        if(aOrB==1){ //user finds a
            let a:number=randint(-15,15);
            while(a==-1 || a==1){
                a=randint(-15,15);
            }
            solutions.push(a);

            steps.push(`We can see that we are given the x-value of a solution.`);
            steps.push(`We should start by using the elimination method to eliminate y and solve for only x.`);
            if(y1%y2==0){
                let step:string=`We can multiply the entire second equation by `;
                if((y1>0&&y2>0) || (y1<0&&y2<0)){
                    step+=`-${y1/y2}.`;
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n${x1}ax ${(y1<0)?`- ${Math.abs(y1)}`:`+ ${y1}`}by = ${c1}\n${x2*(-y1/y2)}ax ${(-y2<0)?`- ${Math.abs(y1)}`:`+ ${Math.abs(y1)}`}by = ${c2*(-y1/y2)}`);
                }else if(y1>0&&y2<0 || y1<0&&y2>0){
                    step+=`${y1/y2}.`;
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n
                        ${x1}ax ${(y1<0)?`- ${Math.abs(y1)}`:`+ ${y1}`}by = ${c1}\n
                        ${x2*(y1/y2)}ax ${(y2<0)?`- ${Math.abs(y1)}`:`+ ${y1}`}by = ${c2*(y1/y2)}
                    `);
                } 
            }else if(y2%y1==0){
                let step:string=`We can multiply the entire first equation by `;
                if((y1>0&&y2>0) || (y1<0&&y2<0)){
                    step+=`-${y2/y1}.`;
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n${x1*(-y2/y1)}ax ${(-y1<0)?`- ${Math.abs(y2)}`:`+ ${Math.abs(y2)}`}by = ${c1*(-y2/y1)}\n${x2}ax ${(y2<0)?`- ${Math.abs(y2)}`:`+ ${Math.abs(y2)}`}by = ${c2}`);
                }else if(y1>0&&y2<0 || y1<0&&y2>0){
                    step+=`${y2/y1}.`;
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n
                        ${x1*(y2/y1)}ax ${(y1<0)?`- ${Math.abs(y2)}`:`+ ${y2}`}by = ${c1*(y2/y1)}\n
                        ${x2}ax ${(y2<0)?`- ${Math.abs(y2)}`:`+ ${y2}`}by = ${c2}
                    `);
                } 
            }else if(y1>y2){ //multiply first equation by y2/y1
                let step:string=`We can multiply the entire first equation by `;
                if((y1>0&&y2>0)||(y1<0&&y2<0)){
                    step+=`-${y2}/${y1}.`
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n
                        ${-x1*Math.abs(y2)}/${Math.abs(y1)}ax ${(y1<0)?`+ ${Math.abs(y2)}`:`- ${Math.abs(y2)}`}by = ${-c1*Math.abs(y2)}/${Math.abs(y1)}\n
                        ${x2}ax ${(y2<0)?`- ${Math.abs(y2)}`:`+ ${y2}`}by = ${c2}
                    `);
                }else if((y1<0&&y2>0)||(y1>0&&y2<0)){
                    step+=`${y2}/${y1}.`;
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n
                        ${x1*Math.abs(y2)}/${Math.abs(y1)}ax ${(y1<0)?`- ${Math.abs(y2)}`:`+ ${Math.abs(y2)}`}by = ${c1*Math.abs(y2)}/${Math.abs(y1)}\n
                        ${x2}ax ${(y2<0)?`- ${Math.abs(y2)}`:`+ ${y2}`}by = ${c2}
                    `);
                }
            }else if(y2>y1){ //multiply second equation by y1/y2
                let step:string=`We can multiply the entire second equation by `;
                if((y1>0&&y2>0)||(y1<0&&y2<0)){
                    step+=`-${Math.abs(y1)}/${Math.abs(y2)}`;
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n
                        ${x1}ax ${(y1<0)?`- ${Math.abs(y1)}`:`+ ${Math.abs(y1)}`}by = ${c1}\n
                        ${-x2*Math.abs(y1)}/${Math.abs(y2)}ax ${(y2<0)?`+ ${Math.abs(y1)}`:`- ${Math.abs(y1)}`}by = ${-c2*Math.abs(y1)}/${Math.abs(y2)}
                    `);
                }else if((y1<0&&y2>0)||(y1>0&&y2<0)){
                    step+=`${Math.abs(y1)}/${Math.abs(y2)}`;
                    steps.push(step);
                    steps.push(`We'll get our system as follows:\n
                        ${x1}ax ${(y1<0)?`- ${Math.abs(y1)}`:`+ ${Math.abs(y1)}`}by = ${c1}\n
                        ${x2*Math.abs(y1)}/${Math.abs(y2)}ax ${(y2<0)?`- ${Math.abs(y1)}`:`+ ${Math.abs(y1)}`}by = ${c2*Math.abs(y1)}/${Math.abs(y2)}
                    `);
                }
            }

            if((y1>0&&y2>0)||(y1<0&&y2<0)){
                x1*=y2;
                c1*=y2;
                x2*=-y1;
                c2*=-y1;
            }else if((y1>0&&y2<0)||(y1<0&&y2>0)){
                x1*=Math.abs(y2);
                c1*=Math.abs(y2);
                x2*=Math.abs(y1);
                c2*=Math.abs(y1);
            }

            let axNumerator:number=(c1+c2);
            let axDenominator:number=(x1+x2)*a;
            let axFracsReduced:number[]=reduceFraction(axNumerator,axDenominator);
            if((axFracsReduced[0]<0 && axFracsReduced[1]<0) || (axFracsReduced[0]>0 && axFracsReduced[1]<0)){
                problem+=`(${-axFracsReduced[0]}/${-axFracsReduced[1]}, y). What is the value of a?`;
            }else{
                problem+=`(${axFracsReduced[0]}/${axFracsReduced[1]}, y). What is the value of a?`;
            }
        }else{ //user finds b
            let b:number=randint(-15,15);
            solutions.push(b);
            console.log(b);
            if((x1>0&&x2>0)||(x1<0&&x2<0)){
                y1*=x2;
                c1*=x2;
                y2*=-x1;
                c2*=-x1;
            }else if((x1>0&&x2<0)||(x1<0&&x2>0)){
                y1*=Math.abs(x2);
                c1*=Math.abs(x2);
                y2*=Math.abs(x1);
                c2*=Math.abs(x1);
            }

            let byNumerator:number=(c1+c2);
            let byDenominator:number=(y1+y2)*b;
            let byFracsReduced:number[]=reduceFraction(byNumerator,byDenominator);
            if((byFracsReduced[0]<0 && byFracsReduced[1]<0) || (byFracsReduced[0]>0 && byFracsReduced[1]<0)){
                problem+=`(x, ${-byFracsReduced[0]}/${-byFracsReduced[1]}). What is the value of b?`;
            }else{
                problem+=`(x, ${byFracsReduced[0]}/${byFracsReduced[1]}). What is the value of b?`;
            }
        }
    }

    function typeY():void{
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);
        
        imageVisible=true;
        imgBind=typeyImg;
        problem=`In the given figure, BC is the diameter of the circle. If the length of `;
        let given1:number=randint(1,2);
        if(given1==1){ //bc given
            let bc:number=randint(7,50)*3;
            let factors:number[][]=getFactors(Math.floor(Math.pow(bc,2)/3));
            let chosenScale:number=factors[randint2(1,factors.length-2)][0]; //this is in sqrt
            problem+=`BC is equal to ${bc} and the length of AB is equal to `;
            problem+=(Math.sqrt(Math.pow(bc,2)/chosenScale).toString().includes("."))?`√${Math.pow(bc,2)/chosenScale}, `:`${Math.sqrt(Math.pow(bc,2)/chosenScale)}, `;
            problem+=`what is the value of `;
            let whatToFind:number=randint(1,2);
            //whatToFind=2;
            if(whatToFind==1){ //find ad...?
                problem+=`AD?`;
                let ca:number=Math.pow(bc,2)-(Math.pow(bc,2)/chosenScale);
                solutions.push(Math.sqrt(ca/chosenScale));
            }else if(whatToFind==2){ //find bd
                problem+=`BD?`;
                let bd:number=(Math.pow(bc,2)/chosenScale)/chosenScale;
                solutions.push(Math.sqrt(bd));
            }
        }else if(given1==2){ //bd given
            let ab:number=randint(7,50)*3;
            let factors:number[][]=getFactors(Math.floor(Math.pow(ab,2)/3));
            let chosenScale:number=factors[randint2(2,factors.length-4)][0]; //this is in sqrt
            let bd:number=(Math.pow(ab,2)/chosenScale);
            ab=Math.pow(ab,2);
            problem+=`BD is equal to `;
            problem+=(Math.sqrt(bd).toString().includes("."))?`√${bd} and the length of AB is equal to `:`${Math.sqrt(bd)} and the length of AB is equal to `
            problem+=(Math.sqrt(ab).toString().includes("."))?`√${ab}`:`${Math.sqrt(ab)}`;
            problem+=`, what is the value of `;
            let whatToFind:number=randint(1,3);
            //whatToFind=3;
            if(whatToFind==1){ //find BC
                problem+=`BC, to the nearest whole number?`;
                solutions.push(Math.round(Math.sqrt(ab*chosenScale)));
            }else if(whatToFind==2){ //find CD
                problem+=`CD, to the nearest whole number?`;
                let ad:number=Math.sqrt(ab-bd);
                let bigSmallScale:number=ad/Math.sqrt(bd);
                console.log(Math.round(ad*bigSmallScale));
                solutions.push(Math.round(ad*bigSmallScale));
            }else if(whatToFind==3){ //find AC
                problem+=`AC, to the nearest whole number?`;
                let ad:number=Math.sqrt(ab-bd);
                solutions.push(Math.round(ad*Math.sqrt(chosenScale)));
            }
        }
    }

    function typeZ(){ //one x intercept + vertex or points to find vertex given; what is other x intercept
        mcqdiv.makeVisible(false);
        openResponse.makeVisible(true);
        problem=`The graph of the quadratic function y = f(x) in the xy-plane intersects the x intercept at `;
        let xintercept1:number=randint(-25,25);
        let xintercept2:number=randint(-25,25);
        solutions.push(xintercept2);
        problem+=`(${xintercept1},0) and (k,0), where k is a constant. `;

        let b:number=xintercept1+xintercept2;
        let c:number=xintercept1*xintercept2;
        let h:number=(-b/2);

        let given:number=randint(1,2);
        given=2;
        if(given==1){ //vertex directly given
            problem+=`The maximum value of f(x) occurs at the point (${h},m), where m is a constant. What is the value of k?`;
        }else if(given==2){ //symmetrical points given
            let step:number=randint(5,20);
            problem+=`If f(${h+step}) = f(${h-step}), what is the value of k?`;
        }
    }
</script>

<div class="w-[100vw] h-[100vh] overflow-auto flex justify-around">
    <div class="bg-blue-200 overflow-auto w-[100%]">
        <h1 class="text-center text-[30px]">math.SAT</h1>
        <div class="bg-blue-300 w-[95%] m-auto">
            <div class="flex justify-around w-[100%] box-border p-[10px] m-auto">
                <div class="bg-blue-100 w-[49%] box-border p-[10px]">
                    <h4>Problem {type}</h4>
                    <div style={(imageVisible)?`display:block`:`display:none`}>
                        <img src={imgBind} alt="model of a triangle ABC, where A is a right angle. Point D lies on line BA and point E lies on line AC such that line DE is perpendicular to line BA.">
                        <p>Image is not to scale.</p>
                    </div>
                    <br>
                    <div style={eq1Visible?`display:block`:`display:none`}>
                        <Equation bind:this={equation1}></Equation>
                        <br>
                    </div>
                    <div style={eq2Visible?`display:block`:`display:none`}>
                        <Equation bind:this={equation2}></Equation>
                        <br>
                    </div>
                    <h3>{problem}</h3>
                    <Possibles bind:this={possibles}></Possibles>
                </div>
                <div class="bg-blue-100 w-[49%] box-border p-[10px]">
                    <McqDiv equationConversion={makeEquationArr} bind:this={mcqdiv}></McqDiv>
                    <OpenResponse bind:this={openResponse}></OpenResponse>
                    <br>
                    <button class="bg-[#EBF4FF] border-blue-300 border-[2px] w-[70%] m-auto" style={checkAnswerVisible?`display:block`:`display:none`} onclick={submitAnswer}>check answer</button>
                    <button class="bg-[#EBF4FF] border-blue-300 border-[2px] w-[70%] m-auto"  onclick={makeQuestion} style={makeQuestionVisible?`display:block`:`display:none`}>next question &gt;&gt;&gt;</button>
                </div>
            </div>
            <div style={(feedback=="")?`display:none`:`display:block`} class="pb-[10px]">
                <h1 class="text-center">{feedback}</h1>
            </div>
        </div>
        <!--<button class="bg-[#EBF4FF]">Settings</button>-->
        <br>
        <div class="flex justify-around w-[90%] h-[40px] m-auto">
            <button class="w-[49%] bg-orange-200" onclick={function(){
                if(stepsIndex!=steps.length){
                    displayedSteps.push(steps[stepsIndex]);
                    stepsIndex+=1;
                }
            }}>Help!</button>
            <button class={`${(desmosVisible)?`bg-red-300`:`bg-green-300`} w-[49%]`} onclick={function(){
                desmosVisible=!desmosVisible;
            }}>{(desmosVisible)?`close desmos`:`open desmos`}</button>
        </div>
        <br>
        <div class={`w-[70%] max-h-[40vh] overflow-auto box-border p-[10px] border-[2px] border-white scrollbar ${(displayedSteps.length==0)?`hidden`:`block`}`}>
            {#each displayedSteps as step,i}
                <h3 class="whitespace-pre-line">{i+1}. {step}</h3>
            {/each}
        </div>
    </div>
    <div style={desmosVisible?`display:flex`:`display:none`} class="justify-around w-[100%]">
        <iframe title="desmos" src="https://www.desmos.com/testing/collegeboard/graphing" class="w-[100%]"></iframe>
    </div>
</div>

<style>
    .scrollbar{
        scrollbar-width:thin;
        scrollbar-color: rgb(137, 182, 255) rgb(182, 211, 248);
    }
</style>