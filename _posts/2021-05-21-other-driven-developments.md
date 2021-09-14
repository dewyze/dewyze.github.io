---
layout: post
title: "Other Driven Developments"
date: 2021-05-21
permalink: "/blog/other-driven-developments"
external_link: https://shopify.engineering/other-driven-developments
external_link_title: Shopify Engineering Blog
excerpt: |
  Mental models within an industry, company, or even a person, change constantly.
  As methodologies mature, we see the long term effects our choices have wrought
  and can adjust accordingly. We add more models to transform lessons we’ve
  learned from unconscious habits into public maxims.
tags: development
---

Which <System> Driven Development advocate are you? Test Driven Development
(TDD)? What about Behavior Driven Development (BDD)? Have you read Eric Evans’
book on Domain Driven Design (DDD)? Or maybe your team engages in Sh*tlist
Driven Development? Well it’s a new month; it’s probably about time to add a
few more to the list.

## Why More Models?

Mental models within an industry, company, or even a person, change constantly.
As methodologies mature, we see the long term effects our choices have wrought
and can adjust accordingly. As a team or company grows, methodologies that
worked well for five people may not work as well for 40 people. If all
employees could keep an entire app in their head, we’d need fewer rules and
checks and balances on our development, but that is not the case. As a result,
we summarize things we notice have been implicit in our work.

When I began as an engineer, TDD taught me to consider my outcomes first and
work backwards from the results. As a result, I spent less time writing
unnecessary code. I worked towards the results instead of the imagined
problems. As my career progressed and I worked on harder problems, Sandi Metz’s
Rules forced me to consider the specific code I was writing, instead of writing
code with reckless abandon as long as the tests passed. When I became a tech
lead, problems became human in nature. I did my best to codify what led to
success, and examine what led to failure.

So why do we add more models? We add more to transform lessons we’ve learned
from unconscious habits into public maxims.

Below is a list of development approaches we can adopt and use to measure and
support our development practices.

## Grep Driven Development

Your familiarity with Grep Driven Development likely has a direct relationship
to the number of times you have heard the joke “There are two hard things in
computer science: naming, cache invalidation, and off-by-1 errors."

At least in Ruby, it’s general practice to use expressive variable names.
Rubyists often don’t substitute brevity for clarity. However, the benefits of
good variable names aren’t limited to the future developers reading our code.
Good names serve a variety of other purposes like searchability,
refactorability, abstractability, and consistency.

### Searchability

I recently worked on a new service, and did a quick grep with a few keywords
for a job. Nothing came up. I removed the “job” part of the search and found
what I was looking for. I thought to myself, “I will need to make a job for
this.” I was new to Shopify at the time, so I asked if there was any special
logic or framework I needed to keep in mind for job handling. After a bit of
conversation, a teammate shared, “Oh, you can just add `include
ActsAsBackgroundable`”. This module allows the service to act like a job by
adding enqueue methods that ultimately use the service’s call method. Problem
solved.

However, I realized that when I searched for the word Job that I would have
also missed any of these backgroundable modules. The default in Ruby should be
that your jobs end in the suffix Job. By adding a module that creates job-like
behavior, without including the word job, we reduce discoverability.

### Refactorability and Abstractability

Refactoring is an important part of maintaining healthy codebases over the long
term. Naming our services so they’re easily found when searching, better
enables refactoring practices. In this case, the best practice is easy: do
nothing.

If we’re in a position where we’re duplicating code, Martin Fowler shared a
helpful rule of thumb, the “rule of three.” In other words, don’t go looking
for an abstraction until you have three instances of similar code. Refactoring
after one duplication inevitably produces a premature, and thus likely
incorrect, abstraction

When we copy code from one service to another, we may be tempted to rename some
variables or methods in our newly pasted code. If, in our new case, we’re
working with a Widget instead of a Gizmo, then by all means, rename it.
However, maybe we have a commonly used method like this:

```ruby
def can_be_reviewed_by?(user)
  # <multi-line authorization logic>
end
```

Let’s also say this method is
already used in far more than three places. That would make it a prime
candidate for an abstraction. In this case, we may hate the naming convention,
but don’t have time to find the right abstraction or for a more thorough
refactor of all the places we use this. So instead, we add this method to our
new object:

```ruby
def reviewer?(user:)
  # <same multi-line authorization logic>
end
```

Maybe this is shorter and more idiomatic. Double win! However, we’ve now made a
future refactor of all this more difficult. What if someone is tasked with
finding the right abstraction and they try searching for can_be_reviewed_by?
They won’t find our method and risk further drift.

Admittedly, this example is contrived, and this rule is a little more nebulous.
The specific decision is up to the judgment of the engineers tasked with the
work. It would be equally irresponsible to make decisions solely based on what
other future engineers might do. The main takeaway is that incremental
improvements aren’t always improvements if they make later wholesale
improvements more difficult.

### Consistency

Consistency is another important area to keep grep in mind. It’s really just
another form that searchability and refactorability take. We want future
developers to be able to make assumptions about where behaviors are located. We
want to balance robust code with code that is easily changeable. In order to
make code easily changeable, we need predictable code paths, file locations,
and side effects.

The simple conclusion is to be boring. If your app uses domain verbs phrases
like Creator, Transactor, or ThingDoer, then use those. If your app ends
everything in Service, Job, or Validator, don’t deviate because you find it
redundant or boring. Be friendly to those who come spelunking after you.

## Copy/Paste Driven Development

A common axiom in programming is to Don’t Repeat Yourself (DRY). However, as
mentioned earlier, abstracting prematurely is also not desirable. Sandi Metz
famously puts it, “duplication is far cheaper than the wrong abstraction.” One
of the benefits of a Grep Driven Development mindset, is that when we’re
consistent, the tech debt of duplicated code is mitigated. It’s easier to find
duplicated code when the time comes to perform a proper abstraction.

Knowing when to abstract and finding the right abstraction also vary from
language to language. For example, in the world of front-end frameworks,
there’s a tangible benefit of duplication over infinitely extensible
abstractions. If you constantly abstract all similar behavior between pages,
with the use of flags and conditionals, you’ve gained less duplication at the
cost of your bundle size. If all of your behavior is a pile of spaghetti code
with conditionals that gets loaded on every page, it’s arguable whether it’s
even an abstraction.

What all of this really calls for is a mindset of Copy/Paste Driven
Development. Why would there be value in having a mindset in a behavior that
takes just a few keystrokes? Copying and pasting comes with judgment, implying
laziness. It may be stigmatized as “creating duplication,” all the while being
a normal part of the daily programmer life. We need a proper development
mindset for copy and paste because there’s a lot of inherent behaviors and
beliefs that are a part of copying and pasting code. If we don’t acknowledge
these assumptions, we take them for granted. Copying and pasting have a few
important traits:

- They prioritizes efficiency.
- They are exercises in humility.
- They requires sound engineering skills.

### Efficiency

One of the main benefits of copy/paste is
that it’s fast. Like... really fast. We can produce behavior and functionality
that initially took months to build in a matter of seconds. We don’t need to
rehash lessons that have already been learned or create patterns that now
exist. Often this is a really good thing.

Additionally, it’s tempting to get into the bikeshedding (arguing about trivial
decisions) of trade-offs between tech debt, product roadmaps, system health, or
“just shipping.” They’re all legitimate conversations, and the proper approach
often waxes and wanes given the state of the organization. Instead, we need to
consider copy/paste as a tool to be wielded for a given situation. We reach for
it when we’re low on time, when we’re low on context, or when we have solid
examples to rely on. Likewise, as with all tools, we can decide it’s not an
appropriate tool for a given situation. A wrench can be used as a hammer, but
that doesn’t mean it’s good for the wrench or the nail.

### Humility

Copy/Paste Driven Development is an exercise in humility. It may be tempting to
think something like, “I don’t want to copy code, I want to make sure I
understand it.” So we start writing some similar code from scratch. This comes
with several caveats:

- Is anyone else going to understand it with you?
- Is re-writing the code primarily a learning experience for you? And if so, why do you need the learning experience? (This may be completely valid, but you should ask yourself these questions.)
- Are you sure you understand better than your teammates who wrote the code?
- Also, while you were “just understanding it”, we shipped a bunch of features.

Snark aside, we should be careful when we have the urge to not copy/paste. We
should come from a place of trust for the work our teammates did before us.
Trust doesn’t necessitate a lack of criticism. It just means we require a
higher burden of proof before going our own way.

### It’s a Skill

The most important thing to consider with Copy/Paste Driven Development is that
it’s a skill on both sides. We should strive to write code that’s
copy/pasteable, which means it follows the patterns laid out by our team. We
want code that’s well tested and tests we can quickly read and understand. WET
(repeated code, the opposite of dry) tests are usually very copy/pasteable.
They call out many specific situations and test one situation at a time, making
it easy to delete tests that don’t apply or modify ones that are mostly
similar.

Now you may be thinking, if I am writing code that just can be copied and
pasted anywhere, my app probably doesn’t do a whole lot, or I’m drawing the
wrong abstractions everywhere. This may be true within some of the larger apps
out there, but think about a low traffic common CRUD (Create, Read, Update,
Delete) app. If you have basic functionality, even if the objects are very
different, are your request specs going to be that different? You pass in
parameters, check validation states, and see if you get back a view (or JSON)
or an error. This code is very amenable to being duplicated, and doesn’t
signify unsound practices. It just signifies that a lot of actions on the web
are mostly the same and that the core abstraction, Model-View-Controller (MVC)
in this case, is already present.

This may seem like a tacit approval or copying and pasting all the time. That’s
not the case. Instead, we should treat copying and pasting as a skill, or more
likely, a set of skills:

- Can you read code effectively to know what it’s doing?
- Can you see how edge cases are handled and reason through whether your situation is that different?
- Can you imagine the code you’d likely write and see if it differs that much from the code that’s written?
- And most importantly, if you can do all of these things, can you explain your process and provide answers to your teammates so they can gain these skills too?

These skills should be present when writing any code. When we’ve already
written code that represents a viable solution, these skills help us know what
the most appropriate direction to take is.  Analyzing when copying and pasting
may be sufficient is one of the best contexts for teaching fundamentals of good
software design.

## Ego Driven Development

There’s a tight coupling between Copy/Paste Driven Development and what I’m
calling Ego Driven Development. Ego Driven Development helps us ask why we’re
writing code, or more specifically, why we’re taking on a problem.

### “I Want to Solve This Problem So People Know How Awesome I Am”

Now realistically, very few people say this out loud. And even moreso, many
people may not even know they’re thinking it. A lot of people get excited about
doing green field work. I know I do. I’ve yet to be the person who has typed
rails new (the command for creating a new Ruby on Rails app). I’m not sure if I
ever will be, but I feel like I want to. To be clear, it’s okay to want to be
the person who solves a gnarly problem. It’s okay to want to feel like you’ve
conquered a problem and feel a sense of accomplishment. We will talk more about
that in a second. The important thing is the ability to humbly ask ourselves
the common question, “why?”

When we pick up a bit of work because it’s complex, cool, or fun, we consider:

- Is there someone better equipped to solve this in a timely or robust way? If so, maybe pairing with them would be a good learning opportunity.
- Is there someone on my team who’s been looking for an opportunity to demonstrate career growth who would benefit from taking on the “bigger” project? If so, maybe pairing with them would be both a good teaching opportunity as well as an opportunity for them to level up.
- Do I have a “clever” solution that I think
will work? If so, create a technical design document and request some feedback.
Sometimes, we really do strike gold and your teammates’ input is invaluable in
determining that. Otherwise, be boring and predictable.

### “I Want to Solve This problem So I Know How Awesome I Am”

We can’t discount the value in someone getting “a win.” I remember the first
time I used xargs I felt like a wizard and it made me feel more comfortable in
my job and participating in various discourses. We want to help people get
these wins and produce a culture of humble teammates who want to learn more and
grow as engineers.

If you’re not aware of the idea of Psychological Safety, I encourage you to
take some time to familiarize yourself. In short, people are most effective on
a team when they feel safe. We don’t want teams where people are afraid to push
themselves due to failure. We want our more junior teammates to always be
willing to “take a shot at it.” We want them to be celebrated when they succeed
and supported when they fail.

Some ways you can build this culture on your team are:

- Make an environment where it’s okay to fail, and where you fail as a team, not individuals.
- Help your senior engineers to know their job is more about helping others learn how to solve hard problems and less often about solving them.
- Encourage teammates to advocate and push each other, amplifying their
skills so they feel enabled to take on new challenges.

### “I Want to Solve This Problem So Others Can Be Awesome”

Of course, these are not the only two approaches to software. Sometimes, there
is consensus about the best person to solve a problem. We think it and so do
they. This is the most difficult needle to thread. It’s possible to be great at
something, know it, and still be humble. That comes in the desire not for
accolades, but shared success. So, how do we know when we are in that position?

Humility is a complicated emotion. Humility is self-honesty, not meekness. It’s
a right understanding of our relationship with others. We can indeed know we’re
good at something without thinking that in any way elevates us over our
teammates, or diminishes them. When we can recognize that our sense of
achievement in reducing request times by 50% or inventing a new web framework
or <insert complicated problem here> is no different than a new engineer’s
thrill at writing their first multi-pipe bash script, that’s a good place to
be.

Humility also goes hand in hand with our motivation. If we are curious about
our motivation on the fun problems, we should take a look at how we approach
the boring ones. When we’re presented with a menial busywork that will
nevertheless greatly improve the experience of our teammates or customers, how
do we respond? If our excitement for wrangling yaml files is the same as typing
rails new, that is a good sign your motivations are in the right place.

We shouldn’t strive to prevent good engineers from doing good engineering well.
Instead, we focus on fostering a strong sense of collaboration. We must share
the burden of our large failures and amplify the smallest of successes. We must
encourage humble spirits. If we do these things, we can trust our teams will
act with the right sense of discipline, encouragement, efficiency, and
awesomeness.

## Stickler Driven Development

The final methodology I want to touch on is all about following the rules.
Whether you’re talking about Sandi Metz’s Rules, The Twelve Factor App, or
maybe just a collection of Rubocop rules, we have rules and guidelines that...
well... guide us every day. One problem with these rules is we often learn or
inherit them without context. Additionally, we all have varying relationships
with these rules throughout different points in our careers.

I remember when I was really junior, I’d just stick to the rules and quote them
as a way to avoid critical thinking. Sometimes I’d quote them to new engineers
I was onboarding as a way to try and be a good teacher, even if I didn’t have
the full context. If I’m honest with myself, I quoted them as a way to sound
like a good teacher, while in reality I felt completely lost.

As a more mid-level engineer, I’d operate with reckless abandon at my newfound
wisdom and ignore most rules. Shortly after that, I would get burned by the
aforementioned recklessness and as a senior engineer I’d return to holding us
all to the rules because, “they’re there for a reason! Trust me, I’ve seen
things.” I imagine many have had similar experiences.

In all these cases, a lack of context around those rules means I’d become an
advocate of Stickler Driven Development: do what the rules say because the
rules say it. Even if Sandi Metz’s rule zero is “You should break these rules
only if you have a good reason or your pair lets you.”

### Never and Always vs “It Depends”

While some rules don’t explicitly use the words “never” or “always,” the ideas
are nonetheless present. It’s easy to counter with, “well it always depends.”
In both cases, we fall into the traps of Stickler Driven Development. We get so
caught up in the “ruleness” of a rule, we miss opportunities to explain the
“whyness” of a rule. For example, there’s probably a rule that you shouldn’t
make up words like “whyness.” But if our goal was successful communication, and
a moment of levity, why can’t we say “whyness?”

The point of “never” or “always” is to have a safe default. Often, there are
unknowns and we’re unsure of the best way to write some code, or we’re just
having a foggy brain day and it’s best to stick to tried-and-true methods. The
goal isn’t to provide a one-size-fits-all solution that’s meant to exist
without context.

The point of “it depends” is to know that we, as the engineers facing the
problem, are best equipped to know how it needs to be solved. However, when “it
depends” becomes just as much of an absolute, we miss out on some key nuances.
In almost all human endeavors, “it depends” is true, because we know there can
be some truly odd edge cases (both in code and human endeavors). Nevertheless,
using “it depends” as a rule can be a way to justify not using our “never” and
“always” rules that serve us well most of the time. In other words, “it
depends” can be a justification for the bad kind of Ego Driven Development.

### These Are Not the Rules You’re Looking For

Let’s look at some examples of our “never” or “always” rules. One common adage
in Rubyland is we shouldn’t have methods with “or” in them.

Take, for example, a very contrived method in our service object that will save
an item if it’s saveable, otherwise it will emit it on a Kafka stream.

```ruby
def save_or_emit(item)
  if item.respond_to?(:save)
    item.save
  elsif item.respond_to?(:emit)
    item.emit
  end
end
```

This method has an “or” in it. That’s violating one of our rules, so we decide
to clean it up. Let’s get rid of that pesky “or” and just rename our method.

```ruby
def persist(item)
  if item.respond_to?(:save)
    item.save
  elsif item.respond_to?(:emit)
    item.emit
  end
end
```

We’re no longer violating that rule. Technically. However, the goal of the rule
isn’t really about the word “or” as it’s about unnecessary conditionals and
logic branching (remember the wrong abstraction in Grep Driven Development?).
What if we remove the branches:

```ruby
def persist(item)
  return item.save if item.respond_to?(:save)
  return item.emit if item.respond_to?(:emit)
end
```

Now we have successfully removed the “or” and our “elsif.” Are we done?
If we take the rules at face value, we miss that they’re really not about the
word “or” or conditionals. We’re talking about the Single Responsibility
Principle (SRP). The word “or” isn’t bad and conditionals are an absolute
necessity in any codebase. However, it’s unnecessary complexity and incorrect
coupling we’re really after. If we take a closer look at this method, we see
that the problem is solved if we pass the persistence logic on to our item. The
item will implement a persist method however it sees fit, and our current logic
will become:

```ruby
def persist(item)
  item.persist
end
```

And now we see that our method is entirely unnecessary and we can just
call:

```ruby
item.persist
```

This may seem like an overly contrived example (which it is). However, the
solution may not be as obvious to others. They may be newer engineers, new to
the language at hand, or have never heard about the SRP. Or more importantly,
hearing about the SRP and the problem with “or” isn’t the same as internalizing
the tools necessary to effectively use both rules. If we understand why we’re
doing something, we also know about cases when it just makes sense to break the
rules.

Let’s consider one example in Ruby on Rails. There is a database method called
upsert. If a record exists in a database it will update it with the provided
parameters, and if it does not exist, it will insert (create) one. This method
does one thing or another. Yet, having that method replaces a pattern that has
been duplicated across hundreds if not thousands of codebases where people
check if a record exists before creating, or rescuing from a database error,
etc. Maybe we can say the “Single Responsibility” here is getting information
into the database. Or maybe, it’s okay to sometimes break the rules if the
reasons are clear and it provides genuine benefits.

### Doing the Human Thing

I remember I was at a point in my career where I’d recently learned about
metaprogramming, tried to use it to solve all my problems, and then learned why
that’s bad. Then a PR came my way from a very seasoned ruby developer for some
simple workflow management. The workflow had a variety of methods that were
called in a specific-ish order based on a status.  The complexity was too large
to be encapsulated just in the status field, so we also included a failed_step
column, representing the method that failed.

That resulted in some code that looked something like this:

```ruby
def resume
  send(failed_step)
end
```

Given my recent endeavors, I proudly added a comment to the PR about how
we shouldn’t be using `send`. Because it’s bad. For reasons. Mic drop.

My teammate responded akin to, “yes, that’s usually true. However, in this
case, send provides far more clarity than the alternative would.” This blew my
mind. They were right. Other approaches involve a large case statement,
additional database objects, liberal use of constantize, or complex
serialization. Given the constraints we were working with, this produced the
least amount of code AND, more importantly, it wasn’t complex to reason about.
(I’m sure there was an alternate framework or approach we could’ve used, but
this was the code we were working with, and given that, my teammate was right.)

Yes, this does violate some of the ideas of Grep Driven Development talked
about earlier. Yet, as also mentioned, “it depends.” If you searched for the
method name being replaced by send, you’d only find it in one file, the same
one we’re working in. And if you traced the code path to the resume method, you
easily see what’s happening. The searchability was there, and consistency and
refactoring weren’t applicable given it being one class.

The rules around “metaprogramming equals bad” are mostly based on the human
aspect. It’s cliche, but true, to say the computer doesn’t care. We generally
try and produce code that’s easy to read and understand, because those are the
ultimate factors that lead us to deciding a code base is “nice to work with.”

My teammate’s argument was to do the human thing. In other words, do the thing
for the humans. All of the development models (BDD, TDD, etc.) are about
sharing agreed upon approaches so we can understand each other better. We want
to write code so our future selves can understand our past selves better.
(Maybe we need a Time Travel Driven Development?)

## One More Driven Development

All of our development methodologies, either mentioned or laid out in this
post, are about making our work easier. Sometimes the “easier” is in reducing
time to ship. Sometimes, it’s an ability to refactor or reduce the mental cost
to achieve understanding. It also makes sense why some methodologies go in and
out of fashion over time, while others are brief fads, and others stand the
test of time.

Of the four approaches we laid out here (Grep, Copy/Paste, Ego, and Stickler),
they’re of course not really a form of <X> Driven Development. They’re just
tools and approaches we use daily and usually unconsciously. The goal is to
make them conscious tools instead. We want to be aware we’re wielding a tool
that can be used, bent, or replaced to best serve us.

We want good names because it decreases mental burdens on our teammates. We
should think about how, when, and why we copy/paste, and not do it with no
regard for whether it’s the best tool for the job. We should always be aware of
opportunities for teaching and learning that only can exist within the context
of doing our actual work. We should be aware of why we use rules and why we
choose to break them, not just when.

If our goal for these methodologies is really to do the human thing, making
coding easy for ourselves and teammates, maybe the best way to internalize the
“whyness” is a different name. I suggest:

Human Driven Development.
