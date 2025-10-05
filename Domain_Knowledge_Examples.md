# Domain Knowledge Examples: Project-Specific Learning

## Piano Application Domain Knowledge

### Technical Learnings

#### API and Library Decisions
```json
{
  "learnings": [
    {
      "id": "piano_001",
      "type": "technical",
      "context": "staff notation rendering",
      "solution": "Use MusicXML API instead of SVG",
      "reasoning": "SVG approach caused performance issues with complex note positioning and was difficult to maintain. MusicXML API provides better structure and performance.",
      "confidence": 0.95,
      "frequency": 8,
      "lastUsed": "2024-01-15T10:30:00Z",
      "source": "correction",
      "alternatives": ["SVG rendering", "Canvas-based rendering"],
      "performance_impact": "High - 60% faster rendering"
    },
    {
      "id": "piano_002",
      "type": "technical",
      "context": "audio playback for piano sounds",
      "solution": "Use Web Audio API with preloaded samples",
      "reasoning": "HTML5 Audio API had high latency (200ms+) and poor timing control. Web Audio API provides <50ms latency and better polyphony control.",
      "confidence": 0.90,
      "frequency": 6,
      "lastUsed": "2024-01-14T15:45:00Z",
      "source": "correction",
      "alternatives": ["HTML5 Audio API", "Howler.js"],
      "performance_impact": "High - 75% latency reduction"
    },
    {
      "id": "piano_003",
      "type": "technical",
      "context": "note positioning on staff",
      "solution": "Use absolute positioning with calculated coordinates",
      "reasoning": "Flexbox and CSS Grid caused alignment issues with different note types. Absolute positioning with calculated coordinates provides pixel-perfect alignment.",
      "confidence": 0.85,
      "frequency": 4,
      "lastUsed": "2024-01-13T09:20:00Z",
      "source": "correction",
      "alternatives": ["Flexbox", "CSS Grid", "SVG positioning"]
    }
  ]
}
```

#### Performance Optimizations
```json
{
  "learnings": [
    {
      "id": "piano_004",
      "type": "performance",
      "context": "real-time audio processing",
      "solution": "Use Web Workers for audio processing",
      "reasoning": "Main thread audio processing caused UI stuttering. Web Workers keep audio processing off the main thread.",
      "confidence": 0.88,
      "frequency": 3,
      "lastUsed": "2024-01-12T14:10:00Z",
      "source": "correction",
      "performance_impact": "High - Eliminated UI stuttering"
    },
    {
      "id": "piano_005",
      "type": "performance",
      "context": "note rendering performance",
      "solution": "Use virtual scrolling for large staff displays",
      "reasoning": "Rendering all notes at once caused performance issues with long pieces. Virtual scrolling only renders visible notes.",
      "confidence": 0.82,
      "frequency": 2,
      "lastUsed": "2024-01-11T11:30:00Z",
      "source": "correction",
      "performance_impact": "High - 80% rendering time reduction"
    }
  ]
}
```

### Business Rules and Domain Logic

#### Musical Theory Rules
```json
{
  "businessRules": [
    {
      "id": "rule_001",
      "rule": "measure_total_beats = time_signature_numerator",
      "context": "musical notation",
      "description": "Each measure must contain exactly the number of beats specified by the time signature numerator",
      "examples": [
        "4/4 time signature = 4 beats per measure",
        "3/4 time signature = 3 beats per measure",
        "2/4 time signature = 2 beats per measure"
      ],
      "violations": [
        "Measure with 5 beats in 4/4 time",
        "Measure with 2 beats in 3/4 time"
      ],
      "keywords": ["measure", "beats", "time signature", "numerator"]
    },
    {
      "id": "rule_002",
      "rule": "note_duration_sum <= measure_total_beats",
      "context": "musical notation",
      "description": "The sum of note durations in a measure cannot exceed the total beats allowed",
      "examples": [
        "Quarter note = 1 beat",
        "Half note = 2 beats",
        "Whole note = 4 beats (in 4/4 time)"
      ],
      "violations": [
        "5 quarter notes in a 4/4 measure",
        "3 half notes in a 4/4 measure"
      ],
      "keywords": ["note duration", "measure", "beats", "sum"]
    },
    {
      "id": "rule_003",
      "rule": "rest_duration_sum + note_duration_sum = measure_total_beats",
      "context": "musical notation",
      "description": "The sum of rest durations and note durations must equal the total beats in a measure",
      "examples": [
        "2 quarter notes + 2 quarter rests = 4 beats (4/4 time)",
        "1 half note + 2 quarter rests = 4 beats (4/4 time)"
      ],
      "violations": [
        "Incomplete measures (missing beats)",
        "Overflow measures (too many beats)"
      ],
      "keywords": ["rest", "note", "duration", "sum", "measure"]
    }
  ]
}
```

#### Piano-Specific Rules
```json
{
  "businessRules": [
    {
      "id": "rule_004",
      "rule": "piano_key_range = 88_keys (A0 to C8)",
      "context": "piano interface",
      "description": "Standard piano has 88 keys from A0 (lowest) to C8 (highest)",
      "examples": [
        "Middle C = C4",
        "Lowest key = A0",
        "Highest key = C8"
      ],
      "violations": [
        "Keys below A0",
        "Keys above C8"
      ],
      "keywords": ["piano", "keys", "range", "A0", "C8"]
    },
    {
      "id": "rule_005",
      "rule": "black_keys_pattern = 2-3-2-3-2-3-2",
      "context": "piano interface",
      "description": "Black keys follow a specific pattern: 2 black keys, then 3 black keys, repeating",
      "examples": [
        "C# and D# (2 black keys)",
        "F#, G#, A# (3 black keys)",
        "Pattern repeats across all octaves"
      ],
      "violations": [
        "Incorrect black key positioning",
        "Missing black keys in pattern"
      ],
      "keywords": ["black keys", "pattern", "piano", "octave"]
    }
  ]
}
```

### Anti-Patterns and Rejected Approaches

#### Technical Anti-Patterns
```json
{
  "antiPatterns": [
    {
      "id": "anti_001",
      "name": "SVG for staff notation rendering",
      "description": "Using SVG elements to render musical staff notation",
      "whyRejected": "Performance issues with complex note positioning, difficult to maintain, and poor rendering quality",
      "alternatives": ["MusicXML API", "Canvas-based rendering", "CSS-based positioning"],
      "frequency": 5,
      "lastRejected": "2024-01-15T10:30:00Z",
      "context": "staff notation rendering",
      "performance_impact": "High - 60% slower rendering"
    },
    {
      "id": "anti_002",
      "name": "HTML5 Audio API for piano sounds",
      "description": "Using HTML5 Audio API for piano sound playback",
      "whyRejected": "High latency (200ms+), poor timing control, limited polyphony, and audio glitches",
      "alternatives": ["Web Audio API", "Preloaded samples", "AudioWorklet"],
      "frequency": 4,
      "lastRejected": "2024-01-14T15:45:00Z",
      "context": "audio playback",
      "performance_impact": "High - 75% latency increase"
    },
    {
      "id": "anti_003",
      "name": "CSS Flexbox for note positioning",
      "description": "Using CSS Flexbox to position notes on the staff",
      "whyRejected": "Alignment issues with different note types, inconsistent spacing, and complex responsive behavior",
      "alternatives": ["Absolute positioning", "CSS Grid", "Calculated coordinates"],
      "frequency": 3,
      "lastRejected": "2024-01-13T09:20:00Z",
      "context": "note positioning",
      "performance_impact": "Medium - Layout thrashing"
    }
  ]
}
```

#### Architectural Anti-Patterns
```json
{
  "antiPatterns": [
    {
      "id": "anti_004",
      "name": "Monolithic component for staff rendering",
      "description": "Single large component handling all staff rendering logic",
      "whyRejected": "Difficult to maintain, poor performance, and hard to test individual features",
      "alternatives": ["Component composition", "Custom hooks", "Render props"],
      "frequency": 2,
      "lastRejected": "2024-01-10T16:20:00Z",
      "context": "component architecture",
      "maintainability_impact": "High - 80% harder to maintain"
    },
    {
      "id": "anti_005",
      "name": "Synchronous audio processing",
      "description": "Processing audio on the main thread",
      "whyRejected": "Causes UI stuttering, poor user experience, and blocks other operations",
      "alternatives": ["Web Workers", "AudioWorklet", "Async processing"],
      "frequency": 3,
      "lastRejected": "2024-01-12T14:10:00Z",
      "context": "audio processing",
      "performance_impact": "High - UI stuttering"
    }
  ]
}
```

### Domain-Specific Patterns

#### Successful Patterns
```json
{
  "patterns": [
    {
      "id": "pattern_001",
      "name": "MusicXML-based staff rendering",
      "description": "Use MusicXML API for structured staff notation rendering",
      "codeExample": `
// Use MusicXML API for staff rendering
const staffRenderer = new MusicXMLRenderer({
  container: staffContainer,
  timeSignature: { numerator: 4, denominator: 4 },
  keySignature: 'C'
});

staffRenderer.renderNotes(notes);
      `,
      "useCases": [
        "Staff notation display",
        "Note positioning",
        "Time signature handling"
      ],
      "successRate": 0.95,
      "performance": "High",
      "maintainability": "High"
    },
    {
      "id": "pattern_002",
      "name": "Web Audio API with preloaded samples",
      "description": "Use Web Audio API with preloaded audio samples for piano sounds",
      "codeExample": `
// Preload piano samples
const audioContext = new AudioContext();
const samples = await loadPianoSamples();

// Play note with Web Audio API
function playNote(note) {
  const source = audioContext.createBufferSource();
  source.buffer = samples[note];
  source.connect(audioContext.destination);
  source.start();
}
      `,
      "useCases": [
        "Piano sound playback",
        "Real-time audio",
        "Polyphonic playing"
      ],
      "successRate": 0.90,
      "performance": "High",
      "maintainability": "Medium"
    }
  ]
}
```

### Contextual Learning Examples

#### Learning from User Corrections
```json
{
  "correctionLearnings": [
    {
      "id": "correction_001",
      "originalSuggestion": "Use SVG for staff notation rendering",
      "userCorrection": "Use MusicXML API instead",
      "reasoning": "SVG approach caused performance issues and was difficult to maintain",
      "context": "staff notation rendering",
      "learnedAt": "2024-01-15T10:30:00Z",
      "confidence": 0.95,
      "appliedCount": 8
    },
    {
      "id": "correction_002",
      "originalSuggestion": "Use HTML5 Audio API for piano sounds",
      "userCorrection": "Use Web Audio API with preloaded samples",
      "reasoning": "HTML5 Audio had high latency and poor timing control",
      "context": "audio playback",
      "learnedAt": "2024-01-14T15:45:00Z",
      "confidence": 0.90,
      "appliedCount": 6
    }
  ]
}
```

#### Learning from Project Evolution
```json
{
  "evolutionLearnings": [
    {
      "id": "evolution_001",
      "context": "piano app development",
      "learning": "Staff notation complexity requires specialized libraries",
      "reasoning": "As the app evolved from simple notes to complex staff notation, custom solutions became inadequate",
      "solution": "Adopt MusicXML API for professional-grade notation",
      "learnedAt": "2024-01-15T10:30:00Z",
      "confidence": 0.88,
      "impact": "High - Enabled complex notation features"
    },
    {
      "id": "evolution_002",
      "context": "piano app development",
      "learning": "Real-time audio requires dedicated processing",
      "reasoning": "Main thread audio processing caused UI issues as the app grew in complexity",
      "solution": "Use Web Workers for audio processing",
      "learnedAt": "2024-01-12T14:10:00Z",
      "confidence": 0.85,
      "impact": "High - Eliminated UI stuttering"
    }
  ]
}
```

## E-commerce Application Domain Knowledge

### Business Rules
```json
{
  "businessRules": [
    {
      "id": "ecommerce_001",
      "rule": "cart_total = sum(item_price * quantity) + tax + shipping",
      "context": "shopping cart calculation",
      "description": "Cart total must include item prices, quantities, tax, and shipping",
      "examples": [
        "Item: $10, Quantity: 2, Tax: $1.60, Shipping: $5 = Total: $26.60",
        "Item: $25, Quantity: 1, Tax: $2.00, Shipping: $0 = Total: $27.00"
      ],
      "violations": [
        "Missing tax calculation",
        "Incorrect quantity multiplication",
        "Missing shipping costs"
      ],
      "keywords": ["cart", "total", "price", "quantity", "tax", "shipping"]
    },
    {
      "id": "ecommerce_002",
      "rule": "inventory_check_before_purchase",
      "context": "order processing",
      "description": "Must verify inventory availability before allowing purchase",
      "examples": [
        "Check stock before adding to cart",
        "Verify availability at checkout",
        "Reserve inventory during payment"
      ],
      "violations": [
        "Allowing purchase of out-of-stock items",
        "Overselling inventory",
        "Inventory inconsistencies"
      ],
      "keywords": ["inventory", "stock", "availability", "purchase"]
    }
  ]
}
```

### Technical Learnings
```json
{
  "learnings": [
    {
      "id": "ecommerce_001",
      "type": "technical",
      "context": "payment processing",
      "solution": "Use Stripe API with webhooks for payment confirmation",
      "reasoning": "Direct API calls can fail and don't provide reliable confirmation. Webhooks ensure payment status is accurately tracked.",
      "confidence": 0.92,
      "frequency": 5,
      "lastUsed": "2024-01-15T14:20:00Z",
      "source": "correction",
      "alternatives": ["Direct API calls", "Polling for status"],
      "reliability_impact": "High - 99.9% payment confirmation accuracy"
    }
  ]
}
```

## Financial Application Domain Knowledge

### Business Rules
```json
{
  "businessRules": [
    {
      "id": "finance_001",
      "rule": "account_balance = sum(credits) - sum(debits)",
      "context": "account balance calculation",
      "description": "Account balance is calculated as total credits minus total debits",
      "examples": [
        "Credits: $1000, Debits: $300 = Balance: $700",
        "Credits: $500, Debits: $800 = Balance: -$300"
      ],
      "violations": [
        "Incorrect balance calculation",
        "Missing transaction entries",
        "Double-counting transactions"
      ],
      "keywords": ["balance", "credits", "debits", "account"]
    },
    {
      "id": "finance_002",
      "rule": "transaction_validation_before_processing",
      "context": "transaction processing",
      "description": "All transactions must be validated before processing to ensure data integrity",
      "examples": [
        "Verify account exists",
        "Check sufficient funds",
        "Validate transaction amount",
        "Confirm user authorization"
      ],
      "violations": [
        "Processing invalid transactions",
        "Insufficient fund checks",
        "Unauthorized transactions"
      ],
      "keywords": ["transaction", "validation", "processing", "integrity"]
    }
  ]
}
```

## Gaming Application Domain Knowledge

### Technical Learnings
```json
{
  "learnings": [
    {
      "id": "gaming_001",
      "type": "performance",
      "context": "real-time game rendering",
      "solution": "Use requestAnimationFrame with delta time for smooth animation",
      "reasoning": "setTimeout caused inconsistent frame rates and stuttering. requestAnimationFrame provides smooth 60fps animation.",
      "confidence": 0.94,
      "frequency": 7,
      "lastUsed": "2024-01-15T16:45:00Z",
      "source": "correction",
      "alternatives": ["setTimeout", "setInterval"],
      "performance_impact": "High - Smooth 60fps animation"
    }
  ]
}
```

### Game-Specific Rules
```json
{
  "businessRules": [
    {
      "id": "gaming_001",
      "rule": "player_health = max(0, current_health - damage)",
      "context": "player health system",
      "description": "Player health cannot go below 0 when taking damage",
      "examples": [
        "Health: 50, Damage: 30 = New Health: 20",
        "Health: 10, Damage: 15 = New Health: 0 (not -5)"
      ],
      "violations": [
        "Negative health values",
        "Health exceeding maximum",
        "Invalid damage calculations"
      ],
      "keywords": ["health", "damage", "player", "game"]
    }
  ]
}
```

## Summary

These domain knowledge examples demonstrate how the Project Context Memory System would store and utilize:

1. **Technical Learnings**: API choices, performance optimizations, architectural decisions
2. **Business Rules**: Domain-specific logic and constraints
3. **Anti-Patterns**: Rejected approaches and why they were rejected
4. **Successful Patterns**: Proven solutions and their success rates
5. **Contextual Learning**: Learning from corrections and project evolution

This knowledge would enable Cursor AI to provide contextually relevant suggestions that respect established patterns and avoid previously rejected approaches, dramatically improving the development experience for complex, domain-specific projects.