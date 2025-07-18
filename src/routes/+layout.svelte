<script>
  import "../app.css";
  import { page } from "$app/state";
  let { children } = $props();

  function isActive(path) {
    return page.url.pathname === path;
  }
</script>

<div class="app-container">
  <header class="app-header">
    <nav class="nav-container">
      <div class="nav-logo">Resume AI</div>

      <input type="checkbox" id="menu-toggle" class="menu-toggle-checkbox" />
      <label for="menu-toggle" class="hamburger-icon">
        <span class="bar"></span>
        <span class="bar"></span>
        <span class="bar"></span>
      </label>

      <div class="nav-links-group">
        <a
          href="/grade"
          class="nav-link"
          aria-current={isActive("/grade") ? "page" : undefined}
        >
          Grade Resume
        </a>

        <a
          href="/compare"
          class="nav-link"
          aria-current={isActive("/compare") ? "page" : undefined}
        >
          Compare Resume
        </a>

        <a
          href="/augment"
          class="nav-link"
          aria-current={isActive("/augment") ? "page" : undefined}
        >
          Augment Bullet Points
        </a>
      </div>
    </nav>
  </header>

  <main class="main-content">
    {@render children()}
  </main>

  <footer class="app-footer">
    <p>&copy; 2025 Resume AI App. All rights reserved.</p>
  </footer>
</div>

<style>
  .app-container {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    background-color: #fdf8f0;
    color: #2d3748;
  }

  .app-header {
    background-color: transparent;
    padding: 1rem;
    box-shadow: none;
  }

  .nav-container {
    max-width: 1280px;
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: center;
    padding-top: 0.5rem;
    padding-bottom: 0.5rem;
    padding-left: 1rem;
    padding-right: 1rem;
    margin-left: auto;
    margin-right: auto;
  }

  .nav-links-group {
    display: flex;
    flex-wrap: wrap;
    gap: 1.5rem;
  }

  .nav-logo {
    color: #2d3748;
    font-size: 1.5rem;
    font-weight: 700;
    margin-right: 2rem;
    z-index: 20;
  }

  .hamburger-icon {
    display: none;
    flex-direction: column;
    justify-content: space-around;
    width: 30px;
    height: 25px;
    cursor: pointer;
    z-index: 20;
  }

  .hamburger-icon .bar {
    width: 100%;
    height: 3px;
    background-color: #2d3748;
    border-radius: 2px;
    transition: all 0.3s ease-in-out;
  }

  .menu-toggle-checkbox {
    display: none;
  }

  .nav-link {
    padding: 0.5rem 1rem;
    transition: all 0.3s ease;
    color: #2d3748; /* Set to black */
    font-weight: 500;
    text-decoration: none;
    border: 1px solid transparent;
    position: relative; /* For the underline effect */
    overflow: hidden; /* Hide the initial underline */
  }

  .nav-link::after {
    content: "";
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 3px;
    background-color: #2563eb; /* Blue underline */
    transform: translateY(100%); /* Start below the link */
    transition: transform 0.3s ease-out;
  }

  .nav-link:hover::after {
    transform: translateY(0); /* Move up on hover */
  }

  .nav-link:hover {
    background-color: transparent;
    box-shadow: none;
    border: 1px solid transparent;
    text-decoration: none; /* Ensure no default underline */
  }

  /* Removed .nav-link.active styling entirely */

  .main-content {
    flex-grow: 1;
    max-width: 1280px;
    width: 100%;
    margin-left: auto;
    margin-right: auto;
    padding: 1.5rem;
    background-color: #fdf8f0;
    box-shadow: none;
    margin-top: 4rem; /* Added margin-top to push content down */
    display: flex; /* Added for centering children */
    flex-direction: column; /* Added for centering children */
    align-items: center; /* Added to horizontally center children */
    text-align: center; /* Ensures text within also centers */
  }

  .app-footer {
    background-color: #fdf8f0;
    padding: 1rem;
    text-align: center;
    color: #4b5563;
    box-shadow: none;
    margin-top: auto;
  }

  @media (max-width: 768px) {
    .nav-container {
      padding-left: 1rem;
      padding-right: 1rem;
    }

    .hamburger-icon {
      display: flex;
    }

    .nav-links-group {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(253, 248, 240, 0.95);
      flex-direction: column;
      justify-content: center;
      align-items: center;
      transform: translateX(100%);
      transition: transform 0.3s ease-out;
      z-index: 10;
    }

    .menu-toggle-checkbox:checked ~ .nav-links-group {
      transform: translateX(0);
      display: flex;
    }

    .menu-toggle-checkbox:checked ~ .hamburger-icon .bar:nth-child(1) {
      transform: translateY(11px) rotate(45deg);
    }
    .menu-toggle-checkbox:checked ~ .hamburger-icon .bar:nth-child(2) {
      opacity: 0;
    }
    .menu-toggle-checkbox:checked ~ .hamburger-icon .bar:nth-child(3) {
      transform: translateY(-11px) rotate(-45deg);
    }

    .nav-link {
      font-size: 1.5rem;
      padding: 1rem 2rem;
      width: 80%;
      text-align: center;
      margin-bottom: 1rem;
      color: #2d3748;
    }

    .nav-link::after {
      height: 2px; /* Thinner line for mobile */
    }
  }
</style>
