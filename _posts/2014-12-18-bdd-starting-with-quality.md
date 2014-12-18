---
layout: post
title: BDD - starting with quality
category: bdd, behat
---

Quality isn’t defined by a lack of defects or bugs. You wouldn’t get a cup of barista coffee and exclaim, “This coffee is quality because it doesn’t have a problem with it.” (No bugs in my coffee). You can get a cup of coffee from a petrol station that doesn’t have any problems with it. It’ll be hot, wet and coffee flavoured – there are no defects with this coffee. So, where does this idea of the “quality” of something come from? It’s a subjective thing. A quality cup of coffee to me may not be the same quality cup of coffee to you. If you favour a black americano, your definition of quality is not going to involve a milky heart drawn into the top of it, which is how I like my coffee.

When we talk about the quality of a feature, the fact that it doesn’t have any defects is an implicit part of that quality, but it’s not where quality starts or stops. What makes the quality of a feature is whether or not it does what it’s supposed to do and whether it provides the amount of value to the user that they expect. Again, this is different depending on your context: what you application does, who it does it for and why.

Quality Assurance (QA) plays a fairly key role in any software development team. I know some schools of thought suggest that there shouldn’t be a QA role, and while this is probably the subject of a separate blog post, I feel that this is wrong. We have a QA in the team, just the same as we have a designer in the team. It’s a specialist role that requires certain skills I don’t expect engineers to necessarily have.

That said, I’ve always been troubled with the way that the QA role is executed in a team. Let’s suppose that we’ve got a scrum team that performs well. They commit to a given number of independent stories, work on them sequentially, so they finish the first story before starting the second and so on. Once the feature has been completed, the work of the QA starts in earnest (until that point, the QA will put together a test execution plan and a strategy for dealing with the tests during the sprint). They will begin exploratory testing and creating or updating automated tests. This is all well and good and will ensure that the feature meets the minimum, implied, level of quality. In most cases, it’s enough that it’s free of defects.

For me, this is where the problem lies. But how do we solve the problem?

We realised that actually, we never really discussed what quality meant to a particular story or sprint. We had made assumptions about the quality based on the content of the story and the acceptance criteria. As long as the story met those acceptance criteria and didn’t have any defects, we assumed we were done. In reality, we weren’t really thinking about what constitutes quality but just what constitutes the feature.

So we decided to start with quality. It made sense to talk about what we thought quality meant to any particular story before we talked about anything else. At the beginning of planning a story in sprint planning, we would spend some time discussing what quality meant to this feature. Using the example of a login screen, the story might be:

----
	As a user,
	I need to log in to the site,
	to access all the features.

----

Before we chose to start with quality, we might discuss what the feature looked like, or we may already have a design for it. But then we’d just jump straight into the technical planning: how do we implement it, what code do we need, database schemas – that kind of thing. Instead, now we talk about the feature from a users’ point of view:

What happens if they get their password wrong?
How do they reset their password?
How long should the password be? Should it have characters?
What happens if a user doesn’t have an account, how should we direct them to sign up?
What kind of error messages do we want to show?
Etc.
This opened up a whole new discovery phase. Product Owners cannot think of everything when writing stories and this discovery allowed us to offer our insight into how the feature works, ask questions about how it should work and these are often based on technical knowledge of the platform that the Product Owner may not have. We began by adding these new requirements to the conditions of satisfaction, but they soon become long and arduous to check. So we looked for a better solution than acceptance criteria.

The solution we chose was to use a new tool. BDD (Behaviour Driven Development) is a method of functional testing which allows you to describe the functionality of a feature in a “scenario” file in plain english:

----
	Given I am on the login page 
	When I enter ‘mikepearce’ into the username field 
	And I enter ‘butteryballs’ into the password field 
	And I click “login” 
	Then I should see my dashboard. 

----

We can then use a BDD framework to run these scenarios against our web application. We use Behat as a framework and it works really well (apart from our problem of data fixtures…). It pretends it’s a browser (and, will use Firefox if it needs to work with javascript) and does what your user would do with your system. This allows us to have automated functional testing too. Awesome.

So, when we’re doing this extra discovery step, we record our findings as these step definitions, instead of acceptance criteria:

----
	Given I am on the login page
	When I enter ‘mikepearce’ into the username field
	And I enter ‘wrongpassword’ into the password field
	Then I should see the message ‘Sorry, your password is wrong’
	And I should see the link ‘Did you forget your password?’

----

We slowly build up a specification file for this feature, which is mostly centred around the “happy path” and add edge cases or problem scenarios if we think of them. It’s important to note that we don’t expect to think of EVERYTHING in this session as we time box it to ten minutes and expect other features or ideas to emerge during the sprint.

Once we’ve finished, we’ve got a specification file that we can run against the web app with Behat. The first time we run it, it will fail, because the feature isn’t there – but this is good! This is Test Driven Development for the masses! As the team slowly builds the feature and keeps running the Behat tests against it, it will slowly become more and more green. If new things emerge during the sprint, we add extra steps to the scenario file. By the end of the sprint, all Behat tests will be green and we can have confidence that, not only is the feature defect-free, but it also does what the user expects it to and provides them value.

So, now we have a way of assuring that our software has quality. Not only do we have a slick set of automated functional tests, but we’ve also added a low-friction, low effort step of discovery that allows us to really understand AND define what quality means to us in the context of this feature for our product.

I’d encourage you to try this in your teams. Actually having Behat (or any other BDD framework) isn’t really a requirement to get started. You can start by just writing your scenario file as the first step of your team planning a story and storing it somewhere for future reference. The value is in the discussion you have to define the quality. The artefact you create is useful afterwards for checking that you meet those requirements. The added benefit is that it’s written in a language that anyone can read – from your team and product owner, to stakeholders and anyone else in the business who is interested to learn more about your product.

(The benefits of using BDD are outside the scope of this article for more than I’ve described, there are plenty of articles on the web to suit your style, language and infastructure and I would encourage you to learn more about it. Start with this presentation from [Gojko Adzic](https://www.youtube.com/watch?v=677R07arGtw&feature=youtu.be) and this from [Dan North](http://dannorth.net/introducing-bdd/))

