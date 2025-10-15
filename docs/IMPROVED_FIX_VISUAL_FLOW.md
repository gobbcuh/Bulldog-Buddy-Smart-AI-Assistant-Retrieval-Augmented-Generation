# Visual Flow - Improved Fix

## Complete Query Processing Flow

```
┌─────────────────────────────────────┐
│   User Asks Question                │
└───────────────┬─────────────────────┘
                │
                ▼
┌────────────────────────────────────────┐
│  Is this a follow-up question?         │
└────────┬───────────────────────────────┘
         │
    ┌────┴────┐
    │         │
   YES       NO
    │         │
    │         └──────────────────┐
    │                            │
    ▼                            ▼
┌─────────────────┐    ┌──────────────────────┐
│ Rewrite with    │    │ Check if related to  │
│ conversation    │    │ previous context     │
│ context         │    │                      │
└────────┬────────┘    └──────────┬───────────┘
         │                        │
         │                   ┌────┴────┐
         │                   │         │
         │                  YES       NO
         │                   │         │
         │                   │    ┌────▼─────┐
         │                   │    │  Clear   │
         │                   │    │  Cache   │
         │                   │    └────┬─────┘
         │                   │         │
         └───────────────────┴─────────┘
                             │
                             ▼
              ┌──────────────────────────┐
              │  Retrieve Chunks from    │
              │  Vector Database         │
              └──────────┬───────────────┘
                         │
                         ▼
              ┌──────────────────────────┐
              │  🆕 CHECK RELEVANCE:     │
              │  Are chunks relevant to  │
              │  THIS question?          │
              │  (keyword overlap > 15%) │
              └──────────┬───────────────┘
                         │
                    ┌────┴────┐
                    │         │
                  YES        NO
                    │         │
                    │         ▼
                    │    ┌─────────────────────┐
                    │    │ ⚠️  IRRELEVANT!     │
                    │    │ Clear cache         │
                    │    │ Re-retrieve chunks  │
                    │    └──────┬──────────────┘
                    │           │
                    │           ▼
                    │    ┌─────────────────────┐
                    │    │ Check relevance     │
                    │    │ again               │
                    │    └──────┬──────────────┘
                    │           │
                    │      ┌────┴────┐
                    │      │         │
                    │     YES       NO
                    │      │         │
                    │      │         ▼
                    │      │    ┌──────────────┐
                    │      │    │ Switch to    │
                    │      │    │ General      │
                    │      │    │ Knowledge    │
                    │      │    │ Mode         │
                    │      │    └──────┬───────┘
                    │      │           │
                    └──────┴───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Update Cache with    │
                │ retrieved chunks     │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Generate Response    │
                │ using LLM            │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────────────┐
                │ 🆕 POST-PROCESS:              │
                │ Ensure Proper Formatting     │
                │ - Add line breaks if missing │
                │ - Format bullet points       │
                │ - Space paragraphs properly  │
                └──────────┬───────────────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Return Formatted     │
                │ Response to User     │
                └──────────────────────┘
```

## Relevance Checking Algorithm

```
┌────────────────────────────────────────┐
│ Query: "what if I got inc grade"       │
│ Keywords: [inc, got, grade]            │
└──────────────┬─────────────────────────┘
               │
               ▼
┌────────────────────────────────────────┐
│ Remove stop words:                     │
│ [what, if, I]                          │
│ Remaining: [inc, got, grade]           │
└──────────────┬─────────────────────────┘
               │
               ▼
┌────────────────────────────────────────┐
│ For each retrieved chunk:              │
│                                        │
│ Chunk 1: "grading scale 4.0..."       │
│ Contains: grade ✓                      │
│ Match: 1/3 = 33%                       │
│                                        │
│ Chunk 2: "incomplete INC grades..."   │
│ Contains: inc ✓, grade ✓               │
│ Match: 2/3 = 67%                       │
│                                        │
│ Chunk 3: "tuition fees payment..."    │
│ Contains: (none)                       │
│ Match: 0/3 = 0%                        │
└──────────────┬─────────────────────────┘
               │
               ▼
┌────────────────────────────────────────┐
│ Calculate Average:                     │
│ (33% + 67% + 0%) / 3 = 33%            │
│                                        │
│ Compare to Threshold (15%):           │
│ 33% > 15% ✓                           │
│                                        │
│ Result: CHUNKS ARE RELEVANT ✅         │
└────────────────────────────────────────┘
```

## Before vs After Comparison

### OLD SYSTEM (Before Improved Fix)

```
Q1: "What's the tuition fee?"
    ↓
[Retrieve Financial Chunks]
    ↓
[Store in Cache] ✓
    ↓
"The tuition fee is $10,000 per semester for undergraduate
students. This covers all course fees and materials..."
    ↓
❌ No line breaks, compact text


Q2: "What about INC grades?"
    ↓
[Detected as follow-up]
    ↓
[Use Conversational Chain]
    ↓
[Retrieved: Still has Financial Chunks cached!] ❌
    ↓
"INC grades allow students to complete work later. Regarding
the tuition you asked about, payment plans are available..."
    ↓
❌ Mixed contexts! Mentioned tuition in grading answer
❌ Still no line breaks
```

### NEW SYSTEM (After Improved Fix)

```
Q1: "What's the tuition fee?"
    ↓
[Retrieve Financial Chunks]
    ↓
[Check Relevance: 85% overlap ✓]
    ↓
[Store in Cache] ✓
    ↓
"The tuition fee is $10,000 per semester for undergraduate
students.

This covers all course fees and materials.

Payment options include:
• Full payment upfront
• Semester installments
• Monthly payment plans"
    ↓
✅ Proper line breaks and formatting!


Q2: "What about INC grades?"
    ↓
[Detected as follow-up]
    ↓
[Use Conversational Chain]
    ↓
[Retrieved: Financial Chunks]
    ↓
[Check Relevance: 5% overlap ❌] 🆕
    ↓
[CLEAR CACHE + RE-RETRIEVE] 🆕
    ↓
[Retrieved: Grading Chunks]
    ↓
[Check Relevance: 70% overlap ✓] 🆕
    ↓
[Update Cache with Grading Chunks]
    ↓
"According to university policy, an incomplete (INC) grade
allows you to complete coursework later.

However, the handbook doesn't specifically detail how
individual mid-semester grades are factored into the final
calculation.

I recommend speaking with your academic advisor for details
specific to your program."
    ↓
✅ Correct context (grading only, no tuition mention)
✅ Proper line breaks and formatting!
```

## Formatting Post-Processor Logic

```
┌──────────────────────────────┐
│ LLM Output:                  │
│ "Text text text. More text.  │
│  Even more text. Bullet 1    │
│  Bullet 2"                   │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ Check: Does it have \n\n?    │
└──────────┬───────────────────┘
           │
      ┌────┴────┐
      │         │
     YES       NO
      │         │
      ▼         ▼
   Return    Process
   as-is     with regex
      │         │
      │         ▼
      │    ┌────────────────────┐
      │    │ Add breaks after:  │
      │    │ . ! ? followed by  │
      │    │ capital letter     │
      │    └────────┬───────────┘
      │             │
      │             ▼
      │    ┌────────────────────┐
      │    │ Add breaks before: │
      │    │ • - * bullets      │
      │    │ 1. 2. numbers      │
      │    └────────┬───────────┘
      │             │
      │             ▼
      │    ┌────────────────────┐
      │    │ Add breaks after:  │
      │    │ : colons           │
      │    └────────┬───────────┘
      │             │
      └─────────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │ Formatted Output:    │
         │ "Text text text.     │
         │                      │
         │ More text.           │
         │                      │
         │ Even more text.      │
         │ • Bullet 1           │
         │ • Bullet 2"          │
         └──────────────────────┘
```

## Relevance Threshold Impact

```
        LOW THRESHOLD (10%)        BALANCED (15%)         HIGH THRESHOLD (25%)
              ↓                          ↓                        ↓
    ┌─────────────────┐        ┌─────────────────┐      ┌─────────────────┐
    │ More Lenient    │        │ Current Default │      │ Very Strict     │
    │                 │        │                 │      │                 │
    │ Accepts chunks  │        │ Good balance    │      │ May reject      │
    │ with minimal    │        │ between false   │      │ valid chunks    │
    │ keyword overlap │        │ positives and   │      │                 │
    │                 │        │ false negatives │      │ Fewer false     │
    │ More false      │        │                 │      │ positives but   │
    │ positives       │        │ ✅ RECOMMENDED  │      │ more false      │
    │                 │        │                 │      │ negatives       │
    └─────────────────┘        └─────────────────┘      └─────────────────┘
```

## Cache State Machine

```
      START
        │
        ▼
   ┌─────────┐
   │  Empty  │
   │  Cache  │
   └────┬────┘
        │ Question Asked
        ▼
   ┌─────────┐
   │ Cached  │◄────┐
   │ Context │     │ Related Question
   └────┬────┘     │
        │          │
        │ Relevance Check
        ▼          │
   ┌─────────┐    │
   │Relevant?│    │
   └────┬────┘    │
        │         │
   ┌────┴────┐   │
   │         │   │
  YES       NO   │
   │         │   │
   │         ▼   │
   │    ┌─────────┐
   │    │ Clear & │
   │    │ Refresh │
   │    └────┬────┘
   │         │
   └─────────┴─────► Continue
```

## Legend

```
┌─────┐
│     │  Process/State
└─────┘

   ▼     Flow Direction

  🆕     New Feature Added

  ✅     Success/Correct

  ❌     Problem/Error

  ⚠️      Warning/Attention Needed
```

## Summary in One Diagram

```
           USER QUESTION
                │
                ▼
        ┌───────────────┐
        │  Detect Type  │
        └───────┬───────┘
                │
        ┌───────┴───────┐
        │               │
    Follow-up         New
        │               │
        ▼               ▼
   [Rewrite Q]    [Check Cache]
        │               │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │  Retrieve     │
        │  Chunks       │
        └───────┬───────┘
                │
                ▼
        ┌───────────────────┐
        │ 🆕 CHECK         │
        │ RELEVANCE        │
        │ Keywords Match?  │
        └───────┬───────────┘
                │
        ┌───────┴───────┐
        │               │
       YES             NO
        │               │
        │               ▼
        │       [Clear & Retry]
        │               │
        │               ▼
        │       [Still Bad?]
        │               │
        │               ▼
        │       [General Mode]
        │               │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │  Generate     │
        │  Response     │
        └───────┬───────┘
                │
                ▼
        ┌───────────────────┐
        │ 🆕 FORMAT         │
        │ Add Line Breaks   │
        │ Space Paragraphs  │
        └───────┬───────────┘
                │
                ▼
        ┌───────────────┐
        │  Return to    │
        │  User         │
        └───────────────┘
```
