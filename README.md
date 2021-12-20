# Azure DevOps 80-120 Characters Wiki Editor Ruler

Save this as a bookmarklet in your browser's bookmark bar. Get the single-line
version by running the below script in the browser DevTools console:

```javascript
document.querySelector('.highlight:last-child').textContent.replace(/\n\s*/g, '')
```

```javascript
javascript:
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
