<script>
  import { onMount } from "svelte";
  import Display, { pixelFormatEnum } from "./Display.svelte";
  import Grid from "./Grid.svelte";
  import WebUSB from "./WebSerial.svelte";

  let display;
  let web_serial;

  function isPixelSet(data, x, y) {
    let i = Math.floor(y / 8) * 128;
    y &= 7;
    i = Math.floor(i + x);
    return (data[i] & (1 << y)) == 0;
  }

  function parse_display(display_data) {
    let display_index = 0;

    for (let x = 0; x < display.width; x++) {
      for (let y = 0; y < display.height; y++) {
        if (isPixelSet(display_data, x, y)) {
          display.setPixel(x, y, display.colorRGB(0, 0, 0));
        } else {
          display.setPixel(x, y, display.colorRGB(255, 255, 255));
        }
      }
    }

    display.doRender();
  }

  let noise_timer;
  function connect() {
    clearInterval(noise_timer);
  }

  function disconnect() {
    noise_timer = setInterval(() => {
      display.generateNoise();
    }, 100);
  }

  function send_up() {
    web_serial.send_command("input_send up short");
  }
  function send_down() {
    web_serial.send_command("input_send down short");
  }
  function send_left() {
    web_serial.send_command("input_send left short");
  }
  function send_right() {
    web_serial.send_command("input_send right short");
  }
  function send_ok() {
    web_serial.send_command("input_send ok short");
  }
  function send_back() {
    web_serial.send_command("input_send back short");
  }
</script>

<main>
  <div class="main-panel">
    <div class="display-panel">
      <Display
        bind:this={display}
        width="128"
        height="64"
        scale="8"
        bw_empty_color={{ r: 255, g: 140, b: 41 }}
        bw_filled_color={{ r: 17, g: 17, b: 17 }}
        pixelFormat={pixelFormatEnum.bw}
      />
      <Grid
        width="128"
        height="64"
        scale="20"
        color={{ r: 100, g: 100, b: 100, a: 0.5 }}
      />
    </div>
    <div class="control-panel">
      <div class="input-row">
        <button on:click={send_up}>↑</button>
      </div>
      <div class="input-row">
        <button on:click={send_left}>←</button>
        <button on:click={send_ok}>●</button>
        <button on:click={send_right}>→</button>
      </div>
      <div class="input-row">
        <button on:click={send_down}>↓</button>
      </div>
      <div class="input-row">
        <button on:click={send_back}>↺</button>
      </div>
    </div>
  </div>
  <div class="console-panel">
    <WebUSB
      bind:this={web_serial}
      parse_display_callback={parse_display}
      disconnect_callback={disconnect}
      connect_callback={connect}
    />
  </div>
</main>

<style>
  * {
    -moz-user-select: none;
    -o-user-select: none;
    -khtml-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }

  :global(body) {
    padding: 0;
    margin: 0;
    background: black;
  }

  main {
    width: 100%;
    margin: 0 auto;
  }
  .main-panel {
    display: flex;
    justify-content: center;
    width: 100%;
    padding: 0;
  }

  .display-panel {
    width: 80%;
    position: relative;
  }

  .control-panel {
    padding-left: 20px;
    width: 20%;
  }
  .console-panel {
    width: 100%;
  }

  .input-row {
    text-align: center;
    margin-top: 10px;
  }

  .input-row button {
    width: 50px;
    height: 30px;
    padding: 0;
    border-radius: 10px;
    color: #ff8c29;
    background-color: #111111;
    border: 2px solid #ff8c29;
    display: inline-grid;
    padding-top: 1px;
  }
</style>
