---
layout: post
status: publish
published: true
title: Tracking a vehicle in React Native using Expo's MapView and the lean approach
header_image: '/assets/images/samuel-foster-362611-unsplash.jpg'
image_credit:
  handle: samuelfoster
  name: Samuel Foster
author:
  display_name: marcelkalveram
  login: marcelkalveram
  email: marcel.kalveram@googlemail.com
  url: http://marcelkalveram.com
author_login: marcelkalveram
author_email: marcel.kalveram@googlemail.com
author_url: http://marcelkalveram.com
date: '2018-05-08 11:09:00 +0200'
date_gmt: '2018-05-08 11:09:00 +0200'
tags: []
comments: []
---

Recently, I had the pleasure to experiment with maps and API integration in React Native while working on the iOS app for <a href="https://uncovercity.com/" target="_blank">uncovercity.com</a>.

The challenge that turned out to require a very lean approach can be summarised very easily:

1. a car (ordered via a ride-sharing app) picks you up at a specific location and takes you to a surprise dinner at an unknown venue.
2. when your dinner is over, you can use the app to order a ride home.

What made the challenge even more interesting was that we were forced to test the ride-sharing part of the app in production from the start, so we were constantly in touch with drivers and sitting in fancy cars with a phone in our hands and a computer on our laps.

## The challenge

To make the experience as pleasant and predictable as possible for the user, we implemented a live map showing the current position of the vehicle when it's underway, for both the pickup and the return journey.

<p class="images clearfix">
  <img src="/assets/images/screenshots/uncovercity-pickup.jpg" />
  <img src="/assets/images/screenshots/uncovercity-search.jpg" />
  <img src="/assets/images/screenshots/uncovercity-return.jpg" />
</p>

In the screenshot above, you can see the different states of the app:
- pickup journey: when the vehicle is on its way to pick up the user at home
- searching vehicle: after the user requested the return journey
- return journey: the vehicle on its way to pick up the user at the restaurant

In this post, I’m going to talk about how we did this and what challenges we faced.

## The solution

The technical solution involves three main components:
- <a href="https://redux.js.org/" target="_blank">Redux</a> to update the vehicle position on the map every time we get an answer from the API
- <a href="https://github.com/gaearon/redux-thunk" target="_blank">Redux Thunk</a> actions that dispatch asynchronous API requests to provoke a response from the server (due to limitations, this had to be done via polling) 
- an API proxy that would communicate with the ride-sharing service and filter/map the data according to our needs.
- Expo’s <a href="https://docs.expo.io/versions/v27.0.0/sdk/map-view" target="_blank">MapView</a> component (which is based on AirBnb’s React Native component) to show a marker with the current coordinates of the vehicle. 

### Working around an unfinished API

The API proxy sounds like a weird thing to do, but we actually had to put that layer in between our app and the ride-sharing API for several reasons. There were cases when no vehicles would be available, and we couldn’t rely on the app itself to send another request (because it could be inactive). Instead, a cron job handled that task. 

Unfortunately, at the time we built the app there was also no way to use the ride-sharing API's websocket integration which would push status updates to the app. So we had to go for good ol' polling and send regular requests to the server to obtain the current location data. Since we didn’t expect there to be tons of requests per hour, this seemed a viable solution for the beginning, even though not a very scalable one.

Things got more complicated when we found out that the ride-sharing service’s testing environment was still in development. So any possible scenario (ride ordered, car on the road, car arrived, all drivers busy, etc…) we needed to respond to in our own product could only be tested in production! More on that later...

<!-- That sounds like a no-go. But, according to the startup principle "work with the limited resources you have at your disposal" we didn’t really have another option. -->

### Displaying markers with Expo’s MapView component

The experience of working with Expo’s <a href="https://docs.expo.io/versions/latest/sdk/map-view" target="_blank">MapView</a> to display live markers on the map is pretty straightforward. 
<pre>
<MapView
  ref={(ref) => { this.map = ref; }}
  provider="google"
  initialRegion={ {
      latitude: experience.restaurant.local_latitude,
      longitude: experience.restaurant.local_longitude,
      latitudeDelta: LATITUDE_DELTA,
      longitudeDelta: LONGITUDE_DELTA,
  } }
  customMapStyle={STYLES.MAP}
  onLayout={() => this.onMapLayoutSuccess()}
  >
  { this.props.location &&
    <b><MapView.Marker
      identifier="vehiclePosition"
      coordinate={ {
        latitude: this.props.cabifyReturn.location[0],
        longitude: this.props.cabifyReturn.location[1],
      } }
      image={ICON_MAP}
    /></b>
  }

  { this.props.status === 'DRIVER_ONROAD' &&
    <b><MapView.Marker
      identifier="restaurantPosition"
      coordinate={ {
        latitude: experience.restaurant.local_latitude,
        longitude:experience.restaurant.local_longitude,
      } }
      image={ICON_RESTAURANT}
    /></b>
  }

  { this.props.status === 'CLIENT_ONROAD' &&
    <b><MapView.Marker
      identifier="homePosition"
      coordinate={ {
        latitude: experience.order.latitude,
        longitude: experience.order.longitude,
      } }
      image={ICON_HOME}
    /></b>
  }

</MapView>
</pre>

Updating coordinates is also really simple:
A JavaScript interval fires a redux-thunk event to dispatch a new API request every second. The response then gets processed by a redux reducer and the data sent to the view. Once the MapView <a href="https://github.com/react-community/react-native-maps/blob/master/docs/marker.md">Marker</a> receives the new coordinates, it updates its position instantly.

### Fitting the map to a set of supplied markers

The MapView component also makes it easy to focus the map on a set of markers and animate it smoothly to show the corresponding area of the map with every update, using its `fitToSuppliedMarkers` method. Unfortunately though, the method doesn’t allow for any padding (unlike its `fitToCoordinates` counterpart) to make sure the markers don’t overlap with other overlaying elements on the screen.

Luckily, the <a href="https://github.com/react-community/react-native-maps/issues/1149#issuecomment-344883668" target="_blank">community provides a solution</a> that’s very easy to implement:

<pre>
fitToSuppliedMarkersCustom(coordinates) {
  const markers = [];
  markers.push(this.vehicleCoordinates());
  markers.push(coordinates);

  this.map.fitToCoordinates(markers, {
    edgePadding: {
      bottom: 200, right: 50, top: 150, left: 50,
    },
    animated: true,
  });
}
</pre>

All you have to do is store the markers' reference using React's <a href="https://reactjs.org/docs/refs-and-the-dom.html">ref</a> attribute, push them to an array and use the `fitToCoordinates` method instead.

## Testing our scrappy solution

As you can imagine, the first few attempts of properly syncing the API responses with the UI were pretty unsuccessful. That’s not a big deal when you use a testing environment and your requests don’t have any impact on the real world. Our case though was that every time we "tested" the API, we’d request a real vehicle.

What worked in our favour though was that the ride-sharing service provides a 30-second courtesy time of cancelling a ride, not the most ideal feature for drivers, but at least it would allow us to adjust and fine-tune our system, with the minor downside of sending drivers to random destinations all the time.

I actually ended up in the car of a driver who complained about our request/cancel approach of bulletproofing their service's API. But despite making some drivers really mad at us and even though we didn’t develop the most elegant solution (using polling, testing in production, relying on cron jobs), it actually works solidly now, and we managed to bootstrap our way through limited resources and unexpected constraints to a production-ready app.

