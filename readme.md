Want to practice the hard math questions that could appear on the SAT, but you've already memorized all the questions in the question bank? Tired of sloppy AI generating you sloppy practice problems that don't even have a valid answer? Hoping to familiarize yourself with question formats that appear on the actual test? Then this is for you!

![An example of a problem type on math.SAT.](/readme-images/j.png)

## math.SAT
math.SAT is a huge SAT Math practice app I created from scratch. It holds 24-26 different kinds of questions (as some are currently unfinished, while others may be removed/replaced) modeled after real hard-level SAT math questions that appear on the question bank and actual SATs.

These are not preloaded questions! Instead, using examples as bases, I worked my way through the foundation of and developed an algorithm for each individual question. As a result, I've developed an app that uses completely random numbers to customize values, formatting, and answer choices for every question included in this practice app. No AI involved!! Just pure math.

![An example of a problem type on math.SAT](/readme-images/l.png)
![An example of a problem type on math.SAT](/readme-images/o.png)

Similar to the actual SAT, problems can be either multiple choice or open response. Open responses follow the same rounding and answer format as the actual SAT. Also, just like Bluebook, Desmos is embedded!

![desmos on math.sat](/readme-images/desmos.png)

One additional feature of math.SAT is the "help" button right next to the "open desmos" button. When clicked, the help button gives the user a series of steps to solve the question; one step is revealed with every click of the button.

![example of the help button being used](/readme-images/help.png)

These steps, too, are not AI-generated; instead, I manually created a base outline of steps for each question and substituted any mention of numbers, measurements, or anything variable between different generations of the same question with variables and math from the code. These are then updated using two different string arrays.

### How do I use it?
Just solve the questions-- that's it! If you get it right, you'll get a continue button where your submit button once was; if you get it wrong, you'll be able to try again infinitely, or use the "help" button for help.

![problem after solving correctly](/readme-images/u.png)

### Running locally
Open your terminal and navigate to where you'd like to save the repo.

```git clone "https://github.com/ChuckChuckler/SAT-Math-Practice-App"```

```cd SAT-Math-Practice-App```

```cd sat-math-app```

```npm install```

Wait for installation, then run ```npm run dev``` and navigate to the given localhost!