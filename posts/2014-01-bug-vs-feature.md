<!--
.. title: Bug vs feature
.. slug: bug-vs-feature
.. date: 2014-01-29 12:02:38 UTC
.. tags:
.. category: dev
.. link:
.. description:
.. type: text
-->

We've had a lot of discussions about what should go as a "bug" and what should be a "feature" in our tracker. Turns out there's quite a big gray zone here.

<!--more-->

Example: a website has a table with AJAX-based pagination. Navigating to an entry detail and clicking "back" doesn't bring you to the same page of the table.

The project manager considered it a bug - "back" doesn't really go back, so the navigation is broken and it's a serious user experience problem.

The developer, however, knew that the way to have that working is to assign a unique URL to each page of the table and use the history API to update the URL in the browser upon navigation, so that the back button really goes back.

Nothing was essentially broken within the code, nothing needed a proper "bugfix" or "debugging"; the way forward was to do some maintenance and introduce a new piece of technology to the UI. Doesn't that sound more like a feature than a bug?

It's a communication problem; a typical situation where mixing abstraction levels causes confusion. From the product perspective, it's a clear bug - the behaviour is different from expected and causes some kind of harm to the user. But when you wear the technical hat, you see that the action steps don't resemble bugfixing as understood by programmers.

Another real life case: We installed Raven.js to be notified about Javascript errors throughout our site. But our scripts are concatenated and minified on production, so the tracebacks we see in Sentry aren't readable. Action step: change the Javascript compression toolchain, replace the minifier with another one that happens to support source maps and configure Sentry to read them properly. Bug or feature? Technically, again - it's nowhere near bugfixing.

---

There's two distinct pieces to information to each issue: what is it about (new feature? system behaviour other than expected?) and what are the action steps to fix it (write code? debug? change your infrastructure? all of them?). The proper question to ask here is: Which of these would be useful for issue taxonomy?

I ended up deciding that it's the project perspective that matters - if there's a problem that is hurtful to the product, it should get priority regardless of the fix being coding, debugging, refactoring or changing some colours.

It's also noteworthy that "bugfix vs feature" is not a dichotomy - a lot of issues don't fit here, like design and maintenance activities, so there's still plenty of space to invent some taxonomy.
