<script>
  import {onMount} from 'svelte';
  $: permission = 'unset';
  $: status = 'default';
  $: orientation = {
    alpha: 0,
    beta: 0,
    gamma: 0,
  };
  $: location = {
    latitude: 0,
    longitude: 0,
    altitude: 0,
    accuracy: 0,
    altitudeAccuracy: 0,
    heading: 0,
    speed: 0,
  };
  let canvas, ctx;

  let x = 0,
    y = 0,
    dragging = false;
  let shift = false;
  function handleDragStart(e) {
    x = e.clientX;
    y = e.clientY;
    dragging = true;
    shift = e.shiftKey;
  }
  function handleDragEnd(e) {
    x = e.clientX;
    y = e.clientY;
    dragging = false;
  }
  function handleDrag(e) {
    if (dragging === false) return;
    const dx = e.clientX - x;
    const dy = e.clientY - y;
    x += dx;
    y += dy;
    orientation.alpha += 0.1 * dx;
    if (shift) {
      orientation.gamma += 0.1 * dy;
    } else {
      orientation.beta += 0.1 * dy;
    }
    orientation = sanitize(orientation);
    requestAnimationFrame(updateCanvas);
  }
  function sanitize({alpha, beta, gamma}) {
    alpha = (alpha + 360) % 360;
    beta = Math.max(-90, Math.min(90, beta));
    gamma = Math.max(-90, Math.min(90, gamma));
    return {alpha, beta, gamma};
  }
  function createListeners() {
    status = 'listening';
    window.addEventListener('deviceorientation', (e) => {
      orientation = {
        alpha: e.alpha,
        beta: e.beta,
        gamma: e.gamma,
      };
      requestAnimationFrame(updateCanvas);
    });
    window.addEventListener('orientationchange', (e) => {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      requestAnimationFrame(updateCanvas);
    });
    navigator?.geolocation?.watchPosition((position) => {
      location = {
        latitude: position.coords.latitude,
        longitude: position.coords.longitude,
        altitude: position.coords.altitude,
        accuracy: position.coords.accuracy,
        altitudeAccuracy: position.coords.altitudeAccuracy,
        heading: position.coords.heading,
        speed: position.coords.speed,
      };
    });
  }
  async function handleFirstGesture(e) {
    if (typeof DeviceOrientationEvent?.requestPermission !== 'function' || permission !== 'unset')
      return;
    status = 'permission requested';
    DeviceOrientationEvent?.requestPermission()
      .then((response) => {
        permission = response;
        createListeners();
      })
      .catch((error) => {
        permission = e.message;
      });
  }

  onMount(() => {
    canvas.width = window.innerWidth;
    canvas.height=window.innerHeight;
    ctx = canvas.getContext('2d');
    status = 'mounted';
    if (typeof DeviceOrientationEvent?.requestPermission === 'function') {
      permission = 'unset';
    } else {
      permission = 'no requestPermission function';
      createListeners();
    }
  });

  function updateCanvas() {
    if (canvas === undefined || ctx === undefined) {return;}
    ctx.fillStyle = '#8ad1db';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = '#558688'
    const {alpha, beta, gamma} = orientation;
    const horizon_angle = -Math.PI / 16 + gamma;

    const yHorizon = (canvas?.height ?? 0) * (0.5 + Math.sin(horizon_angle));
    const yHorizonDelta = Math.sin(beta) * (canvas?.width ?? 0) * 0.5;
    ctx.beginPath();
    ctx.moveTo(0, canvas.height);
    ctx.lineTo(0, yHorizon - yHorizonDelta);
    ctx.lineTo(canvas.width, yHorizon + yHorizonDelta);
    ctx.lineTo(canvas.width, canvas.height);
    ctx.closePath();
    ctx.fill();
  }

</script>

<main
  on:click={handleFirstGesture}
  on:mousedown={handleDragStart}
  on:mousemove={handleDrag}
  on:mouseup={handleDragEnd}
>
  <canvas bind:this={canvas}></canvas>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    width: 100vw;
    background: #eee;
  }
</style>
