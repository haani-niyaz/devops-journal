---
layout: post
title:  "Principles and Practices for Continuous Delivery"
date:   2018-04-25
author: "Haani Niyaz"
tags: 
 - devops
 - article 
---

## Values

- "Build tools for others that you want to be built for you." - Kenneth Reitz
- "Simplicity is alway better than functionality." - Pieter Hintjens
- "Beautiful is better than ugly." - [PEP 20](http://www.python.org/dev/peps/pep-0020/)
- Automate ruthlessly.
- "Now is better than never." - [PEP 20](http://www.python.org/dev/peps/pep-0020/)
- "Explicit is better than implicit" - [PEP 20](http://www.python.org/dev/peps/pep-0020/)
- "Readability counts." - [PEP 20](http://www.python.org/dev/peps/pep-0020/)
- Convention over Configuration 

## General Development Guidelines

### Test Driven Development 

> “TDD creates consumption awareness: when you create a unit test, you are creating the first consumer of the code under development.”

### Seperation of Concerns

Write modular, isolated code so that it makes it easier to debug and maintain. 

#### Code vs. Data

Broadly speaking, your **code** and **data** can fall into one of the following:

1. Application logic
2. Business logic (site specific logic)
3. Site specific data
4. Node specific data 


- ##### Application Logic

Defines application behaviour; the logic to install, configure and manage apache for instance. Application logic may also contain conditional logic and data to handle platform specific implementation details. 


- ##### Business Logic 

Business logic manages the application at a higher level. At this point we aren't managing apache but rather a web server. We are not concerned with package names, services, platform specific implementation details etc. instead we are concerned with what components are deployed to run a web server. The details to implement our web server are abstracted away in our application code. 

Business logic is also a forms an interface to pass high level data to your applications e.g: site specific data.

- ##### Site specific Data

Site specific data is the data that is proprietary to your site or environment, and isn’t fundamental to any of your applications. This includes things like your internal yum repository details, sensitive data like usernames and password etc.

> *Always try to provide sane defaults for site specific data, even if those defaults aren’t terribly useful in the real world. This provides significant benefits for anyone attempting to debug, test, or simply play with your your code.*


- ##### Node specific Data

This is a special case of site specific data where a host may require properties that unquely apply to it.


### KISS (Keep It Simple)

> “Indeed, the ratio of time spent reading versus writing is well over 10 to 1. We are constantly reading old code as part of the effort to write new code. ...[Therefore,] making it easy to read makes it easier to write.”
 
Usually clever code comes with a cost. It can hide subtle bugs, make refactoring difficult or potentially have undesired side-effects. Sometimes though complexity is unavoidable but it is important to document why it is there in the first place. For example, complex code introduced to make a module more extendable is perhaps acceptable but  your code needs to be **intention revealing** and if this is definitively not the case, it should be supplemented with appropriate documentation in the README file.

### Single Responsibility Principle

Your application logic should have a single responsibility. This doesn’t mean it does one very specific thing but what it does should be highly related to its purpose and any change should be justified by its purpose.

> When describing the purpose of your code the use of ‘and’ would mean it has more than 1 responsibility. If you describe it with ‘or’ it has more than once responsibility and they are not even related.

When designing your application components it should have a single focus. You should also think about what the inputs and outputs are. The deliberation to determine what your application's functionality 'is' should also be extended to inputs and ouputs.


### DRY over Duplication. Readability over DRY.

Reduce repetion as much as possible. However if reducing repetition in your code can obfuscate your code which can lead to reduced code quality it is preferred to have readability over DRY if there is a conflict in principles.


### Versioning Scheme

Esnure a consistent and standard versioning scheme is followed.

[Semantic Versioning](http://semver.org/).

The versioning standard states:

{% highlight  bash %}
Given a version number MAJOR.MINOR.PATCH, increment the:

MAJOR version when you make incompatible API changes,
MINOR version when you add functionality in a backwards-compatible manner, and
PATCH version when you make backwards-compatible bug fixes.
{% endhighlight %}


### Version Control

- All things must be stored in a remote version control system
- Keep commits small and targeted to a related change

- Branch Naming Convetion

Format:

{% highlight  bash%}
<groupname>/<issue-id>-<short-name>/<username>
{% endhighlight %}

Example:

{% highlight  bash%}
bug/PA-920-fix-yum-repos/jdoe
{% endhighlight %}

- #### How to write a git commit message

Read more [here](https://chris.beams.io/posts/git-commit/#seven-rules).

{% highlight  bash %}
- Separate subject from body with a blank line
- Limit the subject line to 50 characters
- Capitalize the subject line
- Do not end the subject line with a period
- Use the imperative mood in the subject line
- Wrap the body at 72 characters
{% endhighlight %}

### Design for Public Consumption (Even if it is private)

Your application logic should be designed as if it will be released to the public. Although you may never have the need to make it public, designing with this mindset forces you to approach your application design to be well defined, well documented and well tested.

### Fix each Broken Window

Fix bad design, wrong decision, poor code etc. as soon as it is discovered. If there is insufficient time to fix it properly, raise it as refactoring story.

Read more [here](https://pragprog.com/the-pragmatic-programmer/extracts/software-entropy)


### Working Code of over Comprehensive Documentation

Aim to write self documenting code.  In good self documenting code everything does not need to be explained, since variable names, methods etc. have clear semantic names.  

#### README File

While self documenting code tells us **what** the code does, the README file provides instructions on how it can be **used**; *It is an instruction manual for users*.

When attempting to write a README file, follow the language or framework specific README file conventions that are already available. 


### Continuously Deliver

- Only build your binary once
- Deploy same way to every environment
- Each Change should propagate instantly
- If any part fails – stop the line
- Traceability from Binaries to Version Control
- Provide fast and useful feedback
- Do not check-in binaries into version control

