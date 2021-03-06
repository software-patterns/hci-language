# THE PATH BEYOND THE GATE: SPARKING SOFTWARE QUALITY WITH TESTS

> The Way that can be followed is not the timeless Way.
>
> —_Daodejing_

# The Beginning

Software quality is an enduring mystery. We have heuristics,
metrics, and "best practices" for pursuing it, but few
reliable recipes. Entropy always seems to win; even the most
sprightly and promising projects can turn to unruly, buggy
behemoths in just a few years.

We programmers often think of our field as an ever-changing
one, and it is to this change that we look for the solutions
to our quality woes. We want to believe that the next great
language, tool, or process will solve our problems for us.
But the "next great thing" always turns out to be a
rehashing of an old idea. The fact is, the fundamentals of
programming—the semantics of the languages we use, the
shapes of the programs we write—have experienced no
revolutions in the last fifty years.

Lisp, the forerunner of modern functional languages like
Clojure, appeared on the scene in 1958.

Development of Smalltalk, widely considered the precursor of
modern dynamic object-oriented languages like Ruby, began in
the early 1970s.

Though our languages are old, perhaps we can make progress
via new and improved practices, like test-driven and
agile development. Yet these ideas, though they are newly
popular, are also old.

Test-driven development was used in the maintenance of the
AWK language, initially developed in 1977.

> [W]e instituted a rigorous regression test for all of the
> features of AWK. Any of the three of us who put in a new
> feature into the language from then on, first had to write
> a test for the new feature.
>
> —[Alfred Aho](https://www.computerworld.com.au/article/216844/a-z_programming_languages_awk/?pp=5)

Agile development, too, was useful long before it was a
buzzword. Here is what Tony Hoare said about using
short-cycle, iterative development to rescue a failing
operating system project, the Elliot 503 Mark II:

> First, we classified our [Elliot 503 Mark II] customers
> into groups, according to the nature and size of the
> hardware configurations which they had bought [. . .].
> We assigned to each group of customers a small team of
> programmers and told the team leader to visit the
> customers to find out what they wanted; to select the
> easiest request to fulfill, and to make plans (but no
> promises) to implement it. In no case would we consider a
> request for a feature that would take more than three
> months to implement and deliver. The project leader would
> then have to convince me that the customers’ request was
> reasonable, that the design of the new feature was
> appropriate, and that the plans and schedules for
> implementation were realistic. Above all, I did not allow
> anything to be done which I did not myself understand. It
> worked! The software requested began to be delivered on
> the promised dates.
>
> —Tony Hoare, _The Emperor's Old Clothes_

The year was 1965.

Perhaps these aren't the languages and techniques we want,
but they seem to be close to the best we can come up with.
Can we learn to live with them? Can we make working with
them joyful? Can we, in a word, make them home?

I think we can. And more than that: we can make them
beautiful.

There exists, hidden amidst the technological churn of our
era, a way of programming that is timeless. To illuminate
that way is the mission of this book.

Imagine, right now, the most wonderful code you can: code
that succinctly communicates your understanding of some
tricky problem; that is as simple as it can be and as
efficient as it needs to be; that makes plain by its
simplicity that it harbors no bugs. Code that is a joy to
change and test and run.

We all want to write code like this, but most of us don't
know how. Or rather, we know how, but we don't know what
it is we know. We can't explain it, and we can't perform on
demand.

Yet many of us have, at some point, created code that has
this amazing quality, seemingly by accident. The quality
emerges when we forget ourselves, when we are working
quickly and yet paying such close attention to the work that
the world around us disappears. Then, when we step back from
the code, we blink in astonishment at what we've created. We
try to piece together how we did it, but the moment is gone.
We can't remember.

Indeed, when we look at the code afterwards it seems to us
that we could not have designed it, because we do not know
how to design code like that. Nevertheless, it seems to have
been designed. Whether the code grew from the seed of its
own nature, or was dictated to us by a muse, we can't tell.

The fascinating thing about this type of code is that it is
not just a passive object, a tablet impressed with the stamp
of our knowledge. It actually *teaches* us a new way of
thinking about the problem we are solving. When we are done
writing it, we are often a little confused. Then we read
what we have written and become enlightened.

Here is an example of some code I wrote that I have no idea
*how* I wrote. The `apply` function "updates" a value nested
deep in some immutable object, by constructing a new object
with copies of each sub-object that needs to change.

```javascript
let revise = (draft, key, value) => {
  let copy = shallowCopy(draft)
  copy[key] = value
  return copy
}

let apply = (subject, path, transform) =>
  empty(path)?
    transform(subject)
  : revise(
      subject,
      head(path),
      apply(subject[head(path)], tail(path), transform))
```

The apply function is called like this:

```javascript
let data = {
  people: [
    {name: 'Alice'},
    {name: 'Bob'},
    {name: 'Carol'}
  ]
}

apply(data, ['people', 1, 'name'], function() { return 'Robert' })

/* returns:
 *
 * {
 *   people: [
 *     {name: 'Alice'},
 *     {name: 'Robert'},
 *     {name: 'Carol'}
 *   ]
 * }
 */
```

If I look at this code closely I can convince myself it is
correct, but if you asked me to explain in English how it
works, I'd likely just go through the algorithm line by
line, saying words that are already in the code. I cannot
explain it better than it explains itself. And if you put me
in front of a whiteboard, unprepared, and asked me to sketch
out this algorithm, I'd likely be stumped.

Here's a longer fragment, from the heart of a web UI
framework I wrote: a pair of mutually recursive methods,
`run` and `runOrThrow`, that step through a routine that the
user of the framework has plugged in.

```
  function run(returnFromYield) {
    handleErrors(() => runOrThrow(returnFromYield))
  }

  function handleErrors(mightFail) {
    if (error) return
    try {
      error = null
      mightFail()
    } catch (e) {
      error = e
    }
  }

  function runOrThrow(returnFromYield) {
    if (gotosThisTurn > 1000) throw new Error('Infinite retry loop detected')

    if (!stack.length) return
    let {value: effect, done} = lastOf(stack).next(returnFromYield)

    if (done) {
      pop()
      run(effect)
      return
    }

    // ...

    switch (effect.effectType) {
      case 'perform':
      store.emit(effect.action)
      run(store.getState())
      return

      case 'waitForEvent':
      gotosThisTurn = 0
      return

      // ...

      case 'retry':
      gotosThisTurn++
      pop()
      push(effect.generator)
      run()
      return

      // ...

      default:
      error = new Error('You `yield`ed something weird: ' + JSON.stringify(effect))
      return
    }
  }
```

I wrote this code piecemeal, building it up bit by bit, as I
wrote tests for new behavior. Had I designed the system
deliberately and all at once, I never would have found the
mutual recursion solution. My design would likely have
been far more complicated.

Perhaps you think this is all just gee-whiz, who-cares
stuff, and indeed, I'm not really sure what anyone can get
from out-of-context code fragments like this. My main point
is to demonstrate that I really have had this strange
experience, of discovering that I have written code that I
don't know how to design.

Think of what this means for us who write software. We all
have, hidden somewhere within us, the latent power to
create mechanisms whose designs exceed in simplicity and
beauty anything our conscious minds can devise. If we can
only awaken this ability, and make it part of our daily
work, the software we write will grow and flourish around us
like a forest, propelled into being by a million effortless
acts of creation.

If I could say only one word to the programmers of the world
to try to awaken this ability in them, it would be this one:

**TEST!**

As practical advice, of course, this is simplistic. But as a
foundation to build on, it is solid. The rest of this book
will be dedicated to expanding and refining this
monosyllabic mantra.

Here is the first step of that refinement:

**Test early, and test often. Test small units of code.**

Both of the examples above were created using
test-driven development. The experience of working on them
was that I had only to sketch the interface and write the
first test; the implementation blossomed out from each
subsequent test I wrote, as if under its own internal power,
like a drop of dye spreading and blossoming in water.

The code that is produced by this process is by no means
perfect, or even particularly good from the perspective of
modern code quality metrics. There is a lot in the above
examples that could be cleaned up.

But should they be rearchitected? Will I ever throw them out
and rewrite them? I can't imagine it. These pieces of code
are *useful*. They *work*. Their usefulness overrides any
concerns about style or cyclomatic complexity or the length
of the files they are in. Such issues are only passing
shadows: insubstantial, easy enough to fix if and when I
feel they need fixing.

And yet, just as they are, these code fragments are easy
enough to read, and easy enough to change. I know, because
I have read and changed them many times. I can imagine using
them, and growing them, slowly, for as long as I am
programming. They are not stylish or elegant. Yet they have
an easy, sleepy kind of grace, like a rough stone wall or
the gnarled roots of a tree. They have this quality, I
think, because I did not design them; I imposed no external
criteria of "goodness" on them. I merely began them, and let
them unfold. Their continuing usefulness comes simply from
the fact that I began well.

And this simple act, beginning well—I think it is the most
underappreciated thing in our industry. We think can hack as
we go and refactor our way out of any difficulties. But the
fact is that there are design assumptions that are too big
to refactor, too big even to see clearly under most
circumstances. These assumptions can plant the seeds for
future problems before a single line of code is written.

(example: Concourse stdin bug)

So the puzzle of evolutionary design is this: how do we
choose good initial conditions? More concretely, how do we
choose good *interfaces*, at all levels of scale, that allow
for future change, *without* knowing beforehand how the
design will turn out or what the future requirements will
be?



Axioms:

- a happy programmer is more valuable than an angsty one
- the code programmers work on can make them happy
- the unit that must be addressed is the team: it is not
  feasible, nor really desirable, to make programmers
  individually happy, in isolation.

## Who is this book for?

This is a book for software teams.

It is a book for *everyone* on the team: product managers,
UX designers, engineers from junior to senior. All can, I
hope, learn something from this book.

If you don't learn anything, I hope you at least see some of
your own thoughts about software represented here, and can
use this book to speak more confidently about them with
other members of your team.

And if, on the other hand, you can't make heads or tails of
this book, ask around: an experienced teammate may be able
to give you their perspective on it.

My hope, more than anything, is that this book becomes a
kind of hearth: a gathering-place, in which teams find
shared understanding of the work they do every day.

## What is this book about?

In a word? *Quality*.

It explores the idea of *code quality*, its close relative
*software quality*, and how both relate to *quality of life*
for the people who make and use software. My thesis is that
all these forms of quality are connected—that they are in
fact just different aspects of the same thing.

Many words have already been spent on the subject of
software quality, so you might wonder why I'm bothering to
add a book of my own to the pile.

The fact is, I don't believe the other writers on software
quality have expressed their ideas clearly enough. I don't
mean to disparage their work by saying this: I've enjoyed
many of their books, blog posts, and videos, and learned a
great deal from them. This book would not be possible
without them.

But in my professional life, I've too often seen their ideas
taken as dogma and misapplied. It doesn't help that the
various philosophies of software quality appear to be
contradictory, and people tend to gravitate toward one camp
or the other. Should we make our code object-oriented, or
functional, or simply procedural? Write integrated tests, or
unit tests? Test first or test after? Outside-in or
bottom-up? These unresolved tensions have created schisms in
our industry, across which we often fail to communicate
empathetically.

What we need now is to unify all of our disparate ideas
about quality. I believe that, against all appearances, such
a unification is possible. What makes it possible is that
all of the apparently conflicting philosophies of quality
are just grasping different parts of the same proverbial
elephant. If we understand the whole, we will understand how
each piece of advice applies in its proper context.

I've heard it said that "everyone thinks their way is the
right way, and everyone is wrong." I disagree: everyone is
*right*, but *only in a particular context*. What creates
the schisms in our philosophy of quality is that we can't
easily see the boundaries of our current context, so we
think the principles we've discovered are universal.

Though this book, too, is only applicable within a certain
context, I think it is a very broad context. You're most
likely to find this book helpful if:

- You develop software in an agile way: you don't know most
  of the requirements in advance, but instead must
  continuously release new versions based on feedback from
  users.
- You use a runtime environment that frees memory for you.
  Some of the advice in this book is applicable to
  languages (like C) that require you to free memory
  yourself, but proceed with caution.
- You use languages with support for all three major
  programming paradigms (procedural, object-oriented, and
  functional). Popular languages in this category include
  Java, JavaScript, Python, Ruby, Go, Scala, Elixir, Kotlin,
  Swift, C#, Rust, and to some extent C++. The advice in
  this book leans on the ability to opportunistically use
  multiple programming paradigms. As I'll show later, any
  language that has both function closures (lambdas) and
  mutable local variables can be used to write
  object-oriented programs, and so almost certainly falls
  within the scope of this book.

## Summary of Contents

The remainder of this book is divided into four sections.

The first section, **Feeling Quality**, relates our concept
of code quality to visions of quality in other fields, in
particular the mysterious "Quality Without a Name" described
by architect Christopher Alexander. It tries to answer the
questions: How does something as seemingly fuzzy and
subjective as quality fit into the hard, mathematical world
of programming? Are there different types of quality, and if
so which ones do we care about? What is code quality good
for? How do we recognize it? And how do we go about trying
to create it?

In the second section, **Seeing Code**, I lay the groundwork
for the more technical parts of the book by explaining how I
visualize the structure of code and manipulate it in my
mind. These habits of thought are, as far as I can tell,
essential for creating the patterns described in the third
section.

You can use the third section, **Shaping Patterns**, to
guide your process of writing code. The intention is not
to provide a set of dogmatic rules, but to give you a set of
sane defaults, or "first things to try". The point of having
these defaults is to free up space in your mind to pay
attention to quality.

The fourth section, **The Process of Repair**, describes the
larger context of the other three: the software development
culture that we must create for the Quality Without a Name
to fully come into being.
