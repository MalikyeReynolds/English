---
layout: page
title: History
subtitle: Large events since the creation of JavaScript to now
---

##### 1995: The Creation of JavaScript

JavaScript was created in 1995 in just 10 days. It was made by Brendan Eich and implemented in Netscape Navigator, one of the first web browsers. Although it was only able to load static pages, its simplicity made it loved by the public.

##### 1995: The Creation of Internet Explorer

<span id="score"></span>
<span id="highscore"></span>
<canvas id="Game"></canvas>


<html>
  <body>
    <div id="result"></div>
    <script type="text/javascript">  
// THIS IS A VERY SIMPLE ENGINE NOT MADE TO MAKE COMPLEX GAMES
// This was created to make a simple 2d game, there is no camera controls, only rectangles, no rotation, and it isn't at all ready to make games.
// This will work for the project that I am doing, but not much else

// CONTAINS:
// Very simple object creation and handling
// Scenes, easy to swtich between
// Clicking, with the x and y of click. No other User Interaction method
// Very simple AABB collision detection system (might be slightly off idk)
// Delta time
// Very simple list object for data handling
// Very simple drawing (only rects)
// Working simple GameLoop


// USES SCENES TO REDUCE BUGS

class Engine { // CAREFUL, all objects need a reference to an "Engine" to use dependant code
    constructor(canvas = document.createElement('canvas'), width = 600, height = 300) {
        this.canvas = canvas;
        this.canvas.width = width;
        this.canvas.height = height;
        this.ctx = this.canvas.getContext('2d')
        this.createClasses();
        this.objects = this.createList();
        this.background = true;
        this.backgroundColor = '#000000';
        this.update = this.update.bind(this);
        this.delta = 0;
        this.lastDelta = 0;
        this.startScene = this.BaseScene;
        this.scene = null;
        this.raf = null;
    }
    createClick() {
        this.canvas.addEventListener('click', this.handleClick.bind(this));
    }
    handleClick(event) {
        let rect = this.canvas.getBoundingClientRect();
        let x = event.clientX - rect.left;
        let y = event.clientY - rect.top;
    
        this.onClick(x, y);
    }
    onClick() { // Change for game (call createClick first)

    }

    createClasses() {
        let Game = this;
        this.Vector2 = class {
            constructor(x, y) {
                this.x = x;
                this.y = y;
            }
            set(x, y) {
                this.x = x;
                this.y = y;
            }
            copy(v2) { // copys cordinates of another vector2
                this.set(v2.x, v2.y)
            }
        }
        this.BaseList = class {
            constructor() {
                this.contents = [];
                this.removeList = [];
            }
            add(child) {
                this.contents.push(child)
            }
            forceRemove(child) { // Immediate
                let index = this.contents.indexOf(child);
                if (index == -1) {
                    return;
                }
                this.contents.splice(index, 1);
            }
            remove(child) { // Safe, but happens at end of loop (not automatic, call postUpdate)
                this.removeList.push(child)
            }
            iterate(func) { // Doesn't account for removal of objects
                for(let i = 0; i < this.contents.length; ++i) {
                    let ans = func(this.contents[i]);
                    if (ans) { // stops if the function returns trueish value
                        break;
                    }
                }
            }
            postUpdate() { // DOESN'T AUTOMATICALLY CALL FOR SIMPLICITY
                for (let i = 0; i < this.removeList.length; ++i) {
                    let index = this.contents.indexOf(this.removeList[i]);
                    if (index == -1) {
                        continue;
                    }
                    this.forceRemove(this.contents[index]);
                }
                this.removeList = [];
            }
        }
        this.BaseObject = class {
            constructor(game = Game, x = 0, y = 0, width = 10, height = 10, color = "#ffffff") { //locked to rectangles, not making easy to change for simplicity
                this.game = game;
                this.position = this.game.createV2(x, y);
                this.size = this.game.createV2(width, height);
                this.color = color;
                this.anchor = this.game.createV2(0.5, 0.5); // where the x and y coordinates are relative to the top left corner, (0, 0) is the top left, (1, 1) is the bottom right
            }
            draw() { // CAREFULLY CHANGE
                let ctx = this.game.ctx;
                let x = this.getLeft();
                let y = this.getTop();
                ctx.fillStyle = this.color;
                ctx.fillRect(x, y, this.size.x, this.size.y)
            }
            getLeft() {
                return this.position.x - this.size.x * this.anchor.x;
            }
            getTop() {
                return this.position.y - this.size.y * this.anchor.y;
            }
            update() { 
                // Code...
            }
        }
        this.BaseScene = class {
            constructor() {
                this.game = Game;
            }
            ready() {
                // Code...
            }
            update() {
                // Code...
            }
        }
    }
    createList() {
        return new this.BaseList();
    }
    createObject({x = 0, y = 0, width = 10, height = 10, color = '#ffffff'}) { //second set is used if first isn't
        let object = new this.BaseObject(this, x, y, width, height, color)
        this.objects.add(object);
        return object;
    }
    createV2(x, y) {
        return new this.Vector2(x, y)
    }
    draw() {  // only draws rectangles for simplicity
        this.ctx.fillStyle = this.backgroundColor; // always uses a background for simplicity
        this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
        this.objects.iterate(function(child){child?.draw()});
    }
    updateObjects() {
        this.objects.iterate(function(child){child?.update(child.game.delta)})
    }
    checkCollision(r1, r2) { // Uses very simple aabb collision system (MAKE SURE THEY EXIST, CAN'T USE LISTS)
        let s1 = {
            x1: r1.getLeft(),
            x2: r1.getLeft() + r1.size.x,
            y1: r1.getTop(),
            y2: r1.getTop() + r1.size.y
        }
        let s2 = {
            x1: r2.getLeft(),
            x2: r2.getLeft() + r2.size.x,
            y1: r2.getTop(),
            y2: r2.getTop() + r2.size.y
        }

        if (s1.x1 < s2.x2 && s1.x2 > s2.x1 && s1.y1 < s2.y2 && s1.y2 > s2.y1) {
            return true;
        } else {
            return false;
        }
    }
    update(delta) { // GAMELOOP
        if (delta) {
            this.delta = delta - this.lastDelta;
            this.lastDelta = delta;
            if (this.delta > 200) {
                this.delta = 200;
            }
        }
        this?.scene.update();
        this.updateObjects();
        this.draw();
        this.raf = requestAnimationFrame(this.update);
    }
    run(scene) { // start startScene or scene in parameter
        if (this.raf) {
            cancelAnimationFrame(this.raf);
            this.raf = null;
        }
        this.objects = this.createList(); // Very bad, but deletes objects because of no other reference

        scene = scene ?? this.startScene;
        this.scene = new scene();
        this.scene.ready();

        this.update();
       
    }
}

// ˅˅˅˅˅ Game Example ˅˅˅˅˅

let game = new Engine(document.getElementById('Game'), 800, 400);
game.createClick();

game.highScore = 0;
game.scoreDiv = document.getElementById('score');
game.highScoreDiv = document.getElementById('highscore');

class GameScene extends game.BaseScene {
    ready() {
        this.player = game.createObject({x: 150, y: 50, width: 50, height: 50, color: '#0000ff'});
        this.ground = game.createObject({x: 0, y: 0, width: this.game.canvas.width, height: 50, color: '#808080'});
        this.ground.anchor.set(0, 0);
        this.roof = game.createObject({x: 0, y: this.game.canvas.height, width: this.game.canvas.width, height: 50, color: '#808080'});
        this.roof.anchor.set(0, 1);
        this.state = 'stop'
        this.score = 0;
        this.game.onClick = this.onClick.bind(this);
        this.increase = 1;
        this.playerSpeed = 0;
        this.playerSpeedIncrease = 0;
        this.playerDirection = -1;
        this.game.backgroundColor = '#87CEEB'
        this.game.highScoreDiv.innerHTML = 'HighScore: ' + Math.floor(game.highScore);
        this.game.scoreDiv.innerHTML = 'Score: ' + this.score;
        this.spawnTimer = 0;

        this.lastDirNum = 0;
        this.lastDirAmt = 0;

        this.bad = this.game.createList();
        this.createBad();

    }
    createBad() {
        let dir = Math.floor(Math.random() * 2) + 1
        if (this.lastDirNum == dir) {
            this.lastDirAmt += 1;
        } else {
            this.lastDirAmt = 1;
        }
        if (this.lastDirAmt > 2) {
            if (dir == 1) {
                dir = 2;
            } else {
                dir = 1;
            }
            this.lastDirAmt = 1;
        }
        this.lastDirNum = dir;
        let y = dir == 1 ? 75 : 325;
        let bad = game.createObject({x: 850, y: y, width: 50, height: 50, color: '#ff0000'});
        this.bad.add(bad);
        bad.update = function() {
            if (this.game.scene.state == 'start') {
                this.position.x -= 90 * (this.game.delta / 1000)
                if (this.position.x < 120) {
                    this.game.scene.bad.forceRemove(this)
                }
            }
        }
    }
    update() {
        if (this.state == 'stop') {
            this.stop();
        } else if (this.state == 'start') {
            this.start();
        } else if (this.state == 'end') {
            this.end();
        }

        if (this.player.position.y < 75) {
            this.player.position.y = 75;
            this.playerSpeed = 0;
            this.playerSpeedIncrease = 0;
        }
        if (this.player.position.y > 325) {
            this.player.position.y = 325;
            this.playerSpeed = 0;
            this.playerSpeedIncrease = 0;
        }
        this.game.scoreDiv.innerHTML = 'Score: ' + Math.floor(this.score);
    }
    start() {
        this.bad.iterate((e) => {
            let val = this.game.checkCollision(this.player, e);
            if (val) {
                this.state = 'end';
            }
    })
        this.spawnTimer += this.game.delta;
        if (this.spawnTimer >= 1500) {
            this.spawnTimer = 0;
            this.createBad();
        }
        this.playerSpeed += this.playerSpeedIncrease * this.playerDirection * (this.game.delta / 1000);
        this.player.position.y += this.playerSpeed;
        this.score += this.game.delta / 100;
    }
    stop() {

    }
    end() {
        if (this.score > this.game.highScore) {
            this.game.highScore = this.score;
        }
        this.game.highScoreDiv.innerHTML = 'HighScore: ' + Math.floor(game.highScore);
    }
    onClick() {
        if (this.state == 'stop') {
            this.state = 'start';
            return;
        }
        if (this.state == 'start') {
            if (this.playerSpeed == 0) {
                this.playerDirection *= -1;
                this.playerSpeedIncrease = 60;
            }
        }
        if (this.state == 'end') {
            this.game.run();
        }
    }
}
game.startScene = GameScene;



game.run();</script>
  </body>
</html>
