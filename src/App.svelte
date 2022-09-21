<script>
  import {onMount} from 'svelte';
  $: permission = ''; 
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

  onMount(async () => {
    if (typeof DeviceOrientationEvent?.requestPermission === 'function') {
      DeviceOrientationEvent?.requestPermission()
        .then((response) => {
          permission = response;
          if (response == 'granted') {
            createListeners();
          }
        })
        .catch(console.error);
    } else {
      permission = 'no requestPermission function';
      Promise.resolve(
        createListeners();
      );
    }
  });
</script>

<main>
  <h1>Device Orientation</h1>
  <h3>{`Permission: ${permission}`}</h3>
  {JSON.stringify(orientation, null, 2)}
  <h3>Geolocation</h3>
  {JSON.stringify(location, null, 2)}
</main>

