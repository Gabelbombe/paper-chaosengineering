
# Chaos Engineering for Fun and Profit

So lets say you're developing a new web application, that _Next Big Thing_ that's had everyone on the edges of their seats forever. After all the tireless work you've put in, sleepless nights debugging and refining.. It's finally time to launch your service to your would be clientele. Now comes the hard question...


#### When is your Application ready for Production?

Or more to the point, how can you make certain that your distributed system is resilient enough to survive a frequently fickle internet? Let's be honest, you can never be sure (or too sure). You just don't know what (or more importantly when) something bad will happen. There will _always_ be something that can (and inevitably _will_) go wrong. These can range from self-inflicted outages caused by bad configuration values, buggy server images (AMIs or VHDs) or even events that are outside your control such as DDoS'ing attacks or network failures. No matter how hard you try, you can't build the perfect software (let's be honest, you can't build perfect hardware either...). Unfortunately this also bleeds over to the companies that you tend to rely on (I'm looking at you AWS, I remember the US-EAST-1 outage like it was yesterday).

We live in an imperfect world, that's just how it is... It's time to accept that and focus on things that _are_ in our sphere of control, such as creating quality products that are resilient and reactive to failures. Build software that is able to cope with both expected and unexpected results; gracefully degrade whenever necessary. As I often say __Pray for the best, while  always expecting the worst.__

Okay this sounds sweet right? But how? How can you make sure you're ready for disaster?

The first thing(s) that we'll need to do is to isolate and identify the problems that may arise in a production-esque environment. Only then will you be able to address systemic weaknesses and make our systems fault-tolerant.

This is where [Chaos Engineering](http://principlesofchaos.org/) comes in.


#### Principles of Chaos Engineering

The concept of Chaos Engineering is that, rather that waiting for things to break in Prod (which will always seems to happen at the worst time), we will proactively inject failures (or chaos) in order to prepare for scenarios that will likely arise when disaster _does_ strike. This tends to go hand in hand with another oft cited thing which is _Disaster Recovery_. [Netflix](https://medium.com/netflix-techblog/the-netflix-simian-army-16e57fbab116), who was a pioneer in this field and as to date the defacto standard defines __Chaos Engineering__ as:


 > Chaos Engineering is the discipline of experimenting on a distributed system
 > in order to build confidence in the system's capability to withstand turbulent
 > conditions in production.

As a Chaos Engineer, you are testing a systems ability to _survive and thrive_ by simulating potential errors (IE, failure modes) in a series of controlled experiments. These experiments typically consist of the following steps:


 1. Define your systems normal behavior _its "steady state"_ based on measurable output through metrics such as performance, memory, basic activity, persistence and error metrics (I've detailed these in another paper [Redis Health](https://github.com/ehime/paper-redishealth/blob/master/section/metrics.md) that should be applicable here).

 2. Hypothesize about the steady state behavior of an experimental group, as compared to a stable control group.

 3. Expose the experimental group to simulated real-world events such as server crashes, malformed responses, traffic spikes or service outages.

 4. Test the hypothesis by comparing the steady state of the control group and the experimental group. The smaller the differences, the more confidence we have that the system has attained a acceptable state of resiliency.

We can also put this into less scientific terminology: __let's intentionally break shit, compare broken and battered with expected impact and correct any problems uncovered this way.__ This creates a more hardy _Disaster Recovery_ plan with tangible results that we can use in the implementation of a much more robust system.

Let's try an example. Let's say you wanted to know what would happen, if for some reason your database becomes suddenly unavailable for whatever reason? You can hypothesize that your web application would stop serving requests and immediately begin returning errors. To simulate this even we could block access to our database server either by instituting a bad port, garbage credentials or some other self imposed means. Afterwards however, we observe that the application seems to take an unreasonable amount of time to respond. With investigation we find the root cause _a misconfiguration in our timeouts_ and are able to fix it in a matter of seconds.

As this example fairly clearly demonstrates, Chaos Engineering makes for effective resilience testing. Besides the fact that it's a ton of fun, you get to tell people you now break stuff for a living! Who doesn't love breaking stuff?


#### How do we get started?

Netflix went the extra mile and built several autonomous agents, the so-called "monkeys". These purposefully inject failures into your ecosystem to create different types of outages or arbitrarily bad data. As an example, Chaos Monkey will randomly terminate virtual machines which simulates server outages. Latency Monkey will induces artificial delays in API calls to simulate service degradation. Then there is my all time favorite, Chaos Gorilla. He is programmed to take down an entire datacenter. Together they form a fair portion of the [the Simian Army](https://queue.acm.org/detail.cfm?id=2499552).

While the Simian Army is a fairly novel concept, we don't necessarily need to automate our experiments to run continuously when we're just getting started. Honestly it's best (IE, sanest) to introduce your company to the concept of Chaos Engineering by starting small and in measurable increments.

This really means that instead of wreaking havoc on our production system from day one, we would be wise in attempting encapsulated experiments in an isolated staging environment (if you don't have a pre-production environment yet, this would be the time to start creating one!). While your two environments will be different in more or less subtle ways, any resiliency testing is better than no resiliency testing! Later, as we start to gain confidence begin introducing more experiments (or more preferably, all of them) in production. __Remember:__ Chaos Engineering is focused on controlled failure-injection. You are the one making the rules here.

The purpose of our chaos experiments is to simulate _disaster conditions_. Thought his might sound like a difficult task (and yes it requires a creative touch) in the beginning it's easier to focus on availability (or rather the lack of) by injecting failures so that fragments of our infrastructure become unavailable. Intentionally terminating machine clusters, killing workers, dropping database tables, shearing access to internal and external services... You know, all the fun stuff! We will lean a lot about the the inherent coupling of our systems and discover the subtle dependancies tying it together that we would normally overlook.

Later, we would want to look at  simulating other events capable of disrupting our _steady state_, like high latency caused by slow network performance. These experiments are generally harder to pull off and often require special tooling, though the takeaways tend to be amplified by the extra effort.

Whatever you decide to do, I can promise that you will be surprised how much you can learn from chaos.


#### Plan, execute, measure, adjust

Briefly, here are the steps involved in conducting chaos experiments. This list is based on my own experience after reading the excellent article "[Resilience Engineering: Learning to Embrace Failure](https://queue.acm.org/detail.cfm?id=2371297)"

 1. Begin by planning our experiments. Compile a list of potential failure modes, how we'd attempt to simulate them and what we believe the impact will be. I recommend using a spreadsheet, as this will allow us to use pivots and look-ups.

 2. Commit to a date. At this point we should inform all stakeholders of affected systems, especially if you anticipate any trouble for customers.

 3. Gather the whole team in front of a big screen and run through the experiments together. Make sure to record via [Webex](https://www.webex.com/) or some other system. This will be the best way to transfer knowledge, develop a shared mindset and prep newer employees who will be coming in in the future.

 4. After each experiment, log your the actual measured impact. Get this into [Confluence](https://www.atlassian.com/software/confluence), a [Runbook](https://en.wikipedia.org/wiki/Runbook) and any other tool you tend to use for knowledge transfers.

 4. For every flaw uncovered compile a counter-measure list. Don't worry about immediate implementation for fixes, we are compiling a log at this point! Add any follow-up items to your issue tracker.

Rinse and repeat, we should be running our experiments on a regular basis (I recommend quarterly at the minimum) to detect regressions as well as newly surfaced issues. Don't forget to bring your spreadsheet. :wink:


#### Moving further into Chaos

At this point we should have a good understanding of why we need to implement failures proactively (in a controlled manner, of course) in order to gain confidence in our systems. We should also understand that chaos engineering will enable us to verify that things will behave as expected or to fix them when they don't. As we advance you will discover that automating a manual process will be a key success criteria, ensuring a stable and robust platform in which to ply our wares. Though I still believe that manual testing can be at the forefront of this, which can be as simple as terminating a process with the `kill` command. This tends to be the easiest approach to get familiar with the concept of fault injection and to gradually establish the right mindset.

As outlined in the previously mentioned article [The Principles of Chaos Engineering](http://principlesofchaos.org/) there are four advanced principles that should be observed. This paper states:

 > The [advanced] principles describe an ideal application of Chaos Engineering […]
 > The degree to which these principles are pursued strongly correlates to the confidence
 > we can have in a distributed system at scale.

The main principle we're interested in today is described as follows:

 > Running experiments manually is labor-intensive and ultimately unsustainable. Automate
 > experiments and run them continuously. Chaos Engineering builds automation into the system
 > to drive both orchestration and analysis.

In short, the Principles suggestion of automating experiments (that are currently manual) to run in a continual fashion (taking out the Human error) in order to add a compounding increase of confidence in our systems. Thankfully not only does Netflix tell us what to do but gives us a remarkable tool in order to put this theory into practice, which is __Chaos Monkey__.


#### One surly ape
