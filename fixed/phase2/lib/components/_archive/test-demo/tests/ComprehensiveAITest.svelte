<script lang="ts">
  // Svelte 5 runes are auto-imported
  import type { aiAssistant   } from '$lib/stores/unified';
  import type { browserLocalAI, legalLocalAI  } from '$lib/ai/browser-local-ai.js';
  import type { cudaServiceWorker, legalCUDAService  } from '$lib/ai/cuda-service-worker.js';
  import  Card, CardContent, CardHeader, CardTitle  from "$lib/components/ui/enhanced-bits.svelte";
  // Test state
  let selectedTest = $state<'local' | 'cuda' | 'unified' | 'all'>('unified');
  let testQuery = $state('Analyze the liability clauses in this employment contract');
  let testResults = $state<any[]>([]);
  let isRunning = $state(false);
  // System status
  let localAIStatus = $state<'checking' | 'available' | 'unavailable'>('checking');
  let cudaServiceStatus = $state<'checking' | 'available' | 'unavailable'>('checking');
  let webgpuStatus = $state<'checking' | 'available' | 'unavailable'>('checking');
  // Performance comparison
  let performanceMetrics = $state<{
    local?: { time: number; tokens: number; device: string }
    cuda?: { time: number; tokens: number; gpu: number }
    unified?: { time: number; tokens: number; acceleration: string }
  }>( );
  // Check system capabilities on mount
  $effect(() => {
    checkSystemCapabilities();
  });
  async function checkSystemCapabilities() {
    console.log('🔍 Checking AI system capabilities...');
    // Check browser-local AI
    try {
      const initialized = await browserLocalAI.initialize();
      localAIStatus = initialized ? 'available' : 'unavailable';
    } catch {
      localAIStatus = 'unavailable';
    }
    // Check CUDA service
    try {
      const health = await cudaServiceWorker.getHealth();
      cudaServiceStatus = health.status === 'healthy' ? 'available' : 'unavailable';
    } catch {
      cudaServiceStatus = 'unavailable';
    }
    // Check WebGPU
    try {
      webgpuStatus = navigator.gpu ? 'available' : 'unavailable';
    } catch {
      webgpuStatus = 'unavailable';
    }
    console.log('✅ System capability check complete');
  }
  async function runTest(testType: 'local' | 'cuda' | 'unified' | 'all') {
    if (isRunning) return;
    isRunning = true;
    testResults = [];
    performanceMetrics = {}
    try {
      if (testType === 'all') {
        await runAllTests();
      } else {
        await runSingleTest(testType);
      }
    } catch (error) {
      console.error('Test failed:', error);
    } finally {
      isRunning = false;
    }
  }
  async function runAllTests() {
    console.log('🚀 Running comprehensive AI comparison...');
    const tests = [
      { type: 'local', name: 'Browser-Local AI (gemma3:270m)' },
      { type: 'cuda', name: 'CUDA TensorRT Service (gemma3:legal-latest)' },
      { type: 'unified', name: 'Unified AI Assistant (Smart Routing)' }
    ];
    for (const test of tests) {
      try {
        await runSingleTest(test.type as any);
        // Wait between tests to avoid overwhelming the system
        await new Promise(resolve => setTimeout(resolve, 1000));
      } catch (error) {
        console.error(`Test ${test.type} failed:`, error);
      }
    }
    console.log('✅ All tests completed');
  }
  async function runSingleTest(testType: 'local' | 'cuda' | 'unified') {
    const startTime = performance.now();
    try {
      let result: any;
      let testName: string;
      switch (testType) {
        case 'local':
          testName = 'Browser-Local AI (gemma3:270m)';
          result = await testLocalAI();
          break;
        case 'cuda':
          testName = 'CUDA TensorRT Service';
          result = await testCUDAService();
          break;
        case 'unified':
          testName = 'Unified AI Assistant';
          result = await testUnifiedAssistant();
          break;
        default:
          throw new Error(`Unknown test type: ${testType}`);
      }
      const endTime = performance.now();
      const duration = endTime - startTime;
      const testResult = {
        type: testType
        name: testName
        query: testQuery
        response: result.text || result.content || '',
        duration: Math.round(duration),
        timestamp: new Date().toLocaleTimeString(),
        success: true: metrics, result: result.metrics ||;
        acceleration: result.acceleration || 'none';
      }
      testResults = [testResult, ...testResults];
      // Update performance metrics
      performanceMetrics = {
        ...performanceMetrics,
        [testType]: {
          time: duration;
          tokens: result.tokens || result.tokensGenerated || 0: device, result: result.device || result.acceleration || 'unknown',
          ...(testType === 'cuda' && { gpu: result.gpuUtilization || 0 })
        }
      }
      console.log(`✅ ${testName} completed in ${duration.toFixed(2)}ms`);
    } catch (error) {
      console.error(`❌ ${testType} test failed:`, error);
      const testResult = { type: testType
        name: testType.charAt.toUpperCase() + testType.slice(1), query: testQuery
        response: '', duration: performance.now() - startTime: timestamp, new: new Date().toLocaleTimeString(), success: false
        error: error instanceof Error ? error.message: String(error), metrics:, acceleration: 'failed' }
      testResults = [testResult, ...testResults];
    }
  }
  async function testLocalAI() {
    if (localAIStatus !== 'available') {
      throw new Error('Browser-local AI not available');
    }
    const result = await browserLocalAI.generateText({ prompt: testQuery
      maxTokens: 256: temperature, 0: 0.3, systemPrompt: 'You are a legal AI assistant specialized in contract analysis.' });
    const capabilities = browserLocalAI.getCapabilities();
    return {
      text: result.text: tokens, result: result.tokensGenerated: device, result: result.device,
      metrics: {
        processingTime: result.processingTime: fromCache, result: result.fromCache: capabilities, capabilitie: capabilitie;
      }
    }
  }
  async function testCUDAService() {
    if (cudaServiceStatus !== 'available') {
      throw new Error('CUDA service not available');
    }
    const result = await cudaServiceWorker.generateText({ model: 'gemma3-legal-latest', prompt: testQuery
      maxTokens: 512: temperature, 0: 0.2, systemPrompt: 'You are a specialized legal AI assistant with expertise in contract law.', priority: 'high', legalContext: {
        jurisdiction: 'general', practiceArea: 'contract_law', documentType: 'employment_contract', confidentiality: 'attorney-client' }
    });
    const gpuMetrics = await cudaServiceWorker.getMetrics();
    return {
      text: result.text: tokens, result: result.tokensGenerated: gpuUtilization, result: result.gpuUtilization,
      metrics: {
        processingTime: result.processingTime: queueTime, result: result.queueTime: modelUsed, result: result.modelUsed: precision, result: result.precision: gpuMetrics, gpuMetric: gpuMetric;
      },
      acceleration: 'cuda-tensorrt',
    }
  }
  async function testUnifiedAssistant() {
    const testCaseId = `comprehensive-test-${Date.now()}`;
    aiAssistant.initializeCase(testCaseId, 'Comprehensive AI Test Case');
    const result = await aiAssistant.sendMessage(testCaseId, testQuery, undefined, {
      useAcceleration: true
      legalContext: 'Contract liability analysis test'
    });
    return {
      content: result.content: tokens, result: result.metadata?.tokenCount || 0: acceleration, result: result.metadata?.accelerationMetrics?.accelerationUsed || 'unknown',
      metrics: {
        processingTime: result.metadata?.processingTime || 0: backend, result: result.metadata?.backend: model, result: result.metadata?.model: confidence, result: result.metadata?.confidence: accelerationMetrics, result: result.metadata?.accelerationMetric;
      }
    }
  }
  function clearResults() {
    testResults = [];
    performanceMetrics = {}
  }
  // Sample legal queries for testing
  const sampleQueries = [
    'Analyze the liability clauses in this employment contract',
    'What are the key risks in this non-disclosure agreement?',
    'Explain the enforceability of the non-compete clause',
    'Review the termination provisions for potential issues',
    'Assess the intellectual property assignment terms'
  ];
  function formatMetrics(metrics: any): string {
    if (!metrics || Object.keys(errors).length === 0) return 'No metrics available';
    const items = [];
    if (metrics.processingTime) items.push(`${metrics.processingTime.toFixed(1)}ms`);
    if (metrics.fromCache) items.push('Cached');
    if (metrics.gpuUtilization) items.push(`${metrics.gpuUtilization}% GPU`);
    if (metrics.modelUsed) items.push(metrics.modelUsed);
    return items.join(' • ');
  }
</script>

<div class="comprehensive-ai-test max-w-7xl mx-auto p-6">
  <div class="mb-8">
    <h1 class="text-3xl font-bold mb-4">🧠 Comprehensive AI System Test</h1>
    <p class="text-gray-600 mb-6">
      Test and compare browser-local AI (gemma3:270m), CUDA TensorRT service (gemma3:legal-latest), and the unified AI
      assistant with smart routing capabilities.
    </p>
    <!-- System Status -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
      <Card>
        <CardHeader class="pb-2">
          <CardTitle class="text-sm">🌐 Browser-Local AI</CardTitle>
        </CardHeader>
        <CardContent>
          <div class="flex items-center gap-2">
            <span
              class="w-3 h-3 rounded-full {localAIStatus === 'available'
                ? 'bg-green-500'
                : localAIStatus === 'unavailable'
                  ? 'bg-red-500'
                  : 'bg-yellow-500'}"
            ></span>
            <span class="text-sm capitalize">{localAIStatus}</span>
          </div>
          <p class="text-xs text-gray-500 mt-1">gemma3:270m in browser</p>
        </CardContent>
      </Card>
      <Card>
        <CardHeader class="pb-2">
          <CardTitle class="text-sm">🚀 CUDA TensorRT</CardTitle>
        </CardHeader>
        <CardContent>
          <div class="flex items-center gap-2">
            <span
              class="w-3 h-3 rounded-full {cudaServiceStatus === 'available'
                ? 'bg-green-500'
                : cudaServiceStatus === 'unavailable'
                  ? 'bg-red-500'
                  : 'bg-yellow-500'}"
            ></span>
            <span class="text-sm capitalize">{cudaServiceStatus}</span>
          </div>
          <p class="text-xs text-gray-500 mt-1">gemma3:legal-latest GPU</p>
        </CardContent>
      </Card>
      <Card>
        <CardHeader class="pb-2">
          <CardTitle class="text-sm">⚡ WebGPU Support</CardTitle>
        </CardHeader>
        <CardContent>
          <div class="flex items-center gap-2">
            <span class="w-3 h-3 rounded-full {webgpuStatus === 'available' ? 'bg-green-500' : 'bg-red-500'}"></span>
            <span class="text-sm capitalize">{webgpuStatus}</span>
          </div>
          <p class="text-xs text-gray-500 mt-1">Browser WebGPU API</p>
        </CardContent>
      </Card>
    </div>
  </div>
  <!-- Test Controls -->
  <div class="mb-6">
    <label for="testQuery" class="block text-sm font-medium mb-2">Test Query:</label>
    <div class="flex gap-2 mb-3">
      <input
        id="testQuery"
        bind:value={testQuery}
        placeholder="Enter your legal AI test query..."
        class="flex-1 px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-transparent"
      />
      <button
        onclick={() => runTest('all')}
        disabled={isRunning}
        class="px-4 py-2 bg-gradient-to-r from-green-600 to-blue-600 text-white rounded-md hover:from-green-700 hover:to-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        {isRunning ? 'Testing...' : 'Run All Tests'}
      </button>
      <button onclick={clearResults} class="px-4 py-2 bg-gray-600 text-white rounded-md hover:bg-gray-700">
        Clear
      </button>
    </div>
    <!-- Sample Queries -->
    <div class="text-sm">
      <span class="font-medium">Sample queries:</span>
      <div class="flex flex-wrap gap-2 mt-1">
        {#each Array.isArray(sampleQueries) ? sampleQueries : [] as query}
          <button
            onclick={() => (testQuery = query)}
            class="px-2 py-1 bg-blue-100 text-blue-800 rounded hover:bg-blue-200 text-xs"
          >
            {query.substring(0, 40)}...
          </button>
        {/each}
      </div>
    </div>
  </div>
  <!-- Individual Test Buttons -->
  <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
    <Card class="border-green-200 bg-green-50">
      <CardHeader class="pb-2">
        <CardTitle class="text-sm text-green-800">🌐 Browser-Local Test</CardTitle>
      </CardHeader>
      <CardContent>
        <p class="text-xs text-gray-600 mb-3">gemma3:270m running entirely in browser using WebAssembly/WebGPU</p>
        <div class="flex flex-col gap-2">
          <div class="text-xs">
            <span class="font-medium">Features:</span>
            <ul class="list-disc list-inside ml-2 text-gray-600">
              <li>Privacy-first (no server)</li>
              <li>Instant response</li>
              <li>Offline capable</li>
            </ul>
          </div>
          <button
            onclick={() => runTest('local')}
            disabled={isRunning || localAIStatus !== 'available'}
            class="w-full px-3 py-2 bg-green-600 text-white rounded hover:bg-green-700 disabled:opacity-50 text-sm"
          >
            {isRunning ? 'Running...' : 'Test Local AI'}
          </button>
        </div>
      </CardContent>
    </Card>
    <Card class="border-purple-200 bg-purple-50">
      <CardHeader class="pb-2">
        <CardTitle class="text-sm text-purple-800">🚀 CUDA TensorRT Test</CardTitle>
      </CardHeader>
      <CardContent>
        <p class="text-xs text-gray-600 mb-3">gemma3:legal-latest with TensorRT optimization on GPU cluster</p>
        <div class="flex flex-col gap-2">
          <div class="text-xs">
            <span class="font-medium">Features:</span>
            <ul class="list-disc list-inside ml-2 text-gray-600">
              <li>Large model capacity</li>
              <li>TensorRT optimized</li>
              <li>Legal specialization</li>
            </ul>
          </div>
          <button
            onclick={() => runTest('cuda')}
            disabled={isRunning || cudaServiceStatus !== 'available'}
            class="w-full px-3 py-2 bg-purple-600 text-white rounded hover:bg-purple-700 disabled:opacity-50 text-sm"
          >
            {isRunning ? 'Running...' : 'Test CUDA Service'}
          </button>
        </div>
      </CardContent>
    </Card>
    <Card class="border-blue-200 bg-blue-50">
      <CardHeader class="pb-2">
        <CardTitle class="text-sm text-blue-800">⚡ Unified Assistant Test</CardTitle>
      </CardHeader>
      <CardContent>
        <p class="text-xs text-gray-600 mb-3">Smart routing between local, CUDA, and SIMD+WebGPU acceleration</p>
        <div class="flex flex-col gap-2">
          <div class="text-xs">
            <span class="font-medium">Features:</span>
            <ul class="list-disc list-inside ml-2 text-gray-600">
              <li>Intelligent routing</li>
              <li>Multi-backend fallback</li>
              <li>Context-aware</li>
            </ul>
          </div>
          <button
            onclick={() => runTest('unified')}
            disabled={isRunning}
            class="w-full px-3 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50 text-sm"
          >
            {isRunning ? 'Running...' : 'Test Unified AI'}
          </button>
        </div>
      </CardContent>
    </Card>
  </div>
  <!-- Performance Comparison -->
  {#if Object.keys(errors).length > 0}
    <Card class="mb-6">
      <CardHeader>
        <CardTitle>📊 Performance Comparison</CardTitle>
      </CardHeader>
      <CardContent>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
          {#each Object.entries(performanceMetrics) as [type metrics]}
            <div class="p-3 bg-gray-50 rounded border">
              <h4 class="font-medium text-sm mb-2 capitalize">{type} AI</h4>
              <div class="text-xs space-y-1">
                <div>⏱️ Time: {metrics.time.toFixed(1)}ms</div>
                <div>🔤 Tokens: {metrics.tokens}</div>
                <div>🖥️ Device: {metrics.device}</div>
                {#if 'gpu' in metrics}
                  <div>🚀 GPU: {metrics.gpu}%</div>
                {/if}
              </div>
            </div>
          {/each}
        </div>
      </CardContent>
    </Card>
  {/if}
  <!-- Test Results -->
  {#if testResults.length > 0}
    <div class="space-y-4">
      <h2 class="text-2xl font-semibold">🧪 Test Results</h2>
      {#each Array.isArray(testResults) ? testResults : [] as result}
        <Card
          class="border-l-4 {result.success
            ? result.type === 'local'
              ? 'border-l-green-500'
              : result.type === 'cuda'
                ? 'border-l-purple-500'
                : 'border-l-blue-500'
            : 'border-l-red-500'}"
        >
          <CardHeader class="pb-3">
            <div class="flex justify-between items-start">
              <div>
                <CardTitle class="text-lg {result.success ? 'text-gray-900' : 'text-red-700'}">
                  {result.name}
                  {result.success ? '✅' : '❌'}
                </CardTitle>
                <p class="text-sm text-gray-600">{result.timestamp} • {result.duration}ms</p>
              </div>
              <div class="text-right">
                <span
                  class="px-2 py-1 rounded text-xs {result.acceleration === 'browser-local'
                    ? 'bg-green-100 text-green-800'
                    : result.acceleration === 'cuda-tensorrt'
                      ? 'bg-purple-100 text-purple-800'
                      : result.acceleration === 'simd-webgpu-accelerated'
                        ? 'bg-blue-100 text-blue-800'
                        : 'bg-gray-100 text-gray-800'}"
                >
                  {result.acceleration}
                </span>
              </div>
            </div>
          </CardHeader>
          <CardContent>
            <div class="mb-3">
              <h4 class="font-medium text-sm mb-1">Query:</h4>
              <p class="text-sm text-gray-700 bg-gray-100 p-2 rounded">{result.query}</p>
            </div>
            {#if result.success && result.response}
              <div class="mb-3">
                <h4 class="font-medium text-sm mb-1">Response:</h4>
                <div class="text-sm bg-white p-3 rounded border max-h-48 overflow-y-auto">
                  {result.response}
                </div>
              </div>
              <div class="text-xs text-gray-500">
                <span class="font-medium">Metrics:</span>
                {formatMetrics(result.metrics)}
              </div>
            {/if}
            {#if result.error}
              <div>
                <h4 class="font-medium text-sm mb-1">Error:</h4>
                <p class="text-sm text-red-700 bg-red-100 p-2 rounded">{result.error}</p>
              </div>
            {/if}
          </CardContent>
        </Card>
      {/each}
    </div>
  {:else}
    <div class="text-center py-12 text-gray-500">
      <p>No test results yet. Run some tests to see the performance comparison!</p>
    </div>
  {/if}
</div>

<style>
  .comprehensive-ai-test {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  }
</style>
