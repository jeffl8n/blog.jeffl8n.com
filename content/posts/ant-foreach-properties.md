---
title: "Ant Foreach Properties"
date: 2009-03-13T15:38:07-05:00
---

**Update 10/23/2009: [An example of this build has been posted.]({{< ref "/posts/building-axis2-java-classes-from-wsdls-using-ant-foreach">}})**

I’ve been working with [Apache Ant](http://ant.apache.org/) a bit at work the past couple weeks. We’ve been setting up a [Hudson](https://hudson.dev.java.net/) [continuous integration](http://en.wikipedia.org/wiki/Continuous_Integration) server to automate some of our builds and deployments. We’re also hoping to setup more automated tests using tools like [Selenium](http://seleniumhq.org/), [HTTPUnit](http://httpunit.sourceforge.net/), [JUnit](http://en.wikipedia.org/wiki/JUnit), etc., but those are going to be gradually added with the coming releases.

I’ve worked with Ant a lot before, but hadn’t really used the [Ant-Contrib Tasks](http://ant-contrib.sourceforge.net/) until modifying this build script. The [foreach task](http://ant-contrib.sourceforge.net/tasks/tasks/foreach.html) is useful for running a list of things through a certain ant target. We have a properties file that has the webservice name and the associated WSDL URL. We feed that list to an Ant target through the foreach task (for each URL, go get the WSDL file/definition), then we generate [Axis2](http://ws.apache.org/axis2/) stubs from each WSDL file. To automate the generation/retrieving of these WSDL files I was moving the base URL (ex: http://services.test.com/) to the environment property file, since this is different in each environment (Test, QA, Production), but the relative URL (ex: /services/…) stays the same. So to summarize, the webservices properties file now has ExampleService=/services/ExampleService and the environment properties file has SERVICES-DOMAIN=http://services.test.com (SERVICES-DOMAIN is assigned to the ws-domain after the environment properties file is loaded in the ant build file).

Here’s where I ran into a little issue that I wanted to share so no one else has to spend a half hour debugging it. Well, also so I can find the answer here the next time I run into it. The property for the base URL wasn’t evaluating when I was running the WSDL list through the foreach call to get/download the WSDL file. It kept erroring saying ${ws-domain}/services/ExampleService?wsdl couldn’t be found. I spent a few minutes putting echo’s in the ant script to see where it was failing until I saw ${ws-domain} was set up until the ant target that gets the WSDL file. It was then I saw this foreach task attribute:

| Attribute      | Description                                                           |
| :------------- | :-------------------------------------------------------------------- |
| inheritall     | If true, pass all properties to the called target. Defaults to false. |


Now I’m not sure why you wouldn’t want properties passed, but I’m sure there is some good reason why this is false by default. Maybe. So of course, adding this attribute set as true to the foreach ant task made everything work without a hitch.