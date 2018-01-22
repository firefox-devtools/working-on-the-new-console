# Working on the new console

We have been working on a new React+Redux based front-end for the console in Firefox DevTools for a while, and we're quite close to completion, but we still need to migrate a few tests so they work with the new front-end, and also use best practices.

The console is one of the oldest components in DevTools and that's why we're taking this opportunity to reduce technical debt.

## What needs to be done?

We want to make the integration tests in `devtools/client/webconsole/new-console-output/test/mochitest` work.

This is [the list of tests that need work](https://bugzilla.mozilla.org/buglist.cgi?bug_id=1405254%2C1404849%2C1404384%2C1405333%2C1408937%2C1408936%2C1408938%2C1405350%2C1405650%2C1405649%2C1405637%2C1405245%2C1406060%2C1408931%2C1403448%2C1405352%2C1405647%2C1405648%2C1405252%2C1403907%2C1408948%2C1404378%2C1403188%2C1403902%2C1408947%2C1401953%2C1404832%2C1408949%2C1408893%2C1404382%2C1406028%2C1405343%2C1404886%2C1408939%2C1403454%2C1404831%2C1404888%2C1403200%2C1408932%2C1404851%2C1408933%2C1405641%2C1408935%2C1401944%2C1403450%2C1406039%2C1401942%2C1403205%2C1408925%2C1401959%2C1405250%2C1408950%2C1405636%2C1404359%2C1406841%2C1406022%2C1408934%2C1404844%2C1404364%2C1404392%2C1408940%2C1401963%2C1404371%2C1401548%2C1405243%2C1404877%2C1408930%2C1408943%2C1404368%2C1405340%2C1408944%2C1405341%2C1404883%2C1408946%2C1408945%2C1403196%2C1408942%2C1404884%2C1408941%2C1406038%2C1404853&bug_id_type=anyexact&bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&list_id=13967960). You should attempt to work in one which is **not** assigned (i.e. status column reads `NEW`).

## Where are the files?

The old console front-end is in `devtools/client/webconsole`. The tests are in `devtools/client/webconsole/test`.

The new console front-end is in `devtools/client/webconsole/new-console-output`. The new tests are in `devtools/client/webconsole/new-console-output/test`. 
You will notice that the old tests folder had all of them in the same folder, whereas the new one divides the tests by type (e.g. testing the Redux store is separated from integration tests).

## How?

First try to run the new test. Run:

```bash
./mach test /devtools/client/webconsole/new-console-output/test/mochitest/{test-name.js}
```
where `test-name.js` is the name of the file containing tests that you want to work on.

If there are no errors when you run the test, that's great! The end of the output would be something like this:

```bash
Browser Chrome Test Summary
	Passed: 1
	Failed: 0
	Todo: 0
	Mode: e10s
*** End BrowserChrome Test Results ***
Buffered messages finished
SUITE-END | took 19s

Overall Summary
mochitest-browser: 3/3
```

If there are errors, they end of the output would look a bit like this:

```bash
Browser Chrome Test Summary
	Passed: 1
	Failed: 1
	Todo: 0
	Mode: e10s
*** End BrowserChrome Test Results ***
Buffered messages finished
SUITE-END | took 10s

Overall Summary
mochitest-browser: 1/2
TEST-UNEXPECTED-FAIL | devtools/client/webconsole/new-console-output/test/mochitest/browser_webconsole_time_methods.js | Uncaught exception - waitFor - timed out after 500 tries.
```

There's advice on how to fix the errors in the next section.

Supposing your test file has no errors and passes all the tests, the next step is to make sure the test is enabled in the `browser.ini` file in `devtools/client/webconsole/new-console-output/test/mochitest/browser.ini`.

Open the browser.ini file, and look for the file name of the test you're working on. You might encounter a section for the file, surrounded by brackets, like the following: 

```ini
[browser_webconsole_time_methods.js]
skip-if = true #	Bug 1404877
```

It tells you the test is disabled (it'll be skipped when running 'all the tests'). The bug number should provide a reason as to why the test was disabled.

In this case, if you go to our bug tracker and search for that bug number, you'll encounter that [it actually corresponds to the bug migration](https://bugzilla.mozilla.org/show_bug.cgi?id=1404877). Since the test is passing already, the last step to fix the bug is to remove this section from `browser.ini`, so the test is not skipped anymore.

So remove the two lines, and save the file:

```diff
-[browser_webconsole_time_methods.js]
-skip-if = true #	Bug 1404877
```

You should now be ready to [send a patch](http://docs.firefox-dev.tools/contributing/making-prs.html)!

## Fixing the tests

### Identify the new and old test files, and any additional support files

The general process to work in these tests is to first identify the new and the old test files. In general terms you should be able to find them easily as they have the same file name (they're just in different folders). But we might have used a new file name for the new test, if, for example, the old test file included a bug number on the file name. Do let us know if you can't find the counterpart!

Following on the example before, the old test file corresponding to `devtools/client/webconsole/new-console-output/test/mochitest/browser_webconsole_time_methods.js` is in `devtools/client/webconsole/test/browser_webconsole_bug_658368_time_methods.js`. We removed the bug number from the file name when writing the new tests.

A test might also load support files, and it is possible the support files also included bug numbers in the file name. To tidy things up, these need to be renamed to remove the bug number, and evidently you also need to update the file name reference in the test file itself.

`browser_webconsole_time_methods.js` uses a support file called `test-bug-658368-time-methods.html`. It needs to be renamed using your version control command (to avoid losing any history):

* With git: `git mv test-bug-658368-time-methods.html test-time-methods.html`
* With hg: `{TODO}`

Also edit the mention to the support file in the test file:

```javascript
// Old
const TEST_URI = "http://example.com/browser/devtools/client/webconsole/" +
                 "new-console-output/test/mochitest/test-bug-658368-time-methods.html";

// New
const TEST_URI = "http://example.com/browser/devtools/client/webconsole/" +
                 "new-console-output/test/mochitest/test-time-methods.html";
```

### Changes in syntax and helper methods

Following is a list of constructs and methods that replace what was used in the old tests. The new test helpers are simpler and tend to accept fewer arguments, and should also be more robust and easy to understand.

This is not meant to be exhaustive. Please collaborate if you think something should be added: fork this repository and send us a pull request! :-)

### head.js

When you don't know where functions are coming from, you can look at the `head.js` file in the each test directory. This file is included before each test is run, and it's where we defined helper functions such as `openNewTabAndConsole` which are used throughout the Console tests.

The `head.js` file in the new test folder has way fewer helpers than the old one, because we're trying to simplify, but if you feel something should be ported from the previous `head.js` file, by all means do port it (but try to adhere to the code conventions in the new `head.js`).

### Use the `async` keyword with `addTask`

Older tests used `add_task` with a generator:

```javascript
add_task(function* ...
```

New tests should use async functions instead of generator functions:
```javascript
add_task(async function(...
```

### Replace `loadTab` and `openConsole` with `openNewTabAndConsole`

Where you see a sequence like this: 

```javascript
yield loadTab(TEST_URI);
let hud1 = yield openConsole();
```

... you can replace it with a single helper function call:

```js
let hud1 = await openNewTabAndConsole(TEST_URI);
```

###  Use `BrowserTestUtils.browserLoaded` instead of `yield loadBrowser`

Replace:

```javascript
yield loadBrowser(gBrowser.selectedBrowser);
```

with

```javascript
await BrowserTestUtils.browserLoaded(gBrowser.selectedBrowser);
```

### Dealing with `waitForMessages`

`waitForMessages` sets a listener on an event which is emitted when a message is rendered on the console output, which means it works well if you can call `waitForMessages` before actually logging the message.

In some cases, we load a URL which does the logging so we can't really use `waitForMessages` as the event is probably already fired before we set up the listener. In those cases, we fall back to polling with `waitFor`, which takes a function and resolves when it is truthy:

Therefore you could replace:

```javascript
yield waitForMessages({
     hud: hud,
     messages: [{
       name: "text of the message",
	}]
});
```

with

```javascript
await waitFor(() => findMessage(hud, "text of the message"));
```

