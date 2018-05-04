---
layout: post
status: publish
published: true
title: Speeding up the build of a surprise dinner app with Expo
author:
  display_name: marcelkalveram
  login: marcelkalveram
  email: marcel.kalveram@googlemail.com
  url: http://marcelkalveram.com
author_login: marcelkalveram
author_email: marcel.kalveram@googlemail.com
author_url: http://marcelkalveram.com
date: '2018-05-04 11:09:00 +0200'
date_gmt: '2018-05-04 11:09:00 +0200'
tags: []
comments: []
---

React Native can be hard. But a lot of the heavy lifting can be done by <a href="https://expo.io/" target="_blank">Expo</a>.

Recently, I helped Valencia startup <a href="https://uncovercity.com/" target="_blank">uncovercity</a> build a mobile app to accompany their surprise dinner experience. 

## The challenge

Tasks in this project included:

- showing an introductory video header in the app to the user to get them excited about the experience
- picking up a client and showing the current position of the client’s taxi (a <a href="https://www.cabify.com/" target="_blank">Cabify</a> in this case) on a map
- showing several popups indicating the current status of the experience (e.g. Cabify on its way, Cabify arrived, Cabify arrived at restaurant) and sending a push notification in each case

## The solution

I had built several React Native apps before and knew how tricky it would be to implement specific behaviour or components using third-party plugins. 

That’s why I chose Expo in this case, because it provides several predefined components for each of the above mentioned use cases by default: <a href="https://docs.expo.io/versions/v27.0.0/sdk/video" target="_blank">Video</a>, Map (based on AirBnb’s excellent <a href="https://github.com/react-community/react-native-maps" target="_blank">map component</a>), <a href="https://docs.expo.io/versions/v27.0.0/sdk/blur-view" target="_blank">a blurred modal</a> (which doesn’t come with React Native by default).

### The biggest time saver

The biggest time-saver was probably how easy Expo it makes to send push notifications. It’s almost ridiculous, check out <a href="https://docs.expo.io/versions/v27.0.0/guides/push-notifications" target="_blank">this guide</a>, or to sum it up quickly: 
- grab a token on the phone by asking the user for notification using Expo's <a href="https://docs.expo.io/versions/v27.0.0/sdk/permissions" target="_blank">Permissions</a> component
- send the obtained token to your backend and store it for subsequent requests
- use one of the Expo push notification libraries (<a href="https://github.com/exponent/exponent-server-sdk-node" target="_blank">node</a>, <a href="https://github.com/exponent/exponent-server-sdk-ruby" target="_blank">Ruby</a>, <a href="https://github.com/Alymosul/exponent-server-sdk-php" target="_blank">PHP</a> etc...) to trigger a notification from your backend application logic

Testing is easy as well. You can use your phone-specific token use the Expo <a href="https://expo.io/dashboard/notifications" target="_blank">push notification tool</a> to send yourself some push notifications and play around with the parameters.

## Why am I saying this?
Many times, we think of native apps as complex system with a lot of custom programming and the risk of having to dig into native iOS/Android code.

My experience in this case has been the complete opposite. Need a video component? Expo provides one out-of-the-box. Need a map integration? Expo has you covered. Need a push notification solution without spending a week on integrating it? See what I just said.

Even trivial things like using Expo's <a href="https://docs.expo.io/versions/latest/sdk/linear-gradient" target="_blank">LinearGradient</a> to show, well, a linear gradient, can save you some headaches here and there.

### What about testing?

To wrap up, testing the app on your phone is a piece of cake as well. <a href="https://itunes.apple.com/us/app/expo-client/id982107779?mt=8" target="_blank">Expo Client</a> allows you to immediately preview and test the app on your phone in real-time, allowing you to see changes in real-time. Pretty awesome, isn’t it?

# Future posts on this project

I’ll write a few more posts about my experience building the uncovercity app, especially how we did the Cabify integration using their production API for testing (yep, you read that right) and what I learned from doing some real life user tests going out for dinner with uncovercity myself.

I’ll keep you posted.
