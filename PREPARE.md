# Evaluation Setup

This file is outside the editable surface. It defines how results are judged. Agents cannot modify the evaluator or the scoring logic — the evaluation is the trust boundary.

Consider defining more than one evaluation criterion. Optimizing for a single number makes it easy to overfit and silently break other things. A secondary metric or sanity check helps keep the process honest.

eval_cores: 1
eval_memory_gb: 1.0
prereq_command:

## Setup

Install dependencies and prepare the evaluation environment:

```bash
npm install
```

This project is ESM-based JavaScript with no build step. The `index.js` file is used directly.

## Run command

```bash
node -e "
import escapeStringRegexp from './index.js';

const testStrings = [
  'How much \$ for a 🦄?',
  '\\\\ ^ \$ * + ? . ( ) | { } [ ]',
  'foo - bar',
  'test-string-with-many-dashes-and-special-chars-like-\$-and-*-and-?',
  'a'.repeat(100) + '\$' + 'b'.repeat(100),
  'simple string',
  '!!!@@@###\$\$\$%%%^^^&&&***((()))',
];

const iterations = 100000;
const start = Date.now();

for (let i = 0; i < iterations; i++) {
  for (const str of testStrings) {
    escapeStringRegexp(str);
  }
}

const elapsed = (Date.now() - start) / 1000;
const opsPerSec = Math.round((iterations * testStrings.length) / elapsed);

console.log(\`METRIC=\${opsPerSec}\`);
"
```

## Output format

The benchmark must print `METRIC=<number>` to stdout.

## Metric parsing

The CLI looks for `METRIC=<number>` or `ops_per_sec=<number>` in the output.

## Ground truth

The metric represents operations per second when escaping a variety of test strings including:
- Strings with special RegExp characters (\, ^, $, *, +, ?, ., (, ), |, {, }, [, ])
- Strings with hyphens (which require special handling for Unicode compatibility)
- Long strings with repeated patterns
- Simple strings without special characters
- Mixed special character strings

The benchmark runs 100,000 iterations over 7 diverse test strings (700,000 total operations) and reports the throughput in operations per second. Higher values indicate better performance.
