class: tetrate-light, center, middle

# Meet Istio

---

class: tetrate-light, image-break-3
.outer-box[ ]
.inner-box[
  # Problem

  IT shift to a modern distributed architecture has left enterprises unable to monitor, connect, manage, and secure their services in a consistent way.

]

???

One of the best way to understand systems and why we build things the way they are built  is to understand the problems they were built to solve.

That is critical context for the solution.

So this is the problem statement that motivates istio.

As we take our legacy infrastructure, that is monoliths or sometimes services, deployed into static environment, which are environments that don’t change very much, into modern architecture we run into a lot of problems.

The biggest thing behind this move to distributed architectures is the desire for developer velocity. I want my developers to be able to do more things and faster, because that’s what the business requires.

I want to decouple things from each other, so they can move independently and go fast.

It sounds great and it is great; but we have some problems.

The tooling we have today does not cope well with this new world; where things are dynamic and communicate over the network.

There are problems we need to figure out: how do we go about monitoring, connecting, managing and securing their services in a consistent way.


---

layout: true
class: tetrate-light, regular-slide
.company-logo[ ]

---

# Problem
## Modern distributed architecture

- Container based services
- Deployed into dynamic environments
- Composed via the network

???

Let’s explain what we mean by distributed architectures.

We’re talking about…

Service based architectures, probably in containers
Deployed into more dynamic environments that we’ve traditionally dealt with. If we think of kubernetes, pods come and go, change IP addresses and change ports dynamically on the fly all the time. This is in direct contrast to some of the VM based systems where we deployed a vm with a persistent IP address that runs one application for years.
And maybe the most important difference is in the increased importance of the network. As we break monoliths into separate pieces, those separate pieces are composed over the network, so we have to deal with the network in places we never had to deal before; procedure calls become remote procedure calls and remote procedure calls have so many different failure modes that a normal procedure call on a one machine never had that it is staggering. And our tooling just can’t cope with it.

Or in a single buzzword… Microservices.

So istio was built to solve these problems - it helps us monitor, connect, manage and secure services in a consistent way.

Let’s break down the problem into sub-problems and go into a bit more detail.

Starting with monitoring.


---

# Problem
## MONITOR

- Understand what’s actually happening in your deployment through basic tools:
  - Metrics
  - Logs
  - Tracing

???

When we are talking about monitoring services we really mean getting some visibility into the state of our dynamic system. We need to be able to understand what’s actually happening in our systems.

There are 3 key ways we can do this - we generate metrics about our service, so as requests come in and out we want to publish metrics that some other system can consume and gives us a summary of that. So how many request had 500s, 200s and so on.

We also want details logs. Those metrics give us a great view of the front door - stuff that goes in and goes out, however, the metrics don’t give me a lot of insight about what’s actually happening. And that’s where we traditionally use logging to fill in those gaps.

Finally tracing, which the the newest of the 3 from the perspective of the monitoring systems we are building. It’s like logs on steroids. So rather than just printing statements as we go through we can annotate the object that traverses through not just my services but, if our system is set up correctly,  we can propagate these traces through an entire set of services, so we can see for one request that the user made what happened. This is very powerful.

One of the benefits we will see overtime as these systems mature and develop is better correlation capability. That is the key piece missing today. When you hear people talk about observability a hot topic know is an event - and the idea of an event is to capture these things together in a unit, so that we can start to correlate metrics with logs and traces and I can see for that slow request why was it slow, what were the vital stats for the service and so on.

---

# Problem
## CONNECT

- Get network out of the application
  - **Service discovery**
  - **Resiliency**
    > Retries, outlier detection, circuit breaking, timeouts, etc.
  - **Load balancing**
    > Client side

???

We want to connect services together.

How many people have implemented retries with a for loop in your application code? How many of you implemented this correctly, every time, without bugs or caused an outage? It is fairly easy to DDOs internal infrastructure like this.

And this is really the problem. If we look at when we first started to build programs that were communicating with each other we had a set of problems we had to solve. We had to solve naming, we had to solve delivery (making sure the other end actually got the message we sent), we had congestion control to not overwhelm the other server, we had to do all these things. And we saw these things start to be repeated in applications that need to communicate to each other and we eventually extracted that out into what we call the OSI network model. That is the interface we think about and interact with the network today.

And then as soon as we had that nice model that handled a lot of that complexity around naming, retries, congestion control, and delivery we immediately started rebuilding the same logic again in our applications at a higher level. That is what your retry loop is, rate limiting is congestion control, service discovery is naming - it’s exactly the same principles we have and that we have solved at L4 and below, pop-up again at L7. And service meshes goal is to push these networking abstractions below the line that application really needs to think about.

And there’s a bunch of things that  come into that - things like service discovery, automatic resiliency, retries, circuit breaking and these other things that make your system settle into a steady state that’s healthier than it might otherwise be.

Load balancing is a key piece here - in particular if we can enable client side load balancing we can do some fancy things with point-to-point communication without having to go through the intermediaries, without introducing more points of failure. We will cover more how service mesh can enable this and how Istio does it.

---

# Problem
## Manage

- Control which requests are allowed, and how and where they flow
  - **Fine-grained traffic control**
    > L7, not L4! Route by headers, destination or source, etc.
  - **Policy on requests**
    > Authentication, rate limiting, arbitrary policy based on L7 metadata

???

Now we have all these services and they are deployed everywhere and one of the other problems we face is - when I had a monolith there were only a couple of ports the app listens on and then all the communication is internal and it spits out some answers. So when we need to secure this thing it’s relatively simple - we can lock down any port that’s not the one I need and then  start applying some IP rules to control who can communicate with the monolith and who cannot. And that is great in the static model, but when IP change a lot this traditional model of network security becomes brittle.

So instead what we’d really like is a fine grained control on who can talk to whom, service to service communication, and within that I’d like to say for specific requests whether it’s allowed or not, whether it routes to one instance or another. I want to be able to do fine grained control over where and how traffic flows and what the traffic is and which is allowed.

---

# Problem
## Secure

- Elevate security out of the network
  - **Service-to-service authentication**
  - **Workload identity (L7)**

???

And finally we want to secure these things. Management and security go hand in hand. Today the security primitive the identity primitive we use most often is IP and port pair. That does not translate well to a dynamic world and so the idea is that we’d like to issue identities to the workloads themselves and before we issue that identity, we want to make sure that the environment in which that workload is executing is appropriate and that the workload itself is good.

I don’t want the developers building something on their laptop, deploying it to prod and getting a production identity. It’s the wrong binary - it’s the right environment, but it’s not a vetted workload that went through my build system.

Conversely, I don’t want a developer to run a production binary to run on their laptop and get a production identity. Because it’s a right binary but it’s a wrong environment.

So if we can verify those two things - if we can authenticate the environment then we can authorize the issuance of identity and then we can use that identity at runtime for policy and feel secure. And we will talk about how we do this.

---


# Service Mesh
## What is service mesh

- Service mesh moves these facets out of the application  for better division of labor and...
  - **Consistency across fleets**
  - **Centralized control**
  - **Ease of change**
    > Update configurations without redeployment

???

Service mesh can touch each of these areas and provide those features. But one of the coolest thing about the mesh is the way it centralizes control.

So before, all of these features that I talked about can be implemented in something like a client library for instance. Twitter is famous for having done this in Finagle and original Linkerd wrapped that in a binary. But this is a tried and true thing. Google did the same thing with client set of libraries that handled all of these.

The problem is updates - I need to go individual app teams and say “rebuild and redeploy the binary to pick up a new version of SDK”. And now you need to start doing coordination because some teams updates slower than others and some versions might not be compatible and it introduces a slew of problems. These systems traditionally did not allow any form of centralized control. I could not go and affect the retry policy of all callers to a service even if they are all calling through the same SDK.

So service mesh centralizes this control and what this does is it empowers a small group of people in organization to be responsible for a huge amount of operational expense. For example, a central metrics team that’s responsible for metrics ingestion, collection and presentation for the entire org, can operate that part of the mesh, can affect changes across the entire org just through config pushes independent of other teams and full own that. They can then also offer dashboards to other teams to see their services.

And we can do the same for security and traffic management. We can move control out of the individual team and into these centralized groups. The other benefit is that because I have centralized control I can also delegate that control and give it back to individual teams for things that shouldn’t be managed by a central team. For example no central team is going to set retry policy and timeouts for every individual service - service owners should set those up, the centralized team can delegate that.

Finally, when we need effect change on the system we need to be able to do it quickly. WE need to be able to respond to a dynamic environment at the rate of change of the environment itself and typically pushing new binaries is too slow. So if my config is encoded in the binary, I need to go do a whole new rollout to do a change and typically that’s not fast enough when you have a production event. WE should be able to push configuration to effect change and not push binaries.

Then we start to do really nice things like treat your config in the same way you treat binaries, do progressive rollouts, continuous deployments and those kind of things.

---
layout: false
class: image-background-text

# What is .text-dark-orange[Istio?]

Istio is a platform to **monitor**, **secure**, **connect** and **manage** services consistently