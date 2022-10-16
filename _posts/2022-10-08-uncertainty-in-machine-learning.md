---
title: "Uncertainty in Machine Learning"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Physics
  - Machine Learning
---

Experimental physicists spend most of their research time doing 2 things: building their experiment, and trying to figure out how wrong their results could be. You'll notice that I skipped right over the part where you actually do your experiment and analyze your data. It's not that those things don't happen, it's just that trying to figure out what you don't know or didn't account for is usually a lot harder than doing the thing you set out to do! After all, a result or measurement doesn't mean much if you don't know how wrong it could be.

Take the case of the [Muon g-2](https://muon-g-2.fnal.gov/) experiment at Fermilab.[^1] This experiment was built make an extreeeemely precise measurement of something called the "anomalous magentic moment" of the muon[^2]. The reason physicists are interested in measuring this thing? In large part, it's because the value that is predicted by particle theory does not agree with the number that has been measured with experiments. But it's not just the fact that the numbers are different that is interesting, it's the fact that experimentalists have been able to measure this number to incredible precision, and it's waaay off from what theorists and their integrals say it should be. Uncertainty estimation (in this case, very small uncertainty estimation), is the linchpin of this experiment.

Physicists are also very excited to use machine learning in their research. However, most machine learning algorithms do not include any way to estimate the uncertainty of their predictions. This is a huge problem for physicists if they want to use the output of a machine learning model to infer anything interesting. To be sure, there are use cases in physics for machine learning with no uncertainty estimation, but you probably can't unlock the full potential of machine learning models if you have to constrain the use of them to the parts of an analysis where uncertainty quantification is not necessary.

A lack of uncertainty quantification in machine learning is also a huge problem for other people, maybe most obviously the people who are trying to create self-driving cars. If your car pulls up to an intersection and thinks there's no stop sign, should it still stop? How sure is it that there's no stop sign? Is it worse for a car to accidentally stop at an intersection where there is not actually a stop sign, or to blow through a stop sign at an intersection where there could be pedestrians expecting it to stop? If the decision-making algorithm the car is using can only say "there is a stop sign" or "there is not a stop sign", then it's always going to treat those two possible mistakes as equally bad, even though the car speeding though a stop sign is clearly much worse than stopping (or at least slowing down) at an intersection with no stop sign. Ideally you want the car to say to itself "I can't really tell if there's a stop sign here, so I'll slow down and keep looking until I can get a better idea". You may recognize this as what probably goes through your head in this situation as well. We're all doing uncertainty quantification all the time!

# A section

## A section

### A section

#### A section

##### A section

[^1]: If you think the name g-2 is bad, be advised that its predecessor was named E821...
[^2]: A muon is a fundamental particle sort of like a heavier version of an electron. Unlike an electron, a muon is not stable, and will decay to an electron and two neutrinos after about 2 milliseconds if it's at rest.