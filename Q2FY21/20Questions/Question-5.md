## Question 5

**An Example of completing a user task via the REST API**

Often when integrating your business process with other services you need for that external service to handle part of the task, and maybe even complete it. This is a very common usage pattern. You have a REST client that you want to claim a task, and complete it, and then return control to your Camunda process.

The first thing is to decide if you will implement this as an `External Task` or as a straight REST call from an external application.

## The External Task method

First, let's go through doing this as an external task. And let's take a very simple BPMN as our example:

![the simplest BPMN you can imagine](images/diagram.png)

A call comes in, and a Service Task will handle the call, and complete the task.

Next we have to define which Service Task will handle this task

![Service Task definition in properties pane of Camunda Modeler](images/ServiceTask.png)

Notice that we have defined the service task as an `external` implementation, and have then added a `topic` of `handle_call` for that task.

Now it's time to implement the REST calls. We're going to do this using the Go Library for Camunda. The very first thing to do is create a Camunda client with the proper URL for your Camunda instance:

```go
client := camundaclientgo.NewClient(camundaclientgo.ClientOptions{
		EndpointUrl: "https://localhost:8090/engine-rest",
		// ApiUser:     "demo",
		// ApiPassword: "demo",
		// RESET to 10
		Timeout: time.Second * 10,
	})
  ```

  Now that we have an instance, we can create a processor to handle activity:

  ```go
  proc := processor.NewProcessor(client, &processor.ProcessorOptions{
		WorkerId:                  "MyWorkerID",
		LockDuration:              time.Second * 5,
		MaxTasks:                  10,
		MaxParallelTaskPerHandler: 100,
		LongPollingTimeout:        5 * time.Second,
	}, logger)
  ```
  And finally we'll create a handler that will deal with the tasks as they arrive:

  ```go
  proc.AddHandler( // Handle the call that just came in
		&[]camundaclientgo.QueryFetchAndLockTopic{
			{TopicName: "handle_call"},
		},
		func(ctx *processor.Context) error {
			return handle_call(ctx.Task.Variables, ctx)
		},
	)
  ```

  With that setup, every time a call comes in, it will be place on the outgoing task queue waiting for an external task to come claim it. the `addHandler()` function is just sitting around looking for anything to show up on that queue, at which point it fetches the topic, and locks it so no one else can come claim it, and then calls the go method `handle_call()`.

  That method could look something like this:

  ```go
  func handle_call(newVars map[string]camundaclientgo.Variable, contx *processor.Context) error {
	  fmt.Printf("Running task %s. WorkerId: %s. TopicName: %s\n", contx.Task.Id, contx.Task.WorkerId, contx.Task.TopicName)
	  in_vars := contx.Task.Variables // gets all the context variables
	  callerOK := isValueInList(fmt.Sprintf("%v", in_vars["caller"].Value), getCallers())
	out_vars := make(map[string]camundaclientgo.Variable)
	out_vars["callerOK"] = camundaclientgo.Variable{Value: callerOK, Type: "boolean"}
	out_vars["status"] = camundaclientgo.Variable{Value: "true", Type: "boolean"}
	if !callerOK {
		out_vars["message_type"] = camundaclientgo.Variable{Value: "failure", Type: "string"}
	} else {
		out_vars["message_type"] = camundaclientgo.Variable{Value: "success", Type: "string"}
	}
	err := contx.Complete(processor.QueryComplete{
		Variables: &out_vars,
	})
	if err != nil {
		return err
	}
	return nil
}
```
This method extracts all the context variables from the process, checks the value of one, and then sets a few more before returning everything and telling the engine that the process has been completed.

One thing to be aware of is that in the call to `processor.NewProcessor()` we set a lock duration of 5 seconds. If the execution time of your external task takes longer than 5 seconds, an error will actually be thrown in the engine because the lock timed out. So be aware of how long your external process may take, and set the appropriate timeouts.

## The REST Method

Now, let's say you don't want to do external tasks at all, but you just want to be able to do a straight REST call to grab a task. That's doable as well, but you're going to need a little more information.

First, you can set the task up as, say, a `user task` instead of an `external task` as before.

Next, you would then call the REST API:

```java
  String url = "http://localhost:8080/engine-rest/task/c44b7c61-5026-11e7-9d74-064bed1e2b33/complete";
  URL obj = new URL(url);
  HttpURLConnection con = (HttpURLConnection) obj.openConnection();
	// Setting basic post request
	con.setRequestMethod("POST");
	con.setRequestProperty("Content-Type","application/json");
  //if you want to add some variables
	String postJsonData = "{\"variables\":\r\n    {\"aVariable\": {\"value\": \"aStringValue\"},\r\n    \"anotherVariable\": {\"value\": 42},\r\n    \"aThirdVariable\": {\"value\": true}}\r\n}";
  // Send post request
	con.setDoOutput(true);
	DataOutputStream wr = new DataOutputStream(con.getOutputStream());
	wr.writeBytes(postJsonData);
	wr.flush();
	wr.close();
  int responseCode = con.getResponseCode();
  System.out.println("nSending 'POST' request to URL : " + url);
  System.out.println("Post Data : " + postJsonData);
  System.out.println("Response Code : " + responseCode);
```

**Note:** you **must** know the `task id` in order for this to work! You can't simply call this without a specific task id. Finding the task id is a whole other matter. Your best bet for *that* is contained in the [get tasks](https://docs.camunda.org/manual/7.13/reference/rest/task/get-query/) query.

Even with that, you will need some detailed information in order to get the list of Tasks, and find the task id you are looking for.

One of the nice things about a pure REST implementation is that if you have [Postman](https://postman.com)or a similar REST client, and you've enabled the Rest APIs on your Camunda engine, then you can test out all your REST calls via the client application.

If you're using Postman (or [Paw](https://paw.cloud) which is my personal favorite) Once you have the API call working properly, you can have the client application generate the code for you in almost any language! That way, you know that call is right, and you can be certain that the code to _make_ the call is right as well!
