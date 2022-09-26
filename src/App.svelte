<script>
  import {onMount} from 'svelte';
  $: permission = 'unset';
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
    orientation.alpha += dx;
    if (shift) {
      orientation.gamma += dy;
    } else {
      orientation.beta += dy;
    }
    orientation = sanitize(orientation);
  }
  function sanitize({alpha, beta, gamma}) {
    alpha = (alpha + 360) % 360;
    beta = Math.max(-90, Math.min(90, beta));
    gamma = Math.max(-90, Math.min(90, gamma));
    return {alpha, beta, gamma};
  }
  function createListeners() {
    window.addEventListener('deviceorientation', (e) => {
      orientation = {
        alpha: e.alpha,
        beta: e.beta,
        gamma: e.gamma,
      };
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
    DeviceOrientationEvent?.requestPermission()
      .then((response) => {
        permission = response;
        createListeners();
      })
      .catch((error) => {
        permission = e;
      });
  }

  onMount(() => {
    if (typeof DeviceOrientationEvent?.requestPermission === 'function') {
      permission = 'unset';
    } else {
      permission = 'no requestPermission function';
      createListeners();
    }
  });
</script>

<main
  on:touchstart={handleFirstGesture}
  on:mousedown={handleDragStart}
  on:mousemove={handleDrag}
  on:mouseup={handleDragEnd}
>
  <h1>Device Orientation</h1>
  <h3>{`Permission: ${permission}`}</h3>
  {JSON.stringify(orientation, null, 2)}
  <h3>Geolocation</h3>
  {JSON.stringify(location, null, 2)}
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
