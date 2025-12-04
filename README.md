# Simplicity Editor Support

Syntax highlighting, code completion, and IDE integration for the **Simplicity** smart contract language (Simfony dialect).

Simplicity is a low-level programming language for Bitcoin and Liquid smart contracts, designed by Blockstream. This repository provides editor support for both **CodeMirror** and **Monaco Editor**.

## Features

✨ **Syntax Highlighting**
- Keyword highlighting (fn, let, match, if, else, etc.)
- Type highlighting (u8, u32, Pubkey, Signature, etc.)
- Function definitions and calls
- Jet operations (jet::add_32, jet::sig_all_hash, etc.)
- Witness references (witness::PK, witness::SIG, etc.)
- Comments, strings, and numbers

🔧 **Code Completion (Monaco only)**
- IntelliSense for all Simplicity jets (100+ functions)
- Type suggestions
- Keyword completion
- Code snippets (fn main, assert!, witness declarations)

📖 **Hover Information (Monaco only)**
- Function signatures and descriptions
- Type information for jets

🎨 **Theme**
- VS Code Dark+ compatible theme
- Carefully matched colors for optimal readability

## Quick Start

### CodeMirror Integration

1. Include CodeMirror and the Simplicity mode:

```html
<!-- CodeMirror core -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.2/codemirror.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.2/codemirror.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.2/addon/mode/simple.min.js"></script>

<!-- Simplicity mode and theme -->
<link rel="stylesheet" href="codemirror/themes/simplicity-theme.css">
<script src="codemirror/modes/simplicityhl.js"></script>
```

2. Initialize the editor:

```html
<textarea id="code-editor"></textarea>

<script>
const editor = CodeMirror.fromTextArea(document.getElementById('code-editor'), {
    mode: 'simplicityhl',
    theme: 'simplicity',
    lineNumbers: true,
    matchBrackets: true,
    autoCloseBrackets: true
});

// Load example code
editor.setValue(`fn main() {
    let pk: Pubkey = witness::PK;
    let msg: u256 = jet::sig_all_hash();
    jet::bip_0340_verify((pk, msg), witness::SIG)
}`);
</script>
```

### Monaco Editor Integration

1. Create an HTML file:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Simplicity Editor</title>
    <style>
        #monaco-editor {
            width: 100%;
            height: 600px;
            border: 1px solid #ccc;
        }
    </style>
</head>
<body>
    <div id="monaco-editor"></div>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.3.6/require.min.js"></script>
    <script src="monaco/monaco-simplicityhl.js"></script>
    <script>
        // Initialize Monaco editor with Simplicity support
        const simplicityEditor = new MonacoSimplicityConfig();
        
        simplicityEditor.init('monaco-editor').then(editor => {
            console.log('Editor initialized!');
            
            // Load example code
            editor.setValue(`fn main() {
    let pk: Pubkey = witness::PK;
    let msg: u256 = jet::sig_all_hash();
    jet::bip_0340_verify((pk, msg), witness::SIG)
}`);
        });
    </script>
</body>
</html>
```

## File Structure

```
simplicity-editor-support/
├── codemirror/
│   ├── modes/
│   │   └── simplicityhl.js          # CodeMirror syntax mode
│   └── themes/
│       └── simplicity-theme.css     # Dark theme for CodeMirror
├── monaco/
│   └── monaco-simplicityhl.js       # Monaco language support
├── data/
│   └── jets-data.js                 # Comprehensive jet definitions
├── LICENSE
└── README.md
```

## Language Features

### Syntax Elements

#### Keywords
- **Control flow**: `fn`, `let`, `match`, `if`, `else`, `return`
- **Primitives**: `true`, `false`, `None`, `Some`, `Left`, `Right`

#### Types
- **Integers**: `u8`, `u16`, `u32`, `u64`, `u128`, `u256`
- **Crypto**: `Pubkey`, `Signature`, `Scalar`, `Point`, `FE`, `GE`, `GEJ`
- **Generic**: `Either`, `Option`, `bool`
- **Bitcoin**: `Height`, `Time`, `Distance`, `Duration`, `Lock`, `Outpoint`

#### Jets (Built-in Functions)

Jets are optimized built-in functions for common operations. They're accessed via the `jet::` namespace:

**Arithmetic**: `add_32`, `subtract_64`, `multiply_32`, `div_mod_64`

**Bitwise**: `and_32`, `or_32`, `xor_32`, `left_shift_32`

**Comparison**: `eq_32`, `eq_256`, `lt_32`, `is_zero_32`

**Hashing**: `sha_256`, `sha_256_ctx_8_init`, `sha_256_ctx_8_finalize`

**Crypto**: `bip_0340_verify`, `check_sig_verify`, `fe_normalize`

**Transaction**: `sig_all_hash`, `current_value`, `num_inputs`, `check_lock_height`

See `data/jets-data.js` for the complete list with type signatures and descriptions.

#### Witnesses

Witnesses are runtime values provided when spending:

```javascript
let pk: Pubkey = witness::PK;
let sig: Signature = witness::SIG;
let preimage: u256 = witness::PREIMAGE;
```

### Example: Pay-to-Public-Key-Hash (P2PKH)

```javascript
fn sha2(string: u256) -> u256 {
    let hasher: Ctx8 = jet::sha_256_ctx_8_init();
    let hasher: Ctx8 = jet::sha_256_ctx_8_add_32(hasher, string);
    jet::sha_256_ctx_8_finalize(hasher)
}

fn main() {
    let pk: Pubkey = witness::PK;
    let expected_pk_hash: u256 = 0x132f39a98c31baaddba6525f5d43f2954472097fa15265f45130bfdb70e51def;
    let pk_hash: u256 = sha2(pk);
    assert!(jet::eq_256(pk_hash, expected_pk_hash));

    let msg: u256 = jet::sig_all_hash();
    jet::bip_0340_verify((pk, msg), witness::SIG)
}
```

### Example: Hash Time-Locked Contract (HTLC)

```javascript
fn main() {
    match witness::COMPLETE_OR_CANCEL {
        Left(preimage_sig: (u256, Signature)) => {
            let (preimage, recipient_sig): (u256, Signature) = preimage_sig;
            // Verify preimage matches hash
            let hash: u256 = sha2(preimage);
            assert!(jet::eq_256(hash, expected_hash));
            // Verify recipient signature
            checksig(recipient_pk, recipient_sig);
        },
        Right(sender_sig: Signature) => {
            // After timeout, sender can cancel
            jet::check_lock_height(timeout);
            checksig(sender_pk, sender_sig)
        }
    }
}
```

## Monaco Editor API

### Initialization

```javascript
const config = new MonacoSimplicityConfig();
const editor = await config.init('container-id', initialCode);
```

### Methods

- `getValue()` - Get current editor content
- `setValue(code)` - Set editor content
- `formatDocument()` - Format the code

### Customization

The Monaco configuration can be customized by modifying the editor options in `init()`:

```javascript
this.editor = monaco.editor.create(container, {
    value: code,
    language: 'simplicityhl',
    theme: 'simplicity-dark',
    fontSize: 14,
    minimap: { enabled: true },
    // Add more options here
});
```

## Jets Reference

The `data/jets-data.js` file contains a comprehensive list of all Simplicity jets organized by category:

- **Core Jets**: Arithmetic, bitwise, comparison, shifts, constants
- **Elements Jets**: Transaction inspection for Liquid Network
- **Bitcoin Jets**: Transaction inspection for Bitcoin

Each jet includes:
- **Name**: Function name
- **Type signature**: Input and output types
- **Description**: What the function does

## CodeMirror Mode Details

The CodeMirror mode (`codemirror/modes/simplicityhl.js`) uses the Simple Mode API and provides:

1. **Token Classification**:
   - Keywords (fn, let, match)
   - Types (u32, Pubkey)
   - Function definitions
   - Function calls
   - Jet operations
   - Witness references
   - Comments, strings, numbers

2. **Features**:
   - Line and block comments
   - Automatic bracket matching
   - Code folding
   - Auto-closing brackets and quotes

3. **MIME Types**:
   - `text/x-simplicityhl`
   - `text/x-simfony`
   - `text/x-simf`

## Monaco Language Details

The Monaco language support (`monaco/monaco-simplicityhl.js`) provides:

1. **Tokenization**: Monarch-based lexer for syntax highlighting
2. **Completion Provider**: IntelliSense for jets, types, and keywords
3. **Hover Provider**: Type signatures and documentation on hover
4. **Theme**: Custom dark theme matching VS Code Dark+

## Browser Compatibility

- **CodeMirror**: All modern browsers (Chrome, Firefox, Safari, Edge)
- **Monaco**: Chrome, Firefox, Safari, Edge (requires modern JavaScript features)

## Contributing

Contributions are welcome! Areas for improvement:

- Add more code snippets
- Improve error detection
- Add signature help (parameter hints)
- Support for more advanced language features

## Resources

- [Simplicity Language Specification](https://github.com/BlockstreamResearch/simplicity)
- [Simfony Documentation](https://github.com/BlockstreamResearch/simfony)
- [CodeMirror Documentation](https://codemirror.net/5/doc/manual.html)
- [Monaco Editor Documentation](https://microsoft.github.io/monaco-editor/)

## License

MIT License - See LICENSE file for details

## Credits

Created for the Simplicity smart contract language by Blockstream Research.

Based on the official [SimplicityHL VSCode extension](https://marketplace.visualstudio.com/items?itemName=Blockstream.simplicityhl).

