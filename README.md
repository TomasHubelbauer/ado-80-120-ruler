# Azure DevOps Wiki Editor Bookmarklet

Save this as a bookmarklet in your browser's bookmark bar. Get the single-line
version by running the below script in the browser DevTools console:

```javascript
document.querySelector('.highlight:last-child').textContent.replace(/\n\s*/g, '')
```

## 80 & 120 Character Rulers

```javascript
/* Display lines for 0, 80 and 120 character columns */

void function() {
  document.querySelector('textarea').style.background = `
    linear-gradient(to right,
      silver 1px, transparent 0,
      
      transparent 80ch,
      silver calc(80ch + 1px), transparent 0,
      
      transparent 120ch,
      silver calc(120ch + 1px), transparent 0
    )
  `
}()
```

## Unused Reference Links

```javascript
/* Detect MarkDown reference links with no usages and show a summary */

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
  .forEach(result => result && alert('Unused reference links:\n' + result))
```
