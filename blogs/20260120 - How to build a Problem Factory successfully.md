# How to build a problem factory, successfully

## The importance of learning stuff

Okay, that will be a happily written blog, after failing at starting game dev a thousend times.  
Before i began this project i watched some game dev starter videos and took their advices.  
A few notes on what to keep in mind if you're starting your first real project:  

- Keep the scope simple (Arcade Games, Action Games, Jump 'n Run)
- Use a engine, a tool that is well documented and has examples 
- Focus on getting stuff done, avoid perfectionism

I heavily encourage everyone new to game development, to keep this notes in mind...  
But... But... i worked as a software developer / system architect for 15 years now.  
I have a tech-stack that i am in love with since the start of my carreer...  

As a fullstack developer (Started with ASP.NET ended on JS (Node/Bun), HTML and CSS) i wanted to use my experience.  
And luckily for game development (or any graphical purpose) we have the **\<canvas/>** element.  
You can use it for 2D and 3D rendering, that is computed on the GPU via WebGL, NICE! 

But when using canvas, you will loose lot's of already implemented Stuff from the DOM-Model.   
- Input-Event-Handling
- Cascading Styling Sheets (mostly)
- Simple inspection of the Image (F12).

Everyone nows, DOM-Rendering ISN'T MEANT FOR GAME DEVELOPMENT.  

So... time to learn new things, aye ?

## The problem factory...

Okay, in total ignorance of what is considered viable by any sane person in the web-dev-space,  
let's forget about the canvas-element.

I want to have a 2D-World with evenly sized tiled (n*n).   
Just throw a js together that is rendering a table, done that dozens of times. 

    <table>
        <tr>
            <td>
                <div class="content">  

Okay, how do we do tiles (graphics) then...  
Pixel-Art is considered "beginner friendly", maybe easier is SVG Art (due to scaling and stuff).  
I am not an artist and i mean i have no creative energy to do any graphics inside any software...  

**PROBLEM:**  
You can't do graphics in code... or can you...

**Solution:**

I have created a simple ASCIIArt Class that i can feed 3x3 Patterns, that renders them to the content cells.  
Add a option to change the font-color, to create grass in green and dust in some kind of brown. 

**Example for my Dust Cell: **
```js
['`', ' ', '*']
[' ', 'o', '.\'']
['..', ' ', '.']
```

NICE! Works like a charm and i don't have to leave my beloved Code-Editor to create my art. 

## Art is one of it's kind

I've done a tree, that was a mistake.

```js
['/', '|', '\\']
['_', '|', '_']
[' ', '|', ' ']
```
I showed a friend of mine my small fun project and he told me, that it was wrong.   
In that type of game a tree is equally sized to the player... that's normal.  
Have a look at the early final fantasy games or the ultima titles.  

He knew it, but because we are boys we like to mock each other up.  
  

You know, sometimes there is this deep thought about the what if.   
What if my tree can be... higher than the player character.  
What if Walls could be... higher.  
What is the player wondering a forest, would feel like it.  

So, this is the absolute Point before the Point of No Return.   
Just choose Unity or Godot, rebuild those 1000 Lines of code and start happily creating your game.   
If i stick to my road, i will encounter singular problems nobody ever encountered,   
and if i ask people, they're going to tell me i use the wrong technology... that's not possible, yadda.

## Chasing horizons

So, one more dimension (kinda) in html - haha.  
But wait... there is... CSS-Transform: rotateX()   

I could rotate my game-space by 45deg and then i would have...   
A game that is barely visible and is unplayable. (STOP IT NOW, I beg you).  

As a huge fan of the first 2 DOOM games (and the whole DOOM runs om toaster) and community,   
Something called raycasting (Wolf3D technology ID-Tech1) came to my mind.

Wolf3D is technically a 2D game, with a good illusion. "A good Illusion".  
What happens if you rotate the game space, but not the objects in it. 

So, after unattaching elements from the table-rendering -   
i just threw them into a new body>div.obj [left:0, top:0]  
and positioned them by translateX and translateY to the game space.  
**Wonderful, the illusion comes to life...**

## If the player is "behind" a tree...

So, if you once started rotating and vibing with you problem factory - it is beautiful.   
I wanted to rotate my world, like a camera around it.   

So i implemented @keyframe animations to do so. 

**PROBLEM: BEHIND**

For those of you, who are not familiar with html, let me introduce a little friend of mine Z-Index.  
By Default elements can overlap others and the one deeper in the html-tree overrides the one that comes before it.   
That behaviour is wonderful and should not be touched without any reason. TL;DR - NEVER TOUCH Z-INDEX  

### Short Journey in insanity

For ASP.NET projects with... let's call it legacy, you always cand find something like this css class:

```css
.InFront {
    z-index:999999999!important; 
}
```

I worked on too many projects with this stuff. But the best one was (until now in production):
```css
.InFronter {
    z-index:99999999999!important;
}
```

Sanity has long gone, but let me tell ya - if you have intelligent people, chained by that kind of legacy, they tend to get creative:  
You can create any tag in JS, even **\<style>** tags. I love it, can you see the oppurtunities.  

After fiddling around for days with some DIV not getting the ultimateFrontest - my coworker said: "Bruteforce is always a solution."...  
Oh boy... he created a function called createTheUltimateFront() - our humor was dark.. the nights were long.  

```js
function createTheUltimateFront() {
    const html = ` 
    <style>
        .ultimateFront {
            z-index:${Number.MAX_VALUE}!important;
        }
    </style>
    `
    $('head').append(html);
}
```

... it worked and the **customer was happy.**
The luck of the unknowing.

### Z-Index

So if i want to be able to rotate, i have to change z-index values of all rendered objects, accordingly to the camera position. 
```js

    function createZIndexMatrix() {
        let zIndex = 1
        const zIndexMatrix = []
        for (let y = 0; y < this.options.roomSize.y; y++) {
            const zIndexRow = []
            for (let x = 0; x < this.options.roomSize.x; x++) {
                zIndexRow.push(zIndex)
                zIndex += 1
            }
            zIndexMatrix.push(zIndexRow)
            zIndex += this.options.roomSize.x + 10
        }

        this.zIndexMatrix = zIndexMatrix
    }

``` 

PROBLEM: My Gamespace is not always n by n, it can be n by x - 

I have to create one inverse variant for all 4 directions...   

But after all that i can get the correct z-index for all elements inside a neat function 
```js
  getZIndexForOrientation(cellCoordinates: Vector2D) {
        const orientation = getWorldReference().worldOrientation
        switch (orientation) {
            case WorldOrientation.North:
                return this.zIndexMatrix[cellCoordinates.y][cellCoordinates.x]
            case WorldOrientation.South:
                return this.zIndexMatrix.at(-cellCoordinates.y).at(-cellCoordinates.x)
            case WorldOrientation.West:
                return this.zIndexMatrixFlipped[cellCoordinates.x][cellCoordinates.y]
            case WorldOrientation.East:
                return this.zIndexMatrixFlipped.at(-cellCoordinates.x).at(-cellCoordinates.y)
        }
    }

```

 
So if you want to flip an squared matrix, you can do that very easily.  



It is working.


After all that stuff, (nearly 2 months of work) the result can be watched at:

https://www.youtube.com/watch?v=vfv4pG56V_s









