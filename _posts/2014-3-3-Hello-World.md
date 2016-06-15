---
layout: post
title: Resolving an Intermittent Test Failure
---

Recently while working on a project I ran into an issue of intermittent test failure.  Due to the randomness of test seeding I couldn't exactly pinpoint when this issue creeped into my code base.  Additionally, I was perplexed since this was still early in the project and I didn't see anything out of the ordinary from other projects I've worked on.  Outside of recently updating versions of Ruby and Rails not much had changed in my routine.  Under most circumstances I would have stopped and researched the cause, but instead I made the decision to move forward in the project with this minor annoyance occasionally popping up.  As my code was working as expected in development mode I attributed this as a one off.

As I continued onward I started noticing more test failing intermittently.  So now I had further confirmation that something was definitely amiss with my test suite.  Now that I had multiple test failing at random intervals I decided to get down to some debugging.  I tossed in `byebug` for the test in question and was able to find the issue, stray records in my test database.  What further dumbfounded me was that I was sure I had constraints on the DB to prevent duplicates and had in fact tested against this happening. I was further perplexed as this issue had never really been a problem for me before, and when I did run into this type of situation I had enough experience resolving it.  But not this time.

It was at this point where instead of simplifying and isolating the test in question I doubled down on my complexity and added another dependency, database_cleaner.  After some configuration and setting my strategy to deletion, I was able to get my test suite passing without errors.  Unfortunately, this solution kicked the can resulting in unknown consequences down the line as another issue crept in, the test database I seeded from fixtures consistently failed after a second run and I had to reload the seeds.  But as the test are random this would intermittently succeed but more often fail.  This resulted in a chicken/egg scenario where changing my strategy to transaction returned the original errors and deletion the later errors.

Unfortunately, I continued to stray.  Instead of attempting to resolve an issue I added more complexity by searching for means of automating a rake task to seed my test database.  I continued to add more code to resolve more problems that I was creating in the first place.  It was at this point I accepted that I needed to backtrack and comment out all test sans one of the failing, and remove the dependency of database_cleaner.  I was straying farther from the path while being oddly stuck in the mire. 

![_config.yml]({{ site.baseurl }}/images/random_test_failure.png)

So at this point I've narrowed down to one failing test. This was not only something I could manage, but the test itself was so minimal that the issue shouldn't be difficult to finally peg.  Instead of creating more issues to track down and research I reduced the problem to its root and with singular focus began to hunt down information as to the cause.  I understood the underlying problem was that the database was not rolling back but I did't fully know the why.  And after hours of research due to the complexities I added, the why was quickly found from the simplicifcation.

It turned out that I had cross-pollinated test.  What had original started off as strict unit test with Minitest::Test morphed into model test as well.  The problem as I was soon to find out was that running these with Minitest is outside of the database transaction functionality that Rails provides.  Test relating to database manipulation should inherit from `ActiveSupport::TestCase`.  So by getting my chocolate in the peanut butter I accidently introduced a bug in my test.

##Lessons Learned:

**Small Changes, Big Ripples:**  Sometimes a small change can have a big ripple through the testing suite.  I have a standard test configuration that I've grown comfortable using and often copy and paste from one project to another.  This allows me a certain amount of understanding and consistency in my test process, but as was evident here, straying from this led me to issues I haven't experienced before.  Short term this was a max pain scenario, but it also resulted in longer term knowledge.

**Solve a problem, don't create solutions:**  Instead of addressing the problem and keeping it in check when it arouse I allowed it to grow.  Instead of simplifying I added layers and dependencies increasing complications.  I induced more bugs that I tried to find solutions to straying further away from the root issues. I found myself resarching solutions to problems I was creating instead of solving the original issue. 

**Failing test is not always failing code:**  My code was fine, my test were not.  Failing test are not always related to failing code but somteims it is incorrect or poorly written test.
