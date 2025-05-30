// ==UserScript==
// @name         T3 Chat Custom Loader
// @namespace    http://tampermonkey.net/
// @version      1.6
// @description  Replace T3 Chat loader with custom Newton's cradle animation
// @match        https://t3.chat/*
// @match        https://beta.t3.chat/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function() {
  'use strict';

  // Prevent multiple initializations
  if (window.t3CustomLoaderInitialized) return;
  window.t3CustomLoaderInitialized = true;

  // --- Configuration ---
  const SELECTORS = {
    loader: 'div.flex.items-center.space-x-2',
    chatContainer:
      'div[role="log"][aria-label="Chat messages"]',
    reasoningButton:
      'button[aria-label="Show reasoning"]',
    assistantMessage:
      'div[role="article"][aria-label="Assistant message"]'
  };

  const CSS_CLASSES = {
    newtonsCradle: 'newtons-cradle',
    newtonsCradleDot: 'newtons-cradle__dot',
    hidden: 't3-loader-hidden',
    container: 't3-custom-loader-container',
    fadeIn: 't3-fade-in',
    fadeOut: 't3-fade-out'
  };

  // Track active loaders
  const activeLoaders = new Set();

  // --- Inject CSS for Newton's Cradle + Fade classes ---
  function injectStyles() {
    const styleId = 't3-custom-loader-css';
    if (document.getElementById(styleId)) return;

    const style = document.createElement('style');
    style.id = styleId;
    style.textContent = `
      .${CSS_CLASSES.hidden} {
        display: none !important;
      }

      .${CSS_CLASSES.container} {
        display: flex;
        align-items: center;
        justify-content: flex-start;
        margin: 0.25rem 0;
        opacity: 0;
        transition: opacity 0.25s ease-out;
      }

      .${CSS_CLASSES.container}.${CSS_CLASSES.fadeIn} {
        opacity: 1;
      }

      .${CSS_CLASSES.container}.${CSS_CLASSES.fadeOut} {
        opacity: 0;
        transition: opacity 0.4s ease-in;
      }

      .${CSS_CLASSES.newtonsCradle} {
        --uib-size: 32px;
        --uib-speed: 1.2s;
        --uib-color: #6b7280;
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        width: var(--uib-size);
        height: var(--uib-size);
      }

      .${CSS_CLASSES.newtonsCradleDot} {
        position: relative;
        display: flex;
        align-items: center;
        height: 100%;
        width: 25%;
        transform-origin: center top;
      }

      .${CSS_CLASSES.newtonsCradleDot}::after {
        content: '';
        display: block;
        width: 100%;
        height: 25%;
        border-radius: 50%;
        background-color: var(--uib-color);
      }

      .${CSS_CLASSES.newtonsCradleDot}:first-child {
        animation: swing var(--uib-speed) linear infinite;
      }

      .${CSS_CLASSES.newtonsCradleDot}:last-child {
        animation: swing2 var(--uib-speed) linear infinite;
      }

      @keyframes swing {
        0% {
          transform: rotate(0deg);
          animation-timing-function: ease-out;
        }
        25% {
          transform: rotate(70deg);
          animation-timing-function: ease-in;
        }
        50% {
          transform: rotate(0deg);
          animation-timing-function: linear;
        }
      }

      @keyframes swing2 {
        0% {
          transform: rotate(0deg);
          animation-timing-function: linear;
        }
        50% {
          transform: rotate(0deg);
          animation-timing-function: ease-out;
        }
        75% {
          transform: rotate(-70deg);
          animation-timing-function: ease-in;
        }
      }

      /* Slow-down and fade styling (unused for instant removal) */
      .${CSS_CLASSES.container}.${CSS_CLASSES.fadeOut}
        .${CSS_CLASSES.newtonsCradle} {
        --uib-speed: 2s;
      }

      .${CSS_CLASSES.container}.${CSS_CLASSES.fadeOut}
        .${CSS_CLASSES.newtonsCradleDot}::after {
        transition: background-color 0.3s ease;
        background-color: rgba(107, 114, 128, 0.5);
      }
    `;
    document.head.appendChild(style);
  }

  // --- Instantly remove a custom loader ---
  function removeCustomLoaderInstantly(spinner) {
    if (!spinner || !spinner.parentNode) return;
    spinner.parentNode.removeChild(spinner);
    activeLoaders.delete(spinner);
    console.log('Custom loader removed instantly');
  }

  // --- Detect the original bouncing-dot loader ---
  function isOriginalLoader(el) {
    if (!el.matches(SELECTORS.loader)) return false;
    const children = el.children;
    if (children.length !== 3) return false;
    for (let child of children) {
      if (
        !child.classList.contains('animate-bounce') ||
        !child.classList.contains('rounded-full') ||
        !child.classList.contains(
          'bg-secondary-foreground/40'
        )
      ) {
        return false;
      }
    }
    return true;
  }

  // --- Build the Newton's cradle DOM ---
  function createNewtonsCradle() {
    const container = document.createElement('div');
    container.className = CSS_CLASSES.container;
    const cradle = document.createElement('div');
    cradle.className = CSS_CLASSES.newtonsCradle;
    for (let i = 0; i < 4; i++) {
      const dot = document.createElement('div');
      dot.className = CSS_CLASSES.newtonsCradleDot;
      cradle.appendChild(dot);
    }
    container.appendChild(cradle);
    return container;
  }

  // --- On content arrival, nuke all loaders instantly ---
  function checkForContentAndRemoveLoaders() {
    const hasReason = Boolean(
      document.querySelector(SELECTORS.reasoningButton)
    );
    const hasAssist = Boolean(
      document.querySelector(SELECTORS.assistantMessage)
    );
    if (hasReason || hasAssist) {
      console.log(
        'Content detected, removing all active loaders immediately'
      );
      activeLoaders.forEach(loader =>
        removeCustomLoaderInstantly(loader)
      );
      activeLoaders.clear();
    }
  }

  // --- Handle a newly spotted original loader ---
  function handleLoader(origLoader) {
    if (origLoader.dataset.t3Processed) return;
    try {
      origLoader.dataset.t3Processed = 'true';
      origLoader.classList.add(CSS_CLASSES.hidden);

      const customSpinner = createNewtonsCradle();
      customSpinner.dataset.t3CustomLoader = 'true';
      origLoader.parentNode.insertBefore(
        customSpinner,
        origLoader.nextSibling
      );
      activeLoaders.add(customSpinner);
      requestAnimationFrame(() =>
        customSpinner.classList.add(CSS_CLASSES.fadeIn)
      );
      console.log(
        'Custom Newton\'s cradle loader added'
      );

      // If the original loader is ever removed, clean ours up:
      const parentObserver = new MutationObserver(muts => {
        muts.forEach(m => {
          m.removedNodes.forEach(node => {
            if (node === origLoader) {
              removeCustomLoaderInstantly(customSpinner);
              parentObserver.disconnect();
            }
          });
        });
      });
      if (origLoader.parentNode) {
        parentObserver.observe(origLoader.parentNode, {
          childList: true
        });
      }

      // Periodically check for content arrival
      const interval = setInterval(() => {
        checkForContentAndRemoveLoaders();
        if (!document.contains(customSpinner)) {
          clearInterval(interval);
        }
      }, 150);
      // safety stop after 30s
      setTimeout(() => clearInterval(interval), 30000);
    } catch (err) {
      console.warn('Failed to handle loader:', err);
    }
  }

  // --- Process any loaders already on the page ---
  function processExistingLoaders() {
    document.querySelectorAll(SELECTORS.loader).forEach(loader => {
      if (isOriginalLoader(loader) && !loader.dataset.t3Processed) {
        handleLoader(loader);
      }
    });
  }

  // --- Watch for content (reasoning button or assistant msg) ---
  function startContentWatcher() {
    const root =
      document.querySelector(SELECTORS.chatContainer) ||
      document.body;
    const obs = new MutationObserver(muts => {
      let flag = false;
      muts.forEach(m => {
        m.addedNodes.forEach(node => {
          if (node.nodeType !== Node.ELEMENT_NODE) return;
          if (
            (node.matches &&
              (node.matches(SELECTORS.reasoningButton) ||
               node.matches(SELECTORS.assistantMessage))) ||
            (node.querySelectorAll &&
              (node.querySelectorAll(
                SELECTORS.reasoningButton
              ).length ||
               node.querySelectorAll(
                 SELECTORS.assistantMessage
               ).length))
          ) {
            flag = true;
          }
        });
      });
      if (flag) setTimeout(checkForContentAndRemoveLoaders, 100);
    });
    obs.observe(root, { childList: true, subtree: true });
    console.log('Content watcher started');
    return obs;
  }

  // --- Watch for new original loaders appearing ---
  function startWatching() {
    const root =
      document.querySelector(SELECTORS.chatContainer) ||
      document.body;
    const obs = new MutationObserver(muts => {
      muts.forEach(m => {
        m.addedNodes.forEach(node => {
          if (node.nodeType !== Node.ELEMENT_NODE) return;
          if (
            isOriginalLoader(node) &&
            !node.dataset.t3Processed
          ) {
            handleLoader(node);
          }
          if (node.querySelectorAll) {
            node
              .querySelectorAll(SELECTORS.loader)
              .forEach(ld => {
                if (
                  isOriginalLoader(ld) &&
                  !ld.dataset.t3Processed
                ) {
                  handleLoader(ld);
                }
              });
          }
        });
      });
    });
    obs.observe(root, { childList: true, subtree: true });
    console.log('Loader replacement observer started');
    return obs;
  }

  // --- Initialize everything ---
  function init() {
    try {
      injectStyles();
      processExistingLoaders();
      startWatching();
      startContentWatcher();
      console.log(
        'T3 Chat Custom Newton\'s Cradle Loader initialized'
      );
    } catch (err) {
      console.error(
        'Failed to initialize custom loader script:',
        err
      );
    }
  }

  init();
})();
