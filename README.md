### Project Summary: Anonymizing Customer Reviews Using Presidio and spaCy

In this project, we aimed to protect sensitive information within a dataset of customer reviews by anonymizing Personally Identifiable Information (PII) using Microsoft Presidio and spaCy. Below is a concise summary of the steps and processes involved in the project:

### Setup and Installation 📦

First, we installed the necessary Python libraries, including `pandas` for data manipulation, `presidio-analyzer` and `presidio-anonymizer` for PII detection and anonymization, and `spacy` for Named Entity Recognition (NER). We also downloaded the spaCy language model (`en_core_web_lg`) to enhance PII detection capabilities.

```bash
pip install pandas presidio-analyzer presidio-anonymizer spacy
python -m spacy download en_core_web_lg
```

### Loading and Exploring the Dataset 📊

We downloaded the "Customer Review" dataset from Kaggle, loaded it into a Pandas DataFrame, and explored its structure, focusing on columns that potentially contain PII, such as 'Customer name'.

```python
import pandas as pd

# Load the dataset
df = pd.read_csv('Customer Review.csv', encoding='ISO-8859-1')
print(df.head())
```

### Initializing Presidio with spaCy 🔍

We configured Presidio to use the spaCy model for NER by setting up the `AnalyzerEngine` with spaCy's `en_core_web_lg` model and initialized the `AnonymizerEngine` to handle the anonymization process.

```python
from presidio_analyzer import AnalyzerEngine
from presidio_analyzer.nlp_engine import NlpEngineProvider
from presidio_anonymizer import AnonymizerEngine
from presidio_anonymizer.entities import OperatorConfig

# Load spaCy model
import spacy
nlp = spacy.load("en_core_web_lg")

# Configure the NLP engine with spaCy
nlp_engine_provider = NlpEngineProvider(nlp_configuration={
    "nlp_engine_name": "spacy",
    "models": [{"lang_code": "en", "model_name": "en_core_web_lg"}]
})
nlp_engine = nlp_engine_provider.create_engine()

# Initialize Presidio Analyzer and Anonymizer with the spaCy NLP engine
analyzer = AnalyzerEngine(nlp_engine=nlp_engine)
anonymizer = AnonymizerEngine()
```

### Identifying and Anonymizing PII 🔒

We defined functions to detect PII in the text using Presidio’s analyzer and to anonymize the detected PII using Presidio’s anonymizer. We applied these functions to the 'Customer name' column in the dataset, replacing detected PII with a placeholder ('ANONYMISED').

```python
# Function to identify PII in text
def identify_pii(text):
    results = analyzer.analyze(text=text, language='en')
    return results

# Function to anonymize PII in text
def anonymize_text(text, results):
    anonymized_text = anonymizer.anonymize(
        text=text,
        analyzer_results=results,
        operators={'DEFAULT': OperatorConfig(operator_name='replace', params={'new_value': 'ANONYMISED'})}
    )
    return anonymized_text
```

We then applied these functions to the 'Customer name' column:

```python
# Anonymize the 'Customer name' column
def anonymize_column(column):
    anonymized_data = []
    for text in column:
        results = identify_pii(text)
        anonymized_result = anonymize_text(text, results)
        anonymized_data.append(anonymized_result.text)
    return anonymized_data

# Apply anonymization to the 'Customer name' column
df['AnonymizedCustomerName'] = anonymize_column(df['Customer name'])
print(df[['Customer name', 'AnonymizedCustomerName']].head())
```

### Saving the Anonymized Dataset 💾

Finally, we saved the anonymized dataset to a new CSV file, ensuring that the sensitive information is protected while retaining the overall structure and usefulness of the dataset.

```python
# Save the anonymized dataset to a new CSV file
df.to_csv('anonymized_customer_reviews.csv', index=False)
```

### Key Takeaways 📚

- **PII Detection and Anonymization:** Successfully used Presidio and spaCy to detect and anonymize PII in customer reviews, demonstrating an effective approach to data privacy.
- **Data Privacy:** Highlighted the importance of anonymizing sensitive information in datasets to comply with privacy regulations and protect user data.
- **Tool Integration:** Showcased the integration of powerful open-source tools (Presidio and spaCy) for practical data protection tasks.

### Pitfalls ⚠️

- **Incomplete Anonymization:** Not all names were successfully anonymized in the project. This highlights the limitation of relying solely on automated tools for PII detection and the need for additional manual verification steps or more advanced custom recognizers to ensure comprehensive anonymization.

This project provides a practical example of how to implement data anonymization techniques to enhance privacy and security in data processing workflows, while also recognizing the challenges and limitations faced in achieving complete anonymization.
