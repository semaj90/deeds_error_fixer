<script lang="ts">
  // Svelte 5 runes are auto-imported
  import type { SearchResults } from "$lib/types/global";
  import type { onMount  } from 'svelte';
  import Button from '$lib/components/ui/enhanced-bits.svelte';
  import 
    Input
   from "$lib/components/ui/enhanced-bits.svelte";
  // Badge replaced with span - not available in enhanced-bits
  import 
    Card,
    CardHeader,
    CardTitle,
    CardContent
   from "$lib/components/ui/enhanced-bits.svelte";
  import type { Loader2, Search, Brain, Zap, Database  } from 'lucide-svelte';
  import type { context7Service, type VectorIntelligence  } from '$lib/services/context7Service';
  // Reactive state from Context7 service
  const { vectorResults, isAnalyzing } = context7Servic;
  let searchQuery = $state('');
  let selectedFilters = $state<string[] >([]);
  let searchResults = $state<VectorIntelligence: null >(null);
  let searchHistory = $state<string[] >([]);
  // Demo data and filters
  const availableFilters = [
    'contracts', 'evidence', 'case-law', 'regulations',
    'high-similarity', 'recent', 'archived'
  ];
  const sampleQueries = [
    'liability clauses in employment contracts',
    'digital evidence chain of custody',
    'precedent cases for intellectual property',
    'regulatory compliance for financial services'
  ];
  $effect(() => {
    (async () => {
await context7Service.initialize();
    // Load search history from localStorage
    const saved = localStorage.getItem('vector-search-history');
    if (saved) {
      searchHistory = JSON.parse(saved);
    }
    })();
  });
  // Subscribe to vector results from service
  $effect(() => {
    searchResults = $vectorResult;
  });
  async function performSearch() { if (!searchQuery.trim()) return;
    await context7Service.vectorSearch(searchQuery, {
      filters: selectedFilters;
      limit: 10 });
    // Add to search history
    if (!searchHistory.includes(searchQuery)) {
      searchHistory = [searchQuery, ...searchHistory.slice(0, 9)]; // Keep last 10
      localStorage.setItem('vector-search-history', JSON.stringify(searchHistory));
    }
  }
  function toggleFilter(filter: string) {
    if (selectedFilters.includes(filter)) {
      selectedFilters = selectedFilters.filter(f => f !== filter);
    } else {
      selectedFilters = [...selectedFilters, filter];
    }
  }
  function useSampleQuery(query: string) {
    searchQuery = query;
    performSearch();
  }
  function useHistoryQuery(query: string) {
    searchQuery = query;
  }
  function clearHistory() {
    searchHistory = [];
    localStorage.removeItem('vector-search-history');
  }
  function getSimilarityColor(similarity: number): string {
    if (similarity >= 0.9) return 'bg-green-500';
    if (similarity >= 0.7) return 'bg-yellow-500';
    return 'bg-red-500';
  }
  function formatSimilarity(similarity: number): string {
    return `${(similarity * 100).toFixed(1)}%`;
  }
</script>
<div class="container mx-auto p-6 max-w-6xl">
  <!-- Header -->
  <div class="mb-8">
    <div class="flex items-center gap-3 mb-4">
      <Brain class="h-8 w-8 text-blue-600" />
      <h1 class="text-3xl font-bold">Vector Intelligence Demo</h1>
      <span class="px-2 py-1 rounded text-xs font-medium bg-gray-200 text-gray-700">Phase 5 Enhanced</span>
    </div>
    <p class="text-gray-600">
      Demonstrate semantic search, document similarity, and AI-powered legal research capabilities
    </p>
  </div>
  <!-- Search Interface -->
  <div class="mb-6 nes-container">
    <div class="yorha-panel-header">
      <h3 class="nes-text is-primary flex items-center gap-2">
        <Search class="h-5 w-5" />
        Semantic Search
      </h3>
      <p class="nes-text">
        Search across legal documents using natural language and AI-powered similarity matching
      </p>
    </div>
    <div class="yorha-panel-content space-y-4">
      <!-- Search Input -->
      <div class="flex gap-2">
        <Input
          bind:value={searchQuery}
          placeholder="Enter your legal research question..."
          class="flex-1"
          keydown={(e) => e.key === 'Enter' && performSearch()}
        />
        <Button
          onclick={performSearch}
          disabled={$isAnalyzing || !searchQuery.trim()}
          class="px-6 bits-btn bits-btn"
        >
{#if $isAnalyzing}
            <Loader2 class="h-4 w-4 animate-spin mr-2" />
            Searching
          {:else}
            <Search class="h-4 w-4 mr-2" />
            Search
          {/if}
</Button>
      </div>
      <!-- Filters -->
      <div class="space-y-2">
        <label class="text-sm font-medium">Filters:</label>
        <div class="flex flex-wrap gap-2">
          {#each Array.isArray(availableFilters) ? availableFilters : [] as filter}
            <Badge
              variant={selectedFilters.includes(filter) ? 'default' : 'outline'}
              class="cursor-pointer hover:bg-blue-100 transition-colors"
              onclick={() => toggleFilter(filter)}
            >
              {filter}
            </Badge>
          {/each}
        </div>
      </div>
      <!-- Sample Queries -->
      <div class="space-y-2">
        <label class="text-sm font-medium">Sample Queries:</label>
        <div class="flex flex-wrap gap-2">
          {#each Array.isArray(sampleQueries) ? sampleQueries : [] as query}
            <Button
              variant="ghost"
              size="sm"
              class="text-xs bits-btn bits-btn"
              onclick={() =>
useSampleQuery(query)}
            >
              {query}
</Button>
          {/each}
        </div>
      </div>
    </div>
  </div>
  <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
    <!-- Search Results -->
    <div class="lg:col-span-2">
      <div class="nes-container">
        <div class="yorha-panel-header">
          <h3 class="nes-text is-primary flex items-center gap-2">
            <Database class="h-5 w-5" />
            Search Results
            {#if searchResults?.results.length}
              <span class="px-2 py-1 rounded text-xs font-medium bg-gray-200 text-gray-700">{searchResults.results.length} matches</span>
            {/if}
          </h3>
        </div>
        <div class="yorha-panel-content">
          {#if $isAnalyzing}
            <div class="flex items-center justify-center py-12">
              <div class="text-center">
                <Loader2 class="h-8 w-8 animate-spin mx-auto mb-4 text-blue-600" />
                <p class="text-gray-600">Analyzing documents with AI...</p>
              </div>
            </div>
          {:else if searchResults?.results.length}
            <div class="space-y-4">
              {#each searchResults.results as result, index}
                <div class="border rounded-lg p-4 hover:bg-gray-50 transition-colors">
                  <div class="flex items-start justify-between mb-3">
                    <div class="flex-1">
                      <h3 class="font-medium text-gray-900 mb-1">
                        Document {index + 1}
                      </h3>
                      <p class="text-sm text-gray-600 line-clamp-3">
                        {(result as { content?: any; similarity?: any; metadata?: any }).content}
                      </p>
                    </div>
                    <div class="ml-4 text-right">
                      <div class="flex items-center gap-2">
                        <div
                          class="w-3 h-3 rounded-full {getSimilarityColor((result as { content?: any; similarity?: any; metadata?: any }).similarity)}"
                        ></div>
                        <span class="text-sm font-medium">
                          {formatSimilarity((result as { content?: any; similarity?: any; metadata?: any }).similarity)}
                        </span>
                      </div>
                    </div>
                  </div>
                  <!-- Metadata -->
                  <div class="flex flex-wrap gap-2 mt-3">
                    {#each Object.entries.metadata) as [key, value]}
                      <span class="px-2 py-1 rounded text-xs font-medium border border-gray-300 text-gray-700">{key}: {value}</span>
                    {/each}
                  </div>
                </div>
              {/each}
            </div>
          {:else if searchQuery}
            <div class="text-center py-12">
              <Database class="h-12 w-12 mx-auto mb-4 text-gray-400" />
              <p class="text-gray-600 mb-2">No results found</p>
              <p class="text-sm text-gray-500">Try adjusting your search terms or filters</p>
            </div>
          {:else}
            <div class="text-center py-12">
              <Search class="h-12 w-12 mx-auto mb-4 text-gray-400" />
              <p class="text-gray-600 mb-2">Start your search</p>
              <p class="text-sm text-gray-500">Enter a legal research question above</p>
            </div>
          {/if}
        </div>
      </div>
    </div>
    <!-- Sidebar -->
    <div class="space-y-6">
      <!-- AI Suggestions -->
      {#if searchResults?.suggestions.length}
        <div class="nes-container">
          <div class="yorha-panel-header">
            <h3 class="nes-text is-primary flex items-center gap-2">
              <Zap class="h-5 w-5" />
              AI Suggestions
            </h3>
          </div>
          <div class="yorha-panel-content">
            <div class="space-y-2">
              {#each Array.isArray(searchResults.suggestions) ? searchResults.suggestions : [] as suggestion}
                <div class="p-3 bg-blue-50 rounded-lg text-sm">
                  {suggestion}
                </div>
              {/each}
            </div>
          </div>
        </div>
      {/if}
      <!-- Search History -->
      {#if searchHistory.length > 0}
        <div class="nes-container">
          <div class="yorha-panel-header">
            <div class="flex items-center justify-between">
              <h3 class="nes-text is-primary text-lg">Recent Searches</h3>
              <Button class="bits-btn" variant="ghost" size="sm" onclick={clearHistory}>
Clear
</Button>
            </div>
          </div>
          <div class="yorha-panel-content">
            <div class="space-y-2">
              {#each Array.isArray(searchHistory) ? searchHistory : [] as query}
                <button
                  class="w-full text-left p-2 text-sm rounded hover:bg-gray-100 transition-colors"
                  onclick={() => useHistoryQuery(query)}
                >
                  {query}
</Button>
              {/each}
            </div>
          </div>
        </div>
      {/if}
      <!-- System Stats -->
      <div class="nes-container">
        <div class="yorha-panel-header">
          <h3 class="nes-text is-primary text-lg">System Status</h3>
        </div>
        <div class="yorha-panel-content">
          <div class="space-y-3 text-sm">
            <div class="flex justify-between">
              <span>Vector Database:</span>
              <span class="px-2 py-1 rounded text-xs font-medium border border-gray-300 text-gray-700">Active</span>
            </div>
            <div class="flex justify-between">
              <span>AI Engine:</span>
              <span class="px-2 py-1 rounded text-xs font-medium border border-gray-300 text-gray-700">Ready</span>
            </div>
            <div class="flex justify-between">
              <span>Cache:</span>
              <span class="px-2 py-1 rounded text-xs font-medium border border-gray-300 text-gray-700">Enabled</span>
            </div>
            <div class="flex justify-between">
              <span>Context7:</span>
              <span class="px-2 py-1 rounded text-xs font-medium border border-gray-300 text-gray-700">Connected</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
<style>
  .line-clamp-3 {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
</style>