---
layout: post
title: Reverse Engineering the First Bus API
---
Sorry for not writing on here for some time, things have been busy and I haven't had as much time to dedicate to what I call personal coding projects as I'd like.

But the other day I saw something I thought I'd investigate a bit further. I live in the UK and am a regular passenger on First Buses in the town where I live, to get to work and generally get around. In some cities like London, the buses are all satellite tracked and every stop has information on when the next bus will be down to the minute. Unfortunately things in Norfolk aren't quite that advanced, but they're improving.

The local operator - the aforementioned First Buses - has recently started to launch real-time tracking which is a great improvement and most welcome. They've even made a nice little JavaScript web app at https://firstgroup.com/next-bus that will get your geolocation, show you the nearest stops on a map and then if you tap on one, show you the live countdown for that stop.

I like their little web app a lot, it's bookmarked on my iPhone and it's useful every day. But it's a little clunky and there are definitely some improvements that could be made. For example, as a regular passenger, I have my "usual" stops and it's a bit of a pain to have to go to the map, accept geolocation and tap the stop I want. You can't seem to set any favourites and have a quick way to get to them. Also, as nice as the web app is, I'd quite like a native app on my phone. And, because I only live 2 minutes from the stop, it'd be nice to see the next bus times on my computer, rather than having to reach for my phone.

As the web app has a JavaScript front-end, there must be a back-end somewhere accepting parameters and feeding it data. But there's no public API available. Could I reverse engineer the web app to access the raw data?

Safari helpfully lets you use Web Inspector on a connected iPhone, so I fired this up and started browsing around on the web app to try and get an understanding of the architecture. And it turns out that the raw data is easily accessible through a standard REST interface.

<b>Bus Stop Data</b>

Every bus stop has a unique code. But if you don't know the bus stop codes you probably want a way of finding all the bus stops in a radius and the metadata for each one. This is how the First web app's map works. Imagine that you want to draw a square on a map and you want the bus stops inside the square. You need the lat/long coordinates of each corner in order to make your square.

Once you've got those, just make a `POST` request to `https://www.firstgroup.com/getStops` with some URL-encoded form data parameters:

<pre>
bounds_0: [string]latitude,longitude of lower left hand corner
bounds_1: [string]latitude,longitude of upper left hand corner
bounds_2: [string]latitude,longitude of upper right hand corner
bounds_3: [string]latitude,longitude of lower right hand corner
</pre>

You'll get back a JSON-object with an array of stops found, including the all important `code` you need to actually get the bus data for that stop. Here's an example:

Request:
<pre>
curl -X "POST" "https://www.firstgroup.com/getStops" \
	-H "Content-Type: application/x-www-form-urlencoded; charset=utf-8" \
	--data-urlencode "bounds_3=52.673582,1.232411" \
	--data-urlencode "bounds_2=52.674577,1.232411" \
	--data-urlencode "bounds_1=52.674577,1.230169" \
	--data-urlencode "bounds_0=52.673582,1.230169"
</pre>

Response:
<pre>
{
  "stops": [
    {
      "code": "2900D1110",
      "sms": "nfoamjgd",
      "direction": "SE",
      "latitude": "52.673690796",
      "longitude": "1.231520057",
      "name": "stop adj Carter Road"
    },
    {
      "code": "2900D1111",
      "sms": "nfoamjgj",
      "direction": "NW",
      "latitude": "52.673648834",
      "longitude": "1.231179953",
      "name": "stop opp Carter Road"
    }
  ],
  "stopcount": 2
}
</pre>

<b>Next Bus Data for an individual stop</b>

So now we can make another `POST` request to `https://www.firstgroup.com/getNextBus` with just one URL-encoded form parameter, the `stop` code:

Request:
<pre>
curl -X "POST" "https://www.firstgroup.com/getNextBus" \
	-H "Content-Type: application/x-www-form-urlencoded; charset=utf-8" \
	--data-urlencode "stop=2900D1110"
</pre>

Response:
<pre>
{
  "atcocode": "2900D1110",
  "smscode": "nfoamjgd",
  "request_time": "2016-07-18T09:52:50Z",
  "departures": {},
  "source": "VIX",
  "times": [
    {
      "ServiceRef": "0",
      "ServiceNumber": "28",
      "Destination": "Castle Meadow",
      "Due": "2 mins",
      "IsFG": "Y",
      "IsLive": "Y"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "29B",
      "Destination": "Castle Meadow",
      "Due": "11:03",
      "IsFG": "Y",
      "IsLive": "N"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "X29",
      "Destination": "Norwich City Centre, Bus Station",
      "Due": "11:20",
      "IsFG": "N",
      "IsLive": "N"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "28",
      "Destination": "Castle Meadow",
      "Due": "29 mins",
      "IsFG": "Y",
      "IsLive": "Y"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "28",
      "Destination": "Castle Meadow",
      "Due": "11:36",
      "IsFG": "Y",
      "IsLive": "N"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "X29",
      "Destination": "Norwich City Centre, Bus Station",
      "Due": "11:50",
      "IsFG": "N",
      "IsLive": "N"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "X29",
      "Destination": "Norwich City Centre, Bus Station",
      "Due": "12:20",
      "IsFG": "N",
      "IsLive": "N"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "X29",
      "Destination": "Norwich City Centre, Bus Station",
      "Due": "13:20",
      "IsFG": "N",
      "IsLive": "N"
    },
    {
      "ServiceRef": "0",
      "ServiceNumber": "X29",
      "Destination": "Norwich City Centre, Bus Station",
      "Due": "14:20",
      "IsFG": "N",
      "IsLive": "N"
    }
  ],
  "stop": {
    "code": "2900D1110",
    "mark": "1",
    "sms": "nfoamjgd",
    "direction": "SE",
    "latitude": "52.673690796",
    "longitude": "1.231520057",
    "indicator": "adj",
    "name": "stop adj Carter Road",
    "locality": "Drayton (Norfk)",
    "qualifier": "Norfolk"
  },
  "fromTraveline": "18/07/2016 10:52"
}
</pre>

Then it is simply a case of iterating through the `times` array to get the live countdown times for each bus!

Unfortunately it doesn't appear that every bus is live tracked right now - some routes are a bit better than others. You can check the `IsLive` field for this.

So armed with this data, it should now be easy to make some sort of mobile app or desktop script to give information on the next buses. Or even some sort of notification system. Or whatever! With open data you can do whatever you want!

Here's a super quick and dirty command line PHP example that gives the next buses for your favourite stop:

<script src="https://gist.github.com/lovattj/903f1457b00c3e582a7cd6e036bd1eaa.js"></script>