---
layout: post
title: Resolving All the Promises in AngularJS
date: '2014-10-23 12:48:20'
---

I frequently read that people are "making the migration" from "callback hell" to promises in JavaScript.  Honestly, I have not been doing JavaScript for so long to have really experienced "callback hell"...but I am embracing promises.  In my more complicated AngularJS pages in my Rails app (we still do mainly server-based routing and page-rendering with one or more AngularJS controllers on most pages) I recently encountered the situation in which I needed several asynchronous server requests to resolve before I could perform some logic.  Thankfully, a co-worker more experienced in JavaScript showed me a solution.

[AngularJS docs on $q](https://code.angularjs.org/1.2.26/docs/api/ng/service/$q)  If you scroll down to the bottom of that page, you'll noticed that there's an `all(promises)` method.  We often have AngularJS services provide the promises along with other methods that do "data crunching" on the resolved data, so this is what we were able to do...

      $q.all([
        templatesQuery.$promise,
        Account.projectPriorities({id: id}).$promise,
        CampaignService.getCampaignsPromise()
      ]).then(function(data) {
        var templates = data[0];
        var topThreeProjects = _.first(data[1], 3);
        var campaigns = data[2];

I realize this is not beautiful code (I'm still traveling down the path of getting things functional and learning "beautiful" as I go), but it (hopefully) demonstrates how you can effectively wait until multiple promises resolve and then get the data from each promise.

Comments are welcome for improved approaches or anything to clean up the bit that I've written.