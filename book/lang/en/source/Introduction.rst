Introduction
############

Yet Another PHP Security Book?
==============================

There are many ways to start a guide or book on PHP Security. Unfortunately, I haven't read any of them, so I have to make this up as I go along. So let's start at the beginning and hopefully it will make sense.

If you consider a web application that has been pushed online by Company X, you can assume that there are many components under the boot that, if hacked, could cause significant damage. That damage may include:

1. Damage to users - which can include the exposure of emails, passwords, personal identity data, credit card details, business secrets, family and friend contacts, transaction history, and the revelation that someone called their dog Sparkles. Such information damages the user (person or business). Damage can also arise from the web application misusing such data or by playing host to anything that takes advantage of user trust in the application.

2. Damage to Company X - due to user damage, loss of good reputation, the need to compensate victims and partners, the cost of any business data loss, infrastructure and other costs to improve security and cleanup the aftermath, travel costs for when employees end up in front of regulators, golden handshakes to the departing CIO, and so on.

I'll stick with those two because they capture a lot of what web application security should prevent. As every target of a serious security breach will quickly note in their press releases and websites: Security is very important to them and take it very seriously. Taking this sentiment to heart before you learn it the hard way is recommended.

Despite this, security is also very much an afterthought. Concerns such as having a working application which meets the needs of users within an acceptable budget and timeframe take precedence. It's an understandable set of priorities, however we can't ignore security forever and it's often far better to keep it upfront in your mind when building applications so that we can include security defenses during development while change is cheap.

The afterthought nature of security is largely a product of programmer culture. Some programmers will start to sweat at the very idea of a security vulnerability while others can quite literally argue the definition of a security vulnerability to the point where they can confidently state it is not a security vulnerability. In between may be programmers who do a lot of shoulder shrugging since nothing has gone completely sideways on them before. It's a weird world out there.

Since the goal of web application security is to protect the users, ourselves and whoever else might rely on the services that application provides, we need to understand a few basics:

1. Who wants to attack us?
2. How can they attack us?
3. What can we do to stop them?

Who Wants To Attack Your Application?
=====================================

The answer to the first question is very easy: everyone and everything. Yes, the entire Universe is out to get you. That kid in the basement with a souped up PC running BackTrack Linux? He probably already has. The suspicious looking guy who enjoys breaking knees? He probably hired someone to do it. That trusted REST API you suck data from every hour? It was probably hacked months ago to serve up contaminated data. Even I might be out to get you! So don't believe this guide, assume I'm lying, and make sure you have a programmer around who can spot my nefarious instructions. On second thought, maybe they are out to hack you too...

The point of this paranoia is that it's very easy to mentally compartmentalise everything which interacts with your web application into specific groups. The User, the Hacker, the Database, the Untrusted Input, the Manager, the REST API and then assign them some intrinsic trust value. The Hacker is obviously not trusted but what about the Database? The Untrusted Input has that name for a reason but would you really sanitise a blog post aggregated from a trusted colleague's Atom feed?

Someone serious about hacking a web application will learn to take advantage of this thinking by striking at trusted sources of data less likely to be treated with suspicion and less likely to have robust security defenses. This is not a random decision, entities with higher trust values simply are less likely to be treated with suspicion in the real world. It's one of the first things I look for in reviewing an application because it's so dependable.

Consider Databases again. If we assume that a database might be manipulated by an attacker (which, being paranoid, we always do) then it can never be trusted. Most applications do trust the database without question. From the outside looking in, we view the web application as a single unit when internally it is really a collection of distinct units with data passing between them. If we assume these parts are trustworthy, a failure in one can quickly ripple through the others unchecked. This sort of catastrophic failure in security is not solved by saying "If the the database is hacked, we're screwed anyway". You might be - but that doesn't mean you would have been if you had assumed it was out to get you anyway and acted accordingly!

How Can They Attack Us?
=======================

The answer to the second question is a really long list. You can be attacked from wherever each component or layer of a web application receives data. Web applications are all about handling data and shuffling it all over the place. User requests, databases, APIs, blog feeds, forms, cookies, version control repositories, PHP's environmental variables, configuration files, more configuration files, and even the PHP files you are executing could potentially be contaminated with data designed to breach your security defenses and do serious damage. Basically, if it wasn't defined explicitly in the PHP code used in a request, it's probably stuffed with something naughty. This assumes that a) you wrote the PHP source code, b) it was properly peer reviewed, and c) you're not being paid by a criminal organisation.

Using any source of data without checking to ensure that the data received is completely safe and fit for use will leave you potentially open to attack. What applies to data received has a matching partner in the data you send out. If that data is not checked and made completely safe for output, you will also have serious problems. This principle is popularly summed up in PHP as "Validate Input; Escape Output".

These are the very obvious sources of data that we have some control over. Other sources can include client side storage. For example, most applications identify users by assigning them a unique Session ID which can be stored in a cookie. If the attacker can grab that cookie value, they can use it to impersonate the original user. Obviously, while we can mitigate against some risks of having user data intercepted or tampered with, we can never guarantee the physical security of the user's PC. We can't even guarantee that they'll consider "123456" as the dumbest password since "password". What makes life even more interesting is that cookies are not the only client side storage medium these days.

An additional risk that is often overlooked is the integrity of your source code. A growing practice in PHP is to build applications on the back of many loosely coupled libraries and framework-specific integration modules or bundles. Many of these are sourced from public repositories like Github and installable via a package installer and repository aggregator such as Composer and its companion website, Packagist.org. Outsourcing the task of source code installation relies entirely on the security of these third parties. If Github is compromised it might conceivably serve altered code with a malicious payload. If Packagist.org were compromised, an attacker may be able to redirect package requests to their own packages.

At present, Composer and Packagist.org are subject to known weaknesses in dependency resolution and package sourcing so you should always double check everything in production and verify the canonical source of all packages listed on Packagist.org. 

What Can We Do To Stop Them?
============================

Breaching a web application's defenses can be either a ludicrously simple task or an extremely time consuming task. The correct assumption to make is that all web applications are vulnerable somewhere. That conservative assumption holds because all web applications are built by Humans - and Humans make mistakes. As a result, the concept of perfect security is a pipe dream. All applications carry the risk of being vulnerable, so the job of programmers is to ensure that that risk is minimised.

Mitigating the risk of suffering an attack on your web application requires a bit of thinking. As we progress through this guide, I'll introduce possible ways of attacking a web application. Some will be very obvious, some not. In all cases, the solution should take account of some basic security principles.

Basic Security Thinking
=======================

When designing security defenses, the following considerations can be used when judging whether or not your design is sufficient. Admittedly, I'm repeating a few of these since they are so intrinsic to security that I've already mentioned them.

1. Trust nobody and nothing
2. Assume a worse-case scenario
3. Apply Defense-In-Depth
4. Keep It Simple Stupid (KISS)
5. Principle of Least Privilege
6. Attackers can smell obscurity
7. RTFM but never trust it
8. If it wasn't tested, it doesn't work
9. It's always your fault!

Here is a brief run through of each.

1. Trust nobody and nothing
---------------------------

As covered earlier, the correct attitude is simply to assume that everyone and everything your web application interacts with is out to attack you. That includes other components or application layers needed to serve a request. No exceptions.

2. Assume a worse-case scenario
-------------------------------

One feature of many defenses is that no matter how well you execute them, chances are that it still might be broken through. If you assume that happens, you'll quickly see the benefit of the next item on this list. The value of assuming a worst-case scenario is to figure out how extensive and damaging an attack could become. Perhaps, if the worst occured, you would be able to mitigate some of the damage with a few extra defences and design changes? Perhaps that traditional solution you've been using has been supplanted by an even better solution?

3. Apply Defense-In-Depth
-------------------------

Defense in depth was borrowed from the military because bad ass people realised that putting numerous walls, sandbags, vehicles, body armour, and carefully placed flasks between their vital organs and enemy bullets/blades was probably a really good idea. You never know which one of these could individually fail, so having multiple layers of protection ensured that their safety was not tied up in just one defensive fortification or battle line. Of course, it's not just about single failures. Imagine being an attacker who scaled one gigantic medieval wall with a ladder - only to see the defenders bunched up on yet another damn wall raining down arrows. Hackers get that feeling too.

4. Keep It Simple Stupid (KISS)
-------------------------------

The best security defenses are simple. Simple to design, simple to implement, simple to understand, simple to use and really simple to test. Simplicity reduces the scope for manual errors, encourages consistent use across an application and should ease adoption into even the most complex-intolerant environment.

5. Principle of Least Privilege
-------------------------------

TBD

6. Attackers can smell obscurity
--------------------------------

Security through obscurity relies on the assumption that if you use Defence A and tell absolutely nobody about what it is, what it does, or even that it exists, this will magically make you secure because attackers will be left clueless. In reality, while it does have a tiny security benefit, a good attacker can often figure out what you're up to - so you still need all the non-obscure security defenses in place. Those who become so overly confident as to assume an obscure defense replaces the need for a real defense may need to be pinched to cure them of their waking dream.

7. RTFM but never trust it
--------------------------

The PHP Manual is the Bible. Of course, it wasn't written by the Flying Spaghetti Monster so technically it might contain a number of half-truths, omissions, misinterpretations, or errors which have not yet been spotted or rectified by the documentation maintainers. The same goes for Stackoverflow. 

Dedicated sources of security wisdom (whether PHP oriented or not) are generally of a higher quality. The closest thing to a Bible for PHP security is actually the OWASP website and the articles, guides and cheatsheets it offers. If OWASP says not to do something, please - just don't do it!

8. If it wasn't tested, it doesn't work
---------------------------------------

As you are implementing security defences, you should be writing sufficient tests to check that they actually work. This involves pretending to be a hacker who is destined for hard time behind bars. While that may seem a bit farfetched, being familiar with how to break web applications is good practice, nets you some familiarity with how security vulnerabilities can occur, and increases your paranoia. Telling your manager about your newfound appreciation for hacking web applications is optional. Use of automated tools to check for security vulnerabilities, while useful, is not a replacement for good code review and even manual application testing. Like most things, the results are more reliable as the resources dedicated to such testing increases.

9. Fail Once, Fail Twice, Dead
------------------------------

Habitually, programmers seek to view security vulnerabilities as giving rise to isolated attacks with minimal impact.

For example, Information Leaks (a widely documented and common vulnerability) are often viewed as an unimportant security issue since they do not directly cause trouble or damage to a web application's users. However, information leaks about software versions, programming languages, source code locations, application and business logic, database design and other facets of the web application's environment and internal operations are often instrumental in mounting successful attacks.

By the same measure, security attacks are often committed as attack combinations where one attack, individually insignificant, may enable further attacks to be successfully executed. An SQL Injection, for example, may require a specific username, which could be discoverable by a Timing Attack against an administrative interface in lieu of the far more expensive and discoverable Brute Force approach. That SQL Injection may in turn enable a Stored Cross-Site Scripting (XSS) attack on a specific administrative account without drawing too much attention by leaving a massive audit log of suspicious entries in the attackers wake.

The risk of viewing security vulnerabilities in isolation is to underestimate their potential and to treat them carelessly. It is not unusual to frequently see programmers actively avoid fixing a vulnerability because they judge it as being too insignificant to warrant their attention. Alternatives to fixing such vulnerabilities often involve foisting responsibility for secure coding onto the end-programmer or user, more often than not without documenting the issues so as not to admit the vulnerability even exists.

Apparent insignificance is irrelevant. Forcing programmers or users to fix your vulnerabilities, particularly if they are not even informed of them, is irresponsible.

Conclusion
==========

TBD