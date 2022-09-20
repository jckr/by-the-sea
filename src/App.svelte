<script>
  import {onMount} from 'svelte';
  $: permission = ''; 
  $: orientation = {
    alpha: 0,
    beta: 0,
    gamma: 0,
  };
  onMount(async () => {
    if (typeof DeviceOrientationEvent?.requestPermission === 'function') {
      DeviceOrientationEvent?.requestPermission()
        .then((response) => {
          permission = response;
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
      permission = 'no requestPermission function';
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
  <h3>{permission}</h3>
  {JSON.stringify(orientation)}
</main>

