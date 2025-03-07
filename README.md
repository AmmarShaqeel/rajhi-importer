# rajhi-importer
An importer for PDF Rajhi Card Statements (Should work for Cashback/Credit Cards).
Based on the beangulp framework: https://github.com/beancount/beangulp.


## Usage

Create a setup.py file as below.
Substitute the account/card number with your own.

Then run:

```python import.py extract YourCardStatement.PDF```


Setup.py File Example:


```
from rajhi_importer import rajhi_importer

importers = [

    rajhi_importer.Importer("Liabilities:Rajhi:CashbackCard", "SAR", "0000 00XX XXXX 0000"),
]

def clean_up_descriptions(extracted_entries):
    """Example filter function; clean up cruft from narrations.

    Args:
      extracted_entries: A list of directives.
    Returns:
      A new list of directives with possibly modified payees and narration
      fields.
    """
    clean_entries = []
    for entry in extracted_entries:
        if isinstance(entry, data.Transaction):
            if entry.narration and " / " in entry.narration:
                left_part, _ = entry.narration.split(" / ")
                entry = entry._replace(narration=left_part)
            if entry.payee and " / " in entry.payee:
                left_part, _ = entry.payee.split(" / ")
                entry = entry._replace(payee=left_part)
        clean_entries.append(entry)
    return clean_entries


def process_extracted_entries(extracted_entries_list, ledger_entries):
    """Example filter function; clean up cruft from narrations.

    Args:
      extracted_entries_list: A list of (filename, entries) pairs, where
        'entries' are the directives extract from 'filename'.
      ledger_entries: If provided, a list of directives from the existing
        ledger of the user. This is non-None if the user provided their
        ledger file as an option.
    Returns:
      A possibly different version of extracted_entries_list, a list of
      (filename, entries), to be printed.
    """
    return [(filename, clean_up_descriptions(entries), account, importer)
            for filename, entries, account, importer in extracted_entries_list]


hooks = [process_extracted_entries]

if __name__ == '__main__':
    ingest = beangulp.Ingest(importers, hooks)
    ingest()
```

