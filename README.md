Invalid Maven Project: Sibling Modules
========================

A quick demonstration of a broken approach to modules in a Maven
project.   
 
*Note:* The Invalid Maven project is an attempt to clarify note only best
practices but to identify "worst-practices" in Maven.   In my time as
a working developer I've seen many self-appointed Maven experts push
odd and often incorrect approaches to Maven integration.   While the
books I've written have helped people learn how to use Maven, they
haven't covered the negative case: how not to use Maven.

So here it is: How not to use Maven.

Sibling Modules
------------

Explanation, in [the Maven Book](http://bit.ly/Jf3HaO) I don't just
warn against creating Maven projects with sibling projects, I tell you
it won't work.   A few people have asked about this recently, and I'd
like to clarify.

What is a sibling module?
--------------------

Assume you have three projects: module-a, modules-b, and module-c.
In a normal multi-module Maven project module-a will be the parent
directory of module-b and module-c.   module-a will have a pom.xml
that lists both projects as modules:

    <modules>
      <module>module-b</module>
      <module>module-c</module>
    </modules>
    
This leads to a very predictable outcome for builds.  If you build the
project from the module-a/ directory, Maven will run a multi-module
build including both module-b and module-c which are subdirectories.
So the file system looks like this:

    module-a/pom.xml
    module-a/module-b/pom.xml
    module-a/module-c/pom.xml   

Every once in a while, you'll stumble upon a build engineer who has
decided that modules can be sibling directories instead of
subdirectories.   

For example, the projects could still have the same relationships with
module-a refering to module-b and module-c as modules with all of the
modules being siblings in the same directory.  For example, module-a
would refer to both modules using the following modules syntax in the
pom.xml:

    <modules>
      <module>../module-b</module>
      <module>../module-c</module>
    </modules>

And, the filesystem would have a flat layout:

    module-a/pom.xml
    module-b/pom.xml
    module-c/pom.xml   

This will work.  If you define relative directories for modules, Maven
will make this work, but just because it works doesn't mean it is
right.

Go ahead and try it.  Change directories to module-a and run "mvn
clean install".    It'll work.  Try to load this project into m2eclipse.
It'll work.   You might think that you are on to something with this
sibling module thing.

It's not "invalid", but it'll break assumptions...
-------------------------------------

But, do me a favor, try running "mvn release:prepare" and just wait
until the Release plugin tries to tag something in the SCM.   Just
know that you will probably see some errors, or maybe you won't.  I
can guarantee that you are going to be confused.   The Release plugin,
along with other tools, makes a few assumptions about where modules
live.... it wants them to live in a subdirectory.

So that's the problem.  Maven may let you do this, but the toolsets
and plugins you use often make assumptions.   Think about Jenkins or
Hudson plugins - in my experience CI plugins are "The Wild West" of
Maven support, and I can guarantee you that CI plugins make a lot of
assumptions.

I'm caught in a relative directory ghetto
-------------------------------

The other reason to shy away from this practice is that you recreate
one of the problems that led to the creation of Maven.   If your
projects depend upon relative directories, it breaks the idea that you
can checkout one component of a larger build and not have to worry
about anything outside of that one isolated directory. 

Carry this approach out a few years and you'll end up with a big
directory of directories all of which relate to one another.  It will
get to the point where you will be afraid to modify one module for
fear of some cascading module dependency that isn't apparent (because
you decided you knew better than the Maven Book).

But, still, people persist...

Why would people do this?
----------------------

There are many reasons.  First, is a belief that Eclipse (or some
other IDE) doesn't support nested directory structures for Maven
projects.  Second is an idea that a flat project structure is
"cleaner".  Third, is an attempt to mix and match modules and create
multiple project structures using sibling directories as a way to mix
and match project.  And, last, is this idea that Maven's approach to
multi-module project layout is somehow wrong.

Let's address each of these one at a time:

Q1: Is Eclipse incompatible with nested projects?
----------------------------------------

Before m2eclipse this was very much the case.   Eclipse people held on
to this idea that Maven project just got it all wrong when it came to
project structure.     *Luckily this is no longer the case.*

[m2eclipse supports nested projects](http://bit.ly/IXkkpm).   If you
are not using m2eclipse yet, [go get it](http://bit.ly/IXkkpm).    If
your build engineer is telling you that you need sibling modules for
Eclipse, please tell them about m2eclipse.    If you are trying to
develop Maven projects without m2eclipse you are doing it wrong.

Q2: Is a flat project structure "cleaner"?
--------------------------------

The question is based on an invalid premise which is that this is even
an assumption of Maven that is up for discussion.   It isn't.   Maven
has certain expectations, one of them is that project modules will be
subdirectories.     I hate to say it, but if you are using Maven and
asking questions like this, it is either very, very late, or you don't
understand your project's relationship to Maven.

Maven has a Way about it.  If you don't like it, then leave it.  No
one's going to judge you if you go use Gradle.   In fact, Gradle's
awesome, especially for people who are going to get hung up on things
like sibling modules.

If you are working somewhere and your build engineer insists on
sibling modules because they "are cleaner".   It's time to do one of
two things:  1. Quickly find a replacement, 2. Suggest moving to
Gradle immediately, or 3. Seek alternative employment.   Honestly,
using Gradle would be a better alternative to trying to shoehorn Maven
into someone's idea of how it should run.

Q3: Our build person does this to create dynamic module sets, bad?
-----------------------------------------------------

Very bad, don't do this.     But maybe people don't understand what
this means?    Look at module-d/ in this project.  Isn't that great,
module-d has a different set of modules than module-a and there's
overlap.   Some people see this and think that can use this to create
different builds for different target environments.

Sure, you could go off on your own, embrace a worst practice, and use
this feature to reimplement Maven Profiles.  Or you could read the
Maven documentation and learn about Maven Profiles.  If you really
need to be able to change module sets involved in a build.  You can
define (and redefine modules) in a profile.    Do that, don't do this.

Q4: C'mon Maven's approach to multi-module projects is wrong...
------------------------------------------------------

...Stop right there.  Maven's assumptions are Maven's assumptions.
Your opinion or my opinion makes no difference.   If you want to go
make your own build system from scratch go and do that.    Better yet,
go use Gradle, Gradle is better for people who find pages like this
annoying. 

Man, chill out, if I want to do sibling modules, it's my prerogative.
----------------------------------------------------

Whatever, go do whatever you want to do, but please don't come crying
back to Twitter or your low traffic blog when the Release Plugin
starts throwing out springs and spewing smoke.    If I had my way,
Maven would notice you were trying to do this and print out a big
disclaimer form for you to fax into the project's maintainers.  If
would read something like this:

    "The undersigned understands that what they are trying to do is
    both backwards and ill-advised.   By signing this document, the
    developer agrees not to hold Maven liable on Twitter for the fact
    that sibling modules totally screw everything up."
    
Or, just do whatever.  I don't care what you do unless I have to come
in and clean up the aftermath, which, if you choose to do this will be
somewhat messy once you have a hundred projects all pointing at each
other.

You know what it's going to eventually look like?   That scene in
Reservoir Dogs when they are all ready to shoot each other.   In other
words, it ain't going to be pretty.

