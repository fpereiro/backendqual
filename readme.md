# The elements of backend quality

In this document I summarize four elements that are (in my opinion) essential for writing and maintaining quality backends for web applications. These elements are quite independent of your choice of architecture, programming language or framework.

## What is backend quality and why is quality important?

For our purposes, a good definition of a quality product is *something that replicably fulfills the expectations*. A quality backend, therefore, is a piece of code (or rather, a web service) that does exactly what it is supposed to do. This might not sound very exciting, but I believe it represents a world of difference, for both your users and your devs.

A quality backend enables a solid experience for your users - the best designed user interface cannot provide reliability if the backend lacks in quality. A backend designed with quality, in combination with a properly designed interface, can provide a great experience for your users. This will stabilize your user base and provide a strong foundation for further growth of your product.

Almost as importantly, a quality backend vastly improves the experience of your devs (and if you're an indie maker, your own experience while working, which is the lifeblood of your product). Quality has two positive effects on devs: 1) it vastly reduces the friction to set up, develop and maintain a product; and 2) it raises the bar to encourage and require quality work. Working with a quality backend eliminates a lot of unnecessary problems, while raising the bar for those essential things that make a quality product in the first place. This is reminiscent of the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory).

Quality is not just a nice-to-have property of a backend; it's essential for both the users and the makers of a product. Building a backend that does what it's supposed to do, reliably, can take you far. It is my sincere hope that it does so and that's why I'm sharing this document with you.

## Four elements of backend quality

Based on [my experience](https://github.com/fpereiro/backendlore) writing backends for web applications, I offer the following four elements of backend quality. I believe these to be the fundamental four; while they are not exhaustive, they are essential for the creation of a quality backend, in that the lack of even one of them will very likely compromise quality, whereas the presence of all four will very likely ensure quality.

### #1: thorough validation of the inputs of each of the endpoints of the backend

A (web) backend exposes a certain number of HTTP routes to requests by clients. Many (but not all) of these routes receive input from the client, in the form of query parameters, request bodies or (much less often) request headers.

The first element of backend quality is to thoroughly validate these inputs. Invalid input should be summarily rejected with a `400` code and an accurate error message. This is the case even if your backend only receives requests from a client which you fully control.

Strictly validating inputs have the following effects on the quality of your backend:
- It forces you to specify what are the possible inputs for an endpoint. This specification process makes you notice corner cases or exceptions which are not obvious in the first iteration.
- Once the validations are in place, you eliminate an entire source of bugs, which are invalid inputs that throw off the implementation of a given endpoint. This also allows for more succint implementation code that can rely on inputs conforming to certain specific shapes.
- The accurate error messages inform your client of invalid inputs, which helps to vastly reduce debugging time - particularly so since errors across the wire are more time-consuming to determine.

The following are examples of what I mean by thorough validation:
- If you're expecting an object with some keys and values, don't just validate the values, but also check that the object contains no extraneous keys.
- If certain values should not be present when others are, duly place a conditional that checks that inconsistent values are not present simultaneously.
- Don't just check for the type of things, but also for their value - for example, if your ids are positive integers, make sure that ids are integers larger than 0; or if a certain property can be `'a'`, `'b'` or `'c'`, check that it actually is one of those, instead of just looking for a string.

Internal functions don't need to be validated, in my opinion, but validating them will not - in my view - decrease the quality of the backend. It is, however, critical that you *validate the inputs of everything that you expose*. If you build an internal tool/dependency that exposes a certain number of functions, those functions should thoroughly validate their inputs (even if this entails a performance hit). And if you develop [internal services](https://gist.github.com/chitchcock/1281611) that provide functionality to your main API over the wire, those internal services must thoroughly validate the inputs of the endpoints that they expose to your API.

Distinguishing normal from abnormal input is one half of the principle of [auto-activation]; auto-activation is one of the two main pillars of the [Toyota Production System](https://en.wikipedia.org/wiki/Toyota_Production_System).

### #2: thorough and zero-tolerance end-to-end tests of all the endpoints of the backend

Once that all the endpoints have valid inputs, it's time to test them, directly and thorougly. In the case of a web backend, this entails performing a sequence of HTTP requests and expecting certain results.

The testing consists of two parts: first, the validations themselves must be tested, by sending invalid payloads of different shapes. The number of possible payloads might be infinite, so a brute force approach won't do. I also strongly discourage you from using a random component in your tests, since that will make them hard or even impossible to replicate. A way to thoroughly (yet succintly) test a set of validations is to target each of them in the order that they're implemented. For example, if an endpoint checks first the validity of an id, then that of a filter, first create payloads that violate the requirements for the id; then send payloads that contain valid ids but invalid filters.

A very thorough test would also check that the error messages returned by the API are in fact correct. I haven't gone that far yet but it might be worth it.

Once all the validations of an endpoint are tested, it's time to send payloads to cover all of the valid use cases. Again, the total amount of valid payloads might be infinite, so you must again determine a list of cases that can cover all potential uses. The structure of the implementation of the endpoint will be helpful to do this: if a certain endpoint supports (for example) two different types of queries, each with its own sublogic, each of the query types should be tested separately. Another example is that, within a certain use case, a property might be present or absent, in which case you should test both for its presence and absence bringing back valid values.

In general, I find that tests with invalid input are isomorphic to the validations, while tests with valid input are isomorphic to the implementation of the endpoint.

This sounds like hard work, and it is. But it's going to be much less work than not doing it in the first place, for the following reasons:
- It's less work to find the bugs early in the development of a feature than later on, when the database consistency is compromised.
- If the backend is sufficiently debugged, the development of the client is going to be much easier - most bugs can be assumed first to lie in the client.
- By doing most of the debugging in the test suite, you expose your users to fewer bugs.
- By having a full test suite, it is less likely that new features will introduce bugs to the existing ones - if you introduce a bug onto an existing feature, the tests for said feature will warn you.

What about errors in tests? I highly recommend that you make your test suite zero-tolerance: if a test fails, the entire test suite fails and stops. This is the other half of auto-activation, not tolerating invalid output. This has the following positive consequences:
- The standard for quality is raised; there's no such thing as an endpoint that sometimes or mostly works.
- If each endpoint works well, you vastly reduce the probability of complex bugs which emerge from the interaction of small bugs in different parts of the application.
- If there are unreliable parts on your application stack, they will stick out like a sore thumb and you'll fix them or remove them altogether. Backends should be reliable and create reproducible outputs.

I cannot fully sidestep the debate on unit tests vs. integration tests vs. end-to-end tests. In my minimalistic, insular and radical view, end-to-end tests are the only essential tests for a backend. Unit tests might pass with flying colors but bugs might still lurk in the space between the tested function and the usage of said function by an endpoint. This is particularly so if a unit test uses a mock layer to represent the database, the filesystem, or another service. The client doesn't care whether parts of an endpoint, or the entire endpoint in isolation, work well. The client cares (rightly) about the endpoint as a whole functioning correctly. If you order pudding, you don't care whether the cook used the right amount of flour; you care about how the pudding tastes. A test suite that checks the correctness of invalid and valid payloads for an exposed endpoint is a much stronger argument for the correctness of a backend, since the definition of a correct backend is one that processes correctly both invalid and valid inputs.

I don't want to discourage you from writing unit tests (or upsetting you regarding testing preferences, for that matter). All I can say is that I don't believe there's a substitute for testing the endpoints directly.

### #3: A replicable setup process.

To set up the app (both for local development and for production), there should be a deterministic, replicable process for doing so. This process should cover both the provision of the environment where the app runs (including operative system, databases, file system and linkages to external services) and the app itself.

Whether this is done by hand, through a bash script, or through a vastly more sophisticated pipeline involving builds, push notifications and containerization, is a related but different question. You can achieve a replicable setup process running commands by hand; and you might have state-of-the-art tools that, because of the way they're used, generate an inconsistent or broken environment or app. What I consider important is that *everything should be included explicitly in your process*. In other words, an entity (be it a human or a script) following the script should end up with a working version of your app, without any gotchas whatsoever. There should be no need to ask others for a particular piece of lore on how to get a certain library to run, or where to find a required environment variable. The one exception for these are application secrets for interacting with external services, which is reasonable to keep outside of version control.

I strongly discourage you from making your setup process reliant on black boxes or artifacts. These can be, for example, old application builds which were created with commands that are now lost; or database dumps that contain structural information about the database, not just mere data. These builds or dumps contain critical but unspecified data, and while strictly replicable, are fickle (especially if they rely on old versions of tools that eventually need to be upgraded). Your process should be able to construct the app in full, minus external dependencies (which have their own build process, if they're built with quality) and the data on the database.

A replicable setup process provides the following advantages:
- Minimize the onboarding time for new devs.
- Remove or minimize bugs related to the environment in which the app runs.
- Increase the maintainability of the app, especially in the case it is deployed again after months or years of inactivity.

### #4: An unified funnel for errors.

Any abnormal activity in the app should be noted and sent to an unified funnel for errors. Instances of abnormal activity are:

- Uncaught exceptions.
- Caught exceptions that were not expected.
- Requests that are answered with a 4xx or 5xx code.
- Queries that take an unusually long time.
- Errors when reaching databases, filesystems or external services.

All errors will be sent to a funnel, which can be as simple as a log file, or can be as sophisticated as a set of external services that receive, manipulate, aggregate and fire notifications when certain items are received. The essential thing is to note all the errors and send them to the same place, with as much background information as possible.

I advocate zero-tolerance to abnormal activity: there should be no known errors or warnings that are tolerated but might hamper the correctness of the backend. Every event can only be error or an expected outcome, never both.

By noting errors and sharply distinguishing from normal operations, errors are fixed faster and new errors (or old errors that were lurking behind the surface) appear much clearer. Eliminating the sources of error must be the highest priority for those working on the backend. Errors are, after all, unexpected outcomes, which are the very opposite of quality according to our initial definition.

Errors should have different levels of criticality; errors above a certain level should trigger notifications for those maintaining the backend. Alerts of a critical nature should never be "normalized" by those who receive them - if you get used to receiving alerts about irrelevant things, you might miss a critical alert. You will very likely have to put protocols in place to turn off or qualify alerts during production deployments (where there might be some downtime), or when reporting untargeted security attacks on your app that haven't been successful (which happen constantly to servers exposed on the internet).

## Conclusion

To summarize, I believe the following four elements to be the essential features of a quality backend:

1. Thorough validation of the inputs of each of the endpoints of the backend
2. Thorough and zero-tolerance end-to-end tests of all the endpoints of the backend
3. A replicable setup process.
4. An unified funnel for errors.

Why are each of these important? #1 defines the domain of your endpoints; #2 vastly increases the probability that the backend is correct (but doesn't prove it!). #3 enables the backend itself to be replicable, either locally or in production. #4 enables you to find existing or new errors as quickly as possible, and more importantly, to develop a very low tolerance for them.

I left out many things that I still consider necessary and desirable in a backend, but are not strictly essential for ensuring its quality:

- Conceptual integrity.
- Documentation.
- Simplicity of implementation.
- Performance.

Notice that my recommendations don't directly determine any of the following aspects on how you write a backend:

- Microservice vs monolith.
- Using many dependencies vs writing self-contained code.
- Self-hosting vs hosting on the cloud.
- Using external services vs not using external services at all.
- Server vs serverless.
- Choice of programming language.
- Choice of paradigm.
- Strict or loose typing.
- Test-driven development vs writing the tests later.
- Development methodology (Agile, Waterfall, Cowboy).

I also find these recommendations to be applicable for backends of any size and scope; from small backends with a single developer to massive web services - though, admittedly, my experience with the latter is very limited.
