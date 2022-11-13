<script setup>
import Two from "two.js";

var two = new Two({
    fullscreen: true,
    autostart: true,
}).appendTo(document.body);

let line = {
    startX: 0,
    startY: 0,
    endX: 0,
    endY: two.height,
};

function updateLineCoordinate(tick) {
    let relativeTick = tick % (two.width + two.height);
    let loopBack = Math.floor(tick / (two.width + two.height)) % 2 != 0;

    if (relativeTick < two.width && !loopBack) line.endX++;
    if (relativeTick >= two.width && !loopBack) line.endY--;
    if (relativeTick < two.height && loopBack) line.endY++;
    if (relativeTick >= two.height && loopBack) line.endX--;
}

function drawCircle(circle) {
    let c = two.makeCircle(circle.x, circle.y, circle.radius);
    c.fill = "white";
    return c;
}

function sdf(rayPos, circle) {
    const a = rayPos.x - circle.x;
    const b = rayPos.y - circle.y;
    return Math.sqrt(a * a + b * b) - circle.radius;
}

let tick = 0;

const circles = [
    {
        x: 100,
        y: 360,
        radius: 120,
    },
    {
        x: 430,
        y: 800,
        radius: 120,
    },
    {
        x: 520,
        y: 100,
        radius: 50,
    },
    {
        x: 1080,
        y: 200,
        radius: 100,
    },
    {
        x: 1320,
        y: 780,
        radius: 200,
    },
];

circles.forEach((e) => drawCircle(e));
//  const rect = two.makeRectangle(drawArea.centerX, drawArea.centerY, drawArea.width, drawArea.height);
let lineObject = two.makeLine(line.startX, line.startY, line.endX, line.endY);
let tmpCircle = [];

two.bind("update", function () {
    two.remove(lineObject, ...tmpCircle);

    tmpCircle = [];

    updateLineCoordinate(tick++);

    lineObject = two.makeLine(line.startX, line.startY, line.endX, line.endY);
    lineObject.stroke = "white";

    const minDistance = 0.01;
    const maxDepth = 1000;
    const maxStep = 30;
    let step = 0;
    let depth = 0;

    let rayDir = {
        x: line.endX - line.startX,
        y: line.endY - line.startY,
    };

    let rayLength = Math.sqrt(rayDir.x * rayDir.x + rayDir.y * rayDir.y);

    rayDir = {
        x: rayDir.x / rayLength,
        y: rayDir.y / rayLength,
    };

    while (step < maxStep && depth < maxDepth) {
        const rayPos = {
            x: rayDir.x * depth,
            y: rayDir.y * depth,
        };

        const dist = Math.min(...circles.map((e) => sdf(rayPos, e)));

        if (dist <= minDistance) break;
        depth += dist;
        step++;

        let c = drawCircle({
            x: line.startX + rayPos.x,
            y: line.startY + rayPos.y,
            radius: dist,
        });

        c.fill = "transparent";
        c.stroke = "white";
        tmpCircle.push(c);
    }

    console.log(tmpCircle);
});
</script>

<template>
    <header></header>

    <main>
        <TheWelcome />
    </main>
</template>
