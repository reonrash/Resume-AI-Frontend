<script>
  import { PUBLIC_API_BASE_URL } from "$env/static/public";

  let selectedResumeFile = null;
  let jobApplicationText = "";
  let isLoading = false;
  let comparisonResults = null;
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

  function handleResumeFileChange(event) {
    if (event.target.files.length > 0) {
      selectedResumeFile = event.target.files[0];
      comparisonResults = null;
      errorMessage = null;
    } else {
      selectedResumeFile = null;
    }
  }

  function handleJobApplicationTextChange(event) {
    jobApplicationText = event.target.value;
    comparisonResults = null;
    errorMessage = null;
  }

  function splitSentences(text) {
    const stringText = String(text || "");
    if (!stringText.trim()) return [];
    return stringText
      .split(/(?<=[.!?])\s+|\n+/)
      .filter((s) => s.trim().length > 0);
  }

  async function handleCompareResumes() {
    if (!selectedResumeFile) {
      showCustomMessage("Please upload a resume file.");
      return;
    }
    if (!jobApplicationText.trim()) {
      showCustomMessage("Please paste job application information.");
      return;
    }

    const sentences = jobApplicationText
      .split(/[.!?]+\s*|\n+/)
      .filter((s) => s.trim().length > 0);
    if (sentences.length <= 4) {
      showCustomMessage(
        "Please provide more detailed job application information (at least 5 sentences)."
      );
      return;
    }

    isLoading = true;
    comparisonResults = null;
    errorMessage = null;

    try {
      const formData = new FormData();
      formData.append("file", selectedResumeFile);
      formData.append("job_application_text", jobApplicationText);

      const response = await fetch(`${PUBLIC_API_BASE_URL}/comparison`, {
        method: "POST",
        body: formData,
      });

      if (response.ok) {
        comparisonResults = await response.json();
      } else {
        const errorData = await response.json();
        errorMessage = `Error: ${response.status} ${errorData.detail || "Unknown Error"}`;
      }
    } catch (error) {
      console.error("API call failed:", error);
      errorMessage =
        "Failed to connect to the comparison service. Please ensure the backend is running.";
    } finally {
      isLoading = false;
    }
  }
</script>

<div class="compare-page-container">
  <h1 class="page-title">Compare Resume to Job Application</h1>
  <p class="page-explanation">
    Upload your resume and paste the job application details to see how well you
    match. Get insights into keywords, strengths, and areas for improvement
    tailored to the specific role.
  </p>

  <div class="input-section">
    <div class="file-upload-area">
      <label
        for="resume-upload"
        class="upload-button"
        class:disabled={isLoading}
      >
        {selectedResumeFile ? selectedResumeFile.name : "Upload Your Resume"}
        <input
          type="file"
          id="resume-upload"
          accept=".pdf,.doc,.docx"
          on:change={handleResumeFileChange}
          class="hidden-input"
          disabled={isLoading}
        />
      </label>
    </div>

    <div class="text-area-section">
      <textarea
        placeholder="Paste job application details here (job description, requirements, etc.)"
        bind:value={jobApplicationText}
        on:input={handleJobApplicationTextChange}
        class="job-app-textarea"
        disabled={isLoading}
      ></textarea>
    </div>

    <button
      on:click={handleCompareResumes}
      class="compare-button"
      disabled={!selectedResumeFile || !jobApplicationText.trim() || isLoading}
    >
      {#if isLoading}
        Comparing...
      {:else}
        Compare Resume
      {/if}
    </button>
  </div>

  {#if isLoading}
    <div class="loading-indicator">
      <div class="spinner"></div>
      <p>Comparing your resume with the job application...</p>
    </div>
  {/if}

  {#if errorMessage}
    <div class="error-message">
      <p>{errorMessage}</p>
    </div>
  {/if}

  {#if comparisonResults}
    <div class="results-section">
      <h2 class="results-title">
        Comparison Grade: {comparisonResults.Grade}
      </h2>

      <div class="result-card">
        <h3>Keyword Difference</h3>
        <ul class="comparison-list">
          {#each comparisonResults["Keyword difference"] as keywordDiff}
            <li><span>{keywordDiff}</span></li>
          {/each}
        </ul>
      </div>

      <div class="result-card">
        <h3>Skill Gap Analysis</h3>
        <p>{comparisonResults["Skill Gap Analysis"]}</p>
      </div>

      <div class="result-card">
        <h3>Impact & Clarity Gap</h3>
        <p>{comparisonResults["Impact & Clarity Gap"]}</p>
      </div>

      <div class="result-card">
        <h3>Recommendations</h3>
        <p>{comparisonResults.Recommendations}</p>
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
  .compare-page-container {
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

  .input-section {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
    width: 100%;
    max-width: 600px;
    margin-top: 1.5rem;
    align-items: center;
  }

  .file-upload-area {
    width: 100%;
  }

  .upload-button,
  .compare-button {
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

  .text-area-section {
    width: 100%;
  }

  .job-app-textarea {
    width: 100%;
    min-height: 200px;
    padding: 1rem;
    border: 1px solid #cbd5e0;
    border-radius: 0.5rem;
    font-size: 1rem;
    color: #4a5568;
    resize: vertical;
    box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.05);
    transition:
      border-color 0.3s ease,
      box-shadow 0.3s ease;
  }

  .job-app-textarea:focus {
    outline: none;
    border-color: #2563eb;
    box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.2);
  }

  .job-app-textarea:disabled {
    background-color: #f0f4f8;
    cursor: not-allowed;
  }

  .compare-button {
    background-color: #2563eb;
  }

  .compare-button:hover:not(:disabled) {
    background-color: #1d4ed8;
    box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
  }

  .compare-button:disabled {
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
    background-color: #f0f4f8;
    padding: 1.5rem;
    border-radius: 0.75rem;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
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

  .comparison-list {
    list-style: none;
    padding-left: 0;
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem;
  }

  .comparison-list li {
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
    display: flex;
    align-items: center;
  }

  @media (max-width: 600px) {
    .page-title {
      font-size: 2rem;
    }

    .page-explanation {
      font-size: 1rem;
    }

    .input-section {
      max-width: 90%;
    }

    .job-app-textarea {
      min-height: 150px;
    }

    .results-title {
      font-size: 1.75rem;
    }

    .result-card h3 {
      font-size: 1.2rem;
    }

    .result-card p,
    .comparison-list li {
      font-size: 0.95rem;
    }

    .comparison-list li {
      padding-left: 0.75rem;
      padding-right: 0.75rem;
      flex-direction: column;
      align-items: flex-start;
    }

    .comparison-list li span {
      margin-right: 0;
      margin-bottom: 0.5rem;
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
