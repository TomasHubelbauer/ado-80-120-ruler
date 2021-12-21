# Azure DevOps Wiki Editor Bookmarklet

Save these as a bookmarklets in your browser's bookmark bar by wrapping them in
`javascript:void function() { ... }` and making them single-line.

Use the below snippet in the browser DevTools console to automatically merge and
transform into a bookmarklet all of the snippets in this document.

```javascript
// Display in an alert instead of console to avoid escaping
alert(

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
  + '}()'

)
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

  // Combine into a single-item array to call forEach with the whole result
  .reduce((value, item, _index, array) => {
    const dupe = item.original !== item.index ? ` - DUPLICATE! of line ${array[item.original].line}` : '';
    value[0] += `- ${item.link} (line ${item.line})${dupe}\n`;
    return value;
  }, [''])

  // Call forEach on the combined result and present it to the user
  .forEach(result => result && alert('Unused reference links:\n' + result));
```
