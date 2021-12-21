# Azure DevOps Wiki Editor Bookmarklet

Save these as a bookmarklets in your browser's bookmark bar by wrapping them in
`javascript:void function() { ... }` and making them single-line.

Use the below snippet in the browser DevTools console to automatically merge and
transform into a bookmarklet all of the snippets in this document.

```javascript
// Use this to avoid trimming or escaping in console, alert, prompt etc. and document-not-focused with Clipboard API
document.querySelector('.highlight pre').textContent =

  // Introduce the void function to self-invoke the bookmark
  'javascript:void function() {' +

  // Collect and transform the snippets' contents
  [...document.querySelectorAll('.highlight')]
    // Skip the fenced code block for this script itself
    .slice(1)

    // Extract the individual snippets' text content
    .map(div => div.textContent)

    // Join the text contents into a single script
    .join('')

    // Transform all comments to be single-line friendly
    .replace(/(^|\n\s*)\/\/\s?(?<comment>.+)(\n|$)/g, '/* $<comment> */')

    // Replace newlines and whitespace to make single-line bookmarklet
    .replace(/\n\s*/g, '')

    // Close the void function and make it self-call
    + '}()';

document.querySelector('.highlight clipboard-copy').value = document.querySelector('.highlight pre').textContent;
```

## 80 & 120 Character Rulers

```javascript
// Display lines for 0, 80 and 120 character columns

document.querySelector('textarea').style.background = `
  linear-gradient(to right,
    silver 1px, transparent 0,

    transparent 80ch,
    silver calc(80ch + 1px), transparent 0,

    transparent 120ch,
    silver calc(120ch + 1px), transparent 0
  )
`;
```

### MarkDown Render Area UI Hijack

```javascript
// Hijack the MarkDown preview render area to display custom UI by other scripts

const markdownRenderArea = document.querySelector('.markdown-render-area');
const textarea = document.querySelector('textarea');

const button = document.createElement('button');
button.className = 'bolt-button';
button.dataset.adoBookmarklet = true;
button.textContent = 'Reset';
button.onclick = () => {
  const text = textarea.value;
  textarea.select();
  document.execCommand('insertHTML', false, '');
  document.execCommand('insertHTML', false, text);
};

function presentUi(...content) {
  if (markdownRenderArea.firstChild?.dataset.adoBookmarklet !== 'true') {
    markdownRenderArea.innerHTML = '';
    markdownRenderArea.append(button);
  }

  markdownRenderArea.append(...content);
}

if (markdownRenderArea.firstChild?.dataset.adoBookmarklet === 'true') {
  button.click();
}
```

## Reference Links Summary

```javascript
// Summarize MarkDown reference links with their usages and duplicate detection

// Find all MarkDown reference link definitions in the ADO wiki editor
[...document.querySelector('textarea').value.matchAll(/(^|\n)(?<outer>\[(?<inner>.+)\]):/g)]

  // Tranform to include line numbers, usages and duplicate information
  .map((match, index, array) => {
    const link = match.groups.inner;
    const line = [...match.input.slice(0, match.index).matchAll(/\n/g)].length;
    const original = array.findIndex(item => item.groups.inner === link);
    const dupe = original !== index ? original : undefined;

    // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions#escaping
    const regex = link.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    const usages = [...match.input.matchAll(new RegExp(`([^\n^]\\[${regex}\\]|[\n^]\\[${regex}\\][^:]|\\]\\[${regex}\\])`, 'g'))].map(match => {
      const text = match[0];
      const line = [...match.input.slice(0, match.index).matchAll(/\n/g)].length;
      return { text, line };
    });

    return { link, line, dupe, usages };
  })

  // Transform to an HTML li element to present in the UI
  .map((link, index, array) => {
    const li = document.createElement('li');
    li.textContent = `${link.link} (${link.line})`;

    if (link.dupe !== undefined) {
      li.textContent += ` | duplicate (${array[link.dupe].line})`;
    }

    if (link.usages.length === 0) {
      li.textContent += ' | unused';
    }
    else {
      li.textContent += ` | used ${link.usages.length}x (${link.usages.map(usage => usage.line).join(', ')})`;
    }

    return li;
  })

  // Prepend the header for the UI section before the actual list items
  .reduce((accumulator, current) => [...accumulator, current], [document.createElement('br'), 'Links:'])

  // Insert the header and the individual list items to the hijacked UI area
  .forEach(item => presentUi(item));
```

## To-Do

### Add a script for summarizing to-do items in the document

### Add a script for adorning the textarea with line numbers
