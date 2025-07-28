# OpenAI Fine-Tuning: Lipyum Auto Refund Decision

This project automates the decision process for refund requests in a service platform where field personnel submit cancellation claims. The model learns to decide whether to accept or reject each request based on structured inputs from historical decisions.

---

## Objective

To replace the manual final decision made by internal staff with an AI model that can:

- Review refund request reason  
- Analyze provider and partner call transcripts  
- Consider time elapsed, call attempts, customer/partner/system notes  
- Decide if the provider's reason is valid and non-abusive  
- Return a decision (`Accepted by Personnel` or `Rejected by Personnel`) along with a short explanation in Turkish

---

## Model Used

- Base model: [`gpt-4o-mini-2024-07-18`](https://platform.openai.com/docs/models/gpt-4o-mini)
- Method: Supervised Fine-Tuning (Chat format)
- Platform: OpenAI API (fine-tuning v2)
- File type: JSONL (`messages`: system + user + assistant)

---

## Data Preparation Pipeline

The entire process is implemented in 8 sequential Jupyter Notebook cells:

| Step                   | Description                                                          |
| ---------------------- | -------------------------------------------------------------------- |
| 01_settings_check      | Environment setup, data load, model & key check                      |
| 02_data_cleaning       | Cleans timestamps, fills missing text, computes time differences     |
| 03_feature_call_counts | Counts provider and partner calls from transcripts                   |
| 04_filter_and_prompt   | Formats training data in Chat JSONL format                           |
| 05_cost_estimation     | Computes total tokens and estimated cost before training             |
| 06_fine_tune_start     | Uploads file and starts fine-tuning                                  |
| 07_follow_job          | Tracks training status and saves model name when ready               |
| 08_model_test          | Sends structured test input to the trained model and prints response |

> Cells `09–12` were excluded: They sampled limited evaluation sets (e.g. 100 examples) which added extra cost. Since the full dataset is already used for training and can be tested directly, these steps were removed for efficiency.

---

## Training Format (OpenAI Chat JSONL)

Each training example contains:

```json
{
  "messages": [
    { "role": "system", "content": "Defines AI role and refund policy" },
    { "role": "user", "content": "Structured input fields (reasons, transcripts, counts, time, etc.)" },
    { "role": "assistant", "content": "{\"decision\": \"Accepted by Personnel\", \"reason\": \"Müşteri vazgeçtiği belirtildi.\"}" }
  ]
}
```

---

## Cost Considerations

Token count and cost are calculated in advance using `tiktoken`.  
While cost estimates are large-scale accurate, OpenAI's backend may discard malformed rows or tokenize differently.  
Thus, cost projections reflect a **high-confidence approximation**, not a guaranteed total.  
True cost will be shown in OpenAI's usage dashboard after training.

---

## Compatibility with Business Logic

This project strictly follows the business flow described in `Lipyum Oto Karar.pdf`, including:

- Terminology: Partner, Provider, Customer, Internal Personnel  
- Workflow: Job assignment → Provider call → Refund request → Partner check → Final decision  
- Columns used: Refund reason, call data, rejection notes, timestamps, etc.  
- Filtering: Only "Personel Kabul Etti" and "Personel Redetti" are used

---

## Outputs

Once trained, the model receives structured prompt input and returns:

```json
{
  "decision": "Accepted by Personnel",
  "reason": "Müşteri adres bilgisinin yanlış olduğunu belirtti."
}
```

---

## Resources

- [OpenAI Fine-Tuning Docs](https://platform.openai.com/docs/guides/fine-tuning)  
- [OpenAI gpt-4o-mini Model Reference](https://platform.openai.com/docs/models/gpt-4o-mini)  
- Project reference document: `Lipyum Oto Karar.pdf`

---

## Notes

- No confidential or personal information is included  
- Dataset is cleaned and normalized for model compatibility  
- The model is expected to generalize well on real-world refund decisions  
- Further tests will be run after deployment with the trained model

---

## Summary

The project has successfully implemented a full fine-tuning pipeline using real support data.  
The trained model can evaluate refund requests and provide decisions with brief Turkish justifications, aligning with internal review criteria.

> The model will be fine-tuned and tested using the company's OpenAI account.
