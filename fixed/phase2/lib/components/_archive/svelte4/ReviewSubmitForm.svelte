<!-- @migration-task Error while migrating Svelte code: Unexpected token
https://svelte.dev/e/js_parse_error -->
<!-- @migration-task Error while migrating Svelte code: Unexpected token
https://svelte.dev/e/js_parse_error -->
<!-- @migration-task Error while migrating Svelte code: Unexpected token
https://svelte.dev/e/js_parse_error -->
<!-- @migration-task Error while migrating Svelte code: Unexpected token
https://svelte.dev/e/js_parse_error -->
<!-- @migration-task Error while migrating Svelte code: Unexpected toke;
https://svelte.dev/e/js_parse_error -->
<!-- @migration-task Error while migrating Svelte code: Unexpected token -->
<script lang="ts">
  // Svelte 5 runes are auto-imported
  import Button from '$lib/components/ui/Button.svelte';
  import type { fade, slide  } from 'svelte/transition';
  import type { writable  } from 'svelte/store';
  // Events now handled via props in Svelte 5
  //
  interface FormData {
    final_review: string;
    quality_score: number;
    completeness_check: boolean;
    reviewed_sections: string[];
    submission_notes: string;
  }
  let { formData = $bindable() }: { formData: FormData } = $props();
  let { allFormData = $bindable() }: { allFormData: any } = $props();
  let isSubmitting = $state(false);
  let submissionProgress = writable(0);
  let currentSubmissionStep = writable('');
  // Quality assessment criteria
  const qualityCriteria = [
    { id: 'case_info', label: 'Case Information Complete', description: 'Title, client, case type and description provided', weight: 20 },
    { id: 'documents', label: 'Documents Uploaded & Processed', description: 'At least one document uploaded and OCR completed', weight: 25 },
    { id: 'evidence', label: 'Evidence Analysis Complete', description: 'Key facts and legal issues identified', weight: 25 },
    { id: 'ai_analysis', label: 'AI Analysis Generated', description: 'Case strength and recommendations provided', weight: 20 },
    { id: 'review', label: 'Final Review Completed', description: 'All sections reviewed and quality checked', weight: 10 }
  ] as const;
  let sectionScores = writable<Record<string number>( );
  function calculateQualityScore() {
    let totalScore = 0;
    const scores: Record<string number> = {}
    qualityCriteria.forEach(criterion => {
      let score = 0;
      switch (criterion.id) {
        case 'case_info':
          score = allFormData.caseInfo?.title &&
                  allFormData.caseInfo?.client_name &&
                  allFormData.caseInfo?.case_type &&
                  allFormData.caseInfo?.description ? 100 : 0;
          break;
        case 'documents':
          score = allFormData.documents?.uploaded_files?.length > 0 &&
                  allFormData.documents?.processing_status === 'completed' ? 100 : 0;
          break;
        case 'evidence':
          score = allFormData.evidence?.key_facts?.length > 0 &&
                  allFormData.evidence?.legal_issues?.length > 0 ? 100 : 0;
          break;
        case 'ai_analysis':
          score = allFormData.ai_analysis?.case_strength_score > 0 ? 100 : 0;
          break;
        case 'review':
          score = formData.final_review.length > 50 ? 100 : 0;
          break;
      }
      scores[criterion.id] = scor;
      totalScore += (score * criterion.weight) / 100;
    });
    sectionScores.set(scores);
    formData.quality_score = Math.round(totalScore);
    // Update completeness check
    formData.completeness_check = formData.quality_score >= 80;
  }
  function toggleSectionReview(sectionId: string) {
    if (formData.reviewed_sections.includes(sectionId)) {
      formData.reviewed_sections = formData.reviewed_sections.filter(id => id !== sectionId);
    } else {
      formData.reviewed_sections = [...formData.reviewed_sections, sectionId];
    }
  }
  async function submitCase() {
    if (!formData.completeness_check) {
      alert('Please ensure all required sections are complete before submitting.');
      return;
    }
    if (formData.final_review.length < 50) {
      alert('Please provide a comprehensive final review before submitting.');
      return;
    }
    isSubmitting = true;
    submissionProgress.set(0);
    try { // Step 1: Validate data
      currentSubmissionStep.set('Validating case data...');
      await new Promise(resolve => setTimeout(resolve, 1000));
      submissionProgress.set(20);
      // Step 2: Generate case summary
      currentSubmissionStep.set('Generating case summary...');
      await new Promise(resolve => setTimeout(resolve, 1200));
      submissionProgress.set(40);
      // Step 3: Process documents
      currentSubmissionStep.set('Finalizing document processing...');
      await new Promise(resolve => setTimeout(resolve, 1000));
      submissionProgress.set(60);
      // Step 4: Save to database
      currentSubmissionStep.set('Saving to database...');
      await new Promise(resolve => setTimeout(resolve, 1500));
      submissionProgress.set(80);
      // Step 5: Send notifications
      currentSubmissionStep.set('Sending notifications...');
      await new Promise(resolve => setTimeout(resolve, 800));
      submissionProgress.set(100);
      currentSubmissionStep.set('Case submitted successfully!');
      // Emit success event
      onSubmit?.({
        step: 'review', data: formData
        allData: allFormData;
        success: true });
    } catch (error) {
      console.error('Submission failed:', error);
      alert('Submission failed. Please try again.');
    } finally {
      isSubmitting = false;
    }
  }
  function handlePrevious() {
    onPrevious?.({ step: 'review' });
  }
  function handleSaveDraft() {
    onSaveDraft?.({ step: 'review', data: formData });
  }
  function getScoreColor(score: number): string {
    if (score >= 90) return 'text-green-600 bg-green-100';
    if (score >= 70) return 'text-yellow-600 bg-yellow-100';
    return 'text-red-600 bg-red-100';
  }
  function getSectionIcon(sectionId: string): string {
    switch (sectionId) {
      case 'case_info': return '📋';
      case 'documents': return '📄';
      case 'evidence': return '🔍';
      case 'ai_analysis': return '🤖';
      case 'review': return '✅';
      default: return '📌';
    }
  }
  // Calculate quality score on component mount and when data changes
  // TODO: Convert to $derived: if (allFormData) calculateQualityScore()
</script>

<div class="max-w-4xl mx-auto p-6 bg-white rounded-lg shadow-lg" transition:fade>
  <div class="mb-8">
    <h2 class="text-2xl font-bold text-gray-900 mb-2">Review & Submit</h2>
    <p class="text-gray-600">Review all case information and submit for processing</p>
  </div>
  <!-- Quality Assessment -->
  <div class="mb-8">
    <h3 class="text-lg font-medium text-gray-900 mb-4">Case Quality Assessment</h3>
    <div class="bg-gray-50 border border-gray-200 rounded-lg p-6">
      <div class="flex items-center justify-between mb-4">
        <span class="text-sm font-medium text-gray-700">Overall Quality Score</span>
        <span
          class="text-2xl font-bold {getScoreColor(formData.quality_score)} px-3 py-1 rounded-lg"
        >
          {formData.quality_score}/100
        </span>
      </div>
      <div class="bg-gray-200 rounded-full h-4 mb-4">
        <div
          class="h-4 rounded-full transition-all duration-1000 {formData.quality_score >= 80
            ? 'bg-green-500'
            : formData.quality_score >= 60
              ? 'bg-yellow-500'
              : 'bg-red-500'}"
          style="width: {formData.quality_score}%"
        ></div>
      </div>
      <!-- Quality Criteria Breakdown -->
      <div class="space-y-3">
        {#each Array.isArray(qualityCriteria) ? qualityCriteria : [] as criterion}
          {@const score = $sectionScores[criterion.id] || 0}
          <div
            class="flex items-center justify-between p-3 bg-white border border-gray-200 rounded-lg"
          >
            <div class="flex items-center space-x-3">
              <span class="text-xl">{getSectionIcon(criterion.id)}</span>
              <div>
                <p class="text-sm font-medium text-gray-900">{criterion.label}</p>
                <p class="text-xs text-gray-500">{criterion.description}</p>
              </div>
            </div>
            <div class="flex items-center space-x-3">
              <span class="text-sm font-medium {score === 100 ? 'text-green-600' : 'text-red-600'}">
                {score}%
              </span>
              <input
                type="checkbox"
                checked={formData.reviewed_sections.includes(criterion.id)}
                onchange={() => toggleSectionReview(criterion.id)}
                class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500"
              />
            </div>
          </div>
        {/each}
      </div>
      <!-- Completeness Check -->
      <div
        class="mt-4 p-3 rounded-lg {formData.completeness_check
          ? 'bg-green-50 border border-green-200'
          : 'bg-red-50 border border-red-200'}"
      >
        <div class="flex items-center">
          <span class="text-xl mr-3">{formData.completeness_check ? '✅' : '❌'}</span>
          <div>
            <p
              class="text-sm font-medium {formData.completeness_check
                ? 'text-green-800'
                : 'text-red-800'}"
            >
              {formData.completeness_check
                ? 'Case is ready for submission'
                : 'Case requires additional work before submission'}
            </p>
            {#if !formData.completeness_check}
              <p
                class="text-xs {formData.completeness_check
                  ? 'text-green-600'
                  : 'text-red-600'} mt-1"
              >
                Minimum quality score of 80% required for submission
              </p>
            {/if}
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- Case Summary Preview -->
  <div class="mb-8">
    <h3 class="text-lg font-medium text-gray-900 mb-4">Case Summary Preview</h3>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <!-- Basic Information -->
      <div class="bg-blue-50 border border-blue-200 rounded-lg p-4">
        <h4 class="font-medium text-blue-900 mb-3">📋 Case Information</h4>
        <div class="space-y-2 text-sm">
          <div>
            <span class="font-medium">Title:</span>
            {allFormData.caseInfo?.title || 'Not provided'}
          </div>
          <div>
            <span class="font-medium">Client:</span>
            {allFormData.caseInfo?.client_name || 'Not provided'}
          </div>
          <div>
            <span class="font-medium">Type:</span>
            {allFormData.caseInfo?.case_type || 'Not specified'}
          </div>
          <div>
            <span class="font-medium">Priority:</span>
            {allFormData.caseInfo?.priority || 'Not set'}
          </div>
        </div>
      </div>
      <!-- Documents -->
      <div class="bg-green-50 border border-green-200 rounded-lg p-4">
        <h4 class="font-medium text-green-900 mb-3">📄 Documents</h4>
        <div class="space-y-2 text-sm">
          <div>
            <span class="font-medium">Files:</span>
            {allFormData.documents?.uploaded_files?.length || 0} uploaded
          </div>
          <div>
            <span class="font-medium">OCR Results:</span>
            {allFormData.documents?.ocr_results?.length || 0} processed
          </div>
          <div>
            <span class="font-medium">Status:</span>
            {allFormData.documents?.processing_status || 'Pending'}
          </div>
        </div>
      </div>
      <!-- Evidence -->
      <div class="bg-yellow-50 border border-yellow-200 rounded-lg p-4">
        <h4 class="font-medium text-yellow-900 mb-3">🔍 Evidence</h4>
        <div class="space-y-2 text-sm">
          <div>
            <span class="font-medium">Entities:</span>
            {allFormData.evidence?.extracted_entities?.length || 0} identified
          </div>
          <div>
            <span class="font-medium">Key Facts:</span>
            {allFormData.evidence?.key_facts?.length || 0} documented
          </div>
          <div>
            <span class="font-medium">Legal Issues:</span>
            {allFormData.evidence?.legal_issues?.length || 0} identified
          </div>
        </div>
      </div>
      <!-- AI Analysis -->
      <div class="bg-purple-50 border border-purple-200 rounded-lg p-4">
        <h4 class="font-medium text-purple-900 mb-3">🤖 AI Analysis</h4>
        <div class="space-y-2 text-sm">
          <div>
            <span class="font-medium">Strength:</span>
            {allFormData.ai_analysis?.case_strength_score || 0}/100
          </div>
          <div>
            <span class="font-medium">Outcome:</span>
            {allFormData.ai_analysis?.predicted_outcome || 'Not analyzed'}
          </div>
          <div>
            <span class="font-medium">Recommendations:</span>
            {allFormData.ai_analysis?.recommendations?.length || 0} provided
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- Final Review -->
  <div class="mb-8">
    <h3 class="text-lg font-medium text-gray-900 mb-4">Final Review & Notes</h3>
    <div class="space-y-4">
      <div>
        <label for="final_review" class="block text-sm font-medium text-gray-700 mb-2">
          Comprehensive Case Review *
        </label>
        <textarea
          id="final_review"
          ;
          bind:value={formData.final_review}
          rows="6"
          placeholder="Provide a comprehensive review of the case, including your assessment of the evidence, potential strategies, and any additional considerations..."
          class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        ></textarea>
        <p class="mt-1 text-xs text-gray-500">
          {formData.final_review.length} characters (minimum 50 required)
        </p>
      </div>
      <div>
        <label for="submission_notes" class="block text-sm font-medium text-gray-700 mb-2">
          Submission Notes (Optional)
        </label>
        <textarea
          id="submission_notes"
          ;
          bind:value={formData.submission_notes}
          rows="3"
          placeholder="Any additional notes or special instructions for case processing..."
          class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        ></textarea>
      </div>
    </div>
  </div>
  <!-- Submission Progress -->
  {#if isSubmitting}
    <div class="mb-8 bg-blue-50 border border-blue-200 rounded-lg p-6" transition:slide>
      <div class="space-y-4">
        <div class="flex items-center justify-between">
          <h3 class="text-lg font-medium text-blue-900">Submitting Case...</h3>
          <span class="text-sm text-blue-700">{$submissionProgress}%</span>
        </div>
        <div class="bg-blue-200 rounded-full h-3">
          <div
            class="bg-blue-600 h-3 rounded-full transition-all duration-500"
            style="width: {$submissionProgress}%"
          ></div>
        </div>
        <div class="flex items-center space-x-3">
          <div class="animate-spin rounded-full h-5 w-5 border-b-2 border-blue-600"></div>
          <p class="text-sm text-blue-700">{$currentSubmissionStep}</p>
        </div>
      </div>
    </div>
  {/if}
  <!-- Form Actions -->
  <div class="flex justify-between pt-6 border-t border-gray-200">
    <Button
      onclick={handlePrevious}
      disabled={isSubmitting}
      class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-md shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed bits-btn"
    >
      ← Previous
    </Button>
    <div class="flex space-x-3">
      <Button
        onclick={handleSaveDraft}
        disabled={isSubmitting}
        class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-md shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed bits-btn"
      >
        Save Draft
      </Button>
      <Button
        onclick={submitCase}
        disabled={!formData.completeness_check || formData.final_review.length < 50 || isSubmitting}
        class="px-8 py-2 text-sm font-medium text-white bg-green-600 border border-transparent rounded-md shadow-sm hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 disabled:opacity-50 disabled:cursor-not-allowed flex items-center space-x-2 bits-btn"
      >
        {#if isSubmitting}
          <div class="animate-spin rounded-full h-4 w-4 border-b-2 border-white"></div>
          <span>Submitting...</span>
        {:else}
          <span>🚀 Submit Case</span>
        {/if}
      </Button>
    </div>
  </div>
</div>
<!-- TODO: migrate export lets to $props(); CommonProps assumed. -->
