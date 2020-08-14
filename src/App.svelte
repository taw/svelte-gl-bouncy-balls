<script>
  import { onMount } from 'svelte';
  import * as GL from '@sveltejs/gl';

  let rand = (min, max) => {
    return min + (max - min) * Math.random();
  }
  // these are mostly bad colors
  let rand_color = () => {
    return Math.floor(Math.random() * 0x1000000);
  }

  let light = {};
  let balls = [];

  for(let i=0; i<10; i++) {

    balls.push({
      x: rand(-4, 4),
      y: rand(0.5, 2),
      z: rand(-4, 4),
      dx: rand(-1, 1),
      dy: rand(-1, 1),
      dz: rand(-1, 1),
      color: rand_color(),
      sz: rand(0.2, 0.5),
    });
  }
  onMount(() => {
    let frame;

    let loop = () => {
      frame = requestAnimationFrame(loop);

      light.x = 3 * Math.sin(Date.now() * 0.001);
      light.y = 2.5 + 2 * Math.sin(Date.now() * 0.0004);
      light.z = 3 * Math.cos(Date.now() * 0.002);

      let dt = 0.02;
      let g = 0.01;
      let max_x = 5;
      let min_x = -5;
      let max_z = 5;
      let min_z = -5;

      for (let ball of balls) {
        // move
        ball.x += dt * ball.dx;
        ball.y += dt * ball.dy;
        ball.z += dt * ball.dz;
        // gravity
        ball.dy -= g;
        // bounce if hit a wall
        if (ball.y - ball.sz <= ball.sz) {
          ball.dy = Math.abs(ball.dy);
        }
        if (ball.x + ball.sz >= max_x) {
          ball.dx = -Math.abs(ball.dx);
        }
        if (ball.x - ball.sz <= min_x) {
          ball.dx = Math.abs(ball.dx);
        }
        if (ball.z + ball.sz >= max_z) {
          ball.dz = -Math.abs(ball.dz);
        }
        if (ball.z - ball.sz <= min_z) {
          ball.dz = Math.abs(ball.dz);
        }
      }
      // tell svelte it updated
      balls = balls;
    };

    loop();

    return () => cancelAnimationFrame(frame);
  });
</script>

<GL.Scene>
  <GL.Target id="center" location={[0, 0, 0]}/>

  <GL.OrbitControls maxPolarAngle={Math.PI / 2} let:location>
    <GL.PerspectiveCamera {location} lookAt="center" near={0.01} far={1000}/>
  </GL.OrbitControls>

  <GL.AmbientLight intensity={0.3}/>
  <GL.DirectionalLight direction={[-1,-1,-1]} intensity={0.5}/>

  <!-- floor -->
  <GL.Mesh
    geometry={GL.plane()}
    location={[0,-0.01,0]}
    rotation={[-90,0,0]}
    scale={10}
    uniforms={{ color: 0xffffff }}
  />

  <!-- spheres -->
  {#each balls as {x,y,z,color,sz}}
    <GL.Mesh
      geometry={GL.sphere({ turns: 36, bands: 36 })}
      location={[x,y,z]}
      scale={sz}
      uniforms={{ color, alpha: 0.9 }}
      transparent
    />
  {/each}

  <!-- moving light -->
  <GL.Group location={[light.x,light.y,light.z]}>
    <GL.Mesh
      geometry={GL.sphere({ turns: 36, bands: 36 })}
      location={[0,0.2,0]}
      scale={0.1}
      uniforms={{ color: 0xffffff, emissive: 0xcccc99 }}
    />

    <GL.PointLight
      location={[0,0,0]}
      color={0xffffff}
      intensity={0.6}
    />
  </GL.Group>
</GL.Scene>

<style>
</style>
