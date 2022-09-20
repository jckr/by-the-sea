<script>
  import {onMount} from 'svelte';

  let orientation = {
    alpha: 0,
    beta: 0,
    gamma: 0,
  };
  onMount(async () => {
    if (typeof DeviceOrientationEvent?.requestPermission === 'function') {
      DeviceOrientationEvent?.requestPermission()
        .then((response) => {
          if (response == 'granted') {
            window.addEventListener('deviceorientation', (e) => {
              orientation = {
                alpha: e.alpha,
                beta: e.beta,
                gamma: e.gamma,
              };
            });
          }
        })
        .catch(console.error);
    } else {
      Promise.resolve(
        window.addEventListener('deviceorientation', (e) => {
          orientation = {
            alpha: e.alpha,
            beta: e.beta,
            gamma: e.gamma,
          };
        })
      );
    }
  });
</script>

<main>
  <h1>Device Orientation</h1>
  {JSON.stringify(orientation)}
</main>

