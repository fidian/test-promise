Test-Promise
============

A simple, synchronous Promise object that is useful for tests.  Not completely [Promises/A+] compatible, but really close.  Also comes with useful inspection methods.

This is **ONLY** for testing code.  It uses the global function `spyOn` that is supplied by Jasmine.


Including
---------

First, add this to your project.  This library works in ES5, node, browsers and practically everywhere JavaScript runs -- including IE8.

Next, just use this promise library instead of your built-in one.  That may be a lot harder.  To make the swap easier, you should design projects around dependency injection.  I use [Dizzy] for node development.  Perhaps you need to make a special build and replace your other Promise library.  Maybe, just for testing, you allow the Promise object to be passed in?  The structure of your application is entirely up to you.

After that, you start making Promises!


Usage
-----

Let's suppose we are testing how your asynchronous code operates.  We want to prepare a Promise object with a resolved value, run your code, then get the resulting Promise.  In most cases and with external libraries mocked to return right away, we suddenly have synchronous code!

    describe("your object", function () {
        var TestPromise, yourObject;

        beforeEach(function () {
            TestPromise = require("test-promise");

            // For demonstration purposes only
            yourObject = {
                reverseAsync: function (promise) {
                    return promise.then(function (val) {
                        return val.toString().split('').reverse().join('');
                    });
                }
            };
        });
        it("now appears synchonous", function () {
            var endPromise, startPromise;

            startPromise = new TestPromise();
            startPromise.resolve(1234);
            endPromise = yourObject.reverseAsync(startPromise);
            expect(endPromise.status()).toEqual({
                success: true,
                value: 4321
            });
        });
        it("allows you to reject easily", function () {
            var endPromise, err, startPromise;

            err = new Error("fake error");
            startPromise = new TestPromise();
            startPromise.reject(new Error("fake error"));
            endPromise = yourObject.reverseAsync(startPromise);
            expect(endPromise.status).toBe(false);
            expect(endPromise.value).toBe(err);
        });
        it("lets you trigger success/failure when you like", function () {
            var endPromise, startPromise;

            startPromise = new TestPromise();
            endPromise = yourObject.reverseAsync(startPromise);
            expect(endPromise.status).toBe(null);
            startPromise.resolve("testing");
            expect(endPromise.status).toBe(true);
            expect(endPromise.value).toBe("gnitset");
        });
        it("even supports you grabbing the success/error handlers", function () {
            var startPromise;

            startPromise = new TestPromise();
            yourObject.reverseAsync(startPromise);
            expect(startPromise.then.mostRecentCall.args[0]).toEqual(jasmine.any(Function));
            expect(startPromise.then.mostRecentCall.args.length).toBe(1);
        });
        it("helps out with async functions in Jasmine 1.x", function () {
            var endPromise;

            runs(function () {
                var startPromise;

                startPromise = new TestPromise();
                startPromise.resolve("async works");
                endPromise = yourObject.reverseAsync(startPromise);
            });
            waitsFor(function () {
                return endPromise.status();
            });
            runs(function () {
                expect(endPromise.value).toEqual("skrow cnysa");
            });
        });
        it("assists with debugging", function () {
            var callbackSet, endPromise, startPromise;

            startPromise = new TestPromise();
            endPromise = yourObject.reverseAsync(startPromise);
            expect(startPromise.then).toHaveBeenCalled();
            expect(startPromise.children.length).toBe(1);
            callbackSet = startPromise.children[0];
            expect(callbackSet.promise).toBe(endPromise);
        });
    });


API
---

* `new TestPromise()` - Creates a new TestPromise.
* `new TestPromise(otherPromise)` - Attaches a new TestPromise to the result of `otherPromise`.
* `promise.then(success, failure)` - Works exactly as expected.


[Dizzy]: https://github.com/tests-always-included/dizzy
[Promises/A+]: https://promisesaplus.com/
