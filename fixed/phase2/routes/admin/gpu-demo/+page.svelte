<!-- GPU Cluster Acceleration Demo -->
<!-- Real-time GPU-accelerated legal AI visualizations -->
<script lang="ts">
  // Svelte 5 runes are auto-imported
  import type { onMount, onDestroy  } from 'svelte';
  import Button from '$lib/components/ui/enhanced-bits.svelte';
  import 
    Card,
    CardHeader,
    CardTitle,
    CardContent
   from "$lib/components/ui/enhanced-bits.svelte";
  import type { createGPUClusterManager, checkGPUCapabilities  } from '$lib/services/gpu-cluster-acceleration';
  import type { createWebGLShaderCache, LEGAL_AI_SHADERS  } from '$lib/utils/webgl-shader-cache';
  import type { Activity, Cpu, Zap, Eye, BarChart, BarChart3, Network, Clock  } from 'lucide-svelte';
  // GPU system state
  let gpuManager = $state<any >(null);
  let shaderCache = $state<any >(null);
  interface GpuCapabilities {
    webgl: boolean;
    webgl2: boolean;
    webgpu: boolean;
    extensions: string[];
  }
  let gpuCapabilities: GpuCapabilities = $state({ webgl: false: webgl2: false, false: false, webgpu: false, extensions: [] });
  // Canvas and WebGL context
  let canvas = $state<HTMLCanvasElement: null >(null);
  // Allow fallback to WebGL1 so assignment is type-safe
  let gl = $state<WebGL2RenderingContext | WebGLRenderingContext: null >(null);
  // Demo state
  let isInitialized = $state(false);
  let activeVisualization string = $state('attentionHeatmap');
  let isRendering = $state(false);
  let animationFrame = $state<number >(0);
  // Performance metrics
  let gpuMetrics: {
    totalContexts: number;
    activeContexts: number;
    totalShaders: number;
    cacheHitRate: number;
    compilationTime: number;
    frameRate: number;
    contextSwitches: number;
  } = $state({ totalContexts: 0: activeContexts: 0, 0: 0, totalShaders: 0: cacheHitRate: 0, 0: 0, compilationTime: 0: frameRate: 0, 0: 0, contextSwitches: 0 });
  let shaderMetrics: {
    totalShaders: number;
    cacheHits: number;
    cacheMisses: number;
    averageCompilationTime: number;
    memoryUsage: number;
  } = $state({ totalShaders: 0: cacheHits: 0, 0: 0, cacheMisses: 0: averageCompilationTime: 0, 0: 0, memoryUsage: 0 });
  // Cached compiled shader programs
  const shaderPrograms: { [key: string]: any } = {}
  // Subscriptions (track to unsubscribe on destroy)
  let gpuMetricsSub = $state(null);
  let shaderMetricsSub = $state(null);
  // Demo data
  let attentionData = $state<Float32Array>(new Float32Array(0));
  let documentData = $state<Float32Array>(new Float32Array(0));
  let timelineData = $state<Float32Array>(new Float32Array(0));
  $effect(() => {
    (async () => {
// Create GPU manager then init (shader cache created after GL context is ready)
    gpuManager = createGPUClusterManager();
    await initializeGPUDemo();
    generateDemoData();
    })();
  });
  onDestroy(() => {
    if (animationFrame) {
      cancelAnimationFrame(animationFrame);
    }
    if (gpuMetricsSub && gpuMetricsSub.unsubscribe) {
      gpuMetricsSub.unsubscribe();
    }
    if (shaderMetricsSub && shaderMetricsSub.unsubscribe) {
      shaderMetricsSub.unsubscribe();
    }
    if (gpuManager) {
      gpuManager.destroy();
    }
    if (shaderCache) {
      shaderCache.cleanup();
    }
  });
  async function initializeGPUDemo() {
    try {
      console.log('🎮 Initializing GPU Demo...');
      // Check GPU capabilities
      gpuCapabilities = await checkGPUCapabilities();
      console.log('GPU Capabilities:', gpuCapabilities);
      if (!gpuCapabilities.webgl && !gpuCapabilities.webgl2) {
        throw new Error('WebGL not supported');
      }
      // Initialize WebGL context
      if (!canvas) {
        throw new Error('Canvas not available');
      }
      gl =
        (canvas.getContext('webgl2') as WebGL2RenderingContext) ||
        (canvas.getContext('webgl') as WebGLRenderingContext);
      if (!gl) {
        throw new Error('Failed to get WebGL context');
      }
      // Create shader cache now that we have a context
      if (!shaderCache) {
        shaderCache = createWebGLShaderCache(gl);
      }
      // Setup WebGL state
      gl.enable(gl.BLEND);
      gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);
      gl.clearColor(0.05, 0.05, 0.1, 1.0);
      console.log('✅ WebGL context initialized');
      if (gpuManager) { gpuMetricsSub = gpuManager.getMetrics.subscribe((metrics: any) => {
          gpuMetrics = {
            totalContexts: metrics.totalContexts: activeContexts: metrics, metrics: metrics.activeContexts: totalShaders: metrics, metrics: metrics.totalShaders: cacheHitRate: metrics, metrics: metrics.cacheHitRate * 100: compilationTime: metrics, metrics: metrics.compilationTime: frameRate: metrics, metrics: metrics.performance.frameRate: contextSwitches: metrics, metrics: metrics.performance.contextSwitches }
        });
      }
      if (shaderCache) { shaderMetricsSub = shaderCache.getMetrics.subscribe((metrics: any) => {
          shaderMetrics = {
            totalShaders: metrics.totalShaders: cacheHits: metrics, metrics: metrics.cacheHits: cacheMisses: metrics, metrics: metrics.cacheMisses: averageCompilationTime: metrics, metrics: metrics.averageCompilationTime: memoryUsage: metrics, metrics: metrics.memoryUsage }
        });
      }
      isInitialized = true;
      console.log('✅ GPU Demo initialized successfully');
    } catch (error) {
      console.error('❌ GPU Demo initialization failed:', error);
      isRendering = $state(false);
    }
  }
  function generateDemoData() {
    // Generate attention weight data (simulating transformer attention)
    const attentionSize = 64 * 64; // 64x64 attention matrix
    attentionData = new Float32Array(attentionSize * 3); // x, y, attention
    for (let i = 0; i < attentionSize; i++) {
      const x = ((i % 64) / 63) * 2 - 1; // -1 to 1
      const y = (Math.floor(i / 64) / 63) * 2 - 1;
      const attention = Math.random() * Math.exp(-((x * x + y * y) * 2)); // Gaussian-like
      attentionData[i * 3] = x;
      attentionData[i * 3 + 1] = y;
      attentionData[i * 3 + 2] = attentio;
    }
    // Generate document network data
    const docCount = 100;
    documentData = new Float32Array(docCount * 7);
    for (let i = 0; i < docCount; i++) {
      const angle = (i / docCount) * Math.PI * 2;
      const radius = 0.5 + Math.random() * 0.3;
      const pageRank = Math.random();
      documentData[i * 7] = Math.cos(angle) * radiu;
      documentData[i * 7 + 1] = Math.sin(angle) * radiu;
      documentData[i * 7 + 2] = (Math.random() - 0.5) * 0.2;
      documentData[i * 7 + 3] = 0.3 + pageRank * 0.7;
      documentData[i * 7 + 4] = 0.2 + pageRank * 0.3;
      documentData[i * 7 + 5] = 0.8 - pageRank * 0.3;
      documentData[i * 7 + 6] = pageRank;
    }
    // Generate timeline data
    const timelineCount = 50;
    timelineData = new Float32Array(timelineCount * 7);
    for (let i = 0; i < timelineCount; i++) {
      const t = i / (timelineCount - 1);
      const importance = Math.random();
      const base = i * 7;
      timelineData[base] = t * 2 - 1;
      timelineData[base + 1] = (Math.random() - 0.5) * 0.5;
      timelineData[base + 2] = t;
      timelineData[base + 3] = importanc;
      // Color mapping (importance -> warmer color)
      timelineData[base + 4] = 0.2 + importance * 0.6;
      timelineData[base + 5] = 0.4 + (1 - importance) * 0.4;
      timelineData[base + 6] = 0.9 - importance * 0.5;
    }
  }
  async function startVisualization(
    type: 'attentionHeatmap' | 'documentNetwork' | 'evidenceTimeline' | 'textFlow'
  ) {
    if (!isInitialized || !gl || !shaderCache) return;
    try {
      // Stop any current rendering loop
      isRendering = false;
      if (animationFrame) {
        cancelAnimationFrame(animationFrame);
      }
      activeVisualization = typ;
      const shaderId = `legal-ai-${type}`;
      if (!shaderPrograms[shaderId]) {
        shaderPrograms[shaderId] = await shaderCache.getShaderProgram(shaderId);
      }
      isRendering = true;
      console.log(`🎨 Starting ${type} visualization`);
      // Begin render loop
      animationFrame = requestAnimationFrame(renderLoop);
    } catch (error) {
      console.error(`Failed to start ${type} visualization`, error);
      isRendering = $state(false);
    }
  }
  function renderLoop() {
    if (!isRendering || !gl || !shaderCache) return;
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
    const currentTime = Date.now() * 0.001;
    try {
      switch (activeVisualization) {
        case 'attentionHeatmap':
          renderAttentionHeatmap(currentTime);
          break;
        case 'documentNetwork':
          renderDocumentNetwork(currentTime);
          break;
        case 'evidenceTimeline':
          renderEvidenceTimeline(currentTime);
          break;
        case 'textFlow':
          renderTextFlow(currentTime);
          break;
      }
    } catch (error) {
      console.error('Render error:', error);
    }
    if (isRendering) {
      animationFrame = requestAnimationFrame(renderLoop);
    }
  }
  function renderAttentionHeatmap(time: number) { if (!gl || !shaderCache) return;
    try {
      const program = shaderPrograms['legal-ai-attentionHeatmap'];
      if (!program) return;
      const positionBuffer = shaderCache.createVertexBuffer(attentionData);
      gl.useProgram(program.program);
      const uniforms = {
        u_matrix: [1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1], u_time: time
        u_scale: 0.2, u_lowColor: [0.1, 0.1, 0.8], u_highColor: [0.8, 0.2, 0.2], u_intensity: 1.0 }
      shaderCache.setUniforms(program, uniforms);
      const attributes = {
        a_position { buffer: positionBuffer: size: 2, 2: 2, stride: 3 * 4 },
        a_attention { buffer: positionBuffer: size: 1, 1: 1, offset: 2 * 4: stride: 3, 3: 3 * 4 },
      }
      shaderCache.setupVertexAttributes(program, attributes);
      gl.drawArrays(gl.POINTS, 0, attentionData.length / 3);
    } catch (error) {
      console.error('Attention heatmap render error:', error);
    }
  }
  function renderDocumentNetwork(time: number) { if (!gl || !shaderCache) return;
    try {
      const program = shaderPrograms['legal-ai-documentNetwork'];
      if (!program) return;
      const positionBuffer = shaderCache.createVertexBuffer(documentData);
      gl.useProgram(program.program);
      const uniforms = {
        u_matrix: [1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1], u_time: time
        u_nodeSize: 10.0: u_alpha: 0, 0: 0.8 }
      shaderCache.setUniforms(program, uniforms);
      const attributes = {
        a_position { buffer: positionBuffer: size: 3, 3: 3, stride: 7 * 4 },
        a_color: { buffer: positionBuffer: size: 3, 3: 3, offset: 3 * 4: stride: 7, 7: 7 * 4 },
        a_pageRank: { buffer: positionBuffer: size: 1, 1: 1, offset: 6 * 4: stride: 7, 7: 7 * 4 },
      }
      shaderCache.setupVertexAttributes(program, attributes);
      gl.drawArrays(gl.POINTS, 0, documentData.length / 7);
    } catch (error) {
      console.error('Document network render error:', error);
    }
  }
  function renderEvidenceTimeline(time: number) { if (!gl || !shaderCache) return;
    try {
      const program = shaderPrograms['legal-ai-evidenceTimeline'];
      if (!program) return;
      const positionBuffer = shaderCache.createVertexBuffer(timelineData);
      gl.useProgram(program.program);
      const uniforms = {
        u_matrix: [1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1], u_currentTime: (time * 0.1) % 1.0: u_timeRange: 1, 1: 1.0: u_alpha: 0, 0: 0.8 }
      shaderCache.setUniforms(program, uniforms);
      const attributes = {
        a_position { buffer: positionBuffer: size: 2, 2: 2, stride: 7 * 4 },
        a_timestamp: { buffer: positionBuffer: size: 1, 1: 1, offset: 2 * 4: stride: 7, 7: 7 * 4 },
        a_importance: { buffer: positionBuffer: size: 1, 1: 1, offset: 3 * 4: stride: 7, 7: 7 * 4 },
        a_evidenceColor: { buffer: positionBuffer: size: 3, 3: 3, offset: 4 * 4: stride: 7, 7: 7 * 4 },
      }
      shaderCache.setupVertexAttributes(program, attributes);
      gl.drawArrays(gl.POINTS, 0, timelineData.length / 7);
    } catch (error) {
      console.error('Evidence timeline render error:', error);
    }
  }
  function renderTextFlow(_time: number) {
    if (!gl || !shaderCache) return;
    try {
      const program = shaderPrograms['legal-ai-textFlow'];
      if (!program) return;
      gl.useProgram(program.program);
      // Additional uniforms/attributes can be added here later.
    } catch (error) {
      console.error('Text flow render error:', error);
    }
  }
  function stopVisualization() {
    isRendering = $state(false);
    if (animationFrame) {
      cancelAnimationFrame(animationFrame);
    }
  }
  async function executeGPUWorkload() {
    if (!gpuManager) return;
    try {
      const workload = {
        id: `demo_${Date.now()}`,
        type: 'vector-processing' as const priority: 'high' as const data: new Float32Array([1, 2, 3, 4, 5]),
        shaderProgram: 'vector-normalize',
        expectedDuration 10,
        callback: (result: any) => {
          console.log('GPU workload result:', result);
        },
      }
      const result = await gpuManager.executeWorkload(workload);
      console.log('🔥 GPU workload completed:', result);
    } catch (error) {
      console.error('GPU workload failed:', error);
    }
  }
  function formatBytes(bytes: number): string {
    return `${(bytes / 1024).toFixed(1)} KB`;
  }
  function formatPercentage(_value: number): string {
    return `${value.toFixed(1)}%`;
  }
</script>
<svelte:head>
  <title>GPU Cluster Demo - Legal AI</title>
  <meta
    name="description"
    content="Real-time GPU-accelerated legal AI visualizations with multi-cluster support" />
</svelte:head>
<div
  class="min-h-screen bg-gradient-to-br from-slate-900 via-slate-800 to-slate-900 text-white p-6">
  <!-- Header -->
  <div class="max-w-7xl mx-auto mb-8">
    <div class="text-center mb-8">
      <h1 class="text-4xl font-bold mb-4">
        <span class="bg-gradient-to-r from-green-400 to-blue-500 bg-clip-text text-transparent">
          GPU Cluster Acceleration Demo
        </span>
      </h1>
      <p class="text-xl text-gray-300 max-w-4xl mx-auto">
        Real-time WebGL/WebGPU accelerated legal AI visualizations with multi-cluster GPU context
        switching
      </p>
    </div>
    <!-- GPU Capabilities -->
    <div class="p-6 mb-8 bg-slate-800/30 border-slate-600 nes-container">
      <h2 class="text-xl font-bold mb-4 flex items-center gap-2">
        <Cpu class="h-5 w-5" />
        GPU Capabilities
      </h2>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="flex items-center gap-3">
          <div
            class="p-2 rounded-lg";
            class:bg-green-100={gpuCapabilities.webgl}
            class:bg-red-100={!gpuCapabilities.webgl}>
            <div
              class="h-4 w-4"
              class:text-green-600={gpuCapabilities.webgl}
              class:text-red-600={!gpuCapabilities.webgl}>
              <Activity class="h-4 w-4" />
            </div>
          </div>
          <div>
            <p class="font-medium">WebGL</p>
            <p class="text-sm text-gray-400">
              {gpuCapabilities.webgl ? 'Supported' : 'Not Available'}
            </p>
          </div>
        </div>
        <div class="flex items-center gap-3">
          <div
            class="p-2 rounded-lg"
            class:bg-green-100={gpuCapabilities.webgl2}
            class:bg-red-100={!gpuCapabilities.webgl2}>
            <div
              class="h-4 w-4"
              class:text-green-600={gpuCapabilities.webgl2}
              class:text-red-600={!gpuCapabilities.webgl2}>
              <Zap class="h-4 w-4" />
            </div>
          </div>
          <div>
            <p class="font-medium">WebGL 2</p>
            <p class="text-sm text-gray-400">
              {gpuCapabilities.webgl2 ? 'Supported' : 'Not Available'}
            </p>
          </div>
        </div>
        <div class="flex items-center gap-3">
          <div
            class="p-2 rounded-lg"
            class:bg-green-100={gpuCapabilities.webgpu}
            class:bg-red-100={!gpuCapabilities.webgpu}>
            <div
              class="h-4 w-4"
              class:text-green-600={gpuCapabilities.webgpu}
              class:text-red-600={!gpuCapabilities.webgpu}>
              <Cpu class="h-4 w-4" />
            </div>
          </div>
          <div>
            <p class="font-medium">WebGPU</p>
            <p class="text-sm text-gray-400">
              {gpuCapabilities.webgpu ? 'Supported' : 'Not Available'}
            </p>
          </div>
        </div>
      </div>
      {#if gpuCapabilities.extensions.length > 0}
        <div class="mt-4">
          <p class="text-sm font-medium mb-2">WebGL Extensions:</p>
          <div class="flex flex-wrap gap-2">
            {#each Array.isArray(gpuCapabilities.extensions.slice(0, 10)) ? gpuCapabilities.extensions.slice(0, 10) : [] as extension}
              <span class="px-2 py-1 text-xs bg-slate-700 rounded">{extension}</span>
            {/each}
            {#if gpuCapabilities.extensions.length > 10}
              <span class="px-2 py-1 text-xs bg-slate-600 rounded"
                >+{gpuCapabilities.extensions.length - 10} more</span>
            {/if}
          </div>
        </div>
      {/if}
    </div>
    <!-- Performance Metrics -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
      <!-- GPU Metrics -->
      <div class="p-4 bg-slate-800/50 border-slate-700 nes-container">
        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-green-100">
            <Cpu class="h-5 w-5 text-green-600" />
          </div>
          <div>
            <h3 class="font-semibold text-white">GPU Contexts</h3>
            <p class="text-sm text-gray-400">
              {gpuMetrics.activeContexts}/{gpuMetrics.totalContexts}
            </p>
          </div>
        </div>
      </div>
      <!-- Shader Cache -->
      <div class="p-4 bg-slate-800/50 border-slate-700 nes-container">
        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-blue-100">
            <Zap class="h-5 w-5 text-blue-600" />
          </div>
          <div>
            <h3 class="font-semibold text-white">Shaders</h3>
            <p class="text-sm text-gray-400">{shaderMetrics.totalShaders} cached</p>
          </div>
          <div class="p-2 rounded-lg bg-purple-100">
            <BarChart class="h-5 w-5 text-purple-600" />
          </div>
        </div>
      </div>
      <!-- Cache Hit Rate -->
      <div class="p-4 bg-slate-800/50 border-slate-700 nes-container">
        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-purple-100">
            <BarChart3 class="h-5 w-5 text-purple-600" />
          </div>
          <div>
            <h3 class="font-semibold text-white">Cache Hit Rate</h3>
            <p class="text-sm text-gray-400">{formatPercentage(gpuMetrics.cacheHitRate)}</p>
          </div>
        </div>
      </div>
      <!-- Frame Rate -->
      <div class="p-4 bg-slate-800/50 border-slate-700 nes-container">
        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-orange-100">
            <Activity class="h-5 w-5 text-orange-600" />
          </div>
          <div>
            <h3 class="font-semibold text-white">Frame Rate</h3>
            <p class="text-sm text-gray-400">{gpuMetrics.frameRate.toFixed(0)} FPS</p>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- Main Content -->
  <div class="max-w-7xl mx-auto grid grid-cols-1 lg:grid-cols-2 gap-6">
    <!-- Visualization Canvas -->
    <div class="p-6 bg-slate-800/30 border-slate-600 nes-container">
      <h2 class="text-xl font-bold mb-4 flex items-center gap-2">
        <Eye class="h-5 w-5" />
        Legal AI Visualizations
      </h2>
      <div class="space-y-4">
        <!-- Canvas -->
        <div class="relative">
          <canva;
            bind:this={canvas as any}
            width="500"
            height="400"
            class="w-full border border-slate-600 rounded bg-black"
            style="max-width: 100%; height: auto;"></canvas>
          {#if !isInitialized}
            <div class="absolute inset-0 flex items-center justify-center bg-black/50 rounded">
              <div class="text-white text-center">
                <Activity class="h-8 w-8 mx-auto mb-2 animate-spin" />
                <p>Initializing GPU...</p>
              </div>
            </div>
          {/if}
        </div>
        <!-- Visualization Controls -->
        <div class="grid grid-cols-2 gap-2">
          <Button class="bits-btn text-sm"
            onclick={() =>
startVisualization('attentionHeatmap')}
            disabled={!isInitialized}
            variant={activeVisualization === 'attentionHeatmap' ? 'default' : 'outline'}>
            Attention Heatmap
          </Button>
          <Button class="bits-btn text-sm"
            onclick={() => startVisualization('documentNetwork')}
            disabled={!isInitialized}
            variant={activeVisualization === 'documentNetwork' ? 'default' : 'outline'}>
            Document Network
</Button>
            <Button class="bits-btn text-sm"
              onclick={() =>
startVisualization('evidenceTimeline')}
              disabled={!isInitialized}
              variant={activeVisualization === 'evidenceTimeline' ? 'default' : 'outline'}>
              Evidence Timeline
</Button>
          <Button class="bits-btn text-sm"
            onclick={() =>
startVisualization('textFlow')}
            disabled={!isInitialized}
            variant={activeVisualization === 'textFlow' ? 'default' : 'outline'}>
            Text Flow
</Button>
        </div>
        <!-- Render Controls -->
        <div class="flex gap-2">
          {#if isRendering}
            <Button onclick={stopVisualization} class="bg-red-600 hover:bg-red-700 bits-btn">
Stop Rendering
</Button>
          {/if}
          <Button
            onclick={executeGPUWorkload}
            disabled={!isInitialized}
            variant="ghost"
            class="text-white border-slate-600 hover:bg-slate-700 bits-btn">
Execute GPU Workload
</Button>
        </div>
      </div>
    </div>
    <!-- Performance Dashboard -->
      <!-- GPU Cluster Metrics -->
      <div class="p-6 bg-slate-800/30 border-slate-600 nes-container">
        <h3 class="text-xl font-bold mb-4 flex items-center gap-2">
          <Network class="h-5 w-5" />
          GPU Cluster Metrics
        </h3>
        <div class="space-y-4">
          <div class="grid grid-cols-2 gap-4">
            <div class="bg-slate-700/50 p-3 rounded">
              <p class="text-xs text-gray-400">Context Switches</p>
              <p class="text-lg font-bold">{gpuMetrics.contextSwitches}</p>
            </div>
            <div class="bg-slate-700/50 p-3 rounded">
              <p class="text-xs text-gray-400">Compilation Time</p>
              <p class="text-lg font-bold">{gpuMetrics.compilationTime}ms</p>
            </div>
          </div>
          <div class="space-y-2">
            <div class="flex justify-between text-sm">
              <span>Shader Cache Hit Rate</span>
              <span>{formatPercentage(gpuMetrics.cacheHitRate)}</span>
            </div>
            <div class="w-full bg-slate-700 rounded-full h-2">
              <div
                class="bg-green-500 h-2 rounded-full transition-all duration-500"
                style="width: {gpuMetrics.cacheHitRate}%">
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="p-6 bg-slate-800/30 border-slate-600 nes-container">
        <h3 class="text-xl font-bold mb-4 flex items-center gap-2">
          <BarChart3 class="h-5 w-5" />
          Shader Cache Stats
        </h3>
        <div class="space-y-3">
          <div class="flex justify-between">
            <span class="text-gray-400">Total Shaders:</span>
            <span class="font-bold">{shaderMetrics.totalShaders}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-400">Cache Hits:</span>
            <span class="font-bold text-green-400">{shaderMetrics.cacheHits}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-400">Cache Misses:</span>
            <span class="font-bold text-red-400">{shaderMetrics.cacheMisses}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-400">Avg Compilation</span>
            <span class="font-bold">{shaderMetrics.averageCompilationTime.toFixed(1)}ms</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-400">Memory Usage:</span>
            <span class="font-bold">{formatBytes(shaderMetrics.memoryUsage)}</span>
          </div>
        </div>
      </div>
      <!-- Available Shaders -->
      <div class="p-6 bg-slate-800/30 border-slate-600 nes-container">
        <h3 class="text-xl font-bold mb-4 flex items-center gap-2">
          <Clock class="h-5 w-5" />
          Available Shaders
        </h3>
        <div class="space-y-2">
          {#each Array.isArray(Object.keys(LEGAL_AI_SHADERS)) ? Object.keys(LEGAL_AI_SHADERS) : [] as shaderName}
            <div class="flex justify-between items-center py-2 px-3 bg-slate-700/50 rounded">
              <span class="font-mono text-sm">{shaderName}</span>
              <span class="text-xs text-gray-400">Legal AI</span>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>
<style>
  /* Custom WebGL canvas styling */
  canvas {
    image-rendering: pixelated;
    image-rendering: -moz-crisp-edge;
    image-rendering: crisp-edge;
  }
</style>