<script>
  import { onMount } from "svelte";

  export let width = 320;
  export let height = 240;
  export let line_width = 1;
  export let color = { r: 255, g: 255, b: 255, a: 1 };
  export let scale = 10;
  let canvasWidth = width * scale;
  let canvasHeight = height * scale;

  let canvas;

  onMount(() => {
    if (canvas.getContext) {
      let ctx = canvas.getContext("2d");
      ctx.strokeStyle =
        "rgba(" +
        color.r +
        ", " +
        color.g +
        ", " +
        color.b +
        ", " +
        color.a +
        ")";

      ctx.beginPath();
      for (let i = 0; i < width; i++) {
        ctx.moveTo(i * scale, 0);
        ctx.lineTo(i * scale, canvasHeight);
      }

      for (let i = 0; i < height; i++) {
        ctx.moveTo(0, i * scale);
        ctx.lineTo(canvasWidth, i * scale);
      }
      ctx.stroke();
    }
  });
</script>

<canvas width={canvasWidth} height={canvasHeight} bind:this={canvas} />

<style>
  canvas {
    position: absolute;
    width: 100%;
    left: 0;
    top: 0;
  }
</style>
