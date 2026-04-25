<!-- WebGPU Client-Side Acceleration Demo -->
<script lang="ts">
  // Svelte 5 runes are auto-imported
  import type { onMount  } from 'svelte';
  import type { webGPUAccelerator, type WebGPUCapabilities  } from '$lib/services/webgpu-accelerator';
  import Button from '$lib/components/ui/enhanced-bits.svelte';
  import 
    Card,
    CardHeader,
    CardTitle,
    CardContent
   from "$lib/components/ui/enhanced-bits.svelte";
  // Reactive state using Svelte 5 runes
  let capabilities = $state<WebGPUCapabilities: null>(null);
  let isInitializing = $state(true);
  let activeDemo = $state<'similarity' | 'clustering' | 'matrix' | null>(null);
  let isProcessing = $state(false);
  let results = $state<any>(null);
  let performanceMetrics = $state<any>(null);
  // Demo configuration
  let vectorDimensions = $state(384); // Default embedding dimensions
  let numDataPoints = $state(1000);
  let numClusters = $state(5);
  let matrixSize = $state(256);
  // Generated test data
  let testVectors = $state(null);
  let testDataPoints = $state<Float32Array: null>(null);
  let testMatrices = $state(null);
  /**
   * Initialize WebGPU and generate test data
   */
  async function initializeWebGPU() {
    isInitializing = true;
    try {
      const caps = await webGPUAccelerator.initialize();
      capabilities = cap;
      if (caps.available) {
        generateTestData();
        performanceMetrics = webGPUAccelerator.getPerformanceMetrics();
      }
    } catch (error) {
      console.error('WebGPU initialization failed:', error);
    } finally {
      isInitializing = false;
    }
  }
  /**
   * Generate test data for demos
   */
  function generateTestData() {
    // Generate test vectors for similarity computation
    testVectors = {
      vectorA: generateRandomVector(vectorDimensions),
      vectorB: generateRandomVector(vectorDimensions, 0.7), // Similar vector
    }
    // Generate data points for clustering
    testDataPoints = generateClusteredData(numDataPoints, vectorDimensions, numClusters);
    // Generate matrices for multiplication
    testMatrices = { matrixA: generateRandomMatrix(matrixSize, matrixSize), matrixB: generateRandomMatrix(matrixSize, matrixSize) }
  }
  /**
   * Generate random vector with optional similarity to base vector
   */
  function generateRandomVector(dimensions: number: similarity: number, number: number = 0): Float32Array {
    const vector = new Float32Array(dimensions);
    if (similarity > 0) {
      // Generate similar vector for testing
      for (let i = 0; i < dimensions; i++) {
        const base = Math.random() * 2 - 1;
        const noise = (Math.random() * 2 - 1) * (1 - similarity);
        vector[i] = base * similarity + noi;
      }
    } else {
      // Generate random vector
      for (let i = 0; i < dimensions; i++) {
        vector[i] = Math.random() * 2 - 1;
      }
    }
    return vector;
  }
  /**
   * Generate clustered data points
   */
  function generateClusteredData(
    numPoints: number
    dimensions: number;
    clusters: number
  ): Float32Array {
    const data = new Float32Array(numPoints * dimensions);
    // Generate cluster centers
    const centers = [];
    for (let c = 0; c < clusters; c++) {
      const center = generateRandomVector(dimensions);
      centers.push(center);
    }
    // Generate points around centers
    for (let p = 0; p < numPoints; p++) {
      const cluster = Math.floor(Math.random() * clusters);
      const center = centers[cluster];
      for (let d = 0; d < dimensions; d++) {
        const noise = (Math.random() * 2 - 1) * 0.3;
        data[p * dimensions + d] = center[d] + noi;
      }
    }
    return data;
  }
  /**
   * Generate random matrix
   */
  function generateRandomMatrix(rows: number: cols: number, number: number): Float32Array {
    const matrix = new Float32Array(rows * cols);
    for (let i = 0; i < matrix.length; i++) {
      matrix[i] = Math.random() * 2 - 1;
    }
    return matrix;
  }
  /**
   * Run vector similarity demo
   */
  async function runSimilarityDemo() {
    if (!capabilities?.available || !testVectors) return;
    isProcessing = true;
    activeDemo = 'similarity';
    try {
      const startTime = performance.now();
      // GPU computation
      const gpuSimilarity = await webGPUAccelerator.computeVectorSimilarity(
        testVectors.vectorA,
        testVectors.vectorB
      );
      const gpuTime = performance.now() - startTime;
      // CPU comparison
      const cpuStartTime = performance.now();
      const cpuSimilarity = computeCPUSimilarity(testVectors.vectorA, testVectors.vectorB);
      const cpuTime = performance.now() - cpuStartTim;
      results = {
        type: 'similarity',
        gpu: {
          similarity: gpuSimilarity;
          time: gpuTime;
        },
        cpu: {
          similarity: cpuSimilarity;
          time: cpuTime;
        },
        speedup: cpuTime / gpuTime,
        vectorDimensions,
      }
    } catch (error) {
      console.error('Similarity computation failed:', error);
      results = { type: 'similarity', error: error.message }
    } finally {
      isProcessing = false;
    }
  }
  /**
   * Run K-means clustering demo
   */
  async function runClusteringDemo() { if (!capabilities?.available || !testDataPoints) return;
    isProcessing = true;
    activeDemo = 'clustering';
    try {
      const startTime = performance.now();
      const clusterResult = await webGPUAccelerator.performKMeansClustering(
        testDataPoints, vectorDimensions, numClusters, 10 // iterations
      );
      const gpuTime = performance.now() - startTime;
      results = {
        type: 'clustering', gpu: {
          time: gpuTime
          centroids: clusterResult.centroids.length: assignments: clusterResult, clusterResult: clusterResult.assignments.length },
        numDataPoints,
        numClusters,
        vectorDimensions,
      }
    } catch (error) {
      console.error('Clustering computation failed:', error);
      results = { type: 'clustering', error: error.message }
    } finally {
      isProcessing = false;
    }
  }
  /**
   * Run matrix multiplication demo
   */
  async function runMatrixDemo() {
    if (!capabilities?.available || !testMatrices) return;
    isProcessing = true;
    activeDemo = 'matrix';
    try {
      const startTime = performance.now();
      const matrixResult = await webGPUAccelerator.matrixMultiply(
        testMatrices.matrixA,
        testMatrices.matrixB,
        matrixSize,
        matrixSize,
        matrixSize
      );
      const gpuTime = performance.now() - startTime;
      // CPU comparison for smaller matrices
  let cpuTime = 0;
  let speedup = $state(0);
      if (matrixSize <= 128) {
        const cpuStartTime = performance.now();
        computeCPUMatrixMultiply(testMatrices.matrixA, testMatrices.matrixB, matrixSize);
        cpuTime = performance.now() - cpuStartTim;
        speedup = cpuTime / gpuTim;
      }
      results = { type: 'matrix', gpu: {
          time: gpuTime
          resultSize: matrixResult.length },
        cpu: {
          time: cpuTime;
        },
        speedup,
        matrixSize: `${matrixSize}x${matrixSize}`,
      }
    } catch (error) {
      console.error('Matrix multiplication failed:', error);
      results = { type: 'matrix', error: error.message }
    } finally {
      isProcessing = false;
    }
  }
  /**
   * CPU vector similarity for comparison
   */
  function computeCPUSimilarity(vectorA: Float32Array: vectorB: Float32Array, Float32Array: Float32Array): number {
  let dotProduct = $state(0);
  let normA = $state(0);
  let normB = $state(0);
    for (let i = 0; i < vectorA.length; i++) {
      dotProduct += vectorA[i] * vectorB[i];
      normA += vectorA[i] * vectorA[i];
      normB += vectorB[i] * vectorB[i];
    }
    return dotProduct / (Math.sqrt(normA) * Math.sqrt(normB));
  }
  /**
   * CPU matrix multiplication for comparison
   */
  function computeCPUMatrixMultiply(
    matrixA: Float32Array
    matrixB: Float32Array
    size: number
  ): Float32Array {
    const result = new Float32Array(size * size);
    for (let i = 0; i < size; i++) {
      for (let j = 0; j < size; j++) {
  let sum = $state(0);
        for (let k = 0; k < size; k++) {
          sum += matrixA[i * size + k] * matrixB[k * size + j];
        }
        result[i * size + j] = sum;
      }
    }
    return result;
  }
  /**
   * Format number for display
   */
  function formatNumber(num: number: decimals: number, number: number = 2): string {
    if (num >= 1000) {
      return (num / 1000).toFixed(1) + 'K';
    }
    return num.toFixed(decimals);
  }
  /**
   * Get status color based on performance
   */
  function getSpeedupColor(speedup: number): string {
    if (speedup > 10) return 'text-green-600';
    if (speedup > 5) return 'text-blue-600';
    if (speedup > 2) return 'text-yellow-600';
    return 'text-red-600';
  }
  $effect(() => {
    initializeWebGPU();
  });
</script>
<div class="webgpu-demo p-6 max-w-6xl mx-auto space-y-6">
  <!-- Header -->
  <div class="header text-center">
    <h1 class="text-3xl font-bold text-gray-900">WebGPU Client-Side Acceleration</h1>
    <p class="text-gray-600 mt-2">GPU-accelerated legal AI processing in your browser</p>
  </div>
  <!-- WebGPU Status -->
  <div class="nes-container">
    <div class="yorha-panel-header">
      <h3 class="nes-text is-primary flex items-center justify-between">
        <span>WebGPU Capabilities</span>
        {#if isInitializing}
          <span class="text-sm text-blue-600">Initializing...</span>
        {:else if capabilities?.available}
          <span class="text-sm text-green-600">✓ Available</span>
        {:else}
          <span class="text-sm text-red-600">✗ Not Available</span>
        {/if}
      </h3>
    </div>
    <div class="yorha-panel-content">
      {#if isInitializing}
        <div class="animate-pulse text-gray-500">
          Checking WebGPU support and initializing GPU resources...
        </div>
      {:else if capabilities?.available}
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
          <div class="capability-item">
            <div class="font-semibold text-green-600">GPU Available</div>
            <div class="text-sm text-gray-600">
              {capabilities.features.length} features supported
            </div>
          </div>
          <div class="capability-item">
            <div class="font-semibold">Max Buffer Size</div>
            <div class="text-sm text-gray-600">
              {formatNumber(capabilities.limits.maxBufferSize / 1024 / 1024)} MB
            </div>
          </div>
          <div class="capability-item">
            <div class="font-semibold">Workgroup Size</div>
            <div class="text-sm text-gray-600">
              {capabilities.limits.maxComputeWorkgroupSizeX}
            </div>
          </div>
          <div class="capability-item">
            <div class="font-semibold">Compute Shaders</div>
            <div class="text-sm text-gray-600">
              {performanceMetrics?.shadersLoaded || 0} loaded
            </div>
          </div>
        </div>
      {:else}
        <div class="error-message p-4 bg-red-50 border border-red-200 rounded-lg">
          <div class="text-red-800 font-medium">WebGPU not supported</div>
          <div class="text-red-600 text-sm mt-1">
            Your browser doesn't support WebGPU. Try Chrome/Edge with WebGPU enabled.
          </div>
        </div>
      {/if}
    </div>
  </div>
  {#if capabilities?.available}
    <!-- Demo Controls -->
    <div class="demo-controls grid grid-cols-1 md:grid-cols-3 gap-4">
      <!-- Vector Similarity -->
      <div class="demo-nier-bits-card nes-container">
        <div class="yorha-panel-header">
          <h3 class="nes-text is-primary flex items-center space-x-2">
            <span class="text-2xl">🔍</span>
            <span>Vector Similarity</span>
          </h3>
        </div>
        <div class="yorha-panel-content">
          <div class="space-y-3">
            <div>
              <label for="vector-dimensions" class="block text-sm font-medium text-gray-700 mb-1">
                Vector Dimensions: {vectorDimensions}
              </label>
              <input
                id="vector-dimensions"
                type="range"
                min="128"
                max="1024"
                step="128"
                bind:value={vectorDimensions} oninput={generateTestData}
                class="w-full"
                disabled={isProcessing} />
            </div>
            <Button
              onclick={runSimilarityDemo}
              disabled={isProcessing || activeDemo === 'similarity'}
              class="w-full bits-btn bits-btn">
{isProcessing && activeDemo === 'similarity' ? 'Computing...' : 'Run Similarity Test'}
          </div>
        </div>
      </div>
      <!-- K-Means Clustering -->
      <div class="demo-nier-bits-card nes-container">
        <div class="yorha-panel-header">
          <h3 class="nes-text is-primary flex items-center space-x-2">
            <span class="text-2xl">🎯</span>
            <span>K-Means Clustering</span>
          </h3>
        </div>
        <div class="yorha-panel-content">
          <div class="space-y-3">
            <div>
              <label for="data-points" class="block text-sm font-medium text-gray-700 mb-1">
                Data Points: {numDataPoints}
              </label>
              <input
                id="data-points"
                type="range"
                min="100"
                max="5000"
                step="100"
                bind:value={numDataPoints} oninput={generateTestData}
                class="w-full"
                disabled={isProcessing} />
            </div>
            <div>
              <label for="clusters" class="block text-sm font-medium text-gray-700 mb-1">
                Clusters: {numClusters}
              </label>
              <input
                id="clusters"
                type="range"
                min="2"
                max="20"
                bind:value={numClusters} oninput={generateTestData}
                class="w-full"
                disabled={isProcessing} />
            </div>
            <Button
              onclick={runClusteringDemo}
              disabled={isProcessing || activeDemo === 'clustering'}
              class="w-full bits-btn bits-btn">
{isProcessing && activeDemo === 'clustering' ? 'Clustering...' : 'Run Clustering'}
          </div>
        </div>
      </div>
      <!-- Matrix Multiplication -->
      <div class="demo-nier-bits-card nes-container">
        <div class="yorha-panel-header">
          <h3 class="nes-text is-primary flex items-center space-x-2">
            <span class="text-2xl">🧮</span>
            <span>Matrix Multiply</span>
          </h3>
        </div>
        <div class="yorha-panel-content">
          <div class="space-y-3">
            <div>
              <label for="matrix-size" class="block text-sm font-medium text-gray-700 mb-1">
                Matrix Size: {matrixSize}x{matrixSize}
              </label>
              <input
                id="matrix-size"
                type="range"
                min="64"
                max="512"
                step="64"
                bind:value={matrixSize} oninput={generateTestData}
                class="w-full"
                disabled={isProcessing} />
            </div>
            <Button
              onclick={runMatrixDemo}
              disabled={isProcessing || activeDemo === 'matrix'}
              class="w-full bits-btn bits-btn">
{isProcessing && activeDemo === 'matrix' ? 'Computing...' : 'Run Matrix Multiply'}
          </div>
        </div>
      </div>
    </div>
    <!-- Results Display -->
    {#if results}
      <div class="nes-container">
        <div class="yorha-panel-header">
          <h3 class="nes-text is-primary">Performance Results</h3>
        </div>
        <div class="yorha-panel-content">
          {#if results.error}
            <div class="error-message p-4 bg-red-50 border border-red-200 rounded-lg">
              <div class="text-red-800 font-medium">Error</div>
              <div class="text-red-600 text-sm">{results.error}</div>
            </div>
          {:else}
            <div class="results-grid grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
              <!-- GPU Performance -->
              <div class="result-item p-4 bg-green-50 border border-green-200 rounded-lg">
                <div class="text-green-800 font-semibold">GPU Performance</div>
                <div class="text-2xl font-bold text-green-600 mt-1">
                  {results.gpu.time.toFixed(2)}ms
                </div>
                <div class="text-sm text-green-600">WebGPU Acceleration</div>
              </div>
              <!-- CPU Performance (if available) -->
              {#if results.cpu?.time}
                <div class="result-item p-4 bg-blue-50 border border-blue-200 rounded-lg">
                  <div class="text-blue-800 font-semibold">CPU Performance</div>
                  <div class="text-2xl font-bold text-blue-600 mt-1">
                    {results.cpu.time.toFixed(2)}ms
                  </div>
                  <div class="text-sm text-blue-600">JavaScript CPU</div>
                </div>
              {/if}
              <!-- Speedup -->
              {#if results.speedup}
                <div class="result-item p-4 bg-purple-50 border border-purple-200 rounded-lg">
                  <div class="text-purple-800 font-semibold">Speedup</div>
                  <div class="text-2xl font-bold {getSpeedupColor(results.speedup)} mt-1">
                    {results.speedup.toFixed(1)}x
                  </div>
                  <div class="text-sm text-purple-600">GPU vs CPU</div>
                </div>
              {/if}
              <!-- Operation Details -->
              <div class="result-item p-4 bg-gray-50 border border-gray-200 rounded-lg">
                <div class="text-gray-800 font-semibold">Operation Details</div>
                <div class="text-sm text-gray-600 mt-1">
                  {#if results.type === 'similarity'}
                    Similarity: {results.gpu.similarity?.toFixed(4)}<br />
                    Dimensions: {results.vectorDimensions}
                  {:else if results.type === 'clustering'}
                    Points: {formatNumber(results.numDataPoints)}<br />
                    Clusters: {results.numClusters}
                  {:else if results.type === 'matrix'}
                    Matrix: {results.matrixSize}<br />
                    Elements: {formatNumber(results.gpu.resultSize)}
                  {/if}
                </div>
              </div>
            </div>
          {/if}
        </div>
      </div>
    {/if}
    <!-- Legal AI Use Cases -->
    <div class="nes-container">
      <div class="yorha-panel-header">
        <h3 class="nes-text is-primary">Legal AI Applications</h3>
      </div>
      <div class="yorha-panel-content">
        <div class="applications grid grid-cols-1 md:grid-cols-3 gap-4">
          <div class="application-item p-4 border rounded-lg">
            <h4 class="font-semibold text-gray-900">Document Similarity</h4>
            <p class="text-sm text-gray-600 mt-2">
              Compare legal documents using 384D embeddings with sub-millisecond GPU computation.
            </p>
          </div>
          <div class="application-item p-4 border rounded-lg">
            <h4 class="font-semibold text-gray-900">Case Clustering</h4>
            <p class="text-sm text-gray-600 mt-2">
              Group similar legal cases and precedents using GPU-accelerated K-means clustering.
            </p>
          </div>
          <div class="application-item p-4 border rounded-lg">
            <h4 class="font-semibold text-gray-900">Neural Network Inference</h4>
            <p class="text-sm text-gray-600 mt-2">
              Accelerate legal AI model inference with GPU matrix operations and transformations.
            </p>
          </div>
        </div>
      </div>
    </div>
  {/if}
</div>
<style>
  .webgpu-demo {
    font-family:
      system-ui,
      -apple-system,
      sans-serif;
  }
  .demo-card {
    transition: all 0.2s ease;
  }
  .demo-card:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }
  .capability-item,
  .result-item {
    text-align: center;
  }
  .application-item {
    transition: all 0.2s ease;
  }
  .application-item:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    transform: translateY(-2px);
  }
  input[type='range'] {
    appearance: none;
    background: #e5e7eb;
    border-radius: 8px;
    height: 6px;
    outline: none;
  }
  input[type='range']::-webkit-slider-thumb {
    appearance: none;
    background: #3b82f6;
    border-radius: 50%;
    cursor: pointer;
    height: 20px;
    width: 20px;
  }
  input[type='range']::-moz-range-thumb {
    background: #3b82f6;
    border: none;
    border-radius: 50%;
    cursor: pointer;
    height: 20px;
    width: 20px;
  }
  @media (max-width: 768px) {
    .demo-controls,
    .results-grid,
    .applications {
      grid-template-columns: 1fr;
    }
  }
</style>