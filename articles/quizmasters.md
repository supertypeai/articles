---
title: Calling for Quizmasters for Supertype Fellowship | The Fellowship Thesis
post_excerpt: Quizmasters are the ones that pull the strings behind Fellowship, and they are the supply line for well-crafted, well-engineered Challenges on Supertype Fellowship
taxonomy:
    category:
        - internal-guides
        - notes
    post_tag:
        - fellowship
---

## The Fellowship Thesis

_Starting from the premise that the current education system is inadequate, Fellowship is a new approach to graduating software engineers into the real world._

[Supertype Fellowship](https://fellowship.supertype.ai) is a community of learners, educators, and open source developers who are interested in helping to shape industry-prepared developers by providing them with the opportunity to contribute to open source projects. The program features a combination of expert mentoring, peer-to-peer learning, and plenty of opportunity to help build real software being used by real people.

This method of training software engineers is sorely missed in the current landscape dominated by "run code in browsers" or "run code in Notebook" coding classes that are not representative of the real world. Instead, Fellowship advocates for an education that is:
    - **Real**: You learn how to develop a real feature for a real open source project, and then ship it to potentially millions of other users in the real world
    - **Collaborative**: You learn to code by working with your peers, mentors, and other contributors to the open source project. This teamwork and collaborative dynamic is a key skill required in the real world
    - **Impactful**: You won't just be coding for the sake of passing an assignment or a test. Your code could be used by millions of people, and you will make an impact to the projects you contribute to
    - **Fun**: There is no incentive to learn if you don't enjoy the process. We want projects on Fellowship to be fun and engaging, so your introduction to software development is an encouraging one

Our mission isn't to help people learn to code. There are plenty of outlets for that. We think there are many who code, there are too little who have the ability to build and ship software for the real world. We hope that an environment that fully emulates the real world -- where each project completion (we call it "Challenge") is another feature shipped to the real world -- will be a first step towards that goal.

### Proof of Contribution
In Fellowship, there are no grades, no tests, and no assignments. Instead, you unlock badges and effort points by shipping your code to open source projects (i.e "Challenge"). When your code is merged (i.e "accepted" into the project), you will be awarded effort points and badges on Fellowship in recognition of your contribution and skills. Because of how Pull Requests work, this will leave a traceable record of your contribution ("Proof of Contribution") and this is a big part of how Fellowship verifies your skills and effort. Every Challenge unlocks a related badge, which will be displayed on your timeline view.

The "Proof of Contribution" mechanism has several advantages over traditional programming education:
    - **It is real**: You are contributing to a real open source project, and goes without saying, this provides a more realistic, tangible, impactful learning experience than checking off some quiz questions
    - **Automated**: You don't need to wait for a teacher to grade your work. When your Pull Request is merged, you can submit that as proof of your contribution and Fellowship does an API call to GitHub to verify this ("Proof of Contribution") and automatically awards you effort points and badges
    - **Cannot be cheated**: You cannot cheat the system by copying code from the internet, or randomly guessing between multiple choices. You can't use ChatGPTs (yet!) to cheat a system involving Pull Requests, resolving issues, discussions, handling merge conflicts, agreeing on features, and other community-based collaboration skills. And even if you can, you wouldn't deprive yourself of the fun and thrill of building something of value in the world

Tangentially related to this, we also believe that the Proof of Contribution mechanism is a better way to verify skills than the traditional resume. A resume is a static document that can be easily faked, and it is not a good representation of your skills. A Proof of Contribution is a traceable record of your contributions that are public, verifiable, and unforgeable. It is also a better proxy of skills and competence.

A resume _tells_ me what _you_ say you can do. Pull requests _shows_ me what _you_ have done.

#### Proof of Identity
To safeguard against cheating, or impersonation, Fellowship has a unique, one-time-only, fully-automated Proof of Identity mechanism. A special hashing algorithm is used to generate a unique ID that you link to your GitHub account. This links your GitHub account to your Fellowship account, and is how we verify that you are who you say you are. This is a one-time-only process, and you will not need to do this again.

Details: [Proof of Identity instructions](https://github.com/supertypeai/onboarding)

## The Fellowship Quizmaster
Quizmasters (or "Quiz Masters") are responsible for authoring and designing the Challenge on Fellowship. They are also responsible for ensuring that the Challenge is relevant and up-to-date. Since each Challenge is essentially a feature that is shipped through GitHub Pull Requests, the Quizmaster is also responsible for a clear and concise set of instructions that will guide the learner through the process of contributing to the project.

Quizmaster is an extremely central role in Fellowship, so want only the most passionate and creative software engineers to take on this role. Designing fun, but fair, and meaningful Challenges is a difficult task, and almost an artistry in itself. We hope that each Quizmaster will bring a set of meaningful open source projects to Fellowship, and design well-crafted Challenges for this new generation of software engineers.

### Examples of Challenges

- [Onboarding Challenge](https://github.com/supertypeai/onboarding): This is a great Challenge as its instructions are clear, and serves as a good introduction to using Git and GitHub for the first time. Completion of this Challenge awards the `github` badge, which is a prerequisite for all other Challenges

- [Front End Development Challenge](https://github.com/supertypeai/collective/wiki): This is a fun and engaging Challenge that requires the learner to build a simple Developer Profile page using HTML, CSS, and the React-based Next.js framework. Completion of this Challenge awards the `frontend` badge. This Challenge is also a good example of how a Challenge can be designed to be fun and engaging, and also serve as a good introduction to a new technology

- [Unit Test with Python Challenge](https://github.com/onlyphantom/emailnetwork/wiki/Supertype-Fellowship-Challenge): This might be a Challenge more suited for intermediate developers, but it is also a good Challenge. Learners are required to write unit tests for a real-world Python library, to increase the robustness and reliability of the library. It's a highly transferrable skill that is useful to any developer who wants to ship high-quality code. Completion of this Challenge awards the `python` and `unittest` badge

### Becoming a Quizmasters
The ideal candidate for Quizmasters are experienced developers who are already owners or maintainers of one or more open source projects. They should also have the ability, and pedagogical inclination, to formulate clear and concise instructions that take a learner through the process of making their first code contribution to the project. Quizmasters are not necessarily Mentors on Fellowship (a different role), but they should be able to provide guidance on the Challenge and constructive feedback to the learner.

Perhaps the most important attributes are:
- Empathy: You should be able to empathize with the learner and understand their perspective. This means putting yourself in their shoes and understand what they are going through; In turn, using that understanding to guide you in your design of the Challenge
- Inventiveness: You should strive to make your Challenge fun and meaningful (i.e real impact, not trivial MCQs), so that the learner has an intrinsic motivation to see it through to completion. This is far easier said than done, but a meaningful, well designed Challenge is a massive win for the learner, and a win for the project's community and contributors

If all else fails, remember that Challenge isn't knowledge checks. It isn't a test. You want learners to demonstrate skills through their contributions, not through a 4/5 score on a quiz. This is a big part of the Fellowship Thesis. 

#### What's in it for you?

- **Impact**: Help us shape the future of education, and empower the next generation of developers
- **Recognition**: Your name will be displayed on the Challenge page, and you will be credited as the Quizmaster. The Challenge is hosted on your GitHub profile and linked to the project's repository, bringing more exposure to the project and your work
- **Contributions**: You will gain contributors to your open source project, and help build a community or user base around the project
- **Experience**: You will gain experience in designing and authoring educational content, and learn how to teach and mentor others, and how to build a community around a project
- **Ecosystem**: If you have an idea for an open source project, you can use [Supertype Incubator](https://supertype.ai/incubate) to incubate the project, counting on the support of other developers and engineers to help you build it. You can then use [Supertype Fellowship](https://fellowship.supertype.ai/) to maintain the longevity of the project by attracting a community of contributors and maintainers

If you want to, you can also become a Mentor on Fellowship, and help guide the learner through the Challenge. This is a separate role, and you can read more about it in a separate piece when it's ready.

Please write to me (s@supertype.ai) if you are interested in becoming a Quizmaster, and we shall discuss further.

### Additional Remarks
Jeff Atwood is the founder of Stack Overflow, and wrote this [inquisitive article](https://blog.codinghorror.com/why-cant-programmers-program/). He cited [another article](http://tickletux.wordpress.com/2007/01/24/using-fizzbuzz-to-find-developers-who-grok-coding/?ref=coding-horror) and yet [another article](http://www.joelonsoftware.com/items/2005/01/27.html?ref=coding-horror) and [yet another](http://www.kegel.com/academy/getting-hired.html?ref=coding-horror) by various software engineers in the field. You should give Jeff's article a read, but here are a few paragraphs:

> [199 out of 200](http://www.joelonsoftware.com/items/2005/01/27.html?ref=coding-horror) applicants for every programming job can't write code at all. I repeat: they can't write any code whatsoever...people who struggle to code don't just struggle on big problems, or even smallish problems (i.e. write a implementation of a linked list). They struggle with tiny problems.
> 
> So I set out to develop questions that can identify this kind of developer and came up with a class of questions I call "FizzBuzz Questions" named after a game children often play (or are made to play) in schools in the UK. An example of a Fizz-Buzz question is the following:
> 
> Write a program that prints the numbers from 1 to 100. But for multiples of three print "Fizz" instead of the number and for the multiples of five print "Buzz". For numbers which are multiples of both three and five print "FizzBuzz".
> 
> Most good programmers should be able to write out on paper a program which does this in a under a couple of minutes. Want to know something scary? The majority of comp sci graduates can't.
>
> A surprisingly large fraction of applicants, even those with masters' degrees and PhDs in computer science, fail during interviews when asked to carry out basic programming tasks. For example, I've personally interviewed graduates who can't answer "Write a loop that counts from 1 to 10"...these are basic skills; anyone who lacks them probably hasn't done much programming.
>
> Everybody has to start somewhere. But I am disturbed and appalled that any so-called programmer would apply for a job without being able to write the simplest of programs. That's a slap in the face to anyone who writes software for a living.
> The vast divide between those who can program and those who cannot program is well known. Lest you think the FizzBuzz test is too easy – and it is blindingly, intentionally easy – a commenter to Imran's post notes its efficacy:
> "I'd hate interviewers to dismiss [the FizzBuzz] test as being too easy - in my experience it is genuinely astonishing how many candidates are incapable of the simplest programming tasks."
> t's a shame you have to do so much pre-screening to have the luxury of interviewing programmers who can actually program. It'd be funny if it wasn't so damn depressing.

It's a sobering article and perhaps a rude awakening for some. But it's a reality check that is all too timely in the age of "code in the browser" / "solve data algorithms in browser" style programming education. 

Stane Aurelius, another fellow Quizmaster, quipped that platforms like these applies a strong contextual filter to the problem, isolating it down to a single unit of problem, and when the learner leaves the comfort of such contexts, they are unable to transfer the same skills to similar problems in the real world. This is a core tenet of the Fellowship Thesis, and is why we believe that the best way to graduate a developer is to have them write features and make a tangible contribution to a real-world project. This helps the learner build a portfolio of work, and tune the learner's sensitivity to the nuances of real world software engineering. It's a better context than solving some pointless algorithmic problem in a browser to climb a leaderboard, or by "running" some small code snippet one at a time between 5-minute videos. It's a hell lot more meaningful too than completing a 10 hour lecture series before taking a 10 question quiz to get a certificate.