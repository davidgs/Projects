# Q&A The one where you're Retrieving Process Variables

You know there's a process variable in your model, but you're not sure what, or where, it is. So, how do you go about retrieving it? Turns out, we get this qquestion a lot. And there are a few ways of getting at the variable -- if it even exists.

## Does the process variable even exist?

That's really the first question to ask, isn't it? Because if you attempt to retrieve, and then use, a process variable that doesn't exist in the first place you'll have a whole host of other problems that are worse than you started with.

We previously covered how to handle a [non-existent variable](https://camunda.com/blog/2021/11/qa-the-one-with-the-non-existent-variable/) so we won't cover that again, but I encourage you to go back and read that post so you know how to solve that part of the problem as well.

## So the variable exists, but how do we get at it?

As I said in the beginning, there are several ways to go about this. First, you can get _all_ the variables in the process and then go through them all looking for the one you want:

```js
const { Variables } = require("camunda-external-task-client-js");

client.subscribe("topicName", async function({ task, taskService }) {
// Given:
// {
//    score: { value: 5, type: "integer", valueInfo: {} }
//    isWinning: { value: false, type: "boolean", valueInfo: {} }
// }
const values = task.variables.getAll();
console.log(values);
```

Will print out:

```json
{  score: 5, isWinning: false }
```

That will get you all the variables in the process and then you can go through them one at a time to get the one you want. That's clearly not very efficient, especially if you plan on doing it a lot.

But you can use much the same approach to get a single process variable:

```js
// Given:
// score: { value: 5, type: "integer", valueInfo: {} }
const score = variables.get("score");
console.log(score);
```

Will print out:

```
5
```

Again, you should be sure that the variable exists _before_ you try this as doing this for a variable that _doesn't_ exists will result in a runtime error.

## Bonus content: what to do with the variable once you have it?

So now you've retrieved the variable, so what do you do with it? And then how do you put it _back_ with the new value?

```js
// Given:
// score: { value: 5, type: "integer", valueInfo: {} }
var score = variables.get("score");
console.log(score);
score += 10;
variables.set("score", { value: score, type: "integer", valueInfo: {} });
console.log(variables.get("score"));
```
Will output:

```
5
15
```

As you can see, you retrieved the variable, changed it, put it back, and when you retrieved it a second time you got the changed value.

