# Voice Agent Design: Home Credit LAP Qualification

An AI voice agent for Home Credit that calls existing customers about a pre-approved Loan Against Property (LAP) offer of up to ₹75 lakh, runs a 7-point preliminary eligibility check, and hands qualified customers to a senior loan expert.

Built and tested on **Retell AI** (single-prompt agent, English, agent "Priya", female voice).

## Links

- **Published page (prompt summary, rules, audio and transcripts):** https://claude.ai/artifact/Vj4RQJvLNt1AbYqorA3yJk
- **Screen recording:** https://drive.google.com/file/d/1cD4VGArRX-9aEOI_6KVnn7gbMpxFxzP0/view?usp=sharing

## Repository contents

```
├── README.md
├── system_prompt.md        # the full system prompt used in Retell
├── test_plan.md            # 50 planned scenarios, assumptions, setup notes
├── transcripts/            # one transcript per test call
└── recordings/             # one audio file per test call
```

## How the prompt works

1. **Identity and voice:** role, runtime variables, language, strict gender, short spoken turns.
2. **Gates:** verification, busy customer with callback time, and the existing-loan / EMI-reduction transfer switch.
3. **Business rules:** one explicit eligibility table, with nothing invented (no minimum property value, no interest rates).
4. **Per-turn algorithm:** extract every fact, resolve corrections, check transfer, check disqualification, check the ₹75 lakh limit, update state, answer any diversion, then ask only the first missing item.
5. **Edge cases:** ambiguity (one clarifying question), fillers, interruptions, product questions answered only from the retrieved context, hard "never" rules.

The handoff happens only after all 7 items are answered and every criterion is met. Skipped items are asked before handoff.

## Eligibility rules

| Item | Eligible | Not eligible |
|---|---|---|
| Property type | Residential, Commercial, Industrial | Agricultural |
| Ownership | Sole or Joint | none |
| Original documents | Available for verification | Not available |
| Loan amount | Up to ₹75,00,000 | Above: explain limit, offer ₹75 lakh |
| Occupation + income | Salaried or Self-employed, paid via Bank | Cash income |
| Market value | Recorded only (no minimum) | none |
| Tenure | 3 to 15 years | Outside the range |

## Test results

| # | Scenario | Result |
|---|---|---|
| 1 | Happy path | Passed: qualified, handed to senior expert, agent ended call |
| 2 | Agricultural property | Passed: disqualified at property type, call ended |
| 3 | Cash income | Passed: disqualified at income answer, call ended |
| 4 | Existing loan / EMI reduction | Passed: transfer-specialist message, call ended |
| 5 | Busy customer | Passed: callback time captured (Monday, 5 PM), call ended |
| 6 | Combined: out-of-order answers, loan above ₹75 lakh, correction | Passed: skipped answered questions, explained the limit, accepted the corrected tenure, handed off only after all 7 items |
| 7 | Interest-rate question | Pending (add after testing) |

## Assumptions (the assignment is silent on these)

- An existing-loan or EMI-reduction mention takes priority over a disqualifying fact in the same sentence.
- The agent does not ask about existing loans proactively; it acts only when the customer mentions one.
- Mixed cash and bank income: the agent asks for the main source. Mainly bank counts as eligible.
- An occupation that is neither salaried nor self-employed is treated as not eligible for this offer.
- Customer cannot estimate market value after repeated tries: the agent moves on and records it as unsure.

## Setup used

Retell AI single-prompt agent, English, welcome message "Hello, may I speak with {{customer_name}}, please?", `end_call` function enabled. Dynamic variables: `company_name`, `customer_name`, `agent_name`, `agent_gender`, `language_to_speak`, `current_date`, `current_day`, `current_time`, `additional_context_from_rag`. The conversation history and latest utterance are supplied by the platform.
