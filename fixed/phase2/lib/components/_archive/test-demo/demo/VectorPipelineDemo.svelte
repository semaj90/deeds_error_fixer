<!-- Vector Pipeline Demo Component -->
<script lang="ts">
  // Svelte 5 runes are auto-imported
  import type { vectorPipelineState, vectorPipelineActions, type VectorPipelineJob  } from '$lib/machines/vector-pipeline-machine';
  import Button from '$lib/components/ui/enhanced-bits.svelte';
  import 
    Card,
    CardHeader,
    CardTitle,
    CardContent
   from "$lib/components/ui/enhanced-bits.svelte";
  // Using Svelte 4 store pattern instead of conflicting $state runes
  let machineState = $vectorPipelineStat;
  // Sample job data
  const sampleJobs: Array<Omit<VectorPipelineJob 'jobId' | 'status' | 'progress' | 'createdAt'> = [
    { ownerType: 'evidence', ownerId: 'evidence-001', event: 'upsert' },
    { ownerType: 'document', ownerId: 'doc-legal-brief-2024', event: 'reembed' },
    { ownerType: 'case', ownerId: 'case-murder-investigation', event: 'upsert' },
    { ownerType: 'report', ownerId: 'forensic-report-dna-analysis', event: 'delete' }
  ];
  function submitSingleJob() {
    const randomJob = sampleJobs[Math.floor(Math.random() * sampleJobs.length)];
    vectorPipelineActions.submitJob(randomJob);
  }
  function submitBatchJobs() {
    vectorPipelineActions.submitBatch(sampleJobs);
  }
  function runHealthCheck() {
    vectorPipelineActions.healthCheck();
  }
  function enableWebGPU() {
    vectorPipelineActions.enableWebGPU();
  }
  function disableWebGPU() {
    vectorPipelineActions.disableWebGPU();
  }
  function resetPipeline() {
    vectorPipelineActions.reset();
  }
  function retryFailedJobs() {
    vectorPipelineActions.retryFailedJobs();
  }
  // Get status indicators using derived values
  let pipelineStatus = $derived(machineState.context?.pipeline || null);
  let batchInfo = $derived(machineState.context?.batch || null);
  let metrics = $derived(machineState.context?.metrics || null);
  let currentState = $derived(typeof machineState.value === 'string' ? machineState.value : 'unknown');
  let errors = $derived(machineState.context?.errors || []);
  function getStatusColor(status: boolean): string {
    return status ? 'text-green-600' : 'text-red-600';
  }
  function getStatusIcon(status: boolean): string {
    return status ? '✅' : '❌';
  }
</script>

<div class="w-full max-w-6xl mx-auto p-4 space-y-6">
  <div class="nes-container">
    <div class="yorha-panel-header">
      <h3 class="nes-text is-primary text-2xl font-bold">Vector Pipeline Demo</h3>
      <p class="text-gray-600">
        XState machine orchestrating PostgreSQL → Redis Streams → Go microservice → CUDA worker →
        Qdrant
      </p>
    </div>
    <div class="yorha-panel-content">
      <!-- Current State Display -->
      <div class="mb-6">
        <div class="flex items-center gap-2 mb-2">
          <span class="font-semibold">Current State:</span>
          <span class="px-3 py-1 rounded-full text-sm font-medium bg-blue-100 text-blue-800">
            {currentState}
          </span>
        </div>
        {#if errors.length > 0}
          <div class="text-red-600 text-sm">
            <strong>Errors:</strong>
            {errors.join(', ')}
          </div>
        {/if}
      </div>
      <!-- Control Buttons -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-3 mb-6">
        <Button
          class="bits-btn"
          onclick={submitSingleJob}
          disabled={currentState === 'processingJob'}
        >
          Submit Single Job
        </Button>
        <Button
          class="bits-btn"
          onclick={submitBatchJobs}
          disabled={currentState === 'processingBatch'}
        >
          Submit Batch
        </Button>
        <Button class="bits-btn" onclick={runHealthCheck} variant="ghost">Health Check</Button>
        <Button class="bits-btn" onclick={resetPipeline} variant="error">Reset Pipeline</Button>
        <Button class="bits-btn" onclick={enableWebGPU} disabled={pipelineStatus?.webgpu}
          >Enable WebGPU</Button
        >
        <Button class="bits-btn" onclick={disableWebGPU} disabled={!pipelineStatus?.webgpu}
          >Disable WebGPU</Button
        >
        <Button
          class="bits-btn"
          onclick={retryFailedJobs}
          disabled={!batchInfo?.failedJobs || batchInfo.failedJobs === 0}
        >
          Retry Failed
        </Button>
      </div>
      <!-- Pipeline Status Grid -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <!-- Service Status -->
        <div class="nes-container">
          <div class="yorha-panel-header">
            <h3 class="nes-text is-primary text-lg">Service Status</h3>
          </div>
          <div class="yorha-panel-content">
            <div class="space-y-2">
              <div class="flex justify-between">
                <span>PostgreSQL:</span>
                <span class={getStatusColor(pipelineStatus?.postgresql || false)}>
                  {getStatusIcon(pipelineStatus?.postgresql || false)}
                </span>
              </div>
              <div class="flex justify-between">
                <span>Redis:</span>
                <span class={getStatusColor(pipelineStatus?.redis || false)}>
                  {getStatusIcon(pipelineStatus?.redis || false)}
                </span>
              </div>
              <div class="flex justify-between">
                <span>Go Microservice:</span>
                <span class={getStatusColor(pipelineStatus?.goMicroservice || false)}>
                  {getStatusIcon(pipelineStatus?.goMicroservice || false)}
                </span>
              </div>
              <div class="flex justify-between">
                <span>CUDA Worker:</span>
                <span class={getStatusColor(pipelineStatus?.cudaWorker || false)}>
                  {getStatusIcon(pipelineStatus?.cudaWorker || false)}
                </span>
              </div>
              <div class="flex justify-between">
                <span>Qdrant:</span>
                <span class={getStatusColor(pipelineStatus?.qdrant || false)}>
                  {getStatusIcon(pipelineStatus?.qdrant || false)}
                </span>
              </div>
              <div class="flex justify-between">
                <span>WebGPU:</span>
                <span class={getStatusColor(pipelineStatus?.webgpu || false)}>
                  {getStatusIcon(pipelineStatus?.webgpu || false)}
                </span>
              </div>
            </div>
          </div>
        </div>
        <!-- Batch Information -->
        <div class="nes-container">
          <div class="yorha-panel-header">
            <h3 class="nes-text is-primary text-lg">Batch Status</h3>
          </div>
          <div class="yorha-panel-content">
            <div class="space-y-2">
              <div class="flex justify-between">
                <span>Total Jobs:</span>
                <span class="font-mono">{batchInfo?.totalJobs || 0}</span>
              </div>
              <div class="flex justify-between">
                <span>Completed:</span>
                <span class="font-mono text-green-600">{batchInfo?.completedJobs || 0}</span>
              </div>
              <div class="flex justify-between">
                <span>Failed:</span>
                <span class="font-mono text-red-600">{batchInfo?.failedJobs || 0}</span>
              </div>
              <div class="flex justify-between">
                <span>Progress:</span>
                <span class="font-mono">{batchInfo?.progress || 0}%</span>
              </div>
              {#if batchInfo && batchInfo.progress > 0}
                <div class="w-full bg-gray-200 rounded-full h-2">
                  <div
                    class="bg-blue-600 h-2 rounded-full transition-all duration-300"
                    style="width: {batchInfo.progress}%"
                  ></div>
                </div>
              {/if}
            </div>
          </div>
        </div>
        <!-- Performance Metrics -->
        <div class="nes-container">
          <div class="yorha-panel-header">
            <h3 class="nes-text is-primary text-lg">Performance</h3>
          </div>
          <div class="yorha-panel-content">
            <div class="space-y-2">
              <div class="flex justify-between">
                <span>Processed:</span>
                <span class="font-mono">{metrics?.totalJobsProcessed || 0}</span>
              </div>
              <div class="flex justify-between">
                <span>Avg Time:</span>
                <span class="font-mono">{metrics?.averageProcessingTime || 0}ms</span>
              </div>
              <div class="flex justify-between">
                <span>Throughput:</span>
                <span class="font-mono">{metrics?.throughputPerMinute || 0}/min</span>
              </div>
              <div class="flex justify-between">
                <span>Last Run:</span>
                <span class="text-sm">
                  {metrics?.lastProcessedAt
                    ? new Date(metrics.lastProcessedAt).toLocaleTimeString()
                    : 'Never'}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- Current Jobs Display -->
      {#if batchInfo && batchInfo.jobs.length > 0}
        <div class="mt-6 nes-container">
          <div class="yorha-panel-header">
            <h3 class="nes-text is-primary text-lg">Active Jobs</h3>
          </div>
          <div class="yorha-panel-content">
            <div class="overflow-x-auto">
              <table class="w-full text-sm">
                <thead>
                  <tr class="border-b">
                    <th class="text-left p-2">Job ID</th>
                    <th class="text-left p-2">Type</th>
                    <th class="text-left p-2">Owner ID</th>
                    <th class="text-left p-2">Event</th>
                    <th class="text-left p-2">Status</th>
                    <th class="text-left p-2">Progress</th>
                  </tr>
                </thead>
                <tbody>
                  {#each Array.isArray(batchInfo.jobs) ? batchInfo.jobs : [] as job}
                    <tr class="border-b hover:bg-gray-50">
                      <td class="p-2 font-mono text-xs">{job.jobId}</td>
                      <td class="p-2">{job.ownerType}</td>
                      <td class="p-2 font-mono text-xs">{job.ownerId}</td>
                      <td class="p-2">{job.event}</td>
                      <td class="p-2">
                        <span
                          class="px-2 py-1 rounded text-xs font-medium
                          {job.status === 'succeeded'
                            ? 'bg-green-100 text-green-800'
                            : job.status === 'failed'
                              ? 'bg-red-100 text-red-800'
                              : job.status === 'processing'
                                ? 'bg-blue-100 text-blue-800'
                                : 'bg-gray-100 text-gray-800'}"
                        >
                          {job.status}
                        </span>
                      </td>
                      <td class="p-2">{job.progress}%</td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      {/if}
      <!-- Integration Test Results -->
      <div class="mt-6 nes-container">
        <div class="yorha-panel-header">
          <h3 class="nes-text is-primary text-lg">Service Integration Test</h3>
        </div>
        <div class="yorha-panel-content">
          <p class="text-sm text-gray-600 mb-4">
            Test connectivity to backend services that the vector pipeline depends on:
          </p>
          <Button
            class="bits-btn mr-2"
            onclick={async () => {
              try {
                // removed unused response assignment
                const data = await response.json();
                alert(`Enhanced RAG Service: ${data.status} (${data.service})`);
              } catch (error) {
                alert(`Enhanced RAG Service: Error - ${error}`);
              }
            }}
            variant="ghost"
          >
            Test Enhanced RAG (8094)
          </Button>
          <Button
            class="bits-btn"
            onclick={async () => {
              try {
                // removed unused response assignment
                if (response.ok) {
                  alert('Qdrant Service: Healthy');
                } else {
                  alert(`Qdrant Service: Status ${response.status}`);
                }
              } catch (error) {
                alert(`Qdrant Service: Error - ${error}`);
              }
            }}
            variant="ghost"
          >
            Test Qdrant (6333)
          </Button>
        </div>
      </div>
    </div>
  </div>
</div>
