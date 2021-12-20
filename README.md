# Azure DevOps 80-120 Characters Wiki Editor Ruler

Save this as a bookmarklet in your browser's bookmark bar (remove line breaks):

```javascript
javascript:
void function() {
  document.querySelector('textarea').style.background = `
    linear-gradient(
      to right,
      silver 1px, transparent 0, transparent 80ch,
      silver calc(80ch + 1px), transparent 0, transparent 120ch,
      silver calc(120ch + 1px), transparent 0)`
}()
```
