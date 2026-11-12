# Request and response samples

[Back to the class](../README.md) · [Speaker notes](../docs/speaker-notes.md)

| File | What it is | Run by us? |
| --- | --- | --- |
| [jev-request.json](jev-request.json) | The request the example sends for one line: three questions over one state | Yes |
| [jev-response.json](jev-response.json) | The answer `jev-1.14.0` gave on November 10, 2026 | Yes |
| [microsoft-decision-1-request.json](microsoft-decision-1-request.json) | The same question in the shape Microsoft Learn documents | No |
| [gemini-decisions-request.json](gemini-decisions-request.json) | The same question in the shape Google's documentation describes | No |

The text in every sample is invented: "Medical Director, Intensive Care Unit".

## Try the Jev call yourself

```bash
curl -s https://api.typesafe.ai/v1/systemone \
  -H "Authorization: Bearer $TYPESAFE_API_KEY" \
  -H "Content-Type: application/json" \
  -d @jev-request.json
```

## Reading the response

- `time_class.probabilities` sums to 1 across the ten options. The example
  adds up the options that share a class to get the probability of each
  class.
- `time_class.confidence` is how concentrated those probabilities are. It is
  not the chance the answer is right.
- `is_provider_component.noul` is a single probability of yes. Notice that
  the model was certain the phrase is administration (1.00) and gave only
  0.74 to the yes or no question about the same idea. Two wordings of one
  question do not have to agree, which is a reason to test your wording on
  your own data.
- The raw response also carries a `usage` block with the input token count.
  It is left out of the sample because the application's route returns the
  answers only. That call used about 850 input tokens; nothing is generated.
- The application shuffles the order of the options for each payload. The
  request file shows them in their canonical order.

## Porting between vendors

| | Jev | Microsoft-Decision-1 | Gemini Decision 1 |
| --- | --- | --- | --- |
| Where the text goes | `state` | `state` | `input` |
| Where the questions go | `questions` | `questions` | `questions` |
| `model` is | A model id | Your deployment name | A model id |
| Auth header | `Authorization: Bearer` | `api-key`, or an Entra ID token | `x-goog-api-key` |

The Microsoft and Google files were written from the vendors' documentation
on November 11, 2026 and have not been sent to either service. Check them
against the current documentation before relying on them.
