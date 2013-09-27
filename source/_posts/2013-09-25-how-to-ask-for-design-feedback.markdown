---
layout: post
title: "How to avoid designing at the water cooler"
date: 2013-09-21 16:22
external-url: http://stewartmccoy.com/how-to-ask-for-design-feedback/
author: Stewart McCoy
comments: false
categories:
  - product development
  - design feedback
---

I recently had an experience I expect is common for designers who work at startups: I presented a well-polished mockup to a key stakeholder late in the design phase of a project and he told me flat out that the design didn’t address his users’ needs. This, after I’d met with him twice before.

This was my first project at my new company and I was still getting to know everybody and their roles, but excuses aside, I quickly realized this happened because my team never got our business stakeholders together at the beginning of the project. Because we didn’t make time to discuss the problem we were trying to solve, everybody was operating under their own assumptions. If we were going to improve communication on our next project, my product manager and I needed to discuss how to set expectations about our design process; we needed to be more intentional about when we got feedback from which groups of stakeholders, and what kind of feedback we were seeking from each group. If we were successful, we’d be able to get consensus quicker and reduce iterations needed to get to a solution that was ready to be specced.

<!-- more -->

This is our refined approach to the design phase of product development. I realize it applies to our particular context, but my hope is to encourage you to reflect on how you gather feedback on your own projects. You don’t have to subject yourself to every feature request from a co-worker or user, and you don’t have to design your product a certain way because your VP said so. You have the power to guide the conversation and to get buy-in in a way that’s less stressful, more democratic, and more efficient.

# Identify the problem

At the outset of any project my product manager will have identified the core problem she’s trying to solve for our users (rather than prescribing a feature). Ideally, she’s done the work of gathering the necessary user research (both qualitative and quantitative) to validate the problem is, in reality, a problem for our users. She will have also prioritized this problem against business goals, ensuring it’s a problem that _needs_ to be solved, and that it’s one that should be solved now rather than later. The result of this work is that our standup team has a clear understanding of _why_ we should be solving that problem. Once we’re in agreement, our PM takes the following steps to get buy-in on the goal of the project from key stakeholders at the company:

## Write a design brief

- A design brief is a one-pager that explains what we’ll be doing for the project. The product teams at my company have agreed that a good brief provides a one-sentence goal statement; limiting the goal to one sentence is important because it should be easy for the designer to reference by memory during any conversation throughout the design process. If I can’t remember the goal of the project without referencing the brief, that’s a strong indicator of scope creep.
- The brief should also identify the target users I’ll be designing for. For example, is this release limited to a beta group or will it be released to all users?
- I love constraints, so I’m glad that with every brief, my PM lists criteria under “An effective design should…”. An example might be, “allow organizers to understand who is in their audience (age/gender/location) so they can develop ideas for more relevant content.” Notice this criterion doesn’t prescribe how the feature should be designed—that’s my job, after all.
- The last aspect of a good brief is defining how we’ll measure success. There’s usually a separate meeting with our research and data teams to get their feedback on this section. With our research lead, my PM tries identify any qualitative feedback that needs to be captured and how we’ll go about recording, analyzing, and reporting that feedback. And with our data lead, our goal should be to gain an understanding of the data infrastructure that’s in place or that will need to be built to collect, analyze, and report key success metrics for the project. I also make sure my PM has put time on the roadmap in an upcoming quarter to process qualitative and quantitative feedback among our team and stakeholders—_before_ we begin our next iteration of the product.

## Kickoff the business stakeholder meeting

Once the brief is ready, we present it to business stakeholders, who represent the diverse interests of our company (e.g., Business Development, Marketing, User Research, and Strategic Partnerships). The outcome of the meeting is agreement on the goal and the design criteria for the project. This ensures everybody has the opportunity to speak their mind about the problem we’re trying to solve. I find it’s particularly important to get everybody in the same room, not just have them on a call or to speak with them individually. Doing so facilitates the kind of constructive conversation that brings up all the potential issues surrounding the problem, such that we carefully craft the goal statement to encapsulate all these nuances.

## Finalize the design brief

Once agreement is reached, we share an updated design brief with all stakeholders. I can now refer to this brief to for solution exploration and general design direction for the project. This document also gives me tremendous leverage to articulate why certain design decisions are best—given the goal and constraints that were agreed upon by the business stakeholders. If I’ve any questions or concerns or am unclear about the goal statement or any of the design criterion, I make sure to address those with my PM before I begin sketching UIs.

# Design potential solutions

You might be familiar with Apple’s 10-3-1 design process. The idea is that an Apple designer must come up with 10 entirely different mockups for any new feature, before whittling down to 3, further refining those directions, and finally settling on the best design. This process encourages breadth over a linear iterative process, and helps ensure that the best possible solutions have been given due consideration. The following steps are how my team puts 10-3-1 into practice.

1. **Explore with sketches**: My goal is to sketch 10 concepts that attempt to solve the problem articulated in the design brief. Depending on the scope of the project, I might not be able to generate 10, but the aspiration is to generate as many potential solutions as possible. I work quickly in Illustrator, so I draw thumbnails on paper to get the gists of various approaches, and then move to wireframes.
2. **Review with standup team**: Once those sketches are ready, I review each exploration with my engineers and PM, making sure to address how each solution addresses the project goal and how aspects of the design address each design criterion. We then pick 3 concepts that best solve the problem and can be implemented in the amount of time allocated on the product roadmap.
3. **Review with Tech and Creative**: At this point, it’s important to get feedback from implementation experts (e.g. leadership from Product, Engineering, Creative/Design) and then refine the three options in light of technical constraints such as database infrastructure and front-end development costs, as well as constraints regarding our branding and user experience.

## Identify the best solution

- **Review wireframes with business stakeholders**: By now I have a set of 3 detailed wireframes. I personally print them out on A3 and post them on poster board because my stakeholders are all in the same office; you might need a different presentation approach if you’re team is distributed. My goal with this meeting is to review each solution and gain consensus on which design best solves for the goal articulated in the design brief. With the direction agreed upon, all design feedback from this meeting on should be focused on execution details (i.e. information hierarchy, messaging, and visual and interaction design).
- **Conduct usability research**: Ultimately, real users are the people I trust to help me determine whether my design is any good. During the tests I’m trying to discern issues users have with comprehension, navigation, and sentiment. I like to capture this kind of feedback at this point in the process before investing lots of time in the details of hi-fi mockups or prototypes. Since researchers are usually few in numbers or nonexistent in many organizations, you’ll might be the one doing this research. If so, be sure to reference the canonical usability book [Don’t Make Me Think](http://www.amazon.com/Dont-Make-Think-Revisited-Usability/dp/0321965515/ref=dp_ob_title_bk) by Steve Krug—if you haven’t already—for more insight into usability heuristics for web and mobile, as well as for solid testing methodologies.
- **Make a functional prototype (optional)**: Sometimes my team needs to validate a design with real content or real data—there might be too many variables that could break the design. When that’s the case, I make a point to pair with my engineering team to build a barebones prototype that simulates the UX. That said, sometimes a “prototype”, per se, isn’t necessary. I currently work on a dashboard product and when I’ve new projects, sometimes my PM runs SQL queries and generates various charts in Excel, which gives us a pretty good approximation of what our users would see.

# Ship it

- **Mock up hi-fis**: Having validated that that my design won’t be entirely confusing to real users or fail under poor content, I move on to hi-fi mockups that account for all the states and transitions that my engineering team will need to build. These include zero states, changes in calls-to-action based on the user type and information to be displayed, as well as the nuances of particular transition. (I try to include responsive states, if applicable.) During this phase it’s important to regularly put your mocks through review with your design team because they will give you feedback on aspects of the design that other team members can’t give you. I’ve also found it tremendously valuable to my growth as a designer to set aside time to pair with another (senior) designer on the details of the design. As Charles Eames said, **“The details are not the details. The details make the design.”**
- **Write the spec**: Once the mocks are done, my PM uses them to write a spec that fully defines the implementation of the design. The spec is a detailed glossary and set of instructions written specifically for the engineering team so that they don’t have to guess at what they’re supposed to be building. While my PM is writing the spec, I take responsibility for reading it and commenting where my PM has interpreted the mockups differently than I intended—the design phase isn’t over until the design has shipped.
- **Build & ship awesomesauce**: I also take responsibility for talking regularly with my engineers during the implementation phase. I attend daily scrum team standups, sprint planning and storytime meetings, and sprint retrospectives, when I get to see the progress of the design in code and make sure it’s to spec. Even if your organization doesn’t use an agile methodology, communication issues come up during all points of the product development cycle, and engineering work is no exception, so you should be talking with your engineers regularly while they’re building out your designs.

# Iterate

Once my design has shipped, I make sure my PM and I have set aside time to take a look at all the feedback and analytics that have been produced by our users, and to meet with our team and stakeholders to make sense of it all. Our goal is to distill all that information into some key insights that can inform decision making for future iterations of the product/feature we shipped.

Coming full circle, these insights help to fill out the design criteria of our next brief, providing us with strong evidence for justifying our product strategy and roadmap, and giving us confidence and leverage with our stakeholders at the beginning of every project.