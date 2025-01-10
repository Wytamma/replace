<script lang="ts">
    import { onMount } from "svelte";
  
    export let columns: string[] = [];
    export let contenteditableDiv: HTMLElement | null = null;

    

    let currentRange: Range | null = null;
  
    // Function to save the current cursor position
    function saveCursorPosition() {
      const selection = window.getSelection();
      if (!selection || selection.rangeCount === 0) return;
  
      const range = selection.getRangeAt(0);
  
      // Ensure the range is inside the contenteditableDiv
      if (contenteditableDiv && contenteditableDiv.contains(range.commonAncestorContainer)) {
        // Ensure the cursor is not inside a chip
        let container = range.commonAncestorContainer as HTMLElement;
        if (container.nodeType === Node.TEXT_NODE) {
          container = container.parentElement!;
        }
  
        if (container && container.classList.contains("chip")) {
          currentRange = null; // Invalidate the range if inside a chip
        } else {
          currentRange = range;
        }
      }
    }
  
    // Function to restore the saved cursor position
    function restoreCursorPosition() {
      const selection = window.getSelection();
      if (selection && currentRange) {
        selection.removeAllRanges();
        selection.addRange(currentRange);
      }
    }
  
    // Function to insert chip HTML at the current cursor position
    function insertAtCursor(html: string) {
      if (!contenteditableDiv) return;
  
      if (currentRange) {
        const tempDiv = document.createElement("div");
        tempDiv.innerHTML = html;
        const fragment = document.createDocumentFragment();
  
        while (tempDiv.firstChild) {
          fragment.appendChild(tempDiv.firstChild);
        }
  
        currentRange.deleteContents();
        currentRange.insertNode(fragment);
  
        // Move the cursor to the end of the inserted content
        currentRange.collapse(false);
        restoreCursorPosition();
      } else {
        // Fallback to appending at the end if no valid cursor position is saved
        const tempDiv = document.createElement("div");
        tempDiv.innerHTML = html;
  
        while (tempDiv.firstChild) {
          contenteditableDiv.appendChild(tempDiv.firstChild);
        }
      }
  
      contenteditableDiv.focus(); // Ensure the input div is focused
    }
  
    // Handle keydown events to save the cursor position
    function handleKeydown(event: KeyboardEvent) {
      if (!contenteditableDiv) return;
  
      const selection = window.getSelection();
      if (!selection || !selection.anchorNode) return;
  
      const node = selection.anchorNode.parentElement;
  
      // Check if the current node is a chip and handle deletion
      if (
        node &&
        node.classList.contains("chip") &&
        (event.key === "Backspace" || event.key === "Delete")
      ) {
        event.preventDefault();
        node.remove(); // Remove the entire chip
      }
    }
  
    // Add event listeners to chips
    function initializeChipListeners() {
      const chips = document.querySelectorAll(".chip");
  
      chips.forEach((chip) => {
        chip.addEventListener("click", () => {
          const chipHTML = `<span class="chip variant-filled py-1" contenteditable="false">${chip.textContent}</span>`;
          insertAtCursor(chipHTML);
        });
      });
    }
  
    // Initialize chip listeners and attach event handlers
    onMount(() => {
      initializeChipListeners();
  
      if (contenteditableDiv) {
        contenteditableDiv.addEventListener("keydown", handleKeydown);
        contenteditableDiv.addEventListener("mouseup", saveCursorPosition);
        contenteditableDiv.addEventListener("keyup", saveCursorPosition);
        contenteditableDiv.addEventListener("click", saveCursorPosition);
      }
    });
  </script>
  
  <div class="flex flex-col space-y-2 w-full min-w-64">
    <div class="label">
        <span >With</span>
        <div
        class="w-full border-b-2 border-gray-300 p-2 min-h-10 bg-surface-700"
        contenteditable="true"
        bind:this={contenteditableDiv}
      ></div>
    </div> 
    <div class="flex flex-wrap">
        {#each columns as column}
          <span class="chip variant-filled hover:variant-soft-surface mr-1 mb-1">{column}</span>
        {/each}
      </div>
  </div>
  
  <style>

    .chip[contenteditable="false"] {
      user-select: none; /* Prevent selection */
    }
  </style>
  