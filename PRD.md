Product Requirements Document (PRD) for SDLT
Overview
This product is a custom token system called Swift Deep Link Token (SDLT), designed as a JWT equivalent for iOS deep linking. It enables secure encoding, signing, and validation of parameters in deep link URLs for iOS apps using custom URL schemes. The primary use case is passing secure, tamper-proof data to apps via deep links, with support for merging multiple links into one while preserving validity. This addresses scenarios where multiple data sources need consolidation into a single deep link. The system supports environment flags (debug, production, testing) and basic bitwise operations on parameters (interpreted as merging integer/bit-flag parameters via OR during conflicts, with standard conflicts resolved by taking the “left” value).
Target platform: iOS (native Swift implementation). A JavaScript prototype is provided for demo and validation of logic before Swift porting.
Target Audience
	•	iOS developers building apps with deep link support.
	•	Backend teams generating deep links for mobile apps.
	•	Security-focused teams needing tamper-proof parameter passing.
Key Features
	•	Token encoding/decoding with secure parameter storage (JSON payload).
	•	Signing and validation using HMAC (default) for integrity.
	•	Environment flags (debug, production, testing) in token header.
	•	Merge function: Combine two deep links/tokens, preserving all parameters, resolving conflicts by taking left-side value (or bitwise OR for integer parameters).
	•	Basic bitwise operations: Support for OR/AND on integer parameters during merge or as utility functions.
Functional Requirements
	•	Token Creation: Input parameters (dict), environment flag, secret key; output signed token string embeddable in URL.
	•	Token Validation: Input token string, secret key; output decoded parameters if valid, else error.
	•	Merge: Input two deep link URLs or tokens, secret key; output new signed deep link with merged parameters. Merge logic: Union of keys; for conflicts, take left value (if integers, optional bitwise OR mode).
	•	Bitwise Utilities: Functions to perform OR/AND on specific integer parameters (e.g., for bit flags) before encoding or during merge.
	•	Deep Link Format: scheme://path?sdlt= (e.g., myapp://action?sdlt=header.payload.signature).
	•	Environment Handling: Header includes env field; validation can enforce matching expected env.
Non-Functional Requirements
	•	Security: Use HMAC-SHA256 by default; resistant to tampering. Assume shared secret (for demo); production could extend to asymmetric keys.
	•	Performance: Token operations < 100ms on typical iOS device.
	•	Compatibility: iOS 14+; Swift 5+.
	•	Error Handling: Clear errors for invalid signatures, malformed tokens, merge mismatches (e.g., different schemes).
	•	Extensibility: Design allows future alg support (e.g., RSA).
Assumptions
	•	Shared secret key is securely managed (not in code).
	•	Deep links use app-supported URL schemes.
	•	Parameters are simple types (strings, integers); no nested objects.
	•	Bitwise operations apply only to integer params; not enforced universally to avoid complexity.
	•	Merging requires signing key access (issuer-side tool).
	•	No internet dependency; all ops local.
Detailed Specification
Token Structure
Similar to JWT:
	•	Header: Base64-encoded JSON: { "alg": "HS256", "typ": "SDLT", "env": "debug" | "production" | "testing" }.
	•	Payload: Base64-encoded JSON: { "param1": "value", "param2": 42, ... }. Supports strings and integers.
	•	Signature: Base64-encoded HMAC-SHA256 of base64(header).base64(payload) using secret key.
	•	Full Token: base64(header).base64(payload).base64(signature) (URL-safe base64, no padding).
Encoding/Decoding
	•	Encode: Serialize header/payload to JSON, base64, concat with signature.
	•	Decode: Split by ‘.’, base64-decode, parse JSON.
Signing/Validation
	•	Sign: Compute HMAC-SHA256 on headerB64.payloadB64 with secret (UTF-8 bytes).
	•	Validate: Recompute signature; compare with provided. Also check typ and optional env match.
Merge Operation
	•	Input: Two deep link URLs (extract sdlt params), or direct tokens; signing secret.
	•	Steps:
	1	Validate both tokens (optional, but recommended).
	2	Decode headers and payloads.
	3	Check compatibility: Same alg and scheme; if env differs, use left’s.
	4	Merge payloads: Create new dict as union. For each key:
	▪	If only in one, include it.
	▪	If in both and both integers, perform bitwise OR (to support “basic bitwise operations”).
	▪	If in both but not both integers, take left’s value.
	5	Use left’s header for new token.
	6	Encode and sign new payload with secret.
	7	Output new deep link with updated sdlt.
	•	Example: Merge left {a:1, b:2} with right {b:4, c:3} → {a:1, b: (2 | 4)=6, c:3} (assuming integers for b).
Bitwise Operations
	•	Utility functions: orParams(value1: Int, value2: Int) → Int, andParams(value1: Int, value2: Int) → Int.
	•	Integrated into merge for integer conflicts (default OR).
	•	Not applied to strings; error or fallback to left if mixed types.
Swift Implementation Notes
	•	Use CryptoKit for HMAC-SHA256.
	•	JSONEncoder/Decoder for serialization.
	•	URLComponents for parsing/generating deep links.
	•	Struct: struct SDLT { func create(...), func validate(...), func merge(...) }.
JavaScript Functional Demo Page
Below is a complete, self-contained HTML page with JavaScript implementing the SDLT logic as a prototype. It demonstrates creation, validation, merging, and bitwise ops. Copy-paste into an HTML file and open in a browser. It uses CryptoJS for HMAC (included via CDN for demo; production would bundle).
The demo assumes a fixed scheme myapp://action and secret "mySecretKey". Adjust inputs to test.


    
    
    
    


    
Swift Deep Link Token (SDLT) JS Prototype Demo
    
    
        
Create Token
        Environment: debugproductiontesting
        Parameters (JSON): { "param1": "value", "flags": 1 }
        Secret: mySecretKey
        Generate
        
Deep Link: 
    
    
    
        
Validate Token
        Token: 
        Secret: mySecretKey
        Validate
        
Result: 
    
    
    
        
Merge Two Deep Links
        Left Deep Link: myapp://action?sdlt=eyAiYWxnIjogIkhTMjU2IiwgInR5cCI6ICJTRExUIiwgImVudiI6ICJkZWJ1ZyIgfQ==.eyAicGFyYW0xIjogInZhbHVlIiwgImZsYWdzIjogMX0=.signature
        Right Deep Link: myapp://action?sdlt=eyAiYWxnIjogIkhTMjU2IiwgInR5cCI6ICJTRExUIiwgImVudiI6ICJkZWJ1ZyIgfQ==.eyAiZmxhZ3MiOiAyLCAicGFyYW0yIjogM30=.signature
        Secret: mySecretKey
        Merge
        
Merged Deep Link: 
    
    
    


