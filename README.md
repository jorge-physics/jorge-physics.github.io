# jorge-physics.github.io
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Wave Interference Demonstration</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background-color: #f0f0f0;
    }
    #controls {
      margin-top: 20px;
    }
    .slider-container {
      margin: 10px 0;
    }
    label {
      display: inline-block;
      width: 150px;
      text-align: right;
      margin-right: 10px;
    }
    input[type=range] {
      width: 300px;
    }
    #buttons {
      margin-top: 20px;
    }
    button {
      padding: 10px 20px;
      margin: 0 10px;
      font-size: 16px;
      cursor: pointer;
    }
  </style>
</head>
<body>

  <h1>Wave Interference Demonstration</h1>
  <div id="canvas-container">
    <!-- p5.js canvas will be inserted here -->
  </div>

  <div id="controls">
    <div class="slider-container">
      <label for="wavelength">Wavelength (λ):</label>
      <input type="range" id="wavelength" min="0.5" max="20" value="5" step="0.1">
      <span id="wavelength-value">5.0</span>
    </div>
    <div class="slider-container">
      <label for="amplitude">Amplitude (A):</label>
      <input type="range" id="amplitude" min="0.5" max="5" value="1" step="0.1">
      <span id="amplitude-value">1.0</span>
    </div>
    <div class="slider-container">
      <label for="phase">Phase (φ₁):</label>
      <input type="range" id="phase" min="-3.1416" max="3.1416" value="0" step="0.01">
      <span id="phase-value">0.00 rad</span>
    </div>
  </div>

  <div id="buttons">
    <button id="play-button">Play Sound</button>
    <button id="stop-button">Stop Sound</button>
  </div>

  <!-- Include p5.js and p5.sound.js libraries -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/addons/p5.sound.min.js"></script>

  <script>
    let wavelengthSlider, amplitudeSlider, phaseSlider;
    let wavelengthValue, amplitudeValue, phaseValue;
    let playButton, stopButton;
    let player; // p5.Gain object to control audio
    let osc1, osc2; // Oscillators for Wave 1 and Wave 2
    let isPlaying = false; // Flag to check if audio is playing

    function setup() {
      // Create canvas and attach to container
      let canvas = createCanvas(800, 400);
      canvas.parent('canvas-container');

      // Initialize sliders
      wavelengthSlider = select('#wavelength');
      amplitudeSlider = select('#amplitude');
      phaseSlider = select('#phase');

      // Initialize display spans
      wavelengthValue = select('#wavelength-value');
      amplitudeValue = select('#amplitude-value');
      phaseValue = select('#phase-value');

      // Initialize buttons
      playButton = select('#play-button');
      stopButton = select('#stop-button');

      // Attach button callbacks
      playButton.mousePressed(playSound);
      stopButton.mousePressed(stopSound);

      // Initialize oscillators and gain
      osc1 = new p5.Oscillator('sine');
      osc2 = new p5.Oscillator('sine');
      player = new p5.Gain();
      osc1.amp(0); // Start silent
      osc2.amp(0); // Start silent
      osc1.connect(player);
      osc2.connect(player);
      player.connect(); // Connect to master output

      // Initialize wave parameters
      updateOscillators();
    }

    function draw() {
      background(255);

      // Get current slider values
      let lambda = wavelengthSlider.value();
      let A = amplitudeSlider.value();
      let phi1 = phaseSlider.value();

      // Update display values
      wavelengthValue.html(lambda.toFixed(1));
      amplitudeValue.html(A.toFixed(1));
      phaseValue.html(phi1.toFixed(2) + ' rad');

      // Calculate wave number
      let k = TWO_PI / lambda;

      // Generate wave data
      let y_sum = [];
      let x_max = lambda * 100; // Scaling factor for visualization
      let x_values = linspace(0, x_max, width);

      for (let x = 0; x < width; x++) {
        let y1 = A * sin(k * x + phi1);
        let y2 = 1 * sin(k * x + PI / 2); // Wave 2 has fixed amplitude and phase
        let y_total = y1 + y2;
        y_sum.push(y_total);
      }

      // Find maximum absolute value for normalization
      let max_val = max(abs(y_sum));
      if (max_val === 0) {
        max_val = 1; // Prevent division by zero
      }

      // Draw Wave 1
      stroke('blue');
      noFill();
      beginShape();
      for (let x = 0; x < width; x++) {
        let y_val1 = A * sin(k * x + phi1);
        vertex(x, height / 2 - y_val1 / max_val * (height / 4));
      }
      endShape();

      // Draw Wave 2
      stroke('red');
      beginShape();
      for (let x = 0; x < width; x++) {
        let y_val2 = 1 * sin(k * x + PI / 2);
        vertex(x, height / 2 - y_val2 / max_val * (height / 4));
      }
      endShape();

      // Draw Sum Wave (Interference)
      stroke('black');
      strokeWeight(3);
      beginShape();
      for (let x = 0; x < width; x++) {
        vertex(x, height / 2 - y_sum[x] / max_val * (height / 4));
      }
      endShape();
    }

    // Function to handle Play Sound button
    function playSound() {
      if (isPlaying) {
        return; // Already playing
      }

      // Update oscillator parameters based on sliders
      updateOscillators();

      // Fade in amplitude
      osc1.amp(amplitudeSlider.value() * 0.1, 0.5); // Fade in over 0.5 seconds
      osc2.amp(1 * 0.1, 0.5); // Wave 2 has fixed amplitude

      isPlaying = true;
    }

    // Function to handle Stop Sound button
    function stopSound() {
      if (!isPlaying) {
        return; // Not playing
      }

      // Fade out amplitude
      osc1.amp(0, 0.5); // Fade out over 0.5 seconds
      osc2.amp(0, 0.5); // Fade out over 0.5 seconds

      // After fade-out, stop oscillators
      setTimeout(() => {
        osc1.stop();
        osc2.stop();
        isPlaying = false;
      }, 500); // Wait for fade-out to complete
    }

    // Function to update oscillator frequencies and phases based on sliders
    function updateOscillators() {
      let lambda = wavelengthSlider.value();
      let A = amplitudeSlider.value();
      let phi1 = phaseSlider.value();

      // Calculate frequencies based on wave speed
      let v = 1000; // Wave speed (units/s)
      let f1 = v / lambda;
      let f2 = v / 5; // Wave 2 frequency is fixed based on initial wavelength (lambda = 5)

      // Update oscillator frequencies and phases
      osc1.freq(f1);
      osc1.phase(phi1);
      osc2.freq(f2);
      osc2.phase(PI / 2); // Wave 2 has fixed phase
    }

    // Helper function to generate linearly spaced array
    function linspace(a, b, n) {
      let arr = [];
      let step = (b - a) / (n - 1);
      for (let i = 0; i < n; i++) {
        arr.push(a + step * i);
      }
      return arr;
    }

    // Cleanup when closing the window
    function windowResized() {
      // Optionally handle window resizing
    }
  </script>
</body>
</html>
