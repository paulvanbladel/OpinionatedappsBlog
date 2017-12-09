+++
date = "2016-06-23T18:07:37+00:00"
draft = false
title = "My favorite software development 'principles'"
slug = "my-favorite-software-development-principles"

+++

## Introduction
Today I felt the need to write down some 'principles' based on my experiences in software development of the last years. It's by no means a full list, so I'll try to update this post regularly when progressive insight pops up.
I separated 3 categories: Coding, requirement gathering and agile methodology.

## Coding
#### Code readability
Code is read more often than it is written. Therefor, write only readable code. Readable code is not code that demonstrates how smart you are but code written with the genuine intention to be understood by others.  Talk with other team members about code readability and enjoy the effect.
#### Semantics is everything
Even if you are not a Domain Driven Design practitioner, contemplating about using an ubiquitous language, make sure your code expresses correctly its intention. There is nothing wrong with a refactoring cycle purely devoted to fine tuning the code semantics and making code simpler.
#### What's the best code ever?
The best code is the code not written because that's the only code which is 100% bug free.  So, talk with the client/business about complex features with low added value. KISS (keep it simple and stupid) starts already during the requirement gathering phase. Customers will appreciate that you think together with them in "Pareto terms (80/20 rule)".
#### Learn test driven development
Apply always Test Driven development even for simple features. It will lead to better code composition and  provides a generous safety net for incremental refactoring and future changes.
#### Learn functional programming
Apply functional programming principles. That doesn't necessaraly require  pure functional and often more academic languages, also in javascript or c# a functional programming style is very well possible, has a huge pay-off and thus highly encouraged.
#### Patterns or not?
There is nothing wrong with going through the classic software patterns (style  Gang of four) but in the end the S.O.L.I.D. principles really matter. Favor Composition over inheritance. Ensure low coupling but without dogmatism.
## Requirement gathering/interaction with business client
#### release early prototypes
Clients/business parties who are kept too long in a insecure state about the software under development tend to formulate more complex requirements. Therefor early and even premature prototypes are key. Prefer working prototypes over wireframes because the abiltity to inject live data in a prototype makes clients more expressive in formulating requirements as simple as possible but no simpler.
#### modeling tools
If you think that tools like UML can help you in the communication process with the client and between your team members, use it but often it's better to stick to simple user stories and some enlightening diagrams which generally don't fit in what general requirement gathering tools offer. The best communication is the one that is tailored for the specific background and needs of the customer. Creativity, fantasy and imagination are key here.
#### Favor Living documentation
If you have the occasion don't hesitate to adopt some techniques that lead to living documentation e.g. Behavior driven development and https://openapis.org/
## Agile Methodology
#### scrum is a learning experience
Only join an Agile/Scrum team if everyone understands the rules of the game. Avoid teams where management adopted scrum because a 'sprint' sounds faster. Join teams where agile is a joint and pleasant learning experience for both developers and business.
#### apply scrum like a monk
Although they work very hard, monks in a monastery usually experience low stress. That's because they have a very strict and repetitive daily rhythm.  Agile/Scrum is offering the same type of cadence under the form of the daily stand-up and open communication based on trust. Embrace this opportunity to reduce stress.
