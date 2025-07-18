<script>
  import { PUBLIC_API_BASE_URL } from "$env/static/public";

  let selectedFile = null;
  let isLoading = false;
  let gradeResults = null;
  let errorMessage = null;
  let showMessage = false;
  let messageText = "";

  function showCustomMessage(message) {
    messageText = message;
    showMessage = true;
    setTimeout(() => {
      showMessage = false;
      messageText = "";
    }, 3000);
  }

  function handleFileChange(event) {
    if (event.target.files.length > 0) {
      selectedFile = event.target.files[0];
      gradeResults = null;
      errorMessage = null;
    } else {
      selectedFile = null;
    }
  }

  async function handleGradeResume() {
    if (!selectedFile) {
      showCustomMessage("Please select a resume file to grade.");
      return;
    }

    isLoading = true;
    gradeResults = null;
    errorMessage = null;

    try {
      const formData = new FormData();
      formData.append("file", selectedFile);

      const response = await fetch(`${PUBLIC_API_BASE_URL}/grader`, {
        method: "POST",
        body: formData,
      });

      if (response.ok) {
        gradeResults = await response.json();
      } else {
        const errorData = await response.json();
        errorMessage = `Error: ${response.status} - ${errorData.detail || response.statusText}`;
      }
    } catch (error) {
      console.error("API call failed:", error);
      errorMessage =
        "Failed to connect to the grading service. Please ensure the backend is running.";
    } finally {
      isLoading = false;
    }
  }

  function splitSentences(text) {
    if (!text) return [];
    return text.split(/(?<=[.!?])\s+|\n+/).filter((s) => s.trim().length > 0);
  }
</script>

<div class="grade-page-container">
  <h1 class="page-title">Grade Your Resume</h1>
  <p class="page-explanation">
    Upload your resume here to get an instant, AI-powered grade. Our system
    analyzes your resume for keywords, formatting, and overall effectiveness to
    help you stand out.
  </p>

  <div class="upload-section">
    <label for="resume-upload" class="upload-button" class:disabled={isLoading}>
      {selectedFile
        ? selectedFile.name.length > 27
          ? selectedFile.name.substring(0, 24) + "..."
          : selectedFile.name
        : "Upload Resume File"}
      <input
        type="file"
        id="resume-upload"
        accept=".pdf,.doc,.docx"
        on:change={handleFileChange}
        class="hidden-input"
        disabled={isLoading}
      />
    </label>
    <button
      on:click={handleGradeResume}
      class="grade-button"
      disabled={!selectedFile || isLoading}
    >
      {#if isLoading}
        Grading...
      {:else}
        Grade Resume
      {/if}
    </button>
  </div>

  {#if isLoading}
    <div class="loading-indicator">
      <div class="spinner"></div>
      <p>Analyzing your resume...</p>
    </div>
  {/if}

  {#if errorMessage}
    <div class="error-message">
      <p>{errorMessage}</p>
    </div>
  {/if}

  {#if gradeResults}
    <div class="results-section">
      <h2 class="results-title">
        Resume Grade: {gradeResults.Grade} ({gradeResults.Score}%)
      </h2>

      <div class="result-card">
        <h3>Highlights</h3>
        <ul class="highlights-list">
          {#each splitSentences(gradeResults.Highlights) as sentence}
            <li>{sentence}</li>
          {/each}
        </ul>
      </div>

      <div class="result-card">
        <h3>Improvements</h3>
        <ul class="improvements-list">
          {#each splitSentences(gradeResults.Improvements) as sentence}
            <li>{sentence}</li>
          {/each}
        </ul>
      </div>

      <div class="result-card">
        <h3>Key Keywords</h3>
        <ul class="keyword-list">
          {#each gradeResults.Keywords as keyword}
            <li>{keyword}</li>
          {/each}
        </ul>
      </div>

      <div class="criteria-cards-container">
        <div
          class="result-card status-card"
          class:status-ok={gradeResults.SectionFormatting === "ok"}
          class:status-needs-work={gradeResults.SectionFormatting ===
            "needs work"}
        >
          <h3>ATS-friendly Formatting</h3>
          {#if gradeResults.SectionFormatting === "needs work"}
            <p class="reasoning-text">
              {gradeResults.SectionFormattingReasoning}
            </p>
          {/if}
        </div>

        <div
          class="result-card status-card"
          class:status-ok={gradeResults.ClarityAction === "ok"}
          class:status-needs-work={gradeResults.ClarityAction === "needs work"}
        >
          <h3>Clarity & Action-Oriented Language</h3>
          {#if gradeResults.ClarityAction === "needs work"}
            <p class="reasoning-text">{gradeResults.ClarityActionReasoning}</p>
          {/if}
        </div>

        <div
          class="result-card status-card"
          class:status-ok={gradeResults.QuantifiableImpact === "ok"}
          class:status-needs-work={gradeResults.QuantifiableImpact ===
            "needs work"}
        >
          <h3>Quantifiable Impact</h3>
          {#if gradeResults.QuantifiableImpact === "needs work"}
            <p class="reasoning-text">
              {gradeResults.QuantifiableImpactReasoning}
            </p>
          {/if}
        </div>

        <div
          class="result-card status-card"
          class:status-ok={gradeResults.KeywordRelevance === "ok"}
          class:status-needs-work={gradeResults.KeywordRelevance ===
            "needs work"}
        >
          <h3>Keyword Relevance</h3>
          {#if gradeResults.KeywordRelevance === "needs work"}
            <p class="reasoning-text">
              {gradeResults.KeywordRelevanceReasoning}
            </p>
          {/if}
        </div>

        <div
          class="result-card status-card"
          class:status-ok={gradeResults.BrevityFormatting === "ok"}
          class:status-needs-work={gradeResults.BrevityFormatting ===
            "needs work"}
        >
          <h3>Brevity & Formatting</h3>
          {#if gradeResults.BrevityFormatting === "needs work"}
            <p class="reasoning-text">
              {gradeResults.BrevityFormattingReasoning}
            </p>
          {/if}
        </div>
      </div>
    </div>
  {/if}

  {#if showMessage}
    <div class="custom-message-box">
      <p>{messageText}</p>
    </div>
  {/if}
</div>

<style>
  .grade-page-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 2rem;
  }

  .page-title {
    font-size: 2.5rem;
    font-weight: 700;
    color: #2d3748;
    margin-bottom: 1rem;
    text-align: center;
    width: 100%;
    max-width: 600px;
  }

  .page-explanation {
    font-size: 1.1rem;
    color: #4a5568;
    max-width: 600px;
    line-height: 1.6;
    margin-bottom: 2rem;
    text-align: left;
    width: 100%;
  }

  .upload-section {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
    width: 100%;
    max-width: 600px;
    margin-top: 1.5rem;
    align-items: center;
  }

  .upload-button,
  .grade-button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0.75rem 1.5rem;
    color: white;
    font-weight: 600;
    border: none;
    border-radius: 0.5rem;
    cursor: pointer;
    transition:
      background-color 0.3s ease,
      box-shadow 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    width: 100%;
    font-size: 1rem;
  }

  .upload-button {
    background-color: #2563eb;
  }

  .upload-button:hover:not(.disabled) {
    background-color: #1d4ed8;
    box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
  }

  .upload-button.disabled {
    background-color: #cbd5e0;
    cursor: not-allowed;
    box-shadow: none;
  }

  .hidden-input {
    display: none;
  }

  .grade-button {
    background-color: #10b981;
  }

  .grade-button:hover:not(:disabled) {
    background-color: #059669;
    box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
  }

  .grade-button:disabled {
    background-color: #cbd5e0;
    cursor: not-allowed;
    box-shadow: none;
  }

  .loading-indicator {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 2rem;
    color: #4a5568;
    font-size: 1.1rem;
  }

  .spinner {
    border: 4px solid rgba(0, 0, 0, 0.1);
    border-left-color: #2563eb;
    border-radius: 50%;
    width: 40px;
    height: 40px;
    animation: spin 1s linear infinite;
    margin-bottom: 1rem;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  .error-message {
    margin-top: 2rem;
    padding: 1rem;
    background-color: #fee2e2;
    color: #ef4444;
    border: 1px solid #ef4444;
    border-radius: 0.5rem;
    width: 100%;
    max-width: 600px;
    text-align: left;
  }

  .results-section {
    margin-top: 2rem;
    width: 100%;
    max-width: 800px;
    text-align: left;
  }

  .results-title {
    font-size: 2rem;
    font-weight: 700;
    color: #2d3748;
    margin-bottom: 1.5rem;
    text-align: center;
  }

  .result-card {
    background-color: #f0f4f8;
    padding: 1.5rem;
    margin-bottom: 1rem;
    border-radius: 0.75rem;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
    color: #4a5568;
  }

  .result-card h3 {
    font-size: 1.4rem;
    font-weight: 600;
    color: #2563eb;
    margin-bottom: 0.75rem;
  }

  .result-card p {
    font-size: 1rem;
    line-height: 1.6;
    color: #4a5568;
  }

  .reasoning-text {
    font-size: 0.9rem;
    color: #4a5568;
    margin-top: 0.5rem;
    line-height: 1.4;
  }

  .keyword-list,
  .highlights-list,
  .improvements-list {
    list-style: none;
    padding-left: 0;
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem;
  }

  .keyword-list li,
  .highlights-list li,
  .improvements-list li {
    font-size: 1rem;
    color: #4a5568;
    position: relative;
    background-color: #e2e8f0;
    padding: 0.75rem 1rem;
    border-radius: 0.5rem;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
    word-break: break-word;
    line-height: 1.6;
    flex-shrink: 0;
    flex-grow: 0;
    max-width: fit-content;
  }

  .criteria-cards-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1rem;
    margin-top: 1.5rem;
    width: 100%;
    max-width: 800px;
  }

  .status-card {
    background-color: #f0f4f8;
    color: #4a5568;
    padding: 1.2rem;
    border-radius: 0.75rem;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
    position: relative;
    display: flex;
    flex-direction: column;
  }

  .status-card h3 {
    color: #2563eb;
    margin-bottom: 0.5rem;
    font-size: 1.2rem;
  }

  .status-card p {
    color: #4a5568;
    line-height: 1.5;
  }

  .status-ok {
    background-color: #d4edda;
    color: #155724;
  }

  .status-ok h3,
  .status-ok p,
  .status-ok .reasoning-text {
    color: #155724;
  }

  .status-needs-work {
    background-color: #f8d7da;
    color: #721c24;
  }

  .status-needs-work h3,
  .status-needs-work p,
  .status-needs-work .reasoning-text {
    color: #721c24;
  }

  @media (max-width: 600px) {
    .page-title {
      font-size: 2rem;
    }

    .page-explanation {
      font-size: 1rem;
    }

    .upload-section {
      flex-direction: column;
      max-width: 90%;
    }

    .results-title {
      font-size: 1.75rem;
    }

    .result-card h3 {
      font-size: 1.2rem;
    }

    .result-card p,
    .keyword-list li,
    .highlights-list li,
    .improvements-list li {
      font-size: 0.95rem;
    }

    .keyword-list li,
    .highlights-list li,
    .improvements-list li {
      padding-left: 0.75rem;
      padding-right: 0.75rem;
    }

    .criteria-cards-container {
      grid-template-columns: 1fr;
    }
  }

  .custom-message-box {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    background-color: #333;
    color: white;
    padding: 10px 20px;
    border-radius: 5px;
    z-index: 1000;
    opacity: 0;
    animation: fadeInOut 3s forwards;
  }

  @keyframes fadeInOut {
    0% {
      opacity: 0;
    }
    10% {
      opacity: 1;
    }
    90% {
      opacity: 1;
    }
    100% {
      opacity: 0;
    }
  }
</style>
