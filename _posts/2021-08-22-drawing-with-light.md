---
title: Drawing With Light
layout: post
date: '2021-08-23 01:09:30 -0400'
categories:
- Projects
tags: javascript canvas
---

![Canvas Capture](/assets/img/canvas.png)


This weekend I worked on a way of drawing on the HTML5 canvas that would simulate what it would be like to draw with a flashlight on a 2D plane. Written purely in javascript, you can try out the demo [here](/sightdraw.html). 

Using an event listener for input fields, you can adjust the RGB value of the light along with the polygonal opacity and degrees of the light. 
```javascript
// Grab all the input fields
var inputAll = document.querySelectorAll('input[type="range"]');
// Loop through all of the input fields to listen for each input slide
for (i = 0; i < inputAll.length; i++) {
  // Listen for the change/slide on each input
  inputAll[i].addEventListener('input', function(r, g, b, o, d){
    // Get the value for the input fields
      r = red.value;
      b = blue.value;
      g = green.value;
      o = op.value;
      d = deg.value;
      range = d * (Math.PI/180);
      if (o == 10) {
        light = 'rgba(' + r + ',' + g + ',' + b + ',1)';
      } else {
        light = 'rgba(' + r + ',' + g + ',' + b + ',' + '.' + o + ')';
      }
      bg.style.backgroundColor = light;
      bg.style.opacity = o;
      colorOutput.innerHTML = light;
      degreeOutput.innerHTML = d + ' degrees';
   });
}
```
The light cast from the first place you click to the direction of your mouse pointer, a polygon consisting of every point visible to that flashlight is drawn. This is done by casting two rays that span your flashlight's FOV. Then checking for all vertices of pre made shapes within that FOV as well as intersections of those two rays made from the flashlights range.

```javascript
  var direction = Math.atan2(Mouse.y-MouseStart.y,Mouse.x-MouseStart.x);
  if (direction >= Math.PI / 2 || direction <= -Math.PI / 2) {
    direction = normalizeAngle(direction);
  }
  var startAngle = direction - range / 2;
  var endAngle = direction + range / 2;
```

There is a lot more to it and you can feel free to inspect the code to find out more as well as use any of it in your own projects. For now, I hope you like playing around with it.
