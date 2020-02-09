---
layout: post
title: Uses for the First Bus API
---

The other day I wrote about reverse engineering the First Bus API. Okay, so I did it, but what use cases could it have apart from just showing bus stops on a map (which is what is done by the First Bus web app anyway).

I demonstrated a quick command line application but again that's not super-useful. Could there be anything that I could do with it actually in practice that would be useful?

I have the <a href="http://workflow.is" target="_blank">Workflow</a> app for my iPhone which is a fantastic app for being able to automate common tasks on the iPhone. One of the other great things it can do is to let you run Workflows in Notification Centre, when you pull down on the top of the screen to see your Today view. Wouldn't it be really cool to be able to pull down Notification Centre and see the next bus? I though so too!

Workflow has a "Get contents of URL" action which submits an HTTP GET request on the specified URL. Okay, but the First Bus API needs a POST request. Could I adapt the little script I sampled the other day to accept a stop parameter on the querystring, make the POST request to the First Bus API and then return the next bus as text that Workflow could display.

Here's a little PHP script that does just that. It accepts one parameter on the querystring: `code=abcdef` and returns the next bus in human-readable format (Workflow can decode JSON but I made the decision to decode it on the server rather than in Workflow):

<script src="https://gist.github.com/lovattj/ace7a358b3c257b3adde693b19ea273d.js"></script>

Then, it's just a case of making a Workflow script to Get Contents of URL, hit the URL making sure there's a bus stop code parameter and then show an alert for the response!

The only limitation at the moment is the Querystring is hard coded, i.e. it can only display one stop. I'd have to duplicate the Workflow to change it. Or I could get Workflow to show some sort of pop up menu of my favourite stops!