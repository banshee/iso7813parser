# @banshee.com/iso7813Parser

A TypeScript library for parsing [ISO/IEC 7813](https://en.wikipedia.org/wiki/ISO/IEC_7813) magnetic stripe card data, built with [Chevrotain](https://chevrotain.io/).

Supports **Track 1** (IATA, alphanumeric) and **Track 2** (ABA, numeric) formats.

## Installation

```bash
pnpm add @banshee.com/iso7813Parser
```

## Usage

```typescript
import { parse, parseTrack1, parseTrack2 } from "@banshee.com/iso7813Parser";

// Auto-detect track type from the input
const result = parse("%B4111111111111111^DOE/JOHN^2512101?");

if (result.data?.track === 1) {
  console.log(result.data.pan);            // "4111111111111111"
  console.log(result.data.name.surname);   // "DOE"
  console.log(result.data.name.givenName); // "JOHN"
  console.log(result.data.formatCode);     // "B"
}

// Or parse a specific track directly
const t2 = parseTrack2(";4111111111111111=2512101?");
console.log(t2.data?.pan);                     // "4111111111111111"
console.log(t2.data?.expirationDate?.month);   // 12
console.log(t2.data?.serviceCode);             // "101"
```

## API

### `parse(input: string): ParseResult<Track1Data | Track2Data>`

Auto-detects the track format by inspecting the first character (`%` → Track 1, `;` → Track 2) and parses accordingly. Throws if the input is empty or starts with an unrecognized character.

### `parseTrack1(input: string): ParseResult<Track1Data>`

Parses ISO/IEC 7813 Track 1 (IATA) magnetic stripe data.

### `parseTrack2(input: string): ParseResult<Track2Data>`

Parses ISO/IEC 7813 Track 2 (ABA) magnetic stripe data.

### `ParseResult<T>`

```typescript
interface ParseResult<T> {
  data?: T;              // Parsed data (undefined if parsing failed)
  lexErrors: ILexingError[];        // Lexer errors
  parseErrors: IRecognitionException[]; // Parser errors
}
```

### `Track1Data`

```typescript
interface Track1Data {
  track: 1;
  formatCode: string;        // Typically "B" for banking
  pan: string;               // Primary Account Number (up to 19 digits)
  name: CardholderName;      // Parsed cardholder name
  expirationDate?: ExpirationDate;
  serviceCode?: string;      // 3-digit service code
  discretionaryData?: string;
  lrc?: string;              // Longitudinal Redundancy Check
}
```

### `Track2Data`

```typescript
interface Track2Data {
  track: 2;
  pan: string;
  expirationDate?: ExpirationDate;
  serviceCode?: string;
  discretionaryData?: string;
  lrc?: string;
}
```

### `CardholderName`

```typescript
interface CardholderName {
  raw: string;          // Raw name as encoded (e.g. "DOE/JOHN.MR")
  surname: string;
  givenName?: string;
  title?: string;       // e.g. "MR", "DR"
}
```

### `ExpirationDate`

```typescript
interface ExpirationDate {
  raw: string;   // 4-digit YYMM string
  year: number;  // Two-digit year
  month: number; // 1–12
}
```

## Track Formats

### Track 1 (IATA)

```
%B4111111111111111^DOE/JOHN^25121011234?
│ │               │        │    │  │   │
SS FC  PAN        FS Name  FS ED SC DD ES
```

- Up to 79 alphanumeric characters
- Start sentinel: `%`, End sentinel: `?`, Field separator: `^`

### Track 2 (ABA)

```
;4111111111111111=25121011234?
│               │    │  │   │
SS    PAN       FS ED SC DD ES
```

- Up to 40 numeric characters
- Start sentinel: `;`, End sentinel: `?`, Field separator: `=`

## License

Apache 2.0
