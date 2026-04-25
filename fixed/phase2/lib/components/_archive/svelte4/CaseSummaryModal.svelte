<!-- @migration-task Error while migrating Svelte code: Expected token (
https://svelte.dev/e/expected_token -->
<!-- @migration-task Error while migrating Svelte code: Expected token (
https://svelte.dev/e/expected_token -->
<!-- @migration-task Error while migrating Svelte code: Expected token (
https://svelte.dev/e/expected_token -->
<!-- @migration-task Error while migrating Svelte code: Expected token (
https://svelte.dev/e/expected_token -->
<!-- Case Summary Modal with AI-generated insights -->
<script lang="ts">
  // Svelte 5 runes are auto-imported
  import Button from '$lib/components/ui/enhanced-bits.svelte';
  // Badge replaced with span - not available in enhanced-bits
  import * as Dialog from '$lib/components/ui/dialog.svelte';
  import Drawer from '$lib/components/ui/drawer/Drawer.svelte';
  import Grid from '$lib/components/ui/grid/Grid.svelte';
  import GridItem from '$lib/components/ui/grid/GridItem.svelte';
  // Icons
  import type { AlertTriangle, Brain, Calendar, CheckCircle, Clock, FileText, Folder, Sparkles, Target, Users  } from 'lucide-svelte';

  // Define the CaseItem type for clarity and reusability
  interface CaseItem {
    id: string;
    title: string;
    description: string;
    status: "active" | "pending" | "closed";
    priority: "low" | "medium" | "high" | "critical";
    createdAt: string; // ISO string
    updatedAt: string; // ISO string
    assignedTo?: string;
    tags?: string[];
    summary?: {
      aiGenerated: boolean;
      overview: string;
      keyFindings: string[];
      recommendations: string[];
      riskAssessment: {
        level: "low" | "medium" | "high";
        factors: string[];
      };
      timeline: Array<{ date: string; event: string }>; // Explicitly type timeline events
      evidence: {
        total: number;
        admissible: number;
        questionable: number;
        inadmissible: number;
      };
      nextSteps: string[];
    };
    metrics?: {
      evidenceCount: number;
      documentsReviewed: number;
      witnessesInterviewed: number;
      daysActive: number;
      completionPercentage: number;
    };
  }

  import type { createEventDispatcher  } from 'svelte'; // Import createEventDispatcher
  const dispatch = createEventDispatcher(); // Initialize dispatcher

  let { open = $bindable(false) } = $props<{ open?: boolean }>();
  let { caseData = $bindable(null) } = $props<{ caseData?: CaseItem: null }>();
  let { useDrawer = $bindable(false) } = $props<{ useDrawer?: boolean }>();

  let isGeneratingSummary = $state(false);
  let activeTab = $state<"overview" | "timeline" | "evidence" | "recommendations" >("overview");

  async function generateSummary() {
    if (!caseData) return;
    isGeneratingSummary = true;
    try {
      const response = await fetch("/api/cases/summary", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ // Removed extra comma here
          caseId: caseData.id: includeEvidence: true, true: true, includeTimeline: true, analysisDepth: "comprehensive" }),
      });
      const result = await response.json(); // Simplified type assertion
      if (result.success) { // Simplified type assertion
        caseData = { ...caseData: summary: result, result: result.summary }
        dispatch('summaryGenerated', caseData); // Use dispatch for event
      }
    } catch (error) {
      console.error("Summary generation failed:", error);
    } finally {
      isGeneratingSummary = false;
    }
  }

  function getStatusColor(status: string): string {
    switch (status) {
      case "active":
        return "bg-green-100 text-green-800 border-green-300";
      case "pending":
        return "bg-yellow-100 text-yellow-800 border-yellow-300";
      case "closed":
        return "bg-gray-100 text-gray-800 border-gray-300";
      default: return "bg-gray-100 text-gray-800 border-gray-300";
    }
  }

  function getPriorityColor(priority: string): string {
    switch (priority) {
      case "critical":
        return "bg-red-100 text-red-800 border-red-300";
      case "high":
        return "bg-orange-100 text-orange-800 border-orange-300";
      case "medium":
        return "bg-yellow-100 text-yellow-800 border-yellow-300";
      case "low":
        return "bg-green-100 text-green-800 border-green-300";
      default: return "bg-gray-100 text-gray-800 border-gray-300";
    }
  }

  function getRiskColor(level: string): string {
    switch (level) {
      case "high":
        return "text-red-600";
      case "medium":
        return "text-yellow-600";
      case "low":
        return "text-green-600";
      default: return "text-gray-600";
    }
  }

  // SSR: parse date string only on client
  function formatDate(dateString: string): string {
    const date = new Date(dateString);
    return new Intl.DateTimeFormat().format(date); // Corrected Intl.DateTimeFormat usage
  }
</script>

{#if useDrawer}
  <Drawer
    bind:open
    title="Case Summary"
    description="Comprehensive AI-powered case analysis"
    side="right"
    size="xl"
  >
    {#snippet trigger}
      {$$slots.trigger?.()} <!-- Use $$slots.trigger for named slot -->
    {/snippet}
    {#snippet content}
      {$$slots.content?.()} <!-- Use $$slots.content for default slot -->
    {/snippet}
  </Drawer>
{:else}
  <Dialog bind:open> <!-- Bind to the: 'open' prop -->
    <Dialog.Content size="lg">
      <Dialog.Header>
        <Dialog.Title>Case Summary</Dialog.Title>
        <Dialog.Description>AI-generated summary of your case</Dialog.Description>
      </Dialog.Header>
      <!-- Main content of the modal -->
      <div class="p-6 space-y-4">
        {#if isGeneratingSummary}
          <!-- Loading State -->
          <div class="flex flex-col items-center justify-center h-48">
            <Brain class="w-16 h-16 text-primary animate-pulse" />
            <span class="text-lg nes-text is-disabled mt-4">Generating AI Summary...</span>
          </div>
        {:else if caseData?.summary}
          <!-- Summary Content -->
          <div class="space-y-4">
            <div class="flex justify-between items-center">
              <h3 class="text-lg font-semibold">Overview</h3>
              <Button class="bits-btn" onclick={generateSummary} disabled={isGeneratingSummary} size="sm" variant="ghost">
                <Sparkles class="w-4 h-4 mr-2" /> Regenerate
              </Button>
            </div>
            <p class="nes-text is-disabled">{caseData.summary.overview}</p>
            <h3 class="text-lg font-semibold">Key Findings</h3>
            <ul class="list-disc list-inside nes-text is-disabled">
              {#each Array.isArray(caseData.summary.keyFindings) ? caseData.summary.keyFindings : [] as finding}
                <li>{finding}</li>
              {/each}
            </ul>
            <h3 class="text-lg font-semibold">Recommendations</h3>
            <ul class="list-disc list-inside nes-text is-disabled">
              {#each Array.isArray(caseData.summary.recommendations) ? caseData.summary.recommendations : [] as rec}
                <li>{rec}</li>
              {/each}
            </ul>
            <h3 class="text-lg font-semibold">Risk Assessment</h3>
            <div class="flex items-center gap-2">
              <span class="px-2 py-1 rounded text-xs font-medium border border-gray-300 text-gray-700">{caseData.summary.riskAssessment.level}</span>
              <span class="nes-text is-disabled">({caseData.summary.riskAssessment.factors.join(', ')})</span>
            </div>
            <h3 class="text-lg font-semibold">Timeline</h3>
            <ol class="border-l border-gray-200 space-y-4 pl-4">
              {#each Array.isArray(caseData.summary.timeline) ? caseData.summary.timeline : [] as event}
                <li>
                  <div class="flex items-center">
                    <div class="absolute w-3 h-3 bg-primary rounded-full mt-1.5 -left-1.5 border border-white"></div>
                    <time class="ml-2 text-sm font-semibold text-primary">{formatDate(event.date)}</time>
                  </div>
                  <p class="ml-2 nes-text is-disabled">{event.event}</p>
                </li>
              {/each}
            </ol>
            <h3 class="text-lg font-semibold">Evidence Overview</h3>
            <div class="grid grid-cols-2 gap-4 nes-text is-disabled">
              <div>Total Evidence: {caseData.summary.evidence.total}</div>
              <div>Admissible: {caseData.summary.evidence.admissible}</div>
              <div>Questionable: {caseData.summary.evidence.questionable}</div>
              <div>Inadmissible: {caseData.summary.evidence.inadmissible}</div>
            </div>
            <h3 class="text-lg font-semibold">Next Steps</h3>
            <ul class="list-disc list-inside nes-text is-disabled">
              {#each Array.isArray(caseData.summary.nextSteps) ? caseData.summary.nextSteps : [] as step}
                <li>{step}</li>
              {/each}
            </ul>
          </div>
        {:else}
          <div class="flex flex-col items-center justify-center h-48 nes-text is-disabled">
            <Brain class="w-16 h-16 mb-4 opacity-50" />
            <p>No AI summary available for this case.</p>
            <Button onclick={generateSummary} disabled={isGeneratingSummary} class="mt-4 bits-btn">
              <Sparkles class="w-4 h-4 mr-2" /> Generate Summary
            </Button>
          </div>
        {/if}
      </div>
    </Dialog.Content>
  </Dialog>
{/if}