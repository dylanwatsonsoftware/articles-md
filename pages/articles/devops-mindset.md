# The Enterprise DevOps Mindset

Firstly, let's start with 2 commonly heard statements:

> Enterprise doesn't do DevOps

> or

> Releasing is hard

_- signed, every enterprise developer ever_

Whilst Enterprise companies tend to have a lot of bureaucracy (especially around releasing), we should still look to improve the pace at which we deliver working software to our customers.

# Does this apply to my Enterprise org?

Yep! Any organisation whose customers, regulators, and developers appreciate faster, more reliable software releases with less risk can employ these thought processes.

# What is DevOps?

Let's start with a simple shared vision of what DevOps actually is.

> DevOps is

> a set of practices

> that works to

> **automate processes**

> so software teams can

> **build, test, and release**

> software **faster and more reliably.**

_From [Atlassian DevOps](https://www.atlassian.com/devops)_

## The target

What are we looking to achieve here?

We are hoping to:

* Reduce the time it takes for our work (including bug fixes) to land in our user's hands

* Reduce the time it takes to become aware of a problem in Production

* Reduce the risk and longevity of downtime due to a release

* Reduce the impact of any particular bug introduced

Ok, but how?

## Git master is sacred

You must ensure that the latest version of your code is always deployable. If you need to "cut a release" and branch off master, you will be less inclined to release on a regular basis.

If your code is not "ready to release" at any time, your ability to safely respond when _shit hits the fan_ is heavily affected.

Think about it, if you first need to rollback your code to a "known" safe place, then this is yet another step in the process before you can even begin fixing your production incident.

In order to keep master "production-ready", all merges to master must also be ready for production. 

This is achieved by 2 key things:

1. Automation

  - Anything that gives you the confidence to release, should be automated and it should prevent branches from merging to master if they fail

  - Your automation should make it VERY hard to accidentally break production

2. Coding Practices

  - Write your code in a way that allows you to deploy to production safely without causing an outage due to a mistake.  

 * Feature Toggles, Keystones, and backward compatibility are just some of the techniques that can help maintain prod readiness.

Part 2 of this series "Coding Practices" will give some examples of ways to ensure your code is production-ready.

## You build it, you run it

The person best placed to fix issues with a feature is the person that just wrote code for it.

If that person is in charge of dealing with any customer issues with that piece of work they will:

* Better understand the problems when they occur

* Be more inclined to take care when writing the code

* Have better context and incentive to improve the supportability of that code in the future

By getting your Devs at least involved in this process, your software quality will be given a good chance to improve, if only through empathy for the user and themselves.

See Atlassian's take on [You build it, you run it](https://www.atlassian.com/incident-management/devops/you-built-it-you-run-it) and another on some of their ideas around [site reliability engineering](https://www.atlassian.com/incident-management/devops/sre)

## Small releases are best

If you spend a year developing software before giving it to a user, you are spending a year building up risk. Any piece of that years (or even months) development could (and likely does) have an undiscovered bug.

If anything goes wrong (and it will), you will have to sift through a whole year of code and features to discover the issue.

On the other hand, if you make a single change and deploy it to production you will likely have much more confidence about deploying to production. Why? Well, if it breaks, it is much easier to have confidence around which change caused the issue... And you can simply rollback to the previous version (or even write a test, fix the bug, and redeploy).

Due to our new confidence in finding issues and rolling back changes quickly, the *risk* of any 1 change is _drastically reduced_.  We also _massively increase_ our ability to avoid downtime due to an introduced bug.

With additional confidence, we can release faster which gives us much better context when things do go wrong since, we have only just finished working on that section of the code.

This only helps give us the confidence we need to release at an even faster cadence and so on.

Simply stated: 

> *Small change == less risk == faster to release*

For further reading here is an interesting Kent Beck article about taking small changes to the extreme: [Testing the boundaries of collaboration] (https://increment.com/testing/testing-the-boundaries-of-collaboration/)

Here is also an interesting read on the [usefulness of pull-requests](https://betterprogramming.pub/are-pull-requests-holding-back-your-team-e8aec48986c2), I think the key takeaway here should be that small, very short-lived branches are best. 

## Automated testing is the key

The other lynchpin in our DevOps plan is testing. If we don't have sufficient automated testing to give us the confidence to release then the plan falls apart. If we rely on manual testing for this confidence, and this manual testing is performed AFTER merging to master, then your manual testing becomes a blocker to production. However, because you have merged to master, you don't just block your code from being released but also any other code that is merged after yours. This puts pressure on the QAs, causing unnecessary delays or forcing them to drop their testing standards.

This is not to say that manual exploratory testing doesn't have its place, it can.

Just that with sufficient automated testing, manual testing shouldn't block a release. If used correctly however, it can be used to validate functionality (often best done by someone with first-hand experience in the business). Manual testing should never be used for anything that you ever want to run more than once to give you confidence you haven't broken something.

For 2 very interesting points of view on appropriate automated testing levels, check out these articles by:

**Kent Beck**

* [”Unit” tests](https://m.facebook.com/nt/screen/?params=%7B%22note_id%22%3A387720532357705%7D&path=%2Fnotes%2Fnote%2F&_rdr)

**Kent C. Dodds.**

* [Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests)

(Apparently, all Kents have strong feelings about tests!)

However, if your QAs are finding issues that if deployed to production would impact users, then you are in luck! The next post in this series "DevOps Coding Practices" will take you through several techniques to avoid blocking production deployments, whilst still maintaining safe and reliable production environments.

# Let's put it into practice

Next up are some simple DevOps coding practices to follow to ensure master is ready to release at any time.

{% post https://dev.to/dylanwatsonsoftware/devops-coding-practices-281e %}

