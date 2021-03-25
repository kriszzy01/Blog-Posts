## Build Flutterwave's Design Team's Value Card Pile using GSAP

Flutterwave's design team showcased their team's values using an interactive and animated pile of cards, which you can check out on their [blog](https://www.flutterwave.design/about/). In this article, we will be building out a similar pile of cards, and add the same interactions, and animations using GSAP. 

GSAP is a powerful JavaScript animation library which gives animators lots of flexibility, and control to create complex animations with a few lines of code. It provides a suite of tools for scripted property manipulation, and it's flexibility means it can animate anything JavaScript can "touch" (JavaScript variables, objects, arrays, CSS properties, SVG, DOM etc). Check out their  [website](https://greensock.com/get-started/) for more information.

## Getting Started
To get started, run the following commands on your terminal to create a React App called Card-Pile, and install GSAP in our new app.

```
npx create-react-app card-pile
``` 

```
#NPM
npm install gsap

#Yarn
yarn add gsap
``` 

## Creating the Pile of Cards
To create the pile of cards, we will create six squares and incrementally offset their respective rotation angles via CSS. In our Card-Pile app folder, copy and replace the existing code on your App.js and App.css files with the following code respectively.

```
import React from "react";
import "./App.css";

const App = () => {
  return (
    <>
      <header>
        <h1>Flutterwave's Value Card Pile</h1>
        <p>Scroll Down</p>
      </header>

      <div className="container">
        <div className="panel box1"></div>
        <div className="panel box2"></div>
        <div className="panel box3"></div>
        <div className="panel box4"></div>
        <div className="panel box5"></div>
        <div className="panel box6"></div>
      </div>
    </>
  );
};

export default App;

``` 

```
body {
  overflow-x: hidden;
}

header {
  text-align: center;
}

.container {
  overflow-x: hidden;
  width: 200px;
  height: 200px;
  margin: 30rem auto;  /*Added to allow scrolling*/
  position: relative;
}

@media screen and (min-width: 768px) {
  .container {
    width: 450px;
    height: 450px;
  }
}

.panel {
  width: 100%;
  height: 100%;
  position: absolute;
}

.box1 {
  transform: rotate(20deg);
  background-color: lightcoral;
}

.box2 {
  transform: rotate(16deg);
  background-color: lightseagreen;
}

.box3 {
  transform: rotate(12deg);
  background-color: lightgreen;
}

.box4 {
  transform: rotate(8deg);
  background-color: lightskyblue;
}

.box5 {
  transform: rotate(4deg);
  background-color: cornflowerblue;
}

.box6 {
  transform: rotate(0deg);
  background-color: darkolivegreen;
}

``` 
## Animating the Pile of Cards with GSAP's Scroll Trigger Plugin
Flutterwave's pile of cards has a "pile up" effect which is initialised when we scroll into the section containing the card piles. GSAP's [Scroll Trigger plugin](https://greensock.com/docs/v3/Plugins/ScrollTrigger) enables us to trigger animations based on the position of the scroll. On our App.js, let's update our code as shown.  

```
import React, { useEffect } from "react";
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

const App = () => {
  useEffect(() => {
    gsap.registerPlugin(ScrollTrigger); //Register plugins
    const cards = gsap.utils.toArray(".panel"); //Array of DOM elements with classname of panel

    gsap.from(cards, {
      autoAlpha: 0,
      scale: 0.4,
      stagger: 0.08,
      force3D: true, 
      ease: "Power4.easeInOut",
      scrollTrigger: {
        trigger: ".container",
        start: "top center",
      },
    });
  }, []);

  ...
``` 
Here, we are creating the "pile-up" effect using a [Tween](https://greensock.com/docs/v3/GSAP/Tween), which animates from autoAlpha of 0 (which is simply an opacity of 0, visibility of hidden), and a scale of 0.4. The stagger property adds 0.08s delay between each card, while the scrollTrigger plays the animation when the top of the card container touches the center of the viewport.
## Adding Drag Interactions using GSAP's Draggable Plugin
Each value card on the Flutterwave design team's blog can be dragged to reveal the card below it. Depending on the drag direction, the dragged card moves out of the pile. We will take advantage of GSAP's [Draggable plugin](https://greensock.com/docs/v3/Plugins/Draggable) which provides a simple way to drag, spin or toss DOM elements to implement this interaction. Let's update our code as shown below.

```
import React, { useEffect } from "react";
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { Draggable } from "gsap/Draggable";

function handleDragDirection() {
  const direction = this.getDirection(); //drag direction

  gsap.to(this.target, {
    duration: 0.3,
    x: direction.includes("right") ? 1000 : -1000,
    ease: "Power0.easeInOut",
  });
}

const App = () => {
  useEffect(() => {
    gsap.registerPlugin(ScrollTrigger, Draggable); //Register plugins
    const cards = gsap.utils.toArray(".panel"); //Array of DOM elements with classname of panel
  ...

  Draggable.create(cards, {
      type: "x",
      force3D: true,
      onDragEnd: handleDragDirection,
    });

  ...
   
``` 
Here, we make the cards Draggable only in the horizontal direction by setting the type property of the Draggable config object to "x". The onDragEnd config property is a callback that is called when the Drag event ends. The callback function provides a getDirection function which we use to determine the drag direction. We then animate the dragged card(which becomes the target) in that direction.

## Returning the Pile after the last Card is Dragged
To return the pile, we need to track each drag event, and determine when the last card has been dragged. To do this, we will use a useState hook to count the drags, and use that number to determine the last drag. Let's update our code as shown below.

```
import React, { useEffect, useState } from "react";
...

function handleDragDirection(setCount) {
  ...

  setCount((count) => count + 1); //track number of drags
}

const App = () => {
  const [count, setCount] = useState(0);
  ...

  useEffect(() => {
  ...

  Draggable.create(cards, {
      type: "x",
      force3D: true,
      onDragEnd: handleDragDirection,
      onDragEndParams: [setCount],
    });
  }, []);

  useEffect(() => {
    const cards = gsap.utils.toArray(".panel");

    if (count === cards.length) {
      setCount(0); //Reset the count

      //Return cards to original position
      gsap.to(cards, {
        duration: 0.3,
        delay: 0.5,
        x: 0,
        stagger: 0.06,
        ease: "Power4.easeOut",
        force3D: true,
      });
  }, [count]);
  
  ...
``` 
Here, we pass setCount function as a parameter to the onDragEnd callback to track the the number of drags. We then use a useEffect hook to to return the pile when the last card is dragged.
## Rotate card on each drag
Whenever we drag a card from the pile, we want the preceding card to have a rotate degree of 0. We can do this by removing 4 degrees from the rotation angles of all the cards on each drag. Let's update our code as shown below. 

```
import React, { useEffect, useState, useRef } from "react";
...

function handleDragDirection(cards, setCount, rotateAnglesRef) {
  ...

  //update rotate values
  cards.forEach((card, index) => {
    gsap.to(card, {
      rotate: rotateAnglesRef.current[index] - 4,
    });

    rotateAnglesRef.current[index] = rotateAnglesRef.current[index] - 4;
  });
}

export const App = () => {
  const [count, setCount] = useState(0);
  const rotateAnglesRef = useRef([20, 16, 12, 8, 4, 0]);

  useEffect(() => {
    ...

    Draggable.create(cards, {
      type: "x",
      force3D: true,
      onDragEnd: handleDragDirection,
      onDragEndParams: [cards, setCount, rotateAnglesRef],
    });
  }, []);

  useEffect(() => {
    const cards = gsap.utils.toArray(".panel");

    if (count === cards.length) {
      ...

      //Reset rotation and stacking order
      cards.forEach((card, index) => {
        gsap.to(card, {
          zIndex: index * 2,
          rotate: rotateAnglesRef.current[index],
        });
        rotateAnglesRef.current = [20, 16, 12, 8, 4, 0];
      });
    }
  }, [count]);

  ...
``` 
Here we stored an array of rotation angles using a useRef hook. We then use the onDragEnd callback to update the rotation angles in the ref, and animate the angles of the cards. Once the last card is dragged, we use our useEffect hook to reset the skew angles to the initial values.
## Conclusion
At this point, we now have a similar implementation of the pile of value cards on the Flutterwave Design team's blog. You check out the live [demo](https://kriszzy01.github.io/flutterwave-card-pile/), and the repository on  [GitHub](https://github.com/kriszzy01/flutterwave-card-pile).

Hope you had fun building this out as I did!




