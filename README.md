<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>🧊 3D Ice Crystal</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
  body {
    margin: 0;
    background: #000010;
    verflow: hidden;
  }
  canvas {
    display: block;
    filter: brightness(1.4) blur(0.6px);
  }
</style>
</head>

<body>
<canvas id="crystal"></canvas>

<script>
// --- Setup ---
const canvas = document.getElementById("crystal");
const ctx = canvas.getContext("2d");

let w, h;
function resize() {
  w = canvas.width = window.innerWidth;
  h = canvas.height = window.innerHeight;
}
window.addEventListener("resize", resize);
resize();

let t = 0;

// Signed Distance Function for crystalline geometry
function crystalSDF(x, y, z) {
  return Math.max(
    Math.abs(x) + Math.abs(y) + Math.abs(z) - 1.2,
    Math.max(
      Math.abs(x * 1.3 + y * 0.4) - 1.2,
      Math.abs(y * 1.1 - z * 0.5) - 1.0
    )
  );
}

// Raymarch algorithm (simplified)
function raymarch(xx, yy, time) {
  let ox = 0, oy = 0, oz = -4; // camera origin
  let dx = (xx - w / 2) / h;
  let dy = (yy - h / 2) / h;
  let dz = 1;

  let dist, totalDist = 0;

  for (let i = 0; i < 60; i++) {
    let x = ox + dx * totalDist;
    let y = oy + dy * totalDist;
    let z = oz + dz * totalDist;

    // animation: rotate the crystal
    let angle = time * 0.7;
    let rx = x * Math.cos(angle) - z * Math.sin(angle);
    let rz = x * Math.sin(angle) + z * Math.cos(angle);
    let ry = y;

    dist = crystalSDF(rx, ry, rz);

    if (dist < 0.01) break;
    totalDist += dist * 0.8;
  }

  if (dist < 0.01) {
    let glow = Math.max(0, 1 - totalDist / 4);
    return glow;
  }

  return 0;
}

// Render loop
function draw() {
  let img = ctx.createImageData(w, h);
  let data = img.data;

  let index = 0;

  for (let y = 0; y < h; y++) {
    for (let x = 0; x < w; x++) {
      let v = raymarch(x, y, t);

      // Blue–white ice glow
      let hue = 180 + v * 40;
      let light = 30 + v * 70;
      let sat = 20 + v * 80;

      // convert HSL to RGB
      let c = (1 - Math.abs((2 * light / 100) - 1));
      let x2 = c * (1 - Math.abs((hue / 60) % 2 - 1));
      let m = light / 100 - c / 2;
      let r,g,b;

      if (hue < 60) [r,g,b] = [c, x2, 0];
      else if (hue < 120) [r,g,b] = [x2, c, 0];
      else if (hue < 180) [r,g,b] = [0, c, x2];
      else if (hue < 240) [r,g,b] = [0, x2, c];
      else if (hue < 300) [r,g,b] = [x2, 0, c];
      else [r,g,b] = [c, 0, x2];

      data[index++] = (r + m) * 255;
      data[index++] = (g + m) * 255;
      data[index++] = (b + m) * 255;
      data[index++] = 255;
    }
  }

  ctx.putImageData(img, 0, 0);

  t += 0.02;
  requestAnimationFrame(draw);
}

draw();
</script>

</body>
</html>
