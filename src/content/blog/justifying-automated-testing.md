---
title: Justifying Automated Testing
pubDate: Oct 27 2024
draft: false
---
# Justifying Automated Testing

There is a phenomenon called [The Curse of Knowledge](https://www.google.com/search?q=the+curse+of+knowledge).
It states that as you become more and more of an expert in a topic,
you run a higher and higher risk of being out of touch with what it's like to learn that topic -
which in turn, makes you less and less effective at teaching that topic.

As an expert, you understand the *why* of things implicitly, because you've had the experiences and internalized
the consequences enough for it to seem second nature, common knowledge. A lot of the rhetoric around software testing
(and more broadly, software) can make it seem like it's obvious. But without that context of experience, it actually isn't.

There are a myriad of tutorials on the topic of software testing. The critical problem with most of these (and many others in the software domain)
is that they show you the **how**, but they don't illustrate a realistic scenario that actually proves and internalizes
the **value** of the topic. They assume that you already see the value, or they don't understand what you actually need to hear to be convinced.
As an instructor, making this mistake is the fastest way to lose your students from the jump.

The reason why I'm writing this post is because this was probably the topic that frustrated me the most when I was learning.
It's also the topic that now makes (almost) perfect sense, and has provided the most "a-ha!" moments since I started working as a full-time software engineer.

I'm hoping to shed some light on a few of the fundamental justifications for **why**
it's important to write tests that I think are critically under-voiced in tutorials on this subject.

I'll also touch on the mistake of writing tests solely for the sake of having them, and some of the pitfalls of herd mentality or "groupthink" around
writing tests that lead to test suites that don't actually catch bugs, or worse, create them.

#### Why write a test for something that you already know works?

A few years ago, I watched the [Fireship tutorial on TDD with Jest](https://youtu.be/Jv2uxzhPFl4?si=OVVVh59W6oCYRDov&t=365),
where he used the example of implementing a stack with Javascript.
First, he wrote tests for the methods. Then, he implemented the functionality to make them pass. It was a great video on how to do
TDD, but I just couldn't understand the point.

The problem is, it's so obvious that the software he wrote works, that it makes no sense as an example.
It showed exactly **how** to do TDD, but it didn't properly justify **why** writing the tests is valuable.
After a novice engineer watches this video, why would they take the extra time to
implement tests in their projects if the value for writing tests hasn't been illustrated?

Around the same time, I was building a lot of projects in my last few years of school, and I kept seeing rhetoric like
["Software features that can’t be demonstrated by automated tests simply don’t exist"](https://softwareengineering.stackexchange.com/questions/33869/software-features-that-cant-be-demonstrated-by-automated-tests-simply-dont-ex)
and ["Test-driven development is a way of managing fear during programming."](https://www.oreilly.com/library/view/learning-test-driven-development/9781098106461/preface01.html)

But when I went to implement tests in my projects, my question always came back the same. Why would you write a test for something that you already know works?
In other words, **why would you need a test if you already verified that it works by looking at the output?**
It just didn't make sense from a value perspective - it felt contrived, like I was blindly following dogma.

I knew I was being naive, but I just couldn't reasonably justify spending the time and effort.
But when I look back on it now, I realize that my intuition wasn't totally wrong, just under-informed.

#### Exposure

In a CS curriculum, you do a lot of "contained" work. Small, individual projects or programming assignments. Usually, alone or with a partner.
These projects probably aren't anywhere near the scale of a production application, and are
usually short-lived - at most, a semester or two.

Like many things in life, procedures, terms, and justifications for best-practices in software engineering
can often-times only be understood by exposure. You have to experience the problem to understand the solution. That's why it's so
difficult for new engineers to break into the industry - they don't know what it means to be an engineer yet.

You can know a lot about programming paradigms, protocols, algorithms, and the particular
tech stack you've chosen, but you just can't understand the workflow of a software engineering team until you've
worked in one.

And it wasn't until I was fairly deep in my first role as a software developer,
building new features on a legacy codebase and experiencing countless bugfixes, refactors, outdated code, and deployment errors,
that I understood the value of testing. It took experiencing the consequences of both un-tested code and code with bad tests first-hand,
over many months and feature cycles, for me to to realize what they were talking about in those books and tutorials.

#### Software Changes

Here is the justification I wish I had heard when I was learning:

The reason you write tests *isn't* to prove that your code works *when you write it*. That's a side-effect, but it's not the main purpose.
The reason you write tests is to ensure that **as the software changes**, and as others (or, you) change the code
you wrote, that **both the individual units of functionality and the system as a whole still work as you expect**.

They key distinction here, and the one I don't think is emphasized enough, is that **software changes**. And when that change happens, unexpected things can go wrong.
While you can still verify that the code you wrote works by looking at the output, you don't always know if the code you wrote just so happened
to break something else. Writing tests allows you to have more confidence that as the software changes, the existing code still works.

I'll illustrate what I mean with an example.

Let's say you had a `UserProfileUtils` class, which has a method that returns formatted user data. Before, it just returned `id`, `name` and `email`.

```javascript
class UserProfileUtils {
  static formatUserData(userData) {

    return {
      id: userData.id,
      name: userData.name,
      email: userData.email
    };
  }
}
```

You add a new feature to return the user's formatted address based on their region, which requires a user's
region to be passed. This will be displayed in the user's profile page.

```javascript
class UserProfileUtils {
  static formatUserData(userData, regionSettings) { // Added regionSettings parameter

    return {
      id: userData.id,
      name: userData.name,
      email: userData.email,
      // Added formattedAddress which depends on 'regionSettings' input
      formattedAddress: this.computeRegionalAddress(userData.address, regionSettings)
    };
  }
  // added method to compute formattedAddress
  static computeRegionalAddress(address, regionSettings) {
    return `${address} (${regionSettings.format})`;
  }
}
```

When you use this in the `UserProfile` component, it works as expected:
```javascript
const UserProfile = ({ userData, regionSettings }) => {
  const formattedData = UserProfileUtils.formatUserData(userData, regionSettings);

  return (
    <div>
      <h2>{formattedData.name}</h2>
      <p>{formattedData.email}</p>
      <p>{formattedData.formattedAddress}</p>
    </div>
  );
};
```

Now, you put the code up for review, and it gets merged in to the `dev` branch. You start working on new features.

A few months go by, and it's time to do a major version release. As QA is going through manual E2E tests a few days before, they notice that
a new error has been raised. In another part of the app, a component called `AdminUserList` is returning a "cannot read properties
of null" error.

```javascript
const AdminUserList = ({ users }) => {
  const formattedUsers = users.map(user =>
    UserProfileUtils.formatUserData(user);
  );

  return (
    <div>
      {formattedUsers.map(user => (
        <div key={user.id}>
          <span>{user.name}</span>
          <span>{user.email}</span>
        </div>
      ))}
    </div>
  );
};
```

This causes panic to ensue, as the release deadline is tomorrow, and the bug gets assigned to you.
You haven't worked on the original issue in months, so you don't know how this error could have happened
(assume that the components, classes and methods would be significantly more complex in a real scenario).

After spending hours debugging, you realize that the change you made to include the formatted address in the user
profile is the problem, as `regionSettings` is now a required input to the `formatUserData` method, which `AdminUserList` is calling. This causes the null error,
as UserProfileUtils is trying to compute a value from a null input. This wasn't caught before
because it's in a very little-used part of the app, and no one had considered that changes to
the User Profile could have affected this.

If there had been an automated test to check that `AdminUserList` rendered what it was expected to,
and that test was run prior to pushing the new changes to `dev`, this error could
have been caught before the code was even pushed to code review. And because the output of that test
would have happened during the development of the new feature, it would have been significantly easier to find
the root of the problem.

```javascript
// What the test could have looked like
describe('AdminUserList', () => {
  it('should render list of users', () => {
    const mockUsers = [
      { id: 1, name: 'John', email: 'john@example.com' },
      { id: 2, name: 'Jane', email: 'jane@example.com' }
    ];

    // The error would have been raised here, instead of during manual E2E testing
    render(<AdminUserList users={mockUsers} />);

    expect(screen.getByText('John')).toBeInTheDocument();
    expect(screen.getByText('Jane')).toBeInTheDocument();
  });
});
```

By writing this test, you are proving that `AdminUserList` works as you expect it to. But that's not why the test is valuable.
The test is valuable because it ensures that `AdminUserList` **continues** to work as you expect it to over time. It gives
you the confidence that changes won't create unexpected work.

#### Writing Tests is Hard

Remember how I mentioned that when I was writing tests for my personal projects in school, I had an intuitive feeling that my tests weren't serving a point?
I still think part of that was correct. It's probably not necessary to write tests for a static webpage, or a small personal project that's only
constrained to a few files. UNLESS you foresee them becoming much larger, more complex, being worked on/updated over a long period, or incorporating multiple other engineers.

An interesting part of software engineering is group-think. When Kent Beck says you must write unit tests, suddenly it becomes
unacceptable to write code without them, and the gospel is spread far and wide.

I think this is problematic, because it encourages behavior of writing tests just for the sake of having them. One thing I've learned is that
**writing effective tests is hard**. It requires a lot of critical thinking, and forces you to understand your code, its relationships, and how it could change over time
at a very high degree.

Combine this with the pressure for 100% test coverage, release deadlines, and confusing legacy code, and you have a recipe for shortcuts, concessions, and bad tests.

Testing frameworks give you a lot of power over how your code behaves. One example is mocks, a particularly powerful tool in a testing framework.
While sometimes necessary in order to not test implementation details, mocks give you the power to assume that a method will always return a
specific result. But what if the method you're mocking changes? This could lead to a situation where you have a passing test, but the unit under test is actually failing.
In this case, you now have a test that gives you *false* confidence - which has the potential to be even worse than none at all.

#### On Value

It's important to know if your tests serve a purpose. You must ask yourself, **does this test have value?**
If you can't verbally say what it's testing, how it will prevent issues, and how it will remain resilient,
you should reconsider your approach. It's clear that testing is important, and that it must be done well to be effective.
But it's also clear that doing it well is hard.

As I researched, I realized that this is a topic with a lot of layers, a lot of opinions, and a lot of unanswered questions. And that's probably
why I found it frustrating in the first place. Because experts can be wrong, group-think exists, and pressure to write tests is high.
And explaining testing is almost as difficult as writing the tests themselves. There's a lot more to this topic than what I've shared here, but
hopefully this helps others looking for that justification that seems to be missing from a lot of the tutorials out there.
