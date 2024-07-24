# vscode
## user snippets markdown

```json
1. settings.json
    "[markdown]": {
        "editor.unicodeHighlight.ambiguousCharacters": false,
        "editor.unicodeHighlight.invisibleCharacters": false,
        "diffEditor.ignoreTrimWhitespace": false,
        "editor.wordWrap": "on",
        // "editor.quickSuggestions":true
        "editor.quickSuggestions": {
            "comments": "true",
            "strings": "true",
            "other": "true"
            
        }
    }
2. markdown.json
	"Print to ```js": {
		"prefix": "```js",
		"body": [
			"```js",
			"$1",
			"$2",
			"```",
		],
		"description": "js代码片段"
	},
```