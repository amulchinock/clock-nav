<script setup lang="ts">
// import HelloWorld from './components/HelloWorld.vue'
// import TheWelcome from './components/TheWelcome.vue'

  import { ref, onMounted } from 'vue';
  import romans from 'romans';

  const main = ref<HTMLCanvasElement | null>(null);

  const resizeCanvas = () => {
    if (main.value !== null) {
      main.value.width = window.innerWidth;
      main.value.height = window.innerHeight;
    }
  }

  const drawClock = (ctx? : CanvasRenderingContext2D | null) => {
    if (!ctx) throw new Error('thing not thingied');
    
    const radius : number = ((() => window.innerWidth <= window.innerHeight ? window.innerWidth : window.innerHeight)() / 2) * 0.9;
    const xMiddle : number = main.value?.width !== undefined ? main.value.width / 2 : 0;
    const yMiddle : number = main.value?.height !== undefined ? main.value.height / 2 : 0;

    ctx.fillStyle = 'orange';
    ctx.strokeStyle = 'white';
    ctx.textBaseline = "middle";
    ctx.textAlign = "center";
    ctx.font = radius * 0.06 + "px courier";

    ctx.beginPath();
    ctx.translate(xMiddle, yMiddle);

    // Draw hours and minutes
    for (let i = 0; i < 360; i++) {
      const rad = (i * Math.PI) / 180;

      ctx.rotate(rad);
      ctx.translate(0, -radius * 0.85);
      ctx.rotate(-rad);

      if (i % 30 === 0) {
        const h = i / 30;
        ctx.fillText(romans.romanize(h === 0 ? 12 : h), 0, 0);
      }

      if (i % 6 === 0 && i % 30 !== 0) {
        ctx.fillText('.', 0, 0);
      }

      ctx.rotate(rad);
      ctx.translate(0, radius * 0.85);
      ctx.rotate(-rad);
    }

    drawTime(ctx, radius);

    ctx?.stroke();
  }

  const drawTime = (ctx : CanvasRenderingContext2D, radius : number) => {
    const now = new Date();
    const second = now.getSeconds() * Math.PI / 30;
    const minute = (now.getMinutes() * Math.PI / 30) + (now.getSeconds() * Math.PI / (30 * 60));
    const hour = (now.getHours() % 12) * Math.PI / 6 + (now.getMinutes() * Math.PI/(6*60)) + (now.getSeconds() * Math.PI/(360*60));

    const direction = (key : string) => {
      const isMorning = now.getHours() - 12 < 0;
      const s = isMorning ? hour + (((12 * Math.PI / 6) - hour) / 2) : hour / 2

      switch (key) {
        case 'n':
          return s + Math.PI;

        case 's':
          return s;

        case 'e':
          return s + (-Math.PI / 2)

        case 'w':
          return s + (Math.PI / 2);

        default:
          return 0;
      }
    }

    // Cardinal directions
    drawHand(ctx, direction('n'), radius, radius*0.01, '#0081cd');
    drawLabel(ctx, direction('n'), radius * 0.9, 'N');

    drawHand(ctx, direction('s'), radius, radius*0.01, '#bc1e47');
    drawLabel(ctx, direction('s'), radius * 0.9, 'S');

    drawHand(ctx, direction('e'), radius, radius*0.01, '#fec200');
    drawLabel(ctx, direction('e'), radius * 0.9, 'E');

    drawHand(ctx, direction('w'), radius, radius*0.01, '#009246');
    drawLabel(ctx, direction('w'), radius * 0.9, 'W');
    
    // Time
    drawHand(ctx, hour, radius*0.5, radius*0.07);
    drawHand(ctx, minute, radius*0.7, radius*0.06);
    drawHand(ctx, second, radius*0.7, radius*0.02);

    // Celestial bodies
    drawLabel(ctx, hour, (radius / 2) + 5, '☀️');
  }

  const drawHand = (ctx : CanvasRenderingContext2D, pos : number, length : number, width : number, color? : string) => {
    ctx.beginPath();
    ctx.lineWidth = width;
    ctx.lineCap = "round";
    ctx.strokeStyle = color ?? 'white';
    ctx.moveTo(0,0);
    ctx.rotate(pos);
    ctx.lineTo(0, -length);
    ctx.stroke();
    ctx.rotate(-pos);
  }

  const drawLabel = (ctx : CanvasRenderingContext2D, pos : number, rad : number, text : string) => {
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.rotate(pos);
    ctx.translate(0, -rad * 1.2);
    ctx.fillText(text, 0, 0);
    ctx.translate(0, rad * 1.2);
    ctx.rotate(-pos);
  }

  const setupStuff = () => {
    resizeCanvas();
    drawClock(main.value?.getContext('2d'));
  }

  const createInterval = () => window.setInterval(setupStuff, 500);

  onMounted(() => {
    let int = createInterval();
    // setupStuff();
    
    window.addEventListener('resize', () => {
      window.clearInterval(int);
      setupStuff();
      int = createInterval();
    });
  });
</script>

<template>
  <canvas id="main" ref="main"></canvas>
</template>

<style scoped>
  #main {
    display: block;
    width: 100vw;
    height: 100vh;
    background-color: darkslategray;
  }
</style>
