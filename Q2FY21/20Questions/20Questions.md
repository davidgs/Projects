# 20 Populare Camunda Questions

Recently we were asked an intereting question: What are the 20 most popular questions about Camunda Platform BPMN? And you know, we never really thoguht about it, so we decided to.

First problem was to figure out how to determine what the "popular" questions were. The first thing to do, or course was head over to the [forums](https://forum.camunda.org) and do some sleuthing, but that's not the only way to ask questions. So we checked in with the [Question Corner](https://page.camunda.com/wb-camunda-question-corner) as well to see what was popular over there.

So, now that we had a list of what we thought were the 20 most popular questions, it's time to bring all the answers together! These questions are not an ordered list, just a collection.

## Question 1
**Reading from the applciation.properties file**: We see this question, in a variety of forms, quite a bit so, even though it is referenced in the [documentation](https://docs.camunda.org/manual/7.14/user-guide/process-engine/delegation-code/) there are still often questions about it.

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

**Are there better ways of modelling repetitive common tasks?**

For example, a task that is asking for ‘additional information’ required at every task can get cumbersome.

This is a great question that inevitably leads us back to some of our [best practices](https://camunda.com/best-practices/creating-readable-process-models/) for modeling. tl;dr it's better to model _everything_ and leave nothing out. It may seem like it will make your model overly-complicated and confusing but the reality is that by documenting every step in the process you end up making the model more complete, more readable and easier to understand.

Let's say this is your original model:

![Multi-step model](images/moreinforequired.png)

Yep, that's complicated alright! As you can see, there are multiple places where more information is required, but the overall flow looks complicated.

If we clean that model up using some of the best practices we can see a new model:

![cleaned up model](images/moreinforequired.png)

Yes, this model is substantially larger overall, but it's _much_ easier to read, and easier to see what is actually happening at each step.