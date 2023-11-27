---
layout: post
title: 'Unit Testing a Widget Library'
date: '2022-02-24'
categories: unit-testing
---
I recently was working on a widget library in Python. Without getting into too much detail, one thing that plagued me was, **how do I want to unit test this?**

I recently stumbled onto [lvgl](https://github.com/lvgl/lvgl), which is everything I want my library to be.

It is lightweight, has similar widgets, and is geared towards the embedded space. Furthermore, it unit tests all of its widgets in a format that I like.

I particularly like the way it compares the widgets to screenshots for testing. This post, if nothing else, will serve as a reminder/inspiration to continue working on the widget library.
