# 20 Popular Camunda Questions

Recently we were asked an intereting question: What are the 20 most popular questions about Camunda Platform BPMN? And you know, we never really thoguht about it, so we decided to.

First problem was to figure out how to determine what the "popular" questions were. The first thing to do, or course was head over to the [forums](https://forum.camunda.org) and do some sleuthing, but that's not the only way to ask questions. So we checked in with the [Question Corner](https://page.camunda.com/wb-camunda-question-corner) as well to see what was popular over there.

So, now that we had a list of what we thought were the 20 most popular questions, it's time to bring all the answers together! These questions are not an ordered list, just a collection.

## Question 1
**Reading from the application.properties file**: We see this question, in a variety of forms, quite a bit so, even though it is referenced in the [documentation](https://docs.camunda.org/manual/7.14/user-guide/process-engine/delegation-code/) there are still often questions about it.

But in that document, the important bits are:
> To implement a class that can be called during process execution, this class needs to implement the org.camunda.bpm.engine.delegate.JavaDelegate interface and provide the required logic in the execute method. When process execution arrives at this particular step, it will execute this logic defined in that method and leave the activity in the default BPMN 2.0 way.
> ...
> The classes that are referenced in the process definition (i.e., by using camunda:class ) are NOT instantiated during deployment. Only when a process execution arrives at the point in the process where the class is used for the first time, an instance of that class will be created. If the class cannot be found, a ProcessEngineException will be thrown. The reason for this is that the environment (and more specifically the classpath) when you are deploying is often different than the actual runtime environment.

That last part is especially important in that you won't see a potential problem (`Exception`) until the first time the process is run and that class is called. In other words, a successful deployment is not evidence enough that there isn't a problem. You will need to have the process run in order to ensure that there won't be a classpath problem in your runtime environment.

But wait, that's not got anything to do with my properties file! Well, yes and no.

![Properties Tab in Camunda Platform](images/properties.png)

If that's what your properties entry looks like, and this is what your `TestServiceImpl` class looks like

```java
public class TestServiceImpl implements JavaDelegate {

    public void execute(DelegateExecution execution) throws Exception {
      String var = (String) execution.getVariable("input");
      var = var.toUpperCase();
      execution.setVariable("input", var);
    }

}
```
You're almost there. It's just down to how you are referring to your class in the properties tab. `${TestServiceImpl}` should do the trick. But keep in mind the above caveat that you likely won't see any problem with your class being referenced until you attempt to run it.

## Question 2

**Are there better ways of modeling repetitive common tasks?**

For example, a task that is asking for ‘additional information’ required at every task can get cumbersome.

This is a great question that inevitably leads us back to some of our [best practices](https://camunda.com/best-practices/creating-readable-process-models/) for modeling. tl;dr it's better to model _everything_ and leave nothing out. It may seem like it will make your model overly-complicated and confusing but the reality is that by documenting every step in the process you end up making the model more complete, more readable and easier to understand.

Let's say this is your original model:

![Multi-step model](images/moreinforequired.png)

Yep, that's complicated alright! As you can see, there are multiple places where more information is required, but the overall flow looks complicated.

If we clean that model up using some of the best practices we can see a new model:

![cleaned up model](images/moreinforequired.png)

Yes, this model is substantially larger overall, but it's _much_ easier to read, and easier to see what is actually happening at each step.

## Question 3

**How can I check if a variable is set?**

This is an interesting question, with a couple of great answers, depending on your implementation.

First, you _could_ define a variable that gets set when the process begins, like:

```js
var MyVar
```
And then in your execution, you could have an expression that checks this variable:

```
${ MyVar != null }
```
But then you have a variable hanging around in your process that may or may not get used. Probably not the ideal solution.

If you try to use that expression without having the variable set, it will cause an exception, which is *definitely* not what you want.

The better solution here would be to change the `implementation` in the sequence flow from `expression` to `script`. Then, in the implementation box, enter a short javascript implementation:

```js
execution.getVariable('MyVar') != null
```

Which will check if the variable exists at all, and return `true` if it exists (is not `null`).

But it turns out that there is a [documented](https://docs.camunda.org/javadoc/camunda-bpm-platform/7.7/org/camunda/bpm/engine/delegate/VariableScope.html#hasVariable(java.lang.String)) `hasVariable()` call that can simplify this even more. If you're into ternary operators:

```js
(execution.hasVariable("myVar") ? execution.getVariable("myVar") : "null")
```

if you're not,

```js
if ((execution.hasVariable("myVar")) && (execution.getVariable("myVar")) == "whatever")) { .... }
```
will do the trick nicely.

I definitely recommend this last solution as it's a cleaner, more robust solution.

## Question 4

**Strengths of Camunda vs. Flowable**

The answer to this question is basically a summary of [this blog post](https://camunda.com/blog/2016/10/camunda-engine-since-activiti-fork/) from about 5 years ago.

## Question 5

**Single sign-on in Camunda**

This is a very popular topic, and one that has about as many answers as you can dream up. Much of this depends on what you plan to use to facilitate your Single Sign On (SSO) capabilities. We can go through a couple of examples here.

First thing is to understand the Authentication method provided by Camunda. YOu can read about the [Basic Authentication](https://docs.camunda.org/manual/latest/reference/rest/overview/authentication/) method, which is what most of the solutions are based on.

You can therefore provide your own implementation, add it to the web.xml (as described in the basic auth example) and all requests should be authenticated using your implementation.

This only solves the authentication part of the issue, and Authorization details still need to be provided in Camunda itself (usernames and groups need to exist, etc.) but it is a jumping-off point.

Now, if you're looking for a more fully-complete solution, the Camunda Consulting Team has provided a complete solution using the Camunda Springboot Starter, with Springboot Security. You can find that entire project on the [Camunda Consulting](https://github.com/camunda-consulting/code/tree/master/snippets/springboot-security-sso) Github site.

Likewise there is a Camunda [JBoss SSO](https://github.com/camunda/camunda-sso-snippets) example, and a [Keycloak example](https://github.com/camunda/camunda-sso-snippets) as well.

With all these examples, combined with the [documentation](https://docs.camunda.org/manual/latest/reference/rest/overview/authentication/) you should be well on your way to implementing a custom SSO solution.

## Question 6

**