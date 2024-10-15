# Homework 5: Clean Architecture User Login System Example

## Assignment Preamble

In this homework, you will work with a program designed using Clean Architecture.
The program initially has three complete use cases: signup, login, and change password.

The assignment is split into two phases. Phase 1 should be completed individually ahead of your next lab;
you may work with other students, but make sure to follow all steps yourself and push your own code to your
fork of this GitHub repo.

Phase 2 will be completed with your project group during your next lab.

During this assignment, you will:

* Practice exploring, understanding, and working with a program with a layered architecture.
* Implement a unit test for a use case interactor. 
* Continue practicing with git in preparation for the group project.
* Become comfortable with the idea of dependency injection.
* See an example of using an API similar to what you will do in your group project.
* Add a use case to the program.

### Timeline

* Tue 15 Oct: Phase 1 of this assignment is posted
* Fri 18 Oct: aim to complete Phase 1 ahead of your Monday lab; asking for help as needed if you get stuck
* Mon 21 Oct: Phase 2 will be posted; it will have you complete a team activity involving this code during the lab
* Fri 25 Oct: individual code for Phase 1 and group code for Phase 2 is due on MarkUs;
  make sure the provided self tests are all passing once they are made available!

# Phase 1 [for credit]

## Task 0: Fork this repo on GitHub
**To get started, fork this repo on GitHub and then make a clone.**

Open the project in IntelliJ and make sure you can successfully run `app/MainWithDB.java`.
Note: you may need to set the Project SDK in the `Project Structure...` menu, and possibly
also manually link the Maven project.

## Task 1: Understanding the Program

Try the signup, login, and change password use cases by running the program.
Notice that the "Log Out" button doesn't do anything when you click it — to test whether
change password worked, you'll need to quit and rerun the program. Note: some other buttons,
like the "Cancel" buttons, are also not fully functional.

### Packaging

Explore the package structure in `src\main\java\`. There are packages for
the CA layers — `view`, `interface_adapter`, `use_case`,
`entity`, and `data_access` — as well as `app`, a package for the main program
and a couple factories.

Two of these packages, `use_case` and `interface_adapter`, have subpackages for each of the
two use cases: `login` and `signup`. None of the Interactor and Interface Adapter code is
shared between use cases.

Several packages _don't_ have subpackages: `data_access`, `entity`, `view`, and
`app`.

* The same View object may have several Use Case buttons inside a single `JPanel`,
  so separating by use case isn't possible.
* Entities represent the data from the problem domain that all Use Cases manipulate.
* The Data Access layer is responsible for saving and reading the Entities.
* The main application is responsible for building the CA engine and starting the GUI.
  After the engine is built and the UI becomes visible, the program is driven by the user and
  the main program has nothing left to do.

### A note on English: verb phrases vs. nouns

"Sign up" is a verb phrase and "signup" is a noun. That generalizes: "check in"
vs "checkin", "log in" vs "login". Two words for the verb phrase, 1 word for the
noun phrase.

For example, to complete a login, you need to log in. (Say it out loud. They sound different.)

In "the login process", "login" is a noun acting as an adjective to describe
"process". "Basketball coach" is another example of this English construct.

### Comparing the signup and login code

Let's compare these two use cases.

#### Controllers

In IntelliJ, find `LoginController` and double click it to open it.

Now right-click on `SignupController` and select `Open in Right
Split`. When you do, you will see the two controllers side by side.
They are identical in structure, differing only in the
details.

**This is powerful:** most controllers will look similar. Most presenters
will look similar. Most interactors will look similar. Any programmer who
learns about CA will have a good understanding of
any controller, interactor, and use case.

**Thought question**: open the L-shaped CA diagram and compare the types
in `LoginController` to the diagram. You'll notice that both controllers
have an Input Boundary that is _injected_ in the constructor, both create
Input Data from the parameters in method `execute`, and both
of them call the Use Case execute method, passing in the Input Data. All
the arguments for the `execute` method come from the View.

#### Presenters

Open `LoginPresenter` and `SignupPresenter` side by side. Both have
View Model variables and a View Manager Model that are injected into the
constructor.

Both also have a `prepareSuccessView` method that the Use Case calls
when it is complete. The job of this method is to update the View Models.
Read the code for this method in either presenter.

Notice that both of the `prepareSuccessView` methods mutate the state of a View Model
and call `firePropertyChanged` to alert the relevant View Model that
the state has changed, and ends with code that tells the View Manager Model
what the active View should be.

Both Presenters also have a `prepareFailView` method to handle errors.

#### Interactors

Now compare `LoginInteractor` and `SignupInteractor` side by side. (You can drag
tabs around if you like.)

**Thought question:** Why doesn't the `LoginInteractor` have a `UserFactory`
but `SignupInteractor` does?

A Controller calls the `execute` method in an Interactor to start processing
the Use Case data. When it's done, the Interactor tells its Presenter what the result
is, and the Presenter puts it into the View Model and tells the View Model to change
which View is showing.

Compare `LoginInteractor` and `SignupInteractor`. Notice that both
use an Input Boundary, Input Data, Output Boundary, and
Output Data. Both also have a Data Access Interface, which is what the Interactor
uses to get data relevant to the Use Case.

The Data Access Interface and Output Boundary are injected in the constructor.

Method `execute` is passed Input Data to process. The Interactor fetches the
appropriate piece of persistent data from the Data Access Interface, does some error checking
to make sure the Use Case makes sense, and then does whatever the Use Case
is supposed to do. Notice the Interactors both end by creating Output Data and
telling the Presenter to present it.

## Data Access Object

There are three DAOs in package `data_access`! All three implement the Data Access Interface
from the use cases. The Use Case code works with any of them.

* Class `FileUserDataAccessObject` manages data storage and retrieval in a
CSV file, and also keeps the data in a `Map` for easier access. This temporary storage
is called a *cache* of the information in the file.

* Class `DBUserDataAccessObject` uses okhttp to use an API, working with JSON data. Your team
  might want to refer to this when you do your API work. This API is similar to the one from the third lab;
  you can read its documentation
  [here](https://www.postman.com/cloudy-astronaut-813156/csc207-grade-apis-demo/documentation/fg3zkjm/5-password-protected-user).

* Class `InMemoryDataAccessObject` doesn't save the user data to any kind of file at all,
  and is intended to be used by the unit tests.
  * It's also simple to write, which means that you can start
    programming your Use Cases before you even have the details of data persistence worked out.

### Task 1: Switch from DBUserDataAccessObject to InMemoryDataAccessObject [for credit]

The program is currently connecting to an external API — the one
from lab 3, but a different API endpoint for storing a password protected
user. For this activity, we want to be able to develop and test our
code independently of this external API, so we will use the in-memory
version of the program's DAO instead.

1. First, make a branch named the first part of your UofT email address, everything before the `@`.
For example, if your email address is `paul.gries@mail.utoronto.ca`, then the branch name would
be `paul.gries`.

Make sure you switch to the new branch.

In the terminal, this would look like below, but replaced with your own information:
```
git branch paul.gries
git switch paul.gries
```

2. In the `app` package, make a copy of `MainWithDB.java` and call it
   `MainWithInMemory.java`.

3. To change the type of DAO, you need to edit the place where it is
   instantiated. To find it, right-click on `DBUserDataAccessObject` in the Project tree and
   select `Find Usages`. There should be several results. Click the one where
   the new instance is created in `MainWithInMemory` and go to the code.

4. Now change `DBUserDataAccessObject` to `InMemoryUserDataAccessObject`. That should
   only occur twice in class `MainWithInMemory`. The `InMemoryUserDataAccessObject` constructor
   does not need a `UserFactory` because it doesn't need to convert from a raw storage format
   (strings and numbers) to `User` objects and vice versa, unlike `DBUserDataAccessObject`.

We'll work with `InMemoryUserDataAccessObject` for the rest of the assignment.

Try running the `MainWithInMemory` program to make sure that it works.

5. Add and commit `MainWithInMemory.java`. Push your code to GitHub; making sure that it is on the branch you just made.

## Task 2: Editing the Login Use Case [for credit]

Here's the heart of the login use case code. All the rest is just input validation and error checking.

```
User user = userDataAccessObject.get(loginInputData.getUsername());

LoginOutputData loginOutputData = new LoginOutputData(user.getName(), false);
loginPresenter.prepareSuccessView(loginOutputData);
```

Notice that the code gets the user information from the DAO,
puts the username into the Output Data, and tells the Presenter
to prepare a success view with that username.

### Task 2.1

**The DAO doesn't currently keep track of which user is logged in!**
You'll fix that now by adding a method to the Data Access Interface.

First, check that you're still on your branch. You can do this
by typing `git status` and reading the first line of output. Use `git switch` if you need to.

1. Add one more line before the Output Data is created:
```
      userDataAccessObject.setCurrentUser(user.getName());
```
IntelliJ won't like that: the `setCurrentUser` method doesn't exist. Get IntelliJ to generate
the method for you. Notice that it puts it into the `LoginUserDataAccessInterface`. That causes a problem with
the three implementing classes: all of them need to be updated.

Let's quickly make the File and DB ones compile; that's just so we can run the program. They won't be used.

2. Open them and let IntelliJ generate the missing methods. Leave the method bodies empty. 

3. Now focus on class `InMemoryUserDataAccessObject`. Open it and get IntelliJ to fix it so that it compiles,
   which creates an empty method. In that method, type this:
```
      this.currentUser = name;
```
That causes an error. Let IntelliJ fix it for you by introducing an instance variable.
Let's say `null` means that nobody is logged in, so we leave the default value.

4. Add and commit your work, then push to your branch on GitHub.

* * *

### Aside: Session Cookies, Yum!

This article explains [how passwords are often managed](https://blog.bytebytego.com/p/password-session-cookie-token-jwt).

Here's a short explanation: validation in a web browser involves session IDs. Once a user
logs in, the system
sends a session ID to the web browser, which saves it in a cookie. A cookie is
a little file that your browser creates to manage login status and other
kinds of information. In subsequent requests, the browser will include the session ID
so that the server can authenticate the request.

* * *

### Task 2.2: Update a unit test

Now that you've added some new logic to the code, we should update our login tests to
verify that our code successfully stores the current user.

Open `src/test/java/use_case/login` and edit `LoginInteractorTest`. We want to add a new test for
the login status to make sure that when someone logs in the user is recorded as being the current user.

The following sequence of steps summarizes how one might test this code:

* add a user to the DAO 
* check that nobody is logged in
* execute the login use case for the user
* check that the user is logged in

1. **This sounds a lot like `successTest` in that test file! To get started, copy and paste that method and rename the new method
as `successUserLoggedInTest`.**

The method starts by creating the Input Data and saving it in the repository. Nobody is logged in.

Then the success Presenter is instantiated from an anonymous class that implements interface `LoginOutputBoundary`.
Method `prepareSuccessView` is called at the very end of the Interactor code when the use case is complete.

2. **Update the `assertEquals` call to assert that `"Paul"` is the return value of `userRepository.getCurrentUser()`.**

3. **Method `getCurrentUser` doesn't exist yet, so IntelliJ will yell at you. Get it to fix it for you by
   creating the method. This gets added to `LoginUserDataAccessInterface`. Now all implementers have to have it
   (just like before with the `setCurrentUser` method). Get IntelliJ to add that method to all three implementing classes.**

  - For `DB` and `File`, you can leave the generated code as it is.

  - **Fix `InMemoryUserDataAccessObject` so that the new method returns the current username as a `String`.**

**Thought question:** We've added a getter because the test needs it, not because the program needs it. Is this bad?

4. **There's one more thing to check: before the use case is executed, the current user should initially be `null`.
Add an `assertNull` call just before the `interactor.execute(inputData);` call to make sure this is true.**

Now run the tests. All of them, including your new one, should pass.

**Thought question:** Several of the tests have similar code to set them up; based on what we
learned about Junit, what might we do to improve the efficiency of our testing structure?

5. Run Checkstyle with the provided `mystyle.xml` configuration to make sure you didn't introduce any style issues.
   Fix any style errors.

5. Add and commit your work, then push to your branch on GitHub.

And you're done with Phase 1!

We encourage you to continue to explore the code and consider
trying some of the extra practice listed below in preparation for your project coding.

**Submission instructions for what you need to submit on MarkUs will be posted later.**

## Phase 2
The instructions for phase 2 will be posted as part of the October 21st lab activity. Each group
member will be asked to complete a small amount of code which will then be combined to implement
a complete use case in the program! This will be a useful team building activity and should
help you get a sense of what kind of code you will be writing in your group project over the
rest of the term!

## Extra Practice

You likely noticed that some features of the program aren't working yet.
For example:
- the `Log out` button doesn't work once a user logs in
- the `Cancel` buttons don't actually cancel anything
- the program isn't doing checks for password length or username requirements
- the program doesn't have an overall menu to allow the user to choose what to do once they log in.
- and many more!

If you feel you need more practice with Clean Architecture before getting started on
the project with your team, we encourage you to try adding some of these other bits
of functionality to this program.
