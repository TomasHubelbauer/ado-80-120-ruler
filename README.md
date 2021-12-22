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

### Make the UI thing not be a hijack but instead a tabbed UI for render/tool

Introduce tab buttons atop the MarkDown render or in the tool bar and make
clicking them change the right pane between MarkDown render and the tool UI.

```javascript
function makeTab(name, icon, contentDiv) {
  const div = document.createElement('div');
  div.className = 'markdowntoolbar-button';
  div.dataset.bookmarkletTab = name;

  const button = document.createElement('button');
  button.className = 'ms-CommandBarItem-link';
  button.addEventListener('click', () => selectTab(name));

  // https://fontdrop.info AzDevMDL2.woff https://docs.microsoft.com/en-us/windows/apps/design/style/segoe-ui-symbol-font#icon-list
  const i = document.createElement('i');
  i.className = 'ms-CommandBarItem-icon root-41';
  i.textContent = icon;

  const span = document.createElement('span');
  span.style = 'position: relative; top: -1ex;';

  button.append(i, span);
  div.append(button);
  document.querySelector('.ms-CommandBar-sideCommands').insertAdjacentElement('afterbegin', div);

  if (!contentDiv) {
    contentDiv = document.createElement('div');
    contentDiv.style.display = 'none';
    contentDiv.textContent = name;
    document.querySelector('.markdown-preview').insertAdjacentElement('beforebegin', contentDiv);
  }

  contentDiv.dataset.bookmarkletTabContent = name;
}

function selectTab(name) {
  const tabContents = document.querySelectorAll('[data-bookmarklet-tab-content]');
  for (const tabContent of tabContents) {
    tabContent.style.display = 'none';
  }

  document.querySelector(`[data-bookmarklet-tab-content="${name}"]`).style.display = 'initial';

  const tabs = document.querySelectorAll('[data-bookmarklet-tab]');
  for (const tab of tabs) {
    tab.classList.toggle('is_checked', false);
  }

  document.querySelector(`[data-bookmarklet-tab="${name}"]`).classList.toggle('is_checked', true);
}

function updateTabIcon(name, icon, badge) {
  document.querySelector(`[data-bookmarklet-tab="${name}"] i`).textContent = icon;
  document.querySelector(`[data-bookmarklet-tab="${name}"] span`).textContent = badge;
}

function updateTab(name, ...contents) {
  document.querySelector(`[data-bookmarklet-tab-content="${name}"]`).innerHTML = '';
  document.querySelector(`[data-bookmarklet-tab-content="${name}"]`).append(...contents);
}

makeTab('links', '\uF302'); // F302 all used/F303 some unused + badge
updateTabIcon('links', '\uF303', 2);
updateTab('links', 'this is the tab for links');

makeTab('todos', '\uE73A'); // E73A all done/E762 some undone + badge
updateTabIcon('todos', '\uE762', 3);
updateTab('todos', 'this is the tab for todos');

makeTab('render', '\uE8A1', document.querySelector('.markdown-preview'));
```

### Update link summary title to include the number of unused links if any

Sort by usage count so unused appear first if there are any, if not, sort by
line number (the default).
