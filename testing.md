# Testing

Recommendations and best practices for testing in Node (and general sw testing)

For any type of test (unit, acceptance, integration...) and for any implementation or tool of said test, there are certain rules it should follow.

Provide easy tracking of what is being tested. When naming a test make sure it includes what's being tested, under which circumstances and what's the expected behaviour. This will provide better understanding of what the test should do.

When implementing the test, follow the Given - When - Then pattern. Preconditions, action and result. Set up the conditions for the test, default values, mocks and such. Then execute the action, should be just one. Finally, assert the results.

``` gherkin
Feature: User model
  Scenario: Create already created user
    Given a user
    When the same user is created
    Then there is an error
    And the user is not created twice
```

``` js
// Keep tests independent
beforeEach(done => {
  User.collection.drop(() => done())
})
// descriptive names
describe('User model', function() {
  it('Should not create user when when already created', ()=> {
    // Given
    // mimic real values (name)
    const user1 = new User({name: 'Mike'})
    // When
    const user2 = new User({name: 'Mike'})
    // Then
    expect(user2).toBeNull()
  })
})

/**
 * User model - Should create user when valid name
 * User model - Should not create user when invalid name
 * User model - Should not create user when already created
 * ...
 * /
```

Keep language easy and understandable by non-engineering profiles (even more important for acceptance tests). For this try to keep the code clean. If testing an error, don't add try catch, use something like `expect(method).to.throw` to make it easier to read.

It's also important to define what needs to be tested. Sometimes it's better to start from the outside and test the system as a blackbox and then go deeper into the implementation.

Any test should mimic the real execution as much as possible. That's why when testing features that use external services it's better to stub or spy than to mock. Spies are clearer and closer to the real behaviour of the system. Also when providing values for tests try to keep them as close to reality as possible (e.g. real names, addresses etc).

Tests should be independent (as much as possible). Don't seed the db globaly before testing. Drop everything from there and have each test add whatever is necessary for it.

<https://www.npmjs.com/package/sinon>
<https://www.npmjs.com/package/node-mocks-http>
<https://github.com/goldbergyoni/node-chaos-monkey>

Aim for a 80% test coverage. Anything over that could be too expensive and won't provide more confidence.
> I would be suspicious of anything like 100% - it would smell of someone writing tests to make the coverage numbers happy, but not thinking about what they are doing.
<https://martinfowler.com/bliki/TestCoverage.html>

Don't forget linting. This could prevent many errors and it's fast to implement and run. Should be the first step.

<https://github.com/goldbergyoni/javascript-testing-best-practices>
