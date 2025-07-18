<script>
  import { PUBLIC_API_BASE_URL } from "$env/static/public";

  let inputBulletPoints = "";
  let isLoading = false;
  let enhancedBulletPoints = null;
  let errorMessage = null;
  let copiedMessage = "";
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

  function handleInputChange(event) {
    inputBulletPoints = event.target.value;
    errorMessage = null;
    copiedMessage = "";
  }

  async function handleEnhanceBulletPoints() {
    if (!inputBulletPoints.trim()) {
      showCustomMessage("Please enter some bullet points to enhance.");
      return;
    }

    isLoading = true;
    enhancedBulletPoints = null;
    errorMessage = null;
    copiedMessage = "";

    try {
      const response = await fetch(`${PUBLIC_API_BASE_URL}/augment`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ bullet_point: inputBulletPoints }),
      });

      if (response.ok) {
        const data = await response.json();
        enhancedBulletPoints = data.options;
      } else {
        const errorData = await response.json();
        errorMessage = `Error: ${response.status} ${errorData.detail || "Unknown Error"}`;
      }
    } catch (error) {
      console.error("API call failed:", error);
      errorMessage =
        "Failed to connect to the enhancement service. Please ensure the backend is running.";
    } finally {
      isLoading = false;
    }
  }

  function copyToClipboard(text) {
    const textarea = document.createElement("textarea");
    textarea.value = text;
    document.body.appendChild(textarea);
    textarea.select();
    try {
      document.execCommand("copy");
      copiedMessage = "Copied!";
      setTimeout(() => {
        copiedMessage = "";
      }, 2000);
    } catch (err) {
      console.error("Failed to copy text: ", err);
      copiedMessage = "Failed to copy!";
    }
    document.body.removeChild(textarea);
  }
</script>

<div class="augment-page-container">
  <h1 class="page-title">Augment Your Bullet Points</h1>
  <p class="page-explanation">
    Paste your existing resume bullet points here. Our AI will suggest stronger,
    more impactful versions to help your achievements stand out to recruiters.
  </p>

  <div class="input-section">
    <div class="text-area-section">
      <textarea
        placeholder="Enter your bullet points here (one per line or as a paragraph)"
        bind:value={inputBulletPoints}
        on:input={handleInputChange}
        class="bullet-points-textarea"
        disabled={isLoading}
      ></textarea>
    </div>

    <button
      on:click={handleEnhanceBulletPoints}
      class="enhance-button"
      disabled={!inputBulletPoints.trim() || isLoading}
    >
      {#if isLoading}
        Enhancing...
      {:else}
        Enhance Bullet Point
      {/if}
    </button>
  </div>

  {#if isLoading}
    <div class="loading-indicator">
      <div class="spinner"></div>
      <p>Enhancing your bullet points...</p>
    </div>
  {/if}

  {#if errorMessage}
    <div class="error-message">
      <p>{errorMessage}</p>
    </div>
  {/if}

  {#if enhancedBulletPoints}
    <div class="results-section">
      <h2 class="results-title">Recommended Bullet Points</h2>
      {#if copiedMessage}
        <div class="copied-message">{copiedMessage}</div>
      {/if}
      <ul class="enhanced-list">
        {#each enhancedBulletPoints as bullet}
          <li class="enhanced-list-item">
            <span>{bullet}</span>

            <button
              class="copy-button"
              on:click={() => copyToClipboard(bullet)}
            >
              Copy
            </button>
          </li>
        {/each}
      </ul>
    </div>
  {/if}

  {#if showMessage}
    <div class="custom-message-box">
      <p>{messageText}</p>
    </div>
  {/if}
</div>

<style>
  .augment-page-container {
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

  .text-area-section {
    width: 100%;
  }

  .bullet-points-textarea {
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

  .bullet-points-textarea:focus {
    outline: none;
    border-color: #2563eb;
    box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.2);
  }

  .bullet-points-textarea:disabled {
    background-color: #f0f4f8;
    cursor: not-allowed;
  }

  .enhance-button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0.75rem 1.5rem;
    background-color: #2563eb;
    color: white;
    font-weight: 600;
    border: none;
    border-radius: 0.5rem;
    cursor: pointer;
    transition:
      background-color 0.3s ease,
      box-shadow 0.3s ease;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    width: 100%;
    font-size: 1rem;
  }

  .enhance-button:hover:not(:disabled) {
    background-color: #1d4ed8;
    box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
  }

  .enhance-button:disabled {
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

  .copied-message {
    text-align: center;
    color: #10b981;
    margin-bottom: 1rem;
    font-weight: 600;
  }

  .enhanced-list {
    list-style: none;
    padding-left: 0;
  }

  .enhanced-list-item {
    display: flex;
    align-items: flex-start;
    font-size: 1rem;
    color: #4a5568;
    margin-bottom: 0.5rem;
    position: relative;
    padding: 0.75rem 1rem;
    border-radius: 0.5rem;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
    word-break: break-word;
    background-color: #e2e8f0;
  }

  .enhanced-list-item span {
    flex-grow: 1;
    margin-left: 0.75rem;
    margin-right: 1rem;
    line-height: 1.6;
  }

  .copy-button {
    padding: 0.3rem 0.8rem;
    background-color: #10b981;
    color: white;
    border: none;
    border-radius: 0.3rem;
    cursor: pointer;
    font-size: 0.85rem;
    transition:
      background-color 0.3s ease,
      box-shadow 0.3s ease;
    flex-shrink: 0;
  }

  .copy-button:hover {
    background-color: #059669;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
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

    .bullet-points-textarea {
      min-height: 150px;
    }

    .results-title {
      font-size: 1.75rem;
    }

    .enhanced-list-item {
      flex-direction: column;
      align-items: flex-start;
      padding-left: 0.75rem;
      padding-right: 0.75rem;
    }

    .enhanced-list-item span {
      margin-right: 0;
      margin-bottom: 0.5rem;
    }

    .copy-button {
      width: 100%;
      font-size: 0.9rem;
      padding: 0.5rem;
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
