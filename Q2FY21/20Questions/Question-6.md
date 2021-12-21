## Question 6

**What are the biggest advantages of Camunda BPM over custom java workflow application?**

This can be a tricky question to answer because it can be heavily dependent on your specific business process, your future plans for automating more business processes, and the level of comfort your developers have with Java, BPMN, etc. but let's go ahead and dive in anyway.

First, it might be instructive to read this [post](https://blog.bernd-ruecker.com/the-7-sins-of-workflow-b3641736bf5c) by Bernd Ruecker on a related topic. Of particular note:

> Throughout my career I saw hundreds, if not thousands of home-grown engines. It literally always happens when you start with no-engine. After a while, projects start to recognize they are missing out on something. They introduce a first small status flag in an order table. But hey, you also need a timeout if customer payments are not received in time. Let’s quickly implement this. After a while projects introduce a small XML or JSON dialect to make processes more flexible. Congratulations, you have built your own engine!
>
> All companies I know want to get rid of this engine. It is a nightmare to maintain. It is not rare to have 1–4 people working full time on this. And it will always lag behind professional engines in terms of features and maturity, especially in the fast-changing IT world.

He is not wrong. Even I've seen it happen, and I haven't worked in the word of BPMN as long as Bernd has!

If you read through [this post](https://forum.camunda.org/t/what-are-the-biggest-advantages-of-camunda-bpm-over-custom-java-workflow-application/3145/3) on the [Camunda Platform Forums](https://forum.camunda.org/) there is a phenomenal answer to this very question that I will attempt to summarize here.

> **BPM/Case engines and long-running business-transactions**
>
> Assuming you’re looking at a BPM or Case engine/framework to help handle business-transactions. These are the potentially very long running transactions that would normally bring your traditional transaction managers to its knees. This means you wouldn’t ask your application server and/or database to keep track of and maintaining transactions (aka ACID) for some business-transaction lasting several hours or days.

While the task you are currently responsible for implementing may not necessarily be a 'long running' task, chances are fairly high that you will have fallen into the trap of implementing your own engine, and the *next* task may very well be a long-running one.

> So, very good messaging systems can handle this - true. But, you’ll seriously sacrifice BPMN modelling concepts - which are now mainstream. And, this is a very serious deal, without a representative model reflecting these long-running transaction/object instances, all their hooks, dependencies, context, and relationships, becomes a mystery. Just thinking in terms of stacking up a set of inter-dependent MQ banking/transfer contexts - how would you do this in a reasonably informative visualization w/o the aid of a BPM-engine and associated instance reporting?

It's hard to overstate how useful a visual representation of the entire task, complete with all its steps and implementation details, is after a task is implemented. Would you rather go back and have to read through all the code (which may, or, more likely may not, be well documented) or would you rather have a self-documented process model to look over? I've read through enough source-code in my career to know that I'd prefer to start with the model and only dive into the source code if I need to see the details of how a specific part of the model is implemented. (See the next section for more on this.)

> **see:** Bernstein, Philip A., and Eric Newcomer. Principles of transaction processing. Morgan Kaufmann, 2009.

> **BPMN models vs UML or Visio (informal) activity diagrams?**
>
> BPMN gives you more notation features. You could use Visio (etc). But a Visio-model diagram doesn’t provide the ability to magically visualize process instances (i.e. in-flight process).

Further, using BPMN can give you a layer of abstraction that can allow you to alter your business process as business needs change *without* having to go in and touch the underlying code. You can alter the implementation without the fear of introducing a bug into an already functioning system.

> **Operations: Configuration Management and Life-cycle**
>
> This is the “oh-darn” situation when you realize your production systems require an upgrade or patch. Camunda provides both methods and approach for managing this situation.

As with the ability to make changes to the model without making changes to the code, using an engine such as Camunda allows you to upgrade the underlying engine without fear that the upgrade may bring the whole system down due to some unforeseen incompatibility.

> **Built-in Documentation Management**
>
> Thinking of your process/case models - fully illustrated and annotated. These diagrams provide user-self-training opportunities. For example, answering the question “how do I do this business task?” Well, refer to the process-portal and browse through available model types (aka business functions).

These models can also provide you with pre-built parts with which to implement *new* models and processes. Again, without writing new code (with the inherent risk of creating bugs). Don't under-estimate the importance of reusable process building blocks as you move forward with automating more and more processes.

> **Reporting**
>
> Who loves to write reports from scratch? I prefer to get these off-the-shelf and with pre-existing hooks into business-task and process state.

With Camunda, you can have the flexibility of built-in reporting tools that you can use to generate on-demand or other reports as needed. All without writing any additional code. And every line of code you have to write yourself is a potential business liability. It creates the potential for costly bugs, it creates technical debt that must be serviced regularly, and it creates further dependencies that may break in the future.

All of these are things you want to avoid if at all possible.