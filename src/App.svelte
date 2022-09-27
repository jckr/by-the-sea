<script>
  import {onMount} from 'svelte';
  $: permission = 'unset';
  $: status = 'default';
  $: orientation = {
    absolute: null,
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
    orientation.alpha += 0.1 * dx;
    if (shift) {
      orientation.gamma += 0.1 * dy;
    } else {
      orientation.beta += 0.1 * dy;
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
    status = 'listening';
    window.addEventListener('deviceorientation', (e) => {
      orientation = {
        absolute: e.absolute,
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
  function handlePhoto(e) {
    if (e.target.files[0]) {
      const {name, type, lastModified} = e.target.files[0];
      const record = {
        name,
        type,
        lastModified,
        ...orientation,
        ...location,
      };
      console.log(record);
    }
  }
  onMount(() => {
    status = 'mounted';
    if (typeof DeviceOrientationEvent?.requestPermission === 'function') {
      permission = 'unset';
    } else {
      permission = 'no requestPermission function';
      createListeners();
    }
  });
</script>

<main
  on:click={handleFirstGesture}
  on:mousedown={handleDragStart}
  on:mousemove={handleDrag}
  on:mouseup={handleDragEnd}
>
  <section class="camera">
    <label for="camera-input" class="camera-label">Take photo</label>
    <input id="camera-input" on:change={handlePhoto} type="file" accept="image/*" />
  </section>
  <section>
    <h3>Debug status</h3>
    <p>Permission: {permission}</p>
    <p>Status: {status}</p>
    <h3>Device orientation</h3>
    <p>Absolute: {orientation.absolute}</p>
    <p>Alpha: {orientation.alpha}</p>
    <p>Beta: {orientation.beta}</p>
    <p>Gamma: {orientation.gamma}</p>
    <h3>Device Location</h3>
    <p>Latitude: {location.latitude}</p>
    <p>Longitude: {location.longitude}</p>
    <p>Altitude: {location.altitude}</p>
    <p>Accuracy: {location.accuracy}</p>
    <p>Altitude Accuracy: {location.altitudeAccuracy}</p>
    <p>Heading: {location.heading}</p>
    <p>Speed: {location.speed}</p>
  </section>
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
  .camera .camera-label {
    cursor: pointer;
    display: block;
    line-height: 3rem;
    text-align: center;
    border-radius: 4px;
    box-shadow: 2px 2px 4px rgb(0 0 0 / 20%);
    min-height: 3rem;
    min-width: 10rem;
    background: #fff;
  }
  .camera #camera-input {
    display: none;
  }
</style>
