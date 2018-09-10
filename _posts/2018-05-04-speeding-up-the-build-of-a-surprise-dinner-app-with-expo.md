---
layout: post
status: publish
published: true
title: Speeding up the build of a React Native app with Expo
header_image: "/assets/images/blogs/todd-diemer-113294-unsplash.jpg"
image_credit:
  handle: todd_diemer
  name: Todd Diemer
author:
  display_name: marcelkalveram
  login: marcelkalveram
  email: marcel.kalveram@googlemail.com
  url: http://marcelkalveram.com
author_login: marcelkalveram
author_email: marcel.kalveram@googlemail.com
author_url: http://marcelkalveram.com
date: "2018-05-04 11:09:00 +0200"
date_gmt: "2018-05-04 11:09:00 +0200"
tags: []
comments: []
---

React Native can be hard. But a lot of the heavy lifting can be done by its open source companion toolchain <a href="https://expo.io/" target="_blank">Expo</a>.

Recently, I helped Spanish startup <a href="https://uncovercity.com/" target="_blank">uncovercity</a> build a mobile app to accompany their surprise dinner experience.

## The challenge

Tasks in this project included:

- showing an introductory video header in the app to the user to get them excited about the experience
- picking up the user at a specific location using a ride-sharing API and showing the current position of the client’s ride on a map
- showing several popups indicating the current status of the user's experience (e.g. taxi on its way, taxi arrived, taxi arrived at restaurant) and sending a push notification for each status change

<p class="images clearfix">
  <amp-img layout="responsive" class="img" width="230" height="454" alt="Screenshot of the uncovercity iOS app experience screen" src="/assets/images/screenshots/uncovercity-experience.jpg"></amp-img>
  <amp-img layout="responsive" class="img" width="230" height="454" alt="Screenshot of the uncovercity iOS app pickup screen" src="/assets/images/screenshots/uncovercity-pickup.jpg"></amp-img>
  <amp-img layout="responsive" class="img" width="230" height="454" alt="Screenshot of the uncovercity iOS app popup screen" src="/assets/images/screenshots/uncovercity-popup.jpg"></amp-img>
</p>

## The solution

I had built several React Native apps before and knew how tricky it would be to implement custom styling or third-party components.

That’s why I chose Expo in this case, because it provides several predefined components for each of the above mentioned scenarios by default:

- <a href="https://docs.expo.io/versions/v27.0.0/sdk/video" target="_blank">Video</a>: a relatively hassle-free integration of mp4 files
- <a href="https://docs.expo.io/versions/v27.0.0/sdk/map-view" target="_blank">MapView</a> (based on AirBnb’s excellent <a href="https://github.com/react-community/react-native-maps" target="_blank">map component</a>)
- <a href="https://docs.expo.io/versions/v27.0.0/sdk/blur-view" target="_blank">a blurred modal</a> (which doesn’t come with React Native by default).
- and <a href="https://docs.expo.io/versions/latest/" target="_blank">many others...</a>

### The biggest time saver

The biggest time-saver was probably how easy Expo it makes to send push notifications. It’s almost too simple. See <a href="https://docs.expo.io/versions/v27.0.0/guides/push-notifications" target="_blank">this guide</a>, or let me sum it up quickly:

- grab a token on the phone by asking the user for notification permissions using Expo's <a href="https://docs.expo.io/versions/v27.0.0/sdk/permissions" target="_blank">Permissions</a> component
- send the obtained token to your backend and store it for subsequent requests
- use one of Expo's push notification libraries (<a href="https://github.com/exponent/exponent-server-sdk-node" target="_blank">node</a>, <a href="https://github.com/exponent/exponent-server-sdk-ruby" target="_blank">Ruby</a>, <a href="https://github.com/Alymosul/exponent-server-sdk-php" target="_blank">PHP</a> etc...) to trigger a notification from your backend application logic

<div class="callout">
I've created a very simple <a target="_blank" href="https://github.com/marcelkalveram/expo-push-notifications-server">example of an Expo push notification server on GitHub</a> using node.js and mongoDB.
</div>

Testing those notifications is easy (and fun) as well. You can use your phone-specific token and use the Expo <a href="https://expo.io/dashboard/notifications" target="_blank">push notification tool</a> to send yourself some push notifications and play around with the parameters.

## Why am I saying this?

Many times, we think of native apps as complex systems with a lot of custom programming and the risk of having to dig deep into native iOS/Android code.

My experience in this case has been the complete opposite. Need a video component? Expo provides one out-of-the-box. Need a map integration? Expo has you covered. Need a push notification solution without spending a week on integrating it? Let me cite the Expo docs here: "it's almost too easy".

Even trivial things like using Expo's <a href="https://docs.expo.io/versions/latest/sdk/linear-gradient" target="_blank">LinearGradient</a> to show, well, a linear gradient, can save you some headaches here and there.

### What about testing?

To wrap up, testing the app on your phone is a piece of cake as well. <a href="https://itunes.apple.com/us/app/expo-client/id982107779?mt=8" target="_blank">Expo Client</a> allows you to immediately preview and test the app on your phone within minutes, allowing you to see changes in real-time. Pretty awesome, isn’t it?

## Future posts about this project

I’ve written <a href="/2018/05/08/using-maps-in-react-native-to-track-a-vehicle.html">another post</a> about my experience building the uncovercity app, where I cover how we did the API integration and what I learned from doing some real life user testing by living the experience as an uncovercity user myself.
