<script>
  let state = "Connect";
  let flipper = null;
  let reader;
  let do_read = false;

  export let parse_display_callback = null;
  export let disconnect_callback = null;
  export let connect_callback = null;

  const filter_flipper = { usbVendorId: 1155, usbProductId: 22336 };

  export async function send_command(command) {
    if (flipper != null) {
      try {
        if (flipper.writable) {
          const textEncoder = new TextEncoderStream();
          const writableStreamClosed = textEncoder.readable.pipeTo(
            flipper.writable
          );
          const writer = textEncoder.writable.getWriter();
          await writer.write(command + "\r\n");
          writer.close();
        }
      } catch (error) {
        console.error(error);
      } finally {
      }
    }
  }

  function on_connect() {
    console.log("on_connect");
    get_port().then(function (ports) {
      if (ports !== null && Array.isArray(ports) && ports.length > 0) {
        flipper = ports[0];
        if (connect_callback != null) {
          connect_callback();
        }
        connect(flipper);
      }
    });
  }

  function on_disconnect() {
    console.log("on_disconnect");
    disconnect(flipper).then(() => {
      flipper = null;
      if (disconnect_callback != null) {
        disconnect_callback();
      }
    });
  }

  async function get_port() {
    return await navigator.serial.getPorts({ filters: [filter_flipper] });
  }

  async function request_port() {
    return await navigator.serial.requestPort({ filters: [filter_flipper] });
  }

  async function disconnect(port) {
    try {
      do_read = false;
      if (reader.locked) {
        reader.cancel();
      }
      await port.close();
    } catch (error) {
      console.error(error);
    }

    state = "Connection lost";
  }

  async function get_display() {
    await send_command("screen_stream");
  }

  async function connect(port) {
    await port.open({ baudRate: 115200 });
    state = "Disconnect";

    do_read = true;

    let screen_array = new Uint8Array(1024);
    let screen_array_index = 0;
    let screen_array_fill = false;

    let screen_header = new Uint8Array([0xf0, 0xe1, 0xd2, 0xc3]);
    let screen_header_index = 0;

    get_display();

    while (port.readable && do_read) {
      reader = port.readable.getReader();
      try {
        while (true) {
          const { value, done } = await reader.read();
          if (done) {
            console.log("done");
            break;
          }

          for (var i = 0; i < value.length; i++) {
            if (value[i] == screen_header[screen_header_index]) {
              screen_header_index++;
              if (screen_header_index >= screen_header.length) {
                screen_array_fill = true;
                //console.log("screen found");
                continue;
              }
            } else {
              screen_header_index = 0;
            }

            if (screen_array_fill) {
              screen_array[screen_array_index] = value[i];
              screen_array_index++;
              if (screen_array_index >= screen_array.length) {
                screen_array_index = 0;
                screen_array_fill = false;
                //console.log("screen ok");
                if (parse_display_callback != null) {
                  parse_display_callback(screen_array);
                  setTimeout(() => {
                    get_display();
                  }, 100);
                }
              }
            }
          }
        }
      } catch (error) {
        console.error(error);
      } finally {
        reader.releaseLock();
      }
    }
  }

  if ("serial" in navigator) {
    navigator.serial.onconnect = on_connect;
    navigator.serial.ondisconnect = on_disconnect;

    get_port().then(function (ports) {
      if (ports !== null && Array.isArray(ports) && ports.length > 0) {
        flipper = ports[0];
        console.log(flipper);

        if (flipper.length == 0) {
          state = "Get Access";
        } else {
          connect(flipper);
        }
      }
    });
  } else {
    state = "Unsupported";
  }

  function mouseClick() {
    if (flipper == null) {
      request_port().then(function (port) {
        flipper = port;
        console.log(flipper);

        if (flipper.length != 0) {
          flipper.onconnect = on_connect;
          connect(flipper);
        }
      });
    } else {
      disconnect(flipper);
    }
  }
</script>

<button on:click={mouseClick}>{state}</button>
<button on:click={get_display}>1234</button>
