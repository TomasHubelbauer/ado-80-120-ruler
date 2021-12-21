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

## Unused Reference Links

```javascript
// Detect MarkDown reference links with no usages and show a summary

// Find all MarkDown reference link definitions in the ADO wiki editor
[...document.querySelector('textarea').value.matchAll(/(^|\n)(?<outer>\[(?<inner>.+)\]):/g)]

  // Keep the ones which do not have any usage in the MarkDown document
  .filter(match => match.input.indexOf(']' + match.groups.outer) === -1)

  // Expand matches with line numer and duplicate detection result
  .map((match, index, array) => ({
    link: match.groups.inner,
    line: [...match.input.slice(0, match.index).matchAll(/\n/g)].length,
    original: array.findIndex(item => item.groups.inner === match.groups.inner),
    index
  }))
  
  .map(item => {
    const dupe = item.original !== item.index ? ` - DUPLICATE! of line ${array[item.original].line}` : '';
    
    const li = document.createElement('li');
    li.textContent = `${item.link} (line ${item.line})${dupe}`;
    return li;
  })

  // Prepend the header for the UI section before the actual list items
  .reduce((accumulator, current) => [...accumulator, current], [document.createElement('br'), 'Unused reference links:'])

  // Insert the header and the individual list items to the hijacked UI area
  .forEach(item => presentUi(item));
```
