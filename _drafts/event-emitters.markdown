---
layout: post
title:  "Event Emitters"
date:   2017-10-23 08:00:00
categories: nodejs async
author: Ted Hagos
---

This is just another markdown `this is code`

{% highlight javascript %}
const EventEmitter = require('events').EventEmitter; 
let server = () => {
  let e = new EventEmitter();

  return e; 
}
{% endhighlight %} 