# DevOps Coding Practices

Following on from the previous post in this series [The Enterprise DevOps Mindset](https://dev.to/dylanwatsonsoftware/the-enterprise-devops-mindset-49g2), this article explores some of the more practical things you can do within your organisation (or startup) to enable faster, less risky releases.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t61rs0urp2au41yya2eb.jpg)

# Tiny changes are the best
Small changesets can have **massive** benefits.

Smaller reviews can:
* Allow developers to give better focus and feedback to reviews
* Reduce the overhead of any one review
* Enable faster reviews
* Lessen the cost (and risk) of merge conflicts
* Get changes into the hands of users (or QAs) earlier for earlier feedback
* Can have lower impact, allowing other devs in the same code base to deploy and launch their features without being impacted by your changes
* Contribute to a better, more collaborative design

{% twitter 397664295875805184 %}

Whilst funny, there is a huge amount of truth in the tweet above! We've all seen it happen, where in order to review, developers are forced to context-switch away from their own work only to find that the pull request is enormous. They take a cursory glance at it and say.. LGTM!

How do we combat it? Well, mostly it comes down to:
* Having a plan
  * Taking the time to think and plan a series of changes or PRs that are individually releasable
* Ensuring a pull request is focussed on only 1 thing
* Writing tests early, so you have the confidence to merge your change

Take a look at this for an interesting rule-of-thumb with regards to review sizing: [150-150 rule](https://www.linkedin.com/pulse/note-code-reviews-150-150-cr-venkataramanan-subramanian/?articleId=6682230820951904256)
He suggests that 150 lines of production code and 150 lines of test code serves as a good suggested maximum someone should be expected to review and review well. Obviously this one is subjective, but it's a good number to at least get a feel for what is "too big".

## Backwards Compatibility
When developing APIs, as with any piece of software, you want to ensure that you can deploy your change at any time. You don't want to force all consumers of your API to be deployed at the same time as the API (even if there's only one consumer). This would create a [lock-step deployment](https://www.google.com/amp/s/blog.steadycoding.com/problems-with-lock-step-deployment/amp/).

Coupling your backend and frontend deployments can cause massive headaches. If you discover an issue with either of them you'll need to rollback both. Creating extra work and delaying all the other pieces of work in both systems.

To avoid this we must only make *non-breaking* API changes.

This means:
* Only add endpoints, don't remove them
* Never add new *mandatory* parameter to your requests (they must always start out optional)
* Never remove a field from a response
* Never rename a field or endpoint (This will actually have the same effect as removing).

Ok ok, I know that sounds a bit dramatic. When I say never, I mean before making these removals you must consider the impact it will have on production systems before making this change.

In order to do that, first update all your consumers to handle your change (not using the field, or sending the new field).. then and only then can you make your API change.

One tool I've found useful for this is [Swagger Brake](https://github.com/redskap/swagger-brake), as well as it's plugins for Maven and Gradle.

## Keystone Interfaces
One good way to allow your code to be merged and tested without affecting the user is with a [Keystone Interface](https://martinfowler.com/bliki/KeystoneInterface.html).

Basically, this entails building all the functionality (with all required tests) for a particular feature (over multiple PRs) then, only once everything is ready, we build the UI (or API endpoint) to expose the functionality. This is obviously simplest when the feature can be exposed via a single button but can work well for all kinds of features. In this case, the button becomes the Keystone that is placed at the end and allows us access to the functionality.

This method is very powerful. As Martin Fowler states, it can even be used to hide whole pages of functionality, we can just use a link to that page as the Keystone. It can work just as well in an API where all the functionality is slowly fleshed out, including DB schema and migrations, and then once it's ready and tested, raise a PR to include the change in an existing endpoint. For internal APIs, where you control all consumers you probably don't even need to go to this much effort... Just add the new endpoint.

I've previously felt odd about this particular technique as it feels odd to have code in your codebase that doesn't get used yet (See [Dead Code](https://refactoring.guru/smells/dead-code), however as long as the feature is still being worked on, this approach can have some serious benefits and avoid some of the complexities and effort with more involved techniques like feature toggles.

## Dark Launches
Martin Fowler also mentions [Dark Launching](https://martinfowler.com/bliki/DarkLaunching.html), which is the idea of calling the code for a new feature without the user being able to tell. If done correctly, this could allow you to test out new functionality and it's performance impacts in production without overly affecting the user.

You might wish to expand some existing functionality to display more information to the user. You could flesh this extra info out, collecting it and gathering production stats on performance or errors (obviously you need to ensure the errors don't affect the user). Once you're happy, add the code to expose the new functionality to the user.. a bit like a Keystone Interface again.

## Feature Toggles
Once you have considered all other avenues (such as Dark Launching and Keystone Interfaces), in order to avoid launching your feature until you're ready you might need to use a [Feature Toggle]( https://martinfowler.com/articles/feature-toggles.html).

It's often still useful to think about your functionality in terms of Keystones. If your feature allows it, create a Keystone and put your feature toggle around that. Once you have to finished building out and testing your feature (with the feature toggled on for you), you can switch on the toggle.. exposing your feature toggled Keystone to the world.

Not all changes allow you to toggle your code at the UI level but the deeper down in the code you place your toggle, the more intertwined with the test of the code it will be and the harder it will be to test and gain confidence in it. 

So place your toggles at the highest level of abstraction you can in your code and try to avoid littering several of the same toggles throughout your code.

## If you can't see it, it didn't happen
Lastly, but possibly the most important point is that you must have the ability to see what is going on in production. That means alerting, logging and monitoring.

It should go without saying that if we want to reduce the time taken to become aware of a production issue then we need to be monitoring production for issues!

In the modern software space, you will be inundated with tools that can get you visibility of your production system.
Let's take a look at some of the types of tools to watch out for.

### Frontend App

* Alerting for errors
* HTTP call logging
* User tracking
* Event Analytics
* Demographic analysis

There are so many things that you COULD track and as you build out your system you will likely need to know more and more about how your system is really used.
The key thing is that your chosen solution allows you to continually expand out your knowledge of the system as required.

Some tools that I've seen be used with success:
* Sentry (I can't recommend this enough!)
* Rollbar
* Google Analytics
* Firebase
* LogRocket
* AppSignal

### Backend Services
Again there are plenty of things you could track, hopefully, some (like performance or resource usage) are already handled by your platform.

As with frontend monitoring, it doesn't matter so much what you choose, just have something in place that can expand as your visibility requirements increase.. and they will.

Here are a few backend monitoring tools that I've seen work well:
* Splunk (Really great at alerting, querying and data transformation)
* ELK (Elasticsearch & Kibana)
* Prometheus
* AWS CloudWatch
* xMatters
* PagerDuty
* ServiceNow
* DataDog

Alerts should raise incidents in your incident management system (severity determined by log).

You can even easily calculate your system uptime as:
{% katex %}\%\space upTime = 100 - \frac{numOfErrors * recoveryTime * 100} {time}{% endkatex %}

For a really good breakdown of what we are actually looking for when monitoring production, take a look at:
[Monitoring Production Methodologically](https://dev.to/dpsoft/monitoring-production-methodologically-talk-with-transcript-80g)

## Continuous Delivery
Continuous Delivery is all about automating everything in your process. Everything, right up to the point where you make a manual decision to deploy to production.

The more that is automated, the easier your release process becomes, the more releases that you will be willing to do.

Whilst it might be difficult for highly controlled industries (e.g Government and Finance), at some point we could even automate the decision to deploy to production. We call this continuous deployment. This is just a natural progression of continuous delivery but isn't required to make serious strides towards faster, easier releases.

The medium ground is having a tool with all the codified criteria you have to approve a release to go to production, then just click the button yourself. But given everything we've discussed increasing risk by delaying, if you've got a tool to do the work, once you have confidence in that, why not just let the tool release when it's ready?

For some tips from Atlassian on how to automate visibility of what you are sending into production, see: [6 steps to better release management in Jira Software](https://www.atlassian.com/blog/jira-software/jira-release-management-steps) 

# Automate, Automate, Automate
Despite everything, you may find that the organisation mandates a whole raft of other checks, post-merge. Don't be disheartened! Once you and your team are following these methodologies the next steps are to begin slowly automating (or even better, negotiating) away the additional post-merge steps.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/najmokfhiz3z2mbixvdv.png)

# Conclusion
There are many techniques out there to help us release faster.
The key things to keep in mind are:
* Prevent your work from blocking other developers
* Commit and merge in small bite-sized chunks
* Trust your tests (and if you can't, fix them!)
* If you can't see what's happening in production, it's probably broken

And most importantly, using these techniques and others, find a way to release to production on a regular basis. Even if you are in a heavily regulated industry, even if you are required to make no customer-facing changes, you can still gain massive benefits from deploying your (dark launched, toggled off, inactive) code to production on a regular basis.
